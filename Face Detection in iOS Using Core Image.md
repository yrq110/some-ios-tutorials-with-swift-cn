#Face Detection in iOS Using Core Image
##使用Core Image的iOS人脸识别

***

>* 原文链接 : [Face Detection in iOS Using Core Image](http://www.appcoda.com/face-detection-core-image/)
* 原文作者 : [GREGG MOJICA](http://www.appcoda.com/author/greggmojica/)
* 译者 : [yrq110](https://github.com/yrq110)

***

Core Image是Cocoa Touch中一个强大的API，是iOS SDK中的关键部分，不过它经常被忽略。在这篇教程中，会验证Core Image的人脸识别特性并在你的iOS app中应用这项技术!

> 注意: 这是一篇中高级的iOS教程，假定你已经使用过UIImagePicker, Core Image等技术。若你不太熟悉这些技术，可以看看我们的[系列教程](http://www.appcoda.com/ios-programming-course)然后准备好了再回来。

## 将要构建的东西

iOS的人脸识别从iOS 5(2011)就有了不过一直被无视。面部识别API允许开发者不仅可以检测人脸，也可以检测到面部的一些特殊属性，比如说微笑或眨眼。

首先，为了了解Core Image的人脸识别技术我们会创建一个app来识别照片中的人脸并用一个方框来标记它。在第二个例子中，让用户拍摄一张照片，检测其中的人脸并检索用户面部坐标。这样一来，就充分掌握了iOS中的人脸识别，并且学会如何利用这个强大却总被忽略的API。

开搞!

## 建立工程

下载[开始工程](https://github.com/appcoda/FaceDetector/raw/master/FaceDetectorStarter.zip)在Xcode中打开。如你所见，里面只有一个关联了IBOutlet和imageView的故事板。

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-storyboard-1024x566.jpg)

> 感谢: 绑定的图像来自[unsplash.com](http://unsplash.com/)。

## 使用Core Image识别人脸

在开始工程中，故事板中的imageView组件与代码中的IBOutlet已关联，接下来要实现人脸识别的代码部分。在swift文件中插入如下代码，之后进行讲解:

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
编译并运行app，结果应如下所示:

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-2.png)

根据控制台的输出来看，貌似识别器识别到了人脸:

```swift
Found bounds are (177.0, 415.0, 380.0, 380.0)
```
当前的实现中一些没有解决的问题:

* 人脸识别是在原始图像上执行的，原始图像的分辨率比image view要高，因此需要设置image view的content mode为aspect fit(保持纵横比的情况下缩放图片)。为了合适的绘制矩形框，需要计算image view中人脸的实际位置与尺寸
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

The code changes are highlighted in yellow above. First, we use the affine transform to convert the Core Image coordinates to UIKit coordinates. Secondly, we write extra code to compute the actual position and size of the rectangular view.

Now run the app again. You should see a box around the face. Great job! You’ve successfully detected a face using Core Image.

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-result-1024x668.jpg)

## Building a Camera App with Face Detection

Let’s imagine you have a camera/photo app that takes a photo. As soon as the image is taken you want to run face detection to determine if a face is or is not present. If any given face is present, you might want to classify that photo with some tags or so. While we’re not here to build a photo storing app, we will experiment with a live camera app. To do so, we’ll need to integrate with the UIImagePicker class and run our Face Detection code immediately after a photo is taken.

In the starter project, I have already created the CameraViewController class. Update the code like this to implement the camera feature:

```swift

2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
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

The first few lines setup the UIImagePicker delegate. In the didFinishPickingMediaWithInfo method (this is a UIImagePicker delegate method), we set the imageView to the image passed in from the method. We then dismiss the picker and call the detect function.

We haven’t implemented the detect function yet. Insert the following code and let’s take a closer look at it:

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

Our detect() function is very much similar to its previous implementation. This time, however, we’re using it on the captured image. When a face is detected, we display an alert message “We detected a face!” Otherwise, we display a “No Face!” message. Run the app and have a quick test.

![](http://www.appcoda.com/wp-content/uploads/2016/08/faces-1024x862.png)

CIFaceFeature has several properties and methods we have already expolored. For example, if you wanted to detect if the person was smiling, you can call .hasSmile which returns a boolean. Alternatively, you could call .hasLeftEyePosition to check if the left eye is present (let’s hope it is) or .hasRightEyePosition for the right eye, respectively.

We can also call hasMouthPosition to check if a mouth is present. If a mouth is present, we can access those coordinates with the mouthPosition property as seen below:

```swift
if (face.hasMouthPosition) {
     print("mouth detected")
}
```
As you can see, detecting facial features is incredibly simple using Core Image. In addition to detecting a mouth, smile, or eyePosition, we can also check if an eye is open or closed by calling leftEyeClosed for the left eye and rightEyeClosed to check for the right eye.

## Wrapping Up

In this tutorial, we explored Core Image’s Face Detection APIs and how to use this in a camera app. We setup a basic UIImagePicker to snap a photo and detect whether a person present in an image or not.

As you can see, Core Image’s face detection is a powerful API with many applications! I hope you found this tutorial useful and an informative guide to this less known iOS API!

> NOTE: Stay tuned for our tutorial series on neural nets, the technology that powers facial detection!

Feel free to download the final project [here](https://github.com/appcoda/FaceDetector).
