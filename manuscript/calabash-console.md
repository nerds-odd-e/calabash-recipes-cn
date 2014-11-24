# Calabash 控制台
Calabash 的控制台是一个命令行的交互环境，在控制台中能够完整的操作要测试的应用。

## 打开 Calabash 控制台
对于 iOS 项目，只需要在项目的根目录，执行命令

    calabash-ios console
    
对于 Android 项目，需要指定 APK 文件的路径

    calabash-android console build/target/YourApp.apk
    
## Calabash 控制台指令（Android & iOS）
本节会介绍两个版本的 Calabash 共有的指令，在后面的内容中会有特定于 Android/iOS 平台的指令。
### start_test_server_in_background
这个指令会自动打开并运行我们的应用，如果是 iOS 项目，并且也使用的是模拟器，这个指令同时也会运行 iOS 模拟器。对于 Android 项目，需要每次手工打开模拟器。

## Calabash for Android 控制台指令
本节内容介绍的是仅用于 Android 项目的控制台指令。

## reinstall_apps
重新安装应用以及 Test Servers。

## Calabash for iOS 控制台指令
本节内容介绍的是仅用于 iOS 项目的控制台指令。
