# Building a Barcode and QR Code Reader in Swift 3 and Xcode 8
## Xcode 8中使用Swift 3构建条形码与二维码扫描器

***

>* 原文链接 : [Building a Barcode and QR Code Reader in Swift 3 and Xcode 8](http://www.appcoda.com/barcode-reader-swift/)
* 原文作者 : [SIMON NG](http://www.appcoda.com/author/admin/)
* 译者 : [yrq110](https://github.com/yrq110/)

***

So, what’s QR code? I believe most of you know what a QR code is. In case you haven’t heard of it, just take a look at the above image – that’s a QR code.

QR (short for Quick Response) code is a kind of two-dimensional bar code developed by Denso. Originally designed for tracking parts in manufacturing, QR code has gained popularity in consumer space in recent years as a way to encode the URL of a landing page or marketing information. Unlike the basic barcode that you’re familiar with, a QR code contains information in both the horizontal and vertical direction. Thus this contributes to its capability of storing a larger amount of data in both numeric and letter form. I don’t want to go into the technical details of the QR code here. If you’re interested in learning more, you can check out the official website of QR code.

> Editor’s note: This is the sample chapter of our Intermediate iOS Programming with Swift book.

With the rising prevalence of iPhone and Android phones, the use of QR codes has been increased dramatically. In some countries QR codes can be found nearly everywhere. They appear in magazines, newspapers, advertisements, billboards, name cards and even food menu. As an iOS developer, you may wonder how you can empower your app to read a QR code. Prior to iOS 7, you had to rely on third party libraries to implement the scanning feature. Now, you can use the built-in AVFoundation framework to discover and read barcodes in real-time.

Creating an app for scanning and translating QR codes has never been so easy.

> Quick tip: You can generate your own QR code. Simply go to http://www.qrcode-monkey.com
