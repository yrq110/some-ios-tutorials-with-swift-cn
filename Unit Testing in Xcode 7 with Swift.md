#Unit Testing in Xcode 7 with Swift
##使用Swift在Xcode7中的单元测试

***

>* 原文链接 : [Unit Testing in Xcode 7 with Swift](http://www.appcoda.com/unit-testing-swift/)
* 原文作者 : [MAXIME DEFAUW](http://www.appcoda.com/author/maximedefauw/)
* 译者 : [yrq110](https://github.com/yrq110)

***

每个iOS开发者都需要时不时测试他们的app。如果你不是某种狂热的码农，你应该体会过那种找了好几个小时的bug结果原因出在一个很简单的语法错误时的绝望的感觉，如果没有找到bug那将会更糟。不论你是初学者还是开发过很多app的老手，有规划的进行单元测试都会使你的代码有更高的可信度、更安全，也更容易的debug！

幸运的是Xcode7与Siwft都支持单元测试。虽然单元测试不意味着app没有bug，不过这依旧是一种非常有效的方法去证明代码的每一个部分或单元都尽职尽责，帮助debug过程。

顾名思义，在单元测试中会根据一个代码单元创建小巧且特定的功能性测试。如果通过了测试则会显示一个绿色log，如果由于某种原因没有成功，Xcode会给测试标记“failed”，可以通过这个标记来检视你的代码，去查找失败的原因。


**目录**
* [来看一个Demo](#demo)
* [开始编写单元测试](#开始编写单元测试)
* [验证百分比计算](#验证百分比计算)
* [验证标签](#验证标签)
* [来修复Bug](#bug)


<a name="demo"></a>
##来看一个Demo

首先，下载这个为你准备的[开始工程](https://github.com/appcoda/SwiftUnitTestDemo/blob/master/PercentageCalculatorStarter.zip?raw=true)，一个计算数字百分比结果的计算器app (比如:80的10%是8)

这个百分比计算器工程很简单的，只需要看看ViewController.swift这个基本文件，代码的注释很直观。

其中有5个IBOutlets: 对应除title以为的屏幕中的每一个控件, 与2个IBActions：分别对应两个滑条。 每个IBAction的名字准确的表达了这个方法的作用。若改变一个滑条的值时，则百分比或数字就会改变。

这里有2个简单的方法，“updateLabels()” 和 “percentage()”。第一个方法的作用的是当滑条的值改变时刷新label显示的内容，第二个的作用是接受两个浮点数然后返回百分比的计算结果。

在模拟器中运行app。一开始一切都很正常，不过如果你改变数字的话会注意到结果会出错。为了找出bug，我们可以将代码分为不同单元然后分别测试它们。这个过程不会解决bug，不过可以缩小你查找bug的范围。

![](http://www.appcoda.com/wp-content/uploads/2016/02/unit-test-demo-app.png)


在创建工程时默认选择包含一个测试文件。（如果你想手动添加，可以在选择 File > New > File > Unit Test Case Class under iOS Source）这时，Xcode已经帮你创建好了测试文件，在工程中的“PercentageCalculatorTests” 文件夹下。

![](http://www.appcoda.com/wp-content/uploads/2016/02/xcode-unit-test-option.png)

在PercentageCalculatorTests.swift中的PercentageCalculatorTests class里有4个准备好的方法。其中有两个是示例程序，可以删除(可以通过test关键字看出是测试方法)。另外两个，setUp()与tearDown() 是特殊的样板方法，分别对应开始测试与结束测试时执行的方法。

##开始编写单元测试

是时候来写出第一个单元测试方法了！在这篇教程中我们仅测试ViewController类，需要在PercentageCalculatorTests中添加一个实例。
````swift
class PercentageCalculatorTests: XCTestCase {
    var vc: ViewController!
    
    override func setUp() {
        super.setUp()
        // Put setup code here. This method is called before the invocation of each test method in the class.
    }
    
    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }
    
}
````

PercentageCalculatorTests类是XCTestCase的一个子类，使用了XCTest框架进行打包，每一个XCTextCase子类的实例负责你工程中某一部分的测试，例如一个特性。

在setup方法中初始化vc，这样在setUp()调用后，在每一个测试方法中都能得到一个ViewController实例。 像这样修改setUp()方法
：
````swift
override func setUp() {
    super.setUp()
 
    let storyboard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
    vc = storyboard.instantiateInitialViewController() as! ViewController
}
````
记住，所有测试方法命名必须以关键字test开头，否则Xcode不会识别它。添加一个新的方法testPercentageCalculator()来验证使用ViewController的percentage()方法时是否会正常工作。
````swift
func testPercentageCalculator() {
}
````
在单元测试中会测试一个代码块是否起了作用。通常来说代码块的测试都是简短并且很典型的几行，用来测试一个特定的方法或函数。在测试时需要提供包含输入值的代码单元，允许使用输入值执行代码，然后看看输出值是否与预期一致。

![](http://www.appcoda.com/wp-content/uploads/2016/02/Example.png)

至于“是否与预期一致”这个问题使用的是XCTAssert函数。最简单的XCTAssert函数是XCTAssert(expression: BooleanType)，它接受一个布尔型表达式(类似5 > 3, 8.90 == 8.90 或 true)，若表达式值为true则使测试通过，若为false则测试失败。

试试看! 首先在testPercentageCalculator()方法中添加如下代码。然后将鼠标光标移动到方法名左侧的图标那里，悬停时点击就会开始执行这个测试。

````swift
func testPercentageCalculator() {
        XCTAssert(true)
}
````
如果一切正常，测试通过后会在方法旁显示一个绿色的标记。

![](http://www.appcoda.com/wp-content/uploads/2016/02/unit-test-green-mark.png)
##验证百分比计算

该进入正题了: 测试percentage()方法! 作为vc的属性调用这个方法，vc就是ViewController的实例对象。给它两个浮点数，比方说50和50，将结果保存进一个常量p，此时p应该为25(50的50%是25)。用XCTAssert(p == 25)来检验，执行测试方法。使用如下代码替换testPercentageCalculator()方法的内容:
````swift
func testPercentageCalculator() {
        // Should be 25
        let p = vc.percentage(50, 50)
        XCTAssert(p == 25)
}
````
测试成功了，意味着ViewController的percentage()方法起到了作用，需要去其他地方寻找bug，有可能在updateLabels()方法中？
##验证标签

添加一个新的测试方法testLabelValuesShowedProperly()验证标签是否能正确显示文本内容。这次调用ViewController中的updateLabels()方法，检验每个标签是否会显示我们期望的内容。

注意你给了XCTAsset函数一个新的参数:消息字符串. 由于我们要检查多个值(调用3个XCTAssert)所以这么做很方便。如果测试不成功，这条消息会准确地告诉我们哪里有问题。
````swift
func testLabelValuesShowedProperly() {
        vc.updateLabels(Float(80.0), Float(50.0), Float(40.0))
        
        // The labels should now display 80, 50 and 40
        XCTAssert(vc.numberLabel.text == "80.0", "numberLabel doesn't show the right text")
        XCTAssert(vc.percentageLabel.text == "50.0%", "percentageLabel doesn't show the right text")
        XCTAssert(vc.resultLabel.text == "40.0", "resultLabel doesn't show the right text")
}
````

当执行这个方法时会得到一个错误，三个标签都为空。这怎么可能？

在故事板文件中创建了这些标签，因此在视图加载时它们初始化了，不过在单元测试中loadView()方法并未被触发，并没有创建标签所以为空。对这个问题的解决方法可以是调用一下vc.loadView()，不过Apple在她的文档中不推荐这么做，因为对象在加载后再次加载会引起内存泄漏。

应该访问vc的view属性，这样的话不仅仅是loadView()可以触发所有视图所需要的方法。修改testLabelValuesShowedProperly()中的代码：
````swift
func testLabelValuesShowedProperly() {
        let _ = vc.view
        vc.updateLabels(Float(80.0), Float(50.0), Float(40.0))
        
        // The labels should now display 80, 50 and 40
        XCTAssert(vc.numberLabel.text == "80.0", "numberLabel doesn't show the right text")
        XCTAssert(vc.percentageLabel.text == "50.0%", "percentageLabel doesn't show the right text")
        XCTAssert(vc.resultLabel.text == "40.0", "resultLabel doesn't show the right text")
}
````
注意这里使用了下划线( _ ) 作为常量名，这是因为我们不需要view也不会用到它，告诉编译器“假装访问一下view触发它的方法就行了”。

执行测试(如果你想执行测试类中的所有测试可以点击“class PercentageCalculatorTests”旁的方形图标)

![](http://www.appcoda.com/wp-content/uploads/2016/02/unit-test-demo-fail.png)

<a name="bug"></a>
##来修复Bug

如你所见，测试失败了! 我们在方法中输入的详细错误信息会帮助我们迅速识别出引起bug的潜在原因。测试结果告诉我们resultsLabel没有显示正确的文本。来进入ViewController看看标签的text属性是如何设置的，在ViewController.swift中的updateLabels()中我们可以发现引起bug的原因:
````swift
self.resultLabel.text = "\(rV + 10)"
````
应该改为:
````swift
self.resultLabel.text = "\(rV)"
````
改下你的代码然后再次运行测试，现在就一切正常了!

##总结

在这篇教程中，学习了Xcode中的单元测试与如何帮助你在代码中找到bug，单元测试也可以用来进行性能测试与异步测试。另一件你也许感兴趣的就是UI测试，通过记录app的行为来测试你的app在实际的生产环境中的显示情况，可以看看这个[WWDC视频](https://developer.apple.com/videos/play/wwdc2015-406/)来了解一下UI测试。

可以从Github上下载[最后的工程](https://github.com/appcoda/SwiftUnitTestDemo)
