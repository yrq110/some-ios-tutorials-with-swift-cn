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

* 水平居中
* 竖直居中

这些约束表示出了界面中按钮布局的规则。

Auto layout的核心就是一些约束，当用文字描述约束时，在auto layout中这些约束会通过数学形式来表示。比如说，当定义一个按钮的位置时，也许想说“按钮的左侧边界应距容器视图的左侧边界30pt。” 变成数学形式就是 button.left = (container.left + 30)。

幸运的是不需要我们去处理这些公式，你只需知道如何使用Interface Builder创建合适的约束即可。

好了，auto layout的基本知识差不多够了，现在来看看在Interface Builder中如何定义处于中心位置“Hello World”按钮的约束。

## Interface Builder中的实时预览

首先，打开HelloWorld工程中的Main.storyboard(或者从这儿下载:http://www.appcoda.com/resources/swift3/HelloWorld.zip). 在UI界面上添加布局约束之前，先简要介绍一下Xcode 8中的一个方便的特性。

除了在模拟器中观察app的UI外，Xcode 8还在Interface Builder中提供了一个配置栏，方便开发者进行用户界面的实时预览。

默认情况下Interface Builder被设为预览iPhone 6s的界面，为了观察在其他设备上的效果，点击`View as: iPhone 6s`按钮展开配置栏，然后选择你想要测试的iPhone/iPad设备，也可以改变设备的朝向。下图展示了一个Hello World app在iPhone 6s横屏时的实时预览。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-3.png?12321214124)

配置栏是Xcode 8中的一个很棒的特性，花点时间把玩一下。

> 注意: 可能会想“(wC hC)”是什么意思，先把它忘掉专注于学习auto layout。若对这个感兴趣可以看下我们的[Swift教程](https://www.appcoda.com/swift)。

## 使用Auto Layout居中按钮

来继续讨论auto layout，Xcode提供了两种定义约束的方法:

1. Auto layout bar
2. Control-drag

在文章中会使用两种方法来进行约束。首先使用auto layout bar，在Interface Builder编辑器的右下角应该会看到四个按钮，这些按钮都属于布局栏，可以使用它们来定义不同类型的布局约束。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-4.png?12321214124)

每个按钮都有自己的方法:

* 对齐 – 新建对齐约束，比如对齐两个视图的左边界。
* 固定 – 新建空间约束，比如定义一个UI空间的宽度。
* 问题 – 解决布局问题
* 栈 – 把视图嵌入到栈视图中，栈视图是Xcode 7中出现的新特性，在下一节会讨论这个。

如之前所说的那样，要居中“Hello World”按钮需要定义两个约束：水平居中与竖直居中，这两种约束都是相对于视图的。

需要使用对齐方法来创建约束。首先在Interface Builder选择按钮然后点击布局栏中的对齐图标。在弹出的菜单中，选中“Horizontal in container”和“Vertically in container”选项，然后点击“Add 2 Constraints”选项。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-5.png?12321214124)

> 提示: 可以按下command+0来隐藏项目导航栏，这样可以释放一些屏幕空间，设计起来更方便。

可以看到一些约束线条，展开左侧文档视图的Constraints选项，可以看到刚刚给按钮添加的两个约束，这些约束确保按钮一直保持在视图中心的位置，你也可以在右上角的Size inspector中查看这些约束。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-6.png?12321214124)

> 提示: 蓝色的约束线条表示视图布局的设置是正确无冲突的。

准备好可以测试app了。点击Run使用iPhone 6/6s Plus(或iPhone 4s)模拟器启动app，也可以在底部的配置栏中选择其他设备与设备朝向来验证布局的情况。不论何种屏幕尺寸与朝向，按钮都会完美的出现在屏幕中心。

## 解决布局的约束问题

The layout constraints that we have just set are perfect. But that is not always the case. Xcode is intelligent enough to detect any constraint issues.

Try to drag the Hello World button to the lower-left part of the screen. Xcode immediately detects some layout issues and the corresponding constraint lines turns orange that indicates a misplaced item.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-7.png?12321214124)

Auto layout issues occur when you create ambiguous or conflicting constraints. Here we said the button should be vertically and horizontally centered in the container (i.e. the view). However, the button is now placed at the lower-left corner of the view. Interface Builder found this confusing, therefore it uses orange lines to indicate the layout issues.

When there is any layout issue, the Document Outline view displays a disclosure arrow (red/orange). Now click the disclosure arrow to see a list of the issues. Interface Builder is smart enough to resolve the layout issues for us. Click the indicator icon next to the issue and a pop-over shows you a number of solutions. In this case, select the “Update Frame” option and click “Fix Misplacement” button. The button will then be moved to the center of the view.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-8.png?12321214124)

This layout issue was triggered manually. I just wanted to demonstrate how to find the issues and fix them. As you go through the exercises in the later chapters, you will probably face a similar layout issue. You should know how to resolve layout issues easily and quickly.

## An Alternative Way to Preview Storyboards

While you can use the configuration bar to live preview your app UI, Xcode provides an alternate Preview feature for developers to preview the user interface on different devices simultaneously.

In Interface Builder, open the Assistant pop-up menu > Preview (1). Now press and hold option key, then click Main.storyboard (Preview). You can refer to the figure below for the steps.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-9.png?12321214124)

Xcode will display a preview of your app’s UI in the assistant editor. By default, it shows you the preview on a iPhone 6s device. You can click the + button at the lower-left corner of the assistant editor to add other iOS devices (e.g. iPhone 4s/SE) for preview. If you want to see how the screen looks like in landscape orientation, simply click the rotate button. The Preview feature is extremely useful for designing your app’s user interface. You can make changes to the storyboard (say, adding another button to the view) and see how the UI looks on the chosen devices all at once.

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-10.png?12321214124)

If you want to free up some more screen space for the preview pane, hold both command and option keys and then press 0 to hide the Utility area.

> Quick tip: When you add more devices in the preview assistant, Xcode may not be able fit the preview of all devices sizes into the screen at the same time. If you’re using a trackpad, you can scroll through the preview by swiping left or right with two fingers. What if you’re still using a mouse with a scroll wheel? Simply hold the shift key to scroll horizontally.
