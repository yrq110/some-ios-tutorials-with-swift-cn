#Building a Chat App in Swift Using Multipeer Connectivity Framework
##使用Multipeer Connectivity框架构建一个聊天app

Multipeer Connectivity-多点连接

在iOS编程中，总有一些SDK相比其它而言更有意思更吸引开发者的注意，Multipeer Connectivity就是其中一个。如你所知，MPC框架并不是iOS8中的新东西，
在7中就出现了。之前我写过一些相关的教程，不过说实话，我很惊讶大家对它如此的感兴趣。现在过了一段时间后，又写了一篇新的教程，相信这其中还
有一些东西需要说明下。

你也许会疑惑为何会说这个有些古老的话题，而不是iOS8中的新特性。好吧，我这么做有三个原因：

1. 很多读者都通过邮件告诉我想知道在multipeer connectivity框架中如何处理多种不同的任务，在回答这些问题时，我发现了一些以前没注意到的问题，而且有些是不容易发现的。
2. 在之前的教程中我使用了一个默认的、创建好的视图控制器来实现用户的邀请与建立连接。读者反馈想要看看手动的实现，这也是接下来要做的。
3. 我觉得使用Swift来实现MPC是很有用的，对初学者也很有帮助。

![](http://www.appcoda.com/wp-content/uploads/2015/01/mpc-swift-featured.jpg)

