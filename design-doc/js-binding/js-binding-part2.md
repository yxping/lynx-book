### JS Binding 技术

[Lynx（一个高效的跨平台框架）](https://github.com/hxxft/lynx-native/) 的 JS Binding 技术最主要的目的是搭建一个高效的与 JS 引擎解耦的通信桥梁，同时具备 JS 引擎切换的能力。该技术经历了多次迭代，最终通过抽象的引擎接口层设计，在代码层面做到对于 JS 引擎的解耦。目前 Lynx 在 Android 端支持 V8 和 JSC 引擎的切换。



#### 遇到的问题

Lynx 是一个 JS 驱动的跨平台框架，提供了 JS 调用 Android 和 iOS 等平台层的渲染能力，同时允许开发者拓展平台能力，因此在 Lynx 中和 JS 通信的除了核心 Runtime 层，还包括了处于平台层的 Module 和 RenderObjectImpl，同时在框架中存在线程间通信的情况。结合上述 Lynx 框架的特性，在 JS Binding 迭代时遇到的主要的问题：

1. 代码解耦：对于不同的 JS 引擎的初始化等流程和 Extension （需要定义静态方法）方式的统一
2. C++ 对象生命周期管理
3. 跨线程和跨平台的参数转化



#### 设计

> 整体设计代码在 [runtime 目录](https://github.com/hxxft/lynx-native/tree/master/Core/runtime)。

对于**跨线程和跨平台的参数转化**的问题，为了便于参数在上下游的转化（JS 与核心 C++ 层的转化，核心层与平台层 Android & iOS 的转化），定义了 LynxValue 作为通用传递参数，并根据不同平台制定 LynxValue 的转化规则，减少参数在跨层调用时繁琐的转化步骤。转化规则现在包括 JSC 到核心层的 [JSCHelper](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/jsc/jsc_helper.h)，V8 到核心层的 [V8Helper](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/v8/v8_helper.h)，核心层到 iOS 层的 [OCHelper](https://github.com/hxxft/lynx-native/blob/master/Core/base/ios/oc_helper.h)，以及核心层到 Android 层的 [JNIHelper](https://github.com/hxxft/lynx-native/blob/master/Core/base/android/jni_helper.h)。下面的图可以看出 LynxValue 流通与不同层次。

<img src="./jsbinding1.png" width="450"></img>



**主要数据结构**

- LynxValue 是参数传递规则的基类，其中使用了联合体定义了支持转化的参数。包括基本数据类型，数组，键值对，LynxObject 和 LynxFunction 等。除了 LynxFunxtion 和 LynxObject，其余参数均不能直接和 JS 通信，仅用于参数转化，同时支持跨线程跨平台传递。
- LynxArray 有序的有限个的 LynxValue 的集合，对应 JS 端和平台层的数组。
- LynxMap 键值对，仅支持字符串作为 key，对应 JS 端的 Map 或者 Object 和平台层的键值对。
- LynxFunction 存储了 JS 端的 function，用于在合适时机回调 JS function。
- LynxObject 通信基类，借助 ClassTemplate 构建与 JS 通信桥梁的对象（请看后续分析），可以对 JS 对象进行间接的操作，如 ProtectJSObject 的操作，使 JS 对象脱离 GC。

**在 JS 引擎代码解耦方面**，JSC 和 V8 在 JS 原型和 Extension 上的设计都是相似的逻辑，只是在实现的细节上不一致。如在 JSC 中利用 JSClassRef 描述原型上所具有的属性和方法，同时可以构造原型链，而 V8 中利用 FunctionTemplate 和 PrototypeTemplate 代替；JSC 中使用 JSObjectSetPrivate 接口为 JS 对象绑定一个 C++ 对象，而 V8 则利用 ObjectTemplate::SetInternalField 方法代替。基于上述特点，Lynx 的 JSBinding 抽象了一层 JS 的原型构造器和方法钩子的接口，以满足与 JS 的通信功能。

<img src="./jsbinding2.png" width="450"></img>

[JSVM](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/js/js_vm.h) 是代表 JS 运行的虚拟机，真正的实现文件交由各自引擎实现。

[JSContext](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/js/js_context.h) 为 JS 引擎的控制上下文，同时是一个模板类，其中包含全局对象 Global，对于真正的 V8 和 JSC 的操作由其实现类 [V8Context](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/v8/v8_context.h) 和 [JSCContext](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/jsc/jsc_context.h) 决定。而与 JS 通信主要使用对外接口 ClassTemplate 和内部接口 ObjectWrap。

[ClassTemplate](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/js/class_template.h) 用于构造 JS 原型的模板，通过该模板可以注册函数和变量钩子等（Extension 功能。该对象持有PrototypeBuilder，PrototypeBuilder 由对应的 JS 引擎实现，用于构建 JSC 的 JSClassRef 或者是 V8 的 FunctionTemplate，同时可以根据原型创建 JS 对象。 ClassTemplate 提供了宏定义帮助定义默认 ClassTemplate 的静态方法，下面是宏定义的意义和用法：

- `DEFINE_CLASS_TEMPLATE_START` 默认 ClassTemplate 构建的方法定义的开始
- `REGISTER_PARENT` 定义 ClassTemplate 的父亲（原型链），在 START 和 END 之间使用。
- `EXPOSE_CONSTRUCTOR` 在 JS 上下文中暴露该 ClassTemplate 作为构造器，在 START 和 END 之间使用。
- `REGISTER_METHOD_CALLBACK` 向 ClassTemplate 中注册函数钩子，在 START 和 END 之间使用。
- `REGISTER_GET_CALLBACK` `REGISTER_SET_CALLBACK` 向 ClassTemplate 中注册变量钩子，在 START 和 END 之间使用。
- `DEFINE_CLASS_TEMPLATE_END` 默认 ClassTemplate 构建的方法定义的结束。
- `DEFAULT_CLASS_TEMPLATE` 获取默认 ClassTemplate。

[defines.h](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/js/defines.h) 头文件含有用于定义 JS 引擎钩子函数的宏规则，C++ 类需要根据宏定义钩子函数，并将函数指正注册到 ClassTemplate 中，同时自身需要有对应的类方法（钩子函数会进行回调）进行真正实现，才能完成原型的构建。ClassTemplate.h 中通过宏定义提供了快速构建一个与 C++ 对象默认的 ClassTemplate 对象。结合两个宏定义规则，可以实现快速构建与 JS 通信的 C++ 类。下面是 defines.h 中宏定义的意义：

- `DEFINE_METHOD_CALLBACK` 用于定义 JS 引擎函数钩子，`DEFINE_GROUP_METHOD_CALLBACK` 用于定义带方法名称作为参数的函数钩子，`METHOD_CALLBACK` 用于获取钩子名称
- `DEFINE_SET_CALLBACK` `DEFINE_GET_CALLBACK` 用于定义 JS 引擎变量钩子，`SET_CALLBACK` `GET_CALLBACK` 用于获取钩子名称。

> 自定义类方法钩子示例：
>
> JS 变量的 Get 钩子：base::ScopedPtr\<LynxValue> Function();
>
> JS 变量的 Set 钩子： void Function(base::ScopedPtr\<jscore::LynxValue>& value);
>
> JS 方法钩子：base::ScopedPtr\<LynxValue> Function(base::ScopedPtr\<LynxArray>& array);

[ObjectWrap](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/js/object_wrap.h) 用于建立 JS 对象和 C++ 对象（这里指 LynxObject）的关系，即用于**管理 C++ 对象生命周期**，C++ 对象的生命周期是跟随 JS 对象（当然 JS 对象只是对 C++ 对象进行引用计数上的增减，确保 C++ 对象在被其他类引用时可以被安全释放或使用）。JS对象和C++对象绑定的时机在 ClassTemplate 创建 JS 对象时，这个时机由 JS 运行上下文决定（在 defines.h 中的钩子函数中处理），无需开发者关心。

**JS Binding 整体运行图示**，在 Lynx 开发中，JS 引擎的具体实现或者参数转化规则对外无感知，利用 LynxObject 和 LynxValue 就可以与 JS 通信，完成 API 调用工作。LynxValue 和 JSValue 的转化均是在 JSObject 和 LynxObject 相互调用时进行。

<img src="./jsbinding3.png" width="450"></img>

**实例**：定义与 JS 对象 console 关联的 [Console 类](https://github.com/hxxft/lynx-native/blob/master/Core/runtime/console.cc)，实现 console.log 的函数调用，主要步骤如下

1. 继承 LynxObject ，定义被钩子函数调用的 Log 类方法
2. 定义需要进行 Extension 的 Log 函数钩子
3. 根据 ClassTemplate 提供的宏定义，快速创建默认的 ClassTemplate，在构造函数中传入默认的 ClassTemplate。

```c++
namespace jscore {
    class Console : public LynxObject {
    public:
        Console(JSContext* context);
        virtual ~Console();
		// 定义 JS 引擎函数钩子回调的类方法
        base::ScopedPtr<LynxValue> Log(base::ScopedPtr<LynxArray>& array);
    };
}
```

```c++
namespace jscore {

    #define FOR_EACH_METHOD_BINDING(V)    \
        V(Console, Log)                   

    // 定义需要进行 Extension 的函数钩子
    FOR_EACH_METHOD_BINDING(DEFINE_METHOD_CALLBACK)

    // 定义默认的 ClassTemplate
    DEFINE_CLASS_TEMPLATE_START(Console)
        FOR_EACH_METHOD_BINDING(REGISTER_METHOD_CALLBACK)
    DEFINE_CLASS_TEMPLATE_END
    
	// 构造函数中传入默认的 ClassTemplate
    Console::Console(JSContext* context) : LynxObject(context, DEFAULT_CLASS_TEMPLATE(context)) {
    }

    Console::~Console() {}

    base::ScopedPtr<LynxValue> Console::Log(base::ScopedPtr<LynxArray>& array) {
    	// Print log
        return base::ScopedPtr<LynxValue>(NULL);
    }

}
```



#### 优缺点

优点：隔离 JS 引擎代码，易于切换；无额外消耗的函数钩子（通信）实现，比 RN 的通信更快；快速上手，相比于 Web IDL 没有学习成本。

缺点：仍然需要在通信类中手动编写一定代码；暂时只满足于和 JS 引擎通信的功能，相比于 Web IDL 而言功能相对简单，暂时无涉及多种外部语言。



#### 尝试

Git 拉 [Lynx 工程](https://github.com/hxxft/lynx-native)源码，根据 [How To Build](https://github.com/hxxft/lynx-native#how-to-build) 运行 Android 工程，在 Android 工程根目录的 gradle.properties 中，通过设置 `js_engine_type=v8/jsc` 进行 V8 引擎和 JSC 引擎的切换。iOS 仅支持 JSC 引擎。
