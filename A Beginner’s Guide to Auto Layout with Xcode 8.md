# A Beginner’s Guide to Auto Layout with Xcode 8
## Xcode 8中的Auto Layout新手指南

***

>* 原文链接 : [A Beginner’s Guide to Auto Layout with Xcode 8](http://www.appcoda.com/auto-layout-guide/)
* 原文作者 : [SIMON NG](http://www.appcoda.com/author/admin/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

Auto layout is a constraint-based layout system. It allows developers to create an adaptive UI that responds appropriately to changes in screen size and device orientation. Some beginners find it hard to learn and avoid using it. Some developers even avoid using it. But believe me, you won’t be able to live without it when developing an app today.

With the release of iPhone 6/6s and 6/6s Plus, Apple’s iPhones are available in different screen sizes including 3.5-inch, 4-inch, 4.7-inch and 5.5-inch displays. Without using auto layout, it would be very hard for you to create an app that supports all screen resolutions. And, starting from Xcode 6, it is inevitable to use auto layout to design the user interface. This is why I want to teach you auto layout at the very beginning of this book, rather than jumping right into coding a real app.

In this tutorial, I want to help you build a solid foundation on designing an adaptive user interface using auto layout.

Auto layout is not as difficult as some developers thought. Once you understand the basics, you will be able to use auto layout to create complex user interfaces for all types of iOS devices.

## Why Auto Layout?

Let me give you an example, and you’ll have a better idea why auto layout is needed. Download and open the starter project. Instead of running the app on iPhone 6/6s simulator, run it using the iPhone 6 Plus or iPhone 5 simulator. You’ll end up with the results illustrated in figure below. It turns out that the button isn’t centered when running on other iPhone devices, except iPhone 6/6s.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-1.png?12321214124)

Let’s try one more thing.

Click the Stop button and run the app using the iPhone 6/6s simulator. After the simulator launches, go up to the menu and select Hardware > Rotate Left (or Rotate Right) from the menu. This rotates the device to landscape mode. Alternatively, you can press command+left arrow/right arrow to rotate the device sideway. Again, the Hello World button is not centered.

Why? What’s wrong with it?

As you know, the iPhone devices have different screen dimensions:

* For iPhone 5/5s, the screen in portrait mode consists of 320 points (or 640 pixels) horizontally and 568 points (or 1136 pixels) vertically.
* For iPhone 6/6s, the screen consists of 375 points (or 750 pixels) horizontally and 667 points (or 1334 pixels) vertically.
* For iPhone 6/6s Plus, the screen consists of 414 points (or 1242 pixels) horizontally and 736 points (or 2208 pixels) vertically.
* For iPhone 4s, the screen consists of 320 points (or 640 pixels) and 480 points (or 960 pixels).
