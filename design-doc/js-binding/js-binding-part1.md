###Lynx 之 JS Binding 技术（1）

#### 背景

[Lynx](https://github.com/hxxft/lynx-native) 作为一个基于 JavaScript 语言（后续简称 JS ）的跨平台开发框架，与 JS 的通信是"与生俱来"的，框架和 JS 引擎打交道是必不可少的能力。JS 引擎提供了 Extension 功能，提供接入方间接和 JS 通信的桥梁，Lynx 的 JS Binding 正是基于这个能力进行了封装，构建一套基础的 JS API，将能力开放给前端开发者。

当前主流浏览器基本都拥有自己的 JS 引擎，在当前移动端最为流行的 JS 引擎属 V8 和 JavascriptCore (别名 Nitro，后续简称 JSC)，Lynx 框架围绕这两个引擎打造高效的 JS Binding。

JS Binding 最主要的目的是利用 JS 引擎和 JS 通信，开放底层框架能力，也可以称它为 JS Bridge，它决定了 JS 和框架层通信的速度。Lynx 早期版本为了快速实现高效通信，依赖 V8 引擎（后续简称 V8），使用的是“纯粹”的 Extension 方式，即依照 V8 的拓展方式实现了 JS Binding，建立一套 JS API ，在 Android 系统上首先实现了渲染层的平台拓展。

Lynx 以这种方式在早期快速实现可靠高效的通信能力。但当 Lynx 把平台拓展到 iOS 时，由于 V8 无法在 iOS 平台使用，JS Binding 必须把 V8 切换成 JSC ，所有关于 V8 的类和函数定义以及初始化流程，均要替换成 JSC 的标准。第一个 JSC 版本的 JS Binding 是基于 JSC iOS 标准实现的。而第二个 JSC 版本的 JS Binding 是基于纯 JSC 标准实现的，这次改动的目的是希望能统一 Android 和 iOS 的底层 JS 引擎。

本文主要介绍主流 JS 引擎使用姿势及 Lynx 中 JS Binding 的内存管理方式，作为 JS Binding 技术演进分析的铺垫。



#### JS 引擎使用姿势

在了解 Lynx 中 JS Binding 技术的做法前，先了解一下 V8 和 JSC 在初始化和 Extension 方面的标准实现，从中发现两个引擎的异同，当掌握了基础的用法之后，能更好的理解 Lynx 中 JS Binding 的发展路线。下面 Extension 拓展以  example 对象为例。

example.h 头文件，这个类定义了即将暴露给 JS 端的 example 对象所具有的接口，包括 TestStatic 和 TestDynamic 方法及变量 num 的设置和获取。

```c++
class Example {
public:
    Example();
    virtual ~Example();
	
	void TestStatic();
    void TestDynamic();
    
    int num();
    void set_num(int num);
    
private:
    void Initialize();
};
```

在具体实现代码中，主要功能是创建 JS 上下文，创建 example 的 JS 对象，静态注册 testStatic 方法和 num 变量，动态注册 testDynamic 并暴露到上下文中。完成后可以通过在 JS 端使用如下代码访问到 example c++ 对象的接口。

```js
example.testStatic();
example.testDynamic();
example.num = 99;
console.log(example.num);
```

接下来将分析两个引擎中的实现代码，包括环境初始化和 Extension 方式，代码主要关注以下点

- 如何初始化环境及运行上下文
- 如何关联 c++ 对象和 JS 对象
- 如何创建对象，并注册到上下文中
- 如何向在 JS 引擎对象原型中静态注册变量和方法的钩子
- 如何向在 JS 引擎对象中动态注册方法钩子
- 如何销毁虚拟机

> 静态注册指的是对 JS 的原型 prototype 设置属性、方法及钩子函数，从持有该原型的构造函数创建的对象均有设置的方法和属性及钩子函数。
>
> 动态注册指直接对 JS 对象设置方法的钩子函数，仅有被设置过的对象才拥有被设置的方法。动态注册属性钩子函数的方式在 JS 引擎中暂时没有提供直接的方式.



**V8 初始化和 Extension 方式**

example_v8.cc 文件，以下为 V8 Extension 示例工程部分代码，完整代码请看[附录](#附录) 。整体流程总结如下：

1. 初始化 V8 的环境

   ```c++
   V8::InitializeICUDefaultLocation(argv[0]);
   V8::InitializeExternalStartupData(argv[0]);
   v8::Platform* platform = v8::platform::CreateDefaultPlatform();
   V8::InitializePlatform(platform);
   v8::V8::Initialize();
   ```

2. 创建 global 对象模板，据此创建 JS 运行上下文 context，从 context 中获取 global 对象

   ```c++
   // 创建isolate
   v8::Isolate* isolate = v8::Isolate::New(create_params);
   v8::Isolate::Scope isolate_scope(isolate);
   v8::HandleScope handle_scope(isolate);
   // 创建global 对象模板
   v8::Local <v8::ObjectTemplate> global_template = v8::ObjectTemplate::New(isolate);
   // 创建 JS 运行上下文 context
   v8::Local <v8::Context> context = v8::Context::New(isolate, nullptr, global_template);
   v8::Context::Scope context_scope(context);
   //  context 中获取 global 对象
   v8::Local<v8::Object> global = context->Global();
   ```

3. 创建 example 对象的构造函数模板，在构造函数模板中获取原型模板，并设置静态方法和变量的钩子

   ```c++
   // 创建 example 的构造函数模板, 使用该 c++ 类的初始化函数作为参数（函数钩子），初始化构造器函数模
   // 板。即当调用构造函数创建对象时，会调用该钩子函数做构造处理
   v8::Local<v8::FunctionTemplate> example_tpl = v8::FunctionTemplate::New(isolate);
   // 设置构造函数模板的类名
   example_tpl->SetClassName(V8Helper::ConvertToV8String(isolate, "Example"));
   // 设置内部关联 c++ 对象的数量为 1
   example_tpl->InstanceTemplate()->SetInternalFieldCount(1);
   // 设置构造函数模板中的原型模板的对应函数名的钩子
   example_tpl->PrototypeTemplate()->Set(V8Helper::ConvertToV8String(isolate, "testStatic"), v8::FunctionTemplate::New(isolate, TestStaticCallback));
   // 设置构造函数模板中的原型模板的属性的 Get 和 Set 钩子方法
   example_tpl->PrototypeTemplate()->SetAccessor(V8Helper::ConvertToV8String(isolate, "num"), GetNumCallback, SetNumCallback);
   ```

   用于静态注册的函数钩子，包括 testStatic 方法钩子和 num 的 get / set 钩子

   ```c++
   // example.testStatic() 调用时对应的 c++ 函数钩子
   static void TestStaticCallback(const v8::FunctionCallbackInfo <v8::Value> &args) {
       Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
       example->TestStatic();
   }

   // console.log(example.num) 调用时对应触发的 c++ 钩子函数
   static void GetNumCallback(v8::Local<v8::String> property, const PropertyCallbackInfo<Value>& info) {
       Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
       int num = example->num();
       info.GetReturnValue().Set(v8::Number::New(isolate, num));
   }

   // example.num = 99 时会触发该的 c++ 函数钩子
   static void SetNumCallback(v8::Local<v8::String> property, v8::Local<v8::Value> value, const PropertyCallbackInfo<void>& info) {
       if (value->IsInt32()) {
           Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
       	example->set_num(value->ToInt32())
       }
   }
   ```

4. 根据函数模板创建 example 对象，关联对应 c++ 对象，动态注册方法钩子

   ```c++
   // 在函数模板中获取可调用的函数
   v8::Local<v8::Function> example_constructor = example_tpl->GetFunction(context).ToLocalChecked();
   // 调用函数的创建对象的方法，创建 JS 引擎的 example 对象
   v8::Local<v8::Object> example =
       example_constructor->NewInstance(context, 0, nullptr).ToLocalChecked();
   // 关联 JS 引擎对象和 c++ 对象
   handle->SetAlignedPointerInInternalField(0, this);
   // 动态注册函数钩子
   v8::Local<v8::Function> dynamic_test_func = v8::FunctionTemplate::New(TestDynamicCallback, args.Data())->GetFunction();
   v8::Local<v8::String> dynamic_test_name = v8::String::NewFromUtf8(isolate, "testDynamic", v8::NewStringType::kNormal).ToLocalChecked();
   dynamic_test_func->SetName(dynamic_test_name);
   example->Set(dynamic_test_name, dynamic_test_func);
   ```

   用于于动态注册的 testDynamic 的函数钩子

   ```c++
   // example.testDynamic() 调用时对应的 c++ 函数钩子
   static void TestDynamicCallback(const v8::FunctionCallbackInfo <v8::Value> &args) {
       Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
       example->TestDynamic();
   }
   ```

5. 将 example 对象作为变量添加到 global 的属性中

   ```c++
   v8::Local<v8::String> example_v8_str = v8::String::NewFromUtf8(isolate, "example", v8::NewStringType::kNormal).ToLocalChecked()
   global->Set(context, example_v8_str, example).FromJust();
   ```
6. 如何销毁虚拟机
对于普通的销毁步骤来说，v8引擎对于虚拟机的销毁分为销毁 Context 和销毁 Isolate ，一般v8的 context 会使用 v8::Persistent\<v8::Context>  持有，在调用 v8::Persistent 的 Reset 方法之后，当前 context 中使用扩展方式注册的对象可能不会被完全回收，因此需要自己手动进行回收


**JSC 初始化和 Extension 方式**

example_jsc.cc 文件，以下为 JSC Extension 示例工程部分代码，完整代码请看[附录](#附录)。整体流程总结如下：

1. 初始化 JSC 的环境

   ```c++
   JSContextGroupRef context_group = JSContextGroupCreate();
   ```

2. 创建 global 类定义，据此创建 global 类，根据 global 类创建 JS 运行上下文 context，从 context 中获取 global 对象

   ```c++
   JSClassDefinition global_class_definition = kJSClassDefinitionEmpty;
   JSClassRef global_class = JSClassCreate(&global_definition);
   JSContextRef context = JSGlobalContextCreateInGroup(context_group, global_class);
   JSObjectRef global = JSContextGetGlobalObject(context);
   ```

3. 创建 example 类定义，向类定义设置静态方法和变量的钩子

   ```c++
   // 定义将要 Extension 的静态方法，其中包含函数钩子
   static JSStaticFunction s_examplle_function_[] = {
       {"testStatic", TestStaticCallback, kJSClassAttributeNone},
       { 0, 0, 0 }
   };
   // 定义将要 Extension 的变量，其中包含 get 和 set 函数钩子
   static JSStaticValue s_example_values_[] = {
       {"num", GetNumCallback, SetNumCallback, kJSPropertyAttributeReadOnly | kJSPropertyAttributeDontDelete },
       { 0, 0, 0, 0 }
   };
   // 创建 example 类的定义
   JSClassDefinition example_class_definition = kJSClassDefinitionEmpty;
   // 设置类的对应函数名和参数名的钩子
   example_class_definition.staticFunctions = s_console_function_;
   example_class_definition.staticValues = s_console_values_;
   // 设置类的名称
   example_class_definition.className = "Example";
   ```

   用于静态注册的函数钩子，包括 testStatic 方法钩子和 num 的 get / set 钩子

   ```c++
   // example.test() 调用时对应的 c++ 函数钩子
   static JSValueRef TestStaticCallback(JSContextRef ctx, JSObjectRef function, JSObjectRef thisObject, size_t argumentCount, const JSValueRef arguments[], JSValueRef* exception) {
       // 获取 JS 引擎对象中持有的 c++ 对象
       Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
       example->TestStatic();
   }

   // console.log(example.num) 调用时对应触发的 c++ 钩子函数
   static JSValueRef GetNumCallback(JSContextRef ctx, JSObjectRef object, JSStringRef propertyName, JSValueRef* exception) {
       Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
       int num = obj->num();
       return JSValueMakeNumber(ctx, num);
   }

   // example.num = 99 时会触发该的 c++ 函数钩子
   static bool SetNumCallback(JSContextRef ctx, JSObjectRef object, JSStringRef propertyName, JSValueRef value, JSValueRef* exception) {
       Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
       obj->set_num(JSValueToNumber(ctx, value, NULL));
   }
   ```

4. 根据类定义创建类，根据类创建 example 对象，关联对应 c++ 对象，动态注册方法钩子

   ```c++
   // 创建 JS 引擎的类
   JSClassRef example_class_ref = JSClassCreate(&example_class_definition);
   JSObjectRef example = JSObjectMake(context, example_class_ref, NULL);
   // 关联 c++ 对象和 JS 引擎对象
   JSObjectSetPrivate(context, example, this)
   JSClassRelease(example_class_ref);
   // 动态注册函数钩子
   JSStringRef dynamic_test_func_name = JSStringCreateWithUTF8CString("testDynamic");
   JSObjectRef dynamic_test_func = JSObjectMakeFunctionWithCallback(context, dynamic_test_func_name, TestDynamicCallback);
   JSObjectSetProperty(context, example, dynamic_test_func_name, dynamic_test_func, kJSPropertyAttributeDontDelete, NULL);
   JSStringRelease(dynamic_test_func_name);
   ```

   用于于动态注册的 testDynamic 的函数钩子

   ```c++
   // example.testDynamic() 调用时对应的 c++ 函数钩子
   static JSValueRef TestDynamicCallback(JSContextRef ctx, JSObjectRef function, JSObjectRef thisObject, size_t argumentCount, const JSValueRef arguments[], JSValueRef* exception) {
       Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
       example->TestDynamic();
   }
   ```

5. 将 exmaple 对象作为变量添加到 global 的属性中

   ```c++
   JSStringRef example_str_ref = JSStringCreateWithUTF8CString("example");
   JSObjectSetProperty(context, global, example_str_ref, example, kJSPropertyAttributeDontDelete, NULL);
   JSStringRelease(example_str_ref);
   ```

6. 如何销毁虚拟机
  JSC引擎中对于虚拟机的销毁相对比较简单，只需要通过调用 JSGlobalContextRelease 和 JSContextGroupRelease 来分别对 Context 和 Context Group 即可，内存中使用扩展方式注册的对象都会在销毁过程中调用 Finalize 回调

  ​

**关于线程安全**

JS的 context 并非是线程安全的，因此一个 context 不能在多个线程之间共享，以避免遇见未知错误。对于应用来说可能需要使用多个 JS Context，而使用多个 Context 的方式有两种，独占式和共享式。

>独占式，所谓独占式是指一个 Context 使用一个线程进行处理
>共享式，所谓共享式是指多个 Context 使用一个线程进行处理，以v8为例，在切换 Context 的时候需要设置v8::Isolate::Scope 和 v8::Context::Scope ,可以仿造下面示例通过宏定义来处理
```c++
#define MakeCurrent(_isolate, _context, _is_runtime_available) \
if (!_is_runtime_available \
|| (_isolate == NULL && v8::Isolate::GetCurrent() == NULL) \
|| context.IsEmpty()) { \
return false; \
} \
v8::Isolate* isolate = _isolate == NULL ? v8::Isolate::GetCurrent() : _isolate; \
v8::Isolate::Scope isolate_scope(isolate); \
v8::HandleScope handleScope(isolate); \
v8::Context::Scope context_scope(v8::Local<v8::Context>::New(isolate, _context));
```



**小结**

在上述的几个示例中，可以看出在创建对象和注册静态方法和变量上，V8 和 JSC 具有各自的 API 和变量命名特点，方法中的参数类型也是完全不一致的，V8 和 JSC 在这一层面上具有极大的差异。这也导致了前述在替换 Lynx 的 JS 引擎时所耗费的成本。

然而两个引擎的整体流程是一致的，同时 API 和参数类型在概念上也是一致的，其原因是 V8 和 JSC 都遵循 JS 的规范。相同点例如：

- JSC 和 V8 基本一致的初始化和创建对象流程（同等概念的构造函数，代码中没展示）
- JSC 中表示的字符串 JSStringRef 和 V8 中同一概念的 v8::Local\<v8::String>
- JSC 中用于设置对象属性的 JSObjectSetProperty 方法，对等于 V8 中的 v8::Local\<v8::object>->Set 方法

> 当对两个引擎的 API 具有一定熟练度，同时对于 JS 已经有一定的掌握，对于两个引擎在使用上的异同，能有更好的理解。



#### JS Binding 中对象生命周期控制

JS 引擎具有内存管理机制，在建立 JS Binding 时，不可避免的要建立本地对象（c++ 对象或者平台对象）与 JS 引擎对象的关系，恰当的对象关系能保证程序在内存上的稳定，这需要使用到 JS 引擎的内存管理相关知识以及其提供可以控制对象在 GC 机制上的行为的接口。

一切由 JS 引擎提供或创建的对象的生命周期管理需要由其内部的 GC 机制把控，JS 引擎提供了两个接口管理 JS 对象在 GC 机制上的行为，一个是持久化操作帮助 JS 对象脱离 GC 管理，一个是使 JS 对象回归 GC 管理。JS 对象在其生命周期内所出现的行为均可以监听（加钩子），例如 JS 对象的初始化和析构监听。V8 引擎和 JSC 引擎涉及的知识和接口均不相同，但是在概念上是一致的，下面看一下两个平台上的区别。

**V8 引擎监听和管理对象生命周期的方法**

V8 引擎在使用上会经常出现 Handle（句柄）的用法，这是引擎对于其对象访问和管理的方式，Handle 提供在堆上的 JS 对象位置的引用。当一个 JS 对象在 JS 上已经无法访问，同时没有任何的句柄指向他，则被认为可被 GC 回收，这是引擎提供句柄的意义。

> 在 V8 引擎中，在 GC 运行阶段，垃圾收集器会经常移动堆上的对象，同时会更新句柄内容使其指向 JS 对象在堆上更新后的位置。

Local handle（局部句柄）和 Persistent handle（持久句柄）是经常使用到的其中两种句柄类型。

Local handle 被存放在栈上面，它的生命周期仅存在于一个句柄域中（handle scope），当程序跳出函数块，句柄域析构，局部句柄也随之被释放，指向 JS 对象的句柄数量随之减少。

> Handle scope 好比一个容器（栈），当初始化之后，它会收集在这期间创建的局部句柄，当被析构之后，所有的局部句柄将被移除，触发局部句柄的析构。

Persistent handle（持久句柄）和局部句柄一样提供了一个引用指向堆上分配的 JS 对象，但对于其引用的生命周期管理与局部句柄不一样。持久句柄存放在全局区，因此生命周期不受局部区域块的影响，能够在其的生命周期内在多个函数中多次使用，既 JS 对象脱离 GC 管理。持久句柄可以通过 `Persistent::New`方法由局部句柄生成，也可以通过 `Local::New` 方法生成局部句柄。可以通过 `Persistent::SetWeak` 方法进行弱持久化，同时也可以通过 `Persistent::Reset` 方法去持久化。

> 弱持久化，设置了弱持久化则 Persistent handle 的 JS 对象会当仅剩下该弱持久句柄指向 JS 对象，GC 收集器将会回收并触发被设置的监听函数。
>
> 去持久化，释放持久句柄对于堆上的对象的引用

Local handle 和 Persistent handle 的转化方式

```c++
void TestHandle() {
    v8::Isolate* isolate = v8::Isolate::GetCurrent();
    // 下面的代码会创建局部句柄，需要使用一个 handle scope 来管理
    HandleScope handle_scope(isolate);
    // 创建一个 JS Array 对象，返回的是一个局部句柄
    v8::Local<v8::Array> local_array = v8::Array::New(isolate, 3);
    // 将局部句柄转为持久句柄
    v8::Persistent<v8::Array> persistent_array = v8::Persistent<v8::Array>::New(isolate, local_array);
    // 将持久句柄转为局部句柄
    local_array = v8::Local<v8::Object>::New(persistent_array);
    // 将持久句柄去持久化
    persistent_array.Reset();
    // 当函数块结束后，局部句柄将被析构， JS Array 对象也在未来某个事件被 GC
}
```

将持久句柄进行弱持久化的方式如下，在弱持久化的 API 中提供了 JS 对象被 GC 时的监听回调的设置。

```c++
static void WeakCallback(const v8::WeakCallbackInfo<ObjectWrap>& data) {
    v8::Isolate *isolate = data.GetIsolate();
    v8::HandleScope scope(isolate);
    // data.GetParameter() 是上述 MakeWeak 时传进来的参数 this
}

void MakeWake() {
    persistent_array_.SetWeak(this, WeakCallback, v8::WeakCallbackType::kParameter);
}
```



**JSC 引擎管理对象 GC 行为的接口**

JSC 引擎上常用于和 JS 打交道的对象包括 JSContextGroupRef / JSContextRef / JSValueRef / JSObjectRef / JSStringRef / JSClassRef 等，对于这些对象的内存管理的方式和 V8 引擎上的方式有所不同。在 JSC 引擎上直接接触到的分为包装和指针，而指针部分由引擎 GC 管理，需要开发者手动管理。

JSStringRef、 JSContextRef、JSContextGroupRef、JSClassRef 就是典型的不受管理的指针，当开发者创建了它们之后，必须在不需要使用的时候释放它们的内存，否则会出现内存泄露。请看下面的示例。

```c++
JSContextGroupRef context_group = JSContextGroupCreate();
JSClassRef global_class = JSClassCreate(&kJSClassDefinitionEmpty);
JSContextRef context = JSGlobalContextCreateInGroup(context_group, global_class);
JSStringRef str = JSStringCreateWithUTF8CString("try");
// 手动回收内存
JSClassRelease(golbal_class);
JSStringRelease(str);
JSGlobalContextRelease(context);
JSContextGroupRelease(context_group);
```

JSValueRef 代表着 JS 对象（ JSObjectRef 也是一个 JSValueRef ） 是由 JSC 引擎中的 GC 机制管理的，当 JSValueRef 创建出来后，当需要超出当前函数块在全局区域或者堆上继续操作，则需要通过 `JSValueProtect` 对 JSValueRef 进行持久化的操作，JS 对象将脱离 GC 的管理。`JSValueUnprotect` 是 JSC 引擎提供的对于 JSValueRef 去持久化的 API。持久化和去持久化必须成对出现，否则会出现内存泄露。

```c++
// 假设已经初始化上下文
JSContextRef context; 
JSObjectRef object = JSObjectMake(context, NULL, NULL);
// 持久化操作
JSValueProtect(context, object);
// 去持久化操作
JSValueUnprotect(context, object);
```

JSC 中没有弱持久化的概念，通过类定义创建出来的对象都可以监听其被 GC 的事件，这一点和 V8 不同。对于普通或是去持久化的 JS 对象， 监听其析构函数的方式是在创建 JS 对象的类定义时候需要加入析构函数的钩子。

```c++
static void FinalizeCallback(JSObjectRef object) {
    // do sth
}

void CreateObject(JSContextRef context) {
    JSClassDefinition definition = kJSClassDefinitionEmpty;
	definition.finalize = FinalizeCallback;
    JSClassRef class_ref = JSClassMake(definition);
    JSObjectRef object = JSObjectMake(context, class_ref, NULL);
    JSClassRelease(class_ref);
}
```

了解了基本的 JS 引擎初始化、 Extension 和内存管理知识，对于后续 Lynx 上各个 JS Binding  的作用以及框架设计会有更好的理解。



#### Lynx 中 JS Binding 前期版本

Lynx 是一个由前端驱动的框架，需要建立本地对象和 JS 对象的关系，本地对象跟随 JS 对象的生命周期。

Lynx 早期的快速实现首先在 Android 平台上进行，依赖 V8 引擎，利用 Java 搭建了核心链路。由于 Android 并没有直接和 V8 沟通的接口，通过引入 V8 的动态 so，Lynx 结合 JNI 实现 JS Binding，建立和 V8 的桥梁。整体设计如下图。JS Binding 是作为 Android 和 JS 的通信使者，JS Binding 包括了 JS 引擎初始化模块 JSContext 和一系列拓展模块，如 JS 上下文中 document、element 对象等。

<img src="./binding1.png" width="450"/>

在前期版本中只有简单的设计，主要目的是满足功能。这里主要介绍基本类及其作用和整体流程。

**基础类介绍**

关键基础类 UML 图

<img src="./binding2.png" width="550"/>

JSContext 用于初始化 V8 环境，向 V8 中注册基础对象，搭建中间层 JS Binding。通过 JS Binding 可以实现 Android 层面的对象和 V8 的间接调用。JS Binding 中包括 DocumentObject、WindowObject、ElementObject 等。

ElementObject 属于 JS Binding 中的成员，作为两端 Element 通信的使者。其父类是 JavaObjectWrap。用于向 V8 中拓展 element 对象及注册相关函数和变量钩子，在 JS 上下文提供基础 API。同时初始化 JNI 函数，准备和 Java Element 通信基础。与 ElementObject 作用类似的还有 DocumentObject、WindowObject 等。

JavaObjectWrap 作为通信使者的基类，通过 V8 引擎和 JNI 提供的持久化接口是建立 Android 端对象和 V8 端对象的关系，使 Android 对象、c++对象和 V8 对象具有一致的生命周期。在 JavaObjectWrap 中持有 Java 持久化对象和 JS 持久化对象。继承 Observer 类。

Observer 作用是确保页面退出时，在 JSBinding 上没有泄露的内存。在页面退出时，防止在页面退出后因为 JS 引擎的内存管理影响，导致 c++ 和 Android 的内存泄露，清理在 JS Binding 中的未被 GC 的对象。

**整体流程**

JS Binding 担任了两个不同平台间交流的使者，是整个 Lynx 框架的基石。JS Binding 整体流程主要包括 JS Binding 初始化流程、JS 调用拓展 API 流程和 JS Binding 结束流程。

在页面启动时，JS Binding 会进行初始化操作，JS 上下文中就能具备了使用 Lynx 框架能力的功能，初始化流程如下

1. Android 层通过 JNI 调用 JSContext::Initialize 方法进行引擎环境和上下文初始化。
2. 调用 ElementObject、ConsoleObject 等 c++ 对象的 BindingClass 方法初始化与 JNI 相关的变量，如 jclass 和 jMethodID，以便后续调用 Java 方法；创建对应类的构造器函数模板，向函数模板中注册钩子。
3. 创建 DocumentObject、ConsoleObject 的 c++ 对象，通过 JNI 方法 NewGlobalRef 持久化对应的 Java jobject 对象，并将自身的地址交给 Java 对象持有，建立 c++ 对象和 Java 对象的关系。
4. 通过 DocumentObject、ConsoleObject 的构造器函数模板创建 JS 对象，如 document、console 等，关联 c++ 对象到 JS 对象中。c++ 对象将 JS 对象持久化，注册析构监听。建立 c++ 对象和 JS 对象的关系。
5.  将拓展的 JS 对象注册到 JS 上下文中。

当 JS 中通过 document.createElement('view') 方法时，会简洁调用到 Java 层接口并返回具体结果，具体流程如下

1. V8 引擎在运行到对应方法时，回调 DocumentObject 的 CreateElementCallback 静态方法钩子。
2. 在 CreateElementCallback 方法中，获取 JS 引擎对象中绑定的本地对象 DocumentObject，调用其 CreateElement 方法。
3. 在 DocumentObject::CreateElement 方法中，通过 JNI 接口和已经初始化的 jMethod 和调用 Java 对象的 createElement 方法。
4. 在 Java Document.createElement 方法中，创建 Element 对象并返回其持有的 c++ 指针 ElementObject 地址。
5. 在 DocumentObject::CreateElement 方法中，强转获得 Java Element 对象持有的  ElementObject 指正地址并返回。
6. 在 CreateElementCallback 方法中，返回 ElementObject 中持有的持久化 JS 对象。
7. JS 中获得 document.createElement('view') 调用结果 element。

Lynx 页面退出代表着 JS Binding 的结束，Android 层通过 JNI 释放 JSContext 内存，进而释放 JS 对象、c++ 对象和 Java 对象所占用的内存。触发析构函数将完成以下操作

1. 释放所有持有的构造器函数模板，触发 V8 引擎 GC。
2. JS 对象 GC 触发函数钩子 JavaObjectWrap::WeakCallback，释放持久化 jobject 对象，释放本身 c++ 指针。



#### 总结

这篇文章主要介绍 JS Binding 入门知识和 Lynx 框架在早期版本 JS Binding 的简单实现，为后续的 Lynx 框架的 JS Binding 的演进抛砖引玉。下一篇文章将分析 Lynx 框架中 JS Binding 演进的技术，并介绍其优缺点。

请持续关注 [Lynx](https://github.com/hxxft/lynx-native)，一个高性能跨平台开发框架。



#### 附录

example_v8.cc 初始化流程具体代码

```c++
// example.testStatic() 调用时对应的 c++ 函数钩子
static void TestStaticCallback(const v8::FunctionCallbackInfo <v8::Value> &args) {
    Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
    example->TestStatic();
}

// example.testDynamic() 调用时对应的 c++ 函数钩子
static void TestDynamicCallback(const v8::FunctionCallbackInfo <v8::Value> &args) {
    Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
    example->TestDynamic();
}

// console.log(example.num) 调用时对应触发的 c++ 钩子函数
static void GetNumCallback(v8::Local<v8::String> property, const PropertyCallbackInfo<Value>& info) {
    Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
    int num = example->num();
    info.GetReturnValue().Set(v8::Number::New(isolate, num));
}

// example.num = 99 时会触发该的 c++ 函数钩子
static void SetNumCallback(v8::Local<v8::String> property, v8::Local<v8::Value> value, const PropertyCallbackInfo<void>& info) {
    if (value->IsInt32()) {
        Example* example = static_cast<Example*>(args.Holder()->GetAlignedPointerFromInternalField(0));
    	example->set_num(value->ToInt32())
    }
}

void Example::Initialize() {
    // 初始化 V8 引擎
    V8::InitializeICUDefaultLocation(argv[0]);
    V8::InitializeExternalStartupData(argv[0]);
    v8::Platform* platform = v8::platform::CreateDefaultPlatform();
    V8::InitializePlatform(platform);
    v8::V8::Initialize();
    v8::Isolate* isolate = v8::Isolate::New(create_params);
    v8::Isolate::Scope isolate_scope(isolate);
    v8::HandleScope handle_scope(isolate);
    
	// 创建全局 global 对象模板，根据模板创建 JS 运行上下文，在上下文 context 中获取 global 对象
    v8::Local <v8::ObjectTemplate> global_template = v8::ObjectTemplate::New(isolate);
    v8::Local <v8::Context> context = v8::Context::New(isolate, nullptr, global_template);
    v8::Context::Scope context_scope(context);
    v8::Local<v8::Object> global = context->Global();
    
    // 创建 example 的构造函数模板, 使用该 c++ 类的初始化函数作为参数（函数钩子），初始化构造器函数模
    // 板。即当调用构造函数创建对象时，会调用该钩子函数做构造处理
    v8::Local<v8::FunctionTemplate> example_tpl = v8::FunctionTemplate::New(isolate);
    // 设置构造函数模板的类名
    example_tpl->SetClassName(V8Helper::ConvertToV8String(isolate, "Example"));
    // 设置内部关联 c++ 对象的数量为 1
    example_tpl->InstanceTemplate()->SetInternalFieldCount(1);
    // 设置构造函数模板中的原型模板的对应函数名的钩子
    example_tpl->PrototypeTemplate()->Set(V8Helper::ConvertToV8String(isolate, "testStatic"),
        v8::FunctionTemplate::New(isolate, TestStaticCallback));
    // 设置构造函数模板中的原型模板的属性的 Get 和 Set 钩子方法
    example_tpl->PrototypeTemplate()->SetAccessor(V8Helper::ConvertToV8String(isolate, "num"), GetNumCallback, SetNumCallback);
    // 在函数模板中获取可调用的函数
    v8::Local<v8::Function> example_constructor = example_tpl->GetFunction(context).ToLocalChecked();
    // 调用函数的创建对象的方法，创建 JS 引擎的 example 对象
    v8::Local<v8::Object> example =
        example_constructor->NewInstance(context, 0, nullptr).ToLocalChecked();
    // 关联 JS 引擎对象和 c++ 对象
    handle->SetAlignedPointerInInternalField(0, this);
    // 动态注册函数钩子
    v8::Local<v8::Function> dynamic_test_func = v8::FunctionTemplate::New(TestDynamicCallback, args.Data())->GetFunction();
    v8::Local<v8::String> dynamic_test_name = v8::String::NewFromUtf8(isolate, "testDynamic", v8::NewStringType::kNormal).ToLocalChecked();
    dynamic_test_func->SetName(dynamic_test_name);
    example->Set(dynamic_test_name, dynamic_test_func);
    // 向 global 对象中设置 example 属性
    v8::Local<v8::String> example_v8_str = v8::String::NewFromUtf8(isolate, "example", v8::NewStringType::kNormal).ToLocalChecked()
    global->Set(context, example_v8_str, example).FromJust();
}
```

初始化流程具体代码

```c++
// example.test() 调用时对应的 c++ 函数钩子
static JSValueRef TestStaticCallback(JSContextRef ctx, JSObjectRef function, JSObjectRef thisObject, size_t argumentCount, const JSValueRef arguments[], JSValueRef* exception) {
    // 获取 JS 引擎对象中持有的 c++ 对象
    Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
    example->TestStatic();
}

// example.testDynamic() 调用时对应的 c++ 函数钩子
static JSValueRef TestDynamicCallback(JSContextRef ctx, JSObjectRef function, JSObjectRef thisObject, size_t argumentCount, const JSValueRef arguments[], JSValueRef* exception) {
    Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
    example->TestDynamic();
}

// console.log(example.num) 调用时对应触发的 c++ 钩子函数
static JSValueRef GetNumCallback(JSContextRef ctx, JSObjectRef object, JSStringRef propertyName, JSValueRef* exception) {
    Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
    int num = obj->num();
    return JSValueMakeNumber(ctx, num);
}

// example.num = 99 时会触发该的 c++ 函数钩子
static bool SetNumCallback(JSContextRef ctx, JSObjectRef object, JSStringRef propertyName, JSValueRef value, JSValueRef* exception) {
    Example* example = static_cast<Example*>(JSObjectGetPrivate(object));
    obj->set_num(JSValueToNumber(ctx, value, NULL));
}

// 定义将要 Extension 的静态方法，其中包含函数钩子
static JSStaticFunction s_examplle_function_[] = {
    {"testStatic", TestStaticCallback, kJSClassAttributeNone},
    { 0, 0, 0 }
};

// 定义将要 Extension 的变量，其中包含 get 和 set 函数钩子
static JSStaticValue s_example_values_[] = {
    {"num", GetNumCallback, SetNumCallback, kJSPropertyAttributeReadOnly | kJSPropertyAttributeDontDelete },
    { 0, 0, 0, 0 }
};

void Example::Initialize(JSVM* vm, Runtime* runtime) {
    // 初始化 JSC 引擎
    JSContextGroupRef context_group = JSContextGroupCreate();
    
	// 创建全局 global 类的定义
    JSClassDefinition global_class_definition = kJSClassDefinitionEmpty;
    // 创建 global 对象的类
    JSClassRef global_class = JSClassCreate(&global_definition);
    // 根据 global 类创建上下文，从上下文获取 global 对象
    JSContextRef context = JSGlobalContextCreateInGroup(context_group, global_class);
    JSObjectRef global = JSContextGetGlobalObject(context);
    
    // 创建 example 类的定义
    JSClassDefinition example_class_definition = kJSClassDefinitionEmpty;
    // 设置类的对应函数名和参数名的钩子
    example_class_definition.staticFunctions = s_console_function_;
    example_class_definition.staticValues = s_console_values_;
    // 设置类的名称
    example_class_definition.className = "Example";
    // 创建 JS 引擎的类
    JSClassRef example_class_ref = JSClassCreate(&example_class_definition);
    JSObjectRef example = JSObjectMake(context, example_class_ref, NULL);
    // 关联 c++ 对象和 JS 引擎对象
    JSObjectSetPrivate(context, example, this)
    JSClassRelease(example_class_ref);
    // 动态注册函数钩子
    JSStringRef dynamic_test_func_name = JSStringCreateWithUTF8CString("testDynamic");
    JSObjectRef dynamic_test_func = JSObjectMakeFunctionWithCallback(context, dynamic_test_func_name, TestDynamicCallback);
    JSObjectSetProperty(context, example, dynamic_test_func_name, dynamic_test_func, kJSPropertyAttributeDontDelete, NULL);
    JSStringRelease(dynamic_test_func_name);
    // 向 global 对象中设置 example 属性
    JSStringRef example_str_ref = JSStringCreateWithUTF8CString("example");
    JSObjectSetProperty(context, global, example_str_ref, example, kJSPropertyAttributeDontDelete, NULL);
    JSStringRelease(example_str_ref);
}
```

