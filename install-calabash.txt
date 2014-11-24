# 安装 Calabash
## 快速安装
Calabash 是使用 Ruby 开发的，所以可以使用 `gem` 来直接安装，前提是我们已经安装好 Ruby。

安装 Calabash for Android

    gem install calabash-android

安装 Calabash for iOS

    gem install calabash-cucumber

## 详细安装
我们推荐如下的安装过程，因为这不仅会大大简化我们的 Calabash 安装过程，而且还能统一不同开发人员的 Ruby 依赖环境以及 Calabash 的版本，也方便了去试用和升级新的 Calabash。

在本节内容中，将会涉及 Windows、Mac OS X、Linux 三个常用环境下的依赖安装，以及如何隔离每个独立的 Android/iOS 项目依赖的 Calabash 版本。

**非 Windows 用户请注意：在下面的安装过程中，如果没有特别指定，请不要使用 root 用户或者 `sudo` 命令来执行任何命令。这个很重要！首先会简化我们的安装及使用过程，其次会避免因此而出现的各种奇怪问题。**

## Windows 环境的依赖
TODO

## Mac OS X 环境的依赖
Mac OS X 环境下建议安装 Homebrew。

### Homebrew 介绍
[Homebrew](http://brew.sh) 是一个 Mac OS X 下的包管理器，类似于 Ubuntu 上的 apt-get。使用 Homebrew 可以方便我们来安装各种依赖的工具和类库，比如 rvm。并且 rvm 在使用过程中也会依赖 Homebrew 来安装各种依赖类库。

安装方法很简单，在官网首页的最下面就有说明。使用我们的默认用户，打开 *Terminal.app* 然后执行如下的命令即可：

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

这个标准的安装脚本会把 Homebrew 安装到 */usr/local* 目录中，然后当你使用 `brew install` 的时候就不再需要 `sudo` 这个命令了。

#### 修复之前使用 root 权限安装过的 Homebrew
如果你在之前已经使用 root 用户或者 `sudo` 命令安装过 Homebrew，最简单的解决办法就是把 Homebrew 的安装目录的所属用户修改为你正在使用的默认用户。执行如下命令：

    sudo chown -R $USER $(brew --prefix)

## Linux 环境的依赖
对于常用的 Linux 发行版来说，几乎没有什么依赖需要安装。即使有，也会在后面的过程中自动安装。

## 为 Calabash 配置项目依赖
统一开发环境对于团队来说是一件非常重要的事情，可以避免程序员最经典的问题『在我这里没问题啊』。

对于在项目中使用 Calabash 来说，首先要统一 Ruby 的环境，其次要统一 Calabash 的版本。Calabash 一直在持续的开发中，不同版本的 API 可能也会发生变化，这会导致自动化测试脚本依赖于特定版本的 Calabash。

要做到这一点，我们会使用 RVM 来统一 Ruby 环境，并且使用 Gemfile 来统一 Calabash 的版本及其依赖。

### RVM（Ruby Version Manager） 介绍
[Ruby Version Manager](http://rvm.io) 是一个命令行工具，可以简化我们安装、管理以及为不同的项目使用不同 Ruby 环境的问题。虽然 几乎所有的 Linux 发行版以及Mac OS X 都已经自带 Ruby 环境，但是我们不推荐直接升级系统的 Ruby，这可能会导致某些个别依赖于系统默认 Ruby 环境的应用出现问题，所以还是建议使用 RVM。

RVM 支持安装在大多数的类 UNIX 环境中，最基本的依赖是 *bash*、*curl* 和所有的 GNU 版本的工具。在 RVM 的安装过程中，会检查需要的依赖并自动安装。

**注意：RVM 不支持 Windows 环境**

#### 安装 RVM
安装方法在 [RVM 官网](http://rvm.io)首页就可以看到，执行如下命令：

    \curl -sSL https://get.rvm.io | bash -s stable

> `\curl` 的意思是不使用 Shell 环境的别名，而是直接在系统的 PATH  环境变量里的所有路径中寻找 `curl` 这个可执行的命令，这样可以防止我们为了方便使用 `curl` 而定义的同名的别名导致安装过程出现问题。
>
> `\curl` 等同于执行 `command curl`

安装完毕 RVM 以后，执行如下命令来加载 RVM 环境，或者请重新打开一个新的终端窗口。

    source ~/.rvm/scripts/rvm

#### 使用 RVM 安装 Ruby
使用 RVM 安装最新版的 Ruby 是一件非常容易的事情，仅仅需要执行如下的命令：

    rvm install ruby

我们还可以安装指定版本的 Ruby，比如要安装 1.8.7 这个版本，就仅仅把版本号加在后面，比如：

    rvm install ruby-1.8.7
    
甚至还可以安装其他的 Ruby 实现，比如 JRuby

    rvm install jruby

不过，对于我们使用 Calabash，安装好最新版就可以了。截止到写下这些文字的时候，最新的 Ruby 版本是 2.1.5。

查看当前 RVM 环境中已经安装了哪些 Ruby，就使用命令

    rvm list

查看当前 RVM 的状态

    rvm current

切换不同的 Ruby 环境，比如切换到最新版的 Ruby

    rvm use ruby

切换到 Ruby 1.8.7

    rvm use ruby-1.8.7
    
或者仅仅使用版本号
    
    rvm use 1.8.7

#### 使用 RVM 来隔离不同的 Ruby 环境
RVM 可以帮我们隔离不同的 Ruby 环境，不止是不同的 Ruby 版本，也包括同样版本的 Ruby 在不同的使用场合下可以有不同的 gems。比如，我们在使用 Calabash 测试 Android 项目时，就有 `calabash-android` 这个 gem，而没有 `calabash-cucumber`。在测试 iOS 项目时，就有 `calabash-cucumber` 而没有 `calabash-android`。

列出当前的所有 Gemset

    rvm gemset list

切换 Gemset

    rvm use calabash-android

切换到 Ruby 2.1.5 下的 Gemset *calabash-android*

    rvm use 2.1.5@calabash-android

在上一条命令中，如果对应的 Gemset 不存在，RVM 会出错并提示响应的创建命令。

在当前使用的 Ruby 版本下创建一个 Gemset

    rvm gemset create calabash-android

注意，不同的 Ruby 版本下是可以有同名的 Gemset。比如 1.9.3@calabash-android 和 2.1.5@calabash-android 两个是不同的 Gemset。

一个有用的技巧是，在切换到一个特定的 Gemset 时使用选项 `--create`，如果对应的 Gemset 不存在就创建

    rvm use 2.1.5@calabash-android --create

RVM 有一个非常有用的特性，通过读取项目目录中的特定配置文件来根据不同的项目自动切换 Ruby 环境及 Gemset。RVM 支持多种配置文件的类型，按照优先级别分别为：
1. `.rvmrc`
这是一个脚本文件，允许我们完整的定制所需的 Ruby 环境，这个配置文件也是 RVM 默认的配置文件，所有的 RVM 版本都支持。

2. `.versions.conf`
一个 key=value 配置文件。

3. `.ruby-version`
仅仅有一个单行的 Ruby 版本。同时还会有一个 `.ruby-gemset` 文件，也是仅仅有一行的 Gemset 的名字。

4. `Gemfile`
支持读取 Gemfile 的注释: #ruby=1.9.3 或指令: ruby "1.9.3"。

如果项目目录中有对应 RVM 配置文件，只需要切换到该目录，RVM 就会自动切换到配置文件中指定的 Ruby 版本和 Gemset。

#### RVM 常用命令
升级 RVM 到最新的稳定版

    rvm get stable

当升级 RVM 后，需要重新加载 RVM 环境

    rvm reload

查看当前正在使用 Ruby 版本以及 Gemset 的名字

    rvm current
    
修改过项目的 RVM 配置文件后，RVM 不会立即切换到新的 Ruby 环境，需要重新加载一下

    rvm rvmrc reload

切换到指定的 Ruby，比如 JRuby，如果没有指定版本就会使用本地安装的最高版本

    rvm use jruby

或者同时也指定版本号
    
    rvm use ruby-1.8.7

如果只指定版本号，默认 Ruby 的实现是 CRuby

    rvm use 2.1.5
    
希望指定一个全局默认的 Ruby 环境，就使用 `--default` 选项

    rvm use 2.1.5 --default
    
### 为项目配置 RVM 的支持
尽管我们可以任选 RVM 支持几种配置方式中的任何一个，目前我们会采用 `.ruby-version` 和 `.ruby-gemset` 这种方式。原因一是非常简单，二是还支持其他的 Ruby 版本管理工具，三是在默认的 RVM 配置下，使用时不会有任何的警告信息出现。

假设下面我们要使用 *2.1.5* 的 Ruby，并且 Gemset 的名字是 *calabash-android*，切换到项目的根目录下：

    echo "2.1.5" > .ruby-version
    echo "calabash-android" > .ruby-gemset
    rvm rvmrc load

**记住，需要把新增的这两个文件 `.ruby-version` 和 `.ruby-gemset` 都要添加到项目的版本管理中。**

### 为项目安装 Calabash
如果给 Android 项目添加 Calabash for Andoir，那就在根目录上添加一个 `Gemfile`，内容为

> gem 'calabash-android'

如果给 iOS 项目添加 Calabash for iOS，那就在项目根目录上添加一个 `Gemfile`，内容为

> gem 'calabash-cucumber'

Gemfile 文件创建好以后，在项目的根目录上执行

    bundle install
    
这会自动安装好最新版的 Calabash，同时也会生成一个名为 `Gemfile.lock` 的文件。

**记住，同样需要把 `Gemfile` 和 `Gemfile.locak` 这两个文件添加到项目的版本管理中。**

#### 为 iOS 项目增加 Calabash 的依赖
首先下载 `calabash.framework`

    calabash-ios download

配置的最简单快速的办法就是使用下面的命令，但是这个命令有可能会搞坏项目配置

    calabash-ios setup ./

##### 手工给 iOS 项目增加 Calabash 的依赖
TODO

## Calabash 安装总结
因为 Calabash 使用 Ruby 开发的，所以需要安装好 Ruby。为了在开发团队中统一所有人员的开发环境，包括每个人使用的 Ruby 版本都要一致。就需要一个 Ruby 版本管理工具，这里使用的是当前流行的 RVM。

为了统一 Calabash 的版本，使用了 Gemfile 及 Gemfile.lock 文件。Gemfile 里配置了项目使用的 Calabash，可能还包括特定的版本。Gemfile.lock 是 bundle 自动根据 Gemfile 的配置生成的，里面记录了 Calabash 依赖的所有相关 gem。

在安装 Calabash 的过程中，可能还需要编译一些二进制文件，所以对于 Windows 来说要安装 Ruby 的开发环境及其 SDK。对于 Mac OS X 来说，建议安装好 Homebrew，RVM 会自动使用 Homebrew 来安装需要的依赖。对于常见的 Linux 发行版来说，RVM 会自动使用对应的包管理器来安装各种依赖。

当安装好 Calabash 后，对于 iOS 项目来说，还需要为项目添加 `calabash.framework` 以及特定的编译目标。