#### 概述

交互动画模块使 Lynx 提供和 Native 一样优秀的跟手动画性能和效果。它是基于 Lepus 脚本语言描述动画操作过程，完成动画操作从 JS 线程到 UI 线程的转移，在 UI 线程中通过协作者方式完成整个动画过程，包括事件的分发响应，确保能在一帧内完成相应的动画（二维动画）。

> Lepus 脚本语言是  JS 语言子集，具体见 Lepus 引擎。



交互动画使用步骤：

1. 在 JS 端使用属性描述 Element 之间的协作关系
2. 编写 Lepus 脚本，定制动画样式
3. 使用指定 API 注册脚本




#### 名词描述

*sponsor* : 分发者，提供分发事件功能。如分发 touch 事件的 View， 分发 scroll 事件的 ScrollView。

*responder* : 响应者，响应来自分发者的事件而进行相应动画。任何一个 View 都可以成为响应者。

*action* : 一个 Lepus 脚本，描述响应者和分发者在事件产生时的相应处理动作。

*affinity* : 亲和性，用于描述 sponsor 和 responder 的关系。

*Element* 和 *View* : Element 指前端的 dom 对象，View 指 dom 对象对应的 Native View。



#### 协作者关系

协作者关系，在 Native 层面上指分发事件的 View 与监听事件并响应的 View 的一个关系链。描述关系链的方法是在 JS 端通过设置 Element 的协作属性，结合交互动画提供的注册 API 完成。一个正确的关系链可以触发正确高效的动画。

在 JS 端定义协作者关系的方法如下

1. Element 关于描述协作者关系的属性列表，这些都是通过 *Element.setAttribute* 方法设置：   

| 属性                 | 作用                                                         | 备注                                                         |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| coordinator-affinity | 亲和性，通过该属性构造关系链                                 | 分发者和响应者均具备                                         |
| coordinator-type     | 分发事件的类型，如 *scroll*                                  | 分发者特有，如果需要支持多个分发事件类型可以使用 *或运算符* 间隔，如： *scroll 或运算符 touch* |
| coordinator-command  | 制定响应事件的指令，用法 *type:FuncName*，如：*scroll:onScrollEvent* | 响应者特有，如果需要支持多个指令可以使用 *或运算符* 间隔，如：*scroll:onScrollEvent 或运算符 touch:onTouchEvent* |
| coordinator-tag      | *Element* 对应的标识，在脚本中用于区别 *Element* 的标识      | 分发者和响应者均具备，如果该属性值改变了会触发脚本中的 *init* 操作 |

2. 注册 API，建立分发者和响应者的链接:

``` js
CoordinatorRegister.registerAction(String sponserAffinity, String reponderAffinity, String action);
```
*registerAction* 向 Native 框架中注册 action，建立 responderAffinity 的 sponsor 和 sponsorAffinity 的 responder 的关联，关联指当事件产生了，sponsor 仅会对关联的 responders 发送事件。sponserAffinity 和 responderAffinity 决定了 action 的唯一性，后续的方法也是通过这两个属性去对 action 进行操作。

> 1. 一个 sponsor 可以关联多个 responder    
> 2. 一个 sponsorAffinity 可以绑定多个 responderAffinity
> 3. sponsor 的 affinity 需要全局唯一，否则会被覆盖。

3. 其他 API，用于更新参数或断开链接：

``` js
CoordinatorRegister.removeAction(String sponserAffinity, String reponderAffinity, String action);
```
*removeAction* 向 Native 框架中移除已经注册过的 action，同时会移除协作者关系。

``` js
CoordinatorRegister.updateProperties(String sponserAffinity, String reponderAffinity, {name : value ...}, boolean notify);
```
*updateProperties* 更新 Native 框架已经注册过的 action 的作用域内的变量值。更新的参数为对象，对象中包含将被更新的作用域对象中的变量名及对应的值（暂时只支持 String / Number / bool）。notify 为 true 时表示需要通知当前关系链中的响应者。

| 属性                 | 作用                                                         |                             备注                             |
| -------------------- | ------------------------------------------------------------ | :----------------------------------------------------------: |
| coordinator-affinity | 亲和性，协作者框架通过该属性构造关系链                       |                     分发者和响应者均具备                     |
| coordinator-type     | 分发事件的类型，如`scroll`                                   | 分发者特有，如果需要支持多个分发事件类型可以使用`或运算符` 间隔，如`scroll 或运算符 touch` |
| coordinator-command  | 制定响应事件的指令，用法 `type:FuncName`，如`scroll:onScrollEvent` | 响应者特有，如果需要支持多个指令可以使用`或运算符`间隔，如`scroll:onScrollEvent 或运算符 touch:onTouchEvent` |
| coordinator-tag      | `Element` 对应的标识，在脚本的方法中用于区别`Element`的标识  | 分发者和响应者均具备，如果该属性值改变了会触发脚本中的`init`操作 |

> 分发者本身就是响应者，或运算符指 |



