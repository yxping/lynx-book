Lynx是一个跨平台开发框架，底层基于C++编写，能方便的在Android/iOS平台上编译运行。

- Android平台编译
  - 执行`{$dir}/lynx-native/lynx/build/prebuild.sh`
  - 使用Android Studio打开`{$dir}/lynx-native/Android`
  - 编译运行example工程
- iOS平台编译
  - 在`{$dir}/lynx-native/iOS/`执行`pod install`, 没有安装pod的请先安装pd
  - 使用xcode打开`{$dir}/lynx-native/iOS/lynx.xcworkspace`