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

这就带来了委托访问的概念。OAuth2让用户授权第三方的app在不提供密码的情况下通过安全令牌来访问web资源。令牌中不会包含密码信息，这意味着你的密码安全的存放在主服务器中，每一个想要连接服务器的app都会得到专用令牌。之后如果你想撤销app访问的话可以撤销令牌。

OAuth2有以下四个要素：
* 授权服务器: 响应认证与授权，提供访问令牌
* 资源服务器: 对有效的令牌提供资源服务
* 资源用户: 数据的拥有者 - Incognito的终端用户
* 客户端: Incognito移动端app

OAuth2规范使用授权流(grant flows)来描述交互行为。

规范中定义了四种不同的授权流，可以被归为两类：

* 3-legged流：这种情况下终端用户需要获得授权权限。简化授权是为了那些不方便存储安全令牌的使用浏览器的app。授权码授权，会生成一个访问令牌与随机生成的令牌，使客户端能安全的保存令牌。在这样的移动app的客户端中会有一个安全存放令牌的地方，类似iOS中的钥匙串。
* 2-legged流: 凭证会直接发给app。不同之处在于资源用户直接在客户端中输入了凭证，比如说你作为一个开发者在访问像Parse等的API时会将密钥存放在你的app中。

在这篇教程中会使用已有的Google Drive账号，并上传Incognito自拍。这种情况下使用3-legged认证比较好。

##授权之舞
虽然使用开源库时会隐藏那些OAuth2协议中棘手的细节，不过了解一下基本的工作原理会有助于你更好的进行配置。

这里是授权认证之舞的步骤：

###Step 0: 注册
app需要注册需要访问的服务。对你来说，Incognito需要注册Google Drive。别担心，之后会介绍具体怎么做。
###Step 1: 授权
舞蹈开始时Incognito会给第三方服务端发一个请求的授权码，包含以下信息：
* 客户端ID: 在注册服务阶段提供。定义与服务会话的app。
* 客户端密钥: 在注册服务阶段提供。一个服务与app之前的密钥，打包进了app的二进制文件。
* 重定向URI: 在用户使用证书访问服务后重定向到的位置，并授权许可。
* 作用域: 用来告诉服务app所拥有的许可级别。

app之后会切换到浏览器界面。一旦用户登入，则Google授权服务器会显示一个授权页: “Incognito想要访问你的相册：允许/拒绝”，当终端用户点击“允许”时，服务器会使用重定向URI跳转回Incognito中，并给app发送一个授权码。
![](https://cdn2.raywenderlich.com/wp-content/uploads/2015/05/oauth2-explained-3.png)

###Step 2: 交换授权码与令牌
授权码是临时的，因此OAuth2库需要用临时码交换得到一个合适的访问令牌，是随机生成的。
###Step 3: 取得资源
使用访问令牌，Incognito可以访问服务器上受保护的资源-用户所授权访问的资源，可以自由的上传图片。

准备好实践了吗？首先，你需要注册OAuth2的提供商-Google。

##注册OAuth2提供商
如果你没有Google账户，现在就去创建一个吧。我们会等你:]

使用你的浏览器打开http://console.developer.google.com 进行操作

点击创建工程并起名为Incognito：
![](https://cdn2.raywenderlich.com/wp-content/uploads/2015/04/RW_OAuth_CreateProject1-480x292.png)

接着，你需要启用API。

进入APIs&auth\APIs界面，然后点击Google Apps APIs\Drive API。在接下来的视图中点击启用API：
![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/04/RW_Oauth_EnableAPI1-437x320.png)

现在你需要创建一个新的证书来访问app中的账户。

进入APIs&auth\Credentials界面点击创建新的客户端ID按钮，点击同意在出现的视图中填写以下信息：
* 邮箱: 选择你的邮箱地址
* 项目名称: Incognito
* 主页: http://www.raywenderlich.com

点击保存，回到客户端ID界面。

选择安装的应用，然后选择iOS输入com.raywenderlich.Incognito作为你的Bundle ID。

授权服务器将会使用这个bundle id作为重定向URI进入app。

最后点击创建客户端ID，最后的界面会显示生成的客户端ID、客户端密钥与重定向URI：
![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/04/RW_OAuth_Tokens-571x500.png)

现在成功注册了Google服务，可以开始使用第一个OAuth2库来实现OAuth2了：依赖外部浏览器的AeroGear。

##使用AeroGear与外部浏览器授权
打开ViewController.swift在文件头部添加如下代码：
````swift
import AeroGearHttp
import AeroGearOAuth2
````
现在，在ViewController类中添加实例变量：
````swift
var http: Http!
````
在viewDidLoad()中初始化它：
````swift
self.http = Http()
````
使用AerGearHttp库中的这个Http的实例来执行HTTP的请求。

在ViewController.swift中找到空的share(:)方法并添加如下代码：
````swift
let googleConfig = GoogleConfig(
clientId: "YOUR_GOOGLE_CLIENT_ID",                               // [1] Define a Google configuration
scopes:["https://www.googleapis.com/auth/drive"])                // [2] Specify scope
 
let gdModule = AccountManager.addGoogleAccount(googleConfig)     // [3] Add it to AccountManager
self.http.authzModule = gdModule                                 // [4] Inject the AuthzModule 
                                                                 // into the HTTP layer object 
 
let multipartData = MultiPartData(data: self.snapshot(),         // [5] Define multi-part 
          name: "image",
          filename: "incognito_photo",
          mimeType: "image/jpg")
let multipartArray =  ["file": multipartData]
 
self.http.POST("https://www.googleapis.com/upload/drive/v2/files",   // [6] Upload image
               parameters: multipartArray,
               completionHandler: {(response, error) in
  if (error != nil) {
    self.presentAlert("Error", message: error!.localizedDescription)
  } else {
    self.presentAlert("Success", message: "Successfully uploaded!")
  }
})
````
上面的方法进行了如下操作：
1. 需要使用你的Google控制台中的客户端ID来替换上面代码中的YOUR_GOOGLE_CLIENT_ID，进行正确的配置。
2. 定义了授权请求的域名，在Incognito中你需要访问drive API。
3. 使用AccountManager的方法进行OAuth模块的初始化。
4. 接着将OAuth2模块导入连接着授权模块的HTTP对象中。
5. 创建一个包含多种数据的对象来封装你想发送给服务器的信息。
6. 最后调用一个简单的HTTP请求上传图片，POST()会检查OAuth2模块是否加进了HTTP中，并产生一个回调：
* 若没有访问令牌存在则开始授权码的授权
* 若需要则重新生成访问令牌
* 若所有令牌都可用，则仅仅调用POST指令

>注意：想了解有关AeroGear OAuth2的信息的话可以看看AeroGear的[在线文档](https://aerogear.org/docs/guides/aerogear-ios-2.X/Authorization/)与[API手册](https://aerogear.org/docs/specs/aerogear-ios-oauth2/)，或者看看Pods中的源代码。
