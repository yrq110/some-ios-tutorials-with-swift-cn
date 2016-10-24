#Geofencing Tutorial with Core Location
##Core Location的地理围栏开发指南

***

>* 原文链接 : [Geofencing Tutorial with Core Location](https://www.raywenderlich.com/136165/core-location-geofencing-tutorial)
* 原文作者 : [Andy Pereira](https://www.raywenderlich.com/u/macandyp)
* 译者 : [yrq110](https://github.com/yrq110)

***

>更新: 这篇教程已更新为 Xcode 8 / Swift 3。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/09/Geofencing-feature-250x250.png)

来创建围栏吧!

当设备进入或离开你所设置的区域时地理围栏会提醒app。这可以让你做一些很cool的事：比如设置一个当你离开家时的通知，或者当附近有喜爱的商店时用最新与最棒的商品来迎接用户。在这篇地理围栏的教程中，会学到如何在iOS中使用Swift进行区域监测 - 使用Core Location中的Region Monitoring API。

特别地，会创建一个名为Geotify的基于位置的提醒应用，让用户创建提示器并与真实世界的实际位置相关联。开始吧!

## 入门

下载[开始工程](https://koenig-media.raywenderlich.com/uploads/2016/09/Geotify-Starter-1.zip)。这个工程提供一个简单的功能：允许用户在地图上添加或删除标注点，每个标注点都表示一个位置提醒，或者可以叫它——地理通知(PS:geotification，自造词)。

构建并运行，会看到如下这样一个空的map view。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/GeoInitial-281x500.png)

点击导航栏的+按钮添加一个新的地理通知。app会出现一个独立的视图来设置地理通知的各种属性。

在这篇教程中需要在库比蒂诺的苹果总部添加一个大头针，如何你不知道在哪，打开google地图去找到这个点，记得活用缩放功能提高准确度！

> 注意：在模拟器中使用捏合手势进行缩放，按住option键的同时按下shift移动捏合中心，然后松开shift单击拖动进行捏合操作。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/GeoLooking-Around-281x500.png)

radius表示与指定位置间的距离，超过这个距离会触发iOS的通知，note表示显示在通知中的信息。app也允许用户使用顶部的分段控件来设置通知的触发时机是进入还是离开圆形的地理围栏时。

在radius中输入1000，note中输入Say Hi to Tim!顶部选择Upon Entry，这样，第一次地理通知就设置好了。

点击Add添加，会看到在map view中出现了一个新标记的大头针，就是你刚刚添加的地理通知，标记点附近有一个圆圈表示定义的地理围栏:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/Geo-Say-Hi-281x500.png)

点击大头针会显示地理通知的详细信息，比如提醒的内容与之前指定的事件类型。别点那个小叉号除非你想删掉它!

可以随意添加或删除任何地理通知。由于app使用NSUserDefaults存储持久化数据，因此重启app时会保留之前的地理通知列表。

## 设置LocationManager与许可

至此，你添加在map view上的任何地理通知都仅仅是显示了出来，并没有实际的检测效果。为了实现围栏检测的功能，需要在Core Location上注册每个与地理通知关联的围栏。

在启用围栏检测前，需要设置一个LocationManager实例并且请求到一些许可。

打开GeotificationsViewController.swift在类顶部声明一个CLLocationManager的常量实例，如下:

```swift
class GeotificationsViewController: UIViewController {
 
  @IBOutlet weak var mapView: MKMapView!
 
  var geotifications = [Geotification]()
  let locationManager = CLLocationManager() // Add this statement
 
  ...
}
```

接着使用如下代码替换viewDidLoad()方法:

```swift
override func viewDidLoad() {
  super.viewDidLoad()
  // 1
  locationManager.delegate = self
  // 2
  locationManager.requestAlwaysAuthorization()
  // 3
  loadAllGeotifications()
}
```

来一步步实现这个方法:

1. 将locationManager实例的委托设为当前视图控制器。
2. 调用requestAlwaysAuthorization()方法，提示用户程序请求一个使用地理位置服务的授权。app的地理围栏功能需要Always authorization(前后台都能使用的授权), 这是因为围栏检测在app未在前台运行时也要进行检测。Info.plist中的NSLocationAlwaysUsageDescription键已经设置好了当请求用户位置时所显示的信息。
3. 调用loadAllGeotifications(),反序列化之前在NSUserDefaults中保存的地理通知列表，并将其保存在本地的地理通知数组中。这个方法也会加载map view上标注的地理通知。

