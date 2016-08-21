#Pattern Matching in Swift
##Swift中的模式匹配

***

>* 原文链接 : [Pattern Matching in Swift](https://www.raywenderlich.com/134844/pattern-matching-in-swift)
* 原文作者 : [Cosmin Pupaza](https://www.raywenderlich.com/u/shogunkaramazov)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/07/PatternMatching-feature-250x250.png)

不论对于哪个编程语言，**模式匹配**都是一个有力的特性，根据这个特性使你可以设计规则来匹配期望的值，使代码更加灵活与简洁。 

苹果在Swift中加入了模式匹配，今天你将会看到Swift中的模式匹配技术。

教程中涵盖的模式:

* **元组模式**
* **类型转换模式**
* **通配符模式**
* **可选模式**
* **枚举用例模式**
* **表达式模式**

为了体现模式匹配的有用之处，你在这篇教程中会以一个独特的视角: 作为raywenderlich.com的主编！使用模式匹配来安排与发布网站上的教程。

> 注意: 这篇教程需要Xcode 8 与Swift 3，并且假定你已经掌握了基本的Swift开发。如果你是Swift新手请先看看我们的其它Swift教程。

## 入门

欢迎，临时主编! 你今天的主要任务是安排在网站上发布的教程。下载开始的[plauground](https://cdn4.raywenderlich.com/wp-content/uploads/2016/07/starter-project.playground-2.zip)并在Xcode中打开**starter project.playground**。

playground中包含两个东西:

* **random_uniform(value:)**函数，会从0与一个实际值间返回一个随机数，使用它来生成随机日期。
* 解析**tutorials.json**文件将内容转换为字典数组并返回的样板代码。使用它来提取需要安排的教程信息。

> 注意：了解更多有关Swift的JSON解析可以查看我们的[教程](https://www.raywenderlich.com/120442/swift-json-tutorial)。

无需理解它们是如何工作的，只需要知道文件结构即可，接下来打开playground资源文件夹中的tutorials.json文件。

每一篇安排的教程都有两个属性: 标题与日期。你的老大为你安排好了发布的日程，每天都分配了一篇教程，1表示周一，5表示周五，依次类推，nil表示不在日程内。

你想将本周的课程安排为每天发布一篇教程，然而在查看日程表时发现你的老大把两篇教程安排到了同一天。需要你来修正这个问题，你想通过一个特定的规则来对这些教程进行排序，该怎么办？

如果你想到了“使用模式!” 那么就上道儿了。 :]

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/07/rage_patterns.png)

## 模式匹配类型

了解一下这篇教程中需要用到的模式类型。

* **元组模式** 用于匹配元组类型的值。
* **类型转换模式** 允许你转换或匹配类型。
* **通配符模式** 用于匹配并忽略任何类型的值。
* **可选模式** 用于匹配可选值。
* **枚举用例模式** 匹配存在的枚举类型的用例。
* **表达式模式** 允许你使用表达式与值进行比较。

在成为一个前所未有的优秀主编的道路上你会使用所有这些模式！

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/07/rw-1-250x250.png)

让这家伙失业吧!(PS:此人为来源站主编)
***
