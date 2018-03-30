#### Document

提供构建页面和获取文档内容的能力。Lynx 默认提供了 document 对象，通过 document 对象即可使用相关 API。



#### API 描述



**body**

> document.body

页面（文档）中的根节点

- 类型：`Element object`



**createElement**

> document.createElement(tag)

创建 Element 元素，如果创建失败则返回 null 值。

- 类型：`function`
- 参数：`string`，需要创建的元素类型（如 view…等）
- 返回： `Element object`，创建的 Element 对象，有可能是 null 值。

使用示例：

```js
var text = document.createElement('label');
var wrap = document.createElement('view');

text.setText('Hello Lynx');

wrap.setStyle({
  backgroundColor: '#0f0',
});

wrap.appendChild(text);

document.body.appendChild(wrap);
```



**createTextNode**

> document.createTextNode(text)

创建 TextNode 元素（文本节点），仅支持将文本节点挂在 label 节点下。

- 类型：`function`
- 参数：`string`，需要设置的文本
- 返回： `Element object`，创建的 TextNode 对象

使用示例：

```js
var textnode = document.createTextNode('这是一段文本');
var label = document.createElement('label');

label.appendChild(textnode);
document.body.appendChild(label);
```



**querySelector**

> document.querySelector(selector)

返回文档中匹配给定选择器的最后一个元素。与 web 标准不同！！！

- 类型：`function`
- 参数：`string`，大小写敏感，选择器，仅支持 # 开头的 id 选择器，与 web 标准不同！！！
- 返回：`Element object`



**getElementById**

> document.getElementById(id)

返回文档中匹配给定 id 的最后一个元素

- 类型：`function`
- 参数：`string`，大小写敏感字符串
- 返回：`Element object`



**addEventListener**

- 缺省



**removeEventListener**

- 缺省



**createEvent**

- 缺省
- 非必要



**dispatchEvent**

- 缺省



**readyState**

- 缺省
- 非必要



**ontouchstart**

- 缺省
- 非必要



**ontouchmove**

- 缺省
- 非必要



**ontouchend**

- 缺省
- 非必要



