#CloudKit Tutorial: Getting Started
##CloudKit指南：入门


***

>* 原文链接 : [CloudKit Tutorial: Getting Started](https://www.raywenderlich.com/134694/cloudkit-tutorial-getting-started)
* 原文作者 : [Ed Sasena](https://www.raywenderlich.com/u/anros)
* 译者 : [yrq110](https://github.com/yrq110)

***

> 更新: 此教程已更新至 Xcode 8, iOS 10, Swift 3。原教程由Michael Katz所著。

CloudKit是苹果基于iCloud的远程数据存储服务。提供了一个低开销的存储与分享app数据的方案，使用用户的iCloud账户进行后端存储。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/CloudKit-feature-1-250x250.png)

学习如何使用CloudKit在云端存储你的app数据!

CloudKit中的两个主要部分:

1. 一个管理record type与其它公共数据的web控制台。
2. 一个在iCloud与设备之间传输数据的API集合。

CloudKit同样很安全，用户的私有数据被完全保护了起来，开发者只能访问他们自己的私有数据库而不能查看用户的私有数据。

对于一个仅有iOS平台并且使用大量数据的app来说，CloudKit是一个好的选择，无需编写服务端的一些繁琐的逻辑。此外，CloudKit也可以应用于web与服务器应用。

在这篇CloudKit教程中，会使用CloudKit创建一个餐厅评分app来进行实践，它有一个拗口的名字-BabiFüd.

> 注意: 进行这篇CloudKit教程中的示例app开发时，需要一个激活的iOS开发者账号，若没有的话将无法使用iCloud entitlements与访问CloudKit控制台。

## 为何使用CloudKit?

也许你会疑惑为何不选Core Data或其他商业型BaaS服务，甚至是自己的服务器。

这里有三个原因: 简便, 信任, 与开销。

### 简便

不像其它的后端解决方案，CloudKit仅需要少量的配置即可使用。无需进行服务器的选择、配置与安装，并且安全问题也是由苹果来处理。

进行简单的iOS开发者注册后即可使用CloudKit，不需要注册额外的服务或创建新账号，当在app中启用CloudKit功能时所有必要的服务器都会神奇的自动设置完成。

无需下载额外的库与配置。CloudKit与其它iOS框架同等重要，CloudKit框架本身也提供了一些针对于通用操作的方便的API。

对用户来说很简单，CloudKit在设备启动好时通过iCloud证书来登入(或者在启动后的设置界面登入)，无需构建复杂的登录界面。只要登入成功后，就可以直接使用你的app了。That should put you on Cloud 9(PS: 也许指的是北美的那支游戏战队)!

### 信任

另一个使用CloudKit的好处就是用户可以放心它们数据的隐私性与安全性，这些数据都是由苹果处理的而不是app开发者。CloudKit将用户数据与开发者隔离开来。

虽然这个访问权限的缺失在debug过程中也许会使人沮丧，不过你不用担心安全性并且思考如何使用户相信他们的数据是安全的。若用户信任iCloud的话他们也会信任你。

### 开销

最后，对开发者而言，运行一个服务总会产生一笔不小的开销，即使是价格最便宜的服务器主机也很难有为小型免费的app提供的低开销解决方案。因此在运行一个app时难免会产生一个相关的开销。

使用CloudKit的话会得到一个免费的用于公共数据传输的容量适当的存储空间。在WWDC 2015视频中的[What’s New in CloudKit](https://developer.apple.com/videos/play/wwdc2015/704/)详细讲解了它的费用。

强有力的CloudKit能为Mac与iOS app提供一个麻烦少的解决方案。

## BabiFüd简介

BabiFüd是这篇教程中的示例app，一个标准的“餐厅评价”类app。然而不通过食物质量、服务速度与价格来进行评估，而是通过针对儿童的友好度，这就包含了可更换的设施，安全座椅与可选择的健康食品选择。

app中包含4个标签：一个附近餐厅的列表，一张附近餐厅的地图，用户评价与设置。附近餐厅的列表是唯一会在教程中用到的，看看下面的app截图

![](http://www.raywenderlich.com/wp-content/uploads/2014/09/nearby.png)
![](http://www.raywenderlich.com/wp-content/uploads/2014/09/map.png)

这些视图后有一个模型类，与调用CloudKit的操作绑定。CloudKit对象被称为records，模型中主要的record类型是**Establishment**，代表app中不同的餐厅。

## 开始

先下载这篇教程的[开始工程](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/BabiFud-Cloudkit-Starter.zip)。

在开始编码前需要修改下Bundle Identifier和app的Team属性，需要设置team来从Apple那里获权，给予一个独一无二的bundle identifier会省不少事儿。

在Xcode中打开BabiFud.xcodeproj，工程导航栏中选择BabiFud工程，接着选择BabiFud target，选中General标签，修改Bundle Identifier，一个经典的做法是反转标记的区域名称再加上工程名称，接着选择适合的Team:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S1-BabiFud-Bundle-ID-480x138.png)

好了，现在需要设置CloudKit，创建一些存放数据的容器(container)。

## 授权与容器

通过app添加数据前需要一个存放app记录(record)的容器。容器是存放在服务器的保存所有app数据的一个概念性位置，分为公有数据库与私有数据库。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/CloudKit-container-diagram.png)

为了创建一个容器，首先需要启用app的iCloud功能，选择target中的Capabilities标签，将iCloud区域的开关拨到ON。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S2-Enable-iCloud-480x220.png)

这时，Xcode可能会让你输入与iOS开发者账号关联的Apple ID，若有的话就输入下。最后，在Services分组里勾选CloudKit复选框来启用CloudKit。

这样就创建了一个默认的名为iCloud.<你的bundle id>的容器，如下所示:

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S3-Enable-CloudKit-480x220.png)

### Xcode中配置iCloud的一些问题

如果你在创建entitlement，构建构成，运行app或者Xcode提示容器ID时遇到警告或错误时，这里有一些提示:

* 若在iCloud部分设置时出现警告或错误，试试点击Fix Issue按钮，可能需要多试几次。
  ![](http://www.raywenderlich.com/wp-content/uploads/2014/09/0_fix_issue.png)
* 有一点很重要：app的bundle id需要与iCloud容器相对应，并且在开发者账号中是存在的。比如说，bundle id是“com.<your domain>.BabiFud”，则iCloud容器名称就应该是“iCloud.”加上bundle id变成“iCloud.com.<your domain>.BabiFud”。
* iCloud容器名必须是唯一的，因为这是Cloudkit用来访问数据所使用的全局标识符。由于iCloud容器名包含bundle id，因此bundle id也必须是唯一的(这就是为何需要修改com.raywendrelich.BabiFud)。
* 为了让entitlements起作用，需要在App的证书、标识符与配置文件中ID的部分列出app/bundle id。这意味着标识的证书使用了设置的team id与app id，从中可得到iCloud容器的id。若已经在一个可用的开发者账号中标识了的话Xcode会自动完成这一切。不巧的是，这有时是不同步的，需要更新ID-使用iCloud功能面板修改CloudKit容器ID。否则的话需要修改info.plist文件或BabiFud.entitlements文件来确保id values与所设置的bundle id一致。

## CloudKit控制台介绍 

设置完必要的entitlements后，下一步就是创建与定义app所使用的record type，可以使用CloudKit控制台来做这件事。在target的Capabilities面板中，在iCloud下方点击CloudKit Dashboard即可。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/S5-CloudKit-Dashboard-Button-480x220.png)

> 提示: 也可以在浏览器中用URL打开CloudKit控制台https://icloud.developer.apple.com/dashboard/ 

控制台长这样:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/S6-CloudKit-Dashboard-480x262.png)

