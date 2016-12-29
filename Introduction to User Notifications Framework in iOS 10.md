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

Open up Main.storyboard and take a look at ViewController. The app we’re making today is going to remind its users to go online and read an AppCoda article. It lets users pick a date and time for a notification reminder. The first change we’re going to make to this app is simple: we’re going to enable notifications! Currently, picking a date and time does nothing. We’re going to update the app so that setting a date and time creates a notification at the specified time.

Go ahead and open ViewController.swift. Let’s get started.

When you open ViewController.swift, you’ll notice it’s pretty empty. There is only an action method named datePickerDidSelectNewDate that will be invoked when a new date is selected in the date picker. As mentioned before, right now, the app does nothing instead of creating a notification. Let’s implement the feature.

The process for creating a notification has changed slightly in iOS 10. Developers must now create requests, which are requests to iOS to present a notification. These requests consist of 2 components: triggers and content. A trigger is a set of conditions that must be met for a notification to deliver. To show a notification in iOS 10, we begin with a trigger. The class we use for date based triggers is UNCalendarNotificationTrigger. Apple provides a few others, such as UNLocationNotificationTrigger (which triggers a notification when the user arrives a particular location).

The first step to displaying a notification on iOS, as stated above, is creating a trigger. Let’s go ahead and make one now.

Go to AppDelegate.swift, and make sure you import the UserNotification framework. All the notification-related APIs are organized in this framework:

```swift
import UserNotifications
```

Next, add a function called scheduleNotification(at date: Date). As its name implies, this function will be responsible for scheduling a notification at a provided date.

```swift
func scheduleNotification(at date: Date) {
    let calendar = Calendar(identifier: .gregorian)
    let components = calendar.dateComponents(in: .current, from: date)
    let newComponents = DateComponents(calendar: calendar, timeZone: .current, month: components.month, day: components.day, hour: components.hour, minute: components.minute)
}
```

In the above code, we use the Gregorian calendar to separate a provided date into components. Dates and components are beyond the scope of this tutorial, but all you need to know is that a date consists of components. An individual component represents a piece of a date, such as the hour, minute, second, etc. The code we have written so far separates the date parameter into components and it saves these components in the components constant. The reason we do this is that UNCalendarNotificationTrigger requires date components, not dates, to function. However, one other oddity of UNCalendarNotificationTrigger is that the complete set of components taken directly from a date do not seem to work. Instead, we need to construct our own instance of DateComponents using some information from the existing instance we have. The third line does just that: it creates a new instance of DateComponents using only the relevant information from date.

Now that we have the components from date, let’s go ahead and make a calendar notification trigger. Like most other things in iOS, Apple has made this process pretty easy. Just add the following line of code in the function:

```swift
let trigger = UNCalendarNotificationTrigger(dateMatching: newComponents, repeats: false)
```
As you can probably infer, this line creates a new UNCalendarNotificationTrigger using the date components we extracted above. We have set repeats to false because we only want the notification to be appeared once.

Now that we have created a trigger, the next thing is to create some content to display in our notification. This can be done through the UNMutableNotificationContent class. You can add the following lines of code to create the notification content. Simply put the code snippet right below the trigger variable:

```swift
let content = UNMutableNotificationContent()
content.title = "Tutorial Reminder"
content.body = "Just a reminder to read your tutorial over at appcoda.com!"
content.sound = UNNotificationSound.default()
```

These lines are pretty self explanatory: we create a new instance of UNMutableNotificationContent, and then set its title, body, and sound accordingly.

It looks like we’re all ready to show our first notification! However, there are still a couple of steps left: 1. creating a notification request, 2. providing it to the system, that tells iOS to show our notification.

To create a notification request, you instantiate a UNNotificationRequest object with the corresponding content and trigger. You also need to give it a unique identifier for identifying this notification request. Insert the following line of code in the scheduleNotification function:

```swift
let request = UNNotificationRequest(identifier: "textNotification", content: content, trigger: trigger)
```

Easy, right? We create a request with 3 parameters:

* identifier: This is a unique identifier for our request. As you’ll see shortly, this identifier can be used to cancel notification requests.
* content: This is the notification content we created earlier.
* trigger: This is the trigger we would like to use to trigger our notification. When the conditions of the trigger are met, iOS will display the notification.

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
