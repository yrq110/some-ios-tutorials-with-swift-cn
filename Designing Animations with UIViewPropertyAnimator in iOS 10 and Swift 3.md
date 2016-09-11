#Designing Animations with UIViewPropertyAnimator in iOS 10 and Swift 3
##在iOS 10中Swift 3中与UIViewPropertyAnimator设计动画

***

>* 原文链接 : [Designing Animations with UIViewPropertyAnimator in iOS 10 and Swift 3](http://jamesonquave.com/blog/designing-animations-with-uiviewpropertyanimator-in-ios-10-and-swift-3/)
* 原文作者 : [Jason Newell](http://jamesonquave.com/blog/author/newell-jasonb/)
* 译者 : [yrq110](https://github.com/yrq110)

***

这篇教程中介绍了iOS 10中的新特性，Swift语言与Xcode 8 beta，都是WWDC 16上刚刚发布的。

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

若有多个动画改变了一个view的同一个属性，则之后添加或开始\*的动画“胜出”(ps:应该是有效的意思)。有趣的是不会发生不协调的变化，而是将新旧动画结合在了一起，当旧动画淡出时新动画淡入。

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

## 一个关键点

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

任何向inactive状态的转换会使所有animator中的动画被清除(along with the animator’s completion block, if it exists)。

我们已经见过了startAnimation方法，接下来研究一下其它两个。

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

## About Durations

There’s a subtle bug in this version. The problem is, every time we stop an animation and start a new one, the new animation will take 4.0 seconds to complete, no matter how close the view is to reaching the end goal.

Here’s how we can fix it:

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
Now, we explicitly stop the animator, attach one of two animations depending on the direction, and restart the animator, using continueAnimationWithTimingParameters:durationFactor:to adjust the remaining duration. This is so that “deflating” from a short expansion does not take the full duration of the original animation. The method continueAnimationWithTimingParameters:durationFactor:can also be used to change an animator’s timing function on the fly*.

* When you pass in a new timing function, the transition from the old timing function is interpolated. If you go from a springy timing function to a linear one, for example, the animations may remain “bouncy” for a moment, before smoothing out.
