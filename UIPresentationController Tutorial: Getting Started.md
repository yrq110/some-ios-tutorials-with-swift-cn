# UIPresentationController Tutorial: Getting Started
## UIPresentationController指南:入门

***

>* 原文链接 : [UIPresentationController Tutorial: Getting Started](https://www.raywenderlich.com/139277/uipresentationcontroller-tutorial-getting-started)
* 原文作者 : [Ron Kliffer](https://www.raywenderlich.com/u/ron.kliffer)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/10/UIPresCon-feature.png)
> 学习如何使用UIPresentationController进行自定义跳转。

在iOS的早些时期，视图控制器的跳转一直是iOS开发者工具箱中的一个独立模块，你可能以前使用过present(\_:animated:completion:)方法，可能像其他大多数开发者一样使用iOS自带的跳转样式。在这篇UIPresentationController的教程中你会学习如何设置自定义的视图控制器跳转样式，不再局限于全屏的、弹出式的默认跳转动画，将枯燥、无趣的视图跳转表现为更加的生动、有活力。

看完这篇教程后你会学到：
1. 如何创建一个UIPresentationController子类
2. 如何使用UIPresentationController进行圆滑的自定义跳转
3. 如何复用UIPresentationController，根据不同的controller调整参数
4. 如何创建自适应的转场动画支持不同设备的屏幕尺寸

> 前提: 这篇教程需要Xcode 8.0或更高版本，这是由于需要使用新版本的Swift语句，并且假定你熟悉Swift与iOS SDK。

## 入门

场景: 2016夏季奥运会结束后，一位客户想让你做一款app，统计国际体育赛事中不同运动员与国家的所得奖牌。

由于功能需求比较简单，你的赞助者要求使用一种比较cool的跳转方式来展示运动项目列表。

起初你觉得有点慌，之后发现只需要动动指头写一个跳转工具即可，小case!

下载[开始工程](https://koenig-media.raywenderlich.com/uploads/2016/08/Medal_Count_Starter.zip)并打开它。

浏览项目目录，熟悉一下以下元素:
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

## iOS Transition的核心概念
When you call present(\_:animated:completion:), iOS does two things.
First, it instantiates a UIPresentationController. Second, it attaches the presented view controller to itself then presents it using one of the built-in modal presentation styles.
You have the power to override this mechanism and provide your own UIPresentationController subclass for a custom presentation.
Understanding these key components is mandatory if you want to build sweet presentations in your apps:
1. The presented view controller has a transitioning delegate that’s responsible for loading the UIPresentationController and the presentation/dismissal animation controllers. That delegate is an object that conforms to UIViewControllerTransitioningDelegate.
2. The UIPresentationController subclass is an object that has many presentation-customizing methods, as you’ll see later in the tutorial.
3. The animation controller object is responsible for the presentation and dismissal animations. It conforms to UIViewControllerAnimatedTransitioning. Note that some use cases warrant two controllers: one for presentation and one for dismissal.
4. A presentation controller’s delegate tells the presentation controller what to do when its trait collection changes. For the sake of adaptivity, the delegate must be an object that conforms to UIAdaptivePresentationControllerDelegate.
That’s all you need to know before you dive in!

