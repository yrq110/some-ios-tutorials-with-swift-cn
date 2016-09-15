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

## Getting Started

Download the starter project. The project provides a simple user interface for adding/removing annotation items to/from a map view. Each annotation item represents a reminder with a location, or as I like to call it, a geotification. :]
Build and run the project, and you’ll see an empty map view.

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/GeoInitial-281x500.png)

Tap on the + button on the navigation bar to add a new geotification. The app will present a separate view, allowing you to set up various properties for your geotification.

For this tutorial, you will add a pin on Apple’s headquarters in Cupertino. If you don’t know where it is, open this google map in a separate tab and use it to hunt the right spot. Be sure to zoom in to make the pin nice and accurate!

> To pinch to zoom on the simulator, hold down option, then hold shift temporarily to move the pinch center, then release shift and click-drag to pinch.

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/GeoLooking-Around-281x500.png)

The radius represents the distance in meters from the specified location, at which iOS will trigger the notification. The note can be any message you wish to display during the notification. The app also lets the user specify whether it should trigger the reminder upon either entry or exit of the defined circular geofence, via the segmented control at the top.

Enter 1000 for the radius value and Say Hi to Tim! for the note, and leave it as Upon Entry for your first geotification.

Click Add once you’re satisfied with all the values. You’ll see your geotification appear as a new annotation pin on the map view, with a circle around it denoting the defined geofence:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/Geo-Say-Hi-281x500.png)

Tap on the pin and you’ll reveal the geotification’s details, such as the reminder note and the event type you specified earlier. Don’t tap on the little cross unless you want to delete the geotification!

Feel free to add or remove as many geotifications as you want. As the app uses NSUserDefaults as a persistence store, the list of geotifications will persist between relaunches.

## Setting Up a Location Manager and Permissions

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
