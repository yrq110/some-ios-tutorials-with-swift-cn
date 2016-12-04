# A Beginner’s Guide to Auto Layout with Xcode 8
## Xcode 8中的Auto Layout新手指南

***

>* 原文链接 : [A Beginner’s Guide to Auto Layout with Xcode 8](http://www.appcoda.com/auto-layout-guide/)
* 原文作者 : [SIMON NG](http://www.appcoda.com/author/admin/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

Auto layout是一个基于约束(constraint-based)的布局系统，允许开发者做出可以响应屏幕尺寸与设备方向的UI组件。一些新人开发者因为它的难度而敬而远之，有些开发者则不去用它。不过相信我，今天来用它开发一个app后你会爱不释手的。

随着iPhone 6/6s、6/6s Plus的发布，苹果手机的屏幕尺寸包含了3.5英寸, 4英寸, 4.7英寸和5.5英寸这几种。若不用auto layout的话很难创建一个适配所有屏幕尺寸的app，并且从Xcode 6开始必然会使用auto layout去设计用户界面。

在这篇教程中，会帮你通过使用auto layout建立一个设计响应式UI的坚实基础。

Auto layout并不像人们想象中的那么难。理解它的基础概念与使用方法后，就可以使用auto layout为所有iOS设备做出复杂的用户界面。

## 为何使用Auto Layout?

来看个例子，下载并运行[开始工程](http://www.appcoda.com/resources/swift3/HelloWorld.zip)。不要使用iPhone 6/6s模拟器，使用iPhone 6 Plus或iPhone 5模拟器运行，会得到下图的结果，按钮在除了iPhone 6/6s外的其他iPhone设备上都不在屏幕中心。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-1.png?12321214124)

来再试一下。

点击Stop按钮，使用iPhone 6/6s模拟器运行，模拟器启动后，在菜单中选择 Hardware > Rotate Left (or Rotate Right)，这会使设备旋转到水平模式。也可以按下command+左箭头/右箭头来旋转设备方向。这一次，Hello World按钮也没有在中间。

这是为何? 出啥幺蛾子了?

如你所知，iPhone设备有不同的屏幕尺寸:

* 对于iPhone 5/5s, 竖屏模式下水平方向为320 pt(640 px)，竖直方向为667 pt(1334 px)。
* 对于iPhone 6/6s, 水平方向为375 pt(750 px)，竖直方向为568 pt(1136 px)。
* 对于iPhone 6/6s Plus, 水平方向为414 pt(1242 px)，竖直方向为736 pt(2208 px)。
* 对于iPhone 4s, 水平方向为320 pt(640 px)，竖直方向为480 pt(960 px)。

> ## 为何使用点(Point pt)而不是像素(Pixel px)?
苹果在2007年发布的初代iPhone是3.5英寸的屏幕，分辨率为320×480。横向是320px，纵向是480px。苹果在iPhone 3G与3GS上也使用的这个屏幕分辨率。显然，那时构建的app是一个点对应一个像素。接着苹果就发布了使用retina显示技术的iPhone 4，屏幕分辨率是640×960的两倍，由于使用了retina显示技术，一个点对应着两个像素。

> 点系统更加方便开发者的使用。不管屏幕分辨率如何变化，使用数值为基础分辨率的点(iPhone 4/4s是320×480，iPhone 5/5s是320×568)来处理即可，把点和像素的转换工作交给iOS处理。

没有auto layout的话，我们在故事版中放置的按钮就是固定位置的。换句话说，“硬编码”了按钮的起始点。在我们的例子中，把“Hello World”按钮的起始点设为了(147, 318)。因此，不管你使用的是3.5英寸、4英寸，还是5.5英寸的模拟器，iOS都会把按钮画在指定的位置。下图中画出了在不同设备中的按钮，解释了为何“Hello World”按钮仅仅在iPhone 6/6s上是中间位置，其它设备上都偏离了中心，水平朝向上也同样偏离了。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-2.png?12321214124)

显然，我们想要在所有iPhone设备和朝向上都如我们期望的那样显示，这就是为何要学习使用auto layout，可以解决这种布局的问题。

## Auto Layout is All About Constraints

auto layout是一个基于约束(constraint-based)的布局系统，使开发者可以做出自动适配屏幕尺寸与朝向的UI组件，这听起来不错，不过“基于约束的布局”是什么意思？让我来解释一下，想想之前的那个“Hello World”按钮，如果你想把它放在视图中心会怎么描述它的位置？也许你会这么描述：

*不管怎样的屏幕分辨率与朝向，按钮都在水平与竖直方向的中心。

实际上你已经定义了两个约束:

* 水平中心
* 竖直中心

这些约束表示出了界面中按钮布局的规则。

Auto layout的核心就是一些约束，当用文字描述约束时，在auto layout中这些约束会通过数学形式来表示。比如说，当定义一个按钮的位置时，也许想说“按钮的左侧边界应距容器视图的左侧边界30pt。” 变成数学形式就是 button.left = (container.left + 30)。

幸运的是不需要我们去处理这些公式，你只需知道如何使用Interface Builder创建合适的约束即可。

好了，auto layout的基本知识差不多够了，现在来看看在Interface Builder中如何定义处于中心位置“Hello World”按钮的约束。

## Interface Builder中的实时预览

First, open Main.storyboard of your HelloWorld project (or download it from http://www.appcoda.com/resources/swift3/HelloWorld.zip). Before we add the layout constraints to the user interface, let me introduce a handy feature in Xcode 8.

Instead of viewing the app UI in simulators, Xcode 8 provides a configuration bar in Interface Builder for developers to live preview the user interface.

By default, Interface Builder is set to preview the UI on iPhone 6s. To see how your app looks on other devices, click View as: iPhone 6s button to reveal the configuration bar and then choose your preferred iPhone/iPad devices to test. You can also alter the device’s orientation to see how it affects your app’s UI. The figure below shows a live preview of the Hello World app on iPhone 6s in landscape orientation.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-3.png?12321214124)

The configuration bar is a great feature in Xcode 8. Take some time to play around with it.

> Note: You may wonder what “(wC hC)” means. Just forget about it right now and focus on learning auto layout. If you’re interested in learning more about it, you can check out our Swift book.

