

## 自动生成JNI代码

[Lynx](https://github.com/hxxft/lynx-native)内核是由C++编写，方便跨平台使用。这样在Android端与Java层通信就需要使用JNI，Lynx在JNI层为了避免直接手写JNI注册代码以及反射调用Java的代码，使用自动化的方式来自动生成这部分代码。

### JNI的注册方式
#####1. Java调用C/C++方法

通常Java调用C/C++方法的JNI方法注册分为静态注册和动态注册两种。

* 静态注册的方式

将Java中的Native方法在C/C++文件声明对应成Java\_\$PackageName_\$MethodName(JNIEnv *env,  args…) 完成完成静态注册。以Lynx代码中的自动化测试模块代码<span id="GTestDriver">GTestDriver.java</span>为例

  ```java
package com.lynx.gtest;

...
      
public class GTestDriver {
    ...
    //java native 方法
    private static native int nativeRunGTestsNative(String[] gtestCmdLineArgs);
}
  ```
  对于C方法的编写如下

```c++
#include <jni.h>
...
JNIEXPORT jint Java_com_lynx_gtest_nativeRunGTestsNative(JNIEnv *env, jobjectArray gtestCmdLineArgs)
...
```

从例子中可以看出静态注册的名字非常长，不便于书写。并且在初次调用的时候需要依据名字找到对应方法，对于大型工程中方法数多的情况下，效率低且易出错。

* 动态注册方式

通过在C/C++中声明一个JNINativeMethod nativeMethod[]数组，然后在JNI_OnLoad中调用RegisterNatives方法来完成动态注册。譬如上面的例子用动态注册的方式为：

  ```c++
#include <jni.h>
...
static jint RunGTestsNative(JNIEnv *env,  jclass jcaller, jobjectArray gtestCmdLineArgs) {
....
}
  
static const JNINativeMethod kMethodsGTestDriver[] = {
    { "nativeRunGTestsNative",
	  "("
	  "[Ljava/lang/String;"
	  ")"
	  "I", reinterpret_cast<void*>(RunGTestsNative) },
};
...
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *jvm, void *reserved) {
    JNIEnv *env;
    if ((*jvm) -> GetEnv(jvm, (void**) &env, JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }

    jclass clz = (*env) -> FindClass(env, "com/lynx/gtest/GTestDriver");

    (*env) -> RegisterNatives(env, clz, kMethodsGTestDriver, sizeof(kMethodsGTestDriver) / sizeof(kMethodsGTestDriver[0]));

    return JNI_VERSION_1_6;
}
  
  ```

动态注册是Lynx JNI的基础，后面展开介绍。 

#####2. C/C++调用Java方法

JNI的C代码调用Java代码。实现原理：使用JNI提供的反射接口来反射得到Java方法，进行调用。以Lynx代码中的文本测量的代码<span id="LabelMeasurer">LabelMeasurer.java</span>为例

```java
package com.lynx.core;
...
public class LabelMeasurer {
	...
    @CalledByNative
    public static Size measureLabelSize(String text, Style style, int width,
                                        int widthMode, int height, int heightMode) {
        ...
    }
}
```

在C/C++代码中调用measureLabelSize需要通过反射来完成，基本实现如下

```c
// 查找LabelMeasurer类
jclass clazz = (*env)->FindClass(env,"com/lynx/core/LabelMeasurer");

//获取measureLabelSize方法
jmethodID method = (*env)->GetMethodID(env,clazz, "measureLabelSize", "(Ljava/lang/String;Lcom/lynx/base/Style;IIII)Lcom/lynx/base/Size;");

//执行
jobject ret = env->CallStaticObjectMethod(clazz, method, text, style,
                                          width, widthMode, height, heightMode);
```

编写这样的调用会有非常多的重复代码，当接口需要进行修改时非常不方便。



### Lynx JNI 自动生成方式

从上述介绍可得，JNI的注册是一个重复性的操作。Lynx为了提高效率，将JNI的注册交由自动化脚本生成。

##### 1. Java调用C/C++方法

Lynx上对Native注册进行了约定，所有对Java注册的Native方法都以native开头，自动化脚本会在Java文件中找到这些方法，并自动生成相应的文件。

同样以[GTestDriver.java](#GTestDriver)为例，可以看到GTestDriver中包含JNI的Native方法nativeRunGTestsNative，并且以约定的native关键开头。对于这个文件会由prebuild.sh脚本执行生成一个GTestDriver_jni.h的头文件

```c++
#ifndef com_lynx_gtest_GTestDriver_JNI
#define com_lynx_gtest_GTestDriver_JNI

#include <jni.h>

#include "base/android/android_jni.h"

// Step 1: forward declarations.
namespace {
const char kGTestDriverClassPath[] = "com/lynx/gtest/GTestDriver";
// Leaking this jclass as we cannot use LazyInstance from some threads.
jclass g_GTestDriver_clazz = NULL;
#define GTestDriver_clazz(env) g_GTestDriver_clazz

}  // namespace

static jint RunGTestsNative(JNIEnv* env, jclass jcaller,
    jobjectArray gtestCmdLineArgs);

// Step 2: method stubs.

// Step 3: RegisterNatives.

static const JNINativeMethod kMethodsGTestDriver[] = {
    { "nativeRunGTestsNative",
"("
"[Ljava/lang/String;"
")"
"I", reinterpret_cast<void*>(RunGTestsNative) },
};

static bool RegisterNativesImpl(JNIEnv* env) {

  g_GTestDriver_clazz = reinterpret_cast<jclass>(env->NewGlobalRef(
      base::android::GetClass(env, kGTestDriverClassPath).Get()));

  const int kMethodsGTestDriverSize =
      sizeof(kMethodsGTestDriver)/sizeof(kMethodsGTestDriver[0]);

  if (env->RegisterNatives(GTestDriver_clazz(env),
                           kMethodsGTestDriver,
                           kMethodsGTestDriverSize) < 0) {
    return false;
  }

  return true;
}

#endif  // com_lynx_gtest_GTestDriver_JNI
```

Lynx的JNI使用的动态注册方式，将Java文件中定义的nativeRunGTestsNative与RunGTestsNative函数指针关联，这样对头文件中定义的RunGTestsNative方法进行实现，并在jni_onload的时候调用RegisterNativesImpl就可以实现对JNI方法的完整注册。这个方式比起JNI原有的方式也简单很多，隐藏了很多重复性质的代码。



##### 2. C/C++调用Java方法

Lynx对C/C++调用Java方法也做了定义，如果在Java文件中定义了可以被C/C++调用的代码，可以在方法前加上@CalledByNative，自动化脚步会在java文件中找到这些方法，并自动生成相应的文件。同样以上文中的[LabelMeasurer.java](#LabelMeasurer)为例。

在Java文件中申明了一个可以被C/C++使用的函数measureLabelSize。进过prebuild.sh脚本处理之后会生成LabelMeasurer_jni.h

```c++
#ifndef com_lynx_core_LabelMeasurer_JNI
#define com_lynx_core_LabelMeasurer_JNI

#include <jni.h>

#include "base/android/android_jni.h"

// Step 1: forward declarations.
namespace {
const char kLabelMeasurerClassPath[] = "com/lynx/core/LabelMeasurer";
// Leaking this jclass as we cannot use LazyInstance from some threads.
jclass g_LabelMeasurer_clazz = NULL;
#define LabelMeasurer_clazz(env) g_LabelMeasurer_clazz

}  // namespace

// Step 2: method stubs.

static intptr_t g_LabelMeasurer_measureLabelSize = 0;
static base::android::ScopedLocalJavaRef<jobject>
    Java_LabelMeasurer_measureLabelSize(JNIEnv* env, jstring text,
    jobject style,
    int width,
    int widthMode,
    int height,
    int heightMode) {
  jmethodID method_id =
      base::android::GetMethod(
      env, LabelMeasurer_clazz(env),
      base::android::STATIC_METHOD,
      "measureLabelSize",

"("
"Ljava/lang/String;"
"Lcom/lynx/base/Style;"
"I"
"I"
"I"
"I"
")"
"Lcom/lynx/base/Size;",
      &g_LabelMeasurer_measureLabelSize);

  jobject ret =
      env->CallStaticObjectMethod(LabelMeasurer_clazz(env),
          method_id, text, style, int(width), int(widthMode), int(height),
              int(heightMode));
  base::android::CheckException(env);
  return base::android::ScopedLocalJavaRef<jobject>(env, ret);
}

// Step 3: RegisterNatives.

static bool RegisterNativesImpl(JNIEnv* env) {

  g_LabelMeasurer_clazz = reinterpret_cast<jclass>(env->NewGlobalRef(
      base::android::GetClass(env, kLabelMeasurerClassPath).Get()));

  return true;
}

#endif  // com_lynx_core_LabelMeasurer_JNI
```

这样在使用的时候就可以直接引入头文件，并调用Java_LabelMeasurer_measureLabelSize方法即可。使用起来也是非常简便的。

自动生成的文件省去了编写反射获取函数方法以及调用的处理，全部有脚本直接生成，对于获取method id的方式，Lynx做了一层封装，可以对method id进行保存，这样查找只会在第一次调用的时候执行，节约整体调用时间 , 具体可以base/android/android_jni.h 中查看。

同时Lynx也对jobject进行了自动引用的处理，在不再使用jobject对象的时候会自动进行DeleteLocalRef，避免忘记释放后local ref过多超出最大值的情况，具体可以在 base/android/scoped_java_ref.h 中查看。


### 如何在已有工程中使用

1. git clone https://github.com/hxxft/lynx-native.git

2. 抽取所需文件并将文件添加入CMakeLists.txt

   > base/android/android_jni.h 
   >
   > base/android/android_jni.cc 
   >
   > base/android/java_type.h
   >
   > base/android/java_type.cc
   >
   > base/android/scoped\_java\_ref.h
   >
   > base/android/scoped\_java\_ref.cc

3. 抽取build文件夹,将jni_load.cc加入CMakeLists.txt，并根据需求修改此文件

4. 修改jni_files加入需要自动生成JNI代码的Java文件

5. 编写Java文件，使用前面讲解时使用的方法注释函数或者修饰方法

6. 修改prebuild.sh中ROOT\_LYNX\_JAVA\_PATH等路径，根据自己工程配置进行修改

7. 在编译之前执行prebuild.sh生成所需要的文件

### 总结

这篇文章主要介绍Lynx的JNI自动生成的方法，自动生成的方法省去了大部分重复代码，因此在编写代码过程中可以专注于方法的实现上，对需要使用JNI的工程来说，可以提供巨大的便利。

请持续关注 [Lynx](https://github.com/hxxft/lynx-native)，一个高性能跨平台开发框架。
