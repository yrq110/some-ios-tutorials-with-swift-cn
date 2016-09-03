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

## About UIViewPropertyAnimator

UIViewPropertyAnimator is the main class we’ll be using to animate view properties, interrupt those animations, and change the course of animations mid-stream. Instances of UIViewPropertyAnimator maintain a collection of animations, and new animations can be attached at (almost) any time.

Note: “UIViewPropertyAnimator instance” is a bit of a mouthful, so I’ll be using the term animator for the rest of this tutorial.

If two or more animations change the same property on a view, the last animation to be added or started* “wins”. The interesting thing is than rather than an jarring shift, the new animation is combined with the old one. It’s faded in as the old animation is faded out.

> Animations which are added later to a UIViewPropertyAnimator instance, or are added with a start delay override earlier animations. Last-to-start wins.

## Interruptable, Reversible Animations

Let’s dive in and add a simple animation to build upon later. With this animation, the circle will slowly expand to twice it’s original size when dragged. When released, it will shrink back to normal size.

First, let’s add properties to our class for the animator and an duration:

```swift
// ...
class ViewController: UIViewController {
    // We will attach various animations to this in response to drag events
    var circleAnimator: UIViewPropertyAnimator!
    let animationDuration = 4.0
// ...
```

Now we need to initialize the animator in viewDidLoad::

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

When we initialized circleAnimator, we passed in arguments for the duration and curveproperties. The curve can be set to one of four simple, predefined timing curves. In our example we choose easeInOut. The other options are easeIn, easeOut, and linear. We also passed an animations closure which doubles the size of our circle.

Now we need a way to trigger the animation. Swap in this implementation of dragCircle:. This version starts the animation when the user begins dragging the circle, and manages the animation’s direction by setting the value of circleAnimator.isReversed.

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

Run this version. Try to make the circle “breathe”. Hold it down for a second..
