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

除了开销外, Xamarin的其它优点有:
* 利用现有的C#库与工具来构建移动app。
* 在不同平台间复用app代码。
* 在ASP.Net后端与面向用户的app间共享代码。

Xamarin同时提供了多种工具来选择，取决于你的需求。使用[Xamarin Forms](https://www.xamarin.com/forms)来最大化跨平台的代码复用，无需针对平台来指定功能与特殊的自定义接口，非常有效。 

若你的app需要某些平台独有的特性或设计，使用Xamarin.iOS、Xamarin.Android与其他针对平台的模块来直接使用本地API与框架来进行交互。这些模块提供灵活创建自定义用户接口的功能，同时也允许跨平台共享代码。

在这篇教程中将会使用**Xamarin.iOS**来创建一个用来显示用户照片的iPhone app。

无需任何iOS或Xamarin开发经验，不过需要你掌握一些C#的基本知识使知识的收益最大化。

## 入门

为了使用Xamarin和Visual Studio开发iOS应用，理论上需要两台机器:

1. **一台Windows**  用来运行Visual Studio与写项目代码。
2. **一台安装了Xcode的Mac**  作为构建主机(build host)。可以不是一个专门用来构建的机子，只要在Windows测试与开发时保持网络畅通就行。

如果这两台机子离的比较近就再好不过了，当你在Windows上构建运行时，Mac上的iOS模拟器就会加载对应的项目。

我知道你会说，“我没两台机子咋整?!”

* **对于仅使用Mac的用户**  Xamarin为OS X提供了一个IDE，不过在这篇教程中只关注Visual Studio。因此若你想跟着教程走可以在Mac上运行一个Windows虚拟机，可以使用[VMWare Fusion](https://www.vmware.com/products/fusion)或免费、开源的[VirtualBox](https://www.virtualbox.org/)来创建虚拟机。

若使用Windows虚拟机，需要确保Windows与Mac间保持正常的网络连接。通常会在Windows中ping Mac的IP地址测试一下。

* **对于仅使用Windows的用户**  现在去买个Mac，我会等你！:]也可以使用像[MacinCloud](http://www.macincloud.com/)或[Macminicolo](https://macminicolo.net/)这种提供远程访问服务的Mac主机来构建。

这篇教程中假定你分别使用Mac与Windows电脑，别担心，在Mac上使用Windows虚拟机也是可以的。

## 安装Xcode与Xamarin

若还没准备好Xcode的话先在你的Mac上[下载并安装Xcode](https://itunes.apple.com/us/app/xcode/id497799835?uo=8&at=11ld4k)，跟安装其它App Store上的app一样，不过大小上G了，需要花一些时间。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/danish-butter-cookies-1032894_1280.jpg)

Perfect time for a cookie break!
***

Xcode安好后，在Mac上下载Xamarin Studio，需要提供你的邮箱地址来下载。

下载完成后打开安装包双击**Xamarin.app**安装，接受条款一路continue。

安装器会搜索已存在的工具与当前平台版本，并列出一个当前开发环境的详细列表。确保选择的是**Xamarin.iOS**，点击**Continue**。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/xamarin-installer.png)

接着是安装项目的确认列表，点击**Continue**执行，会出现一个Summary与一个启动Xamarin Studio的选项，点击**Quit**完成安装。

## 安装Visual Studio与Xamarin

这篇教程中可以使用任何版本的Visual Studio，包括免费的[Community Edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)。有些特性在Community Edition中没有，不过这并不影响你开发出复杂的app来。

查看Windows系统的Visual Studio[最低硬件需求](https://www.visualstudio.com/en-us/downloads/visual-studio-2015-system-requirements-vs.aspx#1)，为了有一个好的开发体验，至少要3G的内存。

若还未安装Visual Studio的话，在[Community Edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)网站点击绿色的**Download Community 2015**按钮下载**Community Edition**安装器。

运行安装器开始安装，选择自定义安装选项，在列表中展开**Cross Platform Mobile Development**，选择C#/.NET (Xamarin v4.0.3)(v4.0.3是教程在撰写时的版本，未来也许会变化)。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/vs-installer-354x500.png)

点击Next等待安装完成，会花点儿时间 :]

若你已准备好Visual Studio了不过还没有Xamarin工具，打开Windows系统的**Programs and Features**找到Visual Studio 2015，访问其安装程序，选择**修改**，会在Cross Platform Mobile Development下面看到**C#/.NET (Xamarin v4.0.3)**，选择它点击**更新**进行安装。

安了一大堆东西啊，现在搞定所有需要的东西了!

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Install_Powers.png)

## 创建App

打开Visual Studio选择**File\New\Project**，在Visual C#下展开**iOS**，选择**iPhone**，右侧选择**Single View App**模板。这个模板会创建一个具有单视图控制器的app。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/NewProject-461x320.png)

**Name**和**Solution Name**都输入**ImageLocation**，选择一个app的保存路径点击**OK**创建工程。
