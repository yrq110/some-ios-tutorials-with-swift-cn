#Building a Speech-to-Text App Using Speech Framework in iOS 10
##使用iOS 10的Speech框架构建一个语音转文本app

***

>* 原文链接 : [Building a Speech-to-Text App Using Speech Framework in iOS 10](http://www.appcoda.com/siri-speech-framework/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : [yrq110](https://github.com/yrq110)

***


在WWDC 2016上苹果介绍了Speech框架-一个用于语音识别的API，其实Speech Kit就是Siri用于语音识别的框架。目前有少数可用的语音识别框架，不过那些要么太贵要么就是不太好。在这篇教程中，将会给你展示如何使用Speech Kit构建一个像Siri那样的语音转文本app。

## 设计App的界面

来从创建一个新的iOS单视图应用开始，命名为SpeechToTextDemo。接着，打开`Main.storyboard`文件添加一个 `UILabel`, 一个`UITextView`和一个 `UIButton`。

你的storyboard应该像这样:

![](http://www.appcoda.com/wp-content/uploads/2016/08/speechkit-demo-1-1024x597.png)

下面需要在`ViewController.swift`中给 `UITextView`和`UIButton`定义outlet变量，在demo中，将`UITextView`命名为“textView”，`UIButton`命名为 “microphoneButton”，新建一个空的action方法作为microphone按钮被按下时的响应方法:

```swfit
@IBAction func microphoneTapped(_ sender: AnyObject) {
 
}
```

若你不喜欢从零开始，直接下载这个[开始工程](https://github.com/appcoda/SpeechToTextDemo/raw/master/SiriDemoStarter.zip)然后跟着下面的教程走吧。

## Using Speech Framework

To use the Speech framework, you have to first import it and adopt the `SFSpeechRecognizerDelegate` protocol. So let’s import the framework, and add its protocol to the `ViewController` class. Now your `ViewController.swift` should look like this:

```swift
import UIKit
import Speech
 
class ViewController: UIViewController, SFSpeechRecognizerDelegate {
    
    
    @IBOutlet weak var textView: UITextView!
    @IBOutlet weak var microphoneButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
    }
 
    @IBAction func microphoneTapped(_ sender: AnyObject) {
 
    }
 
}
```

## User Authorization

Before using the speech framework for speech recognition, you have to first ask for users’ permission because the recognition doesn’t happen just locally on the iOS device but Apple’s servers. All the voice data is transmitted to Apple’s backend for processing. Therefore, it is mandatory to get the user’s authorization.

Let’s authorize the speech recognizer in the viewDidLoad method. The user must allow the app to use the input audio and speech recognition. First, declare a speechRecognizer variable:

```swift
private let speechRecognizer = SFSpeechRecognizer(locale: Locale.init(identifier: "en-US"))  //1
```

And update the the viewDidLoad method like this:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    microphoneButton.isEnabled = false  //2
    
    speechRecognizer.delegate = self  //3
    
    SFSpeechRecognizer.requestAuthorization { (authStatus) in  //4
        
        var isButtonEnabled = false
        
        switch authStatus {  //5
        case .authorized:
            isButtonEnabled = true
            
        case .denied:
            isButtonEnabled = false
            print("User denied access to speech recognition")
            
        case .restricted:
            isButtonEnabled = false
            print("Speech recognition restricted on this device")
            
        case .notDetermined:
            isButtonEnabled = false
            print("Speech recognition not yet authorized")
        }
        
        OperationQueue.main.addOperation() {
            self.microphoneButton.isEnabled = isButtonEnabled
        }
    }
}
```
1. First, we create an SFSpeechRecognizer instance with a locale identifier of en-US so the speech recognizer knows what language the user is speaking in. This is the object that handles speech recognition.
2. By default, we disable the microphone button until the speech recognizer is activated.
3. Then, set the speech recognizer delegate to self which in this case is our ViewController.
4. After that, we must request the authorization of Speech Recognition by calling SFSpeechRecognizer.requestAuthorization.
5. Finally, check the status of the verification. If it’s authorized, enable the microphone button. If not, print the error message and disable the microphone button.

Now you might think that by running the app you would see an authorization alert, but you are mistaken. If you run, the app will crash. But why you may ask?
