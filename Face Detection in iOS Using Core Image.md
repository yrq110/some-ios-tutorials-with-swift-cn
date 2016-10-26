#Face Detection in iOS Using Core Image
##使用Core Image的iOS人脸识别

***

>* 原文链接 : [Face Detection in iOS Using Core Image](http://www.appcoda.com/face-detection-core-image/)
* 原文作者 : [GREGG MOJICA](http://www.appcoda.com/author/greggmojica/)
* 译者 : [yrq110](https://github.com/yrq110)

***

Core Image是Cocoa Touch中一个强大的API，也是iOS SDK中的关键部分，然而它经常被人们忽视。在这篇教程中会讲解一些Core Image的人脸识别特性，并在你的iOS app中应用这项技术!

> 注意: 这是一篇中高级的iOS教程，假定你已经使用过UIImagePicker, Core Image等技术。若你不太熟悉这些技术，可以先看看我们的[系列教程](http://www.appcoda.com/ios-programming-course)，准备好了再回来。

## 将要构建的应用

iOS的人脸识别从iOS 5(2011)就有了，不过一直没怎么被关注过。人脸识别API允许开发者不仅可以检测人脸，也可以检测到面部的一些特殊属性，比如说微笑或眨眼。

首先，为了了解Core Image的人脸识别技术我们会创建一个app来识别照片中的人脸并用一个方框来标记它。在第二个例子中，让用户拍摄一张照片，检测其中的人脸并检索人脸位置。这样一来，就充分掌握了iOS中的人脸识别，并且学会如何利用这个强大却总被忽略的API。

开搞!

## 建立工程

下载[开始工程](https://github.com/appcoda/FaceDetector/raw/master/FaceDetectorStarter.zip)在Xcode中打开。如你所见，里面只有一个关联了IBOutlet和imageView的故事板。

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-storyboard-1024x566.jpg)

> 感谢: 绑定的图像来自[unsplash.com](http://unsplash.com/)。

## 使用Core Image识别人脸

在开始工程中，故事板中的imageView组件与代码中的IBOutlet已关联，接下来要编写实现人脸识别的代码部分。在swift文件中插入如下代码，之后进行讲解:

```swift
func detect() {
    
    guard let personciImage = CIImage(image: personPic.image!) else {
        return
    }
 
    let accuracy = [CIDetectorAccuracy: CIDetectorAccuracyHigh]
    let faceDetector = CIDetector(ofType: CIDetectorTypeFace, context: nil, options: accuracy)
    let faces = faceDetector.featuresInImage(personciImage)
    
    for face in faces as! [CIFaceFeature] {
        
        print("Found bounds are \(face.bounds)")
        
        let faceBox = UIView(frame: face.bounds)
 
        faceBox.layer.borderWidth = 3
        faceBox.layer.borderColor = UIColor.redColor().CGColor
        faceBox.backgroundColor = UIColor.clearColor()
        personPic.addSubview(faceBox)
        
        if face.hasLeftEyePosition {
            print("Left eye bounds are \(face.leftEyePosition)")
        }
        
        if face.hasRightEyePosition {
            print("Right eye bounds are \(face.rightEyePosition)")
        }
    }
}
```
来探讨下发生了什么:

* 行 #3: 创建personciImage变量保存从故事板中的UIImageView提取图像并将其转换为CIImage，使用Core Image时需要用CIImage。
* 行 #7: 创建accuracy变量并设为CIDetectorAccuracyHigh，可以在CIDetectorAccuracyHigh(较强的处理能力)与CIDetectorAccuracyLow(较弱的处理能力)中选择，因为想让准确度高一些在这里选择CIDetectorAccuracyHigh。
* 行 #8: 这里定义了一个属于CIDetector类的faceDetector变量，并输入之前创建的accuracy变量。
* 行 #9: 调用faceDetector的featuresInImage方法，识别器会找到所给图像中的人脸，最后返回一个人脸数组。
* 行 #11: 循环faces数组里的所有face，并将识别到的人脸强转为CIFaceFeature类型。
* 行 #15: 创建名为faceBox的UIView，frame设为返回的faces.first的frame，绘制一个矩形框来标识识别到的人脸。
* 行 #17: 设置faceBox的边框宽度为3。
* 行 #18: 设置边框颜色为红色。
* 行 #19: 将背景色设为clear，意味着这个视图没有可见的背景。
* 行 #20: 最后，把这个视图添加到personPic imageView上。
* 行 #22-28: API不仅可以帮助你识别人脸，也可识别脸上的左右眼，我们不在图像中标识出眼睛，只是给你展示一下CIFaceFeature的相关属性。

需要在viewDidLoad中调用detect方法，插入如下行:

```swift
detect()
```
编译并运行app，结果应如下图所示:

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-2.png)

根据控制台的输出来看，貌似识别器识别到了人脸:

```swift
Found bounds are (177.0, 415.0, 380.0, 380.0)
```
当前的实现中没有解决的问题:

* 人脸识别是在原始图像上进行的，由于原始图像的分辨率比image view要高，因此需要设置image view的content mode为aspect fit(保持纵横比的情况下缩放图片)。为了合适的绘制矩形框，需要计算image view中人脸的实际位置与尺寸
* 还要注意的是，Core Image与UIView使用两种不同的坐标系统(看下表)，因此要实现一个CoreImage坐标到UIView坐标的转换。

![](http://www.appcoda.com/wp-content/uploads/2016/09/core-image-coordinate-1024x690.jpg)

现在使用下面的代码替换detect()方法:

```swift
func detect() {
    
    guard let personciImage = CIImage(image: personPic.image!) else {
        return
    }
 
    let accuracy = [CIDetectorAccuracy: CIDetectorAccuracyHigh]
    let faceDetector = CIDetector(ofType: CIDetectorTypeFace, context: nil, options: accuracy)
    let faces = faceDetector.featuresInImage(personciImage)
 
    // For converting the Core Image Coordinates to UIView Coordinates
    let ciImageSize = personciImage.extent.size
    var transform = CGAffineTransformMakeScale(1, -1)
    transform = CGAffineTransformTranslate(transform, 0, -ciImageSize.height)
    
    for face in faces as! [CIFaceFeature] {
        
        print("Found bounds are \(face.bounds)")
        
        // Apply the transform to convert the coordinates
        var faceViewBounds = CGRectApplyAffineTransform(face.bounds, transform)
        
        // Calculate the actual position and size of the rectangle in the image view
        let viewSize = personPic.bounds.size
        let scale = min(viewSize.width / ciImageSize.width,
                        viewSize.height / ciImageSize.height)
        let offsetX = (viewSize.width - ciImageSize.width * scale) / 2
        let offsetY = (viewSize.height - ciImageSize.height * scale) / 2
        
        faceViewBounds = CGRectApplyAffineTransform(faceViewBounds, CGAffineTransformMakeScale(scale, scale))
        faceViewBounds.origin.x += offsetX
        faceViewBounds.origin.y += offsetY
        
        let faceBox = UIView(frame: faceViewBounds)
 
        faceBox.layer.borderWidth = 3
        faceBox.layer.borderColor = UIColor.redColor().CGColor
        faceBox.backgroundColor = UIColor.clearColor()
        personPic.addSubview(faceBox)
        
        if face.hasLeftEyePosition {
            print("Left eye bounds are \(face.leftEyePosition)")
        }
        
        if face.hasRightEyePosition {
            print("Right eye bounds are \(face.rightEyePosition)")
        }
    }
}
```

上述代码中，首先使用仿射变换将Core Image坐标转换为UIKit坐标，然后编写了计算实际位置与矩形视图尺寸的代码。

再次运行app，应该会看到人的面部周围会有一个框，干得漂亮！你已经成功使用Core Image识别出了人脸。

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-result-1024x668.jpg)

## 构建一个人脸识别的相机应用

想象一下你有一个用来照相的相机app，照完相后你想运行一下人脸识别来检测一下是否存在人脸。若存在一些人脸，你也许想用一些标签来对这些照片进行分类。我们不会构建一个保存照片后再处理的app，而是一个实时的相机app，因此需要整合一下UIImagePicker类，在照完相时立刻进行人脸识别。

在开始工程中已经创建好了CameraViewController类，使用如下代码实现相机的功能:

```swift
class CameraViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    @IBOutlet var imageView: UIImageView!
    let imagePicker = UIImagePickerController()
 
    override func viewDidLoad() {
        super.viewDidLoad()
        
        imagePicker.delegate = self
    }
    
    @IBAction func takePhoto(sender: AnyObject) {
        
        if !UIImagePickerController.isSourceTypeAvailable(.Camera) {
            return
        }
        
        imagePicker.allowsEditing = false
        imagePicker.sourceType = .Camera
        
        presentViewController(imagePicker, animated: true, completion: nil)
    }
 
    func imagePickerController(picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [String : AnyObject]) {
        if let pickedImage = info[UIImagePickerControllerOriginalImage] as? UIImage {
            imageView.contentMode = .ScaleAspectFit
            imageView.image = pickedImage
        }
        
        dismissViewControllerAnimated(true, completion: nil)
        self.detect()
    }
    
    func imagePickerControllerDidCancel(picker: UIImagePickerController) {
        dismissViewControllerAnimated(true, completion: nil)
    }
    
}
```

前面几行设置UIImagePicker委托为当前视图类，在didFinishPickingMediaWithInfo方法(UIImagePicker的委托方法)中设置imageView为在方法中所选择的图像，接着返回上一视图调用detect函数。

还没有实现detect函数，插入下面代码并分析一下:

```swift
func detect() {
        let imageOptions =  NSDictionary(object: NSNumber(int: 5) as NSNumber, forKey: CIDetectorImageOrientation as NSString)
        let personciImage = CIImage(CGImage: imageView.image!.CGImage!)
        let accuracy = [CIDetectorAccuracy: CIDetectorAccuracyHigh]
        let faceDetector = CIDetector(ofType: CIDetectorTypeFace, context: nil, options: accuracy)
        let faces = faceDetector.featuresInImage(personciImage, options: imageOptions as? [String : AnyObject])
        
        if let face = faces.first as? CIFaceFeature {
            print("found bounds are \(face.bounds)")
            
            let alert = UIAlertController(title: "Say Cheese!", message: "We detected a face!", preferredStyle: UIAlertControllerStyle.Alert)
            alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
            self.presentViewController(alert, animated: true, completion: nil)
            
            if face.hasSmile {
                print("face is smiling");
            }
            
            if face.hasLeftEyePosition {
                print("Left eye bounds are \(face.leftEyePosition)")
            }
            
            if face.hasRightEyePosition {
                print("Right eye bounds are \(face.rightEyePosition)")
            }
        } else {
            let alert = UIAlertController(title: "No Face!", message: "No face was detected", preferredStyle: UIAlertControllerStyle.Alert)
            alert.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: nil))
            self.presentViewController(alert, animated: true, completion: nil)
        }
    }
```

这个detect()函数与之前实现的detect函数非常像，不过这次只用它来获取图像不做变换。当识别到人脸后显示一个警告信息“检测到了人脸！”，否则显示“未检测到人脸”。运行app测试一下

![](http://www.appcoda.com/wp-content/uploads/2016/08/faces-1024x862.png)

我们已经使用到了一些CIFaceFeature的属性与方法，比如，若想检测人物是否微笑，可以调用.hasSmile，它会返回一个布尔值。可以分别使用.hasLeftEyePosition与.hasRightEyePosition检测是否存在左右眼。

同样，可以调用hasMouthPosition来检测是否存在嘴，若存在则可以使用mouthPosition属性，如下所示:

```swift
if (face.hasMouthPosition) {
     print("mouth detected")
}
```

如你所见，使用Core Image来检测面部特征是非常简单的。除了检测嘴、笑容、眼睛外，也可以调用leftEyeClosed与rightEyeClosed检测左右眼是否睁开。

## 总结

在这篇教程中尝试了Core Image的人脸识别API与如何在一个相机app中应用它，构建了一个简单的UIImagePicker来选取照片并检测图像中是否存在人物。

如你所见，Core Image的人脸识别是个强大的API！希望这篇教程能给你提供一些关于这个鲜为人知的iOS API有用的信息。

在[这里](https://github.com/appcoda/FaceDetector)下载最终工程。
