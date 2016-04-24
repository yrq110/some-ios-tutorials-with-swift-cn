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

原理就是，Scribble中具有一个能识别手指或Pencil触摸的画板视图，并且根据你触摸的变化实时更新显示的视图。

看看CanvasView.swift中的代码。

最关键的代码在touchesMoved(_:withEvent:)中，当用户与画板视图交互会被触发。这个方法创建了一个CGContext并且将其中的图像绘制在负责显示的画板中。

touchesMoved(_:withEvent:)中会调用drawStroke(_:touch:)在context中的前一触摸与当前触摸之间画一条线

touchesMoved(_:withEvent:)将画板视图显示的图像替换为context中最新的图像。 

看到没？就是这么简单。
##第一次使用Pencil绘制
就算在数字环境中，用你的手指绘画从未是一件优雅的事。使用Pencil来绘制就像传统的模拟方法，使用一个铅笔与纸张的基础UI。

现在准备好见识Pencil的第一个特性——force。当你用力触摸屏幕时得到的线条就比较宽。这个特性并不适用于你的手指，这里有一个小九九之后你就会看到。

Force的大小被记录在touch.force中。一个平均力度触摸的force是1.0，你需要用force乘以某个值来产生正确的线条宽度。更多详情请见下文

打开CanvasView.swift在类的最前面添加如下常量:
~~~~
private let forceSensitivity: CGFloat = 4.0
~~~~
你可以调整这个forceSensitivity量，使线条宽度的变化对压力更敏感。

找到lineWidthForDrawing(_:touch:)，这个方法用来计算线条的宽度。

在return语句前添加如下代码:
~~~~
if touch.force > 0 {
  lineWidth = touch.force * forceSensitivity
}
~~~~
这里你通过计算触摸点的force与forceSensitivity的乘积来得到线条宽度，不过记住这仅适用于Pencil。如果你使用手指的话，touch.force将会是0，你就不能改变线条宽度了。

运行一下。使用Pencil画一些线条，注意线条是怎样随着你触摸屏幕的力度变化而变化的。

![force](http://www.raywenderlich.com/wp-content/uploads/2016/12/Force.png)

##更加平滑的绘制
你会注意到当你绘制时，线条中会有一些尖锐的过渡点而不像自然的曲线那样。在以前，你也许会做一些像将线条转换为样条曲线这样的复杂工作才能得到适宜的图案，不过Pencil及其简化了这些工作。

Apple告诉我们iPad Pro扫描触摸事件的速率是120次/秒，将Pencil在屏幕附近时扫描速率会翻倍达到240次/秒。

iPad Pro的刷新速率是60Hz。这意味着理论上识别出两个触摸点时将只能显示一个。另外，如果在当前场景下有大量事务需要处理时，一些帧的触摸事件会全部丢失因为被占用的主线程无法处理它。

尝试快速的画一个圆。它应该是圆的，不过结果看起来像有一堆锯齿点的多边形:

![circle](http://www.raywenderlich.com/wp-content/uploads/2016/12/Circle.png)

为了解决这个问题，Apple提出了一个合并触摸(coalesced touches)的概念。捕捉那些丢失的触摸放进一个新的UIEvent数组中，你可以通过coalescedTouchesForTouch(_:)来访问。

找到CanvasView.swift中的touchesMoved(_:withEvent:)，将:
~~~~
drawStroke(context, touch: touch)
~~~~
替换为:
~~~~
// 1
var touches = [UITouch]()
 
// 2
if let coalescedTouches = event?.coalescedTouchesForTouch(touch) {
  touches = coalescedTouches
} else {
  touches.append(touch)
}
 
// 3
print(touches.count)
 
// 4
for touch in touches {
  drawStroke(context, touch: touch)
}
~~~~
让我们来逐段分析:
1. 首先，你创建了一个新数组用来放置你需要处理的所有触摸点。
2. 检查合并触摸，如果存在则将他们存放进新的数组
3. 添加一个log来看看你需要处理的触摸点数量
4. 最后，对于新数组中每一个触摸点都要调用一次drawStroke(_:touch:)

运行一下。用你的Pencil画一些优美的[花饰](https://en.wikipedia.org/wiki/Curlicue)，沉醉于黄油般的光滑与笔触宽度调节中:

![curlicue](https://cdn1.raywenderlich.com/wp-content/uploads/2016/12/Curlicue-480x282.png)

将你的注意转到调试窗口。你将会发现当你使用Pencil绘制而不是手指时，你会得到更多的触摸点。

你还会发现使用Pencil画的圆更圆，这就是由于iPad Pro感应到Pencil时会扫描出双倍的触摸点。

##倾斜Pencil
现在你可以熟练的在app里作画了。然而如果你读过任何Pencil评测的话，你会记得讨论过它有阴影绘制的能力。用户们需要做的就是将笔倾斜，殊不知这种阴影是不会自动产生，取决于我们这些聪明的开发者如何利用代码去让它工作。

###高度、方位与单位向量
在这节中，我会向你描述如何测量这种倾斜，你会在下一节添加简易的阴影效果。

当你使用Pencil时，你可以在三维空间中旋转它。上下的方向被叫做高度(altitude),两侧被叫做方位(azimuth):

![altitue](https://cdn3.raywenderlich.com/wp-content/uploads/2016/12/AzimuthDiagram.png)

高度角属性是UITouch在iOS9.1中新添加的属性，仅适用于Apple Pencil。它的单位是[弧度](https://en.wikipedia.org/wiki/Radian)。当Pencil平放在iPad表面时高度角就是0.当它垂直于屏幕时高度角是π/2。记住360度对应的是2π弧度，因此π/2就是90度。

UITouch有两个新方法用来获取方位角:

azimuthAngleInView(_:)与azimuthUnitVectorInView(_:)。根据你的情况选择相应的方法。

感受一下方位角单位向量是如何工作的。仅供参考，一个单位向量的长度为1，从坐标(0,0)点指向一个方向。

![vector](https://cdn3.raywenderlich.com/wp-content/uploads/2016/12/AzimuthVector-361x320.png)

在touchesMoved(_:withEvent:)的最上面，在guard语句之后添加:
~~~~
print(touch.azimuthUnitVectorInView(self))
~~~~
运行一下。
