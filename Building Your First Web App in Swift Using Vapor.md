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

Vapor requires Swift 3, and the latest version of Xcode 8 beta. You can download the new Xcode GM here.

First, you have to choose the latest command line tools. Open you Xcode preferences.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s1.png)

Next, navigate to Locations.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s3-1024x653.png)

Lastly, choose the latest Xcode 8 command line toolset.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s4-1024x666.png)

