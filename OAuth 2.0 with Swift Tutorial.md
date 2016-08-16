#OAuth 2.0 with Swift Tutorial
##Swift的OAuth 2.0使用指南

***

>* 原文链接 : [OAuth 2.0 with Swift Tutorial](https://www.raywenderlich.com/99431/oauth-2-with-swift-tutorial)
* 原文作者 : [Corinne Krych](https://www.raywenderlich.com/u/ckrych)
* 译者 : [yrq110](https://github.com/yrq110)

***

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/05/oauth2-epalined-4b-250x250.png)

在你将内容分享到喜爱的社交网络(Facebook, Twitter等)上或者你企业的OAuth2服务器上时，很可能不经意使用了OAuth2与其他种类的协议，你甚至不知道内部的原理。
不过你知道如何使用OAuth2连接你的服务与iOS app吗？

在这篇教程中，你会使用一个名为Incognito的分享型app来学习如何使用[AeroGear OAuth2](https://github.com/aerogear/aerogear-ios-oauth2), [AFOAuth2Manager](https://github.com/AFNetworking/AFOAuth2Manager)和[OAuthSwift](https://github.com/dongri/OAuthSwift)这些开源OAuth2库来将你的资源分享到GoogleDrive上。

**目录**
* [入门](#入门)
* [讲讲为何需要OAuth2](#why-need)
* [授权之舞](#授权之舞)
  * [Step 0: 注册](#step-0)
  * [Step 1: 授权](#step-1)
  * [Step 2: 交换授权码与令牌](#step-2)
  * [Step 3: 取得资源](#step-3)
* [注册OAuth2提供商](#regis)
* [使用AeroGear与外部浏览器授权](#aerogear)
* [配置URL Scheme](#url-scheme)
* [使用嵌入的Web视图](#web-view)
* [使用嵌入web视图的OAuthSwift](#oauthswift)
* [配置URL](#con-url)
* [显示UIWebView](#display-webview)
* [使用AFOAuth2Manager授权](#afoauthma)

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

<a name="why-need"></a>
##讲讲为何需要OAuth2
周一的清晨，我们的开发小哥Bob在咖啡机前碰到了友好的极客Alice。Bob拿着一大摞文件，好像挺忙的样子，他的老板想让他研究一下OAuth2规范并加入到Incognito应用中。

当然，如果你把俩开发者放到一个咖啡厅中他们会开始聊聊比较极客的话题。Bob问Alice：

“我们要用OAuth2解决哪些问题？”

![](https://cdn1.raywenderlich.com/wp-content/uploads/2015/03/oauth2-explained-1.png)

一方面，你可以使用API形式的服务，比如使用Twitter的API你可以得到跟随者或推文的列表。这些API处理你的在登录与密码的保护下的加密数据。

另一方面，你有使用这些服务的app。那些app会访问你的数据，不过你会信任所有拥有你证书的服务吗？也许会，也许不会。

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

<a name="step-0"></a>
###Step 0: 注册
app需要注册需要访问的服务。对你来说，Incognito需要注册Google Drive。别担心，之后会介绍具体怎么做。

<a name="step-1"></a>
###Step 1: 授权
舞蹈开始时Incognito会给第三方服务端发一个请求的授权码，包含以下信息：
* 客户端ID: 在注册服务阶段提供。定义与服务会话的app。
* 客户端密钥: 在注册服务阶段提供。一个服务与app之前的密钥，打包进了app的二进制文件。
* 重定向URI: 在用户使用证书访问服务后重定向到的位置，并授权许可。
* 作用域: 用来告诉服务app所拥有的许可级别。

app之后会切换到浏览器界面。一旦用户登入，则Google授权服务器会显示一个授权页: “Incognito想要访问你的相册：允许/拒绝”，当终端用户点击“允许”时，服务器会使用重定向URI跳转回Incognito中，并给app发送一个授权码。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2015/05/oauth2-explained-3.png)

<a name="step-2"></a>
###Step 2: 交换授权码与令牌
授权码是临时的，因此OAuth2库需要用临时码交换得到一个合适的访问令牌，是随机生成的。

<a name="step-3"></a>
###Step 3: 取得资源
使用访问令牌，Incognito可以访问服务器上受保护的资源-用户所授权访问的资源，可以自由的上传图片。

准备好实践了吗？首先，你需要注册OAuth2的提供商-Google。

<a name="regis"></a>
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

<a name="aerogear"></a>
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

构建并运行app，选择一张图片，添加一些覆盖物，然后点击分享按钮，会提示你输入Google证书，如果你已经登录过了，你的整数会存放在缓存中。重定向到认证界面，点击接受，然后...

Boom沙卡拉卡 你收到了一条Safari无法打开页面的错误信息。发生了什么？

![](https://cdn5.raywenderlich.com/wp-content/uploads/2015/05/iOS-Simulator-Screen-Shot-21-May-2015-21.48.49-281x500.png)

当你点击接受后，Google OAuth站点会重定向到com.raywenderlich.Incognito://[some url]。因此需要设置app使其能打开这个
URL scheme。

>注意：Safari将你的授权响应信息存放在了模拟器的cookie中，所以别立即就重新授权，为了清空模拟器的这些cookie需要重置iOS模拟器的内容与设置。

<a name="url-scheme"></a>
##配置URL Scheme
允许你的用户重定向回到Incognito中，你需要自定义一个关联的URL scheme。
在Info.plist.文件中点击右键选择Open As\Source Code，在plist的底部，</dict>标签的右边加入如下代码：
````
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>com.raywenderlich.Incognito</string>
        </array>
    </dict>
</array>
````
scheme是一个URL中的第一个部分，在网页中的scheme通常是http或https。iOS app可以自定义它们自己的 URL schemes，比如com.raywenderlich.Incognito://doStuff，重点就是自定义一个独特的scheme使其与设备中安装的其他app有所区别。

在OAuth2之舞中使用你自定义的URL scheme根据请求来返回到app中。自定义Scheme中包含一些参数，在这里，授权码被包含在code参数中。 OAuth2库会从URL中提取授权码然后在下一次请求时与访问令牌进行交换。

你需要在Incognito的AppDelegate类中实现一个基于自定义URL scheme的响应方法。

打开AppDelegate.swift，在文件头部添加如下代码:
````swift
import AeroGearOAuth2
````
接着像下面这样实现application(_: openURL: sourceApplication: annotation)：
````swift
func application(application: UIApplication,
  openURL url: NSURL,
  sourceApplication: String?,
  annotation: AnyObject?) -> Bool {
    let notification = NSNotification(name: AGAppLaunchedWithURLNotification,
      object:nil,
      userInfo:[UIApplicationLaunchOptionsURLKey:url])
    NSNotificationCenter.defaultCenter().postNotification(notification)
    return true
}
````
这个方法中创建了一个包含打开app时URL信息的NSNotification。AeroGearOAuth2库会接收到这个通知，然后调用你之前插入的POST方法中的completionHandler。

构建并运行一下工程，选个时髦的自拍并打扮一下，点击分享按钮，认证，观察一下发生了什么：

![](http://www.raywenderlich.com/wp-content/uploads/2015/03/incognito_flow3.gif)

你可以从[这里](http://www.raywenderlich.com/wp-content/uploads/2015/05/Incognito.aerogear_final.zip)下载完成后的应用。

在OAuth2授权过程中跳转到外部浏览器未免显得太笨重，应该有一个简化的实现方法...

<a name="web-view"></a>
##使用嵌入的Web视图

嵌入的web视图有更好的用户体验。使用UIWebView来实现而不是跳转到Safari中。从安全的角度来讲，使用app的代码来处理登录表单与提供商的数据会使安全性降低。当用户输入信息时app可以使用JS去访问用户证书，如果你的终端用户信任app的安全性的话是一个可考虑的方案。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2015/03/oauth2-explained-2.png)

来使用OAuthSwift库重新实现分享的方法，不过这次使用的是嵌入的web视图。

<a name="oauthswift"></a>
##使用嵌入web视图的OAuthSwift

将会开始一个不同的新工程，所以先关闭Xcode的工作区，下载Incognito初始工程的版本，用Xcode打开Incognito.xcworkspace文件。

构建并运行工程，看起来挺熟悉的。

首先需要将OAuthSwift库加入到工程中。

打开ViewController.swift在文件头部添加如下代码：
````swift
import OAuthSwift
````
在share()方法中添加如下代码:
````swift
// 1 Create OAuth2Swift object
let oauthswift = OAuth2Swift(
  consumerKey:    "YOUR_GOOGLE_DRIVE_CLIENT_ID",         // 2 Enter google app settings
  consumerSecret: "YOUR_GOOGLE_DRIVE_CLIENT_SECRET",
  authorizeUrl:   "https://accounts.google.com/o/oauth2/auth",
  accessTokenUrl: "https://accounts.google.com/o/oauth2/token",
  responseType:   "code"
)
// 3 Trigger OAuth2 dance
oauthswift.authorizeWithCallbackURL(
  NSURL(string: "com.raywenderlich.Incognito:/oauth2Callback")!,
  scope: "https://www.googleapis.com/auth/drive",        // 4 Scope
  state: "",
  success: { credential, response in
    var parameters =  [String: AnyObject]()
    // 5 Get the embedded http layer and upload
    oauthswift.client.postImage(
      "https://www.googleapis.com/upload/drive/v2/files",
      parameters: parameters,
      image: self.snapshot(),
      success: { data, response in
        let jsonDict: AnyObject! = NSJSONSerialization.JSONObjectWithData(data,
          options: nil,
          error: nil)
        self.presentAlert("Success", message: "Successfully uploaded!")
      }, failure: {(error:NSError!) -> Void in
        self.presentAlert("Error", message: error!.localizedDescription)
    })
  }, failure: {(error:NSError!) -> Void in
    self.presentAlert("Error", message: error!.localizedDescription)
})
````
上面的代码进行了如下操作：

1. 首先创建一个OAuth2Swift对象，将为你执行OAuth之舞。
2. 将YOUR_GOOGLE_CLIENT_ID与YOUR_GOOGLE_DRIVE_CLIENT_SECRET替换为你的客户端ID与客户端密钥。
3. 通过oauthswift实例来请求授权。
4. 这个域名是你请求访问的Drive API。
5. 如果授权成功则可以开始上传图片了。

<a name="con-url"></a>
##配置URL
如之前的工程那样的，需要设置一个Incognito接受的URL scheme，你所要做的就是实现处理自定义URL的代码。

打开AppDelegate.swift添加如下代码：
````swift
import OAuthSwift
````
然后像下面这样实现application(_:openURL: sourceApplication: annotation:)方法:
````swift
func application(application: UIApplication,
  openURL url: NSURL,
  sourceApplication: String?,
  annotation: AnyObject?) -> Bool {
    OAuth2Swift.handleOpenURL(url)
    return true
}
````
跟AeroGearOAuth2不同, OAuthSwift使用一个类方法去处理并解析返回的URL。不过如果你看看handleOpenURL(_) 方法的代码时会发现仅仅是发送了一个NSNotification，就像用AeroGearOAuth2时需要你做的那样!

构建并运行你的工程，创建一个新的自拍然后上传。喔! 再一次成功了! 挺简单的吧 :]

<a name="display-webview"></a>
##显示UIWebView
现在就要如约添加web view了。点击Incognito文件夹，在Xcode中的工程导航栏处选择File\New\File，然后选择iOS\Source\Swift File，将其命名为 WebViewController然后保存进工程中。

然后打开WebViewController.swift添加如下代码：
````swift
import UIKit
import OAuthSwift
 
class WebViewController: OAuthWebViewController {
  var targetURL : NSURL?
  var webView : UIWebView = UIWebView()
 
  override func viewDidLoad() {
    super.viewDidLoad()
    webView.frame = view.bounds
    webView.autoresizingMask =
      UIViewAutoresizing.FlexibleHeight | UIViewAutoresizing.FlexibleHeight
    webView.scalesPageToFit = true
    view.addSubview(webView)
    loadAddressURL()
  }
 
  override func setUrl(url: NSURL) {
    targetURL = url
  }
 
  func loadAddressURL() {
    if let targetURL = targetURL {
      let req = NSURLRequest(URL: targetURL)
      webView.loadRequest(req)
    }
  }
}
````
在上面的代码中创建了一个继承OAuthWebViewController的WebViewController，这个类只实现了一个方法：SetUrl:。在viewDidLoad()中调整web view的尺寸然后添加到viewcontroller的父视图中。此外，在OAuth2Swift的实例中加载URL，创建一个request。

接着打开ViewController.swift找到share()方法。在创建完oauthswift实例后添加如下代码：
````swift
oauthswift.webViewController = WebViewController()
````
这里会告诉oauthswift实例使用刚刚创建的web view controller。

最后，打开AppDelegate.swift，修改application(_:openURL: sourceApplication: annotation:) 方法，在return true的前面添加如下代码：
````swift
// [1] Dismiss webview once url is passed to extract authorization code
UIApplication.sharedApplication().keyWindow?.rootViewController?.dismissViewControllerAnimated(true, completion: nil)
````
构建并运行工程，注意当授权窗口出现时，并不是通过Safari显示的，未产生app间的切换。由于默认在你app中是不会存储cookies的因此每次都会出现授权的窗口。

使用一个UIWebView去授权Google当然会更流畅，的确是! :]

你可以在这里下载最终的[Incognito](http://www.raywenderlich.com/wp-content/uploads/2015/05/Incognito.oauthswift_final.zip)。

在这个教程中还剩下一件事。来重新审视share()方法，使用著名的HTTP库AFNetworking在OAuth2中的应用。

<a name="afoauthma"></a>
##使用AFOAuth2Manager授权
AFOAuth2Manager使用一种与其他OAuth2库完全不同的方式：使用基于著名的AFNetworking框架的底层API。至于你想用一个UIWebView还是打开一个外部的浏览器完全由你来决定，开发者可自由选择在OAuth2之舞中第1步的初始化方式。

这个部分的教程从另一个初始工程开始，关闭已有的工程下载这个新的：[Incognito starter project](http://www.raywenderlich.com/wp-content/uploads/2015/05/Incognito.afoauth2manager_start.zip).

打开一个新工程进入ViewController.swift中，首先定义一些帮助方法与扩展。

在文件头部添加如下的String扩展：
````swift
extension String {
  public func urlEncode() -> String {
    let encodedURL = CFURLCreateStringByAddingPercentEscapes(
		  nil,
      self as NSString,
      nil,
      "!@#$%&*'();:=+,/?[]",
      CFStringBuiltInEncodings.UTF8.rawValue)
    return encodedURL as String
  }
}
````
上面的代码扩展了一个String类的函数，使用URL编码一段字符串。

在ViewController中添加如下方法：
````swift
func parametersFromQueryString(queryString: String?) -> [String: String] {
  var parameters = [String: String]()
  if (queryString != nil) {
    var parameterScanner: NSScanner = NSScanner(string: queryString!)
    var name:NSString? = nil
    var value:NSString? = nil
    while (parameterScanner.atEnd != true) {
      name = nil;
      parameterScanner.scanUpToString("=", intoString: &name)
      parameterScanner.scanString("=", intoString:nil)
      value = nil
      parameterScanner.scanUpToString("&", intoString:&value)
      parameterScanner.scanString("&", intoString:nil)
      if (name != nil && value != nil) {
        parameters[name!.stringByReplacingPercentEscapesUsingEncoding(NSUTF8StringEncoding)!]
          = value!.stringByReplacingPercentEscapesUsingEncoding(NSUTF8StringEncoding)
      }
    }
  }
  return parameters
}
````
从URL字符串中提取中查询的参数字符串。比方说如果查询字符串是name=Bob&age=21则方法会返回一个字典：name => Bob, age => 21。

接着你需要在ViewController中定义一个辅助函数，用来提取NSNotification中URL的OAuth码。

在share()方法下面添加如下方法:
````swift
func extractCode(notification: NSNotification) -> String? {
  let url: NSURL? = (notification.userInfo as!
    [String: AnyObject])[UIApplicationLaunchOptionsURLKey] as? NSURL
 
  // [1] extract the code from the URL
  return self.parametersFromQueryString(url?.query)["code"]
}
````
使用刚才实现的方法从查询字符串字典中抓取了键"code"的值。

在share()中添加如下:
````swift
// 1 Replace with client id /secret
let clientID = "YOUR_GOOGLE_CLIENT_ID"
let clientSecret = "YOUR_GOOGLE_CLIENT_SECRET"
 
let baseURL = NSURL(string: "https://accounts.google.com")
let scope = "https://www.googleapis.com/auth/drive".urlEncode()
let redirect_uri = "com.raywenderlich.Incognito:/oauth2Callback"
 
if !isObserved {
  // 2 Add observer
  var applicationLaunchNotificationObserver = NSNotificationCenter.defaultCenter().addObserverForName(
    "AGAppLaunchedWithURLNotification",
    object: nil,
    queue: nil,
    usingBlock: { (notification: NSNotification!) -> Void in
      // [5] extract code
      let code = self.extractCode(notification)
 
      // [6] carry on oauth2 code auth grant flow with AFOAuth2Manager
      var manager = AFOAuth2Manager(baseURL: baseURL,
        clientID: clientID,
        secret: clientSecret)
      manager.useHTTPBasicAuthentication = false
 
      // [7] exchange authorization code for access token
      manager.authenticateUsingOAuthWithURLString("o/oauth2/token",
        code: code,
        redirectURI: redirect_uri,
        success: { (cred: AFOAuthCredential!) -> Void in
 
          // [8] Set credential in header
          manager.requestSerializer.setValue("Bearer \(cred.accessToken)",
            forHTTPHeaderField: "Authorization")
 
          // [9] upload photo
          manager.POST("https://www.googleapis.com/upload/drive/v2/files",
            parameters: nil,
            constructingBodyWithBlock: { (form: AFMultipartFormData!) -> Void in
              form.appendPartWithFileData(self.snapshot(),
                name:"name",
                fileName:"fileName",
                mimeType:"image/jpeg")
            }, success: { (op:AFHTTPRequestOperation!, obj:AnyObject!) -> Void in
              self.presentAlert("Success", message: "Successfully uploaded!")
            }, failure: { (op: AFHTTPRequestOperation!, error: NSError!) -> Void in
              self.presentAlert("Error", message: error!.localizedDescription)
          })
        }) { (error: NSError!) -> Void in
          self.presentAlert("Error", message: error!.localizedDescription)
      }
  })
  isObserved = true
}
 
// 3 calculate final url
var params = "?scope=\(scope)&redirect_uri=\(redirect_uri)&client_id=\(clientID)&response_type=code"
// 4 open an external browser
UIApplication.sharedApplication().openURL(NSURL(string: "https://accounts.google.com/o/oauth2/auth\(params)")!)
````
哇哦，这个方法好长！如果一步步分析的话就会理解的：

1. 通常，你需要使用Google控制台的客户端id与客户端密钥替换掉YOUR_GOOGLE_CLIENT_ID与YOUR_GOOGLE_DRIVE_CLIENT_SECRET
2. 接着添加一个notification的观察者
3. 创建一个OAuth所需的参数列表
4. 使用Safari打开URL开始OAuth之舞
5. 一旦用户授权成功，这个闭包就会执行，开始从回调URL中提取OAuth码
6. OAuth流的第二步：交换OAuth码得到令牌
7. 一得到令牌…
8. …就绑定到HTTP头中…
9. …最后, 上传图片至Google Drive

还差一步!

搞定之前的后，打开AppDelegate.swift添加如下方法：
````swift
func application(application: UIApplication,
  openURL url: NSURL,
  sourceApplication: String?,
  annotation: AnyObject?) -> Bool {
    let notification = NSNotification(
      name: "AGAppLaunchedWithURLNotification",
      object:nil,
      userInfo:[UIApplicationLaunchOptionsURLKey:url])
    NSNotificationCenter.defaultCenter().postNotification(notification)
    return true
}
````
启动了一个notification, 授权代码所监听的对象，为了获得URL的授权码。

最后一次构建并运行工程，再次成功运行! 效果如之前一样不过这次使用了AFOAuth2Manager!

你可以在[这里](http://www.raywenderlich.com/wp-content/uploads/2015/05/Incognito.afoauth2manager_finish.zip)下载最终的Incognito AFOAuth2Manager工程。
