#Building iOS Apps with Xamarin and Visual Studio
##在Xamarin与Visual Studio中构建iOS应用

***

>* 原文链接 : [Building iOS Apps with Xamarin and Visual Studio](https://www.raywenderlich.com/134049/building-ios-apps-with-xamarin-and-visual-studio)
* 原文作者 : [Bill Morefield](https://www.raywenderlich.com/u/bmorefield)
* 译者 : [yrq110](https://github.com/yrq110)

***

在创建iOS app时，开发者通常会选择苹果提供的语言与集成开发环境: Objective-C / Swift与Xcode，然而这并不是唯一的选择，可以使用多种语言与框架进行iOS app的创建。

[Xamarin](https://xamarin.com/)是一个比较流行的选择，一个跨平台的框架，允许你使用C#和Visual Studio进行iOS, Android, OS X与Windows app的开发。Xamarin的一个主要优势在于可以在iOS与Android间共用一个app代码。

Xamarin跟其他跨平台框架比有一个很大的优点: 使用Xamarin的话你的工程会编译成本地代码(native code)，并可以使用内部的本地API，这意味着一个书写良好的Xamarin app与Xcode中的app是没有区别的。想了解更多细节可以看看这篇文章[Xamarin vs. Native App Development](http://willowtreeapps.com/blog/xamarin-vs-native-app-development/) 。

Xamarin在之前存在一个巨大的缺点: 价格。每个平台每年1000美元的高昂价格也许会使你放弃日常的拿铁与卡布奇诺去支付它...没有咖啡的编程是很危险的。因为这个高昂的价格，直到最近Xamarin都还是主要吸引一些有着庞大预算的企业项目的注意。

不过最近发生了变化，Microsoft收购了Xamarin并宣布它将支持全版本的Visual Studio，包括免费的[Community Edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)，方便了个人开发者与小型组织。

免费? 是个值得庆祝的价格!

<img src="https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/dollar-941246_1280.jpg" width="50%" height="50%" />

更多的咖啡钱!

***
