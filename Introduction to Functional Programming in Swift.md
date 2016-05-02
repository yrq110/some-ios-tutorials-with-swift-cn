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
![x](http://www.raywenderlich.com/wp-content/uploads/2015/08/x3.png)

![first-class](http://www.raywenderlich.com/wp-content/uploads/2015/08/firstclass.png)
