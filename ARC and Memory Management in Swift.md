# ARC and Memory Management in Swift
## Swift中的ARC与内存管理

***

>* 原文链接 : [ARC and Memory Management in Swift](https://www.raywenderlich.com/134411/arc-memory-management-swift)
* 原文作者 : [Maxime Defauw](https://www.raywenderlich.com/u/eastprogrammer)
* 译者 : [yrq110](https://github.com/yrq110)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/08/ARC-Memory-Swift-feature-250x250.png)

Swift作为一个现代的高级编程语言，为app处理了大量的内存管理任务，帮助分配与释放内存。她使用一个叫做ARC(自动引用计数,Automatic Reference Counting)的技术来完成这个工作。在这篇教程中，你会学到所有与Swift中ARC和内存管理有关的内容。

理解了ARC的原理后，你可以影响堆对象的生命周期结束的时机。使用ARC的Swift在资源受限的环境中(比如iOS)具有很强的预见性与有效性。

ARC是"自动"的，因此你不需要直接参与对象引用的计数过程，不过为了避免内存泄漏你需要考虑对象间的相互关系，对于新手开发者来说这个是个经常被忽视掉的非常重要的问题。

在教程中会通过学习以下内容来提高你的Swift与ARC水平:

* ARC是如何工作的
* 循环引用是何物与如何中断它们
* 一个循环引用的实例，如何使用新版Xcode的可视化工具来检测
* 如何处理混合值与引用类型

## 入门

打开Xcode新建一个playground，点击File\New\Playground…，选择iOS平台，命名为MemoryManagement，保存在你想要的地方，删掉模板代码后保存一下。

接着在playground中添加如下代码:

```swift
class User {
  var name: String
 
  init(name: String) {
    self.name = name
    print("User \(name) is initialized")
  }
 
  deinit {
    print("User \(name) is being deallocated")
  }
}
 
let user1 = User(name: "John")
```

这里定义了一个User类并且创建了一个实例，User类具有name属性、init方法(在分配内存后调用)和deinit方法(释放前调用)。print语句用来帮助你了解发生了什么。

你会发现在playground的侧边栏会显示"User John is initialized\n"，对应init方法中的print语句。不过，你也会发现deinit方法中的print语句没被调用过。这意味着对象没有被反初始化，即未被释放。这是因为它所初始化的区域一直未关闭，并且playground自身未超出这个域，因此对象没有从内存中移除。

将user1的初始化语句放到do的代码块中:

```swift
do {
  let user1 = User(name: "John")
}
```
这里创建一个包含user1对象初始化部分的域，在域的最后我们希望user1被释放掉。

现在就能在侧边栏看到初始化和反初始化所对应的print语句输出了，这表示对象在从内存中移除之前，在域的最后被反初始化了。

Swift对象的生命周期由5个阶段组成:

1. 分配内存 (从堆栈中取得内存空间)
2. 初始化 (运行初始化代码)
3. 使用 (对象被使用)
4. 反初始化 (运行反初始化代码)
5. 释放内存 (把内存返回给堆栈)

在分配和释放中没有钩子的情况下，可以在初始化和反初始化时使用print语句作为一个代理来监视进程。有时“deallocate(释放)”与“deinit(反初始化)”会交替使用，不过它们实际上是一个对象生命周期中两个不同的阶段。

引用计数(reference counting)根据的是当一个对象不再被需要时会被释放的原理，那么这里有个问题“你怎么能确定一个对象在未来不被需要?”，引用计数中通过使用一个count(计数)来管理，就是所说的引用计数，存在于每个对象实例中。

这个计数表示有多少“东西”引用了对象。当一个对象的引用计数变为0时，没有一个“用户”保留对象，则这个对象会被反初始化和释放。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/Scheme1-480x120.png)

当初始化User对象时，在user1引用该对象后它的引用计数会从1开始计算。在do块的最后，由于use1超出了这个域，则计数减一，引用计数减到了0，那么use1会被反初始化，紧接着被释放。

## 循环引用

多数情况下ARC是很有效的，作为一个开发者，无需总是担心内存泄漏的问题——无用对象一直占用着内存的现象。

不过这并不会一帆风顺，可能会发生泄露的！

那么泄露是如何发生的呢? 想象一下这样一个情况：有两个不再需要的对象，其中每个都引用了另一个，由于两者都有一个非零的引用计数，那么这两个对象永远不会被释放。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/ReferenceCycle-480x153.png)

这被称作循环强引用，它骗过了ARC并且拒绝被清理。如你所见，在结尾时的引用计数并不是零，因此object1与object2将永远不会被释放，即使已经不需要它们了。

来实际操作一下看看，在User类之后，do代码块之前添加如下代码:

```swift
class Phone {
  let model: String
  var owner: User?
 
  init(model: String) {
    self.model = model
    print("Phone \(model) is initialized")
  }
 
  deinit {
    print("Phone \(model) is being deallocated")
  }
}
```
然后把do中的语句改成下面这样:
```swift
do { 
  let user1 = User(name: "John")
  let iPhone = Phone(model: "iPhone 6s Plus")
}
```
这里添加了一个新类Phone并创建了这个新类的实例。

这个新类很简单：包含两个属性：model和owner，两个方法：init和deinit方法。owner属性是可选的，因为一个Phone是可以脱离User存在的。

接下来，在User类中添加如下代码:
```swift
private(set) var phones: [Phone] = []
func add(phone: Phone) {
  phones.append(phone)
  phone.owner = self
}
```
这里添加了一个phone数组来保存一个用户拥有的全部phone，将setter设为private，强制用户使用add(phone:)方法来添加phone而不是操作这个数组。这个方法确保当添加phone时有正确的用户。

现在在侧边栏中可以看到，Phone和User对象都如期被释放了。

把do代码块改成下面这样:
```swift
do {
  let user1 = User(name: "John")
  let iPhone = Phone(model: "iPhone 6s Plus")
  user1.add(phone: iPhone)
}
```
这里将iPhone添加到user1中，自动将iPhone的owner属性设为user1。两个对象间的循环强引用会欺骗ARC使其不会被释放，如此一来，use1和iPhone将永远不会释放。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/UserIphoneCycle-480x202.png)

