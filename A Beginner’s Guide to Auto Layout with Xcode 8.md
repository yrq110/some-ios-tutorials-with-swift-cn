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

As mentioned before, auto layout is a constraint-based layout system. It allows developers to create an adaptive UI that responds appropriately to changes in screen size and device orientation. Okay, it sounds good. But what does the term “constraint-based layout” mean? Let me put it in a more descriptive way. Consider the “Hello World” button again, how do you describe its position if you want to place the button at the center of the view? You would probably describe it like this:

The button should be centered both horizontally and vertically, regardless of the screen resolution and orientation.

Here you actually define two constraints:

* center horizontally
* center vertically

These constraints express rules for the layout of the button in the interface.

Auto layout is all about constraints. While we describe the constraints in words, the constraints in auto layout are expressed in mathematical form. For example, if you’re defining the position of a button, you might want to say “the left edge should be 30 points from the left edge of its containing view.” This translates to button.left = (container.left + 30).

Fortunately, we do not need to deal with the formulas. All you need to know is how to express the constraints descriptively and use Interface Builder to create them.

Okay, that’s quite enough for the auto layout theory. Now let’s see how to define layout constraints in Interface Builder to center the “Hello World” button.
