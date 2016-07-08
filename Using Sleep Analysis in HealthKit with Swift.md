#Using Sleep Analysis in HealthKit with Swift
##使用Swift与HealthKit的睡眠分析

[原文地址](http://www.appcoda.com/sleep-analysis-healthkit)

睡眠革命是一个当今很流行的话题，用户们对这个有着很大的兴趣，不论是对他们自己的睡眠时间，还是对通过分析采集到的数据揭示的周期性睡眠趋势。伴随着硬件中技术的提升，尤其是手机，给予了这个关注度日益增长的问题一个全新的视角。 

苹果提供了个很cool的手段，以一种安全的方式去操作用户的个人健康数据，将数据安全的存放在了内置的Health app中。你不仅可以使用HealthKit来做健身app，也可以获取睡眠分析的数据。

在这篇教程中，会简要介绍HealthKit框架，并告诉你如何构建一个睡眠分析的简单app。

##简介

HealthKit提供了一种在加密数据库中存储数据的结构，叫做HealthKit store。可以使用HKHealthStore类来访问数据库。iPhone和Apple Watch有各自的HealthKit store。Apple Watch与iPhone的健康数据是同步的，不过Apple Watch会定期清理旧数据以节省空间。HealthKit与健康app在iPad上不可用。

如果你想做一个基于健康数据的iOS或watchOS的app，HealthKit将会是一个有力的工具。HealthKit用于管理多个来源的数据，根据用户的喜好来融合整理这些数据，App也可以访问每一个来源的原始数据，并且自己来处理数据。这些数据不仅可以进行人体测量，用来健身、控制营养，也可以用来睡眠分析。

在文章中剩下的部分，将会展示如何利用HealthKit框架来存储与访问iOS上的睡眠分析数据，对于watchOS应用来说也是同样的方法。请注意这篇教程是基于Swift 2.0与Xcode 7所写，请确保你使用的是Xcode 7(或更高)。

在进行下一步之前，先下载开始工程然后解压它。我已经创建好了UI，当你运行工程时，会出现一个计时器的UI，点击start按钮时计时器会开始计时。 

##使用HealthKit框架

应用的目标就是储存睡眠分析信息，使用Start与Stop按钮来检索。为了使用HealthKit，你必须先在app bundle中开启HealthKit功能。在工程中，点击当前工程target -> capabilities 打开HealthKit。

![](http://www.appcoda.com/wp-content/uploads/2016/05/HealthKit-allow-1024x640.png)

接下来，先在ViewController类中创建一个HKHealthStore的实例，使用如下代码:
````swift
let healthStore = HKHealthStore()
````
后面会使用这个HKHealthStore实例来访问HealthKit store。

如前所述，HealthKit允许用户控制他们的健康数据，因此首先你需要得到用户许可才可以对用户的睡眠分析数据进行访问的权限(读/写)。首先import内置的HealthKit框架，然后如下修改ViewDidLoad方法:
````swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    let typestoRead = Set([
        HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis)!
        ])
    
    let typestoShare = Set([
        HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis)!
        ])
    
    self.healthStore.requestAuthorizationToShareTypes(typestoShare, readTypes: typestoRead) { (success, error) -> Void in
        if success == false {
            NSLog(" Display not allowed")
        }
    }
}
````
这段代码会提示用户允许还是拒绝请求的权限。在闭包中，可以处理成功或失败情况下的行为，得到最终的结果。没必要使用户允许app所有的请求权限，不过在app中必须优雅地处理失败的情况。

不过为了测试，你必须选择“Allow”去允许app获得访问设备中健康数据的权限。

![](http://www.appcoda.com/wp-content/uploads/2016/05/Health-App-Permission.png)
