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

## A Sticking Point

Take a look at this video of our circle after it has made it all the way to the end of the animation:

![](http://i0.wp.com/jamesonquave.com/blog/wp-content/uploads/Rev-2.png?resize=404%2C674)

It’s not moving. It’s stuck in at the expanded size.

Ok, so what’s happening here? The short answer is that the animation threw away the reference it had to the animation when it finished.

Animators can be in one of three states:
* inactive: the initial state, and the state the animator returns to after the animations reach an end point (transitions to active)
* active: the state while animations are running (transitions to stopped or inactive)
* stopped: a state the animator enters when you call the stopAnimation: method (returns to inactive)

Here it is, represented visually:

![](http://i0.wp.com/jamesonquave.com/blog/wp-content/uploads/apple-states.png?resize=584%2C370)

(source: UIViewAnimating protocol reference)

Any transition to the inactive state will cause all animations to be purged from the animator (along with the animator’s completion block, if it exists).

We’ve already seen the startAnimation method, and we’ll delve into the other two shortly.

Let’s get our circle unstuck. We need to change up the initialization of circleAnimator:

```swift
expansionAnimator = UIViewPropertyAnimator(duration: expansionDuration, curve: .easeInOut)
```

…and modify dragCircle::

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
Now, whenever the user starts or stops dragging, we stop and finalize the animator (if it’s active). The animator purges the attached animation and returns to the inactive state. From there, we attach a new animation that will send our circle towards the desired end state.

A nice benefit of using transforms to change a view’s appearence is that you can reset the view’s appearance easily by setting its transform property to CGAffineTransform.identity. No need to keep track of old values.

Note that circleAnimator.stopAnimation(true) is equivalent to:

```swift
circleAnimator.stopAnimation(false) // don't finish (stay in stopped state)
circleAnimator.finishAnimation(at: .current) // set view's actual properties to animated values at this moment
```

The finishAnimationAt: method takes a UIViewAnimatingPosition value. If we pass start or end, the circle will instantly transform to the scale it should have at the beginning or end of the animation, respectively.

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
