# Building Your First Web App in Swift Using Vapor
## 使用Vapor构建你的第一个Swift Web应用

***

>* 原文链接 : [Building Your First Web App in Swift Using Vapor](http://www.appcoda.com/server-side-swift-vapor/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : [yrq110](https://github.com/yrq110)

***

在WWDC 2015上苹果宣布swift将会开源，很快，在2015年的12月Swift的代码库就在[GitHub](https://github.com/apple/swift)上公开了。

Swift的开源给了开发者更多的机会与可能性，扩展Swift生态圈的应用。

如所期待的那样，开发者们很快就探索出了Swift的很多其他方面的应用。这篇教程中所介绍的web app就是其中的一种应用。

## 为何关注?

在服务端使用一种强类型的语言是有很多益处的。若要为你的iOS app开发后端功能的话，使用同一种语言与风格比较容易保持一致性。

这里有三种很不错的Swift服务端：Perfect, Kitura与Vapor。这篇教程中会使用Vapor来探索在Swift服务端上的神奇Swift世界。

我会指导你如何安装Vapor与Swift3，学习服务端Swift的基础知识，然后在Heroku上部署网站/后端。

## 前提

这篇教程中会使用很多bash命令，因此需要掌握一些基本的bash与终端知识是很重要的。Vapor是基于Swift3的，因此也需要Xcode8。

Vapor与其他Swift工程(Swift包管理器)都是基于Swift3的，建议你通过这篇[文章](https://github.com/yrq110/Some_IOS_Tutorials_With_Swift/blob/master/What%E2%80%99s%20New%20in%20Swift%203.md)了解下Swift3中的一些变化。

在教程的最后，会教你如何在Heroku上部署Vapor服务端，Heroku是个很火的云服务供应商，推荐尝试下。

开搞吧!

## 安装Vapor

Vapor需要Swift 3与Xcode8，可以在[这里](https://developer.apple.com/download/)下载GM版Xcode。 and the latest version of Xcode 8 beta. You can download the new Xcode GM here.

首先需要选择最新的命令行工具，打开Xcode的preferences(偏好设置)。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s1.png)

接着导航栏中的Locations。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s3-1024x653.png)

最后，选择最新的Xcode8命令行工具集。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s4-1024x666.png)

## 安装Swiftenv

是时候安装Swift 3了。首先先来安Swiftenv。Swiftenv使Swift的安装变得很轻松，并且可以在多个版本间切换。通过clone Github上的仓库来安装Swiftenv，在终端里运行如下代码:

```bash
git clone https://github.com/kylef/swiftenv.git ~/.swiftenv
```

然后在bash文件中初始化Swiftenv，bash文件就是设置bash属性与设置的地方。在终端里使用如下指令打开bash文件:

```bash
open ~/.bash_profile
```

open指令会使用TextEdit打开你的bash文件，若没有bash文件，使用如下指令创建它:

```bash
touch ~/.bash_profile
```

打开bash文件后，复制如下指令，用来初始化Swiftenv。

```bash
export SWIFTENV_ROOT="$HOME/.swiftenv"
export PATH="$SWIFTENV_ROOT/bin:$PATH"
eval "$(swiftenv init -)"
```

你的bash文件(在TextEdit中打开)中现在应该包含了这些指令。注意有些情况下bash文件可能包含其他指令。不要删掉之前的指令，在文件末端添加新的即可。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s6-1024x682.png)

保存bash文件并重启终端。终端重启后，输入下面这个指令确保Swiftenv已经成功安装了:

```bash
swiftenv --version
```

在终端中应如下所示:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s7-1024x643.png)

## 下载Swift 3快照

安好Swiftenv了，接着来下载最新的Swift 3快照(Snapshot)，Swift快照就是个打包好用于安装的Swift。执行如下命令:

```bash
swiftenv install 3.0-GM-CANDIDATE
```

> 更新: 若3.0-GM-CANDIDATE不好使，请使用DEVELOPMENT-SNAPSHOT-2016-09-06-a。

会花点时间下载，下载完成后在终端里将快照版本设置为主Swift版本，这将会是全局的版本，执行如下命令:

```bash
swiftenv global 3.0-GM-CANDIDATE
```

检查下版本是否是3.0-GM-CANDIDATE:

```bash
swiftenv versions
```

应该会看到3.0-GM-CANDIDATE后面有个星号。

## 完整性检验

Vapor需要合适的Swift版本才能正常启动，Vapor提供了一个shell脚本来用于检验完整性，执行如下命令来确保万事俱备:

```bash
curl -sL check.vapor.sh | bash
```

若显示出绿色箭头的emoji表情，就表示准备好安装Vapor了!

![](http://www.appcoda.com/wp-content/uploads/2016/09/s10-1024x598.png)

## 安装Vapor Toolbox

使用如下命令下载并安装Vapor:

```bash
curl -sL toolbox.vapor.sh | bash
```

好吧，这会花点时间，也许会出错，比如"Install failed, trying sudo"。当看到"Vapor Toolbox v0.10.4 Installed"信息时就说明安装成功了。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s11-1024x601.png)

在终端里试试如下命令，来验证下Vapor是否准备就绪:

```bash
vapor
```

会看到如下一个命令列表:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s12-1024x588.png)

恭喜! 你完成了教程中最具挑战的部分(PS:???)。


# Starting a new Vapor Project

Every Swift project uses the Swift Package Manager (SPM). Every SPM project requires a Package.json, a main.swift file, etc. The Swift Package Manager will look for the Package.swift file in your Swift project to download the necessary dependencies. SPM is similar to NPM (Node Package Manager). Vapor creates the Swift project, and adds the required dependencies to the Package.swift file.

To create a new project, you must provide Vapor with a project name. First, navigate to a directory where you would want to create the new project, then run the following command:

```bash
vapor new HelloWorld
```

![](http://www.appcoda.com/wp-content/uploads/2016/09/s13-1024x666.png)

Vapor will create a project called HelloWorld in your current directory. Navigate to your new project directory, which is the same as the name of the project. In this case, run the following:

```bash
cd HelloWorld
```

You should see a directory similar to the following:

```bash
├── Package.swift
├── App
│   ├── main.swift
│   ├── Controllers
│   ├── Middleware
│   └── Models
├── Resources
 |    └── Views
├── Config
├── Localization
└── Public
```

Vapor follows and enforces the MVC (Model, View, Controller) pattern. It creates the folders where Models, Views, and Controllers are located in. If you are not familiar with the MVC design pattern, click here to learn more.

First, Vapor creates the Package.swift file. There is no need to modify it. Then there is the App folder, which contains the Models, Controllers, Middleware, and the main.swift file. The main.swift file is our main app file, and where we initialize our server. Vapor will run this file first.

The Resources folder contains the Views folder, where our HTML files and templates are stored. The Public folder is where our images and styles must go. We won’t be working with the rest of the folders for now.
