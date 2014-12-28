# 在 iOS 项目中使用 Calabash

## 快速开始
在安装完毕 Calabash，并且也配置好 iOS 项目后，下一步需要初始化基本的测试脚本依赖文件。

首先打开终端，比如使用 *Terminal.app* 或者 *iTerm.app*。改变当前目录到 iOS 项目的根目录中，执行命令并根据提示按下回车键确认创建 *features* 目录，如果想终止这个命令就按下 Ctrl-Z。

``` console
$ calabash-ios gen

----------Question----------
I'm about to create a subdirectory called features.
features will contain all your calabash tests.
Please hit return to confirm that's what you want.
---------------------------


----------Info----------
Features subdirectory created.
Make sure you've build your -cal scheme in XCode and
try executing

cucumber
---------------------------
```

在 `calabash-ios gen` 这个命令成功执行后，如果已经在 XCode 中编译了带有 *-cal* 结尾的 Schema，那就可以直接执行 `cucumber` 来运行默认的一个测试。这个测试仅仅是确保所有的环境已经就绪，包括

 * Calabash 已经安装成功
 * iOS 项目中已经正确配置了 Calabash 框架，并且有一个 *-cal* 结尾的 Schema
 * iOS 项目中以 *-cal* 结尾的 Schema 已经成功编译
 * 项目目录下也存在 *features* 目录，以及 Calabash 相关的依赖脚本
 
``` console
$ cucumber
Feature: Running a test
  As an iOS developer
  I want to have a sample feature file
  So I can begin testing quickly

  Scenario: Example steps                            # features/my_first.feature:6
    Given I am on the Welcome Screen                 # features/step_definitions/my_first_steps.rb:1
    Then I swipe left                                # calabash-cucumber-0.11.4/features/step_definitions/calabash_steps.rb:234
    And I wait until I don't see "Please swipe left" # calabash-cucumber-0.11.4/features/step_definitions/calabash_steps.rb:165
    And take picture                                 # calabash-cucumber-0.11.4/features/step_definitions/calabash_steps.rb:229

1 scenario (1 passed)
4 steps (4 passed)
0m12.564s
```

如果一切正常，将会看到 iOS 模拟器自动运行，并且打开了应用。然后应用自动关闭，iOS 模拟器回到首屏界面。

## 详细实战

### 搭建本地的 WordPress 服务
我们将为全球流行的开源博客软件 WordPress 的手机客户端编写自动化验收测试，首先要搭建一个本地的 WordPress 服务器。

