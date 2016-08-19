#Detecting low Power Mode in Swift
##使用Swift检测省电模式

***

>* 原文链接 : [Detecting low Power Mode in Swift](http://www.ios-blog.co.uk/tutorials/swift/detecting-low-power-mode/)
* 原文作者 : [iOS-Blog Admin](http://www.ios-blog.co.uk/)
* 译者 : []()

***

在本篇入门教程中，会为你展示如何检测用户的设备进入省电模式，并且在设备进入省电模式之后还能保持后台程序刷新。我会教你使用**`NSProcessInfo`**类中的**`LowPowerEnabled`**属性，同时使用**`NSProcessInfoPowerStateDidChangeNotification`**监听省电模式状态的变化(省电模式和非省电模式状态改变)来实现所说的功能。

![](http://iosblogimagestorage.s3-us-west-2.amazonaws.com/wp-content/uploads/2016/02/12055744/Swift-Tutorial-Detect-Low-Power-Mode.png)

> 截止目前，Xcode的iOS模拟器没有省电模式的开关选项，所以要测试的话，需要在真机上跑测试工程。

## 省电模式是什么?

省电模式是苹果处理电池电量不断减少的一种手段。省电模式会停止所有的后台活动，调暗屏幕同时使用各种方式来保存电量。直接且有效，我曾经1%的电量用了半小时，听起来很假，但这是事实。

### 为什么要检测省电模式

这样一个情景：

假设你用的app在后台每15分钟连接一次web服务。类似iOS的邮件app(连接频度取决于你的设置)。app会更新数据，用户(假设这个用户叫Linda)接收到重要的数据的时候希望有提示。Linda使用这些消息通知才能在这个快节奏的世界上生存，她的圈子和工作都依赖这些消息通知。但是省电模式开了会完全停掉app后台刷新，现在好了，Linda得一小时后才会有电源到那个时候省电模式才会关闭。一小时过后，她给设备插上电源，为今天松了一口气。之后晚上她遇见了一个熟知最新八卦，行业新闻朋友(叫Sarah)，她正准备去度周末因为她捡了个大便宜。Linda一脸迷惑，因为她没有得到任何关于此事的消息，错失了本来就快要结束的Ebay促销和29刀的周末团购优惠券。Linda十分沮丧，意识到省电模式之后app就再也没响应过，也没有启动它往常的后台活动，这一切让她感到糟透了。

不要伤害像Linda这样的人。

现在，你可以检测省电模式的状态，从而可以相应的控制你的app的活动状态。

这篇教程假定你已掌握[Swift2](http://www.ios-blog.co.uk/tutorials/swift/programming-introduction/)

## 创建检测省电模式的工程

针对这个教程，我们创建一个简单的下载youtube视频的app，实际只做简单的加载并不会下载内容提取到设备上。我们将会使用swift来检测省电模式，同时会延迟省电模式的相关动作直接到省电模式被停用。

[创建新的单页应用](http://www.ios-blog.co.uk/tutorials/xcode/how-to-create-a-new-single-view-application/)，任意取名并放在合适的位置。

首先需要在代码前边定义一些变量，在ViewController.swift文件中的类定义里这样写：

```swift
var StartVideoDownload = true
let processInfo = NSProcessInfo.processInfo()
```

> `NSProcessInfo`对象会在这个方法第一次调用的时候被创建，之后每次的调用都只会返回这个已经被创建好的对象。

现在写一个方法用来下载:

```swift
func downloadStart(){
    guard let url = NSURL(string: "https://www.youtube.com/embed/YQHsXMglC9A") where
       !processInfo.lowPowerModeEnabled else{
       return
    }
StartVideoDownload = false
}

```

这个要下载的视频是Adelle的Hello。不要吐槽我。

现在我们要实现`powerModeChanged(_:)`方法。

```swift

func powerModeChanged(notif: NSNotification){
     guard mustDownloadVideo else{
         return
     }
downloadStart()
}
```

这就是在app里检测到省电模式并继续为用户提供下载的一个简单解决方案。