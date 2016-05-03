#Introduction to Functional Programming in Swift
##Swift函数式编程介绍
[原文地址](https://www.raywenderlich.com/114456/introduction-functional-programming-swift)

![title](https://cdn2.raywenderlich.com/wp-content/uploads/2015/08/intro-250x250.png)

在2014年WWDC上隆重登场的swift不仅仅是作为一门新的语言，它为iOS与OS X平台带来了许多新的特性。

这篇教程会针对其中的FP(Functional Programming)，介绍函数式的思想与技术。

>注意。如果你想深入了解swift的函数式编程，可以看看[Swift by Tutorials](http://www.raywenderlich.com/store/swift-by-tutorials)中的函数式编程章节，[这里](http://www.raywenderlich.com/82599/swift-functional-programming-tutorial)也可以。

##入门
在Xcode中创建一个新的playground命名为IntroFunctionalProgramming，其他选项默认，选择iOS平台，点击create创建。

Keep in mind that this tutorial will cover FP at a high level, 所以将这些概念想象成现实中的场景会很有帮助。在这里，想象一下你在为一个游乐园做一个app，游乐园的游乐设施数据由远程服务器的API提供。

使用如下代码替换playground中的模板代码:
~~~~
enum RideType {
  case Family
  case Kids
  case Thrill
  case Scary
  case Relaxing
  case Water
}
 
struct Ride {
  let name: String
  let types: Set<RideType>
  let waitTime: Double
}
~~~~
这里你创建了一个叫RideType的枚举包含了一些游乐设施的类型，还有一个叫Ride的结构体，包含设施的名称，类型的集合与等待时间。

现在你也许会想象出玩海盗船时失重的恐怖，过山车的激情。不过相信我，没有一种设施的感觉像从其他编程风格到函数式编程的转换。

##什么是函数式编程？
函数式编程与其他表现方式的区别常常被描述为命令式与声明式思想的区别。

* 命令式思想在解决问题时思考的是算法与步骤，或者说如何从A(方法)到B(解决方案)。一个典型的命令式的例子：一个纸杯蛋糕的食谱，告诉你放多少材料，怎么去混合，怎么去烘焙。
* 声明式思想分析一个问题考虑解决方案是什么，而不是如何得到解决方案。用一个食谱的例子来说明这个概念。一个声明式的食谱会给你一张纸杯蛋糕的图片并且有一段描述，而不是告诉你如何混合成分与烘焙。

函数式编程鼓励使用基于函数的声明方法来解决问题。在这篇教程中你将会看到更多的命令式与声明式编程的区别。

##函数式编程概念
在这一节中，你将会学习函数式编程中的重要概念。许多函数式编程的专著中都挑选了不变状态(immutable state)与无副作用(lack of side effects)作为最重要的方面，为何我们不从这里开始呢？
###不变性与副作用
不管你最开始学的什么编程语言，接触到的第一个概念应该就是用变量来表现数据。当你回到这种表现的最初理念时，变量就会变得很古怪。

在你的playground中添加如下看起来理所当然的代码:
~~~~
var x = 3
// other stuff...
x = 4
~~~~
作为一个开发者在学习面向过程或面向对象时就是这么讲的，你不会花一点时间会思考它。不过一个量如何能先等于3然后等于4？

变量意为一个变化的量。考虑数学中的量x，你会在软件行为中将时间作为关键参数。你在改变变量时创造了可变的状态。

若在一个相对简单的系统中，可变状态也许不是一个大问题。不过当连接多个对象时，像一个庞大的OOP系统，可变状态就会很让人头痛。

例如，当2个或多个线程同时访问一个变量时，它们也许在访问或修改时出错，出现不期望的行为。像竞态条件、锁死等问题。

想象一下你可以在一个状态从不改变的地方写代码。在并发系统中的大量问题都会消失———poof！你可以创建一个不变属性来实现它，或者数据在程序运行期间不允许更改。

在objective-C中可以通过一个常量来实现，不会你的属性在创建时默认是可变模式。在swift中默认是不变的。

使用不变数据的优点就是在代码单元中使用时将会没有副作用，意味着你代码中函数不会改变它们外部的元素，当调用函数时不会发生可怕的现象。

在你playground的Ride结构体后面添加如下数组，在这篇教程中使用一个不变的swift常量来表现你的主要数据，看看会发生什么:
~~~~
let parkRides = [
  Ride(name: "Raging Rapids", types: [.Family, .Thrill, .Water], waitTime: 45.0),
  Ride(name: "Crazy Funhouse", types: [.Family], waitTime: 10.0),
  Ride(name: "Spinning Tea Cups", types: [.Kids], waitTime: 15.0),
  Ride(name: "Spooky Hollow", types: [.Scary], waitTime: 30.0),
  Ride(name: "Thunder Coaster", types: [.Family, .Thrill], waitTime: 60.0),
  Ride(name: "Grand Carousel", types: [.Family, .Kids], waitTime: 15.0),
  Ride(name: "Bumper Boats", types: [.Family, .Water], waitTime: 25.0),
  Ride(name: "Mountain Railroad", types: [.Family, .Relaxing], waitTime: 0.0)
]
~~~~
当你使用let创建parkRides而不是var时，这个数组及它的项就是不可变的。使用如下方法尝试改变数组中的一项:
~~~~
parkRides[0] = Ride(name: "Functional Programming", types: [.Thrill], waitTime: 5.0)
~~~~
改变其中的项会发生编译错误。去修改那些ride吧！没门儿，小鬼！
##模块化
![x](http://www.raywenderlich.com/wp-content/uploads/2015/08/x3.png)

![first-class](http://www.raywenderlich.com/wp-content/uploads/2015/08/firstclass.png)
