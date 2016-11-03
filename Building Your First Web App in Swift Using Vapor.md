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

## Installing Swiftenv

Now it’s time to install Swift 3. First, install Swiftenv. Swiftenv allows you to easily install and switch between multiple versions of Swift. To install Swiftenv, clone the respective repository from Github by running the following code on your terminal:

```bash
git clone https://github.com/kylef/swiftenv.git ~/.swiftenv
```

Then, initialize Swiftenv in your bash profile. The bash profile is where your bash properties and settings reside. Open your bash profile by running the following command in terminal:

```bash
open ~/.bash_profile
```

The open command will open your bash profile in TextEdit. If you don’t have a bash profile, create it with the following command:

```bash
touch ~/.bash_profile
```

When you see your bash profile, paste the following commands. These commands initialize Swiftenv.

```bash
export SWIFTENV_ROOT="$HOME/.swiftenv"
export PATH="$SWIFTENV_ROOT/bin:$PATH"
eval "$(swiftenv init -)"
```

Your bash profile (opened in TextEdit) should now contain these commands. Note that in some cases your bash profile might contain other commands. Do not remove the previously placed commands, just append the new ones at the end of the file.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s6-1024x682.png)

Save the bash profile, and restart terminal. After terminal restarts, type in the following command to verify that Swiftenv was installed successfully:

```bash
swiftenv --version
```

You should see the following result in your terminal:

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
