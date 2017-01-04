# Introduction to User Notifications Framework in iOS 10
## iOS10中的User Notifications框架介绍

***

>* 原文链接 : [Introduction to User Notifications Framework in iOS 10](http://www.appcoda.com/ios10-user-notifications-guide/)
* 原文作者 : [PRANJAL SATIJA](http://www.appcoda.com/author/pranjalsatija/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](http://www.appcoda.com/wp-content/uploads/2016/10/user-notifications-part1-1680x1143.jpg)

欢迎阅读iOS 10中的推送教程，今天我们要讲的是如何在iOS10中实现推送，iOS10在推送相关的API上做了大量的改变，包括一个全新、强力的给予触发器的推送请求系统，会简要介绍一下这个系统，此外改变最大的部分就是一个叫做UserNotificationsUI的框架，它允许开发者将自定义的视图控制器嵌入到推送中！这就使我们可以创建出以前无法实现的强有力的推送。

这篇教程的内容中涵盖iOS中不同类型的本地推送，会从一个基本的文本推送开始，之后逐渐在推送中加入图像与用户操作。

## 简单用户推送
并不是所有我们想展现的推送都是漂亮与可交互的，因此从一个基本的文本推送开始，下载[开始项目](https://github.com/appcoda/NotificationsUI-Demo/raw/master/NotificationsUI.zip)，打开NotificationsUI.xcodeproj。浏览一下项目内容，会发现没有什么东西：设计用户界面的Main.storyboard，ViewController.swift，还有AppDelegate.swift。

![](http://www.appcoda.com/wp-content/uploads/2016/10/user-notification-storyboard.png)

打开Main.storyboard并浏览下ViewController中的内容，今天所要做的app具有提醒用户阅读AppCoda文章的功能，会让用户选择一个推送的日期与时间。为了达到目的，第一个任务显而易见：启用推送。现在选择日期与时间没有任何效果，需要修改app来设置一个日期时间使其在设定的时间进行推送。

打开ViewController.swift，开始吧！

ViewController中没什么代码，只有一个名为datePickerDidSelectNewDate的方法，当在日期选择器中选择日期后会调用这个方法。如前面所说的那样，现在app只需要创建一个推送即可，来一起实现这个功能。

iOS 10中推送的创建方法稍有改变，开发者必须先创建请求，请求iOS显示一个推送。这些请求由两部分组成：触发器和内容。为了在iOS 10中显示推送，需要从创建一个触发器开始，要使用一个基于日期的触发器——UNCalendarNotificationTrigger类。苹果同时也提供了其它一些触发器，比如UNLocationNotificationTrigger(当用户到达一个特定的地点后触发推送)。

为了在iOS中显示推送，第一步需要创建触发器，Let's go。

打开AppDelegate.swift文件，确保引入了UserNotification框架，所有与推送有关的API都在这个框架中:

```swift
import UserNotifications
```

接着添加scheduleNotification(at date: Date)方法，顾名思义，这个方法根据提供的日期来安排推送的时间。

```swift
func scheduleNotification(at date: Date) {
    let calendar = Calendar(identifier: .gregorian)
    let components = calendar.dateComponents(in: .current, from: date)
    let newComponents = DateComponents(calendar: calendar, timeZone: .current, month: components.month, day: components.day, hour: components.hour, minute: components.minute)
}
```

在上述代码中使用公历日历将提供的日期分割为不同要素(年、月、日、小时、分钟)。这部分内容超出了教程的范围，不过你需要知道一个日期是由要素组成的。一个独立的要素表示日期中的一部分，比如小时、分钟、秒等。代码将日期分割为不同要素并保存在常量中，这么做是因为UNCalendarNotificationTrigger需要日期要素，而不是日期，有一点奇怪的是，UNCalendarNotificationTrigger不接受一个由日期中所有要素组成的输入，因此需要自己通过已存在的对象建立DateComponents对象。方法中的第三行代码就是那么做的：使用日期中的相关信息来创建DateComponents对象。

现在获得了日期中的要素，可以来制作推送触发器了，这一步很简单，如下添加代码即可:

```swift
let trigger = UNCalendarNotificationTrigger(dateMatching: newComponents, repeats: false)
```
如你所想，这行代码使用刚刚提取的日期要素新建一个UNCalendarNotificationTrigger对象，将repeats属性设为false使其只出现一次。

创建好触发器了，接下来创建想在推送中显示的内容，使用UNMutableNotificationContent类来做这件事。添加如下代码来创建推送的内容，把代码放在触发器变量的后面即可:

```swift
let content = UNMutableNotificationContent()
content.title = "Tutorial Reminder"
content.body = "Just a reminder to read your tutorial over at appcoda.com!"
content.sound = UNNotificationSound.default()
```

上面的代码新建了一个UNMutableNotificationContent对象，然后设置它的title，body和sound属性。

看起来已经做好显示第一个推送的准备了！还有些事儿没做：1.创建一个推送请求。2.提交给系统，告诉iOS显示我们的推送。

需要使用触发器与内容来初始化一个UNNotificationRequest对象，还需要给予请求一个唯一标识符。在scheduleNotification函数中插入如下代码:

```swift
let request = UNNotificationRequest(identifier: "textNotification", content: content, trigger: trigger)
```

小case，不是吗? 使用了三个参数来创建请求:

* 标识符: 请求的唯一标识符，这个标识符可以用来取消推送请求。
* 内容: 之前创建的推送内容。
* 触发器: 用来触发推送的触发器，当满足条件时会iOS会显示推送。

Okay, the last thing we have to do is add the request to the notification center that manages all notifications for your app. The notification center will listen to the appropriate event and trigger the notification accordingly.

Before we add the notification request to the notification center, it is good to remove any existing notification requests. This step helps prevent unnecessary duplicate notifications. Insert the following code snippet in the function:

```swift
UNUserNotificationCenter.current().removeAllPendingNotificationRequests()
UNUserNotificationCenter.current().add(request) {(error) in
    if let error = error {
        print("Uh oh! We had an error: \(error)")
    }
}
```

The first line of code gets the shared instance of the notification center and calls the removeAllPendingNotificationRequests method to remove all pending notification requests. And then we add the request to the notification center. The method also takes in a completion handler. In our implementation, we use this handler to print an error. Before we continue, make sure your scheduleNotification function looks something like this:

```swift
func scheduleNotification(at date: Date) {
    let calendar = Calendar(identifier: .gregorian)
    let components = calendar.dateComponents(in: .current, from: date)
    let newComponents = DateComponents(calendar: calendar, timeZone: .current, month: components.month, day: components.day, hour: components.hour, minute: components.minute)
    
    let trigger = UNCalendarNotificationTrigger(dateMatching: newComponents, repeats: false)
    
    let content = UNMutableNotificationContent()
    content.title = "Tutorial Reminder"
    content.body = "Just a reminder to read your tutorial over at appcoda.com!"
    content.sound = UNNotificationSound.default()
    
    let request = UNNotificationRequest(identifier: "textNotification", content: content, trigger: trigger)
    
    UNUserNotificationCenter.current().removeAllPendingNotificationRequests()
    UNUserNotificationCenter.current().add(request) {(error) in
        if let error = error {
            print("Uh oh! We had an error: \(error)")
        }
    }
}
```
