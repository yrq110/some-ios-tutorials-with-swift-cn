#Designing Animations with UIViewPropertyAnimator in iOS 10 and Swift 3
##在iOS 10与Swift 3中使用UIViewPropertyAnimator设计动画

***

>* 原文链接 : [Designing Animations with UIViewPropertyAnimator in iOS 10 and Swift 3](http://jamesonquave.com/blog/designing-animations-with-uiviewpropertyanimator-in-ios-10-and-swift-3/)
* 原文作者 : [Jason Newell](http://jamesonquave.com/blog/author/newell-jasonb/)
* 译者 : [yrq110](https://github.com/yrq110)

***

This is part of a series of tutorials introducing new features in iOS 10, the Swift programming language, and the new XCode 8 beta, which were just announced at WWDC 16

UIKit in iOS 10 now has “new object-based, fully interactive and interruptible animation support that lets you retain control of your animations and link them with gesture-based interactions” through a family of new objects and protocols.

In short, the purpose is to give developers extensive, high-level control over timing functions (or easing), making it simple to scrub or reverse, to pause and restart animations, and change the timing and duration on the fly with smoothly interpolated results. These animation capabilities can also be applied to view controller transitions.

I hope to concisely introduce some of the basic usage and mention some sticking points that are not covered by the talk or documentation.

## Building the Base App

We’re going to try some of the features of UIViewPropertyAnimator in a moment, but first, we need something to animate.

Create a single-view application and add the following to ViewController.swift.

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
There’s nothing too complicated happening here. In viewDidLoad, we created a green circle and positioned in the center of the screen. Then we attached a UIPanGestureRecognizer instance to it so that we can respond to pan events in dragCircle: by moving that circle across the screen. As you may have guessed, the result is a draggable circle:

![](http://i0.wp.com/jamesonquave.com/blog/wp-content/uploads/Rev-1.png?resize=432%2C702)