## 弱引用(weak)

为了消除循环强引用，你可以将对象间的关系指定为弱引用，若不指明的话所有引用都是抢引用，与强引用相比，弱引用不会增加对象的强引用计数。

换句话说，弱引用不会参与对象的生命周期管理，并且弱引用是被声明为可选类型来使用的，这就意味着当引用计数变为0时，引用会自动设置为空。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/WeakReference-480x206.png)

在上图中，虚线箭头表示弱引用，注意object1的引用计数为1是因为variable1引用了它。object2的引用计数是2，是因为variable2与object1都引用了它。在object2引用object1时使用的是弱引用，这意味着不会影响object1的强引用计数。

在variable1和variable2都被移除时，object1的引用计数变为0并将调用deinit方法，这样会移除object2上的强引用，则object2接下来也会被反初始化。

回到playground中，使用如下方法弱化owner的引用来中断User–Phone之间的循环引用:
```swift
class Phone {
  weak var owner: User?
  // 其它代码...
}
```
![](https://koenig-media.raywenderlich.com/uploads/2016/05/UserIphoneCycleWeaked-480x202.png)

可以在侧边栏看到结果，在do代码块的最后user1和iPhone被正常的释放掉了。

## 无主引用(unowned)

这里有另一个可以使用并且不会增加引用计数的引用修饰符：unowned。

无主引用于弱引用有什么不同吗? 弱引用是可选类型的，当引用对象反初始化时会变为nil，这就是为何必须将weak属性定义为可选的var类型。

相比之下，无主引用并不是可选类型，若尝试去访问一个反初始化后对象的unowned属性的话会触发一个运行时错误。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/Table-480x227.png)