当app提示用户授权时，会显示NSLocationAlwaysUsageDescription中的值，友好的说明为何app需要访问用户的地理位置。这个键是请求位置服务授权时所必须要设置的，若没有这个键，则系统会忽略请求并拒绝提供位置服务。

构建并运行，会看到一个用户提示，显示的信息为之前所设置的:

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/GeoLocationWhenNotUsing-281x500.png)

已完成了设置app请求许可的部分，干得好！点击Allow确保location manager在适当的时候能接收到委托方法的回调。

在实现地理围栏之前，还有一个小问题需要解决：当前的用户位置还未显示在map view上！这个特性还未启用，导致导航栏左上的缩放按钮不起作用。

还好这个问题不是什么大事，需要在app获得授权后启用map view显示当前位置的功能。

在GeotificationsViewController.swift文件中，向CLLocationManagerDelegate的扩展中添加如下委托方法，:

```swift
extension GeotificationsViewController: CLLocationManagerDelegate {
  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    mapView.showsUserLocation = (status == .authorizedAlways)
  }
}
```
当授权状态改变时，location manager就会调用locationManager(\_:didChangeAuthorizationStatus:)方法。若用户已经给予了app使用位置服务的许可，则这个方法会在初始化location manager并设置它的委托后被调用。

这个方法放在了合适的地方，用来检测app是否被授权，若被授权，则在map view中显示用户的当前位置。

构建并运行app，若是在真机上运行的，则会看到主map view中会出现一个位置标记。若是在模拟器上运行的，点击菜单中的Debug\Location\Apple来显示位置标记:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/GeoLocationFar-281x500.png)

另外，导航栏上的缩放按钮现在起作用了。:]

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/GeoLocationZoomed-281x500.png)

## 注册地理围栏

在正常设置完location manager后，接下来需要做的工作是在app中注册用户的地理围栏。

在app中，用户的地理围栏信息是使用自定义的地理通知模型存放的。However, Core Location requires each geofence to be represented as a CLCircularRegion instance before it can be registered for monitoring. 为了解决这个问题，需要创建一个帮助方法来将地理通知对象转换为CLCircularRegion。

打开GeotificationsViewController.swift文件，添加如下方法:
```swift
func region(withGeotification geotification: Geotification) -> CLCircularRegion {
  // 1
  let region = CLCircularRegion(center: geotification.coordinate, radius: geotification.radius, identifier: geotification.identifier)
  // 2
  region.notifyOnEntry = (geotification.eventType == .onEntry)
  region.notifyOnExit = !region.notifyOnEntry
  return region
}
```
上面的代码中做了哪些事情:

1. 首先使用地理围栏的位置、半径与一个识别器(为了使iOS区分app中注册的围栏)初始化了一个CLCircularRegion。初始化是很简单的，Geotification模型已经包含了所需要的属性。
2. CLCircularRegion实例有2个布尔型属性：notifyOnEntry和notifyOnExit。这些标识指定了当设备进入或离开指定围栏时是否触发围栏事件。若app设计为对每个围栏仅允许一个通知，根据Geotification对象中保存的枚举值，将其中一个标识设为true另一个设为false即可。

接着，需要一个当用户添加地理通知时启动监测的方法。

在GeotificationsViewController中添加如下方法:
```swift
func startMonitoring(geotification: Geotification) {
  // 1
  if !CLLocationManager.isMonitoringAvailable(for: CLCircularRegion.self) {
    showAlert(withTitle:"Error", message: "Geofencing is not supported on this device!")
    return
  }
  // 2
  if CLLocationManager.authorizationStatus() != .authorizedAlways {
    showAlert(withTitle:"Warning", message: "Your geotification is saved but will only be activated once you grant Geotify permission to access the device location.")
  }
  // 3
  let region = self.region(withGeotification: geotification)
  // 4
  locationManager.startMonitoring(for: region)
}
```
来逐步分析下:

