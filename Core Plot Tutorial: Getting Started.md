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

选择Get Rates进入HostViewController界面，改变分段控件的选择，app还没有实现这些功能...

是时候学习使用Core Plot绘图了!

### 安装Core Plot

首先需要安装Core Plot，使用CocoaPods安装比较简单。

将下面这行添加到Podfile中:

```swift
pod 'CorePlot', '~> 2.2'
```
打开终端，进入项目目录，执行pod install。

安装完成后构建项目。没毛病，现在准备好Core Plot了，谢谢你，CocoaPods :]

如果出了一些错误，可以尝试一下使用sudo gem install cocoapods更新CocoaPods然后再执行pod install。

## 创建饼图

打开PieChartViewController.swift文件导入Core Plot:

```swift
import CorePlot
```
接着添加如下属性:

```swift
@IBOutlet weak var hostView: CPTGraphHostingView!
```

CPTGraphHostingView负责"管理"一个图表，可以把它当做一个"图表容器"。

接着，在类的大括号结束符后添加如下类扩展:

```swift
extension PieChartViewController: CPTPieChartDataSource, CPTPieChartDelegate {
 
  func numberOfRecords(for plot: CPTPlot) -> UInt {
    return 0
  }
 
  func number(for plot: CPTPlot, field fieldEnum: UInt, record idx: UInt) -> Any? {
    return 0
  }
 
  func dataLabel(for plot: CPTPlot, record idx: UInt) -> CPTLayer? {
    return nil
  }
 
  func sliceFill(for pieChart: CPTPieChart, record idx: UInt) -> CPTFill? {
    return nil
  }
 
  func legendTitle(for pieChart: CPTPieChart, record idx: UInt) -> String? {
    return nil
  }  
}
```
通过CPTPieChartDataSource提供Core Plot图表的数据，通过CPTPieChartDelegate得到用户的响应事件，会在之后填充这些方法的内容。

### 设置图表管理视图

打开Main.storyboard，选择PieChartViewController。

把一个新的UIView拖拽到这个视图中，将它的类改为CPTGraphHostingView，连接到hostView。

在每条边上添加约束使其嵌在父视图上，确保Constrain to margins没有被勾选:

