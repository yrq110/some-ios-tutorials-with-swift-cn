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
