# 在 Android 项目中使用 Calabash

## 快速开始
在 Android 项目中使用 Calabash 做自动化验收测试，安装过程比 iOS 项目简单多了。不像 iOS 那样还要把 Calabash Framework 集成到项目中，还要配置项目。在 Android 项目中仅仅使用 `gem` 命令安装好 **Calabash for Android** 就可以了。

如果还没有安装过 **Calabash for Android**，就使用 `gem` 命令来安装。

``` console
$ gem install calabash-android
Fetching: escape-0.0.4.gem (100%)
Successfully installed escape-0.0.4
Fetching: httpclient-2.3.4.1.gem (100%)
Successfully installed httpclient-2.3.4.1
（省略大量的控制台输出）
Parsing documentation for calabash-android-0.5.5
Installing ri documentation for calabash-android-0.5.5
Done installing documentation for escape, httpclient, awesome_print, rubyzip, multi_test, multi_json, gherkin, diff-lcs, builder, cucumber, slowhandcuke, retriable, calabash-android after 28 seconds
13 gems installed
```

在安装的过程中，需要编译出二进制扩展，所以需要系统中已经安装好相应的编译工具以及 Ruby 开发包。

对于 Mac OS X 来说，首先要有 **Command Line Tools**。如果已经安装了最新版的 Xcode，可能已经安装了 **Command Line Tools**。如果没有就需要在 Xcode 的偏好设置里面的下载那里来安装。

对于 Linux 来说，以 Ubuntu 为例，安装开发工具 **build-essential** 以及 Ruby 的开发包  **ruby-dev**。

安装好 **Calabash for Android** 以后，下一步需要生成运行测试所需的基本目录结构及依赖文件。执行 `calabash-android gen`，并根据提示按下回车键确认创建 *features* 目录，如果想终止这个命令就按下 Ctrl-Z。

``` console
$ calabash-android gen

----------Question----------
I'm about to create a subdirectory called features.
features will contain all your calabash tests.
Please hit return to confirm that's what you want.
---------------------------


----------Info----------
features subdirectory created.
---------------------------
```

与 **Calabash for iOS** 不同的是，执行的时候不会自动打开模拟器。如果模拟器没有运行，或者没有连接开发用的手机。执行测试的时候就会因为找不到设备而出错。所以需要事先打开 Android 模拟器或者连接好已经启用开发模式的 Android 手机。用 `adb devices` 命令来确认一下开发设备已经就绪。

另一个不同的是，执行的命令后面必须添加编译好的 APK 文件路径，而且不能直接使用 `cucumber` 命令，必须始终使用 `calabash-android` 这个命令。比如打开控制台的命令就是 `calabash-android console path/to/yourapp.apk`，执行测试的命令就是 `calabash-android run path/to/yourapp.apk`

最后执行一下默认的测试，验证所有的环境已经就绪。

## 使用预配置好的虚拟机

推荐使用 [Genymotion](http://www.genymotion.com/) 这个 Android 模拟器，因为是基于 Android x86 虚拟机，运行在 VirtualBox 上，所以运行速度要远快于 Android SDK 中提供的模拟器。

### 在虚拟机中连接 Genymotion 模拟器
在 Genymotion 中创建好一个 Android 模拟器，并且运行起来后。我们可以在模拟器的标题栏中看到模拟器的 IP 地址，比如是 **192.168.56.101**，或者通过在本地机器上运行 `adb devices` 命令来查看一下

``` console
$ adb devices
List of devices attached
192.168.56.101:5555	device
```

确定好模拟器的 IP 地址后，在虚拟机中使用 `adb connect` 命令来连接到 Genymotion 模拟器，端口默认都是 **5555**。连接成功后，用 `adb devices` 来确认一下。

``` console
$ adb devices
List of devices attached

$ adb connect 192.168.56.101:5555
connected to 192.168.56.101:5555

$ adb devices
List of devices attached
192.168.56.101:5555	device
```
