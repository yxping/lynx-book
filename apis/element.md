#### Element

元素，描述了所有相同种类元素所共有的基础方法和属性。相同种类的元素例如，view、label…..等。



#### API 描述



**tagName** `readonly`

当前元素标签名

- 类型：`string` 



**nodeType** `readonly`

不同元素节点的类型。框架中仅支持元素节点 1 和 文本节点 3。

- 类型：`number` 



**parentNode** `readonly`

当前元素的父元素

- 类型：`Element object` 



**offsetTop** `readonly`

当前元素距离父元素的上边距

- 类型：`number` 



**offsetLeft **`readonly`

当前元素距离父元素的左边距

- 类型：`number` 



**offsetHeight** `readonly`

当前元素的排版高度

- 类型：`number` 



**offsetWidth **`readonly`

当前元素排版宽度

- 类型：`number` 



**scrollTop**

当前元素上滚动位置

- 类型：`number`



**scrollLeft**

当前元素左滚动位置

- 类型：`number`



**scrollWidth** `readonly`

当前元素可滚动的总宽度

- 类型：`number` 



**scrollHeight **`readonly`

当前元素可滚动的总高度

- 类型：`number` 



**nextSibling**

当前元素在父元素中的下一个兄弟元素，没有则返回 null

- 类型：`Element object` `readonly`



**clientWidth**

根元素的宽度

- 类型：`number` `readonly`



**clientHeight**

根元素的高度

- 类型：`number` `readonly`



**textContent**

当前元素的文本内容，可以进行设置。仅对 label 有效。

- 类型：`string` 



**childNodes**

当前元素的子元素集合，没有则为空。

- 类型：`Array` 



**firstChild**

当前元素的第一个子元素，没有则为空

- 类型：`Element object` 



**appendChild**

> element.appendChild(child)

将元素作为子节点插入当前元素的子元素队列的最后

- 类型：`function`
- 参数：`Element object`，待插入的元素对象

使用示例：

```js
var wrap = document.createElement('view');
var view = document.createElement('view');

wrap.appendChild(view);
```



**appendChildren**

> element.appendChildren([child1, child2…])

将元素队列作为子元素数组插入当前元素的子元素队列的最后

- 类型：`function`
- 参数：`Array`，待插入的元素数组

使用示例：

```js
var wrap = document.createElement('view');

var views = [];
var view1 = document.createElement('view');
var view2 = document.createElement('view');

views.push(view1);
views.push(view2);

wrap.appendChildren(views);
```



**insertChildAtIndex**

> element.insertChildAtIndex(child, index)

将元素作为子元素插入当前元素的子元素队列的指定位置

- 类型：`function`
- 参数 1：`Element object`，待插入元素
- 参数 1：`number`，插入子元素的位置。如果发生越界，则会发生异常。可选，默认 -1，队尾。



**removeChildByIndex**

> element.removeChildByIndex(index)

删除当前元素的指定的顺序位置的子元素

- 类型：`function`
- 参数：`number`，指定待删除子元素的位置



**getChildByIndex**

> var child = element.getChildByIndex(index)

获取当前元素子元素队列中指定顺序位置的子元素，如果越界则为空

- 类型：`function`
- 参数：`number`，指定获取子元素的位置
- 返回：`Element object`，获取到的子元素



**addEventListener**

> element.addEventListener(event, function)

为一个指定事件添加回调函数，回调函数的参数为事件对象

- 类型：`function`
- 参数 1：`string`，指定事件
- 参数 2：`function`，当事件触发后执行的函数

示例

```js
var view = document.createElement('view');

view.addEventListener('click', function(event) {
  console.log(event);
});
```



**removeEventListener**

> element.removeEventListener(event, function)

删除当前元素中的指定事件的指定函数

- 类型：`function`
- 参数 1：`string`，指定事件
- 参数 2：`function`，待删除的函数，必须和 addEventListener 时注册的函数相同，否则不能删除



**setAttribute**

> element.setAttribute(key, value)

给当前元素设置属性

- 类型：`function`
- 参数 1：`string`，需要设置的属性键
- 参数 2：`function`，需要设置的属性值



**setAttribution**

> element.setAttribution(object)

通过对象中的键值对给当前元素设置属性

- 类型：`function`
- 参数：`object`，对象中包括将要设置的键值对



**hasAttribute**

> var result = element.hasAttribute(key)

判断当前的属性是否已经被设置

- 类型：`function`
- 参数：`string`，待查询属性字符串
- 返回：`boolean`，如果属性已经被设置则返回 true，否则 false



**removeAttribute**

> element.removeAttribute(key)

删除指定的属性

- 类型：`function`
- 参数：`string`，待删除属性字符串



**setStyle**

> element.setStyle(key, value)

给当前元素设置样式，

- 类型：`function`
- 参数 1：`string`，需要设置的样式键
- 参数 2：`string ` 或 `number`，需要设置的样式值



**setText**

> element.setText(text)

给当前文本节点设置文本

- 类型：`function`
- 参数：`string`，需要设置的文本内容



**getText**

> var text = element.getText()

获取当前文本节点的内容

- 类型：`function`
- 返回：`string`，当前文本节点内容



**hasChildNodes**

> var result = element.hasChildNodes()

判断当前元素是否具有子元素

- 类型：`function`
- 返回：`boolean`，是否具有子元素



**animate**

> var animation = element.animate(keyframes, option)

为当前的元素创建动画，通过 animation 可以进行动画过程的事件监听和控制。具体请参见动画

- 类型：`function`
- 参数 1：`object`，定义动画路径的一系列关键帧
- 参数 2：`object`，定义动画的参数
- 返回：`Animation object`，进行动画过程的事件监听和控制的对象





