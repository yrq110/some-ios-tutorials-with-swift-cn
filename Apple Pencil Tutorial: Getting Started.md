#Apple Pencil Tutorial: Getting Started
##Apple Pencil指南：入门
[原文地址](https://www.raywenderlich.com/121834/apple-pencil-tutorial)

![pencil](http://www.raywenderlich.com/wp-content/uploads/2016/12/Final3.png)

>更新至Xcode 7.3、iOS9.3与Swift 2.2  04-01-2016

我知道你们大多数人都有一台搭配一只Pencil（触控笔，以下统一用Pencil）的iPad Pro（话外音:并没有）

也许你会像我一样，一旦感受到使用Pencil进行绘画的魅力，就会想将其添加到你的所有app中。

当我买了第一台iPad时我就一直在等待一个类似这样的设备。从我的涂鸦中可以看到虽然我并不是伦勃朗（17世纪荷兰画家）不过我发现用Pencil来做笔记是很棒的。我能想象出那些艺术工作者会使用Apple Pencil创造出各种各样令人惊叹的作品。

在这篇Apple Pencil教程中，你会将会学到Pencil中的细节。这里是你需要学习的一些关键点：
* 如何使用force
* 如何提高准确度
* 如何实现阴影效果
* 如何添加一个橡皮擦
* 如何提升使用预测与实际绘图的体验

到了这篇教程的最后，你就做好了将Apple Pencil集成到你app的准备了！

##前提

学习本教程，你需要：
* 一台iPad Pro与Apple Pencil。在模拟器中无法测试Pencil，而且只能在iPad Pro上使用，老版的iPad上无法使用。听起来你有一个升级设备的理由了！
* 最低Xcode 7.1与iOS 9.1
* 掌握Core Graphics。你需要知道什么是context，如何创建它，怎么进行线条绘图。看看这个[the first part of our Core Graphics tutorial ](http://www.raywenderlich.com/90690/modern-core-graphics-with-swift-part-1)，会使你快速熟悉，那个app还会提醒你喝水:)

##入门
在这篇教程中，你将会创建一个叫做Scribble的app。这是一个可以让你使用响应式UI、压力灵敏度与阴影进行绘图的简易app。

下载[Scribble](http://www.raywenderlich.com/wp-content/uploads/2016/12/Scribble-StarterV1.zip)。在你的iPad Pro上试试，用Pencil与手指进行绘图。

![image](http://www.raywenderlich.com/wp-content/uploads/2016/01/GettingStarted.png)

晃动iPad清屏，就像一个[Etch-A-Sketch](https://en.wikipedia.org/wiki/Etch_A_Sketch)！
