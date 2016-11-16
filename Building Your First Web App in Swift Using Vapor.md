# Building Your First Web App in Swift Using Vapor
## 使用Vapor构建你的第一个Swift Web应用

***

>* 原文链接 : [Building Your First Web App in Swift Using Vapor](http://www.appcoda.com/server-side-swift-vapor/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : [yrq110](https://github.com/yrq110)

***

苹果在WWDC 2015上宣布swift将会开源，很快，在2015年的12月Swift的源码就在[GitHub](https://github.com/apple/swift)上公开了。

Swift的开源给了开发者更多的机会与可能性，扩展Swift生态圈的应用。

如所期待的那样，开发者们很快就探索出了Swift的很多其他方面的应用，这篇教程中所介绍的web app就是其中的一种应用。

## 为何关注?

在服务端使用一种强类型的语言是有很多益处的。如果想给你的iOS app开发后端功能的话，使用同一种语言与风格比较容易保持一致性。

这里有三种很不错的Swift服务端：Perfect, Kitura与Vapor。这篇教程中会使用Vapor探索在Swift服务端上的神奇Swift世界。

我会指导你如何安装Vapor与Swift3，学习服务端Swift的基础知识，然后在Heroku上部署网站/后端。

## 前提

这篇教程中会使用很多bash命令，因此需要掌握一些基本的bash与终端知识。Vapor是基于Swift3的，因此也需要Xcode8。

Vapor与其他Swift工程(比如Swift包管理器)都是基于Swift3的，建议你通过这篇[文章](https://github.com/yrq110/Some_IOS_Tutorials_With_Swift/blob/master/What%E2%80%99s%20New%20in%20Swift%203.md)了解下Swift3中的一些变化。

在教程的最后，会教你如何在Heroku上部署Vapor服务端，Heroku是个很火的云服务供应商，推荐你尝试下。

开搞吧!

## 安装Vapor

Vapor需要Swift 3与Xcode8，可以在[这里](https://developer.apple.com/download/)下载GM版Xcode。

首先需要选择最新的命令行工具，打开Xcode的preferences(偏好设置)。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s1.png)

接着来到导航栏的Locations设置。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s3-1024x653.png)

最后，选择最新的Xcode8命令行工具集。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s4-1024x666.png)

## 安装Swiftenv

是时候安装Swift 3了。首先安装Swiftenv，使用Swiftenv可以很轻松的安装Swift，并且可以在多个版本间切换。通过clone Github上的仓库来安装Swiftenv，在终端里运行如下代码:

```bash
git clone https://github.com/kylef/swiftenv.git ~/.swiftenv
```

然后在bash文件中初始化Swiftenv，bash文件就是存放bash属性与一些设置的地方。在终端里使用如下命令打开bash文件:

```bash
open ~/.bash_profile
```

open指令会使用TextEdit打开你的bash文件，若没有bash文件，使用如下指令创建它:

```bash
touch ~/.bash_profile
```

打开bash文件后，复制如下命令，用来初始化Swiftenv。

```bash
export SWIFTENV_ROOT="$HOME/.swiftenv"
export PATH="$SWIFTENV_ROOT/bin:$PATH"
eval "$(swiftenv init -)"
```

你的bash文件(在TextEdit中打开)中现在应该包含了这些命令。注意有些情况下bash文件可能包含其他命令。不要删掉之前的命令，在文件末端添加新的即可。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s6-1024x682.png)

保存bash文件并重启终端。终端重启后，输入下面这个命令确保Swiftenv已经成功安装了:

```bash
swiftenv --version
```

在终端中应如下所示:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s7-1024x643.png)

## 下载Swift 3快照

安好Swiftenv了，接着来下载最新的Swift 3快照(Snapshot)，Swift快照就是个打包好用于安装的Swift。执行如下命令:

```bash
swiftenv install 3.0-GM-CANDIDATE
```

> 更新: 若3.0-GM-CANDIDATE不好使，请使用DEVELOPMENT-SNAPSHOT-2016-09-06-a。

会花点时间下载，下载完成后在终端里将快照版本设置为主Swift版本，这将会是全局的版本，执行如下命令:

```bash
swiftenv global 3.0-GM-CANDIDATE
```

检查下版本是否是3.0-GM-CANDIDATE:

```bash
swiftenv versions
```

应该会看到3.0-GM-CANDIDATE后面有个星号。

## 完整性检验

Vapor需要合适的Swift版本才能正常启动，Vapor提供了一个shell脚本来用于检验完整性，执行如下命令来确保万事俱备:

