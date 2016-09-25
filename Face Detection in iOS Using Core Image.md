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

## Setting up the Project

Download the starter project linked here and open it up in Xcode. As you can see, it features nothing more than a simple storyboard with a connected IBOutlet and imageView.

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-storyboard-1024x566.jpg)

> Credit: The bundled images are courtesy of unsplash.com.

## Detecting Faces Using Core Image

In the starter project, we have an imageView in the storyboard, and that component has been connected to the code as an IBOutlet. Next, we will implement the code for face detection. Just insert the following code into your swift file, and I’ll explain that later:

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
Let’s talk about what’s going on here:

* Line #3: we create a variable personciImage that extracts the UIImage out of the UIImageView in the storyboard and converts it to a CIImage. CIImage is required when using Core Image
* Line #7: we create an accuracy variable and set to CIDetectorAccuracyHigh. You can pick from CIDetectorAccuracyHigh (which provides high processing power) and CIDetectorAccuracyLow (which uses low processing power). For the purposes of this tutorial and we choose CIDetectorAccuracyHigh because we want high accuracy.
* Line #8: here we define a faceDetector variable and set it to the CIDetector class and pass in the accuracy variable we created above.
* Line #9: by calling the featuresInImage method of faceDetector, the detector finds faces in the given image. At the end, it returns us an array of faces.
* Line #11: here we loop through the array of faces and cast each of the detected face to CIFaceFeature.
* Line #15: We create a UIView called faceBox and set its frame to the frame dimensions returned from faces.first. This is to draw a rectangle to highlight the detected face.
* Line #17: we set the faceBox’s border width to 3.
* Line #18: we set the border color to red.
* Line #19: The background color is set to clear, indicating that this view will not have a visible background.
* Line #20: Finally, we add the view to the personPic imageView.
* Line #22-28: Not only can the API help you detect the face, the detector can detect the face’s left and right eyes. We’ll not highlight the eyes in the image. Here I just want to show you the related properties of CIFaceFeature.

We will invoke the detect method in viewDidLoad. So insert the following line of code in the method:

```swift
detect()
```
Compile and run the app. You will get something like this:

![](http://www.appcoda.com/wp-content/uploads/2016/09/face-detection-2.png)

Base on the console output, it looks like the detector can detect the face:

```swift
Found bounds are (177.0, 415.0, 380.0, 380.0)
```
There are a couple of issues that we haven’t dealt with in the current implementation:

* The face detection is applied on the original image, which has a higher resolution than the image view. And we have set the content mode of the image view to aspect fit. To draw the rectangle properly, we have to calculate the actual position and size of the face in the image view.
* Furthermore, Core Image and UIView (or UIKit) use two different coordinate systems (see the figure below). We have to provide an implementation to translate the Core Image coordinates to UIView coordinates.

![](http://www.appcoda.com/wp-content/uploads/2016/09/core-image-coordinate-1024x690.jpg)

Now replace the detect() method with the following code:

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

