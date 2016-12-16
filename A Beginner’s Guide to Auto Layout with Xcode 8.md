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

做好测试app的准备了，点击Run使用iPhone 6/6s Plus(或iPhone 4s)模拟器启动app，也可以在底部的配置栏中选择其他设备与设备朝向来检查布局的情况。不论何种屏幕尺寸与朝向，按钮都会完美的出现在屏幕中心。

## 解决布局约束的问题

我们刚才设置的布局约束是没问题的，不过总会有出问题的时候，不用担心，Xcode可以检测出这些约束问题。

试着把Hello World按钮拖到屏幕的左下方部分，Xcode会立即检测到布局问题，通过橙色的约束线条指出错位的部件。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-7.png?12321214124)

当创建的约束不明确或有冲突时会出现自动布局的问题。在这里需要按钮出现在容器的水平与竖直方向的中心，不过按钮现在被放置在了视图的左下角，因此Interface Builder会检测出这个问题并用橙色的线条指出布局问题。

当存在布局问题时，文档大纲视图中会显示一个未闭合的箭头(红/橙)，点击箭头可以看到一个问题列表。Interface Builder会帮我们解决这些问题，点击问题旁的指示图标会出现一些解决方法，选择“Update Frame”并点击“Fix Misplacement”，按钮就被移动到视图中心了。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-8.png?12321214124)

为了示范如何找到并修复问题我是手动触发的这个问题。在之后章节的练习中会遇到相似的布局问题，那时就知道如何快速的解决了。

## 另一种预览故事板的方式

除了配置栏实时预览app的UI界面外，Xcode还为开发者提供了一种在不同设备上预览UI的方法。

在Interface Builder中打开辅助用弹出菜单 > Preview (1)，按住option键的同时点击Main.storyboard (Preview)，如下图:

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-9.png?12321214124)

Xcode会在辅助编辑器中显示app的UI预览界面。默认会使用iPhone 6s来显示。可以点击辅助编辑器左下角的+按钮来添加其它iOS设备(如iPhone 4s/SE)，若想看看屏幕水平朝向时的效果，直接点rotate按钮就行了。预览特性对设计app的UI界面非常有帮助。可以在修改故事板中内容的同时实时观察UI在所选设备中的情况。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-10.png?12321214124)

若你想给预览面板腾出更多的屏幕空间，可以在按下command和option的同时按下0来隐藏Utility区域。

> 提示: 如果你在辅助预览中添加了多个设备，那么Xcode也许不能一次性显示所有设备中的预览，有触摸板的话可以使用双指的左划与右划来滚动预览视图，只有鼠标的话可以按下shift的同时使用滚轮进行水平滚动。

## 添加标签

现在你已经对auto layout和预览特性有所了解了，来在视图右下角试着添加一个标签吧，看看如何添加标签的布局约束。iOS中的标签常被用来显示简单的文本与信息。

从Interface Builder编辑器的对象库中拖一个标签到视图的右下角。双击标签把文本改成“Welcome to Auto Layout”或你想要的内容。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-11.png?12321214124)

若再次打开预览辅助，会看到UI界面立马发生了变化。在没有定义标签的布局约束的情况下，在除了iPhone 6s/6s外设备上没有显示出标签。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-12.png?12321214124)

如何解决这个问题? 显然，需要设置一组合适的约束，问题是：应该添加怎样的约束?

来尝试用文字描述一下标签的位置要求，也许会这么表达:

*标签应被放置在视图的右下角。

这么描述可以不过不够具体，下面这样描述标签的位置更加具体:

*标签距离视图右边界0 points，距离下边界20 points。

这就好多了，能具体地描述一个组件位置的话，就可以轻松添加布局约束了。那么标签的约束如下:

1. 标签离视图右边界的内距为0 points。
2. 标签离视图下边界的内距为20points。

我们将这些约束转化为auto layout中的空间约束，可以使用布局按钮中的Pin来创建空间约束，不过这次我们使用Control-drag的方式来进行auto layout。在Interface Builder中，可以在使用control键的同时将一个部件拖拽到你想要添加约束的另一个部件的坐标轴上。

来添加第一个空间约束，按住control键把标签拖拽到右侧直到视图变成蓝色高亮的状态，松开按钮后会出现一个包含约束列表的菜单。选择“Trailing space to container margin”添加一个按钮与视图右边界的空间约束。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-13.png?12321214124)

在文档大纲视图中可以看到新的约束。Interface Builder用红色的线条指示出缺少了一些约束，这是正常的，因为还没有定义第二个约束。

现在按住control把标签拖拽到视图底部，松开按钮选择“Vertical Spacing to Bottom Layout Guide”，创建一个按钮与视图下边界的空间约束。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-14.png?12321214124)

添加完这两个约束后，所有的约束线条都应该是蓝色的实线。在预览UI或在模拟器中运行app时，会看到不管是何种尺寸、何种朝向的屏幕，标签都出现了正确的地方。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-15.png?12321214124)

## Top 和 Bottom Layout Guide

Bottom Layout Guide(底部布局辅助)是啥东东呢？通常来说，Bottom Layout Guide是与底部视图关联的一个约束边界，有时会发生变化，比如存在一个工具栏的话，Bottom Layout Guide就会与工具栏的顶部关联。

Top Layout Guide的话是在距视图顶部20points(状态栏的高度)的位置。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-16.png?12321214124)

这些布局辅助约束对于定义布局约束来说是非常有用的，因为他们会根据视图的布局变化自动调整。比如说，如果视图中有一个导航栏，那么top layout guide就会移动到导航栏的下方(如下图)。因此，只要UI组件的约束是与top/bottom layout guide相关联的，那么在添加一个导航栏或工具栏时界面就会自动调整成正确的布局。T

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-17.png?12321214124)

## 编辑约束

“Welcome to Auto Layout”标签现在被放在了距离视图右边界0 points的地方，若要在标签和视图右边界中添加一些空间怎么办呢？Interface Builder提供了一种很方便的修改约束的方法。

首先选择文档大纲视图中的约束，这里你可以选择“trailingMargin”约束。在属性查看器中找到这个约束的属性，比如relation, constant和priority。constant现在是0，可以改为20添加一些额外的空间。

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-18.png?12321214124)

## 练习

我觉得你现在对适配各种屏幕尺寸的方法应该有一定的了解了，来进行一个简单的练习吧。需要你做的是再添加一个名为Learn Swift的标签到视图中，并满足如下约束：

* 新标签距离top layout guide 40 points。
* 新标签在水平中心位置。

可以把标签的字体大小调整为30points，打开属性观察器编辑Font选项即可，结果应如下所示：

![](http://www.appcoda.com/learnswift/images/chapter-3/auto-layout-19.png?12321214124)

## 总结

在这里只是介绍了一下auto layout的基本内容，在开发app的过程中再更深的去探索一下它的奥秘吧！

可以在[这里](https://www.appcoda.com/resources/swift3/HelloWorldAutoLayout.zip)下载完整项目
