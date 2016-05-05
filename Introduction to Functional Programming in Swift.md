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

![x](http://www.raywenderlich.com/wp-content/uploads/2015/08/x3.png)

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
你将会写出你的第一个函数，在swift的String使用NSString的方法，因此需要在playground的头部添加Foundation框架:
~~~~
import Foundation
~~~~
假设你需要一个按字母表排序的设施名称列表，使用命令式的方法来实现的话，添加如下函数到你的playground底部:
~~~~
func sortedNames(rides: [Ride]) -> [String] {
  var sortedRides = rides
  var i, j : Int
  var key: Ride
 
  // 1
  for (i = 0; i < sortedRides.count; i++) {
    key = sortedRides[i]
 
    // 2
    for (j = i; j > -1; j--) {
      if key.name.localizedCompare(sortedRides[j].name) == .OrderedAscending {
        sortedRides.removeAtIndex(j + 1)
        sortedRides.insert(key, atIndex: j)
      }
    }
  }
 
  // 3
  var sortedNames = [String]()
  for ride in sortedRides {
    sortedNames.append(ride.name)
  }
 
  print(sortedRides)
 
  return sortedNames
}
~~~~
这里你做了:

1. 在函数内遍历设施
2. 执行插入排序
3. 取得排序后设施中的名称

从sortedNames(:)的角度来看，它接收了设施的列表，然后输出了排序后的包含设施名称的列表。sortedNames(:)外面的东西没有受到影响，为了证明，首先我们打印出排序后的设施名称:
~~~~
print(sortedNames(parkRides))
~~~~
然后收集设施列表中的名称并打印出来:
~~~~
var originalNames = [String]()
for ride in parkRides {
  originalNames.append(ride.name)
}
 
print(originalNames)
~~~~
在编辑器中你会发现sortedNames(:)中对设施的排序并没有改变输入的设施列表。你使用命令式代码创建的这个模块函数被认为是准函数式(quasi-functional)的。

即使如此，用命令式语句写出的函数依旧很冗长、笨拙。如果有一种技术可以通过使用像sortedNames(:)这样的函数更进一步的简化代码岂不是很棒？

###第一类与高阶函数
另一个函数式编程的特点就是函数们。第一类函数是其中一种，它意味着可以被分配给一个值，可以作为其它函数的输入与输出。再往上就是高阶函数，它可以将函数作为参数并返回其它函数。

![first-class](http://www.raywenderlich.com/wp-content/uploads/2015/08/firstclass.png)

在这一节中，你将会学习到函数式编程中几个常见的高阶函数——filter、map与reduce。

####Filter
在swift中，filter(:)是CollectionType型值的一个方法，比如说数组(array),它可以将另一个函数作为参数输入返回数组中应用这个函数返回为真的值。

filter(:) 将输入的函数应用到数组中的每一个元素上，返回另一个包含返回值为真的所有元素的数组。

回想一下你的sortedNames做过什么，用声明式的思想去思考一下而不是命令式。

注释掉sortedNames，你将会更有效率的重写它。在playground底部创建一个将Ride作为参数的函数:
~~~~
func waitTimeIsShort(ride: Ride) -> Bool {
  return ride.waitTime < 15.0
}
~~~~
waitTimeIsShort(:)接受一个ride，如果等待时间小于15秒则返回true，否则返回false。

在parkRides上调用filter然后输入你刚刚创建的函数:
~~~~
var shortWaitTimeRides = parkRides.filter(waitTimeIsShort)
print(shortWaitTimeRides)
~~~~
你把shortWaitTimeRides定义为一个变量因为你会在playground中重用它，会在其它地方访问。在playground的输出中，你会看到filter输出中的Crazy Funhouse与Mountain Railroad是正确的结果。

在filter的尾部使用一个闭包(closure)函数可以得到同样的结果:
~~~~
shortWaitTimeRides = parkRides.filter { $0.waitTime < 15.0 }
print(shortWaitTimeRides)
~~~~
在这里，.filter()会去除parkRides数组中的每一个ride，观察它的waitTime属性并计算它是否小于15。你声明式地告诉程序你想让它做什么而不是怎么做，这一开始看起来有点神秘。
####Map
CollectionType类型的方法map(:)也接受一个函数作为参数，在对每一个参数中的元素进行操作后返回一个与之前相同长度的数组。map函数的返回值类型可以不与参数元素的类型相同。

对parkRides数组中的元素执行map，得到一个包含所有设施名称字符串的列表:
~~~~
let rideNames = parkRides.map { $0.name }
print(rideNames)
~~~~
你可以像下面那样排序并打印出名称列表，使用sort来进行MutableCollectionType类型的排序:
~~~~
print(rideNames.sort(<))
~~~~
你的sortedNames(:)方法得益于map、sort与一个闭包，现在变成了仅仅两行！
####Reduce
CollectionType类型方法reduce(:,:)接受两个参数。第一个是一个类型T的值，第二个是函数，它将T的值与另一个元素融合产生一个具有新值的T，输入函数会处理调用的集合中的每一个元素。

假设你想知道公园里所有设施的平均等待时间。

在reduce中输入值0.0，然后使用一个闭包函数对每一种设施等待时间与设施总数的比值求和。
~~~~
let averageWaitTime = parkRides.reduce(0.0) { (average, ride) in average + (ride.waitTime/Double(parkRides.count)) }
print(averageWaitTime)
~~~~
在这个例子中，最初的两次迭代如下所示:

| iteration | average | ride.waitTime / Double(parkRides.count) | resulting average ||—|—|—|—||1|0.0|45.0 / 8.0 = 5.625 | 0.0 + 5.625 = 5.625||2|5.625|10.0 / 8.0 = 1.25 | 5.625 + 1.25 = 6.875|

如你所见，平均值的结果使用不断迭代的平均值。这个过程会持续到遍历完parkRides中的每一个设施才会结束。它使你只用了一行代码就能得到平均值！

###柯里化
>注意。随着现在swift的开源，在swift3.0中貌似会移除柯里化函数语句。你可以在[这里](https://github.com/apple/swift-evolution/blob/master/proposals/0002-remove-currying.md)找到信息，尤其是**Detailed design**部分

函数式编程中又一个高级特性就是柯里化，由数学家Haskell Curry得名。柯里化的思想是将接受多个参数的函数作为一个接受单个参数的函数应用的序列。

![](http://www.raywenderlich.com/wp-content/uploads/2015/08/curry.png)

明确一下，在这篇教程中柯里化不是一个能吃出喜悦泪水的咖喱，不过你可以通过创建一个使用括号将两个参数分开的双参数函数来实践一下柯里化:
~~~~
func rideTypeFilter(type: RideType)(fromRides rides: [Ride]) -> [Ride] {
  return rides.filter { $0.types.contains(type) }
}
~~~~

这里，rideTypeFilter(:)(:)接受了一个设施类型作为第一个参数，一个设施的数组作为第二个参数。

构造另一个函数是柯里化的一个较好应用。使用单个参数来调用你的柯里化函数，然后构造一个单个参数的函数。

在你的playground中添加这个设施类型的过滤器，动手试试:
~~~~
func createRideTypeFilter(type: RideType) -> [Ride] -> [Ride] {
  return rideTypeFilter(type)
}
~~~~
注意到createRideTypeFilter返回映射一个ride数组到另一个ride数组的函数，通过调用一个单个参数的rideTypeFilter。使用createRideTypeFilter 为孩子们创建一个过滤器，然后使用过滤器过滤你的公园设施表，像这样:
~~~~
let kidRideFilter = createRideTypeFilter(.Kids)
 
print(kidRideFilter(parkRides))
~~~~
在playground的输出中，你会看到kidRideFilter过滤掉了所有非孩童类的设施。
###纯函数
