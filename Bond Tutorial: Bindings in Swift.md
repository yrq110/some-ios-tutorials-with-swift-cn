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

##使用500px API

在这个app中需要使用500px API。选择它是因为其具有相关的简易接口与认证机制。

需要先注册成为开发者才能使用接口。非常快捷，易用，而且是免费！ 
在这个地址中注册: [https://500px.com/signup](https://500px.com/signup)

注册完成后会跳转到应用界面: https://500px.com/settings/applications. 在这里会看到一个允许你注册应用的界面。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/12/RegisterApp-700x201.png)

填写注册表单 (有一小部分是必填) ，创建应用。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2015/12/AppRegistered-700x204.png)

点击应用并复制一下key，每个请求都是使用这个key来访问500px的。

开始工程中已经包含了所需要的在500px中查询的代码，会在Model group中找到。

回到PhotoSearchViewModel.swift文件,添加如下属性:

````swift
private let searchService: PhotoSearch = {
  let apiKey = NSBundle.mainBundle().objectForInfoDictionaryKey("apiKey") as! String
  return PhotoSearch(key: apiKey)
}()
````
初始化了PhotoSearch类，这个类提供了一个500px的Swift检索API，通过key来访问。
key应该在plist文件中所包含. 打开Info.plist编辑一下apiKey, 添加500px提供的key:

