# Core Plot Tutorial: Getting Started
## Core Plot指南:入门

***

>* 原文链接 : [Core Plot Tutorial: Getting Started](https://www.raywenderlich.com/131985/core-plot-tutorial-getting-started)
* 原文作者 : [Attila Hegedüs](https://www.raywenderlich.com/u/cynicalme)
* 译者 : [yrq110](https://github.com/yrq110/)

***

Core Plot是一个2D图表绘图库，可以用于iOS、Mac OS X和tvOS，使用了苹果的框架实现：Quartz与Core Animation，有较稳固的测试覆盖，在BSD许可下发布。

在这片教程中，你将会学到如何使用Core Plot创建饼图与柱状图，还会添加很酷的图表交互功能！

开始前需要安装Xcode 8.0并且对Swift、Interface Builder和storyboard有一些基本的认识与理解，如果你对这些方面不太了解需要看看其他教程学习一下。

教程中使用CocoaPods来安装依赖的第三方库，若你没用过CocoaPods可以先看看其他[教程](https://www.raywenderlich.com/97014/use-cocoapods-with-swift)学习一下。

## 入门

在这篇教程中会做一个周期显示货币汇率的app。在这里下载[开始项目](https://koenig-media.raywenderlich.com/uploads/2016/07/SwiftRates-Starter-Swift3.zip)，解压后打开SwiftRates.xcworkspace。

项目中的一些关键类:
* `DataStore.swift`
  一个从Fixer.io请求汇率数据的辅助类。
* `Rate.swift`
  显示所选日期货币汇率的model。
* `Currency.swift`
  货币类型的model，所支持的货币在Resources/Currencies.plist中定义。
* `MenuViewController.swift`
  app启动时显示的第一个view controller，在这里用户选择一个基本货币和两个比较货币。
* `HostViewController.swift`
  一个view controller容器，根据分段控件的选择来显示PieChartViewController或BarGraphViewController，并且将从DataStore中获取的汇率数据应用于所选择的view controller。
* `PieChartViewController.swift`
  使用饼图显示所选日期的汇率，首先会实现这个图表。
* `BarGraphViewController.swift`
  使用柱状图显示一定天数的汇率.实现饼图后这个就是个小case！

构建并运行一下看看。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/Screen-Shot-2016-05-17-at-9.24.00-PM.png)
