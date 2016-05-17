#OAuth 2.0 with Swift Tutorial
##基于Swift的OAuth 2.0使用指南
[原文地址](https://www.raywenderlich.com/99431/oauth-2-with-swift-tutorial)

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/05/oauth2-epalined-4b-250x250.png)

在你将内容分享到喜爱的社交网络(Facebook, Twitter等)上或者你企业的OAuth2服务器上时，很可能不经意使用了OAuth2与其他种类的协议，你甚至不知道内部的原理。
不过你知道如何使用OAuth2连接你的服务与iOS app吗？

在这篇教程中，你会使用一个名为Incognito的分享型app来学习如何使用[AeroGear OAuth2](https://github.com/aerogear/aerogear-ios-oauth2), [AFOAuth2Manager](https://github.com/AFNetworking/AFOAuth2Manager)和[OAuthSwift](https://github.com/dongri/OAuthSwift)这些开源OAuth2库来将你的资源分享到GoogleDrive上。
##入门

[下载Incognito开始工程](http://www.raywenderlich.com/wp-content/uploads/2015/04/Incognito.aerogear_start.zip)。开始工程使用CocoaPods来安装AeroGear与其他你需要的一切，包括生成pods与xcworkspace目录。

在Xcode中打开Incognito.xcworkspace。工程基于Xcode的单视图应用模板，有包含单一ViewController.swift视图控制器的故事版，所有UI与动作在ViewController.swift中都已经处理好了。
Build and run your project to see what the app looks like:
构建并运行你的工程会看到下图这样：

![](http://www.raywenderlich.com/wp-content/uploads/2015/05/OAuth_App.png)
