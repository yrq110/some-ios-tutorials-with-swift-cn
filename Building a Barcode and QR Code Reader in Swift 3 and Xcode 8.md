# Building a Barcode and QR Code Reader in Swift 3 and Xcode 8
## Xcode 8中使用Swift 3构建条形码与QR码识别器

***

>* 原文链接 : [Building a Barcode and QR Code Reader in Swift 3 and Xcode 8](http://www.appcoda.com/barcode-reader-swift/)
* 原文作者 : [SIMON NG](http://www.appcoda.com/author/admin/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-0-1680x1120.jpg)

QR码是什么? 我相信大部分人应该都知道QR码，如果没听说过可以看看上面那张图 - 那就是QR码。

QR(Quick Response的缩写)码是由Denso开发的一种二维条形码，最初是被设计用来跟踪制造过程的，近些年经常被用来对登录页面或商品信息的URL地址编码。跟普通的条形码不同的是，QR码包含水平与竖直两个方向的信息。在这里我不想讨论QR码的技术细节，若感兴趣可以看看QR码的[官网](http://www.qrcode.com/)。

随着iPhone与Android智能手机的流行，QR码的使用越来越广泛。在一些国家QR码随处可见，杂志上，报纸上，广告上，名片上，甚至是菜单上。作为一个iOS开发者，也许很想知道如何给你的app添加读取QR码的功能。在iOS 7之前需要依赖第三方库来实现识别功能，现在可以直接使用内置的AVFoundation框架进行实时识别条形码。

不过，做一个能够扫描与解析QR码的app一直不是个轻松的差事。

> 小提示: 可以在 http://www.qrcode-monkey.com 生成你自己的QR码。

## 构建QR码识别器

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

接着添加下列代码，将captureMetadataOutput对象的委托设为自身(self)，这样就使QRReaderViewController实现了AVCaptureMetadataOutputObjectsDelegate协议。
```swift
// Set delegate and use the default dispatch queue to execute the call back
captureMetadataOutput.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
captureMetadataOutput.metadataObjectTypes = [AVMetadataObjectTypeQRCode]
```
当捕捉到原数据对象时会将其传给委托对象进行下一步的处理。在上面的代码中，指定了执行委托方法的调度队列(dispatch queue)，一个调度队列是串行或并行的。根据苹果的文档，这个队列必须是一个串行队列(serial queue)，因此需要使用DispatchQueue.main取得默认的串行队列。

metadataObjectTypes属性同样很重要，通过这个属性告诉app需要的是哪种格式的元数据，AVMetadataObjectTypeQRCode指明了我们的目标 —— QR码。

现在来设置AVCaptureMetadataOutput对象。使用AVCaptureVideoPreviewLayer在屏幕上显示设备摄像头捕捉到的视频，实际上是一个CALayer。结合预览层(preview layer)与AV capture session来显示视频，预览层作为当前视图的一个子层添加。将如下代码插入到do-catch代码块中:

```swift
// Initialize the video preview layer and add it as a sublayer to the viewPreview view's layer.
videoPreviewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
videoPreviewLayer?.videoGravity = AVLayerVideoGravityResizeAspectFill
videoPreviewLayer?.frame = view.layer.bounds
view.layer.addSublayer(videoPreviewLayer!)
```
最后，调用capture session的startRunning方法启动视频捕捉:
```swift
// Start video capture.
captureSession?.startRunning()
```
若在iOS真机上编译运行app会崩溃并出现以下错误:
```
This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSCameraUsageDescription key with a string value explaining to the user how the app uses this data.
```
iOS要求app开发者在访问摄像头之前需要取得用户的许可。解决方法是在Info.plist中添加一个NSCameraUsageDescription的key，打开文件后在空白区域点击右键选择添加新的一行，将key设为`Privacy – Camera Usage Description`，value设为`We need to access your camera for scanning QR code`。

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-3-1024x211.png)

完成编辑后，再次部署app并在真机上运行。点击scan按钮启动摄像头开始捕捉视频，会发现信息标签和顶部工具栏被隐藏了，可以添加如下代码来解决这个问题，它会将这两个组件移动到视频层的顶部。

```swift
// Move the message label and top bar to the front
view.bringSubview(toFront: messageLabel)
view.bringSubview(toFront: topbar)
```
修改完成后再次重启，屏幕中的信息标签会显示*No QR code is detected*。

## 实现读取QR码

现在这个app看起来像那么回事儿了，那么如何让它能在扫描QR码之后还能解析出其中的内容？其实app已经具有检测QR码这个功能了，只是还没有挖掘出来。如下修改一下app:

* 高亮检测到的QR码。
* 将解码后的信息显示在屏幕底端。

### 初始化高亮视图

为了高亮显示QR码需要创建一个绿色边界的UIView对象。在viewDidLoad方法的do代码块中添加如下代码:

```swift
// Initialize QR Code Frame to highlight the QR code
qrCodeFrameView = UIView()
 
if let qrCodeFrameView = qrCodeFrameView {
    qrCodeFrameView.layer.borderColor = UIColor.green.cgColor
    qrCodeFrameView.layer.borderWidth = 2
    view.addSubview(qrCodeFrameView)
    view.bringSubview(toFront: qrCodeFrameView)
}
```
qrCodeFrameView在屏幕中是不可见的，因为默认情况下它的尺寸是0，当检测到QR码时会设置它的尺寸并将其设为一个绿色边框的视图。

### 解析QR码

当AVCaptureMetadataOutput对象识别出QR码时，会调用AVCaptureMetadataOutputObjectsDelegate中的委托方法:

```swift
optional func captureOutput(_ captureOutput: AVCaptureOutput!, didOutputMetadataObjects metadataObjects: [Any]!, from connection: AVCaptureConnection!)
```

由于至此还未实现这个方法，因此无法解析QR码。为了捕捉并解析QR码中的信息，需要实现这个方法对元数据对象执行额外的操作。代码如下:

```swift
func captureOutput(_ captureOutput: AVCaptureOutput!, didOutputMetadataObjects metadataObjects: [Any]!, from connection: AVCaptureConnection!) {
    
    // Check if the metadataObjects array is not nil and it contains at least one object.
    if metadataObjects == nil || metadataObjects.count == 0 {
        qrCodeFrameView?.frame = CGRect.zero
        messageLabel.text = "No QR code is detected"
        return
    }
    
    // Get the metadata object.
    let metadataObj = metadataObjects[0] as! AVMetadataMachineReadableCodeObject
    
    if metadataObj.type == AVMetadataObjectTypeQRCode {
        // If the found metadata is equal to the QR code metadata then update the status label's text and set the bounds
        let barCodeObject = videoPreviewLayer?.transformedMetadataObject(for: metadataObj)
        qrCodeFrameView?.frame = barCodeObject!.bounds
        
        if metadataObj.stringValue != nil {
            messageLabel.text = metadataObj.stringValue
        }
    }
}
```
方法的第二个参数(metadataObjects)是一个包含所有元数据对象的数组。首先要做的是确认这个数组是否非空，至少需要包含一个对象，否则需要将qrCodeFrameView的尺寸设为0并重置messageLabel所显示的信息。

若存在元数据对象，还需要验证是否是QR码，若是则需要找到QR码的边界，上面那些代码用来设置高亮QR码的视图。通过调用viewPreviewLayer的transformedMetadataObject(for:)方法获取元数据对象对应的视图层坐标，这样就可以确定所要添加的绿色边框视图的尺寸与位置了。

最后将QR码解析成人类可读的信息，这一步很简单，可以使用AVMetadataMachineReadableCode对象的stringValue属性访问解码出的信息。

准备好了！按下Run键在真机上编译并运行app。

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-4-1024x593.png)

启动后点击scan按钮将设备对准QR码，app会立刻检测到QR码并解码出其中的信息。

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-5-1024x637.jpg)


