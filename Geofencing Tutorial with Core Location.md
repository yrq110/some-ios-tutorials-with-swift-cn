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

下载[开始工程](https://koenig-media.raywenderlich.com/uploads/2016/09/Geotify-Starter-1.zip)。这个工程提供一个简单的功能：允许用户在地图上添加或删除标注点，每个标注点都表示一个位置提醒，或者可以叫它————地理通知(PS:geotification，作者自造词)。

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

You’ve set up your app to request the required permission. Great! Click or tap Allow to ensure the location manager will receive delegate callbacks at the appropriate times.

Before you proceed to implement the geofencing, there’s a small issue you have to resolve: the user’s current location isn’t showing up on the map view! This feature is disabled, and as a result, the zoom button on the top-left of the navigation bar doesn’t work.

Fortunately, the fix is not difficult — you’ll simply enable the current location only after the app is authorized.

In GeotificationsViewController.swift, add the following delegate method to the CLLocationManagerDelegate extension:

```swift
extension GeotificationsViewController: CLLocationManagerDelegate {
  func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
    mapView.showsUserLocation = (status == .authorizedAlways)
  }
}
```

The location manager calls locationManager(\_:didChangeAuthorizationStatus:) whenever the authorization status changes. If the user has already granted the app permission to use Location Services, this method will be called by the location manager after you’ve initialized the location manager and set its delegate.

That makes this method an ideal place to check if the app is authorized. If it is, you enable the map view to show the user’s current location.

Build and run the app. If you’re running it on a device, you’ll see the location marker appear on the main map view. If you’re running on the simulator, click Debug\Location\Apple in the menu to see the location marker:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/GeoLocationFar-281x500.png)

In addition, the zoom button on the navigation bar now works. :]

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/GeoLocationZoomed-281x500.png)