来练习一下unowned.，在do代码块的前面添加一个新的类CarrierSubscription:

```swift
class CarrierSubscription {
  let name: String
  let countryCode: String
  let number: String
  let user: User
 
  init(name: String, countryCode: String, number: String, user: User) {
    self.name = name
    self.countryCode = countryCode
    self.number = number
    self.user = user
 
    print("CarrierSubscription \(name) is initialized")
  }
 
  deinit {
    print("CarrierSubscription \(name) is being deallocated")
  }
}
```
CarrierSubscription类有四个属性: name, countryCode, number, 和一个引用的User对象。

接着在User类中添加如下代码:
```swift
var subscriptions: [CarrierSubscription] = []
```
这里添加了subscriptions属性，使用一个数组来保存CarrierSubscrition对象。

在Phone类的顶部，owner属性后面添加如下代码:
```swift
var carrierSubscription: CarrierSubscription?
 
func provision(carrierSubscription: CarrierSubscription) {
  self.carrierSubscription = carrierSubscription
}
 
func decommission() {
  self.carrierSubscription = nil
}
```
添加了一个可选CarrierSubscription属性和两个新函数-provision与decommission，用来添加和移除phone上的运营商描述。

接在在print语句之前添加如下代码，初始化内部的CarrierSubscription:
```swift
user.subscriptions.append(self)
```
将CarrierSubscription添加到用户的描述数组中。

最后，将do代码块中的内容改成如下所示:
```swift
do { 
  let user1 = User(name: "John")
  let iPhone = Phone(model: "iPhone 6s Plus")
  user1.add(phone: iPhone)
  let subscription1 = CarrierSubscription(name: "TelBel", countryCode: "0032", number: "31415926", user: user1)
  iPhone.provision(carrierSubscription: subscription1)
}
```
看看侧边栏输出了什么结果。再次看到了循环引用：不管是user、iphone还是subscription1，最后都没有被释放。现在你可以发现问题出在哪吗?

![](https://koenig-media.raywenderlich.com/uploads/2016/05/UserIphoneSubCycle-480x175.png)

将从use1到subscription1的引用或从subscription1到user1的引用变成unowned即可中断循环，问题是选择哪一个引用？

user拥有运行商描述，而运营商描述不可拥有user。还有，不存在一个没有用户的CarrierSubscription，这就是为何在一开始将其声明为一个不可变的let属性。

因为可以存在一个不包含CarrierSubscription的User，不可存在一个不包含User的CarrierSubscription，所以应该用unowned来修饰user引用。

给CarrierSubscription的user属性添加unowned修饰符:
```swift
class CarrierSubscription {
  let name: String
  let countryCode: String
  let number: String
  unowned let user: User
  // Other code...
}
```
这样就中断了循环引用，使每个对象都被释放掉。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/UserIphoneCycleSubSolve-480x175.png)

## 闭包的循环引用

当对象的属性相互引用时会发生循环引用。同样，闭包也是可引用的类型，也会发生循环引用的问题，闭包在这里捕捉到了所操作的对象。

举个栗子，若有一个闭包被分配给一个类的属性，并且这个闭包使用了相同类下的属性，这就发生了循环引用。换言之，对象通过一个存储属性引用了闭包，闭包通过捕捉的self值引用了对象。

![](https://koenig-media.raywenderlich.com/uploads/2016/06/Closure-Referene-1-480x202.png)

在CarrierSubscription的user属性后添加如下代码:

```swift
lazy var completePhoneNumber: () -> String = {
  self.countryCode + " " + self.number
}
```

这个闭包计算并返回一个完整的电话号码。使用lazy声明这个属性，意味着直到第一次使用时才会分配属性。需要这个修饰符是因为这个属性是依赖self.countryCode和self.number的，它们在初始化后才可用。

在do代码块的最后添加如下代码:

```swift
print(subscription1.completePhoneNumber())
```

你会注意到use1和iPhone都会释放了，不过因为存在的对象与闭包之间的循环强引用，CarrierSubscription并没有被释放。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/ClosureCylce-480x232.png)

