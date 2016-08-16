#Using Sleep Analysis in HealthKit with Swift
##使用Swift与HealthKit的睡眠分析

***

>* 原文链接 : [Building a Chat App in Swift Using Multipeer Connectivity Framework](http://www.appcoda.com/chat-app-swift-tutorial/)
* 原文作者 : [GABRIEL THEODOROPOULOS](http://www.appcoda.com/author/gabrielth/)
* 译者 : [yrq110](https://github.com/yrq110)

***
[原文地址](http://www.appcoda.com/sleep-analysis-healthkit)

译者:[yrq110](https://github.com/yrq110)

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

##写入睡眠分析数据

首先我们如何来检索睡眠分析数据呢？根据苹果的文档，每个睡眠分析的样本只能对应一种状态，为了同时表现用户在上床与熟睡两种状态，HealthKit使用多于2个的样本。比较开始与结束时的这些样本，app可以计算出多种统计数据

* 进入睡眠所花费的时间
* 用户实际睡眠的时间百分比
* 用户在床上醒来的次数
* 上床与睡眠所花费的总时间

![](http://www.appcoda.com/wp-content/uploads/2016/05/record_sleep_data-1024x525.png)

简要的来说，需要执行以下步骤来将睡眠分析数据储存进HealthKit store中:

1. 定义两个NSDate对象，对应开始时间与结束时间
2. 使用HKCategoryTypeIdentifierSleepAnalysis创建一个HKObjectType的实例
3. 创建一个HKCategorySample类型的新对象，通常会使用定义好的类别样本去记录睡眠数据，每个样本都可以表现出用户在单位时间内床上或睡眠时的数据，需要创建在时间上重叠的床上样本与睡眠样本。
4. 最后，使用HKHealthStore的saveObject保存对象

>关于样本类型，可以看下[HealthKit Constants Reference](https://developer.apple.com/library/ios/documentation/HealthKit/Reference/HealthKit_Constants/index.html#//apple_ref/doc/uid/TP40014710)。

下面这段代码用Swift实现了上面的步骤，同时保存了在床上和睡眠两种状态的睡眠分析数据。请在 ViewController类中插入这个方法:

````swift
func saveSleepAnalysis() {
    
    // alarmTime and endTime are NSDate objects
    if let sleepType = HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis) {
        
        // we create our new object we want to push in Health app
        let object = HKCategorySample(type:sleepType, value: HKCategoryValueSleepAnalysis.InBed.rawValue, startDate: self.alarmTime, endDate: self.endTime)
        
        // at the end, we save it
        healthStore.saveObject(object, withCompletion: { (success, error) -> Void in
            
            if error != nil {
                // something happened
                return
            }
            
            if success {
                print("My new data was saved in HealthKit")
                
            } else {
                // something happened again
            }
            
        })
        
        
        let object2 = HKCategorySample(type:sleepType, value: HKCategoryValueSleepAnalysis.Asleep.rawValue, startDate: self.alarmTime, endDate: self.endTime)
        
        healthStore.saveObject(object2, withCompletion: { (success, error) -> Void in
            if error != nil {
                // something happened
                return
            }
            
            if success {
                print("My new data (2) was saved in HealthKit")
            } else {
                // something happened again
            }
            
        })
        
    }
    
}
````
当我们需要保存睡眠分析数据到HealthKit中时就可以调用它。

##读取睡眠分析数据

为了读取睡眠分析数据，需要创建一个查询对象。首先应定义HKCategoryTypeIdentifierSleepAnalysis中需要的HKObjectType类别。也许你想使用谓词过滤器去过滤检索得到的数据，检索的数据是startDate与endDate这两个NSDate对象对应的时间范围内的数据，也可能想创建一个用于整理检索查询结果并选择期望结果的sortDescripter。

检索睡眠分析数据的代码应如下所示:
````swift
func retrieveSleepAnalysis() {
    
    // first, we define the object type we want
    if let sleepType = HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis) {
        
        // Use a sortDescriptor to get the recent data first
        let sortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierEndDate, ascending: false)
        
        // we create our query with a block completion to execute
        let query = HKSampleQuery(sampleType: sleepType, predicate: nil, limit: 30, sortDescriptors: [sortDescriptor]) { (query, tmpResult, error) -> Void in
            
            if error != nil {
                
                // something happened
                return
                
            }
            
            if let result = tmpResult {
                
                // do something with my data
                for item in result {
                    if let sample = item as? HKCategorySample {
                        let value = (sample.value == HKCategoryValueSleepAnalysis.InBed.rawValue) ? "InBed" : "Asleep"
                        print("Healthkit sleep: \(sample.startDate) \(sample.endDate) - value: \(value)")
                    }
                }
            }
        }
        
        // finally, we execute our query
        healthStore.executeQuery(query)
    }
}
````
这段代码查询HealthKit中的数据得到所有睡眠分析数据，并使用降序排列。每一次查询都会打印出开始时间、结束时间与对应的状态值。这里将限制参数设成了30，因此会检索最后30个样本，也可以使用谓词过滤的方法去选择你想要的开始与结束时间。

##App测试

在demo中我使用了一个计时器来表现时间的流逝，点击start按钮启动计时器。点击开始与结束按钮就会创建对应的NSDate对象，保存这个时间段内的睡眠分析数据。在停止的方法内可以调用saveSleepAnalysis()与retrieveSleepAnalysis()方法来保存与取得睡眠数据。

````swift
@IBAction func stop(sender: AnyObject) {
    endTime = NSDate()
    saveSleepAnalysis()
    retrieveSleepAnalysis()
    timer.invalidate()
}
````
在app中你也许会修改保存不同状态数据的开始与结束时间。

做出修改后，启动demo app与计时器，让它运行几分钟然后点击Stop按钮。打开Health app，就会看到睡眠数据了。

![](http://www.appcoda.com/wp-content/uploads/2016/06/sleep-analysis-test-1024x725.png)

##一些HealthKit App的建议

HealthKit为app开发者提供了一个可以分享并能轻松访问到用户数据的公共平台，避免了数据的复制与不一致的问题。在苹果的审核指南中，对于使用HealthKit时对读取/写入权限的请求没有明确说明的app可能会被拒绝。 

存储虚假或不正确数据的Health App也会被拒绝，意味着你不能使用自己的算法去计算不同的健康数据，像这篇教程中的睡眠分析。应读取并使用内置传感器数据，操作各种参数避免计算虚假的数据。

可以从[这里](https://github.com/appcoda/SleepAnalysis)下载完整的Xcode工程。