CloudKit控制台由四个部分组成: Schema(方案), Public Data(公有数据), Private Data(私有数据), and Admin(用户)。

SCHEMA区域展示了CloudKit容器的高级对象: Record Types(记录类型), Security Roles(安全规则), and Subscription Types(订阅类型). 在此教程中只需要关注Record Type。You’ll only be concerned with Record Types in this tutorial.

一个Record Type就是定义了一些record个体的集合。就像面向对象编程中的class。一个record可以作为一个特殊的Record Type实例，表示容器中的结构化数据，就像数据库中的一行数据，封装了一系列的键值对。 

PUBLIC DATA与PRIVATE DATA区域允许你添加、搜索数据。记住，作为一个开发者可以访问所有公有数据与自己的私有数据。User Records存储当前iCloud用户数据，比如姓名与邮箱。 Record Zone中通过进行分组记录来给私有数据库提供一个逻辑性的组织结构。 Custom zones支持原子事务(PS:原子事务：web服务上的操作或者全部发生，或者根本不发生。)，允许多个记录在进行其它操作前同时存储。此教程中不涉及Custom zones。

ADMIN区域允许你配置team成员的控制台访问权限。若你的team中有多个开发者同伴，可以在这里编辑他们的权限。同样，这个也不是此教程的内容。