![](https://cdn2.raywenderlich.com/wp-content/uploads/2015/12/ApiKey.png)

在PhotoSearchViewModel.swift中如下更新executeSearch方法:

````swift
func executeSearch(text: String) {
  var query = PhotoQuery()
  query.text = searchString.value ?? ""
 
  searchService.findPhotos(query) {
    result in
    switch result {
    case .Success(let photos):
      print("500px API returned \(photos.count) photos")
    case .Error:
      print("Sad face :-(")
    }
  }
}
````
构建了一个PhotoQuery对象，包含了查询的参数。 接着调用了searchService的findPhotos方法执行搜索，使用异步回调的方式返回结果，通过枚举值来表现成功或失败的情况。

构建并运行。视图模型会用结构化的方式来初始化查询，会看到如下的输出日志：
````
500px API returned 20 photos
````
如果出错了会输出:
````
Sad face :-(
````
如果出错了，检查下API key与网络连接，祈祷一下，然后再试一次! 如果还不行的话有可能500px的API挂掉了， 不过99%的情况下是你输错了API。仔细检查下输入的key是否与500px中应用的key相同。


##渲染结果

如果能看到查询结果返回的那些图像岂不是更有趣？让我们来实现它吧。

在PhotoSearchViewModel.swift中添加如下属性:
````swift
let searchResults = ObservableArray<Photo>()
````
顾名思义，这时一个观测量的特殊类型，支持数组功能。

在上手之前，我建议看一下ObservableArray的更多细节，按住cmd点击它看看有关的Bond API。

ObservableArray与Observable类似，都属于一种EventProducer，意味着可以设置监测数组改变的事件。此时，事件会触发ObservableArrayEvent实例。

ObservabelArrayEvent基于ObservableArrayOperation枚举对数组中发生的变化进行了编码。

使用间接枚举定义枚举值的关联:
````swift
public indirect enum ObservableArrayOperation<ElementType> {
  case Insert(elements: [ElementType], fromIndex: Int)
  case Update(elements: [ElementType], fromIndex: Int)
  case Remove(range: Range<Int>)
  case Reset(array: [ElementType])
  case Batch([ObservableArrayOperation<ElementType>])
}
````
如你所见，由观测量数组触发的事件是很详细的，这些事件描述了发生的变化，并没有通知新数组值的观测者。

这种程度的详尽是有充分的理由的。可以使用一个Bond观测量将一个数组绑定到UI，可能是一个列表视图。然而如果绑定到一个单一部件的话，观测量只能表明 *某些量* 改变了，作为结果整个UI都需要被重构。观测量数组所提供的详细状态提供了更加高效的UI更新方式。

>注意：ReactiveCocoa没有观测量数组的概念， 这是我之前偶然发现的。你可以在我[博客](http://blog.scottlogic.com/2014/11/04/mutable-array-binding-reactivecocoa.html)中找到解决方案(很像Bond)。

现在你知道观测量数组是什么了，是时候来使用它了。

在PhotoSearchViewModel.swift中找到executeSearch方法，如下更新case Success中的代码:
````swift
  case .Success(let photos):
    self.searchResults.removeAll()
    self.searchResults.insertContentsOf(photos, atIndex: 0)
````
这里清空了数组，添加了新的结果。

在ViewController.swift中的bindViewModel里添加如下代码:
````swift
viewModel.searchResults.lift().bindTo(resultsTable) { indexPath, dataSource, tableView in
  let cell = tableView.dequeueReusableCellWithIdentifier("MyCell", forIndexPath: indexPath)
                                                                      as! PhotoTableViewCell
  let photo = dataSource[indexPath.section][indexPath.row]
  cell.title.text = photo.title
 
  let qualityOfServiceClass = QOS_CLASS_BACKGROUND
  let backgroundQueue = dispatch_get_global_queue(qualityOfServiceClass, 0)
  cell.photo.image = nil
  dispatch_async(backgroundQueue) {
    if let imageData = NSData(contentsOfURL: photo.url) {
      dispatch_async(dispatch_get_main_queue()) {
        cell.photo.image = UIImage(data: imageData)
      }
    }
  }
  return cell
}
````
Bond中有一个为绑定观测量数组与列表视图所准备的EventProducerType扩展协议。这个绑定需要输入一个列表视图实例与一个闭包表达式，像上面看到的那样，闭包中的代码很像标准的列表视图datasource设置代码。

再看看其中的细节，首先调用了观测量数组的lift方法。Bond支持以嵌套数组结构为目录的列表视图。lift操作将观测量数组转换成了一个无目录列表所有需要的嵌套表单。

闭包表达式中是标准的列表视图列表项的关联代码，列表出列，设置多个属性。注意，图像的下载是基于一个后台队列，为了保持UI的持续响应。

构建并运行应用:

![](https://cdn1.raywenderlich.com/wp-content/uploads/2015/12/ArrayBinding-319x500.png)

当在文本框内输入时，UI会自动更新，是不是很酷。

##些许UI的天分

是时候对UI做的什么了！

在PhotoSearchViewModel.swift中添加如下属性y:
````swift
let searchInProgress = Observable<Bool>(false)
````
更新executeSearch方法中的代码t:
````swift
func executeSearch(text: String) {
  var query = PhotoQuery()
  query.text = searchString.value ?? ""
 
  searchInProgress.value = true
 
  searchService.findPhotos(query) {
    [unowned self] result in
    self.searchInProgress.value = false
    switch result {
    case .Success(let photos):
      self.searchResults.removeAll()
      self.searchResults.insertContentsOf(photos, atIndex: 0)
    case .Error:
      print("Sad face :-(")
    }
  }
}
````
这里在进行查询操作之前将属性直接设为了true，当结果异步返回时则设为false。

在ViewController.swift中，给bindViewModel添加如下代码:
````swift
viewModel.searchInProgress
  .map { !$0 }
  .bindTo(activityIndicator.bnd_hidden)
 
viewModel.searchInProgress
  .map { $0 ? CGFloat(0.5) : CGFloat(1.0) }
  .bindTo(resultsTable.bnd_alpha)
````
在查询进行时会显示一个正在加载的指示器，并且会降低列表视图的透明度。

构建并运行一下，看看效果:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/12/SearchInProgress-319x500.png)

现在应该体会到使用Bond的好处了吧！ 效果像马提尼一样美妙 :]

##处理错误
如果500px的查询失败了，ap仅仅会输出错误信息到控制台中。这里应该以一种有参考意义并规范的方式将失败信息反馈给用户。

问题来了，怎么定义这个模型？一个错误不像视图模型属性是状态的改变，而是瞬态发生的事件。

答案很简单: 一个EventProducer，而不是观测量。在PhotoSearchViewModel.swift中添加如下属性:

````swift
let errorMessages = EventProducer<String>()
````
接着，如下修改executeSearch中Error情况下的代码:
````swift
  case .Error:
    self.errorMessages
      .next("There was an API request issue of some sort. Go ahead, hit me with that 1-star review!")
````
在ViewController.swift中修改bindViewModel:
````swift
viewModel.errorMessages.observe {
  [unowned self] error in
 
  let alertController = UIAlertController(title: "Something went wrong :-(",
                                                        message: error, preferredStyle: .Alert)
  self.presentViewController(alertController, animated: true, completion: nil)
  let actionOk = UIAlertAction(title: "OK", style: .Default,
    handler: { action in alertController.dismissViewControllerAnimated(true, completion: nil) })
 
  alertController.addAction(actionOk)
}
````
这里设置了errorMessages属性触发的事件，使用UIAlert来显示错误信息。

构建并运行应用，然后断开网络连接或者移除key来看看显示的错误信息:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/12/ErrorMessage-319x500.png)

完美 :]

##添加搜索设置

