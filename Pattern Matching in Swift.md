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

## 元组模式

首先要出创建一个元组模式来生成一个教程的数组。在playground的底部添加如下代码:

```swift
enum Day: Int {
  case monday, tuesday, wednesday, thursday, friday, saturday, sunday
}
```

创建了一个包含一周七天的枚举，由于初始值是Int类型的，因此每一天都被分配了一个Int型值，Monday是0，Sunday是6，以此类推。

在枚举声明的后面添加如下代码:

```swift
class Tutorial {
 
  let title: String
  var day: Day?
 
  init(title: String, day: Day? = nil) {
    self.title = title
    self.day = day
  }
}
```
这里定义了一个包含两个属性的tutorial类型:tutorial的标题与发布日期。日期是一个可选变量，因此应未发布的教程来说它可以是nil。

实现CustomStringConvertible，这样就可以轻松的将教程信息打印出来了:
```swift
extension Tutorial: CustomStringConvertible {
  var description: String {
    var scheduled = ", not scheduled"
    if let day = day {
      scheduled = ", scheduled on \(day)"
    }
    return title + scheduled
  }
}
```
添加一个数组来存放教程:

```swift
var tutorials: [Tutorial] = []
```

接着，通过在playground底部添加如下代码，将开始工程中的字典数组转换为教程数组:

```swift
for dictionary in json {
  var currentTitle = ""
  var currentDay: Day? = nil
 
  for (key, value) in dictionary {
    // todo: extract the information from the dictionary
  }
 
  let currentTutorial = Tutorial(title: currentTitle, day: currentDay)
  tutorials.append(currentTutorial)
}
```
在这里使用for-in语句遍历了json数组。使用以元组格式为循环值的for-in语句遍历了数组中每个字典的键值对。这就是元组模式的实现。

你把每个教程都添加进了数组中，不过当前教程的属性是空的，需要在下一章节中使用类型转换模式来设置教程的属性。

##类型转换模式

为了从字典中提取教程信息，需要一个类型转换模式。在for-in循环中添加如下代码来替换掉注释:
```swift
// 1
switch (key, value) {
  // 2
  case ("title", is String):
    currentTitle = value as! String
  // 3
  case ("day", let dayString as String):
    if let dayInt = Int(dayString), let day = Day(rawValue: dayInt - 1) {
      currentDay = day
  }
  // 4
  default:
    break
}
```

