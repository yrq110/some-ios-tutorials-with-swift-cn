# UIPresentationController Tutorial: Getting Started
## UIPresentationController指南:入门

***

>* 原文链接 : [UIPresentationController Tutorial: Getting Started](https://www.raywenderlich.com/139277/uipresentationcontroller-tutorial-getting-started)
* 原文作者 : [Ron Kliffer](https://www.raywenderlich.com/u/ron.kliffer)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/10/UIPresCon-feature.png)
> 学习如何使用UIPresentationController进行自定义跳转。

View controller presentation has been an integral part of every iOS developer’s toolkit since the early days of iOS. You’ve probably used present(\_:animated:completion:) before, but if you’re like a lot of developers, you’ve stayed with the default transition styles shipped with iOS. In this UIPresentationController tutorial you’ll learn how to present view controllers with custom transitions and custom presentation styles. No longer will you be limited to full-screen or popover presentations and the standard transition animations. You’ll start with some drab, lifeless view controller presentations and you’ll bring them to life.
看完这篇教程后你会学到：
1. 如何创建一个UIPresentationController子类
2. 如何使用UIPresentationController进行圆滑的自定义跳转
3. 如何复用UIPresentationController，根据不同的controller调整参数
4. 如何创建自适应的转场动画支持不同设备的屏幕尺寸

> 前提: 这篇教程需要Xcode 8.0或更高版本，这是由于需要使用新版本的Swift语句，并且假定你熟悉Swift与iOS SDK。

## 入门
The scenario: With the 2016 summer games all done, a client hires you to create an app that tallies the medal count for international sporting events where athletes and nations earn various medals.
While the functional requirements are pretty simple, your sponsor asked for a rather cool looking slide-in transition to present the list of games.
At first, you feel a bit of panic but then realize you do have transition-building tools at your fingertips. You even put down the paper bag!
Download the starter project and open it.
Take a moment to familiarize your with the project and the following elements:
* MainViewController.swift: the main controller of this project from which all presentations start. This will be the only existing file in the project that you’ll modify.
* GamesTableViewController.swift: displays a list of Games the user can select.
* MedalCountViewController.swift: displays the medal count for the selected sporting event.
* GamesDataStore.swift: represents the model layer in the project. It’s responsible for creating and storing model objects.
Before you start changing things around, build and run the app to see what it does. You’ll see the following screen:

![](https://koenig-media.raywenderlich.com/uploads/2016/08/medal_count_01-282x500.png)

First, tap the Summer button to bring up the GamesTableViewController summer menu.

![](https://koenig-media.raywenderlich.com/uploads/2016/08/medal_gif_01.gif)

Notice the menu is presented the default way, which is from the bottom; the client wants to see a sleek slide-in instead.

Next, select the London 2012 games to dismiss the menu and return to the main screen. This time, you’ll see an updated logo.

![](https://koenig-media.raywenderlich.com/uploads/2016/08/medal_count_02-282x500.png)

Finally, tap the Medal Count button to bring up the MedalCountViewController for the 2012 games.

![](https://koenig-media.raywenderlich.com/uploads/2016/08/medal_count_03-282x500.png)

As you see, the old bottom-up default is used to present this controller too. Tap the screen to dismiss it.
Now that you’ve seen the app you’ll upgrade, it’s time to turn your attention to some core concepts and theory for UIPresentationController.

## Core Concepts for iOS Transition


