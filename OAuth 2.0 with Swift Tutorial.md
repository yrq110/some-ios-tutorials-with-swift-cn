#OAuth 2.0 with Swift Tutorial
##基于Swift的OAuth 2.0使用指南
[原文地址](https://www.raywenderlich.com/99431/oauth-2-with-swift-tutorial)

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/05/oauth2-epalined-4b-250x250.png)

在你将内容分享到喜爱的社交网络(Facebook, Twitter等)上或者你企业的OAuth2服务器上时，很可能不经意使用了OAuth2与其他种类的协议，你甚至不知道内部的原理。
不过你知道如何使用OAuth2连接你的服务与iOS app吗？

在这篇教程中，你会使用一个名为Incognito的分享型app来学习如何使用[AeroGear OAuth2](https://github.com/aerogear/aerogear-ios-oauth2), [AFOAuth2Manager](https://github.com/AFNetworking/AFOAuth2Manager)和[OAuthSwift](https://github.com/dongri/OAuthSwift)这些开源OAuth2库来将你的资源分享到GoogleDrive上。
##入门

[下载Incognito开始工程](http://www.raywenderlich.com/wp-content/uploads/2015/04/Incognito.aerogear_start.zip)。开始工程使用CocoaPods来安装AeroGear与其他你需要的一切，包括生成pods与xcworkspace目录。

在Xcode中打开Incognito.xcworkspace。工程基于Xcode的单视图应用模板，有包含单一ViewController.swift视图控制器的故事版，所有UI与动作在ViewController.swift中都已经处理好了。

构建并运行你的工程，会看到如下图这样：

![](http://www.raywenderlich.com/wp-content/uploads/2015/05/OAuth_App.png)

这个app可以让你选择一张自拍然后在图片上加一些装扮，你认出乔装后的我了吗 :]

>注意：在模拟器中添加照片，可以按Cmd + Shift + H回到home界面然后将你的图片拖到模拟器中。

所有需要在app中添加的东西就是使用了3个不同的OAuth2库来完成分享到Google Drive的特性。

不可能的任务? 不，没有你搞不定的:]

让我来给你讲个故事吧，而非让你看RFC6749 OAuth2的规范以至于太无聊。

##讲讲为何需要OAuth2
周一的清晨，我们的开发小哥Bob在咖啡机前碰到了友好的极客Alice。Bob拿着一大摞文件，好像挺忙的样子，他的老板想让他研究一下OAuth2规范并加入到Incognito应用中。

当然，如果你把俩开发者放到一个咖啡厅中他们会开始聊聊比较极客的话题。Bob问Alice：

“我们要用OAuth2解决哪些问题？”

![](https://cdn1.raywenderlich.com/wp-content/uploads/2015/03/oauth2-explained-1.png)

一方面，你有表格中API的服务器，比如使用Twitter的API你可以得到跟随者或推文的列表。这些API处理你的在登录与密码的保护下的加密数据。

另一方面，你有使用服务器的app。那些app会访问你的数据，不过你会信任所有拥有你证书的服务吗？也许会，也许不会。

这就带来了委托访问的概念。OAuth2让用户授权第三方的app在不提供密码的情况下通过称为访问令牌的安全措施访问它们的web资源。访问令牌中不会包含密码信息，这意味着你的密码安全的存放在主服务器中，每一个想要连接服务器的app都会得到它们的专用令牌。之后如果你想撤销app的访问的话可以撤销令牌。

OAuth2有以下四个要素：
* 授权服务器: 响应认证与授权，提供访问令牌。
* 资源服务器: 对有效的令牌提供资源服务有效
* 资源用户: 数据的拥有者 - Incognito的终端用户
* 客户端: Incognito移动端app
