#CloudKit Tutorial: Getting Started
##CloudKit指南：入门


***

>* 原文链接 : [CloudKit Tutorial: Getting Started](https://www.raywenderlich.com/134694/cloudkit-tutorial-getting-started)
* 原文作者 : [Ed Sasena](https://www.raywenderlich.com/u/anros)
* 译者 : [yrq110](https://github.com/yrq110)

***

> 更新: 此教程已更新至 Xcode 7.3, iOS 9.3, Swift 2.2。原教程由Michael Katz所著。

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

另一个CloudKit的好处就是用户可以放心它们数据的隐私性与安全性，这些数据都是由苹果处理的而不是app开发者。CloudKit将用户数据与开发者隔离开来。

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

## 添加Record Type

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

Select Default Zone under the PUBLIC DATA section in the navigation pane on the left. This zone will contain the public records for your app. Select the Establishment record type from the dropdown list in the center pane if it’s not already selected. Then click the + icon or the New Record button in the right detail pane, as shown in the screenshot below:

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S11-Create-New-Establishment-Record-480x183.png)

This will create a new, empty Establishment record.

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/S12-New-Establishment-Record-Blank-415x320.png)

At this point you’re ready to enter some test data for your app.

The following sample establishment data is fictional. The establishments are located near Apple’s headquarters so they’re easy to find in the simulator.

Enter each record as described below:

> Note: The image for each CoverPhoto element is included in the Supporting Files\Sample Images folder in the Xcode project. To add the image to the establishment record, simply drag it to the CoverPhoto field.

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/T2-Entitlement-Records-Data.png)

Once all three records have been saved, the dashboard should look like this:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S14-All-Three-Records-429x320.png)

For each record, the values entered are the database representation of the data. On the app side, the data types are different. For example, SeatingType and ChangingTable are structs. So the specified Int value for a SeatingType might correspond to a “high chair” or a “booster” seat. For HealthyOption and KidsMenu, the Int values represent Boolean types: a 0 means that establishment doesn’t have that option and a 1 means that it does.

