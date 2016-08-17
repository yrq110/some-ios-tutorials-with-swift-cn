#Building a Speech-to-Text App Using Speech Framework in iOS 10
##使用iOS 10的Speech框架构建一个语音转文本app

***

>* 原文链接 : [Building a Speech-to-Text App Using Speech Framework in iOS 10](http://www.appcoda.com/siri-speech-framework/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : [yrq110](https://github.com/yrq110)

***


At WWDC 2016, Apple introduced the Speech framework, a useful API for speech recognition. In fact, Speech Kit is the framework which Siri uses for speech recognition. There are a handful of speech recognition frameworks available today, but they are either very expensive or simply not as good. In this tutorial, I will show you how to create a Siri-like app for speech to text using Speech Kit.

## Designing the App UI

Let’s start by creating a new iOS Single View Application project with the name SpeechToTextDemo. Next, go to your `Main.storyboard` and add a `UILabel`, a `UITextView`, and a `UIButton`.

Your storyboard should look something like this:

![](http://www.appcoda.com/wp-content/uploads/2016/08/speechkit-demo-1-1024x597.png)

Next you define outlet variable for the `UITextView` and the `UIButton` in `ViewController.swift`. In this demo, I set the name of `UITextView` as “textView”, and the name of the `UIButton` as “microphoneButton”. Also, create an empty action method which is triggered when the microphone button is tapped:

```swfit
@IBAction func microphoneTapped(_ sender: AnyObject) {
 
}
```

If you don’t want to start from scratch, you can download the starter project and continue to follow the tutorial.
