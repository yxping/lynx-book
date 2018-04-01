#### CSS 

在 Lynx 中在 CSS 的使用上有一些默认的约定。

- 布局方式基于 flex 布局
- 盒模型默认为 border-box 类型
- 样式取值有三种类型：`number` 数字、  `string` 字符串、 `length` 长度（如数字、auto或者带有 px 的数字）
- 和数字相关的数值，暂不支持百分数，如果带 px 单位的数字，不会做屏幕适配运算，如：width: 320px。



#### 样式支持

**width**

设置元素的宽度属性

- 类型：`length`
- 默认值：auto



**height**

设置元素的高度属性

- 类型：`length`
- 默认值：auto



**min-width**

设置元素的最大宽度属性

- 类型：`length`
- 默认值：auto



**min-height**

设置元素的最小高度属性

- 类型：`length`
- 默认值：auto



**max-width**

设置元素的最大宽度属性

- 类型：`length`
- 默认值：auto



**max-height**

设置元素的最大高度属性

- 类型：`length`
- 默认值：auto



**margin-left**

设置元素左外边距

- 类型：`length`
- 默认值：auto



**margin-top**

设置元素上外边距

- 类型：`length`
- 默认值：auto



**margin-right**

设置元素右外边距

- 类型：`length`
- 默认值：auto



**margin-bottom**

设置元素的下外边距

- 类型：`length`
- 默认值：auto



**padding-left**

设置元素的左内边距

- 类型：`length`
- 默认值：auto



**padding-top**

设置元素的上内边距

- 类型：`length`
- 默认值：auto



**padding-right**

设置元素的右内边距

- 类型：`length`
- 默认值：auto



**padding-bottom**

设置元素的下外边距

- 类型：`length`
- 默认值：auto



**left**

设置 `absolute` `fixed` 元素左边位置

- 类型：`length`
- 默认值：auto



**top**

设置 `absolute` `fixed` 元素上边位置

- 类型：`length`
- 默认值：auto



**right**

设置 `absolute` `fixed` 元素右边位置

- 类型：`length`
- 默认值：auto



**bottom**

设置 `absolute` `fixed` 元素下边位置

- 类型：`length`
- 默认值：auto



**border-width**

设置元素边框宽度

- 类型：`length`
- 默认值：auto



**border-color**

设置元素边框颜色

- 类型：`length`
- 默认值：auto



**border-radius**

设置元素边框圆角

- 类型：`length`
- 默认值：auto



**border-shadow**

- 缺省



**position**

设置元素定位方式

- 类型：`string`
- 默认值：`relative`

取值

| 值       | 描述                                                       |
| -------- | ---------------------------------------------------------- |
| absolute | 绝对定位                                                   |
| relative | 相对定位，只是作为 absolute 定位的参照系，没有相对定位功能 |
| fixed    | 相对窗口的固定定位                                         |



**display**

设置元素显示方式

- 类型：`string`
- 默认值：`flex`

取值

| 值   | 描述          |
| ---- | ------------- |
| flex | flex 布局方式 |
| none | 隐藏元素      |



**background-color**

设置元素背景颜色

- 类型：`string`
- 默认值：`transparent`

取值

| 值    | 描述                           |
| ----- | ------------------------------ |
| color | 16 进制颜色值、rgb() 或 rgba() |



**background-image**

- 缺省



**background-position**

- 缺省



**background-size**

- 缺省



**background-repeat**

- 缺省



**object-fit**

设置元素的图片裁剪方式

- 类型：`string`
- 默认值：`cover`

取值

| 值      | 描述                                     |
| ------- | ---------------------------------------- |
| fill    | 填充，拉伸到整个空间                     |
| contain | 容纳，按原始比例显示，并且不会裁剪任何边 |
| cover   | 遮盖，裁剪掉超出的部分                   |



**opacity**

设置元素透明度

- 类型：`number`
- 默认值: 1

**取值**

| 值     | 描述        |
| ------ | ----------- |
| number | 0 ~ 1的小数 |



**flex-direction**

定义 flex 容器中子元素的布局方向

