# Building Your First Web App in Swift Using Vapor
## 使用Vapor构建你的第一个Web应用

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

## Downloading the Swift 3 Snapshot

Now that Swiftenv has been installed, you must download the newest Swift 3 Snapshot. A Swift Snapshot is the Swift language in a package ready to be installed. Run the following command:

```bash
swiftenv install 3.0-GM-CANDIDATE
```

> Update: If 3.0-GM-CANDIDATE doesn’t work for you, please use DEVELOPMENT-SNAPSHOT-2016-09-06-a instead.

It will take a while to download the snapshot. Once the download is complete, set the snapshot as the main Swift version on the terminal. This will make this particular version global. Run the following command:

```bash
swiftenv global 3.0-GM-CANDIDATE
```

Check to see that 3.0-GM-CANDIDATE is being used by running the following command:

```bash
swiftenv versions
```

You should see an asterisk next to 3.0-GM-CANDIDATE.

## Compatibility Checking

Vapor needs to have the suitable Swift version to be able to run correctly. Vapor offers a shell script to check that for you. Run the following command to verify that everything is ok:

```bash
curl -sL check.vapor.sh | bash
```

If you get the green checkmark emoji, that means that you are set to install Vapor!

![](http://www.appcoda.com/wp-content/uploads/2016/09/s10-1024x598.png)

## Installing the Vapor Toolbox

Let’s download and install Vapor by running the following command:

```bash
curl -sL toolbox.vapor.sh | bash
```

Ok, this might take a while. You might encounter an error, e.g.: "Install failed, trying sudo". As long as you finally get the "Vapor Toolbox v0.10.4 Installed" message, you are all set up.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s11-1024x601.png)

To verify that Vapor is working, simply enter the following in your terminal:

```bash
vapor
```

You should encounter a list of commands like the following:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s12-1024x588.png)

If you get this result, congratulations! You have completed the most challenging part of this tutorial.


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