![](https://koenig-media.raywenderlich.com/uploads/2016/04/swiftrates-05-700x428.png)

背景色随意设置，我使用透明度是92%的灰度色。

回到PieChartViewController.swif文件，在viewDidLoad()后添加如下方法:

```swift
override func viewDidLayoutSubviews() {
  super.viewDidLayoutSubviews()
  initPlot()
}
 
func initPlot() {
  configureHostView()
  configureGraph()
  configureChart()
  configureLegend()
}
 
func configureHostView() {
}
 
func configureGraph() {
}
 
func configureChart() {
}
 
func configureLegend() {
}
```
子视图布局好后在这里设置需要绘制的图标，这里是最早设置视图尺寸的地方，需要在这里设置图表的参数。

initPlot()中的每一个方法都是图标设置中的一环，这种写法使代码更具组织性。

添加如下代码到configureHostView()方法中:
```swift
hostView.allowPinchScaling = false
```
这里禁用了饼图的捏合缩放，决定饼图的管理视图是否会对捏合手势作出响应。

接着在hostView中添加一个图像。在configureGraph()方法中添加如下代码:
```swift
// 1 - Create and configure the graph
let graph = CPTXYGraph(frame: hostView.bounds)
hostView.hostedGraph = graph
graph.paddingLeft = 0.0
graph.paddingTop = 0.0
graph.paddingRight = 0.0
graph.paddingBottom = 0.0
graph.axisSet = nil
 
// 2 - Create text style
let textStyle: CPTMutableTextStyle = CPTMutableTextStyle()
textStyle.color = CPTColor.black()
textStyle.fontName = "HelveticaNeue-Bold"
textStyle.fontSize = 16.0
textStyle.textAlignment = .center
 
// 3 - Set graph title and text style
graph.title = "\(base.name) exchange rates\n\(rate.date)"
graph.titleTextStyle = textStyle
graph.titlePlotAreaFrameAnchor = CPTRectAnchor.top
```
下面是对代码的分解:
1. 首先新建了一个CPTXYGraph实例，将其设为hostView的hostedGraph属性值，结合图形与管理视图。

CPTGraph包含一个标准图表中的所有元素：边界、标题、绘制的数据、坐标轴和图例。

默认情况下，CPTXYGraph的每一条边有20 px的内边距。看起来并不是很好，你需要将它设为0。

2. 接着通过创建CPTMutableTextStyle实例设置图形的文本样式。
3. 最后设置图表的标题，将它的样式设为刚刚创建好的样式，并将锚点设为视图矩形的顶部。

构建并运行app，你应该会看到屏幕上方显示的图表标题。

![](https://koenig-media.raywenderlich.com/uploads/2016/04/swiftrates-03.png)

### 绘制饼图

标题看着还行，不过怎么能做的更好？来看看如何绘制饼图！

将如下代码添加到configureChart()方法中:

```swift
// 1 - Get a reference to the graph
let graph = hostView.hostedGraph!
 
// 2 - Create the chart
let pieChart = CPTPieChart()
pieChart.delegate = self
pieChart.dataSource = self
pieChart.pieRadius = (min(hostView.bounds.size.width, hostView.bounds.size.height) * 0.7) / 2
pieChart.identifier = NSString(string: graph.title!)
pieChart.startAngle = CGFloat(M_PI_4)
pieChart.sliceDirection = .clockwise
pieChart.labelOffset = -0.6 * pieChart.pieRadius
 
// 3 - Configure border style
let borderStyle = CPTMutableLineStyle()
borderStyle.lineColor = CPTColor.white()
borderStyle.lineWidth = 2.0
pieChart.borderLineStyle = borderStyle
 
// 4 - Configure text style
let textStyle = CPTMutableTextStyle()
textStyle.color = CPTColor.white()
textStyle.textAlignment = .center
pieChart.labelTextStyle = textStyle
 
// 5 - Add chart to graph
graph.add(pieChart)
```
这些代码干了什么:
1. 首先得到一个图的引用。
2. 接着实例化CPTPieChart，设置它的委托与数据源为view controller，并设置它的外观。
3. 设置表的边界样式。
4. 设置表的文本样式。
5. 最后将表添加到图中。

如果你现在就运行app的话会看不到任何改变... 这是因为还需要为饼图实现数据源与委托。

首先使用如下代码替换numberOfRecords(for:)中的内容:
```swift
func numberOfRecords(for plot: CPTPlot) -> UInt {
  return UInt(symbols.count)
}
```
这个方法决定了在图中显示的分块数，会为每一个货币符号分配一个饼图区域。

接着使用如下代码替换number(for:field:record:)中的内容:
```swift
func number(for plot: CPTPlot, field fieldEnum: UInt, record idx: UInt) -> Any? {
  let symbol = symbols[Int(idx)]
  let currencyRate = rate.rates[symbol.name]!.floatValue
  return 1.0 / currencyRate
}
```
饼图使用这个方法在idx中得到货币符号的值。

You should note that this value is not a percentage. Rather, this method calculates the currency exchange rate relative to the base currency: the return value of 1.0 / currencyRate is the exchange rate for “1 base currency per value of another comparison currency.”

CPTPieChart will take care of calculating the percentage value for each slice, which ultimately will determine how big each slice is, using these values.

Next, replace dataLabelForPlot(for:record:) with the following:
```swift
func dataLabel(for plot: CPTPlot, record idx: UInt) -> CPTLayer? {
  let value = rate.rates[symbols[Int(idx)].name]!.floatValue
  let layer = CPTTextLayer(text: String(format: "\(symbols[Int(idx)].name)\n%.2f", value))
  layer.textStyle = plot.labelTextStyle
  return layer
}
```
This method returns a label for the pie slice. The expected return type, CPTLayer is similar to a CALayer. However, a CPTLayer is abstracted to work on both Mac OS X and iOS and provides other drawing niceties used by Core Plot.
Here, you create and return a CPTTextLayer, which is a subclass of CPTLayer designed to display text.
Finally, you’ll add color to the slices by replacing sliceFillForPieChart(for:, record:) with the following:
```swift
func sliceFill(for pieChart: CPTPieChart, record idx: UInt) -> CPTFill? {
  switch idx {
  case 0:   return CPTFill(color: CPTColor(componentRed:0.92, green:0.28, blue:0.25, alpha:1.00))
  case 1:   return CPTFill(color: CPTColor(componentRed:0.06, green:0.80, blue:0.48, alpha:1.00))
  case 2:   return CPTFill(color: CPTColor(componentRed:0.22, green:0.33, blue:0.49, alpha:1.00))
  default:  return nil
  }
}
```
Build and run, and you’ll see a nifty-looking pie chart:

![](https://koenig-media.raywenderlich.com/uploads/2016/04/swiftrates-04.png)

