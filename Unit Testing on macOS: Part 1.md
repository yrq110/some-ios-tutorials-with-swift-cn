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
