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

## 类型转换模式

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

逐步分析下:

1. 切换到键值元组---重载的元组模式。
2. 使用`is`类型转换模式检测教程的标题是否属于String类型，若是则转换它。
3. 使用`as`类型转换模式检测教程的日期是否是String类型，若是则先将其转换成Int，然后传入枚举的可失败构造器init(rawValue:)中，得到对应的枚举值。将dayInt减去1是因为枚举的初始值是0，而tutorial.json中是从1开始的。
4. switch语句应该是完备的，需要添加一个default case，使用break语句来跳出switch。

在playground的底部添加如下代码，在控制台打印数组中的内容:

```swift
print(tutorials)
```

如你所见，现在数组中的每个教程都含有名称与发布日期了。万事俱备只欠完成的任务: 每周每天仅发布一篇教程。

## 通配符模式

使用通配符来安排教程的发布，不过需要先将它们的发布日期置空。在playground底部添加如下代码:

```swift
tutorials.forEach { $0.day = nil }
```

将数组中的所有教程的日期设为nil，为了安排教程的日期，在playground的底部添加如下代码段:

```swift
// 1 
let days = (0...6).map { Day(rawValue: $0)! }
// 2
let randomDays = days.sorted { _ in random_uniform(value: 2) == 0 }
// 3
(0...6).forEach { tutorials[$0].day = randomDays[$0] }
```

这儿有点复杂，分解成如下步骤看下:

1. 首先创建一个有关天数的数组，包含每周中不重复的每一天。
2. “整理”这个数组， random\_uniform(value:)函数用来生成随机数。在闭包中，使用下划线来忽略闭包参数，因为并不需要它。虽然有在技术上更加高效、在数学上更加准确的方法来随机变换数组元素位置，不过这就是一个通配符模式的实现!
3. 最后，将一周中七天随机分配给了最开始的七个教程。

在playground底部添加下列代码，控制台中打印安排好的教程:

```swift
print(tutorials)
```

成功了! 现在为一周中每一天都安排了一篇教程，日程中既没有发布时间重合的教程也没有时间空隙。干得好!

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/07/rage_bestEICever.png)

## 可选模式

日程表被占用了，作为主编你需要整理一下这些教程。The schedule has been conquered, but as editor-in-chief you also need to sort the tutorials. 使用可选模式来处理它。You’ll tackle this with optional patterns.
将教程数组按升序排序，首先根据标题来整理未安排的教程，其次根据日期来整理来已安排的教程，在playground底部添加如下代码:

```swift
// 1
tutorials.sort {
  // 2
  switch ($0.day, $1.day) {
    // 3
    case (nil, nil):
      return $0.title.compare($1.title, options: .caseInsensitive) == .orderedAscending
    // 4
    case (let firstDay?, let secondDay?):
      return firstDay.rawValue < secondDay.rawValue
    // 5
    case (nil, let secondDay?):
      return true
    case (let firstDay?, nil):
      return false
  }
}
```

逐步分析下:

1. 使用sort(\_:)方法来整理教程数组，方法的参数是一个结尾闭包(trailing closure)，在闭包中定义了数组中两个教程的排序顺序。若将教程按升序进行排序则返回true，否则返回false。
2. 切换到由当前被整理的两个教程组成的元组中，这又是一个`元组模式`。
3. 若两个教程都未被安排，它们的day属性是nil，则使用数组的`compare(\_:options:)`方法按标题的升序来整理它们。
4. 使用`可选模式`来检测两个教程是否都被安排了，这种模式仅匹配可以被解绑的值，若两个值都可被解绑，则按它们的raw value的升序来整理。
5. 再次使用可选模式，检测只有一个教程被安排时的情况，若满足，则优先整理未被安排的教程。
在playground的最后添加这行代码，打印出整理后的教程:

```swift
print(tutorials)
```

现在已经将教程按你想要的方式排序好了，做的这么好理应涨工资! 然而 … 还需要做的事有很多。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/07/rage_nomoretrophies.png)


## 枚举用例模式

现在让我们来使用枚举用例模式来决定每个教程发布天的名称。

在Tutorial的扩展中，使用了Day类型中的枚举用例名称来构建自定义字符串。添加一个合适的名称，而不是与原有名称绑定，在playground的底部添加如下代码:

```swift
extension Day {
 
  var name: String {
    switch self {
      case .monday:
        return "Monday"
      case .tuesday:
        return "Tuesday"
      case .wednesday:
        return "Wednesday"
      case .thursday:
        return "Thursday"
      case .friday:
        return "Friday"
      case .saturday:
        return "Saturday"
      case .sunday:
        return "Sunday"
    }
  }
}
```

switch语句中使用当前值(self)来匹配枚举用例，这就是枚举用例模式的实现。

印象挺深的，不是吗? 虽然用数字挺酷的，不过毕竟用名称的话很直观并且更容易理解! :]

## 表达式模式

接着来添加一个描述教程发布顺序的属性。也许你会再次使用枚举用例模式，如下(**别在playground中添加!**):

```swift
var order: String {
  switch self {
    case .monday:
      return "first"
    case .tuesday:
      return "second"
    case .wednesday:
      return "third"
    case .thursday:
      return "fourth"
    case .friday:
      return "fifth"
    case .saturday:
      return "sixth"
    case .sunday:
      return "seventh"
  }
}
```

不过同一件事做两遍的话就太不像主编了，不是吗? ;] 另辟蹊径，使用表示式模式。首先需要重载模式匹配的操作符，这是为了改变默认的功能并且使其对Day属性有效。在playground底部添加如下代码:

```swift
func ~=(lhs: Int, rhs: Day) -> Bool {
  return lhs == rhs.rawValue + 1
}
```
这段代码用来匹配day与整数，会返回数字1~7。使用这个重载的操作符以不同的方法来得到计算后的属性。

在playground底部添加如下代码:

```swift
extension Tutorial {
 
  var order: String {
    guard let day = day else {
      return "not scheduled"
    }
    switch day {
      case 1:
        return "first"
      case 2:
        return "second"
      case 3:
        return "third"
      case 4:
        return "fourth"
      case 5:
        return "fifth"
      case 6:
        return "sixth"
      case 7:
        return "seventh"
      default:
        fatalError("invalid day value")
    }
  }
}
```
多亏了重载的模式匹配操作符，day对象可以使用整数表达式来匹配了。这就是表达式模式的实现。

## Putting It All Together

Now that you’ve defined the day names and the tutorials’ order, you can print each tutorial’s status. Add the following block of code at the end of the playground:

```swift
for (index, tutorial) in tutorials.enumerated() {
  guard let day = tutorial.day else {
    print("\(index + 1). \(tutorial.title) is not scheduled this week.")
    continue
  }
  print("\(index + 1). \(tutorial.title) is scheduled on \(day.name). It's the \(tutorial.order) tutorial of the week.")   
}
```
Notice the tuple in the for-in statement? There’s the tuple pattern again!
Whew! That was a lot of work for your day as editor-in-chief, but you did a fantastic job—now you can relax and kick back by the pool.

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/07/rage_pool.png)

开个玩笑!主编的活儿是忙不完的。回去工作!

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/07/rage_morepatterns.png)
