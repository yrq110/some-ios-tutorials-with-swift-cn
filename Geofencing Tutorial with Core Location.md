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

## 设置位置管理与许可

At this point, any geotifications you’ve added to the map view are only for visualization. You’ll fix this by taking each geotification and registering its associated geofence with Core Location for monitoring.
Before any geofence monitoring can happen, though, you need to set up a Location Manager instance and request the appropriate permissions.
Open GeotificationsViewController.swift and declare a constant instance of a CLLocationManager near the top of the class, as shown below:

```swift
class GeotificationsViewController: UIViewController {
 
  @IBOutlet weak var mapView: MKMapView!
 
  var geotifications = [Geotification]()
  let locationManager = CLLocationManager() // Add this statement
 
  ...
}
```

Next, replace viewDidLoad() with the following code:

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

Let’s run through this method step by step:

1. You set the view controller as the delegate of the locationManager instance so that the view controller can receive the relevant delegate method calls.
2. You make a call to requestAlwaysAuthorization(), which invokes a prompt to the user requesting for Always authorization to use location services. Apps with geofencing capabilities need Always authorization, due to the need to monitor geofences even when the app isn’t running. Info.plist has already been setup with a message to show the user when requesting the user’s location under the key NSLocationAlwaysUsageDescription.
3. You call loadAllGeotifications(), which deserializes the list of geotifications previously saved to NSUserDefaults and loads them into a local geotifications array. The method also loads the geotifications as annotations on the map view.

When the app prompts the user for authorization, it will show NSLocationAlwaysUsageDescription, a user-friendly explanation of why the app requires access to the user’s location. This key is mandatory when you request authorization for location services. If it’s missing, the system will ignore the request and prevent location services from starting altogether.

Build and run the project, and you’ll see a user prompt with the aforementioned description that’s been set:

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