1. isMonitoringAvailableForClass(\_:)表示设备是否含有进行围栏检测所需的硬件。若监测功能不可用则显示一个合适的警告。showSimpleAlertWithTitle(\_:message:viewController)是一个Utilities.swift中的辅助函数，接收标题与消息数据，弹出一个警告视图。
2. 接着，需要检查授权状态，确保app得到使用位置服务所需的许可。若app未被授权，则无法接受到任何围栏相关的通知。
   不过，即使在app未被授权的情况下，也要允许用户保存地理通知，Core Location是允许你注册围栏的。在用户在之后给app授权时，自动启用这些围栏的监测功能。
3. 使用输入的地理通知与之前定义的帮助方法创建一个CLCircularRegion实例。
4. 最后，使用Core Location注册CLCircularRegion实例进行监测。

在启用监测的方法搞定后，也需要一个当用户移除地理通知时停止监测的方法。

在GeotificationsViewController.swift文件中，在startMonitoringGeotificiation(\_:)下面添加如下方法:

```swift
func stopMonitoring(geotification: Geotification) {
  for region in locationManager.monitoredRegions {
    guard let circularRegion = region as? CLCircularRegion, circularRegion.identifier == geotification.identifier else { continue }
    locationManager.stopMonitoring(for: circularRegion)
  }
}
```
这个方法命令locationManager停止监测与输入的地理通知关联的CLCircularRegion。

好了，现在完成启动和停止的方法了，会在添加或移除地理通知时使用这些方法，是时候进行添加通知部分的开发了。

首先，看看GeotificationsViewController.swift文件中的addGeotificationViewController(\_:didAddCoordinate)方法。

这个方法是由AddGeotificationViewController调用的委托方法来创建地理通知，这个方法负责使用来自AddGeotificationsViewController的值创建一个新的Geotification对象，并且更新对应的map view与地理通知列表。接着调用saveAllGeotifications()，更新地理通知列表，并保存到NSUserDefaults。

使用如下代码替换这个方法:
```swift
func addGeotificationViewController(controller: AddGeotificationViewController, didAddCoordinate coordinate: CLLocationCoordinate2D, radius: Double, identifier: String, note: String, eventType: EventType) {
  controller.dismiss(animated: true, completion: nil)
  // 1
  let clampedRadius = min(radius, locationManager.maximumRegionMonitoringDistance)
  let geotification = Geotification(coordinate: coordinate, radius: clampedRadius, identifier: identifier, note: note, eventType: eventType)
  add(geotification: geotification)
  // 2
  startMonitoring(geotification: geotification)
  saveAllGeotifications()
}
```
代码中的两个关键点:

1. 确保半径值与locationManager的maximumRegionMonitoringDistance属性一致，这个属性被定义为地理围栏的最大半径。这很重要，任何超过这个最大值的值都会使监测失败。
2. 调用startMonitoringGeotification(\_:)，使用Core Location注册与新添加的地理通知相关联的监测围栏。

At this point, the app is fully capable of registering new geofences for monitoring. There is, however, a limitation: As geofences are a shared system resource, Core Location restricts the number of registered geofences to a maximum of 20 per app.

While there are workarounds to this limitation (See Where to Go From Here? for a short discussion), for the purposes of this tutorial, you’ll take the approach of limiting the number of geotifications the user can add.

Add a line to updateGeotificationsCount(), as shown in the code below:
```swift
func updateGeotificationsCount() {
  title = "Geotifications (\(geotifications.count))"
  navigationItem.rightBarButtonItem?.isEnabled = (geotifications.count < 20)  // Add this line
}
```
This line disables the Add button in the navigation bar whenever the app reaches the limit.

Finally, let’s deal with the removal of geotifications. This functionality is handled in mapView(_:annotationView:calloutAccessoryControlTapped:), which is invoked whenever the user taps the “delete” accessory control on each annotation.

Add a call to stopMonitoring(geotification:) to mapView(\_:annotationView:calloutAccessoryControlTapped:), as shown below:
```swift
func mapView(_ mapView: MKMapView, annotationView view: MKAnnotationView, calloutAccessoryControlTapped control: UIControl) {
  // Delete geotification
  let geotification = view.annotation as! Geotification
  stopMonitoring(geotification: geotification)   // Add this statement
  removeGeotification(geotification)
  saveAllGeotifications()
}
```
The additional statement stops monitoring the geofence associated with the geotification, before removing it and saving the changes to NSUserDefaults.

