#Bond Tutorial: Bindings in Swift
##学习Bond：Swift中的数据绑定

>注意: 升级到Xcode 7.3, iOS 9.3, Swift 2.2. 2016.04.08

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/01/Binding-with-bond-feature-250x250.png)

对UI代码的手动绑定感到疲倦吗？我想是的。这是个必要的工作我们不得不去做，不是吗？

Swift Bond是一个绑定框架，可以免去在绑定UI时的繁琐工作，这样就有了更多的时间去思考如何解决应用中的问题。

在这篇Bond教程中，会学着使用Bond来节省时间，写出可维护的代码。Bond与ReactiveCocoa很像，在文章的最后我会说一下，现在让我们来使用Bond进行绑定吧 ... Swift Bond。

>注意: 对于不知道Swift Bond的读者，这是一个框架的名字,来源于小说中的间谍-James Bond, 他总是这样介绍自己“Bond … James Bond.” ，
他很喜欢干马天尼，“摇匀，不要搅拌”。放轻松！准备进入Swift Bond的世界！

##快速绑定

启动Xcode开始写一些代码。

首先，下载[开始工程](http://www.raywenderlich.com/wp-content/uploads/2016/01/BindingWithBond-Starter.zip)，包含了应用的基本框架