我们已经提供了一个可用的 Vagrant 配置文件，位于 [Odd-e 在 Github 的仓库](https://github.com/nerds-odd-e/vagrant-wordpress)，直接把仓库克隆到本地

``` console
$ git clone https://github.com/nerds-odd-e/vagrant-wordpress.git
Cloning into 'vagrant-wordpress'...
remote: Counting objects: 36, done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 36 (delta 11), reused 33 (delta 8)
Unpacking objects: 100% (36/36), done.
Checking connectivity... done.
```

然后改变当前路径到 *vagrant-wordpress* 中，执行 `vagrant up` 即可。如果首次执行这个命令，可能会自动下载 *Ubuntu 14.04* 的一个虚拟机。依赖于网络的因素，可能会耗费数十分钟，以后再次执行就无需重复下载了。

``` console
$ cd vagrant-wordpress
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'ubuntu/trusty32' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'ubuntu/trusty32'
    default: URL: https://vagrantcloud.com/ubuntu/trusty32
==> default: Adding box 'ubuntu/trusty32' (v14.04) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/ubuntu/boxes/trusty32/versions/14.04/providers/virtualbox.box
==> default: Successfully added box 'ubuntu/trusty32' (v14.04) for 'virtualbox'!
==> default: Importing base box 'ubuntu/trusty32'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/trusty32' is up to date...
（省略大量的控制台输出）
==> default: Site 000-default disabled.
==> default: To activate the new configuration, you need to run:
==> default:   service apache2 reload
==> default: Enabling site wordpress.
==> default: To activate the new configuration, you need to run:
==> default:   service apache2 reload
==> default: Enabling module rewrite.
==> default: To activate the new configuration, you need to run:
==> default:   service apache2 restart
==> default:  * Restarting web server apache2
==> default:    ...done.
```

命令执行成功后，就在本地运行了一个 WordPress 服务。直接打开浏览器，通过 URL [http://wordpress.local](http://wordpress.local) 就能访问到本地服务。

### 获得 WordPress for iOS 源码

WordPress for iOS 的源码位于 [https://github.com/wordpress-mobile/](https://github.com/wordpress-mobile/)

``` console
$ git clone https://github.com/wordpress-mobile/WordPress-iOS.git
Cloning into 'WordPress-iOS'...
remote: Counting objects: 93355, done.
remote: Compressing objects: 100% (52/52), done.
remote: Total 93355 (delta 28), reused 0 (delta 0)
Receiving objects: 100% (93355/93355), 224.43 MiB | 5.25 MiB/s, done.
Resolving deltas: 100% (65112/65112), done.
Checking connectivity... done.
```

### 把 Calabash 集成到 WordPress for iOS 中

第一步，切换到 *calabash* 分支下

``` console
$ git checkout -b calabash
Switched to a new branch 'calabash'
```

第二步，为项目创建一个隔离的 Ruby 环境

``` console
$ echo 2.1.5 > .ruby-version
$ echo WordPress-iOS > .ruby-gemset
$ rvm rvmrc load
ruby-2.1.5 - #gemset created /Users/chaifeng/.rvm/gems/ruby-2.1.5@WordPress-iOS
ruby-2.1.5 - #generating WordPress-iOS wrappers - please wait
```

第三步，安装 bundler

``` console
$ gem install bundler
Fetching: bundler-1.7.8.gem (100%)
Successfully installed bundler-1.7.8
Parsing documentation for bundler-1.7.8
Installing ri documentation for bundler-1.7.8
Done installing documentation for bundler after 2 seconds
1 gem installed
```

第四步，用 bundler 安装 cocoapods 和 Calabash

在项目的根目录里创建一个名为 *Gemfile* 的文件，内容为：

> source 'http://rubygems.org'
>
> gem 'calabash-cucumber'
> gem 'cocoapods'

文件保存后，执行 `bundle install` 来安装 CocoaPods 和 Calabash

``` console
$ bundle install
Fetching gem metadata from http://rubygems.org/..........
Fetching additional metadata from http://rubygems.org/..
Resolving dependencies...
Installing CFPropertyList 2.2.8
Installing i18n 0.6.11
Using json 1.8.1
Installing minitest 5.4.3
Installing thread_safe 0.3.4
Installing tzinfo 1.2.2
Installing activesupport 4.1.8
Installing awesome_print 1.2.0
Installing builder 3.2.2
Using bundler 1.7.8
Installing diff-lcs 1.2.5
Installing multi_json 1.10.1
Installing gherkin 2.12.2
Installing multi_test 0.1.1
Installing cucumber 1.3.17
Installing calabash-common 0.0.1
Installing edn 1.0.6
Installing geocoder 1.1.9
Installing httpclient 2.3.4.1
Installing retriable 1.3.3.1
Installing thor 0.19.1
Installing run_loop 1.1.0
Installing rack 1.5.2
Installing rack-protection 1.5.3
Installing tilt 1.4.1
Installing sinatra 1.4.5
Installing sim_launcher 0.4.13
Installing slowhandcuke 0.0.3
Installing calabash-cucumber 0.11.4
Installing claide 0.7.0
Installing fuzzy_match 2.0.4
Installing nap 0.8.0
Installing cocoapods-core 0.35.0
Installing cocoapods-downloader 0.8.0
Installing cocoapods-plugins 0.3.2
Installing netrc 0.7.8
Installing cocoapods-trunk 0.4.1
Installing cocoapods-try 0.4.2
Installing colored 1.2
Installing escape 0.0.4
Installing molinillo 0.1.2
Installing open4 1.3.4
Installing xcodeproj 0.20.2
Installing cocoapods 0.35.0
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

第五步，使用 CocoaPod 来安装 WordPross for iOS 的开发依赖

``` console
$ pod install
Analyzing dependencies
Fetching podspec for `EmailChecker` from `https://raw.github.com/wordpress-mobile/EmailChecker/master/ios/EmailChecker.podspec`
Pre-downloading: `KIF` from `https://github.com/SergioEstevao/KIF.git`, commit `f9e3ae3c30447d9efa9dcf2279a77b580157ccbf`
Pre-downloading: `MGImageUtilities` from `git://github.com/wordpress-mobile/MGImageUtilities.git`, commit `e946cf3d97eb95372f4fc4478bf2e0099e73f4b0`
Pre-downloading: `SocketRocket` from `https://github.com/jleandroperez/SocketRocket.git`, commit `04b9c1d135d7791f6d8b2947b3c53ff3f3b1248d`
Pre-downloading: `WordPress-AppbotX` from `https://github.com/wordpress-mobile/appbotx.git`, commit `a0273598d22aac982bec5807e638050b0032a9c9`
Pre-downloading: `WordPress-iOS-Editor` from `git://github.com/wordpress-mobile/WordPress-iOS-Editor`, commit `0912b7ceadd4cecb97a981e15c54475de98d4f8e`
Pre-downloading: `WordPressApi` from `https://github.com/wordpress-mobile/WordPressApi.git`, tag `0.2.0`
Downloading dependencies
Installing AFNetworking (2.3.1)
Installing CTAssetsPickerController (2.7.0)
Installing CocoaLumberjack (1.9.2)
Installing CrashlyticsLumberjack (1.0.2)
Installing DTCoreText (1.6.13)
Installing DTFoundation (1.7.4)
Installing EmailChecker (0.1)
Installing Google-Diff-Match-Patch (0.0.1)
Installing Helpshift (4.8.0)
Installing HockeySDK (3.6.2)
Installing JRSwizzle (1.0)
Installing KIF (3.0.8)
Installing Lookback (0.6.5)
Installing MGImageUtilities (0.0.1)
Installing MRProgress (0.7.0)
Installing Mixpanel (2.5.4)
Installing NSLogger (1.5)
Installing NSLogger-CocoaLumberjack-connector (1.3)
Installing NSObject-SafeExpectations (0.0.2)
Installing NSURL+IDN (0.3)
Installing OCMock (3.1.1)
Installing OHHTTPStubs (1.1.1)
Installing Reachability (3.1.1)
Installing SVProgressHUD (1.0)
Installing Simperium (0.7.4)
Installing SocketRocket (0.3.1-beta2)
Installing UIAlertView+Blocks (0.8.1)
Installing UIDeviceIdentifier (0.4.4)
Installing WordPress-AppbotX (1.0.6)
Installing WordPress-iOS-Editor (0.2.3.1)
Installing WordPress-iOS-Shared (0.1.4)
Installing WordPressApi (0.2.0)
Installing WordPressCom-Analytics-iOS (0.0.13)
Installing WordPressCom-Stats-iOS (0.1.6)
Installing google-plus-ios-sdk (1.7.1)
Installing wpxmlrpc (0.6)
Generating Pods project
Integrating client project
```

第六步，使用 calabash-ios 来自动配置项目，以集成 Calabash

``` console
$ calabash-ios setup WordPress/
Checking if Xcode is running...
Found Project: WordPress

（省略大量的控制台输出）
---------- Info ----------
Found several targets. Please enter name of target to duplicate.
Default target: WordPress. Just hit <Enter> to select default.
UITests
WordPress
WordPressTest
WordPressTodayWidget

input:
Selecting default target (WordPress)
---------- - ----------
Duplicating target: WordPress to new target: WordPress-cal
Adding calabash.framework to target WordPress-cal
Adding Other Linker options to target WordPress-cal

----------Setup done----------
Please validate by running the -cal target
from Xcode.
When starting the iOS Simulator using the
new -cal target, you should see:

  "Started LPHTTP server on port 37265"

in the application log in Xcode.


After validating, you can generate a features folder:
Go to your project (the dir containing the .xcodeproj file).
Then run calabash-ios gen
(if you don't already have a features folder).
---------------------------
```

第七步，用 Xcode 打开 WordPress.xcworkspace，然后手工创建一个名称为 **WordPress-cal** 的 *scheme*，并且 *Target* 是上一步自动创建出来的 **WordPress-cal** 这个 Target。

第八步，创建基本的 Calabash 测试脚本

在项目根目录上执行 `calabash-ios gen`

``` console
$ calabash-ios gen

----------Question----------
I'm about to create a subdirectory called features.
features will contain all your calabash tests.
Please hit return to confirm that's what you want.
---------------------------


----------Info----------
Features subdirectory created.
Make sure you've build your -cal scheme in XCode and
try executing

cucumber
---------------------------
```

第九步、编译并运行 WordPress for iOS。

最简单的编译和运行的方式就是在 Xcode 中打开 WordPress for iOS 的项目，选择 Scheme 为之前刚刚创建的 **WordPress-cal**，然后选择 iPhone 6 模拟器，最后点击『运行』按钮或者按下『⌘R』快捷键。等编译过程结束后，模拟器就会自动打开，并且运行 WordPress for iOS 应用。按下快捷键『⇧⌘C』激活控制台，查看控制台中是否有类似 **Started LPHTTP server on port 37265** 的内容输出，如果看到就表明 Calabash 已经安装并运行成功。比如

``` text
2014-12-18 12:16:05.544 WordPress[26997:1397963] Creating the server: <LPHTTPServer: 0x7fd86d123180>
2014-12-18 12:16:05.545 WordPress[26997:1397963] Calabash iOS server version: CALABASH VERSION: 0.11.4
2014-12-18 12:16:05.546 WordPress[26997:1397963] simroot: /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk
2014-12-18 12:16:05.548 WordPress[26997:1397963] Started LPHTTP server on port 37265
```

第十步，需要对模拟器做一些小调整。

为了统一起见，建议把 iOS 模拟器的语言环境设置为**美国英语**。

另外，需要打开软键盘，否则 Calabash 会无法操作键盘。方法为，打开 WordPress for iOS 客户端后，点击任意的文本框。如果没有看到软键盘出现，就按下 『⌘K』来启用软键盘。再次按下 『⌘K』会关闭软键盘，这里需要启用软键盘。

### 开始使用 Calabash Console

如果之前没有出现任何问题，那现在已经可以在 iOS 模拟器里面运行 WordPress for iOS 客户端了。为了能让 Calabash Console 与应用建立链接，打开 Calabash Console 后运行内置命令 **start_test_server_in_background** 

``` console
$ calabash-ios console
Running irb...
irb(main):001:0> start_test_server_in_background
Calabash::Cucumber::Launcher: Launch Method instruments
Log file: /var/folders/ct/b63y8qts7dz4dmblzzpz28v80000gn/T/run_loop20141224-33581-dif0nk/run_loop.out
```

然后就会看到模拟器自动运行了 WordPress for iOS 应用，并且停留在登录界面。

在前面已经介绍了如何快速的运行一个 WordPress 本地服务，如果已经运行，打开 **Safari** 或者 **Google Chrome** 可以正常访问 [http://wordpress.local](http://wordpress.local)。

因为使用了自建的 WordPress 服务，所以在登录客户端的时候，需要选择添加自托管的主机站点。点击 『Add Self-Hosted Site』

``` console
irb(main):002:0> touch("view marked:'Add Self-Hosted Site'")
[
    [0] {
           "selected" => false,
            "enabled" => true,
               "rect" => {
            "center_x" => 160,
                   "y" => 479,
               "width" => 290,
                   "x" => 15,
            "center_y" => 496,
              "height" => 34
        },
                 "id" => nil,
        "description" => "<WPNUXSecondaryButton: 0x7c293280; baseClass = UIButton; frame = (15 479; 290 34); opaque = NO; autoresize = LM+RM+TM; layer = <CALayer: 0x7c2934a0>>",
              "label" => "Add Self-Hosted Site",
              "alpha" => 1,
              "class" => "WPNUXSecondaryButton",
              "frame" => {
                 "y" => 479,
             "width" => 290,
                 "x" => 15,
            "height" => 34
        }
    }
]
```

点击『Username / Email』 文本框

``` console
irb(main):003:0> touch("view marked:'Username / Email'")
[
    [0] {
               "text" => "Username / Email",
            "enabled" => true,
               "rect" => {
            "center_x" => 174,
                   "y" => 197,
               "width" => 272,
                   "x" => 38,
            "center_y" => 212,
              "height" => 30
        },
                 "id" => nil,
        "description" => "<UITextFieldLabel: 0x7a99cd10; frame = (38 7; 272 30); text = 'Username / Email'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x7a9af0e0>>",
              "label" => "Username / Email",
              "alpha" => 1,
              "class" => "UITextFieldLabel",
              "frame" => {
                 "y" => 7,
             "width" => 272,
                 "x" => 38,
            "height" => 30
        }
    }
]
```

输入用户名『odd-e』

``` console
irb(main):004:0> keyboard_enter_text "odd-e"
nil
```

点击『Password』 文本框

``` console
irb(main):005:0> touch("view marked:'Password'")
[
    [0] {
               "text" => "Password",
            "enabled" => true,
               "rect" => {
            "center_x" => 154,
                   "y" => 200,
               "width" => 232,
                   "x" => 38,
            "center_y" => 215,
              "height" => 30
        },
                 "id" => nil,
        "description" => "<UITextFieldLabel: 0x7a9b4a30; frame = (38 7; 232 30); text = 'Password'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x7a9b4af0>>",
              "label" => "Password",
              "alpha" => 1,
              "class" => "UITextFieldLabel",
              "frame" => {
                 "y" => 7,
             "width" => 232,
                 "x" => 38,
            "height" => 30
        }
    }
]
```

输入密码『 s3cr3t』

``` console
irb(main):006:0> keyboard_enter_text "s3cr3t"
nil
```

点击『Site Address (URL)』文本框

``` console
irb(main):007:0> touch("view marked:'Site Address (URL)'")
[
    [0] {
               "text" => "Site Address (URL)",
            "enabled" => true,
               "rect" => {
            "center_x" => 174,
                   "y" => 244,
               "width" => 272,
                   "x" => 38,
            "center_y" => 259,
              "height" => 30
        },
                 "id" => nil,
        "description" => "<UITextFieldLabel: 0x7a97e2f0; frame = (38 7; 272 30); text = 'Site Address (URL)'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x7a9757b0>>",
              "label" => "Site Address (URL)",
              "alpha" => 1,
              "class" => "UITextFieldLabel",
              "frame" => {
                 "y" => 7,
             "width" => 272,
                 "x" => 38,
            "height" => 30
        }
    }
]
```

输入主机地址『http://wordpress.local』

``` console
irb(main):008:0> keyboard_enter_text "http://wordpress.local"
nil
```

点击『Add Site』按钮

``` console
irb(main):009:0> touch("view marked:'Add Site'")
[
    [0] {
           "selected" => false,
            "enabled" => true,
               "rect" => {
            "center_x" => 160,
                   "y" => 296,
               "width" => 290,
                   "x" => 15,
            "center_y" => 316.5,
              "height" => 41
        },
                 "id" => nil,
        "description" => "<WPNUXMainButton: 0x7a97d240; baseClass = UIButton; frame = (15 296; 290 41); opaque = NO; autoresize = LM+RM; layer = <CALayer: 0x7a9ae1d0>>",
              "label" => "Add Site",
              "alpha" => 1,
              "class" => "WPNUXMainButton",
              "frame" => {
                 "y" => 296,
             "width" => 290,
                 "x" => 15,
            "height" => 41
        }
    }
]
```

到此为止就完成了自定义主机的登录过程，下面再回顾一下刚才输入的所有操作 **WordPress for iOS** 的命令。

 * touch("view marked:'Add Self-Hosted Site'")
 * touch("view marked:'Username / Email'")
 * keyboard_enter_text "odd-e"
 * touch("view marked:'Password'")
 * keyboard_enter_text "s3cr3t"
 * touch("view marked:'Site Address (URL)'")
 * keyboard_enter_text "http://wordpress.local"
 * touch("view marked:'Add Site'")
 
这些命令的序列与我们直接手工操作应用是一致的，如果是手工操作，步骤依次为：

 * 点击『Add Self-Hosted Site』
 * 点击『Username / Email』
 * 输入文本『odd-e』
 * 点击『Password』
 * 输入文本『s3cr3t』
 * 点击『Site Address (URL)』
 * 输入文本『http://wordpress.local』
 * 点击『Add Site』

下一步，就要考虑如何让上述的过程自动化的完成。

### 让登录的过程自动化

因为 WordPress for iOS 会保留登录信息，在登录成功后，再次打开应用会跳过登录界面。所以目前我们需要在每次运行登录的步骤之前，都需要先暂时手工注销当前的登录。
