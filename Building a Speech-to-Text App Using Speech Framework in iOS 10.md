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

## 使用Speech框架

要使用Speech框架首先需要import然后满足`SFSpeechRecognizerDelegate`协议。来先import框架然后将这个协议添加到`ViewController`中，这样做之后`ViewController.swift`应变成下面这样:

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

## 使用授权

在使用speech框架进行语音识别之前，首选需要请求用户许可，这是因为这个识别的过程不是在iOS设备上进行的而是在苹果的服务器上。所有音频数据都会发送给苹果后端去处理，因此需要写一些代码来得到用户的授权。

在viewDidLoad方法中进行speech框架的授权，用户必须允许app使用输入音频与语音识别。首先声明一个speechRecognizer变量:

```swift
private let speechRecognizer = SFSpeechRecognizer(locale: Locale.init(identifier: "en-US"))  //1
```

如下修改viewDidLoad方法:

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

1. 首先新建一个SFSpeechRecognizer常量，设置locale参数为en-US——指明用户将说的是何种语言。这个就是用来处理语音识别的对象。
2. 默认情况下禁用microphone按钮直到speechRecognizer被开启。
3. 接着设置speechRecognizer的委托为self，这里指ViewController。
4. 之后，必须请求语音识别的授权，通过调用SFSpeechRecognizer.requestAuthorization。
5. 最后检查授权的状态，如果被授权则启用microphone按钮，否则输出错误信息并禁用microphone按钮。

现在你可能会认为运行app时会弹出授权提醒的窗口，然而你错了。如果你运行了就会崩溃，这是为何?

## 提供授权信息

苹果要求所有授权都需要有一个来自app的自定义信息(授权描述)。在语音识别的情况下，我们必须得到两样事物的授权:

1. 麦克风
2. 语音识别

为了自定义这个信息，需要在`info.plist`文件中提供这些信息。

打开`info.plist`文件的源代码。首先在`info.plist`上右击，选择Open As > Source Code。最后复制下面的代码粘贴在`</dict>`标签前。

```xml
<key>NSMicrophoneUsageDescription</key>  <string>Your microphone will be used to record your speech when you press the \&quot;Start Recording\&quot; button.</string>
 
<key>NSSpeechRecognitionUsageDescription</key>  <string>Speech recognition will be used to determine which words you speak into this device\&apos;s microphone.</string>
```

在`info.plist`中添加了两个key:

1. `NSMicrophoneUsageDescription` – 授权音频输入的自定义信息。注意音频输入的授权只发生在用户点击microphone按钮时。
2. `NSSpeechRecognitionUsageDescription` – 语音识别的自定义信息。

Feel free to change the values of these records. Now hit the Run button, you should be able to compile and run the app without any errors.

![](http://www.appcoda.com/wp-content/uploads/2016/08/speech-kit-2.png)

> 注意: 若当你运行时没有看到输入音频的授权，是因为app是在模拟器上运行的。iOS模拟器是不会访问Mac的麦克风的。
