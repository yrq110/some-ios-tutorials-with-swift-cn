#Unit Testing in Xcode 7 with Swift

[原文地址](http://www.appcoda.com/unit-testing-swift/)

每个iOS开发者都需要时不时测试他们的app。如果你不是某种狂热的码农，你应该体会过那种找了好几个小时的bug结果原因出在一个很简单的语法错误时的绝望的感觉，如果没有找到bug那将会更糟。不论你是初学者还是开发过很多app的老手，有规划的进行单元测试都会使你的代码有更高的可信度、更安全，也更容易的debug！

幸运的是Xcode7与Siwft都支持单元测试。虽然单元测试不意味着app没有bug，不过这依旧是一种非常有效的方法去证明代码的每一个部分或单元都尽职尽责，帮助debug过程。

顾名思义，在单元测试中会根据一个代码单元创建小巧且特定的功能性测试。如果通过了测试则会显示一个绿色log，如果由于某种原因没有成功，Xcode会给测试标记“failed”，可以通过这个标记来检视你的代码，去查找失败的原因。

##来看一个Demo

首先，下载这个为你准备的[开始工程](https://github.com/appcoda/SwiftUnitTestDemo/blob/master/PercentageCalculatorStarter.zip?raw=true)，一个根据数字和百分数计算结果的百分比计算器app (比如:80的10%是8)

这个百分比计算器工程很简单的，只需要看看ViewController.swift这个基本文件，代码的注释很直观。

其中有5个IBOutlets: 对应除title意外的屏幕中的每一个空间, 与2个IBActions，分别对应两个滑条。 每个IBAction的名字准确的表达了这个方法的作用。若改变一个滑条的值时，则百分比或数字就会改变。

这里有2个简单的方法，“updateLabels()” 和 “percentage()”。第一个方法的作用的是当滑条的值改变时刷新label显示的内容，第二个的作用是接受两个浮点数然后返回百分比的计算结果。

在模拟器中运行app。一开始一切都很正常，不过如果你改变数字的话会注意到结果会出错。为了找出bug，我们可以将代码分为不同单元然后分别测试它们。这个过程不会解决bug，不过可以缩小你查找bug的范围。

![](http://www.appcoda.com/wp-content/uploads/2016/02/unit-test-demo-app.png)
