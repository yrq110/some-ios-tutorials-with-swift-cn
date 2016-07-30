#Creating Gradient Colors Using CAGradientLayer
##使用CAGradientLayer创建渐变色

[原文地址](http://www.appcoda.com/cagradientlayer/)

每个开发者在开发app时都会使用图像的色彩融合来得到一个赏心悦目的UI效果，尝试达到最佳的用户体验。然而有时需要做一些额外的工作，普通的颜色可能不会得到较好的展示效果，而使用渐变色则会产生更好的效果。我有一些使用渐变色的经验以及关于它的建议，在这里跟大家分享下。创建渐变色是很简单的因此开发者不去用的话是较可惜的。

那么，如何可以快速轻松地创建一个渐变效果？这里有三种方法。第一种，也是最不推荐的一种，使用具有渐变效果的图像，最大的缺陷就是如果没有设计出一系列不同状态的渐变的话就不能在运行过程中随意修改渐变效果。第二种方法是使用Core Graphics技术，需要掌握相关的知识(比如图形上下文、色彩空间等等)，并且Core Graphics框架是给相对高级的用户准备的，新人开发者可能用起来比较费劲。最后一种，也是最快速最简便的一种方式:使用CAGradientLayer对象。

CAGradientLayer是一个CALayer类的子类，顾名思义，用来产生渐变效果的类。使用4行代码、少于1分钟就可以生成一个简单的渐变效果，并且通过提供的少量属性的调整就能得到一个很好的结果。几乎所有种类渐变都有动画效果，因此不需要很费劲就能得到较好的结果。在下面的内容中会讨论其中的细节，很重要的一点是：你需要了解当你使用CAGradientLayer类来创建渐变效果时，实际上操作的是运用渐变的视图层。CAGradientLayer的一个缺点就是不支持径向渐变(radial gradient)，不过如果你主要使用的是线性渐变就完全满足需求了。

在之后的章节会介绍跟配置渐变效果有关的每个属性的详细信息，我会使用非常棒的色彩来演示，为了方便起见仅用两种颜色，对于demo来说足够了，你也可以使用更多的色彩。

##创建一个渐变层

创建一个渐变色彩层是项快速并且简单的任务，必须要制定一些参数。实际上你需要先设置少量的属性后就可以接着进行后面的操作，最花时间的也许是在操作渐变层时对最终结果参数的微调，对层属性设定不同的值会产生完全不同的效果。

让我们来逐步看看其中的细节，首先需要一个演示的app。打开Xcode创建一个新应用，确保选择的是单视图界面模板，跟着向导完成创建。

假设你已经准备好了一个新工程，点击ViewController.swift文件，在类中声明如下属性:
````swift
var gradientLayer: CAGradientLayer!
````
在之后会进行gradientLayer属性的设置，为了在目标视图中显示一个渐变层的最小实现步骤如下所示:

1. 初始化一个CAGradientLayer对象(这里是gradientLayer)。
2. 设置渐变层的尺寸。
3. 设置向导产生渐变效果的颜色。
4. 将渐变层作为子层添加到目标视图层。

除了上面这些步骤，还有一些需要配置的参数，之后会讲到。现在先主要关注这些步骤即可，简便起见，使用ViewController类中的默认视图作为目标视图，使用渐变色彩来填充它。

来操作一下ViewController类，创建一个初始化与设置渐变层属性默认值的新方法:
````swift
func createGradientLayer() {
    gradientLayer = CAGradientLayer()
 
    gradientLayer.frame = self.view.bounds
 
    gradientLayer.colors = [UIColor.redColor().CGColor, UIColor.yellowColor().CGColor]
 
    self.view.layer.addSublayer(gradientLayer)
}
````
在上述代码段中发生了: 首先，初始化了渐变层对象，设置它的尺寸，使其与视图控制器视图的边界属性相等。接着指定渐变效果的色彩，最后将渐变层作为子层添加到默认的视图层上。

调用上述的viewWillAppear(_: )方法:
````swift
override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
    createGradientLayer()
}
````
…然后运行app，效果如下:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_1_first_gradient.png)

不是很糟，仅用了4行简单的代码，来深入一下其中的细节。

##渐变色

虽然上面的代码很简单，不过其中包含一个重要的属性:色彩属性。首先我觉得我不需要解释:不设置色彩的话就不会有一丝渐变效果。其次，这个属性需要接受一个色彩数组，而不是一个UIColor对象。在上面的例子中只用了两种颜色，你可以用更多的颜色。比如说设置如下的色彩集:
````swift
gradientLayer.colors = [UIColor.redColor().CGColor, UIColor.orangeColor().CGColor, UIColor.blueColor().CGColor, UIColor.magentaColor().CGColor, UIColor.yellowColor().CGColor]
````
…运行app后如下所示:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_2_second_gradient.png)