## 练习 – 条形码识别器

demo app现在可以识别出QR码，如果你能将它变成一个通用的条形码识别器是不是很棒？除了QR码，AVFoundation框架还支持如下类型的条形码:

* UPC-E (AVMetadataObjectTypeUPCECode)
* Code 39 (AVMetadataObjectTypeCode39Code)
* Code 39 mod 43 (AVMetadataObjectTypeCode39Mod43Code)
* Code 93 (AVMetadataObjectTypeCode93Code)
* Code 128 (AVMetadataObjectTypeCode128Code)
* EAN-8 (AVMetadataObjectTypeEAN8Code)
* EAN-13 (AVMetadataObjectTypeEAN13Code)
* Aztec (AVMetadataObjectTypeAztecCode)
* PDF417 (AVMetadataObjectTypePDF417Code)

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-6-1024x626.png)

你的任务就是修改Xcode项目使其能够扫描其他类型的条形码，需要设置captureMetadataOutput的metadataObjectTypes属性使其可以识别多种条形码。

```swift
captureMetadataOutput.metadataObjectTypes = [AVMetadataObjectTypeQRCode]
```

可以在[Gihub](https://github.com/appcoda/QRCodeReader)下载完整代码。
If you’ve given it your best shot and are still stumped, you can download the solution on GitHub.