Swift has a simple, elegant way to break strong reference cycles in closures. You declare a capture list in which you define the relationships between the closure and the objects it captures.

为了形象的描述捕捉列表是如何工作的，试试如下代码:

```swift
var x = 5
var y = 5
 
let someClosure = { [x] in
  print("\(x), \(y)")
}
 
x = 6
y = 6
 
someClosure()        // Prints 5, 6
print("\(x), \(y)")  // Prints 6, 6
```
The variable x is in the capture list, so a copy of x is made at the point the closure is defined. It is said to be captured by value. On the other hand, y is not in the capture list, and is instead captured by reference. This means that when the closure runs, y will be whatever it is at that point, rather than at the point of capture.

Capture lists come in handy for defining a weak or unowned relationship between objects used in a closure. In this case, unowned is a good fit since the closure could not exist while the instance of CarrierSubscription has gone away.

Change the completePhoneNumber closure in CarrierSubscription to look like this:
```swift
lazy var completePhoneNumber: () -> String = {
  [unowned self] in
  return self.countryCode + " " + self.number
}
```
This adds [unowned self] to the capture list for the closure. It means that self is captured as an unowned reference instead of a strong reference.

This solves the reference cycle. Hooray!

The syntax used here is actually a shorthand for a longer capture syntax, which introduces a new identifier. Consider the longer form:
```swift
var closure = {
  [unowned newID = self] in
  // Use unowned newID here...
}
```
Here, newID is an unowned copy of self. Outside the closure’s scope, self keeps its original meaning. In the short form, which you used above, a new self variable is created which shadows the existing self variable just for the closure’s scope.

In your code, the relationship between self and the completePhoneNumber closure is unowned. If you are sure that a referenced object from a closure will never deallocate, you can use unowned. If it does deallocate, you are in trouble.

Add the following code to the end of the playground:
```swift
// A class that generates WWDC Hello greetings.  See http://wwdcwall.com
class WWDCGreeting {
  let who: String
 
  init(who: String) {
    self.who = who
  }
 
  lazy var greetingMaker: () -> String = {
    [unowned self] in
    return "Hello \(self.who)."
  }
}
 
let greetingMaker: () -> String
 
do {
  let mermaid = WWDCGreeting(who: "caffinated mermaid")
  greetingMaker = mermaid.greetingMaker
}
 
greetingMaker() // TRAP!
```
The playground hits a runtime exception because the closure expects self.who to still be valid, but it was deallocated when mermaid went out of scope. This example may seem contrived, but it can easily happen in real life such as when you use closures to run something much later, such as after an asynchronous network call has finished.

Change the greetingMaker variable in WWDCGreeting to look like this:
```swift
lazy var greetingMaker: () -> String = {
  [weak self] in
  return "Hello \(self?.who)."
}
```
There are two changes you made to the original greetingMaker closure. First, you replaced unowned with weak. Second, since self became weak, you needed to access the who property with self?.who.

The playground no longer crashes, but you get a curious result in the sidebar: "Hello, nil.". Perhaps this is acceptable, but more often you want to do something completely different if the object is gone. Swift’s guard let makes this easy.

Re-write the closure one last time, to look like this:
```swift
lazy var greetingMaker: () -> String = {
  [weak self] in
  guard let strongSelf = self else {
    return "No greeting available."
  }
  return "Hello \(strongSelf.who)."
}
```
The guard statement binds a new variable strongSelf from weak self. If self is nil, the closure returns "No greeting available.". On the other hand, if self is not nil, strongSelf makes a strong reference, so the object is guaranteed to live until the end of the closure.

This idiom, sometimes referred to as the strong-weak dance, is part of the Ray Wenderlich Swift Style Guide since it’s a robust pattern for handling this behavior in closures.
![](https://koenig-media.raywenderlich.com/uploads/2016/07/testskillz-650x241.png)
