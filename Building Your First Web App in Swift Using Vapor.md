# Building Your First Web App in Swift Using Vapor
## 使用Vapor构建你的第一个Swift Web应用

***

>* 原文链接 : [Building Your First Web App in Swift Using Vapor](http://www.appcoda.com/server-side-swift-vapor/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : [yrq110](https://github.com/yrq110)

***

苹果在WWDC 2015上宣布swift将会开源，很快，在2015年的12月Swift的源码就在[GitHub](https://github.com/apple/swift)上公开了。

Swift的开源给了开发者更多的机会与可能性，扩展Swift生态圈的应用。

如所期待的那样，开发者们很快就探索出了Swift的很多其他方面的应用。这篇教程中所介绍的web app就是其中的一种应用。

## 为何关注?

在服务端使用一种强类型的语言是有很多益处的。若要为你的iOS app开发后端功能的话，使用同一种语言与风格比较容易保持一致性。

这里有三种很不错的Swift服务端：Perfect, Kitura与Vapor。这篇教程中会使用Vapor来探索在Swift服务端上的神奇Swift世界。

我会指导你如何安装Vapor与Swift3，学习服务端Swift的基础知识，然后在Heroku上部署网站/后端。

## 前提

这篇教程中会使用很多bash命令，因此需要掌握一些基本的bash与终端知识是很重要的。Vapor是基于Swift3的，因此也需要Xcode8。

Vapor与其他Swift工程(Swift包管理器)都是基于Swift3的，建议你通过这篇[文章](https://github.com/yrq110/Some_IOS_Tutorials_With_Swift/blob/master/What%E2%80%99s%20New%20in%20Swift%203.md)了解下Swift3中的一些变化。

在教程的最后，会教你如何在Heroku上部署Vapor服务端，Heroku是个很火的云服务供应商，推荐尝试下。

开搞吧!

## 安装Vapor

Vapor需要Swift 3与Xcode8，可以在[这里](https://developer.apple.com/download/)下载GM版Xcode。 and the latest version of Xcode 8 beta. You can download the new Xcode GM here.

首先需要选择最新的命令行工具，打开Xcode的preferences(偏好设置)。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s1.png)

接着导航栏中的Locations。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s3-1024x653.png)

最后，选择最新的Xcode8命令行工具集。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s4-1024x666.png)

## 安装Swiftenv

是时候安装Swift 3了。首先先来安Swiftenv。Swiftenv使Swift的安装变得很轻松，并且可以在多个版本间切换。通过clone Github上的仓库来安装Swiftenv，在终端里运行如下代码:

```bash
git clone https://github.com/kylef/swiftenv.git ~/.swiftenv
```

然后在bash文件中初始化Swiftenv，bash文件就是设置bash属性与设置的地方。在终端里使用如下指令打开bash文件:

```bash
open ~/.bash_profile
```

open指令会使用TextEdit打开你的bash文件，若没有bash文件，使用如下指令创建它:

```bash
touch ~/.bash_profile
```

打开bash文件后，复制如下指令，用来初始化Swiftenv。

```bash
export SWIFTENV_ROOT="$HOME/.swiftenv"
export PATH="$SWIFTENV_ROOT/bin:$PATH"
eval "$(swiftenv init -)"
```

你的bash文件(在TextEdit中打开)中现在应该包含了这些指令。注意有些情况下bash文件可能包含其他指令。不要删掉之前的指令，在文件末端添加新的即可。

![](http://www.appcoda.com/wp-content/uploads/2016/09/s6-1024x682.png)

保存bash文件并重启终端。终端重启后，输入下面这个指令确保Swiftenv已经成功安装了:

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

会看到如下一个命令列表:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s12-1024x588.png)

恭喜! 你完成了教程中最具挑战的部分(PS:???)。


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

Vapor是遵循MVC(模型-视图-控制器)模式的，因此它会创建对应的文件夹来存放模型，视图与控制器，若你不太熟悉MVC设计模式，点击[这里](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)学习一下。

首先，Vapor创建好了Package.swift文件，不需要管它。然后是App文件夹，它包含了模型、控制器、中间件与main.swift文件，main.swift文件是我们app的main文件，在这里会初始化服务端，Vapor会首先运行这个文件。

Resources文件夹包含Views，用来存放HTML文件与模板。Public文件夹中存放image与style文件。其他部分现在还用不上暂且不用管。

# The Droplet

Vapor automatically creates an example project for us. It will create a Controller, Model, Middleware, and main.swift file. Open the main.swift file (under /Sources/App), and remove all the code inside it.

A Droplet is the heart of a Vapor server. It contains a plethora of functions that will be the backbone of our server. First, import Vapor in our main.swift file:

```swift
import Vapor
```
Then, let’s create a Droplet:
```swift
let drop = Droplet()
```
A Droplet has a lot of customizable properties. It accepts a ton of arguments, all cited in the Vapor Docs. For this instance we don’t need to customize our Droplet.
Now let’s try to handle the main '/' or index request to our web page:
```swift
drop.get("/") { request in
    return "Hello World!"
}
```
Finally, you need to call the serve() function. The serve function runs the server.
```swift
drop.run()
```
Now save the main.swift file. Vapor can build and run the server for you. Run the following commands:
```swift
vapor build
vapor run
```
Vapor doesn’t have a ‘special’ build or execution. Running the vapor build command simply does a swift build . Entering the vapor run command will run the builds created in the .build/debug/ directory.

This operation might take a while for the first time. After a successful build and execution, you should see something like this:

![](http://www.appcoda.com/wp-content/uploads/2016/09/s14-1024x616.png)

Vapor initially runs the server on the 8080 port. If you would like to change the port, you must change the configuration by modifying the servers.json file here:

```
├── Package.swift
├── App
├── Resources
├── Config
  |    └── servers.json  <--
├── Localization
└── Public
```
Assuming you haven’t changed the port from 8080, navigate to this link to where the server is running:
```swift
0.0.0.0:8080
```
or
```swift
localhost:8080
```
You should see your incredible “Hello World” String!

> Note: Every time you make a change, it is necessary first to build, then run the project. If you only run the project, the previous build will be executed.


## Handling HTTP Requests

Handling an HTTP request is similar to other frameworks such as Express, Flask, etc. We will cover GET in this tutorial.

For instance if you want to output the string "Hello John!" when you open "localhost:8080/name/John". Let’s look at this piece of code:

```swift
drop.get("/name",":name") { request in
    if let name = request.parameters["name"]?.string {
        return "Hello \(name)!"
    }
    return "Error retrieving parameters."
}
```
Our Droplet has a GET handler function called get(). The get() function can take multiple arguments. The first argument will be the name of the GET request parameter. The argument that follows will be the key of the that parameter. E.g. :
```swift
drop.get("route", ":key", "route2", ":key2")
```
This way you could access the value of that parameter by supplying the key to a dictionary. The key, or the second argument, must start with a colon, which indicates that the name is the key of the parameter.

The get() function provides us a request object. This object contains everything related to our GET request. To access the parameter which was submitted, use the parameters dictionary of our request object:
```swift
request.parameters["key"]
```
We ensure that the parameter has a value by using the if let statement:
```swift
if let name = request.parameters["name"].string {
    // Do something
}
```
The HTTP request needs a response from the server. The response can be returned by just returning the function. In this case, we will return the name string, which was taken from our parameters dictionary.