## 添加商家(Establishment)的Record Type

考虑一下app中如何设计，每一条想要获取的信息都含有多个数据：名称，地点，还有多种针对儿童友好度的选项。Record types使用字段来定义记录中的多种数据。

选择Record Types，点击左上角的 + 号添加一个新的record type。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S7-Add-Record-Type-480x139.png)

将这个新的record type命名为Establishment。

会看到一行字段，可以在这里定义Field Name(字段名), Field Type(字段类型) 与 Index(索引)，如下所示。 自动创建的的默认字段名为StringField。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S8-New-Record-Type-480x284.png)

首先替换字段名，字段类型与索引在首次定义时是默认的值，需要改变一些字段。点击**Add Field…** 添加所需的字段，如下所示:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/T1-Entitlement-Record-Fields.png)

搞定后列表应像下面这样:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/S10-List-of-Fields-308x320.png)

点击页面上方的Save进行保存。

好了，现在在数据库中添加一些样本记录。

选择PUBLIC DATA区域的Default Zone，这个区域包含app中的公有记录。从中间面板的列表中选择Establishment记录类型，选择 + 图标或右侧细节面板的New Record按钮，如下图所示:

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S11-Create-New-Establishment-Record-480x183.png)

会创建一个新的Establishment记录。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S12-New-Establishment-Record-Blank-415x320.png)

这时需要录入一些用于app测试的数据。

下面的这些商家数据是虚构的，都设定在Apple总部的附近，容易在模拟器中找到。

录入下面这些记录:

> 注意: 图中的CoverPhoto元素在Xcode项目中的Supporting Files\Sample Images文件夹。将其拖拽到CoverPhoto字段即可添加到establishment record中。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/T2-Entitlement-Records-Data.png)

保存完成后，在控制台中应如下所示:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S14-All-Three-Records-429x320.png)

对于每条记录，输入的值都是数据库中所表现的数据。在app端数据类型是不同的。比如说SeatingType与ChangingTable是结构体，给SeatingType分配的整型值对应“high chair”或“booster”。对于HealthyOption与KidsMenu, 用整型值来表示布尔值，0意味着商家没有这个选项，1则是有。

运行app时需要你有一个用于开发的iCloud账号。