当前应用的查询是基于500px API的简单搜索功能，如果看看API文档的话会发现它支持很多其他的参数。

如果点击app中的Settings按钮，会看到已经有了一个简单的搜索设置UI，现在需要来实现它了。

在视图模型group中添加一个名为PhotoSearchMetadataViewModel.swift的文件，输入如下内容:
````swift
import Foundation
import Bond
 
class PhotoSearchMetadataViewModel {
  let creativeCommons = Observable<Bool>(false)
  let dateFilter = Observable<Bool>(false)
  let minUploadDate = Observable<NSDate>(NSDate())
  let maxUploadDate = Observable<NSDate>(NSDate())
}
````
在PhotoSearchViewModel.swift中添加如下属性:
````swift
let searchMetadataViewModel = PhotoSearchMetadataViewModel()
````
修改executeSearch方法，在设置query.text属性的那行后面添加:
````swift
query.creativeCommonsLicence = searchMetadataViewModel.creativeCommons.value
query.dateFilter = searchMetadataViewModel.dateFilter.value
query.minDate = searchMetadataViewModel.minUploadDate.value
query.maxDate = searchMetadataViewModel.maxUploadDate.value
````
上面的代码将视图模型的状态拷贝给了PhotoQuery对象。

这个视图模型将会与settings视图控制器绑定，是时候来实现了，打开SettingsViewController.swift添加如下属性:
````swift
var viewModel: PhotoSearchMetadataViewModel?
````
那么如何设置这个属性? 当Settings按钮被按下时，故事板会执行一个segue，可以“拦截”这个进程，给视图控制器输入一些数据。

在ViewController.swift中添加如下方法:
````swift
override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
  if segue.identifier == "ShowSettings" {
    let navVC = segue.destinationViewController as! UINavigationController
    let settingsVC = navVC.topViewController as! SettingsViewController
    settingsVC.viewModel = viewModel.searchMetadataViewModel
  }
}
````
这里保证了在ShowSettings segue执行时，目标视图控制器正确加载了视图模型。

在SettingsViewController.swift中viewDidLoad()方法的尾部添加如下代码:
````swift
bindViewModel()
````
然后添加这个函数的实现:
````swift
func bindViewModel() {
  guard let viewModel = viewModel else {
    return
  }
  viewModel.creativeCommons.bidirectionalBindTo(creativeCommonsSwitch.bnd_on)
}
````
上面的代码绑定了视图模型属性与响应的switch开关。

构建并运行应用，打开Settings并触发开关。如果打开的话，会发现加载时返回的照片会有所不同。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/12/Settings-319x500.png)

##绑定日期

Settings界面还有一些需要实现的功能，这就是接下来的任务。

在SettingsViewController.swift中的bindViewModel()中添加如下代码:
````swift
viewModel.dateFilter.bidirectionalBindTo(filterDatesSwitch.bnd_on)
 
let opacity = viewModel.dateFilter.map { $0 ? CGFloat(1.0) : CGFloat(0.5) }
opacity.bindTo(minPickerCell.leftLabel.bnd_alpha)
opacity.bindTo(maxPickerCell.leftLabel.bnd_alpha)
opacity.bindTo(minPickerCell.rightLabel.bnd_alpha)
opacity.bindTo(maxPickerCell.rightLabel.bnd_alpha)
````
将dateFilter开关与视图模型绑定在了一起，并减少了闲置的datePicker单元格的数量。

datePicker单元格包含一个datePicker与几个标签，使用CocoaPod中的DatePickerCell实现，这里有个小问题：Bond并不支持绑定这种单元格类型所暴露的属性。

如果你看看DatePickerCell API的话，会发现一个date属性包含标签与picker。可以对picker进行双向绑定，不过这样的话就意味着跳过了设置标签内容的逻辑。

幸运的是可以“手动”的进行双向绑定，同时观测model与datePicker，很直接的方法。
将如下方法添加到SettingsViewController.swift中:
````swift
private func bind(modelDate: Observable<NSDate>, picker: DatePickerCell) {
  modelDate.observe {
    event in
    picker.date = event
  }
 
  picker.datePicker.bnd_date.observe {
    event in
    modelDate.value = event
  }
}
````

然后将这两行添加到bindViewModel()中:
````swift
bind(viewModel.minUploadDate, picker: minPickerCell)
bind(viewModel.maxUploadDate, picker: maxPickerCell)
````
构建并运行，欢呼吧! 日期现在被正确绑定了:

![](https://cdn2.raywenderlich.com/wp-content/uploads/2015/12/DateBinding-319x500.png)