Running the app requires you to have an iCloud account that can be used for development.
[Creating an iCloud Account for Development](https://developer.apple.com/library/tvos/documentation/DataManagement/Conceptual/CloudKitQuickStart/EnablingiCloudandConfiguringCloudKit/EnablingiCloudandConfiguringCloudKit.html#//apple_ref/doc/uid/TP40014987-CH2-SW7)

You will also need to enter into the iOS Simulator the iCloud credentials associated with this account.
[Enter iCloud Credentials Before Running Your App](https://developer.apple.com/library/tvos/documentation/DataManagement/Conceptual/CloudKitQuickStart/CreatingaSchemabySavingRecords/CreatingaSchemabySavingRecords.html#//apple_ref/doc/uid/TP40014987-CH3-SW12)

Return to Xcode. It’s time to start integrating this data into your app!

## Querying Establishment Records

CKQuery objects are used to select records from a database. A CKQuery describes how to find all records of a specified record type that match certain criteria. These criteria can be something like “all records with a Name field that starts with ‘M’”, or “all records that have booster seats”, or “all records within 3km.” These types of expressions are coded in Cocoa with NSPredicate objects. An NSPredicate evaluates objects to see if they match the specified criteria. Predicates are also used in Core Data and are a natural fit for CloudKit, because predicates commonly are defined as a comparison on a field.

CloudKit supports only a subset of available NSPredicate functions. These include mathematical comparisons, some string and set operations (such as “field matches one of the items in a list”), and a special distance function. The function distanceToLocation:fromLocation: was added to NSPredicate for CloudKit to match records with a location field within a specified radius from a known location. This type of predicate is covered in detail below. For other types of queries, the CKQuery Class 
Reference contains a detailed list of the supported functions and descriptions of how to use them.

> Note: CloudKit includes support for CLLocation objects. These are Core Location Framework objects that contain geospatial coordinates. This makes it quite easy to create a query for finding establishments inside of a geographic region – without doing all of the messy coordinate math yourself.

So, in Xcode, open Model/Model.swift. This file contains stubs for all the server calls your app will make.
Replace the fetchEstablishments(\_:radiusInMeters:) method with the following:

```swift
func fetchEstablishments(location:CLLocation, radiusInMeters:CLLocationDistance) {
  // 1
  let radiusInKilometers = radiusInMeters / 1000.0    
  // 2
  let locationPredicate = NSPredicate(format: "distanceToLocation:fromLocation:(%K,%@) &lt; %f", "Location", location, radiusInKilometers)    
  // 3
  let query = CKQuery(recordType: EstablishmentType, predicate: locationPredicate)    
  // 4
  publicDB.performQuery(query, inZoneWithID: nil) { [unowned self] results, error in
    if let error = error {
      dispatch_async(dispatch_get_main_queue()) {
        self.delegate?.errorUpdating(error)
        print("Cloud Query Error - Fetch Establishments: \(error)")
      }
      return
    }
 
    self.items.removeAll(keepCapacity: true)
    results?.forEach({ (record: CKRecord) in
      self.items.append(Establishment(record: record,
        database: self.publicDB))
    })
 
    dispatch_async(dispatch_get_main_queue()) {
      self.delegate?.modelUpdated()
    }
  }
}
```
Taking each numbered comment in turn:
1. CloudKit uses kilometers in its distance predicates. This line simply converts radiusInMeters to kilometers.
2. The predicate filters establishments based on their distance in kilometers from the current location. This statement finds all establishments with a location value within the specified distance from the user’s current location.
3. CKQuery objects are created using a predicate and a record type. Both will be used when performing the query.
4. Finally, performQuery(\_:inZoneWithID:completionHandler:) sends your query up to iCloud, and waits for any matching results. By passing nil as the inZoneWithID parameter, you’re running the query against your default zone; that is, your public database. If you want to retrieve records from both public and private databases, then you have to query each database using a separate call.

Oh. That reminds me. What did CKQuery say to iCloud?

After making a miraculous recovery from such a bad joke, you’re probably wondering where the CKDatabase instance, publicDB, comes from. Take a look the top of Model.swift.

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
Here you define your databases:
1. The default container represents the one you specified in the iCloud capabilities pane.
2. The public database is the one shared amongst all users of your app.
3. The private database contains only the data belonging to the currently logged-in user; in this case, you.

This code will retrieve some local establishments from the public database, but it has to be wired up to a view controller in order to see anything in the app.

### Setting Up the Requisite Callbacks

You can take care of notifications with the familiar delegate pattern. Here’s the protocol from the top of Model.swift that you’ll implement in your view controller:
```swift
protocol ModelDelegate {
  func errorUpdating(error: NSError)
  func modelUpdated()
}
```

Open MasterViewController.swift and replace the modelUpdated() method with the following:

```swift
func modelUpdated() {
  refreshControl?.endRefreshing() 
  tableView.reloadData() 
}
```

This is called when new data is available. All the wiring up of the table view cells to CloudKit objects has already been taken care of in tableView(\_:cellForRowAtIndexPath:). Feel free to take a look.

Next, in MasterViewController.swift, replace the errorUpdating(\_:) method with the following:

```swift
func errorUpdating(error: NSError) {
    let alertController = UIAlertController(title: nil,
                                            message: error.localizedDescription,
                                            preferredStyle: .Alert)
 
    alertController.addAction(UIAlertAction(title: "Dismiss",
                                            style: .Default,
                                            handler: nil))
 
    presentViewController(alertController, animated: true, completion: nil)
}
```
This method is called when the query produces an error. Errors can occur as a result of poor network conditions or CloudKit-specific issues (such as missing or incorrect user credentials or no records being returned from the query).

Good error handling is essential when dealing with any kind of remote server. For now, this code just displays to the user the message returned with the error.

One very common issue, however, is the user not being logged into iCloud and/or not having the iCloud drive turned on for this app. Can you modify errorUpdating(\_:) to at least handle these situations? Hint: Both errors return a CKErrorCode of 1.

Build and run. You should see a list of nearby establishments.

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Build-Run-1-No-Images-179x320.png)

### Troubleshooting Queries

If you’re using the iOS simulator and the list doesn’t populate, then make sure you have the correct location set by selecting from the Xcode menu, Debug\Simulate Location\San Francisco, CA, USA. If you needed to change the location in Xcode, then pull the table down in the app to force a refresh instead of waiting for a location trigger.

If you’re using an iPhone or iPad, location services are enabled, and the list still isn’t populating, then the establishments aren’t close enough to your current location. You have two options: change the coordinates of the sample data to be closer to your current location or use the simulator to run the app. There is a third option, but it’s not terribly practical as you’d have to travel to Cupertino and hang around the Apple campus.

If the data isn’t appearing properly – or isn’t appearing at all – inspect the sample data using the CloudKit dashboard. Make sure all of the records are present, you’ve added them to the default zone and they have the correct values. If you need to re-enter the data, then you can delete records by clicking the trash icon.

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/S16-Delete-Record-1-480x122.png)

Debugging CloudKit errors can be tricky at times. As of this writing, CloudKit error messages don’t contain a tremendous amount of information. To determine the cause of the error you’ll need to look at the error code in conjunction with the particular database operation you’re attempting. Using the numerical error code, look up the matching CKErrorCode enum. The name and description in the documentation will help narrow down the cause of the issue. See below for some examples.

> Note: For a list of error codes that can be returned by CloudKit, read the CloudKit Framework Constants Reference.

Here are some common error enums and related descriptions:
* .BadContainer – The specified container is unknown or unauthorized.
* .NotAuthenticated – The current user is not authenticated and no user record was available. This might happen if the user is not * logged into iCloud.
* .UnknownItem – The specified record does not exist.

When you fetched the list of establishments, you probably noticed that you can see the establishment name and the services the establishments offer. But none of the images are being displayed! Are the clouds in the way?

When you retrieved the establishment records, you automatically retrieved the images as well. You still, however, need to perform the necessary steps to load the images into your app. That’ll chase those clouds away! :]
