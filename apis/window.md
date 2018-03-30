#### Window

JS 运行环境的上下文，相等于 global 对象。



#### API 描述



**devicePixelRatio**

> window.devicePixelRatio

当前设配屏幕密度

- 类型：`number`
- 非必要，screen 中可以支持



**setTimeout** `待完善`

> var id = window.setTimeout(function, delay)

设置定时器，在定时器到期后执行设定的函数。

- 类型：`function`
- 参数 1：`function`，定时器到期后执行的函数
- 参数 2：`number`，延迟的毫秒数。可选，默认为 0。
- 返回： `number`，正整数，通过这个 id 可以通过 clearTimeout 取消该定时器

使用示例：

```js
setTimeout(function() {
    console.log('一个 1s 的延时调用');
}, 1000);
```



**setInterval**`待完善`

> var id = window.setInterval(function, delay)

设置定时器，按照指定的延时重复执行设定的函数。

- 类型：`function`
- 参数 1：`function`，定时器到期后执行的函数
- 参数 2：`number`，重复执行时间间隔毫秒数。必须设置。
- 返回： `number`，正整数，通过这个 id 可以通过 clearInterval 取消该定时器

使用示例：

```js
setInterval(function() {
    console.log('一个间隔为 1s 的重复调用');
}, 1000);
```



**clearTimeout**

> window.clearTimeout(id)

取消由 setTimeout 方法设置的定时器

- 类型：`function`
- 参数：`number`，setTimout 方法返回的 id



**clearInterval**

> loader.script(url, success, error)

取消由 setInterval 方法设置的定时器

- 类型：`function`
- 参数：`number`，setInterval 方法返回的 id



**height**

> window.height

当前设配屏幕高度，像素值

- 类型：`number`



**promp**

- 缺省
- 非必要



**open**

- 缺省
- 非必要



**addEventListener**

- 缺省



**removeEventListener**

- 缺省



**dispatchEvent**

- 缺省

