#Unit Testing in Xcode 7 with Swift

[原文地址](http://www.appcoda.com/unit-testing-swift/)

每个iOS开发者都需要时不时测试他们的app。如果你不是某种狂热的码农，你应该体会过那种找了好几个小时的bug结果原因出在一个很简单的语法错误时的绝望的感觉，如果没有找到bug那将会更糟。不论你是初学者还是开发过很多app的老手，有规划的进行单元测试都会使你的代码有更高的可信度、更安全，也更容易的debug！

幸运的是Xcode7与Siwft都支持单元测试。虽然单元测试不意味着app没有bug，不过这依旧是一种非常有效的方法去证明代码的每一个部分或单元都尽职尽责，帮助debug过程。

顾名思义，在单元测试中会根据一个代码单元创建小巧且特定的功能性测试。如果通过了测试则会显示一个绿色log，如果由于某种原因没有成功，Xcode会给测试标记“failed”，可以通过这个标记来检视你的代码，去查找失败的原因。

##来看一个Demo

首先，下载这个为你准备的[开始工程](https://github.com/appcoda/SwiftUnitTestDemo/blob/master/PercentageCalculatorStarter.zip?raw=true)，一个计算数字百分比结果的计算器app (比如:80的10%是8)

这个百分比计算器工程很简单的，只需要看看ViewController.swift这个基本文件，代码的注释很直观。

其中有5个IBOutlets: 对应除title意外的屏幕中的每一个空间, 与2个IBActions，分别对应两个滑条。 每个IBAction的名字准确的表达了这个方法的作用。若改变一个滑条的值时，则百分比或数字就会改变。

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

至于“是否与预期一致”这个问题使用的是XCTAssert函数。最简单的XCTAssert函数是XCTAssert(expression: BooleanType)，它接受一个布尔型表达式(类似5 > 3, 8.90 == 8.90 或 true) ，若表达式值为true则使测试通过，若为false则测试失败。

试试看! 首先在testPercentageCalculator()方法中添加如下代码。然后将鼠标光标移动到方法名左侧的图标那里，悬停时点击就会开始执行这个测试。

````swift
func testPercentageCalculator() {
        XCTAssert(true)
}
````
如果一切正常，测试通过后会在方法旁显示一个绿色的标记。

![](http://www.appcoda.com/wp-content/uploads/2016/02/unit-test-green-mark.png)
##验证百分比计算
