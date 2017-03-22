# UIPresentationController Tutorial: Getting Started
## UIPresentationController指南:入门

***

>* 原文链接 : [UIPresentationController Tutorial: Getting Started](https://www.raywenderlich.com/139277/uipresentationcontroller-tutorial-getting-started)
* 原文作者 : [Ron Kliffer](https://www.raywenderlich.com/u/ron.kliffer)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/10/UIPresCon-feature.png)
> Learn how to build custom presentations with UIPresentationController.

View controller presentation has been an integral part of every iOS developer’s toolkit since the early days of iOS. You’ve probably used present(_:animated:completion:) before, but if you’re like a lot of developers, you’ve stayed with the default transition styles shipped with iOS. In this UIPresentationController tutorial you’ll learn how to present view controllers with custom transitions and custom presentation styles. No longer will you be limited to full-screen or popover presentations and the standard transition animations. You’ll start with some drab, lifeless view controller presentations and you’ll bring them to life. By the time you finish, you’ll learn:
1. How to create a UIPresentationController subclass.
2. How to use a UIPresentationController for slick custom presentations; the presented view controller doesn’t even have to know it’s there.
3. How to reuse a UIPresentationController for various controllers where you’ve tweaked presentation parameters.
4. How to make adaptive presentations that support any screen size your app might encounter.

> Prequisites: This tutorial requires Xcode 8.0 or newer in order to be compatible with the latest Swift syntax.
This iOS tutorial also assumes you are familiar with Swift and the iOS SDK. If you’re new to these, try starting out with our written iOS tutorials and Swift tutorials. Make sure you catch some of the video tutorials too!

## Getting Started

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