#### Lepus 引擎相关

1. 处理分发者和响应者事件 API

``` js
function init(tag)
```
*init* 方法是在脚本初始化以及构建关系链时候对 responder 和 sponsor 调用，可以在该方法中初始化 View 的状态。

``` js
// scrollTop / scrollLeft : 该 View 滑动的位置信息
function onDispatchScrollEvent(tag, scrollTop, scrollLeft)

// type : TOUCH_START = 0 / TOUCH_END = 1 / TOUCH_MOVE = 2 / TOUCH_CANCEL = 3
// touchX / touchY : 该 View 发生 touch 事件的相对于该 View 的相对坐标
// timeStamp : 事件发生的时间戳
function onDispatchTouchEvent(tag, type, touchX, touchY, timeStamp)
```
*dispatch* 方法是在事件发生时候向 sponsor 发起的事件分发调用，该方法用于做参数的预处理，不能对 View 做动效设置，只能设置 setConsumed 影响事件流。

``` js
function onTouchEvent(tag)
```
该方法为 coordinator-command 指定在 View 中响应事件的方法名称。调用时机是在事件响应和经过 dispatch 处理后。该方法可以对 View 做动效设置。

2. 引擎中交互动画 API

| 方法                                      | 作用                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| setTranslateX(Number)                     | 设置 View 在 X 轴的平移距离                                  |
| setTranslateY(Number)                     | 设置 View 在 Y 轴的平移距离                                  |
| setScaleX(Number)                         | 设置 View 在 X 轴的缩放大小，范围 0 ~ 1                      |
| setScaleY(Number)                         | 设置 View 在 Y 轴的缩放大小，范围 0 ~ 1                      |
| setRatateX(Number)                        | 设置 View 在 X 轴的旋转角度                                  |
| setRotateY(Number)                        | 设置 View 在 Y 轴的旋转角度                                  |
| setOriginX(Number)                        | 设置 View 在缩放或旋转时的中轴点的 x 值，默认在`View`中心    |
| setOriginY(Number)                        | 设置 View 在缩放或旋转时的中轴点的 y 值，默认在`View`中心    |
| setOpacity(Number)                        | 设置 View 的透明度，范围 0 ~ 1                               |
| setTopOffset(Number)                      | 设置 View 的可视区域的上边距的差值，正值为向外延伸，负值为向内收缩，不改变子 View 排版 |
| setBottomOffset(Number)                   | 设置 View 的可视区域的下边距的差值，正值为向外延伸，负值为向内收缩，不改变子 View 排版 |
| setRightOffset(Number)                    | 设置 View 的可视区域的右边距的差值，正值为向外延伸，负值为向内收缩，不改变子 View 排版 |
| setLeftOffset(Number)                     | 设置 View 的可视区域的左边距的差值，正值为向外延伸，负值为向内收缩，不改变子 View 排版 |
| setConsumed(Bool)                         | 设置该 View 是否消费该事件，若是则父 View 不能响应该事件，仅在事件分发阶段有效。 |
| setDuration(Bool)                         | 设置动画时间，默认为0                                        |
| dispatchEvent(String, String/Number/Bool) | 向前端传递事件，前端获取参数的方法为 e.detail                |
| setTimingFunction(String)                 | 设置动画差值器，默认`LINEAR`，其他为`EASE` `EASE_IN` `EASE_OUT` `EASE_IN_OUT` |

#### 代码示例，以 Vue.js 为实例

- Vue 文件

``` html
<template>
    ...
    <listview
      class="listview"
      coordinator-affinity="sponsorAffinity"
      coordinator-type="scroll">
      ...
      ...
      ...
    </listview>
    ...
    <view
      class="blue-mask"
      coordinator-affinity="responderAffinity"
      coordinator-command="scroll:onScrollEvent"></view>
    ...
</template>

<script>
    ...
    mounted() {
        CoordinatorRegister.registerAction("sponsorAffinity", "responderAffinity", action)
    },
    ...
</script>
<style>
    .listview {
        height: 500;
        width: 750;
    }
</style>
```

- Lepus 文件

``` js
var lastScrollTop = 0
var lastScrollLeft = 0
var max = 500
var min = 120
var minTitleScale = 0.75
function onDispatchScrollEvent(tag, scrollTop, scrollLeft) {
    lastScrollTop = scrollTop
    lastScrollLeft = scrollLeft
}

function onScrollEvent(tag, scrollTop, scrollLeft){
    if (scrollTop < max - min) {
        var y = 0 - scrollTop
        setTranslateY(y)
        var alpha = y / (min - max)
        setAlpha(alpha)
    }
    if (scrollTop > max - min) {
        setTranslateY(min - max)
        setAlpha(1)
    }
}
```



#### 后续

提供 CoodinatorContext 模块简化注册和更新等操作流程。



#### Tip

1. 尺寸转换问题，Lepus 文件中涉及的大小和 css 中设定的无 px 后缀数值表示一致，框架中在最后应用时会转化成 px。

