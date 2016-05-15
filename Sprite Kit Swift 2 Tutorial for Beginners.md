#Sprite Kit Swift 2 Tutorial for Beginners
##Sprite Kit Swift 2 初学指南
[原文地址](https://www.raywenderlich.com/119815/sprite-kit-swift-2-tutorial-for-beginners)

译者:[yrq110](https://github.com/yrq110/)

![](http://www.raywenderlich.com/wp-content/uploads/2015/10/iOS9_feast_SpriteKit.jpg)

就像蝙蝠侠与罗宾、超人与路易斯霍恩一样，Sprite Kit与Swift是一对惊艳的组合:
* **Sprite Kit**是一种非常棒的写iOS游戏的方法，它易学、功能强大，并且由苹果全面支持。
* **Swift**是一门易上手的语言，尤其是若你是一位iOS初级开发者的话。

在这篇教程中，你将学会如果用苹果的2D游戏框架-Sprite Kit去写一款简单的2D游戏，使用Swift语言！

你可以按顺序看也可以直接跳到最后的样例工程，那里会出现一些忍者。
>注意:这篇教程在我心中有特殊的地位，它的初版是我在这个站上发布的第一篇教程，使用Cocos2D与Objective-C所编写的游戏。哎呀嘛呀，时过境迁啊！

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

Sprite Kit由场景的概念所组成，就像一个游戏中显示的层级与视图。比如说你可能有一个主要游戏区域的场景与一个拥有层级的世界地图场景。场景与一个拥有层级的世界地图场景。
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
首先下载这个功能的资源文件拖进你的Xcode工程中。确保Copy items into destination group’s folder (if needed)这一项被选中，并且是在你SpriteKitSimpleGame target被选中的情况下。

下面打开GameScene.swift用如下代码替换内容:
~~~~
import SpriteKit
 
class GameScene: SKScene {
 
  // 1
  let player = SKSpriteNode(imageNamed: "player")
 
  override func didMoveToView(view: SKView) {
    // 2
    backgroundColor = SKColor.whiteColor()
    // 3
    player.position = CGPoint(x: size.width * 0.1, y: size.height * 0.5)
    // 4
    addChild(player)
  }
}
~~~~
让我们来逐步分析。

这里你为player(忍者)定义了一个私有常量，一个精灵的例子。如你所见创建一个精灵挺简单的，输入需要使用的图片名即可。

在Sprite Kit中设置场景的背景色如同设置其他背景色一样简单，这个你将其设为白色。

你将精灵定位在竖直方向10%的高度，水平中心处。

为了让精灵显示在场景中，你必须将其作为一个子项添加进场景中，与添加其它子视图的方法类似。

构建并运行下。女士们先生们，忍者闯进来了！
![](http://www.raywenderlich.com/wp-content/uploads/2015/10/005_Ninja-480x270.png)

##移动怪物
接下来你要在场景中添加一些怪物与你的忍者战斗。为了使事情变得有趣你需要移动这些怪物，否则的话就太简单了！让我来在屏幕右侧创建怪物，然后设定一个动作告诉它们向左移动。

在GameScene.swift中添加如下方法:
~~~~
func random() -> CGFloat {
  return CGFloat(Float(arc4random()) / 0xFFFFFFFF)
}
 
func random(min min: CGFloat, max: CGFloat) -> CGFloat {
  return random() * (max - min) + min
}
 
func addMonster() {
 
  // Create sprite
  let monster = SKSpriteNode(imageNamed: "monster")
 
  // Determine where to spawn the monster along the Y axis
  let actualY = random(min: monster.size.height/2, max: size.height - monster.size.height/2)
 
  // Position the monster slightly off-screen along the right edge,
  // and along a random position along the Y axis as calculated above
  monster.position = CGPoint(x: size.width + monster.size.width/2, y: actualY)
 
  // Add the monster to the scene
  addChild(monster)
 
  // Determine speed of the monster
  let actualDuration = random(min: CGFloat(2.0), max: CGFloat(4.0))
 
  // Create the actions
  let actionMove = SKAction.moveTo(CGPoint(x: -monster.size.width/2, y: actualY), duration: NSTimeInterval(actualDuration))
  let actionMoveDone = SKAction.removeFromParent()
  monster.runAction(SKAction.sequence([actionMove, actionMoveDone]))
 
}
~~~~
这里我会啰嗦一点，为了让事情更好理解。在第一部分应该将场景做成我们之前讨论的那样，对于你想创建对象的位置做一些计算，设置对象的位置，然后将其添加到场景中，像之前添加player那样。

这里的新要素就是添加动作。Sprite Kit提供了大量的内置动作，可以帮助你随时改变精灵的状态，比如移动动作、旋转动作、渐变动作、动画动作等等。这里你可以看到怪物的三种动作:
* **SKAction.moveTo(_:duration:)**: 使用这个动作使对象从屏幕外移动到左边，注意你可以设置移动的持续时间，你可以随机改变移动速度。
* **SKAction.removeFromParent()**: Sprite Kit有一个方便的动作就是从父类中移除一个节点，从场景中删除。当场景中的怪物不可见时你会使用这个动作去移除它，这时很重要的因为否则你会一直添加怪物直到消耗掉所有内存资源。
* **SKAction.sequence(_:)**: 队列动作允许你添加一个动作队列，依次执行。这样你就可以先执行"移动"动作，完成后执行"从父类移除"动作。

>注意:这块代码中包含了一些辅助方法使用arc4random()产生一个范围内的随机数，满足了在这个游戏中的随机数产生需求，如果你想将这块更完善的话看看iOS 9中新的GameplayKit的随机数API。

还差最后一件事，你需要调用创建怪物的方法！为了好玩，我们来不断地一直产生怪物。

在didMoveToView()的最后添加如下代码:
~~~~
runAction(SKAction.repeatActionForever(
  SKAction.sequence([
    SKAction.runBlock(addMonster),
    SKAction.waitForDuration(1.0)
  ])
))
~~~~
这里你运行了一个动作队列，调用了一块block代码，然后等待一秒。一直重复执行这个队列。

搞定了！构建并运行工程，现在你应该会看到一些怪物在屏幕上活蹦乱跳了:
![](http://www.raywenderlich.com/wp-content/uploads/2015/10/006_Monsters-480x270.png)

##射出子弹
在这里需要忍者作出一些动作 - 来添加射击功能吧！可以使用多种方式来实现射击，在这个游戏中需要当用户点击屏幕时进行射击动作，在点击的方向射出一个子弹。

我想用一个“移动”动作来实现这个功能，在这之前需要你做一些数学上的工作。

这是因为“移动”动作需要你提供子弹行动的终点，你不能仅仅利用触摸点因为触摸点只表达了player射击时的方向。实际上你想实现的是让子弹移动时通过触摸点直到飞出屏幕。

下面这张图描绘了这个问题:
![](https://cdn5.raywenderlich.com/wp-content/uploads/2010/02/ProjectileTriangle.jpg)

如你所见，针对触摸点与原始点之前有一个在两个方向上偏移x与y的小三角形。你需要使用一个相似的大三角形，这个三角形的一个顶点在屏幕边界上。

如果你有一些基本数学向量的方法（不如添加与减去向量）来调用的话对计算来说会很有帮助。然后Sprite Kit默认没有这些方法你需要自己去编写。

幸运的是由于方便的Swift操作符加载这些也是很容易实现的。在你的文件头添加这些函数，在GameScene类之前:
~~~~
func + (left: CGPoint, right: CGPoint) -> CGPoint {
  return CGPoint(x: left.x + right.x, y: left.y + right.y)
}
 
func - (left: CGPoint, right: CGPoint) -> CGPoint {
  return CGPoint(x: left.x - right.x, y: left.y - right.y)
}
 
func * (point: CGPoint, scalar: CGFloat) -> CGPoint {
  return CGPoint(x: point.x * scalar, y: point.y * scalar)
}
 
func / (point: CGPoint, scalar: CGFloat) -> CGPoint {
  return CGPoint(x: point.x / scalar, y: point.y / scalar)
}
 
#if !(arch(x86_64) || arch(arm64))
func sqrt(a: CGFloat) -> CGFloat {
  return CGFloat(sqrtf(Float(a)))
}
#endif
 
extension CGPoint {
  func length() -> CGFloat {
    return sqrt(x*x + y*y)
  }
 
  func normalized() -> CGPoint {
    return self / length()
  }
}
~~~~
这些是典型数学向量函数的实现。如果你对向量不太熟悉，很困惑发生了什么，来看看数学向量的[快速入门](http://www.mathsisfun.com/algebra/vectors.html)。

下一步在文件中添加一个新方法:
~~~~
override func touchesEnded(touches: Set<UITouch>, withEvent event: UIEvent?) {
 
  // 1 - Choose one of the touches to work with
  guard let touch = touches.first else {
    return
  }
  let touchLocation = touch.locationInNode(self)
 
  // 2 - Set up initial location of projectile
  let projectile = SKSpriteNode(imageNamed: "projectile")
  projectile.position = player.position
 
  // 3 - Determine offset of location to projectile
  let offset = touchLocation - projectile.position
 
  // 4 - Bail out if you are shooting down or backwards
  if (offset.x < 0) { return }
 
  // 5 - OK to add now - you've double checked position
  addChild(projectile)
 
  // 6 - Get the direction of where to shoot
  let direction = offset.normalized()
 
  // 7 - Make it shoot far enough to be guaranteed off screen
  let shootAmount = direction * 1000
 
  // 8 - Add the shoot amount to the current position
  let realDest = shootAmount + projectile.position
 
  // 9 - Create the actions
  let actionMove = SKAction.moveTo(realDest, duration: 2.0)
  let actionMoveDone = SKAction.removeFromParent()
  projectile.runAction(SKAction.sequence([actionMove, actionMoveDone]))
 
}
~~~~
这儿包含了很多东西，让我们来逐步分析:

1. SpriteKit中一个很酷的事情就是它有一些UITouch的方法，例如locationInNode(_:)与previousLocationInNode(_:)，使你可以通过SKNode的坐标系统获得一个触摸点的坐标。在这里你使用它得到屏幕坐标系统中触摸点的位置。
2. 然后你在player起始位置创建了一个子弹，注意你还没有添加到场景中，因为需要先做一个理智的判断 - 这个游戏不允许忍者向身后射击。
3. 通过计算触摸点位置与位置的差值得到一个从当前位置到触摸点位置的向量。
4. 如果X值小于0则意味着player在向后射击，这是游戏里所不允许的(真正的忍者不会向后看！)，所以直接return跳出。
5. 否则就可以将子弹添加到视图中了。
6. 调用normalized()使offset归一化，这样方便构成一个相同方向上的向量，因为 1 * length = length。
7. 将你想要的射击方向的向量乘以1000，为何是1000?这个数值足够它飞出屏幕边界:)
8. 将shoot amount添加到当前位置以得到它在屏幕上运动的终点。
9. 最后，像之前那样创建moveTo(_:, duration:)与removeFromParent()动作。

构建并运行，现在你的忍者就可以对迎面而来的部队进行连续射击了！

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/10/007_Shoot-480x270.png)

##碰撞检测与物理效果:概述

现在你可以将飞镖扔到任何地方，不过你的忍者应该想要击落一些东西，所以让我们来加一些代码来检测子弹与目标的碰撞吧。

Sprite Kit有一个优点就是自带物理引擎，不仅擅长模拟真实运动，也擅长碰撞检测。

让我们来在游戏中设置Sprite Kit的物理引擎来决定当怪物与子弹碰撞时的效果，在顶层的代码中你需要做:

* 设置物理场。一个物理场(physics world)就是进行物理运算模拟的地方。场景中默认设置了一个，你可以想要设置一个新属性，比方说重力。
* 为每个精灵创建物理体。在Sprite Kit中，你可以为每个精灵分配一个形状来进行碰撞检测，并设置属性，这就叫做一个物理体(physics body)。注意物理体不需要是与精灵完全相同的形状。一般来说为了方便起见选取一个近似的形状而不是每个像素都一样的形状，对于大多数游戏与移动这样做足够了。
* 为每个精灵类型设置类别。类别(category)是你可以设置的一个物理体属性，它用一个掩码来表示所属的群组。在这个游戏中，你需要有两个category，子弹的和怪物的。当两个物理体碰撞时，通过类别你可以知道需要处理的是哪类精灵。
* 设置一个判断接触的委托(contact delegate)方法来监测两个物理体是否碰撞。需要写一些代码去检测物体的类别，如果它们是怪物与子弹就让它们爆炸！

现在你了解了战斗计划，是时候添加进动作了！

##碰撞检测与物理效果:实现
在GameScene.swift的头部添加如下结构体:
~~~~
struct PhysicsCategory {
  static let None      : UInt32 = 0
  static let All       : UInt32 = UInt32.max
  static let Monster   : UInt32 = 0b1       // 1
  static let Projectile: UInt32 = 0b10      // 2
}
~~~~
设置一些物理类型的常量。

>注意:你也许奇怪这些语句有啥用。注意Sprite Kit中的类别仅仅是一个32位的整型数，相当于掩码。用32位整型数表示类别是一个很美妙的方法。这里你设置第一位表示一个怪物，下一位表示一个子弹。

下面让GameScene实现**SKPhysicsContactDelegate**协议:
~~~~
class GameScene: SKScene, SKPhysicsContactDelegate {
~~~~
Then inside  add these lines after adding the player to the scene:
在**didMoveToView(_:)**方法中添加player到视图的代码后添加如下行:
~~~~
physicsWorld.gravity = CGVectorMake(0, 0)
physicsWorld.contactDelegate = self
~~~~
它设置了一个零重力场，并且设置了场景的委托，用来监测两个物理体的碰撞。

在**addMonster()**方法中，在创建怪物精灵的代码后面添加如下行:
~~~~
monster.physicsBody = SKPhysicsBody(rectangleOfSize: monster.size) // 1
monster.physicsBody?.dynamic = true // 2
monster.physicsBody?.categoryBitMask = PhysicsCategory.Monster // 3
monster.physicsBody?.contactTestBitMask = PhysicsCategory.Projectile // 4
monster.physicsBody?.collisionBitMask = PhysicsCategory.None // 5
~~~~
让我们来逐行分析下:

1. 为精灵创建一个物理体。在这种情况下，物理体的形状被定义为与精灵相同尺寸的矩形，因为这是个合适的怪物近似形状。
2. 设置精灵为动态(dynamic)的。这意味着物理引擎将不会控制怪物的移动，需要通过你已经写过的代码来进行（使用移动动作）。
3. 设置categoryBitMask为之前你定义的怪物类别。
4. contactTestBitMask的作用是设置当与另一个目标接触时需要提醒的目标类别，这里设置成子弹。
5. collisionBitMask的作用是设置与另一个物体接触时需要用物理引擎处理做出响应(比方说反弹)的类别。 你不会想让怪物与子弹互相反弹的，在这个游戏中可以让它们穿过彼此，因此设为空。

接下来在touchesEnded(_:withEvent:)中添加一些类似的代码，在设置子弹位置的代码后:
~~~~
projectile.physicsBody = SKPhysicsBody(circleOfRadius: projectile.size.width/2)
projectile.physicsBody?.dynamic = true
projectile.physicsBody?.categoryBitMask = PhysicsCategory.Projectile
projectile.physicsBody?.contactTestBitMask = PhysicsCategory.Monster
projectile.physicsBody?.collisionBitMask = PhysicsCategory.None
projectile.physicsBody?.usesPreciseCollisionDetection = true
~~~~
试试看，自己能否能否理解每一行代码，如果不能再去看看上面的解析吧。

接下来，添加一个当子弹与怪物碰撞时调用的方法。注意它不会自动调用，你如果在后面调用它。
~~~~
func projectileDidCollideWithMonster(projectile:SKSpriteNode, monster:SKSpriteNode) {
  print("Hit")
  projectile.removeFromParent()
  monster.removeFromParent()
}
~~~~
你要做的就是当子弹与怪物碰撞时将它们从场景中移除。挺简单的，不是吗？

现在是时候来实现接触的代理方法了，在文件中添加如下的新方法:
~~~~
func didBeginContact(contact: SKPhysicsContact) {
 
  // 1
  var firstBody: SKPhysicsBody
  var secondBody: SKPhysicsBody
  if contact.bodyA.categoryBitMask < contact.bodyB.categoryBitMask {
    firstBody = contact.bodyA
    secondBody = contact.bodyB
  } else {
    firstBody = contact.bodyB
    secondBody = contact.bodyA
  }
 
  // 2
  if ((firstBody.categoryBitMask & PhysicsCategory.Monster != 0) &&
      (secondBody.categoryBitMask & PhysicsCategory.Projectile != 0)) {
    projectileDidCollideWithMonster(firstBody.node as! SKSpriteNode, monster: secondBody.node as! SKSpriteNode)
  }
 
}
~~~~
在之前你在场景中设置了物理场的接触委托，这个方法会在两个物理体碰撞时被调用。

在这个方法中有两部分：

1. 在这个方法中你传递了两个碰撞的物理体，没有规定传递它们的顺序，因此这段代码利用它们的categoryBitMask属性来进行排序这样你就在后面做出假设。.
2. 在最后检查一下两个碰撞体是否是子弹和怪物，如果是的话调用你之前写的那个方法。
构建并运行一下，现在当你的子弹碰到目标时会一起消失！

##完成触摸

你离一个可运行(不过还是很简陋)的游戏已经很近了。需要添加一些效果与音乐(游戏总要有点声音吧！)还有一些游戏逻辑。

在项目中已经有一些现成的很酷的背景乐与一个很棒的射击音效，你只需要播放它们即可！

在didMoveToView(_:)的最后添加如下代码:
~~~~
let backgroundMusic = SKAudioNode(fileNamed: "background-music-aac.caf")
backgroundMusic.autoplayLooped = true
addChild(backgroundMusic)
~~~~
这个使用了SKAudioNode —— 一个iOS 9中的新类，在你的游戏中播放与循环背景乐。

在touchesEnded(_:withEvent:)中添加如下这行来添加音效:
~~~~
runAction(SKAction.playSoundFileNamed("pew-pew-lei.caf", waitForCompletion: false))
~~~~
非常方便，不是吗？你可以用一行来播放音效！

构建并运行，享受绝妙的旋律吧！

>注意：如果你没听到背景乐，试试在真机上测试。

##哥们！游戏结束！

现在让我们来创建一个新场景，用来显示“你赢了”或“你输了”。创建一个名为GameOverScene的新swift文件。

用如下代码替代GameOverScene.swift中的内容：
~~~~
import Foundation
import SpriteKit
 
class GameOverScene: SKScene {
 
  init(size: CGSize, won:Bool) {
 
    super.init(size: size)
 
    // 1
    backgroundColor = SKColor.whiteColor()
 
    // 2
    let message = won ? "You Won!" : "You Lose :["
 
    // 3
    let label = SKLabelNode(fontNamed: "Chalkduster")
    label.text = message
    label.fontSize = 40
    label.fontColor = SKColor.blackColor()
    label.position = CGPoint(x: size.width/2, y: size.height/2)
    addChild(label)
 
    // 4
    runAction(SKAction.sequence([
      SKAction.waitForDuration(3.0),
      SKAction.runBlock() {
        // 5
        let reveal = SKTransition.flipHorizontalWithDuration(0.5)
        let scene = GameScene(size: size)
        self.view?.presentScene(scene, transition:reveal)
      }
    ]))
 
  }
 
  // 6
  required init(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
  }
}
~~~~
这里有几个需要注意的点：

1. 设置背景色为白色，像之前你对主场景做的那样。
2. 根据won参数来设置消息是“You Won” 还是 “You Lose”（用了三目运算符）。
3. 这就是使用Sprite Kit怎样将一个标签显示到屏幕中的方法，如你所见，这是很简单的 - 你只需要选择你的字体并设定一个新的参数即可。
4. 最后设置并运行一个包含两个动作的队列。我将使用一行来向你展示它是多么的方便(而不是使用两个分离的单独动作变量)。一开始西安等待3秒，然后使用runBlock动作去执行剩下的代码。
5. 这就是如何在Sprite Kit中跳转到另一个场景。首先你要在不同的动画变换中选择出你先要显示出场景的方法 - 你选择了一个持续0.5秒的反转动画。接着你创建了想要显示的场景，在self.view属性上使用presentScene(_:transition:)方法。
6. 如果你要重载一个场景的初始化，你必须实现required init(coder:)这个初始化方法。然而这个初始化永远不会被调用，需要添加一个fatalError(_:)方法来实现。

到现在为止还算顺利，还需要设置加载游戏时的主要场景。

返回GameScene.swift中，在addMonster()方法中, 用下面的代码替换最后一行的怪物动作：
~~~~
let loseAction = SKAction.runBlock() {
  let reveal = SKTransition.flipHorizontalWithDuration(0.5)
  let gameOverScene = GameOverScene(size: self.size, won: false)
  self.view?.presentScene(gameOverScene, transition: reveal)
}
monster.runAction(SKAction.sequence([actionMove, loseAction, actionMoveDone]))
~~~~
这里创建了一个“失败动作”，当一个怪物飞出屏幕时会显示game over画面。看看你是否理解每一行，如果没有请看看之前的解析。

现在你应该处理一下胜利时的情况，不要对你的玩家太残酷！在GameScene的顶部添加一个新属性，在player的声明之后：
~~~~
var monstersDestroyed = 0
~~~~
在projectile(_:didCollideWithMonster:)上添加如下代码：
~~~~
monstersDestroyed++
if (monstersDestroyed > 30) {
  let reveal = SKTransition.flipHorizontalWithDuration(0.5)
  let gameOverScene = GameOverScene(size: self.size, won: true)
  self.view?.presentScene(gameOverScene, transition: reveal)
}
~~~~
直接运行一下，现在有了胜利与失败的条件并能在适当的时候看到game over画面了。

![](http://www.raywenderlich.com/wp-content/uploads/2015/10/008_YouWin.png)