[创建一个开发用的iCloud账号](https://developer.apple.com/library/tvos/documentation/DataManagement/Conceptual/CloudKitQuickStart/EnablingiCloudandConfiguringCloudKit/EnablingiCloudandConfiguringCloudKit.html#//apple_ref/doc/uid/TP40014987-CH2-SW7)

也需要在iOS模拟器中录入与这个账号关联的iCloud证书。

[运行app前录入iCloud证书](https://developer.apple.com/library/tvos/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW12)

回到Xcode中，是时候在app中整合数据了!

## 查询商家记录

使用CKQuery对象从数据库中选择记录，找到所有符合一定规则的记录，这个规则可以是“所有name是M开头的记录”，“包含增高座椅的记录”，“3km之内的记录”。这些表达式在Cocoa中使用NSPredicate对象来编写，NSPredicate通过匹配特殊的规则来评估对象是否符合要求。谓词也同样用在CoreData中，自然支持CloudKit。

CloudKit仅支持一部分可用的NSPredicate函数，包含数学的比较、一些字符串与设置的操作(例如“匹配列表中的一项的字段”)，和一个特殊的距离函数。NSPredicate的distanceToLocation:fromLocation:函数通过与已知位置相距一个特殊半径的位置字段来匹配CloudKit中的记录，这种类型的谓词会在之后来详细说明。对于其他查询类型，看看苹果的官方文档[CKQuery Class 
Reference](https://developer.apple.com/library/ios/documentation/CloudKit/Reference/CKQuery_class/)，列表中包含了所支持的函数与它们的使用方法。

> 注意: CloudKit支持CLLocation对象(CoreLocation框架的对象，包含地理空间坐标)，这使得查询一个地理区域内的商家变得很简单 - 不需要做任何额外的坐标变换等操作。

在Xcode中打开Model/Model.swift，这个文件中包含了app中需要的所有调用服务器方法的存根。

使用如下代码替换fetchEstablishments(\_:radiusInMeters:)方法:

```swift
func fetchEstablishments(_ location:CLLocation, radiusInMeters:CLLocationDistance) {
  // 1
  let radiusInKilometers = radiusInMeters / 1000.0
  // 2
  let locationPredicate = NSPredicate(format: "distanceToLocation:fromLocation:(%K,%@) < %f", "Location", location, radiusInKilometers)
  // 3
  let query = CKQuery(recordType: EstablishmentType, predicate: locationPredicate)
  // 4
  publicDB.perform(query, inZoneWith: nil) { [unowned self] results, error in
    if let error = error {
      DispatchQueue.main.async {
        self.delegate?.errorUpdating(error as NSError)
        print("Cloud Query Error - Fetch Establishments: \(error)")
      }
      return
    }
    self.items.removeAll(keepingCapacity: true)
    results?.forEach({ (record: CKRecord) in
      self.items.append(Establishment(record: record,
                                      database: self.publicDB))
    })
    DispatchQueue.main.async {
      self.delegate?.modelUpdated()
    }
  }
}
```

逐步分析下:

1. CloudKit中距离谓词的单位是千米，这行代码将radiusInMeters的单位转换为千米。
2. 谓词过滤出的商家是基于离当前位置一定的距离得到的。这个语句根据用户当前位置找到所有一定距离内的商家。
3. 使用一个谓词与一个record type来创建CKQuery对象，两者在执行查询时都会被用到。
4. 最后，performQuery(\_:inZoneWithID:completionHandler:)将你的查询提交给iCloud，并等待查询结果。将inZoneWithID参数设为nil，即在你的默认区域中执行查询，这个默认区域就是公有数据库。若你想同时在私有与公有数据库中检索数据，需要分别调用每个数据库的查询功能。

你也许会疑问CKDatabase实例与publicDB到底来自哪里？看看Model.swift的文件顶部。

```swift
let container: CKContainer
let publicDB: CKDatabase
let privateDB: CKDatabase
 
init() {
  // 1
  container = CKContainer.defaultContainer() 
  // 2
  publicDB = container.publicCloudDatabase 
  // 3
  privateDB = container.privateCloudDatabase 
}
```
在这里定义了数据库:

1. 这个默认容器就是在iCloud Capabilities面板中的那个。
2. 公有数据库中的数据由所有app的用户共享。
3. 私有数据库仅包含当前登入用户的数据。

这段代码会从公有数据库中检索到一些本地的商家数据，不过需要将这些数据与一个view controller结合才能在app中显式的看到。

### 准备必要的回调函数

可以用熟悉的代理模式来编写notification函数，在Model.swift的顶部有需要在view controller中实现的协议:
```swift
protocol ModelDelegate {
  func errorUpdating(error: NSError)
  func modelUpdated()
}
```
打开MasterViewController.swift，使用下面的代码替换modelUpdated()方法:

```swift
func modelUpdated() {
  refreshControl?.endRefreshing() 
  tableView.reloadData() 
}
```

当有新的可用数据时会调用这个方法，在tableView(\_:cellForRowAtIndexPath:)方法中已经将所有列表项与CloudKit对象绑定好了，可以如释重负的去瞥一眼。

接下来，在MasterViewController.swift中使用如下代码替换errorUpdating(\_:)方法:

```swift
func errorUpdating(_ error: NSError) {
  let alertController = UIAlertController(title: nil,
                                          message: error.localizedDescription,
                                          preferredStyle: .alert)
 
  alertController.addAction(UIAlertAction(title: "Dismiss", style: .default, handler: nil))
 
  present(alertController, animated: true, completion: nil)
}
```
当查询过程产生错误时会调用这个方法。较差的网络条件或者特殊的CloudKit问题(比如缺失或错误的用户证书、没有返回结果的查询)都会引起错误。

在操作各种远程服务器时一个良好的错误处理是很必要的，在这里仅仅给用户显示错误信息。

有个很常见的问题：用户未登入iCloud或者没有为app打开iCloud drive。你能修改一下errorUpdating(\_:)来处理这几种情况吗？ 提示: 两种错误均会返回一个值为1的CKErrorCode。

解决方法：在MasterViewController.swift文件中使用如下代码替换errorUpdating(_:)方法:
```swift
func errorUpdating(_ error: NSError) {
  let message: String
  if error.code == 1 {
    message = "Log into iCloud on your device and make sure the iCloud drive is turned on for this app."
  } else {
    message = error.localizedDescription
  }
  let alertController = UIAlertController(title: nil,
                                          message: message,
                                          preferredStyle: .alert)
 
  alertController.addAction(UIAlertAction(title: "Dismiss", style: .default, handler: nil))
 
  present(alertController, animated: true, completion: nil)
}
```

构建并运行，应该会看到一个附近商家的列表。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Build-Run-1-No-Images-179x320.png)

### 调试时的问题

如果你使用的是iOS模拟器，并且列表没有被填充，确认一下你的地理位置设置是正确的(Xcode菜单:Debug\Simulate Location\San Francisco, CA, USA)。若需要修改在Xcode中设置的位置，在app中下拉列表刷新一下位置信息。

若你使用的iPhone或iPad，位置服务是可用的并且商家位置并不在你当前位置的附近的话，此时有两个选择:改变样例数据的坐标使其位置靠近你当前的位置或者使用模拟器运行app。不过这里还有第三个选项：你可以去苹果总部附近转转。

若数据没有正确显示或显示不完全，则使用CloudKit控制台检查一下样例数据。为了保证所有记录都是存在的，要将它们添加到默认域中并且有正确的值。若要重新输入数据，可以点击垃圾箱图标删除记录。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S16-Delete-Record-1-480x122.png)

有时修复CloudKit的错误是很棘手的。CloudKit的错误消息并不包含大量的信息，为了搞清楚错误发生的原因需要结合当前所操作数据库的错误代码。使用数字型错误代码，查找对应的CKErrorCode枚举值。查看文档中的名称与描述有助于找到问题所在，看看下面的例子。

> 注意: 可以在[CloudKit Framework Constants Reference](https://developer.apple.com/library/ios/documentation/CloudKit/Reference/CloudKit_constants/#//apple_ref/c/tdef/CKErrorCode)中查看CloudKit返回的错误代码列表。

这里是一些常见的错误枚举值与相关描述:
* .BadContainer – 指定的容器未知或未授权。
* .NotAuthenticated – 当前用户没有认证，没有可用的用户记录。当用户没有登入iCloud时可能会出现这个错误。
* .UnknownItem – 指定的记录不存在。

当抓取到商家列表数据时，会注意到可以看到商家名称与提供的服务，不过没有显示的图片，被云挡住了？

这是因为虽然在检索商家记录的同时也会自动检索图片数据，不过你需要执行一些必要的步骤去加载图片才能显示出来，这样就可以让云散去了! :]

## Working with Binary Assets

An asset is binary data, such as an image, that you associate with a record. In your case, your app’s assets are the establishment photos shown in the MasterViewController table view.
In this section you’ll add the logic to load the assets that were downloaded when you retrieved the establishment records.
Open Model/Establishment.swift and replace the loadCoverPhoto(_:) method with the following code:
```swift
func loadCoverPhoto(completion:@escaping (_ photo: UIImage?) -> ()) {
  // 1
  DispatchQueue.global(qos: DispatchQoS.QoSClass.background).async {
    var image: UIImage!
    defer {
      completion(image)
    }
      // 2
    guard let asset = self.record["CoverPhoto"] as? CKAsset else {
      return
    }
 
    let imageData: Data
    do {
      imageData = try Data(contentsOf: asset.fileURL)
    } catch {
      return
    }
    image = UIImage(data: imageData)
  }
}
```
This method loads the image from the asset attribute as follows:

1. Although you download the asset at the same time you retrieve the rest of the record, you want to load the image asynchronously. So wrap everything in a dispatch_async block.
2. Assets are stored in CKRecord as instances of CKAsset, so cast appropriately. Next load the image data from the local file URL provided by the asset.
3. Use the image data to create an instance of UIImage.
4. Execute the completion callback with the retrieved image. Note that this defer block gets executed regardless of which return is executed. For example, if there is no image asset, then image never gets set upon the return and no image appears for the restaurant.

Build and run. You’ve chased the clouds away and the establishment images should now appear. Great job!

![](https://koenig-media.raywenderlich.com/uploads/2016/06/Build-Run-2-With-Images-180x320.png)

There are two gotchas with CloudKit assets:

1. Assets can only exist in CloudKit as attributes on records. You can’t store them on their own. Deleting a record will also delete any associated assets.
2. Retrieving assets can negatively impact performance because the assets are downloaded at the same time as the rest of the record data. If your app makes heavy use of assets, then you should store a reference to a different type of record that holds just the asset.
