#What’s New in Swift 3
##Swift 3中的新特性

[原文地址](http://www.appcoda.com/swift3-changes/)

译者：[yrq110](https://github.com/yrq110)

苹果在Xcode 8 beta版中集成了Swift 3，并且将在今年年末发布最终版本。这是3的第一个版本，开源并且可运行于MacOS与Linux。如果你从去年12月开始就跟进[Swift-Evolution](https://github.com/apple/swift-evolution)的话，甚至包括已经在[IBM沙盒](https://swiftlang.ng.bluemix.net/#/repl)中体验过的boy，应该知道这里面有很多的变化。如果现在在Xcode8中编译你的app的话，肯定会崩溃的。

Swift 3中的变化可以被分为两类:

* 移除Swift 2.2中已经被弃用的功能
* 语言的现代化问题

从被移除的条目开始，这个比较容易理解，在Xcode7.3中使用它们应该已经出现警告了。

##++ 与 -- 运算符

继承自C语言的自增与自减的操作符与它的功能是显而易见的，对一个量进行加1或减1:
````swift
var i = 0
i++
++i
i--
--i
````
然而，当遇到该选择哪个值进行操作时(运算前和运算后)事情会变复杂。每种操作符都有两种风格：前缀和后缀，它们都是内部函数，得益于操作符重载它会返回一个你可能会使用也可能丢弃的值。

这对初学者来说很不友好，因此需要被移除，转而使用加法(+=)与减法(-=)赋值运算符:
````swift
var i = 0
i += 1
i -= 1
````
当然，你也可以使用常规的加法(+)与减法(-)运算符，虽然使用复合赋值运算符的话更加简便:
````swift
i = i + 1
i = i - 1
````
>延伸阅读: 如果你想了解更多关于这个变化的信息，可以看看 Chris Lattner关于移除++与--操作符的[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md)。

##C语言风格的for循环已成为历史

最常用的增减量运算符就是C语言风格的for循环，这个运算符的移除意味着这种特性也将消失。

如果你有一些编程经历的话，对打印数字1到10这个问题，也许会写出这样的for循环语句:
````swift
for (i = 1; i <= 10; i++) {
  print(i)
}
````
在Swift 3中将不再允许这样做。这是Swift 3中对应的写法 - 注意闭值区间运算符(...)的使用:
````swift
for i in 1...10 {
  print(i)
}
````
你也可以使用带有一个闭包的for-each循环来实现 - 在[这里](https://cosminpupaza.wordpress.com/2015/12/04/for-vs-while-a-beginners-approach/)查看更多关于循环的信息
````swift
(1...10).forEach {
  print($0)
}
````
>延伸阅读: 如果你想了解更多关于这个变化的信息，可以看看Erica Sadun关于移除C风格for循环的[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md)。

##将var从函数参数中移除
通常函数的输入参数都被定义为常量，即在函数体中你不能修改它们，然而某些情况下需要将它们声明为变量才能派上用场。在Swift 2中，你可以使用var关键字将一个函数参数标记为变量，一旦参数被标记为变量，就会在本地创建一个这个值的副本，这样你就可以在函数体内修改它。

举个栗子，下面这个函数会计算出两个数字的最大公约数 - 如果你把数学知识全还给老师了，请读读[这个](https://en.wikipedia.org/wiki/Greatest_common_divisor)
````swift
func gcd(var a: Int, var b: Int) -> Int {
 
  if (a == b) {
    return a
  }
 
  repeat {
    if (a > b) {
      a = a - b
    } else {
      b = b - a
    }
  } while (a != b)
 
  return a
}
````
算法很简单: 如果两个数字相等，则返回它们中的一个；否则比较它们，用较大的数减去较小的数，将结果赋值给较大的那一个变量，反复进行直到它们相等，返回其中的一个。如你所见，将a与b标记为变量后即可在函数中改变它们的值。

Swift 3中将不再允许开发者将函数参数设置为变量，因为这也许会使开发者在var与inout(PS:另一个关键字，自行google)间产生困扰。下一个Swift的版本会将var从函数参数中移除。

因此，若要在Swift 3中写出同样的gcd函数，需要不同的途径来使用。你需要将函数参数保存进本地变量中再进行运算:
````swift
func gcd(a: Int, b: Int) -> Int {
 
  if (a == b) {
    return a
  }
 
  var c = a
  var d = b
 
  repeat {
    if (c > d) {
      c = c - d
    } else {
      d = d - c
    }
  } while (c != d)
 
  return c
}
````
若你想了解这项移除的相关信息，可以查看这个[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0003-remove-var-parameters.md)。

##一致性的函数参数声明方式
函数的参数列表实际上是元组，因此可以通过输入一个元组来调用函数，只要元组的结构与函数参数的结构匹配即可。拿gcd()函数为例，你可以这样调用它:
````swift
gcd(8, b: 12)
````
也可以这么调用:
````swift
let number = (8, b: 12)
gcd(number)
````
如你所见，在Swift 2无需指明第一个参数的标签，不过，第二个以后的参数标签都需要被指明。

这个语句会困扰初学者，因此设计成了规范的标签行为。在Swift 3中，需要这样调用函数:
````swift
gcd(a: 8, b: 12)
````
你需要明确的表明第一个参数的标签，如果不这么做，Xcode 8就会报错。

对于这个变化你的第一个反应可能是“卧槽！现在的代码中要改好多地方啊”。对的，有大量的变动。对于这个苹果提供了一种方法来解决这个问题。你可以在第一个参数的标签前添加一个下划线:
````swift
func gcd(_ a: Int, b: Int) -> Int {
...
}
````
这么做的话，就可以像以前那样调用函数了 - 无需指明第一个标签。这会使你在做Swift 2 到Swift 3的代码移植时会轻松些。

>延伸阅读: 若你想了解这项变化的相关信息，可以查看这个[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0046-first-label.md)。

##选择器字符串将失效
来创建一个按钮，在你点击它时做一些事情。只使用playground，不允许用Interface Builder:
````swift
// 1
import UIKit
import XCPlayground
 
// 2
class Responder: NSObject {
 
  func tap() {
    print("Button pressed")
  }
}
let responder = Responder()
 
// 3
let button = UIButton(type: .System)
button.setTitle("Button", forState: .Normal)
button.addTarget(responder, action: "tap", forControlEvents: .TouchUpInside)
button.sizeToFit()
button.center = CGPoint(x: 50, y: 25)
 
// 4
let frame = CGRect(x: 0, y: 0, width: 100, height: 50)
let view = UIView(frame: frame)
view.addSubview(button)
XCPlaygroundPage.currentPage.liveView = view
````
让我们来逐步分析下:

1. 导入UIKit与XCPlayground框架 - 你需要它们创建按钮然后在playground的辅助编辑器(assistant editor)中显示它。
>注意: 在Xcode中这样启用辅助编辑器: View -> Assistant Editor -> Show Assistant Editor.

2. 定义一个当用户按下按钮时执行的tap方法，创建一个基于NSObject的按钮响应对象 - 为何是NSObject？因为选择器只适用于OC的方法。

3. 声明button按钮并设置属性。

4. 声明view视图与响应框体，在上面添加button，并且在playground的辅助编辑器中显示它。

注意第3部分的第3行代码。button的选择器是一个字符串。如果你敲错了这个字符串，代码会编译成功不过会在runtime时崩溃，因为没有找到任何响应方法。

为了解决这个在编译时的潜在问题，Swift 3使用#selector()来代替字符串选择器。这使得编译器可以在没有得到正确的方法名时提早检测出问题。
````swift
button.addTarget(responder, action: #selector(Responder.tap), for: .touchUpInside)
````
>延伸阅读: 若你想了解这项变化的相关信息，可以查看Doug Gregor的[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0022-objc-selectors.md)。

以上这些是被移除的特性。现在来看看语言的现代化亮点吧。

##Key-paths字符串
这个特性有点像前面那个，不过这个是应用于键值编码(KVC)与键值观察(KVO)的:
````swift
class Person: NSObject {
  var name: String = ""
 
  init(name: String) {
    self.name = name
  }
}
let me = Person(name: "Cosmin")
me.valueForKeyPath("name")
````
创建了一个符合KVC规范的Person类，使用类指定的初始化值来产生我的身份，使用对应的key-path来决定我的名字。检查一下，如果你输错了我会生气的！

幸运的是，在Swift 3中不会发生这种事，Key-path字符串已经被#keyPath()表达式所替代:
````swift
class Person: NSObject {
  var name: String = ""
 
  init(name: String) {
    self.name = name
  }
}
let me = Person(name: "Cosmin")
me.value(forKeyPath: #keyPath(Person.name))
````
>延伸阅读: 若你想了解这项变化的相关信息，可以查看David Hart的[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0062-objc-keypaths.md)。

##取消Foundation类型的NS前缀

JSON解析是一个典型的例子:
````swift
let file = NSBundle.mainBundle().pathForResource("tutorials", ofType: "json")
let url = NSURL(fileURLWithPath: file!)
let data = NSData(contentsOfURL: url)
let json = try! NSJSONSerialization.JSONObjectWithData(data!, options: [])
print(json)
````
使用Foundation类去连接文件并提取JSON数据，通过一种合适的途径: NSBundle -> NSURL -> NSData -> NSJSONSerialization.

Swift 3中取消的了NS前缀, 因此这条路径变成了: Bundle -> URL -> Data -> JSONSerialization:
````swift
let file = Bundle.main().pathForResource("tutorials", ofType: "json")
let url = URL(fileURLWithPath: file!)
let data = try! Data(contentsOf: url)
let json = try! JSONSerialization.jsonObject(with: data)
print(json)
````
>延伸阅读: 关于这个命名规范的变化，可以查看Tony Parker与Philippe Hausler的[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0086-drop-foundation-ns.md)。

##M_PI vs .pi
来根据半径计算一下圆的周长与面积:
````swift
let r =  3.0
let circumference = 2 * M_PI * r
let area = M_PI * r * r
````
之前的Swift版本中，将M_PI作为pi常量使用。Swift 3将pi整合进了Float, Double与CGFloat类型中:
````swift
Float.pi
Double.pi
CGFloat.pi
````
上面的代码在Swift 3中会这么写:
````swift
let r = 3.0
let circumference = 2 * Double.pi * r
let area = Double.pi * r * r
````
由于有类型关联，可以省略掉类型，简便写法:
````swift
let r = 3.0
let circumference = 2 * .pi * r
let area = .pi * r * r
````

##GCD多线程
Grand Central Dispatch(GCD)用于在不阻塞主线程中用户界面情况下进行网络操作。它的底层使用C语言编写，其API的使用对初学者来说有些难度，即使是一些简单的任务，例如创建一个异步队列然后做一些事情:

````swift
let queue = dispatch_queue_create("Swift 2.2", nil)
dispatch_async(queue) {
  print("Swift 2.2 queue")
}
````
Swift 3中去除了所有在面向对象编程的过程中的模板代码与冗余部分:
````swift
let queue = DispatchQueue(label: "Swift 3")
queue.async {
  print("Swift 3 queue")
}
````
>延伸阅读: 关于这个变化，可以查看Matt Wright的[建议](https://github.com/apple/swift-evolution/blob/master/proposals/0088-libdispatch-for-swift3.md)。



##核心图形库(Core Graphics)现在更加Swifty
Core Graphics是一个强大的绘图框架, 不过它与GCD类似，使用C语言风格的API:
````swift
let frame = CGRect(x: 0, y: 0, width: 100, height: 50)
 
class View: UIView {
 
  override func drawRect(rect: CGRect) {
    let context = UIGraphicsGetCurrentContext()
    let blue = UIColor.blueColor().CGColor
    CGContextSetFillColorWithColor(context, blue)
    let red = UIColor.redColor().CGColor
    CGContextSetStrokeColorWithColor(context, red)
    CGContextSetLineWidth(context, 10)
    CGContextAddRect(context, frame)
    CGContextDrawPath(context, .FillStroke)
  }
}
let aView = View(frame: frame)
````
创建了view框体, 扩展了UIView类, 重写了drawRect()方法进行自定义绘制，将新的content赋给了view。

Swift 3中通过一个完全不同的途径进行，首先解绑当前的图形上下文，然后执行所有与之相关的绘制操作:
````swift
let frame = CGRect(x: 0, y: 0, width: 100, height: 50)
 
class View: UIView {
 
  override func draw(_ rect: CGRect) {
    guard let context = UIGraphicsGetCurrentContext() else {
      return
    }
    
    let blue = UIColor.blue().cgColor
    context.setFillColor(blue)
    let red = UIColor.red().cgColor
    context.setStrokeColor(red)
    context.setLineWidth(10)
    context.addRect(frame)
    context.drawPath(using: .fillStroke)
  }
}
let aView = View(frame: frame)
````
>注意: 在view调用它的drawRect()方法之前上下文是空的，因此可以使用[guard](https://cosminpupaza.wordpress.com/2015/11/20/if-vs-guard-a-beginners-approach/)来进行解绑 – 在[这里](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIKitFunctionReference/index.html#//apple_ref/c/func/UIGraphicsGetCurrentContext)了解更多信息。

##动词 vs 名词 命名规范
终于有个跟语法有关的了! Swift 3的group方法分为两类: 返回一个值的方法 – 认为是名词，与执行一个类型的行为的方法 – 认为它们是动词。

这里进行了打印数字从10到1的操作:
````swift
for i in (1...10).reverse() {
  print(i)
}
````
使用reverse()方法对range进行反转。Swift 3将这个操作看做一个名词，返回一个执行了反转指令的range。需要在方法中添加“ed”后缀:
````swift
for i in (1...10).reversed() {
  print(i)
}
````
最常用的元组用法就是打印一个数组的内容:
````swift
var array = [1, 5, 3, 2, 4]
for (index, value) in array.enumerate() {
  print("\(index + 1) \(value)")
}
````
Swift 3将这个方法看做一个名词，因为它返回一个包含数组当前索引与值的元组，因此在方法名后添加“ed”:
````swift
var array = [1, 5, 3, 2, 4]
for (index, value) in array.enumerated() {
  print("\(index + 1) \(value)")
}
````
还有个例子就是数组排序，这里进行了数组升序的排序操作:
````swift
var array = [1, 5, 3, 2, 4]
let sortedArray = array.sort()
print(sortedArray)
````
Swift 3将这个方法看做一个名词，因为它返回一个排序好的数组。sort方法现在叫做sorted:
````swift
var array = [1, 5, 3, 2, 4]
let sortedArray = array.sorted()
print(sortedArray)
````
不使用中间变量来进行数组的就地排序。 在Swift 2中需要这么写:
````swift
var array = [1, 5, 3, 2, 4]
array.sortInPlace()
print(array)
````
使用sortInPlace()对一个可变数组进行排序。Swift 3将这个方法看做一个动词，因为它在执行排序时没有返回任何东西。 它进行的操作仅仅是基础单词所描述的行为，因此 sortInPlace()现在被命名为sort():
````swift
var array = [1, 5, 3, 2, 4]
array.sort()
print(array)
````
>延伸阅读: 对于命名规范的细节，可以查看API的[设计指南](https://swift.org/documentation/api-design-guidelines/).

##Swift风格的API
Swift 3的API运用了简洁的哲♂学理念 – 省略了无用的单词, 因此如果有冗余或可由上下文关联得到的单词就被去掉了:

* `XCPlaygroundPage.currentPage` -> `PlaygroundPage.current`
* `button.setTitle(forState)` -> `button.setTitle(for)`
* `button.addTarget(action, forControlEvents)` -> `button.addTarget(action, for)`
* `NSBundle.mainBundle()` -> `Bundle.main()`
* `NSData(contentsOfURL)` -> `URL(contentsOf)`
* `NSJSONSerialization.JSONObjectWithData()` -> `JSONSerialization.jsonObject(with)`
* `UIColor.blueColor()` -> `UIColor.blue()`
* `UIColor.redColor()` -> `UIColor.red()`

##枚举
Swift 3将枚举的cases看做属性，因此使用小写开头的驼峰式命名，而不是大写开头的驼峰式来标识:

* `.System` -> `.system`
* `.TouchUpInside` -> `.touchUpInside`
* `.FillStroke` -> `.fillStroke`
* `.CGColor` -> `.cgColor`

##@discardableResult 
在Swift 3中，如果你没有使用函数或方法返回值的话Xcode会显示一个警告，举个栗子:

![](http://www.appcoda.com/wp-content/uploads/2016/06/discardable-result-1-1024x217.png)

上面代码中的printMessage方法返回了结果信息，返回值没有被使用。这是个潜在问题，因此Swift 3中的编译器会给出一个警告。

这种情况下没必要使用返回值，你可以在方法声明前添加@discardableResult来避免警告:

````swift
override func viewDidLoad() {
    super.viewDidLoad()
 
    printMessage(message: "Hello Swift 3!")
}
 
@discardableResult
func printMessage(message: String) -> String {
    let outputMessage = "Output : \(message)"
    print(outputMessage)
    
    return outputMessage
}
````
##总结

这就是关于Swift 3的全部内容了。新版的Swift是一个重要的发行版，会使这个语言变得更好，它包含了大量的功能上的变化，对你的现有Swift代码有很大的影响。我希望这篇教程可以帮助你更好的理解这些变化，并且我希望这可以节省一些你移植Siwft项目的时间。

这篇教程中的所有代码都可以在这个[Playground](https://github.com/appcoda/Swift3Playgrounds/blob/master/Swift%203.playground.zip?raw=true)工程中找到，我在Xcode8 beta中测试成功了，确保你使用的是Xcode 8然后再运行这个项目。