色彩属性有个很棒的特性就是可以设置动画，意味着你可以使用动画转换的方式变换渐变效果，为了证明它，来创建一个可用的由色彩数组组成的集合，点击视图时切换执行色彩的色彩集，不过色彩集之间的变换是具有动画效果的。

首先，在ViewController类的顶部，在gradientLayer属性之后声明两种新属性:

````swift
var colorSets = [[CGColor]]()
 
var currentColorSet: Int!
````
colorSets数组接受的元素是由CGColor对象组成的数组，currentColorSet表示当前执行渐变效果的色彩集。

现在来创建色彩集，下面的颜色只是个例子，你可以使用想要的颜色N:
````swift
func createColorSets() {
    colorSets.append([UIColor.redColor().CGColor, UIColor.yellowColor().CGColor])
    colorSets.append([UIColor.greenColor().CGColor, UIColor.magentaColor().CGColor])
    colorSets.append([UIColor.grayColor().CGColor, UIColor.lightGrayColor().CGColor])
 
    currentColorSet = 0
}
````
上面的方法中，将由不同色彩组成的数组添加进了colorSets数组中，并且还设置了currentColorSet属性的初始值。

来在viewDidLoad()方法中调用它:
````swift
override func viewDidLoad() {
    super.viewDidLoad()
 
    createColorSets()
}
````
还需要稍微修改下这个createGradientLayer()方法:
````swift
gradientLayer.colors = [UIColor.redColor().CGColor, UIColor.orangeColor().CGColor, UIColor.blueColor().CGColor, UIColor.magentaColor().CGColor, UIColor.yellowColor().CGColor]
````
…用下面的代码替换:
````swift
gradientLayer.colors = colorSets[currentColorSet]
````
这样就会使用通过currentColorSet属性所指定的色彩集，而不是之前使用的默认色彩。

之前我提到过：点击视图可以在不同色彩集间通过动画的方式变换，这意味着需要给视图添加一个点击的手势识别器，在viewDidLoad()中添加如下代码:
````swift
override func viewDidLoad() {
    ...
 
    let tapGestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(ViewController.handleTapGesture(_:)))
    self.view.addGestureRecognizer(tapGestureRecognizer)
}
````
若视图接收到点击手势时会调用handleTapGesture(_:)方法，离完成不远了。注意到下面代码中用到的CABasicAnimation，使用它来产生层色彩属性变换的动画效果，这里我认为你已经具备了有关CABasicAnimation的知识，了解动画是如何实现的，如果不是的话请去google下。没有包含某些属性，只使用了执行动画所需的基本属性。
````swift
func handleTapGesture(gestureRecognizer: UITapGestureRecognizer) {
    if currentColorSet < colorSets.count - 1 {
        currentColorSet! += 1
    }
    else {
        currentColorSet = 0
    }
 
    let colorChangeAnimation = CABasicAnimation(keyPath: "colors")
    colorChangeAnimation.duration = 2.0
    colorChangeAnimation.toValue = colorSets[currentColorSet]
    colorChangeAnimation.fillMode = kCAFillModeForwards
    colorChangeAnimation.removedOnCompletion = false
    gradientLayer.addAnimation(colorChangeAnimation, forKey: "colorChange")
}
````
首先我们要决定下一个色彩集的索引，如果当前选择的色彩集是colorSets数组中的最后一个，则从最开始(currentColorSet = 0)选取，否则给currentColorSet属性+1。

第二部分的代码是与动画相关的，duration与toValue是最重要的属性，duration意味着动画持续多久，toValue则是设置的期望目标色彩集。kCAFillModeForwards属性使动画结束后保持最后的状态，不会变为初始色彩，不过这并不是永久的，需要明确设置新的渐变色彩，何时设置？在动画完成后即可，使用下面这个方法，当CABasicAnimation完成时会调用它:
````swift
override func animationDidStop(anim: CAAnimation, finished flag: Bool) {
    if flag {
        gradientLayer.colors = colorSets[currentColorSet]
    }
}
````
再修改下handleTapGesture(_:)方法，这样上面的方法就起作用了:
````swift
func handleTapGesture(gestureRecognizer: UITapGestureRecognizer) {
    ...
 
    // Add this line to make the ViewController class the delegate of the animation object.
    colorChangeAnimation.delegate = self
 
    gradientLayer.addAnimation(colorChangeAnimation, forKey: &quot;colorChange&quot;)
}
````
搞定了。可以自由的调整动画持续时间，我设成了2秒，可以看到渐变的效果很明显:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_3_color_change_2.gif)

