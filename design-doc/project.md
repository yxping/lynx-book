## Lynx
Lynx是一个跨平台开发库，可以使用vue的开发模式，开发出与Native体验一直的应用。Lynx的设计可参看[Design Document](./lynx-native-design.md)

<span id="lynx-vue"></span>
## lynx-vue
lynx-vue是Lynx依赖的基础开发框架，lynx-vue是基于`vue v2.4.2`版本的修改，提供vue的开发方式进行开发。lynx-vue针对vue做了如下修改:

* 修改nextTick方法，使用setTimeout实现 （src/core/util/env.js）
* 修改style设置方式，将所有style通过SetStyle接口进行设置 （src/platforms/web/runtime/modules/style.js）
* 添加view,label,img,listview,scrollview,viewstub的支持 （src/platforms/web/util/element.js）


修改lynx-vue之后使用```npm run build```进行编译，编译结果在dist目录内，修改编译完的结果需要导入Lynx的模版内，方便通过模版建立工程时使用，具体参看[lynx-simple-template](#lynx-simple-template)

lynx-vue的应用打包是基于vue-ssr，因此对ssr相关的代码也做了相应的修改，主要修改内容同样是style的设置，修改ssr相关的内容之后使用```npm run build:ssr```进行编译，结果在packages文件夹下，主要有vue-template-compiler和vue-server-renderer, 其中编译完的vue-server-renderer需要导入Lynx的模版内，方便通过模版建立工程时使用，具体参看：lynx-simple-template，另外vue-template-compiler需要上传github并且发布release tag，以便模版工程下载使用。



## lynx-vue-loader
lynx-vue-loader是lynx-vue的loader，是基于`v13.0.2`版本修改，其中主要修改了对于style设置支持，之所以要修改style的设置，其原因是lynx不支持css selector，所有css style都视为inline style

## lynx-cli
lynx-cli是lynx的命令行工程，该工程提供lynx的命令行，使用请参看[lynx-cli](https://github.com/hxxft/lynx-cli).修改之后可以上传至npm，方便开发使用。

<span id="lynx-simple-template"></span>
## lynx-simple-template
lynx-simple-template是开发模版，lynx-simple-template提供了最基本开发的框架，基于这个模版，可以开发跨平台应用，开发模式和vue一致，具体demo可以参看lynx-natvie工程中的Example目录。
lynx-simple-template的libs目录是基础JS库，包含`lynx-vue`和`vue-server-renderer`都生产于lynx-vue，具体生成方式参看[lynx-vue](#lynx-vue)

## lynx-android-template
lynx-android-template是android的模版工程，用于打包应用，在cli的init的过程中会去github拉取模版工程，lynx-native的android修改编译后产生的sdk需要复制到app/libs目录下进行发布。请不要随意改变模版工程的目录结构，不合适的修改可能会导致打包失败。

## lynx-native
lynx-native是lynx的核心，提供了使用JS开发原生应用的能力，具体请参看[lynx-native](https://github.com/hxxft/lynx-native)文档。
