#Bond Tutorial: Bindings in Swift
##学习Bond：Swift中的数据绑定

[原文地址](https://www.raywenderlich.com/123108/bond-tutorial)

>注意: 升级到Xcode 7.3, iOS 9.3, Swift 2.2. 2016.04.08

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/01/Binding-with-bond-feature-250x250.png)

对UI代码的手动绑定感到疲倦吗？我想是的。这是个必要的工作我们不得不去做，不是吗？

Swift Bond是一个绑定框架，可以免去在绑定UI时的繁琐工作，这样就有了更多的时间去思考如何解决应用中的问题。

在这篇Bond教程中，会学着使用Bond来节省时间，写出可维护的代码。Bond与ReactiveCocoa很像，在文章的最后我会说一下，现在让我们来使用Bond进行绑定吧 ... Swift Bond。

>注意: 对于不知道Swift Bond的读者，这是一个框架的名字,来源于小说中的间谍-James Bond, 他总是这样介绍自己“Bond … James Bond.” ，
他很喜欢干马天尼，“摇匀，不要搅拌”。放轻松！准备进入Swift Bond的世界！

##快速绑定

启动Xcode开始写一些代码。

首先，下载[开始工程](http://www.raywenderlich.com/wp-content/uploads/2016/01/BindingWithBond-Starter.zip)，包含了应用的基本框架。

下载后，首先确保你安装了CocoaPods(如果没有，建议你看看这篇[起步教程](http://www.raywenderlich.com/97014/use-cocoapods-with-swift))。打开终端运行pod install进行安装，像下面这样:
````
$ pod install
Updating local specs repositories
Analyzing dependencies
Downloading dependencies
Installing Bond (4.3.0)
Installing DatePickerCell (1.0.4)
Generating Pods project
Integrating client project
[!] Please close any current Xcode sessions and use `BindingWithBond.xcworkspace` for this project from now on.
Sending stats
Pod installation complete! There is 1 dependency from the Podfile and 1 total
pod installed.
````
用Xcode打开刚生成的workspace文件，构建并运行，会看到如下图:
![](https://cdn3.raywenderlich.com/wp-content/uploads/2015/12/Starter-319x500.png)

一切正常吗? 是时候创建你的第一个绑定了!
打开ViewController.swift在viewDidLoad()中添加如下代码:
````swift
searchTextField.bnd_text.observe {
  text in
  print(text)
}
````
Bond框架给UIkit控件添加了各种属性，在这里，bnd_text是一个关于UITextField text属性的观测量(observable)，观测量包含了事件流。 别担心，关于这个会详尽地介绍。现在构建并运行一下，输入“hello”看看发生了什么。

观察一下Xcode控制器台中的输出:
````
Optional("")
Optional("h")
Optional("he")
Optional("hel")
Optional("hell")
Optional("hello")
````
每当你键入一个字符时，代码内的闭包表达式都会执行一次，显示应用中text field的当前状态。

这表明了可以监测到例如属性改变等的事件， 你将会看到这其中蕴含着怎样的力量。

##数据操作
回到ViewController.swift中，更新你的代码:
````swift
let uppercase = searchTextField.bnd_text
  .map { $0?.uppercaseString }
 
uppercase.observe {
    text in
    print(text)
  }
````
如果你用过map来对swift数组进行变换操作的话应该能猜到上述代码做了些什么。

>注意：如果你对map方法不太熟悉可以看看[函数式swift](http://www.raywenderlich.com/82599/swift-functional-programming-tutorial)

构建并运行，然后输入hello
````
Optional("")
Optional("H")
Optional("HE")
Optional("HEL")
Optional("HELL")
Optional("HELLO")
````
使用map操作对btn_text的输出进行了变换操作，返回了一个观测量保存进uppercase中。闭包表达式再一次随着事件变化而执行。

执行完map操作后的结果还是一个观测量，因此不需要使用中间变量进行赋值。更新你的代码:
````swift
searchTextField.bnd_text
  .map { $0?.uppercaseString }
  .observe {
    text in
    print(text)
  }
````
这么写就有了优雅的"流式"语句的风格，构建并运行应用执行相同的操作看看。
你也许觉得这样没啥，印象并不深。让我给你展示一下最棒的代码是怎样的?

使用如下代码替换当前代码:
````swift
searchTextField.bnd_text
  .map { $0!.characters.count > 0 }
  .bindTo(activityIndicator.bnd_animating)
````

构建并运行你的应用，会注意到活动指示器只会在text field中有文本时才会执行动画!这真是个令人印象深刻的技巧，只用三行代码就实现了，比James Bond的汽车追逐戏更令人印象深刻:]

如你所见，Swift Bond提供了一种流畅并明确的方式去描述应用与UI中的数据流。简化了你应用的代码，更容易理解，并能迅速达成目标。

在下一章你会学到一些工作原理，现在花点时间看看它的本事是值得的-Bond … Swift Bond.

![](http://www.raywenderlich.com/wp-content/uploads/2016/01/swiftmartini.png)
