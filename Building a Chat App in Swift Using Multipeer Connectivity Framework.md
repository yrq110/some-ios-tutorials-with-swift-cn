#Building a Chat App in Swift Using Multipeer Connectivity Framework
##使用Multipeer Connectivity框架构建一个聊天app

[原文地址](http://www.appcoda.com/chat-app-swift-tutorial/)

Multipeer Connectivity-多点连接

在iOS编程中，总有一些SDK相比其它而言更有意思更吸引开发者的注意，Multipeer Connectivity就是其中一个。如你所知，MPC框架并不是iOS8中的新东西，
在7中就出现了。之前我写过一些相关的教程，不过说实话，我很惊讶大家对它如此的感兴趣。现在过了一段时间后，又写了一篇新的教程，相信这其中还
有一些东西需要说明下。

你也许会疑惑为何会说这个有些古老的话题，而不是iOS8中的新特性。好吧，我这么做有三个原因：

1. 很多读者都通过邮件告诉我想知道在multipeer connectivity框架中如何处理多种不同的任务，在回答这些问题时，我发现了一些以前没注意到的问题，而且有些是不容易发现的。
2. 在之前的教程中我使用了一个默认的、创建好的视图控制器来实现用户的邀请与建立连接。读者反馈想要看看手动的实现，这也是接下来要做的。
3. 我觉得使用Swift来实现MPC是很有用的，对初学者也很有帮助。

![](http://www.appcoda.com/wp-content/uploads/2015/01/mpc-swift-featured.jpg)

自multipeer connectivity诞生以来，为大量开发者提供了很多实现新点子的可能性。使用一种简单的方式去进行设备间的连接是很诱人的，这也是为何开发者会乐于将它集成在app中。不过如果你还没用过MPC框架的话，我必须提醒你：它有时不是你所期望的那么稳定、健壮，我在我的项目中发现过类似问题，有些开发者也告知过我同样的问题。MPC使用蓝牙和WiFi来连接附近的设备，虽然这听起来很棒很有前景，但是有时由于通信过程中的问题，连接也会失败或变得很慢。若你要传输重要数据的话，这种情况是需要重点考虑的。我建议你使用一个备用的通信解决方案(像一个web服务)，确保有一个备选途径，当MPC失效时app也能继续工作。尽管我这么说，我依然相信MPC是一个对所有iOS开发者都很不错的工具，值得俺再写一篇教程。

我不打算深入MPC的细节中。如果想尝尝鲜的话，看看这篇[教程]()。否则的话，看看我总结的这个MPC的概述。

MPC中包含4个重要的类，分别对应4个概念:

1. Peer-节点 (MCPeerID): 一个节点实则就是一台设备，这通常是需要优先设置的，因为它会作为下一个类实例初始化时的一个参数使用。它包含一个重要的属性 - displayName，这是一台设备对于附近的节点所显示的名字。
2. Session-会话 (MCSession class): 这是两个节点间建立的连接，前提是第一个节点邀请了第二个，并且第二个接受了邀请。一个会话只与两台设备有关。一个第三方设备不能连接进一个已存在的会话。
3. Browser-搜索器 (MCNearbyServiceBrowser class): 这个类的方法被用来寻找附近的设备并邀请他们加入一个会话。作为前提条件，其他设备必须先广播他们自己。在这篇教程中用使用这个类去手动邀请其他设备。
4. Advertiser-广播器 (MCNearbyServiceAdvertiser): 这个类负责广播一个设备，控制这个设备对其他设备可见或不可见，接受或拒绝其他节点的会话邀请。


MPC的逻辑是很简单的:  一台设备(一个节点)使用它的搜索器，寻找周围的其他设备。一台设备必须广播它自己，否则不会被发现。一旦它在搜索时发现了一个或多个节点，就会向其发送一个会话连接的邀请。这个邀请可以在发现周围节点时自动发送，也可以用户手动决定是否发送，这完全取决于应用中国是如何实现的。无论任何情况，只要邀请被接受，会话就会建立，两个节点间就可以收发数据、资源与流。

本文中我的目标是给你演示如果编程实现搜索、邀请与连接其它节点。记住，之后看到的只是其中一种代码实现的方式，很显然，你可以通过使用MPC中提供的所有工具，以你自己想要的方式去实现适合你app的功能。下面的内容只是一个MPC框架实现的例子，实现了精简的从头到尾的编程过程，没有使用任何额外的SDK。我希望在这篇教程最后你们能找到一些所寻找的答案。

最后，如果你从未使用过MPC框架，我建议你扫一眼苹果的官方文档与我之前的教程。哦了不浪费时间了，开始编程吧。

##关于Demo App

来稍微说下demo应用，一个聊天app。好吧这与之前MPC教程中的聊天app有很大的共同之处，不过让我解释一下:不管我怎么想，都找不到一个比这个更合适的类型，聊天app一直会是首选，我相信在教程的最后你会认同我的看法的。

开始工程在[这里](https://www.dropbox.com/s/adbuyk2j1vgmbmd/MPCRevisitedTemplate.zip?dl=0)下载，将在这个的基础上进行编程，下好后先大致浏览下，以便能快速的上手。

如我所讲，将构建一个聊天app，这个app分为两个视图控制器，第二个会呈现给用户。如果你看看工程中的Interface Builder会发现第一个视图控制器中有一个列表视图，在这个列表视图中将要显示的是MPC框架所搜索到的所有附近的设备，列表中的节点是动态刷新的，消失的已存在节点与新发现的节点都会动态显示在列表中。这里还有一个包含一个按钮的工具栏，后面会使用这个按钮去打开和关闭设备的可发现性，针对MPC中广播的特性所设计。

在点击一个列表项节点时，会询问另一端的用户是否开始聊天。如果他拒绝则什么都不会发生，如果接受的话会在屏幕中显示第二个视图控制器，名为ChatViewController，那么两台设备就可以交换文本信息了。这里包含一个用来撰写信息的textfield和一个显示所有信息的列表视图。在顶部工具栏的按钮用来终止聊天。差不多就是所有细节了，来看看具体的实现吧。

在看app的最终效果图之前，我还要说一件事。尽管我们可以用MPC连接7个甚至更多的设备，在这里只连接一个节点进行多次测试，因为使用会话来连接节点与发现和列出所有附近节点是两码事。在下面的部分你会看到逐一实现的步骤，现在先感受一下吧:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_2_ask_chat_alert.png)

##一个自定义类

知道这个工程是做什么的后让我们来创建一个自定义类，在这个类中会处理所有需要实现的MPC框架方法，在最后我们会创建一个使用代理模式的新协议。

当务之急，在Xocde中按住Cmd与N组合键，在出现的引导视图中，选择创建一个新的Cocoa Touch Class:

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_4_class_template_1.png)

第二步中，使新的类继承自NSObject类，命名为MPCManager

![](http://www.appcoda.com/wp-content/uploads/2015/01/t27_5_class_template_2.png)

跟着向导走，确保你正在操作的是MPCManager.swift文件。

在代码的第一行，需要导入multipeer connectivity框架，因此在文件头部添加如下代码:
````swift
import MultipeerConnectivity
````
接着来声明需要用到的MPC框架中的类的对象，在类中的顶部添加如下行:
````swift
var session: MCSession!
 
var peer: MCPeerID!
 
var browser: MCNearbyServiceBrowser!
 
var advertiser: MCNearbyServiceAdvertiser!
````
除了上面那些，还需要声明两个以后要用到的变量:
````swift
var foundPeers = [MCPeerID]()
 
var invitationHandler: ((Bool, MCSession!)->Void)!
````
在foundPeers数组中会存放所有发现的节点。注意此时与这些节点间不会构建任何连接，只需要知道周围有这些节点。这个数组在声明的同时进行了初始化，因此不需要额外设置为nil，期望数组在发现新对象时已经准备就绪。

invitationHandler是一个完整声明的处理程序，不过现在不会管它，等用的时候再说。

接下来需要使自定义类满足特定的MPC协议。这些协议的委托方法使我们可以去处理multipeer connectivity的相关操作，比如搜索、广播、会话等。如下修改类的第一行，添加协议:
````swift
class MPCManager: NSObject, MCSessionDelegate, MCNearbyServiceBrowserDelegate, MCNearbyServiceAdvertiserDelegate
````
我觉得没必要解释每个协议的用途了。

现在来创建一个初始化器来初始化所有的MPC对象。一个一个来，从peer对象开始，它代表了，在初始化时需要提供一个显示的名称(display name)，这个显示名称将会对其他节点可见，并且可以设置为任意字符串。为了使事情简单点，我直接将设备名称作为显示名称，不过我不建议在一个实际的app中这样做，也许你应该让用户自己来输入想要的名字，或者使用其他方法来生成唯一的节点名称。在代码中，如下进行初始化:
````swift
override init() {
    super.init()
 
    peer = MCPeerID(displayName: UIDevice.currentDevice().name)
}
````
通过UIDevice类获得设备名称。

在peer对象成功初始化后，接着进行其他对象的操作。注意peer必须最先被初始化，因为之后的对象都需要使用它。session对象:
````swift
override init() {
    ...
 
    session = MCSession(peer: peer)
    session.delegate = self
}
````
如你所见，一个session的初始化只需要一个参数，就是之前的peer。并且在初始化过程中将session对象委托给了当前类。

接着是搜索器对象:
````swift
override init() {
    ...
 
    browser = MCNearbyServiceBrowser(peer: peer, serviceType: "appcoda-mpc")
    browser.delegate = self
}
````
这个对象的初始化接受两个参数：第一个是peer. 第二个是一个在初始化后无法改变的值，指明了搜索器能搜索到的服务类型(service type)。简单地讲，它用于在大量节点中的准确识别，这样MPC就知道需要搜索什么了，而广播器也必须设置成相同的服务类型(一会儿你会见到)。在设置这个值时需要遵守两条规则：(a)不能超过15个字符(b)只能包含小写的ASCII字符、数字与连字符。如果你不符合这个规则会跳出一个runtime的异常并且app将会崩溃。

对于广播器:
````swift
override init() {
    ...
 
    advertiser = MCNearbyServiceAdvertiser(peer: peer, discoveryInfo: nil, serviceType: "appcoda-mpc")
    advertiser.delegate = self
}
````
注意，在这里设置与之前相同的服务类型，还有一个叫discoveryInfo的参数，这个参数是一个字典，当你想给发现的其他节点发送额外信息时所需要设置的，注意，这个字典的键值都必须是字符串类型。方便起见，将这个参数设为nil。

初始化方法准备好了，这里是完整的代码:
````swift
override init() {
    super.init()
 
    peer = MCPeerID(displayName: UIDevice.currentDevice().name)
 
    session = MCSession(peer: peer)
    session.delegate = self
 
    browser = MCNearbyServiceBrowser(peer: peer, serviceType: "appcoda-mpc")
    browser.delegate = self
 
    advertiser = MCNearbyServiceAdvertiser(peer: peer, discoveryInfo: nil, serviceType: "appcoda-mpc")
    advertiser.delegate = self
}
````
在结束这部分之前，来创建一个实现委托模式的协议。注意我们会声明当前在app的开发过程中所需要的所有委托方法，之后将不会再处理这个协议，直到需要它们的时候。

在自定义类的上方添加如下代码段:
````swift
protocol MPCManagerDelegate {
    func foundPeer()
 
    func lostPeer()
 
    func invitationWasReceived(fromPeer: String)
 
    func connectedWithPeer(peerID: MCPeerID)
}
````
接着会讨论上面每一个函数的作用，在下一个部分中会详细介绍。

最后，在MPCManager类中声明一个delegate对象:
````swift
var delegate: MPCManagerDelegate?
````
目前为止成功完成了项目的第一个重要部分，可以歇歇脚了。我想说下，现在在Xcode中提示的错误是正常情况，在实现MPC的委托方法后就会消失了。

##搜索节点

