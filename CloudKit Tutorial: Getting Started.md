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

也许你会疑惑为何不选Core Data或其他商业型BaaS服务，甚至是自己的服务器。

这里有三个原因: 简便, 信任, 与开销。

### 简便

不像其它的后端解决方案，CloudKit仅需要少量的配置即可使用。无需进行服务器的选择、配置与安装，并且安全问题也是由苹果来处理。

进行简单的iOS开发者注册后即可使用CloudKit，不需要注册额外的服务或创建新账号，当在app中启用CloudKit功能时所有必要的服务器都会神奇的自动设置完成。

无需下载额外的库与配置。CloudKit与其它iOS框架同等重要，CloudKit框架本身也提供了一些针对于通用操作的方便的API。

对用户来说很简单，CloudKit在设备启动好时通过iCloud证书来登入(或者在启动后的设置界面登入)，无需构建复杂的登录界面。只要登入成功后，就可以直接使用你的app了。That should put you on Cloud 9(PS: 也许指的是北美的那支游戏战队)!

### 信任

另一个CloudKit的好处就是用户可以放心它们数据的隐私性与安全性，这些数据都是由苹果处理的而不是app开发者。CloudKit将用户数据与开发者隔离开来。

虽然这个访问权限的缺失在debug过程中也许会使人沮丧，不过你不用担心安全性并且思考如何使用户相信他们的数据是安全的。若用户信任iCloud的话他们也会信任你。

### 开销

最后，对开发者而言，运行一个服务总会产生一笔不小的开销，即使是价格最便宜的服务器主机也很难有为小型免费的app提供的低开销解决方案。因此在运行一个app时难免会产生一个相关的开销。

使用CloudKit的话会得到一个免费的用于公共数据传输的容量适当的存储空间。在WWDC 2015视频中的[What’s New in CloudKit](https://developer.apple.com/videos/play/wwdc2015/704/)详细讲解了它的费用。

强有力的CloudKit能为Mac与iOS app提供一个麻烦少的解决方案。

## BabiFüd简介

BabiFüd是这篇教程中的示例app，一个标准的“餐厅评价”类app。然而不通过食物质量、服务速度与价格来进行评估，而是通过针对儿童的友好度，这就包含了可更换的设施，安全座椅与可选择的健康食品选择。

app中包含4个标签：一个附近餐厅的列表，一张附近餐厅的地图，用户评价与设置。附近餐厅的列表是唯一会在教程中用到的，看看下面的app截图

![](http://www.raywenderlich.com/wp-content/uploads/2014/09/nearby.png)
![](http://www.raywenderlich.com/wp-content/uploads/2014/09/map.png)

这些视图后有一个模型类，与调用CloudKit的操作绑定。CloudKit对象被称为records，模型中主要的record类型是**Establishment**，代表app中不同的餐厅。

## 开始

先下载这篇教程的[开始工程](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/BabiFud-Cloudkit-Starter.zip)。

在开始编码前需要修改下Bundle Identifier和app的Team属性，需要设置team来从Apple那里获权，给予一个独一无二的bundle identifier会省不少事儿。

在Xcode中打开BabiFud.xcodeproj，工程导航栏中选择BabiFud工程，接着选择BabiFud target，选中General标签，修改Bundle Identifier，一个经典的做法是反转标记的区域名称再加上工程名称，接着选择适合的Team:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S1-BabiFud-Bundle-ID-480x138.png)

好了，现在需要设置CloudKit，创建一些存放数据的容器(container)。

## 授权与容器

通过app添加数据前需要一个存放app记录(record)的容器。容器是存放在服务器的保存所有app数据的一个概念性位置，分为公共数据库与私有数据库。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/CloudKit-container-diagram.png)

为了创建一个容器，首先需要启用app的iCloud功能，选择target中的Capabilities标签，将iCloud区域的开关拨到ON。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S2-Enable-iCloud-480x220.png)

这时，Xcode可能会让你输入与iOS开发者账号关联的Apple ID，若有的话就输入下。最后，在Services分组里勾选CloudKit复选框来启用CloudKit。

这样就创建了一个默认的名为iCloud.<你的bundle id>的容器，如下所示:

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S3-Enable-CloudKit-480x220.png)

### Troubleshooting iCloud Setup in Xcode

If you see any warnings or errors when creating entitlements, building the project, or running the app, and Xcode is complaining about the container ID, then here are some troubleshooting tips:

* If there are any warnings or errors shown in the Steps group in the iCloud section, then try pressing the Fix Issue button. This might need to be done a few times.
  ![](http://www.raywenderlich.com/wp-content/uploads/2014/09/0_fix_issue.png)
* It’s important that the app’s bundle id and iCloud containers match and exist in the developer account. For example, if the bundle identifier is “com.<your domain>.BabiFud”, then the iCloud container name should be “iCloud.” plus the bundle bundle id: “iCloud.com.<your domain>.BabiFud”.
* The iCloud container name must be unique because this is the global identifier used by CloudKit to access the data. Since the iCloud container name contains the bundle id, the bundle id must also be unique (which is why it has to be changed from com.raywendrelich.BabiFud).
* In order for the entitlements piece to work, the app/bundle id has to be listed in the App IDs portion of the Certificates, Identifiers, and Profiles portal. This means the certificate used to sign the app has to be from the set team id and has to list the app id, which also implies the iCloud container id.Normally, Xcode does all of this automatically if you are signed in to a valid developer account. Unfortunately, this sometimes gets out of sync. It can help to start with a fresh ID and, using the iCloud capabilities pane, change the CloudKit container ID to match. Otherwise, to fix it you may have to edit the info.plist or BabiFud.entitlements files to make sure the id values there reflect what you set for the bundle id.