##颜色位置

知道渐变效果中颜色的设置与改变是渐变层的基础，离最终的目标还是有一段距离的。还需要知道如何修改层中每个色彩所覆盖的区域，这也是很重要的。

如果你看看之前创建的那些渐变效果的话会发现默认情况下每个颜色都会占据层中的一半区域:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_1_first_gradient.png)

可以使用CAGradientLayer类中的locations属性来改变这个区域，locations属性接受一个由NSNumber对象组成的数组，每个数字都决定一个色彩的起始位置，并且还需要手动设置为0.0到1.0范围内的数。

百闻不如一见，在createGradientLayer()方法中添加如下行:

````swift
gradientLayer.locations = [0.0, 0.35]
````
再次运行app，看看渐变效果的变化:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_5_locations_1.png)

第二个颜色的起始位置是根据我们在locations数组中设置的第二个值而定的，相当于第二个颜色覆盖了层的65%(1.0 – 0.35 = 0.65)的区域。采取一项预防措施，确保每个颜色的位置不会比下一个颜色位置的值大，否则会发生的非期望的重叠现象:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_6_locations_overlap.png)

若你想在app中看到上图的效果，只需要把色彩的location设置成[0.5, 0.35]即可。

来给demo再加点料吧，给视图添加一个点击手势识别器。这次将点击事件设为需要两个手指的操作，在viewDidLoad()中添加如下行:

````swift
override func viewDidLoad() {
    ...
 
    let twoFingerTapGestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(ViewController.handleTwoFingerTapGesture(_:)))
    twoFingerTapGestureRecognizer.numberOfTouchesRequired = 2
    self.view.addGestureRecognizer(twoFingerTapGestureRecognizer)
}
````
在handleTwoFingerTapGesture(_:)方法中会给不同色彩分配随机的位置，需要确保firstColorLocation总小于secondColorLocation，另外在改变位置时会将新的位置值实时打印在控制台中:
````swift
func handleTwoFingerTapGesture(gestureRecognizer: UITapGestureRecognizer) {
    let secondColorLocation = arc4random_uniform(100)
    let firstColorLocation = arc4random_uniform(secondColorLocation - 1)
 
    gradientLayer.locations = [NSNumber(double: Double(firstColorLocation)/100.0), NSNumber(double: Double(secondColorLocation)/100.0)]
 
    print(gradientLayer.locations!)
}
````
使用两个手指操作的效果如下:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_7_change_locations.gif)

注意默认情况下locations属性是空的，需要考虑到可能会崩溃的问题。虽然现在在demo中为了方便起见只使用了两种颜色，在使用更多颜色产生渐变效果时上面的方法同样适用。

##渐变方向

已经知道了渐变层颜色属性的一些相关操作，来着手学习一下如何处理渐变效果的方向吧。首先看看下面这张图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_1_first_gradient.png)

一眼就能看出渐变效果是从顶部开始朝向底部，实际上这是渐变层颜色的默认方向，跟其他属性一样，可以通过重写得到一个不同的方向。

CAGradientLayer类提供了两个指定渐变方向的属性:

* startPoint-起始点
* endPoint-终止点

一般用CGPoint值来表示上面的属性，x与y值的范围都应为0.0到1.0之间。起始点描述了第一个颜色的起始坐标，终止点描述了最后一个颜色的终止坐标，用这种方式来决定渐变的方向。这里有个重要的细节:坐标是与操作系统的坐标空间一致的。

这意味着什么?

为了更好理解，看看下面这张图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_8_iPhone_image-585x1024.png)

在iOS中，零点(起始点)是屏幕左上角的点(x = 0.0, y = 0.0)，终止点是右下角的点(x = 1.0, y = 1.0)，其它点都在这个坐标系中。

上面的坐标系与其他操作系统的并不相同，比如说Mac上的一个窗口(这里是个文本编辑器):

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_9_mac_window.png)

这里的起始点是在左下角，终止点是右上角，坐标系与iOS的是不同的。

默认的起始点是(0.5, 0.0)，终止点是(0.5, 1.0)，注意到x值没变，y值从0.0(顶部)变成了1.0(底部)，创建了向下的方向。来尝试一下不同的方向，在createGradientLayer()方法中添加如下两行:

````swift
func createGradientLayer() {
    ...
 
    gradientLayer.startPoint = CGPointMake(0.0, 0.5)
    gradientLayer.endPoint = CGPointMake(1.0, 0.5)
}
````
x值从0.0变为了1.0，y值不变，产生了一个向右的渐变效果:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t53_10_gradient_right.png)

为了更好的理解渐变方向，在0.0到1.0的范围内随意修改x和y的值看看改变后产生的效果。我们来给demo中添加一个新功能，使用拖动手势来改变方向，支持如下方向:

* 向上
* 向下
* 向右
* 向左
* 从左上到右下From Top-Left to Bottom-Right
* 从右上到左下From Top-Right to Bottom-Left
* 从左下到右上From Bottom-Left to Top-Right
* 从右下到左上From Bottom-Right to Top-Left

回到工程中，新建一个枚举来表示所有的方向:
````swift
enum PanDirections: Int {
    case Right
    case Left
    case Bottom
    case Top
    case TopLeftToBottomRight
    case TopRightToBottomLeft
    case BottomLeftToTopRight
    case BottomRightToTopLeft
}
````
之后再ViewController类中声明一个标示渐变方向的新属性:
````swift
var panDirection: PanDirections!
````
panDirection属性会获取手势移动的方向，The panDirection property will get the proper value depending on the movement of the finger. We are going to handle that as a two-step task: Initially we’ll determine the desired direction and we’ll assign it to the above property. Next, and right after the pan gesture is finished, we’ll set the proper values to the startPoint and endPoint properties, taking into account of course the indicated direction.

Before doing all that, we need to create a new pan gesture recogniser object and add it to the view. For that, go to the viewDidLoad() method and add the lines you see next:
````swift
override func viewDidLoad() {
    ...
 
    let panGestureRecognizer = UIPanGestureRecognizer(target: self, action: #selector(ViewController.handlePanGestureRecognizer(_:)))
    self.view.addGestureRecognizer(panGestureRecognizer)
}
````
In the implementation of the handlePanGestureRecognizer(_:) we will use the velocity property of the gesture recogniser. If that velocity is more than 300.0 points towards any direction (x or y), that direction will be taken into account. The logic is simple to understand: The primary case is to check the velocity on the horizontal axis. The velocity on the vertical axis is a check being made in a secondary level. See the comments to get it:
````swift
func handlePanGestureRecognizer(gestureRecognizer: UIPanGestureRecognizer) {
    let velocity = gestureRecognizer.velocityInView(self.view)
 
    if gestureRecognizer.state == UIGestureRecognizerState.Changed {
        if velocity.x > 300.0 {
            // In this case the direction is generally towards Right.
            // Below are specific cases regarding the vertical movement of the gesture.
 
            if velocity.y > 300.0 {
                // Movement from Top-Left to Bottom-Right.
                panDirection = PanDirections.TopLeftToBottomRight
            }
            else if velocity.y < -300.0 {
                // Movement from Bottom-Left to Top-Right.
                panDirection = PanDirections.BottomLeftToTopRight
            }
            else {
                // Movement towards Right.
                panDirection = PanDirections.Right
            }
        }
        else if velocity.x < -300.0 {
            // In this case the direction is generally towards Left.
            // Below are specific cases regarding the vertical movement of the gesture.
 
            if velocity.y > 300.0 {
                // Movement from Top-Right to Bottom-Left.
                panDirection = PanDirections.TopRightToBottomLeft
            }
            else if velocity.y < -300.0 {
                // Movement from Bottom-Right to Top-Left.
                panDirection = PanDirections.BottomRightToTopLeft
            }
            else {
                // Movement towards Left.
                panDirection = PanDirections.Left
            }
        }
        else {
            // In this case the movement is mostly vertical (towards bottom or top).
 
            if velocity.y > 300.0 {
                // Movement towards Bottom.
                panDirection = PanDirections.Bottom
            }
            else if velocity.y < -300.0 {
                // Movement towards Top.
                panDirection = PanDirections.Top
            }
            else {
                // Do nothing.
                panDirection = nil
            }
        }
    }
    else if gestureRecognizer.state == UIGestureRecognizerState.Ended {
        changeGradientDirection()
    }
}
````
There are two things to notice above (further than how we determine the pan gesture direction of course):

1. The panDirection becomes nil if none of the other conditions is satisfied.
2. The direction is specified in the Changed state of the gesture. When the gesture is ended, a call to the changeGradientDirection() method is taking place, so the new direction to apply based on the value of the panDirection property.