- 类型：`string`
- 默认值: `row`

**取值**

| 值     | 描述     |
| ------ | -------- |
| row    | 横向布局 |
| column | 纵向布局 |



**flex-wrap**

定义 flex 容器中的子元素在布局位置不足时是否换行

- 类型：`string`
- 默认值: `nowrap`

**取值**

| 值     | 描述   |
| ------ | ------ |
| nowrap | 不换行 |
| wrap   | 换行   |



**justify-content**

定义 flex 容器中子元素在主轴上的布局方式

- 类型：`string`
- 默认值: `flex-start`

**取值**

| 值            | 描述                                                   |
| ------------- | ------------------------------------------------------ |
| flex-start    | 子元素全部靠容器元素开始的位置                         |
| flex-end      | 子元素全部靠容器元素结束的位置                         |
| center        | 子元素全部靠容器元素中间位置                           |
| space-between | 在同一条线上分布的子元素对齐容器两端，元素之间间隔相等 |
| space-around  | 在同一条线上分布的每个子元素两侧留下空间相等           |



**align-items**

定义 flex 容器中子元素在交叉轴上的布局方式

- 类型：`string`
- 默认值: `flex-start`

**取值**

| 值         | 描述                                                 |
| ---------- | ---------------------------------------------------- |
| center     | 子元素在交叉轴的中点对齐                             |
| flex-start | 子元素在交叉轴的起点对齐                             |
| flex-end   | 子元素在交叉轴的重终点对齐                           |
| stretch    | 如果子元素威慑高度，子元素被拉伸以填充容器交叉轴大小 |



**align-self**

设置当前元素在 flex 容器中交叉轴上的布局方式

- 类型：`string`
- 默认值: `auto`

**取值**

| 值         | 描述                                    |
| ---------- | --------------------------------------- |
| auto       | 自动布局，表现为 `align-items` 设置的值 |
| center     | 当前元素在交叉轴的中点对齐              |
| flex-start | 当前元素在交叉轴的起点对齐              |
| flex-end   | 当前元素在交叉轴的终点对齐              |



**flex**

元素设置自己在 flex 布局中分布的权值

- 类型：`number`
- 默认值: `none`

**取值**

| 值     | 描述   |
| ------ | ------ |
| number | 正整数 |



**color**

设置元素前景颜色

- 类型：`string`
- 默认值：#000

**取值**

| 值    | 描述                           |
| ----- | ------------------------------ |
| color | 16 进制颜色值、rgb() 或 rgba() |



**font-size**

设置文字大小

- 类型：`length`
- 默认值：14





**text-align**

设置文字对齐方式

- 类型：`string`
- 默认值：left

**取值**

| 值     | 描述 |
| ------ | ---- |
| left   | 居左 |
| center | 居中 |
| right  | 居右 |



**font-weight**

设置文字粗细

- 类型：`string`
- 默认值：normal

**取值**

| 值     | 描述 |
| ------ | ---- |
| normal | 正常 |
| bold   | 粗体 |



**white-space**

设置文字是否换行

- 类型：`string`
- 默认值：normal

**取值**

| 值     | 描述   |
| ------ | ------ |
| normal | 换行   |
| nowrap | 不换行 |



**text-overflow**

文字溢出宽度时的表现形式

- 类型：`string`
- 默认值：clip

**取值**

| 值       | 描述       |
| -------- | ---------- |
| clip     | 截断       |
| ellipsis | 变成点点点 |



**text-decoration**

文字衬线

- 类型：`string`
- 默认值：none

**取值**

| 值           | 描述   |
| ------------ | ------ |
| none         | 无     |
| underline    | 下划线 |
| line-through | 删除线 |



**line-height**

文本行高

- 类型：`length`
- 默认值：auto



**z-index**

元素的 z 轴层级，只支持在当前容器上的层级，不能跨越容器以上的层级。

- 类型：`number`
- 默认值: 1



**flex-shrink**

- 缺省



**flex-grow**

- 缺省



**flex-basis**

- 缺省



**align-content**

- 缺省