At this point, your app is fully capable of monitoring and un-monitoring user geofences. Hurray!

Build and run the project. You won’t see any changes, but the app will now be able to register geofence regions for monitoring. However, it won’t be able to react to any geofence events just yet. Not to worry—that will be your next order of business!

![](https://koenig-media.raywenderlich.com/uploads/2015/03/These_eyes.png)

## Reacting to Geofence Events
You’ll start by implementing some of the delegate methods to facilitate error handling – these are important to add in case anything goes wrong.

In GeotificationsViewController.swift, add the following methods to the CLLocationManagerDelegate extension:
```swift
func locationManager(_ manager: CLLocationManager, monitoringDidFailFor region: CLRegion?, withError error: Error) {
  print("Monitoring failed for region with identifier: \(region!.identifier)")
}
 
func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
  print("Location Manager failed with the following error: \(error)")
}
```
These delegate methods simply log any errors that the location manager encounters to facilitate your debugging.

> Note: You’ll definitely want to handle these errors more robustly in your production apps. For example, instead of failing silently, you could inform the user what went wrong.

Next, open AppDelegate.swift; this is where you’ll add code to properly listen and react to geofence entry and exit events.
Add the following line at the top of the file to import the CoreLocation framework:
```swift
import CoreLocation
```
Ensure that the AppDelegate has a CLLocationManager instance near the top of the class, as shown below:
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
  var window: UIWindow?
 
  let locationManager = CLLocationManager() // Add this statement
  ...
}
```
Replace application(_:didFinishLaunchingWithOptions:) with the following implementation:

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil) -> Bool {
  locationManager.delegate = self
  locationManager.requestAlwaysAuthorization()
  return true
}
```
You’ve set up your AppDelegate to receive geofence-related events. But you might wonder, “Why did I designate the AppDelegate to do this instead of the view controller?”

Geofences registered by an app are monitored at all times, including when the app isn’t running. If the device triggers a geofence event while the app isn’t running, iOS automatically relaunches the app directly into the background. This makes the AppDelegate an ideal entry point to handle the event, as the view controller may not be loaded or ready.

Now you might also wonder, “How will a newly-created CLLocationManager instance be able to know about the monitored geofences?”

It turns out that all geofences registered by your app for monitoring are conveniently accessible by all location managers in your app, so it doesn’t matter where the location managers are initialized. Pretty nifty, right? :]

Now all that’s left is to implement the relevant delegate methods to react to the geofence events. Before you do so, you’ll create a method to handle a geofence event.

Add the following method to AppDelegate.swift:
```swift
func handleEvent(forRegion region: CLRegion!) {
  print("Geofence triggered!")
}
```
At this point, the method takes in a CLRegion and simply logs a statement. Not to worry—you’ll implement the event handling later.

Next, add the following delegate methods in the CLLocationManagerDelegate extension of AppDelegate.swift, as well as a call to the handleRegionEvent(_:) function you just created, as shown in the code below:
```swift
extension AppDelegate: CLLocationManagerDelegate {
 
  func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
    if region is CLCircularRegion {
      handleEvent(forRegion: region)
    }
  }
 
  func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
    if region is CLCircularRegion {
      handleEvent(forRegion: region)
    }
  }
}
```
As the method names aptly suggest, you fire locationManager(_:didEnterRegion:) when the device enters a CLRegion, while you fire locationManager(_:didExitRegion:) when the device exits a CLRegion.

Both methods return the CLRegion in question, which you need to check to ensure it’s a CLCircularRegion, since it could be a 

CLBeaconRegion if your app happens to be monitoring iBeacons, too. If the region is indeed a CLCircularRegion, you accordingly call handleRegionEvent(_:).

> Note: A geofence event is triggered only when iOS detects a boundary crossing. If the user is already within a geofence at the point of registration, iOS won’t generate an event. If you need to query whether the device location falls within or outside a given geofence, Apple provides a method called requestStateForRegion(_:).
```swift
```
