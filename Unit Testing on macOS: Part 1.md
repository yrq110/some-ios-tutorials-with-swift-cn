#Unit Testing on macOS: Part 1
##

***

>* 原文链接 : [Unit Testing on macOS: Part 1/2](https://www.raywenderlich.com/141405/unit-testing-macos-part-12)
* 原文作者 : [Sarah Reichelt](https://www.raywenderlich.com/u/sarah)
* 译者 : []()

***


单元测试就是，我们心里都知道应该去做，但是却感觉它是繁杂无趣而且工作量巨大的一件麻烦事儿。

写代码如此愉悦，为啥还有人愿意花费写代码大半的时间来做代码检查？

理由就是自信！这在这个macOS上的单元测试入门教程中，你会学会如何测试你写的代码，还能获得自信，自信让代码按你的意愿去做事，自信即使对代码进行大改动也不会破坏原有的代码。

## 入门

示例工程使用Swift 3，同时必须使用至少是Xcode 8 beta 6的版本。下载[初始工程](https://cdn5.raywenderlich.com/wp-content/uploads/2016/08/HighRoller-starter.zip)并在Xcode中打开。

如果你看过raywenderlich.com中的任何一个教程，在这步你都想去构建运行程序了。但这次不同--你得去测试。打开**Product**菜单，选择**Test** ，提示你一个快捷键-- **Command-U** -- 之后你会多次用到这个快捷键。


当运行测试的时候，Xcode会先构建应用程序，然后应用程序的窗口出现几次之后，会看到消息提示“Test Succeeded”。在左侧的**Navigator**(导航栏)中，选中**Test Navigator**

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/08/TestNavigator2.png)


这里显示三个默认加入的测试栏目；每个测试栏目的边上都有一个绿色的小对勾，表示测试已经通过。如果想看测试栏目都包含了哪些文件，点击**Test Navigator**的里面第二行有大写T字母图标开头的叫**High Roller Tests**的栏目。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/08/DefaultTests3.png)

有几个要点需要提示一下:

- import引用: **XCTest**是Xcode提供的测试框架。`@testable import High_Roller`引入是用来给测试代码访问`High_Roller`模块用的。每个测试文件都需要引入这二个。
- `setup()`和`tearDown()` : 这二个方法会分别在每个单独的测试方法调用前后被调用。
- `testExample()`和`testPerformanceExample()`:真正的测试部分。第一个方法用来测试代码功能，第二个用来测试代码性能。每个测试方法都必须以test开头，这样Xcode才能区分出这些代码是用于执行测试的。

### 单元测试是什么

在你开始写单元测试之前，有必要简要了解一下什么是单元测试以及我们为什么要用单元测试。

单元测试就是测试单独的代码片段或者一组代码片段的一个函数。它并不会被包含到你开始的应用程序当中，只是在开发阶段用来测试你所写的代码是否按你的期望来运行的。

大部分人对单元测试的第一反应是：“你让我写二份代码？一份是应用程序自己内部的功能代码，另一份是测试这个功能的代码？”，但事实可能比二份还要多--一些大的工程到最后的时候测试代码比功能本身的代码还要多得多。

首先这看起来非常耗费时间和精力--但是一旦你通过测试发现了你代码中不曾注意到的错误，或者发觉了重构代码产生的副作用，你就会体会到单元测试是个多么棒的工具。任何没有单元测试的工程在用不了多久之后就会显得很不靠谱，你不太敢对工程做改动，因为你不知道改动之后会发生什么。

## 测试驱动开发

测试驱动开发(TDD)是单元测试的一个分支，就是只针对必须的测试步骤编写对应的代码。
