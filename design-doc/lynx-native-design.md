# Lynx架构

## Glossary

**Runtime**: JS层是所有JS业务和显示相关的逻辑都在Runtime中执行，Runtime是JS逻辑与各平台原生代码交互的基础。Runtime实现了基础的JSBridge，并通过JSBridge为JS添加了一些基础类型和对象，例如Navigator，Log，Screen，Document，Element，Body等，具体内容可以参看[API文档](../README.md)


**Render**：Render层是实现显示相关的逻辑，Runtime将显示相关的逻辑。Render层会根据Runtime层调用element的操作节点的API后生成的RenderTree，进行排版。Render层包含一个排版引擎，支持使用flex排版进行排版。Render层与Runtime运行在同一个线程(JS线程)，Render层会根据排版结果与Platform层跨线程通行，将排版结果和一些显示属性传递给Platform层。

**Platform**： Platform层是各自iOS和Android平台的实现层，Platform层会根据Render层中RenderTree生成各自平台的ViewTree并用于真正在平台上的显示。Platform层会接受Render层传递过来的排版和显示属性，并转换成平台View的属性进行设置。同时Platorm层也会接受系统事件，例如点击，滚动等等事件，并将事件传递给Render层，再通过Render层传递给Runtime并且触发需要响应的逻辑。

**LayoutFile**： LayoutFile是在编译过程中生成的首屏排版文件，Render层会根据排版文件生成RenderTree，LayoutFile类似于Android中用于排版的XML文件。

**AnimateScript**： AnimateScript是用于执行交互动画的一个可执行文件，AnimateScript是JS的一个子集(请参看[lepus语法](./lepus.md))，负责及时响应系统事件并产生相应的动画。


## 详细架构图

![image](./images/lynx-native-detail.png)