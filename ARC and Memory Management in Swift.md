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

这被称作强循环引用，它骗过了ARC并且拒绝被清理。如你所见，在结尾时的引用计数并不是零，因此object1与object2将永远不会被释放，即使已经不需要它们了。

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
这里将iPhone添加到user1中，自动将iPhone的owner属性设为user1。两个对象间的强循环引用会欺骗ARC使其不会被释放，如此一来，use1和iPhone将永远不会释放。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/UserIphoneCycle-480x202.png)

## 弱引用

为了消除强循环引用，你可以将对象间的关系指定为弱引用，若不指明的话所有引用都是抢引用，与强引用相比，弱引用不会增加对象的强引用计数。

换句话说，弱引用不会参与对象的生命周期管理，并且弱引用是被声明为可选类型来使用的，这就意味着当引用计数变为0时，引用会自动设置为空。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/WeakReference-480x206.png)

In the image above, the dashed arrow represents a weak reference. Notice how the reference count of object1 is 1 because variable1 refers to it. The reference count of object2 is 2, because both variable2 and object1 refer to it. While object2 references object1, it does so weakly, meaning it doesn’t affect the strong reference count of object1.

When both variable1 and variable2 go away, object1 will be left with a reference count of zero and deinit will be called. This removes the strong reference to object2 which then subsequently deinitializes.

Back in your playground, break the User–Phone reference cycle by making the owner reference weak as shown below:

```swift
class Phone {
  weak var owner: User?
  // other code...
}
```

![](https://koenig-media.raywenderlich.com/uploads/2016/05/UserIphoneCycleWeaked-480x202.png)

Now user1 and iPhone deallocate properly at the end of the do block as you can see in the results sidebar.

## Unowned References

There is another reference modifier you can use that doesn’t increase the reference count: unowned.

What’s the difference between unowned and weak? A weak reference is always optional and automatically becomes nil when the referenced object deinitializes. That’s why you must define weak properties as optional var types for your code to compile (because the variable needs to change).

Unowned references, by contrast, are never optional types. If you try to access an unowned property that refers to a deinitialized object, you will trigger a runtime error comparable to force unwrapping a nil optional type.

![](https://koenig-media.raywenderlich.com/uploads/2016/05/Table-480x227.png)

Time to get some practice using unowned. Add a new class CarrierSubscription before the do block as shown:

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
