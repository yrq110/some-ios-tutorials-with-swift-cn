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

## Getting Started

Open Xcode and click File\New\Playground…. Select the iOS Platform, name it MemoryManagement and then click Next. Save it wherever you wish and then remove the boilerplate code and save the playground.

Next, add the following code to your playground:

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

This defines a class called User and creates an instance of it. The class User has one property, name, an init method (called just after memory allocation) and a deinit method (called just prior to deallocation). The print statements let you see what is happening.

You will notice that the playground shows "User John is initialized\n" in the sidebar, on the print statement within init. However, you will notice that the print statement within deinit is never called. This means that the object is never deinitialized, which means it’s never deallocated. That’s because the scope in which it was initialized is never closed — the playground itself never goes out of scope — so the object is not removed from memory.

Change the let user1 initialization by wrapping it in a do like so:

```swift
do {
  let user1 = User(name: "John")
}
```
This creates a scope around the initialization of the user1 object. At the end of the scope, we would expect user1 to be deallocated.

Now you see both print statements that correspond to initialization and deinitialization in the sidebar. This shows the object is being deinitialized at the end of the scope, just before it’s removed from memory.

The lifetime of a Swift object consists of five stages:

1. Allocation (memory taken from stack or heap)
2. Initialization (init code runs)
3. Usage (the object is used)
4. Deinitialization (deinit code runs)
5. Deallocation (memory returned to stack or heap)

While there are no direct hooks into allocation and deallocation, you can use print statements in init and deinit as a proxy for monitoring those processes. Sometimes “deallocate” and “deinit” are used interchangeably, but they are actually two distinct stages in an object’s lifetime.

Reference counting is the mechanism by which an object is deallocated when it is no longer required. The question here is “when can you be sure an object won’t be needed in the future?” Reference counting manages this by keeping a usage count, also known as the reference count, inside each object instance.

This count indicates how many “things” reference the object. When the reference count of an objects hits zero no clients of the object remain, so the object deinitializes and deallocates.

![](https://koenig-media.raywenderlich.com/uploads/2016/05/Scheme1-480x120.png)

When you initialize the User object, it starts with a reference count of 1 since the constant user1 references that object. At the end of the do block, user1 goes out of scope, the count decrements, and the reference count decrements to zero. As a result, user1 deinitializes and subsequently deallocates.
