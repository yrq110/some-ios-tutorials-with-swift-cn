#Designing Animations with UIViewPropertyAnimator in iOS 10 and Swift 3
##在iOS 10中使用Swift 3与UIViewPropertyAnimator设计动画

***

>* 原文链接 : [Designing Animations with UIViewPropertyAnimator in iOS 10 and Swift 3](http://jamesonquave.com/blog/designing-animations-with-uiviewpropertyanimator-in-ios-10-and-swift-3/)
* 原文作者 : [Jason Newell](http://jamesonquave.com/blog/author/newell-jasonb/)
* 译者 : [yrq110](https://github.com/yrq110)

***

iOS 10中的UIKit中加入了一些新的对象与协议，具有“基于对象、完全交互，支持可中断动画来保持对动画的控制，使用基于手势的交互来连接”等特点。

简而言之就是为了给开发者可扩展的、更上层的控制，比如一些时间函数，更方便地去取消、回放、暂停与重启动画, and change the timing and duration on the fly with smoothly interpolated results. 这些动画的功能同样适用于view controller的变换。

我会简单的介绍一些基本用法，并且关注一些文档与讨论中未涉及的关键点。

## 构建基础App

待会儿我们要尝试一些UIViewPropertyAnimator的特性，不过首先需要一些动画。

创建一个单视图应用，在ViewContrller.swift中添加如下代码。

```swift
import UIKit

class ViewController: UIViewController {
    // this records our circle's center for use as an offset while dragging
    var circleCenter: CGPoint!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Add a draggable view
        let circle = UIView(frame: CGRect(x: 0.0, y: 0.0, width: 100.0, height: 100.0))
        circle.center = self.view.center
        circle.layer.cornerRadius = 50.0
        circle.backgroundColor = UIColor.green()

        // add pan gesture recognizer to
        circle.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: #selector(self.dragCircle)))

        self.view.addSubview(circle)
    }

    func dragCircle(gesture: UIPanGestureRecognizer) {
        let target = gesture.view!

        switch gesture.state {
        case .began, .ended:
            circleCenter = target.center
        case .changed:
            let translation = gesture.translation(in: self.view)
            target.center = CGPoint(x: circleCenter!.x + translation.x, y: circleCenter!.y + translation.y)
        default: break
        }
    }
}
```
这儿没啥难的。在viewDidLoad中创建一个绿色的圆将其定位在屏幕中央，之后绑定一个UIPanGestureRecognizer手势，使dragCircle可以响应在它身上的拖动事件:在屏幕上移动圆圈。如你所想，结果就是一个可拖拽的圆:

![](http://i0.wp.com/jamesonquave.com/blog/wp-content/uploads/Rev-1.png?resize=432%2C702)

## 关于UIViewPropertyAnimator

我们主要使用UIViewPropertyAnimator类给view属性添加动画效果、中断动画，与改变动画进行过程中的状态等。UIViewPropertyAnimator实例持有一个动画集，并且可以随时添加进新的动画。

> 注意: “UIViewPropertyAnimator实例”有点拗口，下面我会使用animator来表示它。

若有多个动画改变了一个view的同一个属性，则之后添加或开始<sup>\*</sup>的动画“胜出”(ps:应该是有效的意思)。有趣的是不会发生不协调的变化，而是将新旧动画结合在了一起，当旧动画淡出时新动画淡入。

\* 后来向UIViewPropertyAnimator实例中添加的动画或者是有一个开始延迟覆盖了之前的动画。"后开始者"胜出。

## 可中断, 可逆的动画

来深入研究下，做个简单的动画。这个动画的效果是：当圆圈被拖拽时，会慢慢扩大到原始尺寸的两倍，一松手就会缩回原始大小。

首先，在类中添加animator属性与一个持续时间:

```swift
// ...
class ViewController: UIViewController {
    // We will attach various animations to this in response to drag events
    var circleAnimator: UIViewPropertyAnimator!
    let animationDuration = 4.0
// ...
```

需要在viewDidLoad中初始化animator:

```swift
// ...
// animations argument is optional
circleAnimator = UIViewPropertyAnimator(duration: animationDuration, curve: .easeInOut, animations: {
[unowned circle] in
    // 2x scale
    circle.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
})
// self.view.addSubview(circle) here
// ...
```

当我们初始化circleAnimator时将duration与curve属性作为参数输入。curve可以设置为4个预置的时间变化曲线，在这个例子中选用easeInOut，其它的还有easeIn, easeOut和linear。还需要输入一个倍增圆圈尺寸的闭包函数。

现在需要一个触发动画的方法。使用下面的代码替换dragCircle方法的内容，当用户开始拖拽时开始动画，通过设定circleAnimator.isReversed的值来控制动画的方向。

```swift
func dragCircle(gesture: UIPanGestureRecognizer) {
    let target = gesture.view!

    switch gesture.state {
    case .began, .ended:
        circleCenter = target.center

        if (circleAnimator.isRunning) {
            circleAnimator.pauseAnimation()
            circleAnimator.isReversed = gesture.state == .ended
        }
        circleAnimator.startAnimation()

        // Three important properties on an animator:
        print("Animator isRunning, isReversed, state: \(circleAnimator.isRunning), \(circleAnimator.isReversed), \(circleAnimator.state)")
    case .changed:
        let translation = gesture.translation(in: self.view)
        target.center = CGPoint(x: circleCenter!.x + translation.x, y: circleCenter!.y + translation.y)
    default: break
    }
}
```

执行动画，让"圆圈"呼吸起来，按下一段时间试试..

### 一个关键点

在动画执行后:

![](http://i0.wp.com/jamesonquave.com/blog/wp-content/uploads/Rev-2.png?resize=404%2C674)

没有变化，而是卡在了扩大后的尺寸。

那么到底发生了什么? 简而言之就是当动画完成时丢弃了引用动画。

Animators有三种状态:
* inactive: 初始状态。当动画达到终点时animator会返回这个状态(可转换为active)。
* active: 动画运行时的状态(可转换为stopped或inactive)。
* stopped: 当调用stopAnimation:方法时animator的状态(返回到inactive)。

如图所示:

![](http://i0.wp.com/jamesonquave.com/blog/wp-content/uploads/apple-states.png?resize=584%2C370)

(来源 [UIViewAnimating protocol reference](https://developer.apple.com/reference/uikit/uiviewanimating))

任何指向inactive状态的转换会使所有animator中的动画被清除。

尝试了startAnimation方法，接下来研究一下其它两个。

为了让圆圈动起来，需要改变circleAnimator的初始化代码:

```swift
expansionAnimator = UIViewPropertyAnimator(duration: expansionDuration, curve: .easeInOut)
```

…修改dragCircle:

```swift
// ...
// dragCircle:
case .began, .ended:
    circleCenter = target.center

    if circleAnimator.state == .active {
        // reset animator to inactive state
        circleAnimator.stopAnimation(true)
    }

    if (gesture.state == .began) {
        circleAnimator.addAnimations({
            target.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
        })
    } else {
        circleAnimator.addAnimations({
            target.transform = CGAffineTransform.identity
        })
    }

case .changed:
// ...
```
现在，当用户开始或停止拖拽时，停止或终止animator(如果它是active状态的话)，animator会清除所有关联动画返回到inactive状态。这样我们就可以关联一个新的动画向圆圈发送期望的最终状态。

使用变换来改变视图样式的一个好处就是你可以通过CGAffineTransform.identity来轻松地重置视图的样式，不需要保留旧值。

注意，circleAnimator.stopAnimation(true)与下面的代码是等效的:

```swift
circleAnimator.stopAnimation(false) // 未完成 (停留在stopped状态)
circleAnimator.finishAnimation(at: .current) // 设置视图的实际属性为这一刻的变换值
```

finishAnimationAt:方法接受一个UIViewAnimatingPosition值。若输入start或end，则圆圈最终会变换为起始动画或终止动画。

### 关于持续时间

这里有一个微妙的小bug，每次我们停止一个动画并开始一个新的时，不论视图离最终变换的目标还差多少，新的动画都会持续4秒。

可以这样来修复:

```swift
// dragCircle:
// ...
case .began, .ended:
    circleCenter = target.center

    let durationFactor = circleAnimator.fractionComplete // Multiplier for original duration
    // multiplier for original duration that will be used for new duration
    circleAnimator.stopAnimation(false)
    circleAnimator.finishAnimation(at: .current)

    if (gesture.state == .began) {
        circleAnimator.addAnimations({
            target.backgroundColor = UIColor.green()
            target.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
        })
    } else {
        circleAnimator.addAnimations({
            target.backgroundColor = UIColor.green()
            target.transform = CGAffineTransform.identity
        })
    }

    circleAnimator.startAnimation()
    circleAnimator.pauseAnimation()
    // set duration factor to change remaining time
    circleAnimator.continueAnimation(withTimingParameters: nil, durationFactor: durationFactor)
case .changed:
// ...
```
现在，我们显式的停止了animator，根据方向来关联2个中的一个动画，并重启animator，使用continueAnimationWithTimingParameters:durationFactor:调整剩余的持续时间。这样的话就不会以完整的时间执行原始动画，而是一个`瘦身`过的短时间的扩展动画。continueAnimationWithTimingParameters:durationFactor:方法也被用来修改animator的timing函数<sup>\*</sup>。

\* 当输入一个新的timing函数时，旧timing函数中的变换会被插入进来。若将一个弹性timing函数变成一个线性的话，举个例子，动画在变得平滑之前会保持`弹性`一段时间。

## Timing函数

新的timing函数比之前的要好很多。

旧的 **UIViewAnimationCurve** 参数依旧可用(比如之前用过的easeInOut)，这里有两个新的可用timing对象 : **UISpringTimingParameters** 与 **UICubicTimingParameters**

### UISpringTimingParameters

要使用UISpringTimingParameters实例需要设置衰减率(damping ratio)、质量(mass)、刚度(stiffness)与初始速率(initial velocity)，回使用这些参数通过一些公式计算来得到一个逼真的弹性动画。animator在初始化时可以同时接受UISpringTimingParameters和duration参数，不过后者会被忽略掉，不会参与公式计算，这解决了一些以前的sping动画函数中的弊病。

来做些不同的事情，使用sping animator把圆圈吸附在屏幕的中央。

ViewController.swift
```swift
import UIKit

class ViewController: UIViewController {
    // this records our circle's center for use as an offset while dragging
    var circleCenter: CGPoint!
    // We will attach various animations to this in response to drag events
    var circleAnimator: UIViewPropertyAnimator?

    override func viewDidLoad() {
        super.viewDidLoad()

        // Add a draggable view
        let circle = UIView(frame: CGRect(x: 0.0, y: 0.0, width: 100.0, height: 100.0))
        circle.center = self.view.center
        circle.layer.cornerRadius = 50.0
        circle.backgroundColor = UIColor.green()

        // add pan gesture recognizer to circle
        circle.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: #selector(self.dragCircle)))

        self.view.addSubview(circle)
    }

    func dragCircle(gesture: UIPanGestureRecognizer) {
        let target = gesture.view!

        switch gesture.state {
        case .began:
             if circleAnimator != nil && circleAnimator!.isRunning {
                circleAnimator!.stopAnimation(false)
            }
            circleCenter = target.center
        case .changed:
            let translation = gesture.translation(in: self.view)
            target.center = CGPoint(x: circleCenter!.x + translation.x, y: circleCenter!.y + translation.y)
        case .ended:
            let v = gesture.velocity(in: target)
            // 500 is an arbitrary value that looked pretty good, you may want to base this on device resolution or view size.
            // The y component of the velocity is usually ignored, but is used when animating the center of a view
            let velocity = CGVector(dx: v.x / 500, dy: v.y / 500)
            let springParameters = UISpringTimingParameters(mass: 2.5, stiffness: 70, damping: 55, initialVelocity: velocity)
            circleAnimator = UIViewPropertyAnimator(duration: 0.0, timingParameters: springParameters)

            circleAnimator!.addAnimations({
                target.center = self.view.center
            })
            circleAnimator!.startAnimation()
        default: break
        }
    }
}
```

拖拽圆圈然后松开，它不仅会弹回初始点的位置，并且依旧会保持松手时的动量。这是由于在初始化sping timing的参数时给initialVelocity设置了一个速率:

```swift
// dragCircle: .ended:
// ...
let velocity = CGVector(dx: v.x / 500, dy: v.y / 500)
let springParameters = UISpringTimingParameters(mass: 2.5, stiffness: 70, damping: 55, initialVelocity: velocity)
circleAnimator = UIViewPropertyAnimator(duration: 0.0, timingParameters: springParameters)
// ...
```

![](http://i2.wp.com/jamesonquave.com/blog/wp-content/uploads/Animations-2-1.png?resize=320%2C533)

- 为了体现动画执行的路径，每隔一段时间就在主圆圈的中心点绘制一个小圆圈。可以看出，“直线”路径有些弯曲，因为在松开圆圈时会保留一些动量，并且由于弹性被拉向屏幕中心。

### UICubicTimingParameters

UICubicTimingParameters允许你添加自定义三次贝塞尔曲线的控制点，需要注意的是超过 0.0 - 1.0 范围的坐标点会被调整到这个范围内:

```swift
// Same as setting the y arguments to 0.0 and 1.0 respectively
let curveProvider = UICubicTimingParameters(controlPoint1: CGPoint(x: 0.2, y: -0.48), controlPoint2: CGPoint(x: 0.79, y: 1.41))
expansionAnimator = UIViewPropertyAnimator(duration: expansionDuration, timingParameters: curveProvider)
```

若你不喜欢这些提供的timing曲线，可以在满足UITimingCurveProvider协议的基础上来实现自己想要的时序曲线。

## 擦除动画

可以手动设置一个动画暂停的进度，通过使用animator的fractionComplete属性，输入一个0.0 - 1.0<sup>\*</sup>范围内的值。比如说值为0.5的话，则会将动画属性折半赋给最终的值，并且会忽略timing曲线。注意你设置的位置会被映射到当你重启动画时的timing曲线，因此值为0.5的fractionComplete属性并不意味着剩余的持续时间等于原始持续时间。

来做个别的例子。首先，在viewDidLoad:顶部初始化animator，然后添加两个动画:
```swift
// viewDidLoad:
// ...
circleAnimator = UIViewPropertyAnimator(duration: 1.0, curve: .linear, animations: {
    circle.transform = CGAffineTransform(scaleX: 3.0, y: 3.0)
})

circleAnimator?.addAnimations({
    circle.backgroundColor = UIColor.blue()
}, delayFactor: 0.75)
// ...
```
这次没有使用startAnimation，当动画进行时圆圈会逐渐变大并且在进行到75%的进度时变为蓝色。

同样需要实现一个新的dragCircle:方法:
```swift
func dragCircle(gesture: UIPanGestureRecognizer) {
    let target = gesture.view!

    switch gesture.state {
    case .began:
        circleCenter = target.center
    case .changed:
        let translation = gesture.translation(in: self.view)
        target.center = CGPoint(x: circleCenter!.x + translation.x, y: circleCenter!.y + translation.y)

        circleAnimator?.fractionComplete = target.center.y / self.view.frame.height
    default: break
    }
}
```
现在当拖拽圆圈时会根据它垂直方向的坐标来更新animator的fractionCompleteNow属性:


![](http://i2.wp.com/jamesonquave.com/blog/wp-content/uploads/Rev-3.png?resize=432%2C702)
![](http://i2.wp.com/jamesonquave.com/blog/wp-content/uploads/Rev-4.png?resize=432%2C702)

这里我用的是线性的timing曲线，不过在这个例子中用其它或者提供的timing曲线的话效果会更好。圆圈颜色变蓝的动画伴随一个压缩版的animatortiming曲线。

\* 若自定义的animator支持越过端点的动画，则可以接受超出0.0 - 1.0范围外的值。
