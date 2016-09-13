#CloudKit Tutorial: Getting Started
##CloudKit指南：入门


***

>* 原文链接 : [CloudKit Tutorial: Getting Started](https://www.raywenderlich.com/134694/cloudkit-tutorial-getting-started)
* 原文作者 : [Ed Sasena](https://www.raywenderlich.com/u/anros)
* 译者 : [yrq110](https://github.com/yrq110)

***

> 更新: 此教程已更新至 Xcode 7.3, iOS 9.3, Swift 2.2。原教程由Michael Katz所著。

CloudKit是苹果基于iCloud的远程数据存储服务。提供了一个低开销的存储与分享app数据的方案，使用用户的iCloud账户进行后端存储。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/CloudKit-feature-1-250x250.png)

学习如何使用CloudKit在云端存储你的app数据!

CloudKit中的两个主要部分:

1. 一个管理记录类型与其它公共数据的web控制台。
2. 一个在iCloud与设备之间传输数据的API集合。

CloudKit同样很安全，用户的私有数据被完全保护了起来，开发者只能访问他们自己的私有数据库而不能查看用户的私有数据。

对于一个仅有iOS平台并且使用大量数据的app来说，CloudKit是一个好的选择，无需编写服务端的一些繁琐的逻辑。此外，CloudKit也可以应用于web与服务器应用。 

在这篇CloudKit教程中，会使用CloudKit创建一个餐厅评分app来进行实践，它有一个拗口的名字-BabiFüd.

> 注意: 进行这篇CloudKit教程中的示例app开发时，需要一个激活的iOS开发者账号，若没有的话将无法使用iCloud entitlements与访问CloudKit控制台。

## 为何使用CloudKit?

You might wonder why you should choose CloudKit over Core Data, other commercial BaaS (Back end as a Service) offerings, or even rolling your own server.
There are three reasons: simplicity, trust, and cost.

### Simplicity

Unlike other backend solutions, CloudKit requires little setup. You don’t have to choose, configure, or install servers. Security and scaling are handled by Apple as well.

Simply registering for the iOS Developer Program makes you eligible to use CloudKit. You don’t have to register for additional services or create new accounts. When you enable CloudKit capabilities in your app all necessary server setup magic happens automatically.

There’s no need to download additional libraries and configure them. CloudKit is imported like any other iOS framework. The CloudKit framework itself also provides a level of simplicity by offering convenient APIs for common operations.

It’s also easy for users. Since CloudKit uses the iCloud credentials entered when the device is set up (or entered after set up via the Settings app), there’s no need to build complicated login screens. As long as they are logged in, users can seamlessly start using your app. That should put you on Cloud 9!

### Trust

Another benefit of CloudKit is that users can trust the privacy and security of their data by relying on Apple rather than app developers. CloudKit insulates user data from you, the developer.

While this lack of access can be frustrating while debugging, it is a net plus since you don’t have to worry about security or convince users their data is secure. If an app user trusts iCloud, then they can also trust you.

### Cost

Finally, for any developer, the cost of running a service is a huge deal. Even the least expensive server hosts cannot offer low-cost solutions for small, free or cheap apps. So there will always be a cost associated with running an app.

With CloudKit, you get a reasonable amount of storage and data transfer for public data for free. There is a very thorough explanation of fees in the What’s New in CloudKit WWDC 2015 Video.

These strengths make the CloudKit service a low-hassle solution for Mac and iOS apps.

## Introducing BabiFüd

The sample app for this tutorial, BabiFüd, is the freshest take on the standard “rate a restaurant” style of app. Instead of reviewing restaurants based upon food quality, speed of service, or price, users rate child-friendliness. This includes availability of changing facilities, booster seats, and healthy food options.

The app contains four tabs: a list of nearby restaurants, a map of nearby restaurants, user-generated notes, and settings. The list of nearby restaurants is the only tab you’ll use in this tutorial. You can get a glimpse of the app in action below.

![](http://www.raywenderlich.com/wp-content/uploads/2014/09/nearby.png)

![](http://www.raywenderlich.com/wp-content/uploads/2014/09/map.png)

A model class backs these views and wraps the calls to CloudKit. CloudKit objects are called records. The main record type in your model is an Establishment, which represents the various restaurants in your app.
