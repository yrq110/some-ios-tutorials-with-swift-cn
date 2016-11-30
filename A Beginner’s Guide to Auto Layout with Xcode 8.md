# A Beginner’s Guide to Auto Layout with Xcode 8
## Xcode 8中的Auto Layout新手指南

***

>* 原文链接 : [A Beginner’s Guide to Auto Layout with Xcode 8](http://www.appcoda.com/auto-layout-guide/)
* 原文作者 : [SIMON NG](http://www.appcoda.com/author/admin/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

Auto layout是一个基于constraint的布局系统，允许开发者做出可以响应屏幕尺寸与设备方向的UI组件。一些新人开发者因为它的难度而敬而远之，有些开发者则不去用它。不过相信我，今天来用它开发一个app后你会爱不释手的。

随着iPhone 6/6s、6/6s Plus的发布，苹果手机的屏幕尺寸包含了3.5英寸, 4英寸, 4.7英寸和5.5英寸这几种。若不用auto layout的话很难创建一个适配所有屏幕尺寸的app，并且从Xcode 6开始必然会使用auto layout去设计用户界面。

在这篇教程中，会帮你通过使用auto layout建立一个设计响应式UI的坚实基础。

Auto layout并不像人们想象中的那么难。理解它的基础概念与使用方法后，就可以使用auto layout为所有iOS设备做出复杂的用户界面。

## 为何使用Auto Layout?

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

> ## Why Points instead of Pixels?
Back in 2007, Apple introduced the original iPhone with a 3.5-inch display with a resolution of 320×480. That is 320 pixels horizontally and 480 pixels vertically. Apple retained this screen resolution with the succeeding iPhone 3G and iPhone 3GS. Obviously, if you were building an app at that time, one point corresponds to one pixel. Later, Apple introduced iPhone 4 with retina display. The screen resolution was doubled to 640×960 pixels. So one point corresponds to two pixels for retina display.

> The point system makes our developers’ lives easier. No matter how the screen resolution is changed (say, the resolution is doubled again to 1280×1920 pixels), we still deal with with points and the base resolution (i.e. 320×480 for iPhone 4/4s or 320×568 for iPhone 5/5s). The translation between points and pixels is handled by iOS.

Without using auto layout, the position of the button we lay out in the storyboard is fixed. In other words, we “hard-code” the frame origin of the button. In our example, the “Hello World” button’s frame origin is set to (147, 318). Therefore, whether you’re using a 3.5-inch or 4-inch or 5.5-inch simulator, iOS draws the button in the specified position. Below figure illustrates the frame origin on different devices. This explains why the “Hello World” button can only be centered on iPhone 6/6s, and it is shifted away from the screen center on other iOS devices, as well as, in landscape orientation.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-2.png?12321214124)

Obviously, we want the app to look good on all iPhone models, and in both portrait & landscape orientation. This is why we have to learn auto layout. It’s the answer to the layout issues, that we have just talked about.

## Auto Layout is All About Constraints

As mentioned before, auto layout is a constraint-based layout system. It allows developers to create an adaptive UI that responds appropriately to changes in screen size and device orientation. Okay, it sounds good. But what does the term “constraint-based layout” mean? Let me put it in a more descriptive way. Consider the “Hello World” button again, how do you describe its position if you want to place the button at the center of the view? You would probably describe it like this:

The button should be centered both horizontally and vertically, regardless of the screen resolution and orientation.

Here you actually define two constraints:

* center horizontally
* center vertically

These constraints express rules for the layout of the button in the interface.

Auto layout is all about constraints. While we describe the constraints in words, the constraints in auto layout are expressed in mathematical form. For example, if you’re defining the position of a button, you might want to say “the left edge should be 30 points from the left edge of its containing view.” This translates to button.left = (container.left + 30).

Fortunately, we do not need to deal with the formulas. All you need to know is how to express the constraints descriptively and use Interface Builder to create them.

Okay, that’s quite enough for the auto layout theory. Now let’s see how to define layout constraints in Interface Builder to center the “Hello World” button.
