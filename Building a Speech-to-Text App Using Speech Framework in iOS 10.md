#Building a Speech-to-Text App Using Speech Framework in iOS 10
##使用iOS 10的Speech框架构建一个语音转文本app

***

>* 原文链接 : [Building a Speech-to-Text App Using Speech Framework in iOS 10](http://www.appcoda.com/siri-speech-framework/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : [yrq110](https://github.com/yrq110)

***


在WWDC 2016上苹果介绍了Speech框架-一个用于语音识别的API，其实Speech Kit就是Siri用于语音识别的框架。目前有少数可用的语音识别框架，不过那些要么太贵要么就是不太好。在这篇教程中，将会给你展示如何使用Speech Kit构建一个像Siri那样的语音转文本app。


**目录**
* [设计App的界面](#1)
* [使用Speech框架](#2)
* [使用授权](#3)
* [提供授权信息](#4)
* [处理语音识别](#5)
* [触发语音识别](#6)
* [总结](#7)

<a name="1"></a>
## 设计App的界面

来从创建一个新的iOS单视图应用开始，命名为`SpeechToTextDemo`。接着，打开`Main.storyboard`文件添加一个 `UILabel`, 一个`UITextView`和一个 `UIButton`。

你的storyboard应该像这样:

![](http://www.appcoda.com/wp-content/uploads/2016/08/speechkit-demo-1-1024x597.png)

下面需要在`ViewController.swift`中给 `UITextView`和`UIButton`定义outlet变量，在demo中，将`UITextView`命名为“textView”，`UIButton`命名为 “microphoneButton”，新建一个空的action方法作为microphone按钮被按下时的响应方法:

```swfit
@IBAction func microphoneTapped(_ sender: AnyObject) {
 
}
```

若你不喜欢从零开始，直接下载这个[开始工程](https://github.com/appcoda/SpeechToTextDemo/raw/master/SiriDemoStarter.zip)然后跟着下面的教程走吧。

<a name="2"></a>
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
<a name="3"></a>
## 使用授权

在使用speech框架进行语音识别之前，首选需要请求用户许可，这是因为这个识别的过程不是在iOS设备上进行的而是在苹果的服务器上。所有音频数据都会发送给苹果后端去处理，因此需要写一些代码来得到用户的授权。

在viewDidLoad方法中进行speech框架的授权，用户必须允许app使用输入音频与语音识别。首先声明一个`speechRecognizer`变量:

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

<a name="4"></a>
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

添加完这些记录后点击Run按钮编译并运行app。

![](http://www.appcoda.com/wp-content/uploads/2016/08/speech-kit-2.png)

> 注意: 若当你运行时没有看到输入音频的授权，是因为app是在模拟器上运行的。iOS模拟器是不会访问Mac的麦克风的。

<a name="5"></a>
## 处理语音识别

现在我们已经完成了用户授权部分，来着手语音识别的实现，从在`ViewController`中定义如下对象开始:
```swift
private var recognitionRequest: SFSpeechAudioBufferRecognitionRequest?
private var recognitionTask: SFSpeechRecognitionTask?
private let audioEngine = AVAudioEngine()
```
1. 这个对象用来处理语音识别请求，给语音识别器提供一个音频输入。
2. 保存识别请求的结果，使用这个对象可以方便的取消或停止任务。
3. 这是你的`audioEngine`，负责提供输入音频。

新建一个`startRecording()`方法。
```swift
func startRecording() {
    
    if recognitionTask != nil {
        recognitionTask?.cancel()
        recognitionTask = nil
    }
    
    let audioSession = AVAudioSession.sharedInstance()
    do {
        try audioSession.setCategory(AVAudioSessionCategoryRecord)
        try audioSession.setMode(AVAudioSessionModeMeasurement)
        try audioSession.setActive(true, with: .notifyOthersOnDeactivation)
    } catch {
        print("audioSession properties weren't set because of an error.")
    }
    
    recognitionRequest = SFSpeechAudioBufferRecognitionRequest()
    
    guard let inputNode = audioEngine.inputNode else {
        fatalError("Audio engine has no input node")
    }
    
    guard let recognitionRequest = recognitionRequest else {
        fatalError("Unable to create an SFSpeechAudioBufferRecognitionRequest object")
    }
    
    recognitionRequest.shouldReportPartialResults = true
    
    recognitionTask = speechRecognizer.recognitionTask(with: recognitionRequest, resultHandler: { (result, error) in 
        
        var isFinal = false
        
        if result != nil {
            
            self.textView.text = result?.bestTranscription.formattedString
            isFinal = (result?.isFinal)!
        }
        
        if error != nil || isFinal {
            self.audioEngine.stop()
            inputNode.removeTap(onBus: 0)
            
            self.recognitionRequest = nil
            self.recognitionTask = nil
            
            self.microphoneButton.isEnabled = true
        }
    })
    
    let recordingFormat = inputNode.outputFormat(forBus: 0)
    inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { (buffer, when) in
        self.recognitionRequest?.append(buffer)
    }
    
    audioEngine.prepare()
    
    do {
        try audioEngine.start()
    } catch {
        print("audioEngine couldn't start because of an error.")
    }
    
    textView.text = "Say something, I'm listening!"
    
}
```

这个函数会在开始录音按钮被按下时调用，主要功能是开启语音识别与麦克风，来逐行分析下:

1. 3-6行 – 检查`recognitionTask`是否在运行中，如果是则取消。
2. 8-15行 – 创建一个`AVAudioSession`对象为录音做准备，将category设置为Record(录制音频。静音不停止)，mode设置为Measurement(减少设备在处理音频I/O 的信号量时的影响)，然后启用。注意设置的这些属性有可能抛出异常，因此必须把它们放在一个try catch语句中。
3. 17行 – 初始化`recognitionRequest`，新建`SFSpeechAudioBufferRecognitionRequest`对象，之后会使用它来将音频数据递送给苹果服务器。
4. 19-21行 – 检查`audioEngine`是否有录音的音频输入，没有则报告一个重要错误。
5. 23-25行 – 检查`recognitionRequest`对象是否被初始化是否为空。
6. 27行 – 告诉`recognitionRequest`分段输出用户说话的语音识别结果。
7. 29行 – 调用`speechRecognizer`的`recognitionTask`方法开始识别，这个函数有一个完成句柄，每次识别引擎收到输入时都会调用这个句柄，在被取消或停止时会返回一个最终的记录。
8. 31行 – 定义一个决定识别是否终止的布尔值。
9. 35行 – 若结果非空，则设置`textView.text`属性为结果中最好的记录。若结果为最终结果则设`isFinal`为true。
10. 39-47行 – 若没有任何错误或者有最终结果了，则停止`audioEngine`、`recognitionRequest`与`recognitionRequest`，同时启用开始录音按钮。
11. 50-53行 – 给`recognitionRequest`添加一个音频输入，注意在启动`recognitionTask`之后添加音频输入是可以的。Speech框架会在添加音频输入的同时开始识别。
12. 55行 – 准备并开启`audioEngine`(音频输入)。

<a name="6"></a>
## 触发语音识别

我们需要确保当新建语音识别任务时它的功能是可用的，因此需要在`ViewController`中添加一个委托方法。当语音识别不可用或者状态发生改变时，需要设置`microphoneButton.enable`属性。实现SFSpeechRecognizerDelegate协议中的availabilityDidChange方法，如下所示:
```swift
func speechRecognizer(_ speechRecognizer: SFSpeechRecognizer, availabilityDidChange available: Bool) {
    if available {
        microphoneButton.isEnabled = true
    } else {
        microphoneButton.isEnabled = false
    }
}
```
当可用性改变时会调用这个方法。若语音识别可用，则启用录音按钮。

最后一件事就是更新响应事件的方法`microphoneTapped(sender:)`:
```swift
@IBAction func microphoneTapped(_ sender: AnyObject) {
    if audioEngine.isRunning {
        audioEngine.stop()
        recognitionRequest?.endAudio()
        microphoneButton.isEnabled = false
        microphoneButton.setTitle("Start Recording", for: .normal)
    } else {
        startRecording()
        microphoneButton.setTitle("Stop Recording", for: .normal)
    }
}
```
在这个函数中必须检查`audioEngine`是否正在运行，若运行中则应该停止`audioEngine`，终止`recognitionRequest`的音频输入，禁用`microphoneButton`，设置按钮标题为"开始录音"。

若`audioEngine`可用则app会调用`startRecording()`并设置按钮标题为"停止录音"。

棒！完成了准备工作，下面来测试app，将app部署到iOS 10设备上，点击开始录音按钮，说点什么吧！

![](http://www.appcoda.com/wp-content/uploads/2016/08/speech-kit-3-1024x608.png)

注意:

1. 苹果限制每台设备的识别。不太清楚详情，你可以联系苹果获得更多信息。
2. 苹果限制每个app的识别。
3. 若你经常被限制，联系一下苹果，它们会解决的。
4. 语音识别很费电，很耗资源。
5. 语音识别每次仅持续1分钟左右。

<a name="7"></a>
## 总结

期待开发者们利用新的API作出更多惊艳的app~

可以看看[WWDC 2016 session 509](https://developer.apple.com/videos/play/wwdc2016/509/)获取更多的信息。

在[Github](https://github.com/appcoda/SpeechToTextDemo)上查看完整的工程。