```bash
curl -sL check.vapor.sh | bash
```

若显示出绿色箭头的emoji表情，就表示准备好安装Vapor了!

![](http://www.appcoda.com/wp-content/uploads/2016/09/s10-1024x598.png)

## 安装Vapor Toolbox

使用如下命令下载并安装Vapor:

```bash
curl -sL toolbox.vapor.sh | bash
```

好吧，这会花点时间，也许会出错，比如"Install failed, trying sudo"。当看到"Vapor Toolbox v0.10.4 Installed"信息时就说明安装成功了。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s11-1024x601.png)

在终端里试试如下命令，来验证下Vapor是否准备就绪:

```bash
vapor
```

会看到如下这样的一个命令列表:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s12-1024x588.png)

恭喜! 你完成了教程中最具挑战的部分。


# 开始一个新的Vapor工程

所有Swift项目都使用了Swift包管理器(SPM)，每个SPM项目都需要一个Package.json文件与一个main.swift文件等等。SPM会查找项目中的Package.swift文件并下载必要的依赖库，SPM有点像NPM(Node包管理器)。Vapor会创建Swift工程并将所需的依赖添加到Package.swift文件中。

在创建工程时需要给Vapor提供一个项目名称，首先进入一个想要保存新工程的目录下，执行如下命令:

```bash
vapor new HelloWorld
```

