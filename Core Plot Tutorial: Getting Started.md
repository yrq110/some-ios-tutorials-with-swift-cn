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

Select Get Rates to navigate to the HostViewController and then change the segmented control’s selection. The app really doesn’t do very much… yet. ;]

It’s time in this Core Plot tutorial to get plotting!

### 安装Core Plot

首先需要安装Core Plot。差First in this Core Plot tutorial, you need to install Core Plot. The easiest way to do this is via CocoaPods.

将下面这行添加到Podfile中:

```swift
pod 'CorePlot', '~> 2.2'
```
Open Terminal; cd into your project directory; and run pod install.
After the install completes, build the project.
No errors, right? Great, you’re all setup to use Core Plot. Thanks, CocoaPods. :]
If you do get any errors, try updating CocoaPods via sudo gem install cocoapods and then pod install again.

## Creating the Pie Chart

Open PieChartViewController.swift and add the following import:

```swift
import CorePlot
```
Next, add the following property:

```swift
@IBOutlet weak var hostView: CPTGraphHostingView!
```

CPTGraphHostingView is responsible for “hosting” a chart/graph. You can think of it as a “graph container”.
Next, add the following class extension after the ending class curly brace:

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
You provide data for a Core Plot chart via CPTPieChartDataSource, and you get user interaction events via CPTPieChartDelegate. You’ll fill in these methods as the Core Plot tutorial progresses.

### Setting Up the Graph Host View

To continue this Core Plot tutorial, open Main.storyboard and select the PieChartViewController scene.
Drag a new UIView onto this view. Change its class to CPTGraphHostingView, and connect it to the hostView outlet.
Add constraints on each side to pin this view to its parent view, making sure that Constrain to margins is NOT set:

![](https://koenig-media.raywenderlich.com/uploads/2016/04/swiftrates-05-700x428.png)

Set the background color to any color you like. I used a gray scale color with an opacity of 92%.
Back in PieChartViewController.swift, add the following methods right after viewDidLoad():
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
This sets up the plot right after the subviews are laid out. This is the earliest that the frame size for the view has been set, which you’ll need to configure the plot.
Each method within initPlot() represents a stage in setting up the plot. This helps keep the code a bit more organized.
Add the following to configureHostView():
```swift
hostView.allowPinchScaling = false
```
This disables pinching scaling on the pie chart, which determines whether the host view responds to pinch gestures.
You next need to add a graph to the hostView. Add the following to configureGraph():
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
Here’s a breakdown for each section:
1. You first create an instance of CPTXYGraph and designate it as the hostedGraph of the hostView. This associates the graph with the host view.

The CPTGraph encompasses everything you see in a standard chart or graph: the border, the title, the plotted data, axes, and legend.

By default, CPTXYGraph has a padding of 20 points per side. This doesn’t look great here, so you explicitly set the padding for each side to 0.
2. You next set up the text style for the graph’s title by creating and configuring a CPTMutableTextStyle instance.
3. Lastly, you set the title for the graph and set its style to the one you just created. You also specify the anchoring point to be the top of the view’s bounding rectangle.

Build and run the app, and you should see the chart’s title displayed at the top of the screen:

![](https://koenig-media.raywenderlich.com/uploads/2016/04/swiftrates-03.png)

### Plotting the Pie Chart

The title looks great, but you know what would be even better in this Core Plot tutorial? Actually seeing the pie chart!
Add the following lines of code to configureChart():

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
Here’s what this does:
1. You first get a reference to the graph.
2. You then instantiate a CPTPieChart, set its delegate and data source to be the view controller, and configure its appearance.
3. You then configure the chart’s border style.
4. And then, configure its text style.
5. Lastly, you add the pie chart to the graph.

If you build and run the app right now, you’ll see that nothing has changed… This is because you still need to implement the data source and delegate for the pie chart.
First, replace current numberOfRecords(for:) with the following:
```swift
func numberOfRecords(for plot: CPTPlot) -> UInt {
  return UInt(symbols.count)
}
```
This method determines the number of slices to show on the graph; it will display one pie slice for each symbol.
Next, replace number(for:field:record:) with the following:
```swift
func number(for plot: CPTPlot, field fieldEnum: UInt, record idx: UInt) -> Any? {
  let symbol = symbols[Int(idx)]
  let currencyRate = rate.rates[symbol.name]!.floatValue
  return 1.0 / currencyRate
}
```
The pie chart uses this method to get the “gross” value for the currency symbol at the idx.
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
