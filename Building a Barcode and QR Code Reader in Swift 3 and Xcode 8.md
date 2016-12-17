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

The demo app that we’re going to build is fairly simple and straightforward. Before we proceed to build the demo app, however, it’s important to understand that any barcode scanning in iOS, including QR code scanning, is totally based on video capturing. That’s why the barcode scanning feature is added in the AVFoundation framework. Keep this point in mind, as it’ll help you understand the entire chapter.

So, how does the demo app work?

Take a look at the screenshot below. This is how the app UI looks. The app works pretty much like a video capturing app but without the recording feature. When the app is launched, it takes advantage of the iPhone’s rear camera to spot the QR code and recognizes it automatically. The decoded information (e.g. an URL) is displayed right at the bottom of the screen.

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-1-1024x630.png)

It’s that simple.

To build the app, you can start by downloading the project template from here. I have pre-built the storyboard and linked up the message label for you. The main screen is associated with the QRCodeViewController class, while the scanner screen is associated with the QRScannerController class.

![](http://www.appcoda.com/wp-content/uploads/2016/11/qrcode-reader-2-1024x565.png)

You can run the starter project to have a look. After launching the app, you can tap the scan button to bring up the scan view. Later we will implement this view controller for QR code scanning.

Now that you understand how the starter project works, let’s get started and develop the QR scanning feature in the app.
