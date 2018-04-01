# How to Build

Lynx是一个跨平台开发框架，底层基于C++编写，能方便的在Android/iOS平台上编译运行。



## 获取代码

```shell
$ git clone https://github.com/hxxft/lynx-native.git
```



##编译

- Android平台编译

  - 执行

    ```shell
    $ cd lynx-native
    $ ./lynx/build/prebuild.sh
    ```

  - 使用Android Studio打开`{$dir}/lynx-native/Android`

  - 编译运行example工程

- iOS平台编译

  - 执行

    ```shell
    $ cd lynx-native/iOS/
    $ pod install
    ```

  - 使用xcode打开`{$dir}/lynx-native/iOS/lynx.xcworkspace`

  - 编译运行