![](http://www.appcoda.com/wp-content/uploads/2016/09/s13-1024x666.png)

Vapor会在当前的目录下创建一个名为HelloWorld的工程，使用如下命令进入新工程的目录下，目录名称与工程名称是相同的:

```bash
cd HelloWorld
```

会看到一个如下的目录:

```bash
├── Package.swift
├── App
│   ├── main.swift
│   ├── Controllers
│   ├── Middleware
│   └── Models
├── Resources
 |    └── Views
├── Config
├── Localization
└── Public
```

Vapor是遵循MVC(模型-视图-控制器)模式的框架，因此它会创建对应的文件夹来存放模型，视图与控制器，若你不太熟悉MVC设计模式，点击[这里](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)学习一下。

首先，Vapor创建好了Package.swift文件，不需要管它。然后是App文件夹，它包含了模型、控制器、中间件与main.swift文件，main.swift文件是我们app的main文件，在这里会初始化服务端，Vapor会首先运行这个文件。

Resources文件夹包含Views，用来存放HTML文件与模板。Public文件夹中存放image与style文件。其他部分现在还用不上暂且不用管。

# Droplet

Vapor会自动创建一个示例工程，包含控制器，模型，中间件与main.swift文件。打开main.swift文件(在/Sources/App目录下)，清空里面的代码。

Droplet是Vapor服务器的心脏，它包含了大量后端服务器所使用的函数。首先，在main.swift文件中导入Vapor:

```swift
import Vapor
```
接着来创建一个Droplet:
```swift
let drop = Droplet()
```
Droplet中有很多自定义的属性，初始化时可接受多个命令，在[Vapor的文档](https://vapor.github.io/documentation/guide/droplet.html)中都可以查到。目前阶段我们不需要自定义Droplet。

现在来处理一下对网页的'/'或index请求:
```swift
drop.get("/") { request in
    return "Hello World!"
}
```
最后，需要调用serve()函数来启动服务器。
```swift
drop.run()
```
保存main.swift文件，现在Vapor可以构建并运行服务器了，执行如下命令:
```swift
vapor build
vapor run
```
Vapor没有"特殊"的构建或执行参数，简单的启动`vapor build`命令即可，输入`vapor run`命令会在`.build/debug/`目录下进行构建。

这个操作第一次进行时可能会花点时间，构建执行成功后会看到如下界面:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s14-1024x616.png)

Vapor一开始会启动服务器的8080端口，若你想修改端口号，必须在servers.json文件中修改设置:

```
├── Package.swift
├── App
├── Resources
├── Config
  |    └── servers.json  <--
├── Localization
└── Public
```
假定你没有修改8080端口, 可以访问下面的链接来访问服务器:
```swift
0.0.0.0:8080
```
或
```swift
localhost:8080
```
应该会看到 “Hello World” 字样!

> 注意: 每次改动后都需要build与run一下，如果仅仅执行了run命令则会执行之前构建好的工程。

## 处理HTTP请求

处理HTTP请求的方法跟其他比如Express，Flask等框架的做法挺像的，这篇教程中会演示GET方法的处理。

假设，现在想要在打开"localhost:8080/name/John"地址时输出字符串"Hello John!":

```swift
drop.get("/name",":name") { request in
    if let name = request.parameters["name"]?.string {
        return "Hello \(name)!"
    }
    return "Error retrieving parameters."
}
```
Droplet中用于处理GET的函数get()，get()函数可以接受多个参数，第一个参数是GET请求的参数名称，后面的参数是请求中参数的key值，比如:
```swift
drop.get("route", ":key", "route2", ":key2")
```
使用这种方法可以通过提供字典中的key值来访问参数中对应的具体指。输入key时必须在前面加上冒号，表示这是个参数的key值。

get()函数提供了一个request对象，这个对象包含与GET请求相关的所有信息。使用request对象的parameters字典来访问提交的参数:
```swift
request.parameters["key"]
```
使用if let语句来确保参数中的值存在:
```swift
if let name = request.parameters["name"].string {
    // Do something
}
```
HTTP请求需要一个来自服务端的响应。在这里，我们返回来自参数字典的`name`字符串。

构建并运行工程，使用如下地址来测试一个GET请求:

```
http://localhost:8080/name/John
```
应该会显示 “Hello John!”

注意，这里的路由规则与其它框架的规则(比如Express)是不同的，在Vapor中，你不能使用URI格式来访问参数:
```swift
http://localhost:8080/name?name=John&age=18
```
每个参数前都必须使用斜杠:
```swift
http://localhost:8080/name/John/age/18
```
不过若你想用URI格式的话，可以使用request对象的uri属性。想了解更多请看request对象的[文档](https://vapor.github.io/documentation/http/request.html)。

## 返回视图

Droplet可以给客户端返回一个视图，视图就是保存在/Resources/Views文件夹下的HTML文件。从这里下载一个示例view.html文件，这是个很简单的渲染网页的HTML文件，下载后将它移动到/HelloWorld/Resources/Views中:

```
├── Package.swift
├── App
├── Resources
 |    └── Views     <--
├── Config
├── Localization
└── Public
```

上面这个目录就是Droplet查询视图文件的地方，现在来实现具体代码吧，打开main.swift文件，处理一下/view路径:

```swift
drop.get("/view") { request in
    return try drop.view("view.html")
}
```
Droplet中有一个接受字符串参数的view()函数，参数就是HTML文件的文件名。

view()函数可能会抛出一个异常，因此使用try关键字标记是很必要的。若视图不存在，则函数会抛出一个异常，可使用catch关键字来捕捉异常，不过在这里没有必要这样因为我们知道文件是存在的。

构建并运行一下:
```bash
vapor build
vapor run
```
访问这个地址:
```
http://localhost:8080/view
```
应该会看到包含“Hello World”字样的视图。访问Views的[文档](https://vapor.github.io/documentation/guide/views.html)来学习更多有关模板与渲染的内容。

## 部署到Heorku

首先，创建一个Github仓库，我给仓库起名为VaporExample，到Vapor工程的目录下初始化git:
```bash
git init
```
接着设置remote origin:
```bash
git remote set origin < 这里是你的Github仓库 >
```
添加文件, 提交修改, 然后将仓库push到Github:
```bash
git add .
git commit -m "Init"
git push -u origin master
```
哦了，现在完成将仓库push到Github上的工作了。前往Heroku控制台，创建一个新的app，随意起名。首先来到settings选项卡中。

![](http://www.appcoda.com/wp-content/uploads/2016/09/h2-1024x640.png)

接着，向下滚动到“Buildpacks”选项。

添加一个Buildpack: https://github.com/kylef/heroku-buildpack-swift 。Buildpack构建你的Swift工程并且告诉Heroku工程所使用的语言。

下面前往Deploy区域。

![](http://www.appcoda.com/wp-content/uploads/2016/09/h4-1024x640.png)

将deployment method改为GitHub，并添加Github的仓库名，将你的仓库与Heroku连接起来。向下滚动到Manual Deploy部分，点击Deploy Branch，构建过程可能会花点时间。

![](http://www.appcoda.com/wp-content/uploads/2016/09/h5-1024x640.png)

构建完成后，访问Heroku提供的地址来查看你的站点!

![](http://www.appcoda.com/wp-content/uploads/2016/09/h7-1024x640.png)

## 总结

希望你可以享受学习Vapor的过程！可以在[Github](https://github.com/SahandTheGreat/VaporExample)上下载到完整工程，同时[这里](https://github.com/vapor/vapor/blob/master/Documents/PROJECTS.md)也有一些Vapor作者所写的示例项目。
