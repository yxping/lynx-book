#### 动画

Lynx 的提供了两种方式实现动画：根据时间轴设置关键帧的动画，和支持自定义的交互控制动画（参见交互动画一文）。以下主要介绍关键帧动画的实现。



Lynx 提供了全局 API 支持关键帧动画的设置，其中的参数和返回值在下述进行介绍。

> var animation = element.animate(keyframes, option)



**keyframes**

动画路径的一系列关键帧

- 类型：`object`

可设置的属性

| 属性名          | 类型          | 默认值     | 说明                                                         |
| --------------- | :------------ | ---------- | ------------------------------------------------------------ |
| duration        | number        | 300        | 动画时长，单位毫秒                                           |
| easing          | string        | ease       | 定义当前帧到下一帧的过渡效果，取值 `liner`、`ease`、`ease-in`、`ease-out`、`ease-in-out`、`cubic-bezier(0.29, 1.01, 1, -0.68)` |
| transformOrigin | array: [x, y] | [0.5, 0.5] | 变换原点，相当于元素本身宽高的百分比，不支持z轴设置          |
| tranform        | object        | undefined  | 定义动画变化效果                                             |
| perspective     | number        | 1000       | 透视投影距离                                                 |
| opacity         | number        | 1          | 透明度，范围：[0, 1]                                         |
| offset          | number        | undefined  | 定义每一帧在归一化时间轴上的位置，距离起点的偏移             |

**tranform**

定义一系列变化效果。帧中的变化值如果没有设置，则以默认值为标准

- 类型：`object`

可设置参数

| 参数   | 类型                  | 说明                                                         |
| ------ | ------------------------ | ------------------------------------------------------------ |
| scaleX | number             | 在 X 轴缩放的倍数，默认 1                                   |
| scaleY | number               | 在 Y 轴缩放的倍数，默认 1                                      |
| translateX | number                         | 在 X 轴偏移，默认 0                      |
| translateY | number                        | 在 Y 轴偏移，默认 0                      |
| translateZ | number                         | 在 Z 轴偏移，默认 0                    |
| rotateX | number | 绕 X 轴旋转的角度，默认 0 |
| rotateY | number | 绕 Y 轴旋转的角度，默认 0 |
| rotateZ | number | 绕 Z 轴旋转的角度，默认 0 |
| skewX | number                   | 绕 Y 轴倾斜的角度，默认 0                                       |
| skewY | number                   | 绕 Z 轴倾斜的角度，默认 0                              |



**options**

动画参数配置 

- 类型：`object`

可设置属性

| 属性名     | 类型   | 默认值 | 说明                                                         |
| ---------- | ------ | ------ | ------------------------------------------------------------ |
| duration   | number | 300    | 动画时长，单位毫秒                                           |
| easing     | string | ease   | 动画整体过渡效果，取值 `liner`、`ease`、`ease-in`、`ease-out`、`ease-in-out`、`cubic-bezier(0.29, 1.01, 1, -0.68)` |
| delay      | number | 0      | 动画开始的延时时间                                           |
| endDelay   | number | true   | 动画结束的延时时间                                           |
| id         | string | 1000   | 动画标识                                                     |
| direction  | string | normal | 决定动画播放是否向前播放 `normal`，或是反向播放 `reverse` ，或是在动画重复时先向前播放，接着交替播放 `alternate`，或是在动画重复时候先反向播放，接着交替播放 `alternate-reverse` 。默认是 `normal`。 |
| fill       | string | none   | 设置动画是否在开始提前展示或结束后保持动画状态。`backwards` 在动画开始前展示第一帧；`forwards` 在动画结束后保持第一帧；`both` 包含前述两者； `none` 不提前展示或结束保持状态。 |
| iterations | number | 1      | 动画总体执行次数。0 为不执行，`Infinity` 为无限循环。        |



**Animation API**

进行动画过程的事件监听和控制的对象



**addEventListener**

> animation.addEventListener(event, function)

指定事件添加回调函数，回调函数的参数为事件对象

- 类型：`function`
- 参数 1：`string`，指定事件
- 参数 2：`function`，当事件触发后执行的函数

可添加事件

| 名称   | 说明                   |
| ------ | ---------------------- |
| cancel | 该动画被主动取消的事件 |
| finish | 该动画执行结束的事件   |



**removeEventListener**

> animation.removeEventListener(event, function)

移除指定事件的指定回调函数

- 类型：`function`
- 参数 1：`string`，指定事件
- 参数 2：`function`，待移除回调函数



**cancel**

取消正在执行的动画

- 类型：`function`



**示例**

定义时长为 1s 从透明到不透明同时放大一倍的动画。

```js
var animation = element.animate([ 
    								{
                                        offset: 0,
                                        transform: {
                                            scaleX: 1
                                        },
                                        opacity: 0
                                    },
                                    {
                                        offset: 1,
                                        transform: {
                                            scaleX: 2
                                        },
                                        opacity: 1
                                    }
                                ], {
    								duration: 1000,
    								easing: 'linear'
								});

animation.addEventListener('finish', function() {
    console.log('animation end');
});
```



#### Tips

1. 动画在 Android 上的实现基于 Animation，变化的位置不会引起实际位置的变化，当元素的进行相关变换后，滑动事件就会不准确（点击事件进行了修正）。
2. 如果将动画应用到未渲染出 Native View 的 Element 上，该动画将不会生效。
3. 动画上涉及到的数字，均不会做屏幕适配运算。




#### 后续

支持设置设置带 px 的数值。支持屏幕适配运算的数值。