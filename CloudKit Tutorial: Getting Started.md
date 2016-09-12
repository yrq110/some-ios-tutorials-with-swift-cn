#CloudKit Tutorial: Getting Started
##CloudKit指南：入门


***

>* 原文链接 : [CloudKit Tutorial: Getting Started](https://www.raywenderlich.com/134694/cloudkit-tutorial-getting-started)
* 原文作者 : [Ed Sasena](https://www.raywenderlich.com/u/anros)
* 译者 : [yrq110](https://github.com/yrq110)

***

> 更新: 此教程已更新至 Xcode 7.3, iOS 9.3, Swift 2.2。原教程由Michael Katz所著。

CloudKit是苹果基于iCloud的远程数据存储服务，使用用户的iCloud账户作为后端存储，提供一个低开销的存储与分享app数据的方案。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/CloudKit-feature-1-250x250.png)

学习如何使用CloudKit在云端存储你的app数据!

CloudKit中的两个主要部分:

1. 一个管理记录类型与其它公共数据的web控制台。
2. 一个在iCloud与设备之间传输数据的API集合。

CloudKit同样很安全，用户的私有数据被完全保护了起来，开发者只能访问他们自己的私有数据库而不能查看用户的私有数据。

对于一个仅有iOS平台并且使用大量数据的app来说，CloudKit是一个好的选择，无需编写服务端的一些繁琐的逻辑。此外，CloudKit也可以应用于web与服务器应用。 

在这篇CloudKit教程中，会使用CloudKit创建一个餐厅评分app来进行实践，它有一个拗口的名字-BabiFüd.

> 注意: 进行这篇CloudKit教程中的示例app开发时，需要一个激活的iOS开发者账号，若没有的话将无法使用iCloud entitlements与访问CloudKit控制台。
