#### Loader

网络请求 API。



#### API 描述



**script**

> loader.script(url, success, error)

Loader 用于发送请求的方法 

- 类型：`function`
- 参数 1：`string`，请求地址，必须是完整地址，以 `http` 或 `https` 开头
- 参数 2：`function`，请求成功的回调函数，回参根据函数不同有区别
- 参数 3：`function`，请求失败的回调函数，回参错误信息
- 返回： `Element object`，创建的 Element 对象，有可能是 null 值。

示例，如果要从任意地址中获取内容，则可以简单的使用：

```js
loader.script('https://www.lynx.com', funciont(result) {
              	console.log('success');
              }, funciont(error) {
    			console.log('error');
			  });
```





#### 后续 

后续会在 Loader 模块中提供与 web 标准的 Fetch API 一致的方法。方便使用 Get 和 Post 适应更多的场景。