#Swift Tutorial: Working with JSON
##Swift指南：使用JSON

***

>* 原文链接 : [Swift Tutorial: Working with JSON](https://www.raywenderlich.com/120442/swift-json-tutorial)
* 原文作者 : [Attila Hegedüs](https://www.raywenderlich.com/u/cynicalme)
* 译者 : [yrq110](https://github.com/yrq110)

***

![swift](https://cdn3.raywenderlich.com/wp-content/uploads/2014/06/swift_tut-250x250.jpg)
> 更新至Xcode 7.1与Swift 2   12-21-2015

[JavaScript Object Notation](http://www.json.org/)或简称`JSON`，是一种与Web端服务进行数据传递的通用方法。它具有易用与可读性强的特点，这也是它为何会这么火的原因。

考虑一下下面这段JSON：

````swift
[
  {
    "person": {
      "name": "Dani",
      "age": "24"
    }
  },
  {
    "person": {
      "name": "ray",
      "age": "70"
    }
  }
]
````

在`Objective-C`中，对JSON的解析与反序列化是很明确的：
````Objective-C
NSArray *json = [NSJSONSerialization JSONObjectWithData:JSONData options:kNilOptions error:nil];
NSString *age = json[0][@"person"][@"age"];
NSLog(@"Dani's age is %@", age);
````
在`Swift`中，由于Swift的可选类型与类型安全，同样的操作显得更加繁琐：
````swift
var json: Array!
do {
  json = try NSJSONSerialization.JSONObjectWithData(JSONData, options: NSJSONReadingOptions()) as? Array
} catch {
  print(error)
}
 
if let item = json[0] as? [String: AnyObject] {
  if let person = item["person"] as? [String: AnyObject] {
    if let age = person["age"] as? Int {
      print("Dani's age is \(age)")
    }
  }
}
````
在上面的代码中，当你使用JSON中的每一个object之前都需要通过`可选绑定`(optional binding)进行检查。这会保证你代码的安全性，但是若你的JSON越复杂，你的代码就会变得越丑。

使用Swift2.0中的`guard`语句可以帮助避免if语句的嵌套：
````swift
guard let item = json[0] as? [String: AnyObject],
  let person = item["person"] as? [String: AnyObject],
  let age = person["age"] as? Int else {
    return;
}
print("Dani's age is \(age)")
````
还是太冗长，不是吗？你怎样才能使它更简洁？

在这篇JSON教程中，你会学到一种更轻松的解析JSON的方法——使用开源库[Gloss](https://github.com/hkellaway/Gloss)

你将会使用Gloss去解析一个包含US App商店Top25应用信息的JSON文档。你将会发现在Objective-C中也能轻易的使用！


**目录**
* [入门](#入门)
* [用Swift原生方法解析JSON](#swift-json)
* [JSON对象映射介绍](#ob-map)
* [使用Gloss解析JSON](#gloss)
  * [在你的工程中集成Gloss](#integ-gloss)
  * [映射JSON到对象](#map-obj)
    * [TopApps](#TopApps)
    * [Feed](#Feed)
    * [App](#App)
  * [检索远程JSON](#ret-json)
  * [Gloss的工作原理](#hook-gloss)

##入门
下载这篇教程的[开始playground](http://www.raywenderlich.com/wp-content/uploads/2015/11/TopApps-Starter.zip)

由于用户界面在这篇JSON教程中并不是很重要，因此你将只使用playgrounds进行学习。

在Xcode中打开Swift.playground看一看。

>你可能会发现playground工程的导航默认是关闭的。使用`command`+`1`来显示它。

在开始playground文件中有一些source与resource文件，会使你更加专注于使用Swift进行JSON的解析。看一看它的结构与概要：
* `Resources`文件夹绑定了你的Swift代码中要访问的资源文件
  * `topapps.json`：包含需要解析的JSON字符串
* `Sources`文件夹包含你的主playground可以访问的其他Swift源文件，在Sources文件夹中添加额外的.swift文件辅助会使你的playground变得更加简洁与易读。
  * `App.swift`：一个原始的swift结构体，代表一个app。你的目标就是解析JSON转换为这些对象的一个集合。
  * `DataManager.swift`：管理来自本地或网络的数据检索，之后你将会使用这个文件中的方法加载一些JSON。

当你感到对当前的playground充分理解后请继续向下阅读！

<a name="swift-json"></a>
##用Swift原生方法解析JSON
首先，你需要从使用Swift原生方法解析JSON开始，不使用任何的外部库。这将会帮助你理解使用像Gloss这样的库的益处。
>若你已经深知使用原生方法解析JSON的痛处，请直接跳到下一节

来让我们从提供的JSON文件中提取出AppStore中排名第一App的名字吧！

首先，当准备大量使用字典时，在playground的头部定义一个类型别名：
````swift
typealias Payload = [String: AnyObject]
````
如下面所示完善回调函数getTopAppsDataFromFileWithSuccess中的代码：
````swift
DataManager.getTopAppsDataFromFileWithSuccess { (data) -> Void in
 
  var json: Payload!
 
  // 1
  do {
    json = try NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions()) as? Payload
  } catch {
    print(error)
    XCPlaygroundPage.currentPage.finishExecution()
  }
 
  // 2
  guard let feed = json["feed"] as? Payload,
    let apps = feed["entry"] as? [AnyObject],
    let app = apps.first as? Payload
    else { XCPlaygroundPage.currentPage.finishExecution() }
 
  guard let container = app["im:name"] as? Payload,
    let name = container["label"] as? String
    else { XCPlaygroundPage.currentPage.finishExecution() }
 
  guard let id = app["id"] as? Payload,
    let link = id["label"] as? String
    else { XCPlaygroundPage.currentPage.finishExecution() }
 
  // 3
  let entry = App(name: name, link: link)
  print(entry)
 
  XCPlaygroundPage.currentPage.finishExecution()
 
}
````
上面发生了什么：

1. 首先使用`NSJSONSerialization`对数据进行了反序列化
2. 需要逐一检查JSON对象中的所有值确保他们不为空(nil)。一旦确定它含有一个有效的值则继续检查下一个object。当使用这种方法遍历了所有下标则会得到所需要的name与link值。
3. 最后一步就是使用name与link值初始化一个App对象并使用控制台输出。

保存并运行stroyboard，应该会在调制窗口中有如下的结果：
>App(name: "Game of War - Fire Age", link: "https://itunes.apple.com/us/app/game-of-war-fire-age/id667728512?mt=8&uo=2")

是的。“Game of War – Fire Age”就是JSON文件中的第一个app

仅仅为了检索出第一个app的名字就用了这么冗长的代码，是时候看看Gloss的本事了。

<a name="ob-map"></a>
##JSON对象映射介绍
对象映射是一个将JSON对象转换为本地Swift对象的一种技术。在定义后模型对象与响应的映射规则后，Gloss就会代替你承担解析的重担。

这种方法明显优于你刚刚尝试的方法：
* 代码会变得更加简洁、可复用并且容易维护
* 可以操作对象而不是数组与字典
* 可以扩展模型类，添加一些额外功能

听起来不错？让我们来看看它是怎样工作的！

<a name="gloss"></a>
##使用Gloss解析JSON
为了干净工整的代码环境，新建一个名为Gloss.playground的playground，然后将topapps.json复制进Resources中，将DataManager.swift复制进Soureces中。

<a name="integ-gloss"></a>
###在你的工程中集成Gloss
在工程/playground中集成Gloss：
1. 点击下面的地址[Gloss Repo Zip File](https://github.com/hkellaway/Gloss/archive/master.zip)然后保存到一个熟悉的地方。
2. 解压后将Gloss-master/Gloss/Gloss文件夹拖进你的Sources文件夹中。

你的工程导航栏应该像下面一样：

![image](http://www.raywenderlich.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-14-at-8.23.16-PM.png)

成功了！现在你将Gloss添加进palyground了，可以不用使用令人头疼的可选绑定来解析JSON了！
>也可以使用Cocoapods来安装Gloss。由于playground已经不再支持，只能在标准的Xcode工程中使用这种方法

<a name="map-obj"></a>
###映射JSON到对象
首先，需要根据你的JSON文档定义自己的模型对象

模型对象必须遵守`Decodable`协议，会使他们能。为了实现它，需要在协议中实施`init?(json: JSON)`进行初始化

观察一下topapps.json的结构来创建数据模型吧！

<a name="TopApps"></a>
####TopApps
TopApps模型代表最顶层的对象，它包含一个键值对：
````swift
{
  "feed": {
    ...
  }
}
````
在playground的sources文件夹下创建一个名为`TopApps.swift`的swift文件并添加如下代码：
````swift
public struct TopApps: Decodable {
 
  // 1
  public let feed: Feed?
 
  // 2
  public init?(json: JSON) {
    feed = "feed" <~~ json
  }
 
}
````
1. 定义你模型的属性。在这里仅仅只有一个。接下来你会定义`Feed`模型对象
2. 确保`TopApps`遵守`Decodable`协议并进行了自定义初始化

你也许想知道<~~是个啥。它叫做编码操作符(Encode Operator)，在Gloss中的Operators.swift文件中所定义。它会告诉Gloss抓取属于key键的值或对于进行编码。`Feed`是一个`Decodable`对象，因此Gloss可以将编码的任务委托给这个类。

<a name="Feed"></a>
####Feed
`Feed`对象与顶层的对象非常相似。`Feed`有两对键值。你只对top25的app感兴趣，因此不需要去处理author对象。
````swift
{
  "author": {
    ...
  },
  "entry": [
    ...
  ]	
}
````
在playground的`sources`文件夹下创建一个名为`Feed.swift`的swift文件并添加如下代码：
````swift
public struct Feed: Decodable {
 
  public let entries: [App]?
 
  public init?(json: JSON) {
    entries = "entry" <~~ json
  }
 
}
````

<a name="App"></a>
####App
`App`是定义的最后一个模型对象。它代表表中的一个app：
````swift
{
  "im:name": {
    "label": "Game of War - Fire Age"
  },
  "id": {
    "label": "https://itunes.apple.com/us/app/game-of-war-fire-age/id667728512?mt=8&uo=2",	
    ...
  },
  ...
}
````
在playground的sources文件夹下创建一个名为`App.swift`的swift文件并添加如下代码：
````swift
public struct App: Decodable {
 
  // 1
  public let name: String
  public let link: String
 
  public init?(json: JSON) {
    // 2
    guard let container: JSON = "im:name" <~~ json,
      let id: JSON = "id" <~~ json
      else { return nil }
 
    guard let name: String = "label" <~~ container,
      link: String = "label" <~~ id
      else { return nil }
 
    self.name = name
    self.link = link
  }
 
}
````
1. `Feed`与`Topapp`都使用的可选属性。如果你确信JSON中总有填充值的话可以定义一个非可选属性。
2. 不需要为JSON中的每一个成员都创建模型对象。

现在模型类已经准备好了，让Gloss工作吧！

打开playground文件并添加如下代码：
````swift
import UIKit
import XCPlayground
 
XCPlaygroundPage.currentPage.needsIndefiniteExecution = true
 
DataManager.getTopAppsDataFromFileWithSuccess { (data) -> Void in
  var json: [String: AnyObject]!
 
  // 1
  do {
    json = try NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions()) as? [String: AnyObject]
  } catch {
    print(error)
    XCPlaygroundPage.currentPage.finishExecution()
  }
 
  // 2
  guard let topApps = TopApps(json: json) else {
    print("Error initializing object")
    XCPlaygroundPage.currentPage.finishExecution()
  }
 
  // 3
  guard let firstItem = topApps.feed?.entries?.first else {
    print("No such item")
    XCPlaygroundPage.currentPage.finishExecution()
  }
 
  // 4
  print(firstItem)
 
  XCPlaygroundPage.currentPage.finishExecution()
 
}
````
1. 首先对数据进行了反序列化操作，之前你也这么做过。
2. 使用JSON数据初始化一个静态的`Topapps`对象
3. 得到feed中的第一项，也就是第一个app
4. 在控制台输出app

讲真，这是你需要的所有代码。

保存并运行storyboar你会看到再次成功获得app的名字，不过这次是以更优雅的方式：
````swift
App(name: "Game of War - Fire Age", link: "https://itunes.apple.com/us/app/game-of-war-fire-age/id667728512?mt=8&uo=2")
````
这里关注的是解析本地数据，你如何解析一个远程的数据？

<a name="ret-json"></a>
###检索远程JSON
是时候将这个工程变得更加实用了。通常会检索远程数据而不是本地的。可以通过网络请求轻易的得到AppStore的排行榜。

打开`Datamanager.swift`并定义Top Apps URL：
````swift
let TopAppURL = "https://itunes.apple.com/us/rss/topgrossingipadapplications/limit=25/json"
````
接着添加如下方法:
````swift
public class func getTopAppsDataFromItunesWithSuccess(success: ((iTunesData: NSData!) -> Void)) {
  //1
  loadDataFromURL(NSURL(string: TopAppURL)!, completion:{(data, error) -> Void in
      //2
      if let data = data {
        //3
        success(iTunesData: data)
      }
  })
}
````
上面的代码看起来很熟悉，不过与检索本地文件不同的是，你使用`NSURLSession`从iTunes上获得了数据。里面发生了什么：

1. 当第一次调用`loadDataFromURL`时，使用`NSData`对象中的的URL与一个completion闭包作为输入参数。
2. 接着使用可选绑定确保data存在
3. 最后将data传递给success

打开storyboard并将下面这行:
````swift
DataManager.getTopAppsDataFromFileWithSuccess { (data) -> Void in
````
替换为
````swift
DataManager.getTopAppsDataFromItunesWithSuccess { (data) -> Void in
````
现在你可以从iTunes上检索真实的数据了。

保存并运行你的storyboard，你将会看到数据解析的结果依然是第一个app：
~~~~
App(name: "Game of War - Fire Age", link: "https://itunes.apple.com/us/app/game-of-war-fire-age/id667728512?mt=8&uo=2")
~~~~
上面的值可以会与你的不同，毕竟AppStore的top apps是会变的。

一般人们不仅会对AppStore的top app感兴趣，他们想看到所有top apps的一个列表。为了满足这个不需要写任何额外的代码，可以使用下面这个代码段轻易的实现：
~~~~
topApps.feed?.entries
~~~~

<a name="hook-gloss"></a>
###Gloss的工作原理
可以看到Gloss在解析处理方面的卓越成效，不过它是怎么工作的呢？

<~~是`Decoder.decode`功能中的一个`自定义操作符`(custom operator)。Gloss中支持多种类型的解码：
* Simple types (Decoder.decode)
* Decodable models (Decoder.decodeDecodable)
* Simple arrays (Decoder.decode)
* Arrays of Decodable models (Decoder.decodeDecodableArray)
* Enum types (Decoder.decodeEnum)
* Enum arrays (Decoder.decodeEnumArray)
* NSURL types (Decoder.decodeURL)
* NSURL arrays (Decode.decodeURLArray)

>如果你想了解关于自定义操作符的更多信息，可以看看这个教程：[Operator Overloading in Swift Tutorial](http://www.raywenderlich.com/80818/operator-overloading-in-swift-tutorial)

在这个教程中严重依赖`Decodable`模型。如果你需要实现更复杂的类型，可以扩展`Decoder`并提供你自己的解码implementation

当然Gloss也可以将对象转换为JSON。如果你对这个感兴趣可以看看`Encodable`协议。

这里是教程的[完整playground](http://www.raywenderlich.com/wp-content/uploads/2015/12/Gloss-final.zip)
