#Face Detection in iOS Using Core Image
##使用Core Image的iOS人脸识别

***

>* 原文链接 : [Face Detection in iOS Using Core Image](http://www.appcoda.com/face-detection-core-image/)
* 原文作者 : [GREGG MOJICA](http://www.appcoda.com/author/greggmojica/)
* 译者 : [yrq110](https://github.com/yrq110)

***

Core Image is a powerful API built into Cocoa Touch. It’s a critical piece of the iOS SDK. However, it often gets overlooked. In this tutorial, we’re going to examine Core Image’s face detection features and how to make use of this technology in your own iOS apps!

> Note: This is an intermediate-advanced level iOS tutorial. We assume you have already used technologies such as UIImagePicker, Core Image, etc. If you are unfamiliar with these technologies, then please check out our excellent course series and return here when you are ready.

## What We’re Going to Build

Face detection in iOS has been around since the days of iOS 5 (circa 2011) but it is often overlooked. The facial detection API allows developers to not only detect faces, but also check those faces for particular properties such as if a smile is present or if the person is blinking.

First, we’re going to explore Core Image’s face detection technology by creating a simple app that recognizes a face in a photo and highlights it with a box. In the second example, we’ll take a look at a more detailed use case by creating an app that lets a user take a photo, detect if a face(s) is present, and retrieve the user’s facial coordinates. In so doing, we’re going to learn all about face detection on iOS and how to make use of this powerful, yet often overlooked API.

Let’s roll!

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
