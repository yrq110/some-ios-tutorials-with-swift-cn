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
* **枚举成员模式**
* **表达式模式**

为了体现模式匹配的有用之处，你在这篇教程中会以一个独特的视角: 作为raywenderlich.com的主编！使用模式匹配来整理与发布网站上的教程。

> 注意: 这篇教程需要Xcode 8 与Swift 3，并且假定你已经掌握了基本的Swift开发。如果你是Swift新手请先看看我们的其它Swift教程。

## 入门

Welcome aboard, temporary editor-in-chief! Your main duties today involve scheduling tutorials for publication on the website. Start by downloading the starter playground and open starter project.playground in Xcode.

The playground contains two things:

* The random_uniform(value:) function, which returns a random number between zero and a certain value. You’ll use this to generate random days for the schedule.
* Boilerplate code that parses the tutorials.json file and returns its contents as an array of dictionaries. You’ll use this to extract information about the tutorials you’ll be scheduling.

>Note: To learn more about JSON parsing in Swift, read our tutorial.

You don’t need to understand how all this works, but you should know the file’s structure, so go ahead and open tutorials.json from the playground’s Resources folder.

Each tutorial post you’ll be scheduling has two properties: title and scheduled day. Your team lead schedules the posts for you, assigning each tutorial a day value between 1 for Monday and 5 for Friday, or nil if leaving the post unscheduled.

You want to publish only one tutorial per day over the course of the week, but when looking over the schedule, you see that your team lead has two tutorials scheduled for the same day. You’ll need to fix the problem. Plus, you want to sort the tutorials in a particular order. How can you do all of that?

If you guessed “Using patterns!” then you’re on the right track. :]

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/07/rage_patterns.png)
