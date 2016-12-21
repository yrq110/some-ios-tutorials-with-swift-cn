# Building a Barcode and QR Code Reader in Swift 3 and Xcode 8
## Xcode 8中使用Swift 3构建条形码与QR码扫描器

***

>* 原文链接 : [Building a Barcode and QR Code Reader in Swift 3 and Xcode 8](http://www.appcoda.com/barcode-reader-swift/)
* 原文作者 : [SIMON NG](http://www.appcoda.com/author/admin/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-0-1680x1120.jpg)

QR码是什么? 我相信大部分人应该都知道QR码，如果没听说过可以看看上面那张图 - 那就是QR码。

QR(Quick Response的缩写)码是由Denso开发的一种二维条形码，最初是被设计用来跟踪制造过程的，近些年经常被用来对登录页面或商品信息的URL地址编码。跟普通的条形码不同的是，QR码包含水平与竖直两个方向的信息。在这里我不想讨论QR码的技术细节，若感兴趣可以看看QR码的[官网](http://www.qrcode.com/)。

随着iPhone与Android智能手机的流行，QR码的使用越来越广泛。在一些国家QR码随处可见，杂志上，报纸上，广告上，名片上，甚至是菜单上。作为一个iOS开发者，也许很想知道如何给你的app添加读取QR码的功能。在iOS 7之前需要依赖第三方库来实现扫描功能，现在可以直接使用内置的AVFoundation框架进行实时的扫描条形码。

不过，做一个能够扫描与解析QR码的app一直不是个轻松的差事。

> 小提示: 可以在 http://www.qrcode-monkey.com 生成你自己的QR码。

## 构建QR码扫描器

我们要做的demo是比较简单直接的，在开始构建之前需要了解一下iOS中的条形码与QR码扫描，它们都是基于视频捕捉的，这就是为何需要引入AVFoundation框架。记住这一点有助于对这一章节内容的理解。

那么，demo是如何工作的?

看看下面的截图，这就是app界面的样子。app的功能有点像一个没有录像功能的视频捕捉app。当app启动后会利用iPhone的前置摄像头标示出QR码并自动识别，解码后的信息会显示在屏幕底部。

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-1-1024x630.png)

就是这么简单。

构建这个app前需要先下载[项目模板](https://github.com/appcoda/QRCodeReader/raw/master/QRCodeReaderStarter.zip)，预先设置好了storyboard中的标签，主界面对应QRCodeViewController类，扫描界面对应QRScannerController类。

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-2-1024x565.png)

可以先运行一下试试，点击scan按钮会出现扫描视图，下面我们来实现这个视图中的QR码扫描功能。

## 导入AVFoundation框架

已经在项目模板中写好了app的UI界面，界面中的标签用来显示QR码解码后的信息，与QRScannerController类中的messageLabel属性关联。

之前强调过QR码的扫描功能是基于AVFoundation框架来实现的。首先打开QRScannerController.swift文件并导入AVFoundation框架:
```swift
import AVFoundation
```
接着需要实现AVCaptureMetadataOutputObjectsDelegate协议中的委托方法，这个后面再说，如下添加协议:
```swift
class ViewController: UIViewController, AVCaptureMetadataOutputObjectsDelegate
```
在进行下一步之前在QRScannerController类中声明以下变量:
```swift
var captureSession:AVCaptureSession?
var videoPreviewLayer:AVCaptureVideoPreviewLayer?
var qrCodeFrameView:UIView?
```
## 实现视频捕捉

QR码的获取是使用视频捕捉来实现的，为了进行实时的视频捕捉需要使用AVCaptureSession对象，设置其输入为用于视频捕捉的AVCaptureDevice。在QRScannerController类的viewDidLoad方法中插入如下代码:

```swift
// Get an instance of the AVCaptureDevice class to initialize a device object and provide the video as the media type parameter.
let captureDevice = AVCaptureDevice.defaultDevice(withMediaType: AVMediaTypeVideo)
 
do {
    // Get an instance of the AVCaptureDeviceInput class using the previous device object.
    let input = try AVCaptureDeviceInput(device: captureDevice)
    
    // Initialize the captureSession object.
    captureSession = AVCaptureSession()
    
    // Set the input device on the capture session.
    captureSession?.addInput(input)
    
} catch {
    // If any error occurs, simply print it out and don't continue any more.
    print(error)
    return
}
```
AVCaptureDevice对象表示一个物理捕捉设备。为了获取视频数据，需要调用defaultDevice(withMediaType:)方法，输入AVMediaTypeVideo得到视频捕捉设备。 

为了进行实时的视频捕捉需要初始化一个AVCaptureSession对象并添加视频捕捉设备作为输入，AVCaptureSession对象用来协调从视频输入到输出的数据流。

在这里，session的输出被设为一个AVCaptureMetaDataOutput对象。AVCaptureMetaDataOutput类是QR码识别中的核心部分，这个类结合AVCaptureMetadataOutputObjectsDelegate协议用来截获输入设备中的所有元数据(由设备摄像头所捕捉的QR码)，并将其转换成便于人们可读的格式。

没有理解的话别担心，一会儿就云开雾散了。在viewDidLoad方法中的do代码块中添加如下代码:

```swift
// Initialize a AVCaptureMetadataOutput object and set it as the output device to the capture session.
let captureMetadataOutput = AVCaptureMetadataOutput()
captureSession?.addOutput(captureMetadataOutput)
```

Next, proceed to add the lines of code shown below. We set self as the delegate of the captureMetadataOutput object. This is the reason why the QRReaderViewController class adopts the AVCaptureMetadataOutputObjectsDelegate protocol.
```swift
// Set delegate and use the default dispatch queue to execute the call back
captureMetadataOutput.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
captureMetadataOutput.metadataObjectTypes = [AVMetadataObjectTypeQRCode]
```
When new metadata objects are captured, they are forwarded to the delegate object for further processing. In the above code, we specify the dispatch queue on which to execute the delegate’s methods. A dispatch queue can be either serial or concurrent. According to Apple’s documentation, the queue must be a serial queue. So we use DispatchQueue.main to get the default serial queue.

The metadataObjectTypes property is also quite important; as this is the point where we tell the app what kind of metadata we are interested in. The AVMetadataObjectTypeQRCode clearly indicates our purpose. We want to do QR code scanning.

Now that we have set and configured an AVCaptureMetadataOutput object, we need to display the video captured by the device’s camera on screen. This can be done using an AVCaptureVideoPreviewLayer, which actually is a CALayer. You use this preview layer in conjunction with an AV capture session to display video. The preview layer is added as a sublayer of the current view. Insert the code below in the do-catch block:

```swift
// Initialize the video preview layer and add it as a sublayer to the viewPreview view's layer.
videoPreviewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
videoPreviewLayer?.videoGravity = AVLayerVideoGravityResizeAspectFill
videoPreviewLayer?.frame = view.layer.bounds
view.layer.addSublayer(videoPreviewLayer!)
```
Finally, we start the video capture by calling the startRunning method of the capture session:
```swift
// Start video capture.
captureSession?.startRunning()
```
If you compile and run the app on a real iOS device, it crashes unexpectedly with the following error:
```
This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSCameraUsageDescription key with a string value explaining to the user how the app uses this data.
```
Similar to what we have done in the audio recording chapter, iOS requires app developers to obtain the user’s permission before allowing to access the camera. To do so, you have to add a key named NSCameraUsageDescription in the Info.plist file. Open the file and right-click any blank area to add a new row. Set the key to Privacy – Camera Usage Description, and value to We need to access your camera for scanning QR code.
![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-3-1024x211.png)
Once you finish the editing, deploy the app and run it on a real device again. Tapping the scan button should bring up the built-in camera and start capturing video. However, at this point the message label and the top bar are hidden. You can fix it by adding the following line of code. This will move the message label and top bar to appear on top of the video layer.
```swift
// Move the message label and top bar to the front
view.bringSubview(toFront: messageLabel)
view.bringSubview(toFront: topbar)
```
Re-run the app after making the changes. The message label No QR code is detected should now appear on screen.
```swift
```
