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

在下一章你会学到一些工作原理，现在花点时间了解一下它的本事是值得的-Bond … Swift Bond.

![](http://www.raywenderlich.com/wp-content/uploads/2016/01/swiftmartini.png)

##工作原理

在做一个更有意义的app之前我们先来看看Bond有哪些功能，我建议你浏览一下Bond的类型(按住cmd点击)，看看注释解释。

在刚才代码的基础上开始编写，bnd_text是一个包含String?属性的Bond观测量，观测量是EventProducer的一个子类。

Event producers会根据由泛型类型定义的数据变化触发相应的事件。

当你调用observe方法时，就将闭包表达式注册为了一个观察者。每次触发事件时，都会用事件数据(由泛型类型定义)调用闭包表达式。

比如，考虑一下一开始写的代码:
````swift
searchTextField.bnd_text.observe {
    text in
    print(text)
  }
````
调用observe，每当searchTextField的text属性改变时就会执行一次闭包中的代码。

>注意：这基于一个经典的设计模式 - [观察者模式](https://en.wikipedia.org/wiki/Observer_pattern).

EventProducer遵守EventProducerType协议, 也就是定义map操作时基于的一个协议扩展。你可能听过“面向协议编程”，这就是个很好的例子。

观测量在EventProducer接口上添加了一个值属性，允许它去构建一个属性或变量。通过值属性可以使用‘get’或‘set’来操作这个值，并能观察它的变化，这得益于EventProducer的功能性。

Bond扩展了UIKit，添加了响应观测量属性。比如UITextField有一个text属性，与之对应的就是观测量bnd_text。所以get或set‘bnd_text.value’就等价于对text属性进行相应的操作，使用观测量的话就可以观测到它的改变。

最后一块拼图就是绑定了，bindTo方法连接一对观测量属性，用来同步变化。像下面这样将活动指示器的动画属性与field的text长度绑定:
````swift
searchTextField.bnd_text
  .map { $0!.characters.count > 0 }
  .bindTo(activityIndicator.bnd_animating)
````

##MVVM
Bond使UI属性间的绑定变得很轻松，不过并不是想要的目标。Bond是一个支持模型-视图-视图模型(Model-View-ViewModel,MVVM)模式的框架，如果你没用过这种模式，我建议你看看我[以前的教程中的第一部分](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1)。
理论知识差不多了，来写写代码吧:]

##添加一个视图模型(View Model)
首先，移除ViewController.swift中的示例代码，一个可信度更高的应用结构应有如下的viewDidLoad()方法:
````swift
override func viewDidLoad() {
  super.viewDidLoad()
}
````
创建名为PhotoSearchViewModel.swift的文件, 添加如下代码:
````swift
import Foundation
import Bond
 
class PhotoSearchViewModel {
  let searchString = Observable<String?>("")
 
  init() {
    searchString.value = "Bond"
 
    searchString.observeNew {
      text in
      print(text)
    }
  }
}
````
这里创建了一个简单的包含一个属性的视图模型，使用"Bond"作为值进行初始化。

打开ViewController.swift，在类的顶部添加如下属性:
````swift
private let viewModel = PhotoSearchViewModel()
````
如下修改viewDidLoad():
````swift
override func viewDidLoad() {
  super.viewDidLoad()
  bindViewModel()
}
````
在类中添加这个方法:
````swift
func bindViewModel() {
  viewModel.searchString.bindTo(searchTextField.bnd_text)
}
````
bindViewModel()方法就是之后需要添加绑定的地方，现在仅仅连接text field与视图模型就可以了。

构建并运行应用，会看到显示了文本‘Bond’:

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/12/FirstBinding-319x500.png)

视图模型现在观测的是searchString的变化，不过你会发现在text field中输入时，视图模型并没有及时更新。怎么回事？

现在的绑定是单向的，意味着变化仅仅由源头(视图模型属性)传递给终点(text field的bnd_text属性)

怎么使反向也能传递变化呢? 很简单 — 如下修改绑定方法:
````swift
viewModel.searchString.bidirectionalBindTo(searchTextField.bnd_text)
````
真轻松!
构建并运行, 输入“Bond Love”:
````
Optional("Bond ")
Optional("Bond L")
Optional("Bond Lo")
Optional("Bond Lov")
Optional("Bond Love")
````
很棒 — 现在确信了text field的变化会传递给视图模型了。
##由观测量创建观测量
是时候对观测量映射部分做一些有用的事情了。回到PhotoSearchViewModel.swift，在视图模型中添加如下属性:
````swift
let validSearchText = Observable<Bool>(false)
````
这个布尔属性标识着当前的searchString是否可用。

依旧在视图模型中添加如下初始化代码:
````swift
searchString
  .map { $0!.characters.count > 3 }
  .bindTo(validSearchText)
````
在这里将searchString观测量映射到一个检测字符串长度是否大于3个字符的布尔值，接着与validSearchText属性相绑定。

在ViewController.swift中找到bindViewModel方法添加如下代码:
````swift
viewModel.validSearchText
  .map { $0 ? UIColor.blackColor() : UIColor.redColor() }
  .bindTo(searchTextField.bnd_textColor)
````
这里将validSearchText属性映射到了颜色，然后与text field的文本颜色属性相绑定。

构建并运行:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/12/ShortText-319x500.png)

结果就是，当文本太短不能构成一个可用的搜索关键字时，就会被标注为红色。

Bond将属性间的联系与变换变得非常简单，简单的就像…

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/01/Swift_bond_pie-480x244.png)
派! 不错, 就像这么简单

##准备搜索

这个app会根据用户的输入在[500px](http://500px.com/) 行检索对应的图片。 giving the same kind of immediate feedback users are used to from Google.
每当searchString视图模型的属性改变时触发一次搜索，不过这可能会引起大量的请求。最好用节流的查询方法，限制每秒最多一次或两次查询。
用Bond很轻松就能实现!

在PhotoSearchViewModel.swift的初始化方法中添加如下代码:
````swift
searchString
  .filter { $0!.characters.count > 3 }
  .throttle(0.5, queue: Queue.Main)
  .observe {
    [unowned self] text in
    self.executeSearch(text!)
  }
````
这里过滤掉searchString中的不可用值(长度 <= 4), 得到我们需要的值，然后限制变化的速度，最快每0.5秒会接受一个通知。

observe的闭包表达式中调用了executeSearch method, 因此需要在后面添加一下:
````swift
func executeSearch(text: String) {
  print(text)
}
````
现在这个方法只是打印出结果，可以清楚的看到节流的效果。

构建并运行，试着输入一些文本。

会看到如下的结果，一些经过节流处理的事件，最快每0.5秒显示一条文本
````swift
Bond
Bond Th
Bond Throttles
Bond Throttles Good!
````
如果没有Bond，要实现这个功能会挺费劲的。
