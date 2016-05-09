#Sprite Kit Swift 2 Tutorial for Beginners
##Sprite Kit Swift 2 初学指南
[原文地址](https://www.raywenderlich.com/119815/sprite-kit-swift-2-tutorial-for-beginners)

![](http://www.raywenderlich.com/wp-content/uploads/2015/10/iOS9_feast_SpriteKit.jpg)

就像蝙蝠侠与罗宾、超人与路易斯霍恩一样，Sprite Kit与Swift是一对惊艳的组合:
* **Sprite Kit**是一种非常棒的写iOS游戏的方法，它易学、功能强大，并且由苹果全面支持。
* **Swift**是一门易上手的语言，尤其是若你是一位iOS初级开发者的话。

在这篇教程中，你将学会如果用苹果的2D游戏框架-Sprite Kit去写一款简单的2D游戏，使用Swift语言！

你可以按顺序看也可以直接跳到最后的样例工程，那里会出现一些忍者。
>注意。这篇教程在我心中有特殊的地位，它的初版是我在这个站上发布的第一篇教程，使用Cocos2D与Objective-C所编写的游戏。哎呀嘛呀，时过境迁啊！

##Sprite Kit VS Unity
除了Sprite Kit外最火的游戏框架当属Unity。Unity最初是3D引擎，不过现在也全面支持2D了。

所以在你开始之前，我推荐你考虑一下Sprite Kit与Unity，看看哪个更适合你的游戏。
###Sprite Kit的优点:
* **iOS内置**  不需要下载任何的库或者外部依赖。 你可以在其中直接使用其他iOS API，像iAd、In-App Purchases等等。without having to rely on extra plugins.
* **可利用你所掌握的技能**  如果你懂iOS与Swift开发的话，就可以非常快的上手。
* **苹果出品**  这会让你放下心来，因为它将被所有的苹果新产品所支持。比如说你可以不需要挂载地使用相同的Sprite Kit代码在iOS，OS X与tvOS上运行。
* **免费**  可能这是对独立开发者来说最好的原因！ 无需任何费用就可以使用Sprite Kit得所有功能。Unity虽然有免费版本不过并没有专业版本的所有特性(比如说如果你想修改有Unity标志的启动界面的话就需要升级版本)。

###Unity的优点:
* **跨平台**  这是一个重点。如果你使用Sprite Kit你将会被局限在苹果生态中，使用Unity可以轻松的在安卓、Windows和更多的平台上部署。
* **虚拟场景设计**  在Unity中场景的布局与编辑视图层是非常简单的事情，而且可以通过点击一个按钮就能进行游戏的实时测试。Sprite Kit有一个基础的场景编辑器，跟Unity提供的比起来真的是很“基础”。
* **资源包商店**  Unity有一个内置的资源包商店，你可以在那里为你的游戏购买各种各样的组件。这些组件有时可以省掉大半的开发时间！
* **更强大**  一般来说，Unity拥有比Sprite Kit / Scene Kit更多的特性与功能。


##我该选哪个?
看了这么多你可能就想，“好吧，俺该选哪个2D框架？”

答案取决于你的目标是什么。这里是我的两个选择:
* 如果你完全是个新手，或者仅关注苹果生态：使用Sprite Kit - 内置、好学、能找到工作（PS：真的？）。
* 如果你想跨平台，或者做一个复杂的游戏：使用Unity – 更强大、灵活。
If you think Unity is for you, check out some of our Unity written tutorials or our new Unity video tutorial series.
如果你觉得Unity适合你，找一些Unity的教程或者教学视频。

若不是，就开始学习Sprite Kit吧!

##Hello, Sprite Kit!
从一个简单的Hello World项目开始，使用Xcode7中的Sprite Kit游戏模板来运行。

打开Xcode，选择File\New\Project，选择iOS\Application\Game模板，然后点击Next:

![](https://cdn2.raywenderlich.com/wp-content/uploads/2014/09/001_New_Game-480x282.png)

工程名起为SpriteKitSimpleGame, 语言选择Swift, Game Technology选SpriteKit, 设备是iPhone, 然后点Next:

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/10/002_Options-452x320.png)

选择存放工程的地点，点击Create. 选择iPhone 6s模拟器, 点击play按钮运行工程, 在一个简易的启动界面后你应该会看到下图:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/10/003_Hello_World-281x500.png)

Sprite Kit由一些场景的概念所组成，就像一个游戏中的显示的层级与视图。比如说你可能有一个主要游戏区域的场景与一个拥有层级的世界地图场景。
如果你看一眼工程会发现模板已经为你创建好了一个默认的场景 - GameScene。打开GameScene.swift你会看到包含了一段在屏幕中显示文本框的代码，还有一个当你点击就会添加的旋转宇宙飞船。

在这篇教程中主要在GameScene上进行工作。在开始前你需要进行一些调整使你的游戏可以横屏运行而不是竖屏。
##初始设置
这个模板对于你来说有两个问题。第一，游戏需要被设置为横屏而它是竖屏的。第二，它正在使用Sprite Kit的场景编辑器，你在这篇教程中并不需要。来让我们解决这些问题。

第一，在你的工程导航栏中点击你的SpriteKitSimpleGame工程打开target setting，选择SpriteKitSimpleGame target。然后在Deployment Info部分，去掉Portrait使仅有Landscape Left and Landscape Right被选中，像下面这样:

![](https://cdn3.raywenderlich.com/wp-content/uploads/2015/10/004_Landscape-700x377.png)

第二，删除GameScene.sks选择送到废纸篓。这个文件允许你布局精灵与可视场景的其他组件，不过对于我们这个游戏可以通过代码轻松创建所以不需要这个。

然后，打开GameViewController.swift用如下代码替换内容:
~~~~
import UIKit
import SpriteKit
 
class GameViewController: UIViewController {
 
  override func viewDidLoad() {
    super.viewDidLoad()
    let scene = GameScene(size: view.bounds.size)
    let skView = view as! SKView
    skView.showsFPS = true
    skView.showsNodeCount = true
    skView.ignoresSiblingOrder = true
    scene.scaleMode = .ResizeFill
    skView.presentScene(scene)
  }
 
  override func prefersStatusBarHidden() -> Bool {
    return true
  }
}
~~~~
GameViewController是一个UIViewController类, 它的一个根视图是SKView, 是一个包含Sprite Kit场景的视图.

这里你实现了viewDidLoad()去创建一个启动时的GameScene，尺寸与视图本身相同。这就是最初的设置，现在屏幕上有些东西了！
##添加一个精灵
