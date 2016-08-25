#Managing SQLite Database with SwiftyDB
##使用SwiftyDB管理SQLite数据库

***

>* 原文链接 : [Managing SQLite Database with SwiftyDB](http://www.appcoda.com/swiftydb/)
* 原文作者 : [GABRIEL THEODOROPOULOS](http://www.appcoda.com/author/gabrielth/)
* 译者 : []()

***

开发应用的过程中总是需要一种数据持久化的方案，有很多方式供我们选择:使用本地文件存储，用CoreData或者SQLite数据库。最后一种方式(使用SQLite)有些麻烦，就是在应用程序使用数据库之前首先必须创建数据库，定义好所有数据库表和数据表字段。而且在代码层面来看，使用SQLite管理数据的存储，更新和获取并不是那么容易。

上面所说的这些问题貌似被突然出现在GitHub上的一个叫[SwiftDB](https://github.com/Oyvindkg/swiftydb)的库给解决了。按作者的话说，这是一个“即插即用”的第三方库。SwiftDB让开发者从手动建库，建表，定义字段的繁杂代码中解脱出来。它会自动提取类中的属性做为数据模型。另外，所有的数据库操作都是黑盒(外部看不到具体的数据库操作)，开发者就只需要关心应用的业务逻辑。简单而强大的API会让你在处理数据库相关业务的时候尝到甜头。

有必要提一下就是，你可千万不要以为SwiftDB是万能的。它的确是一个非常靠谱的库，能做的事都做得非常好，但仍然有一些特性不支持，可能在不久的将来这这些特性会被加上。无论如何，这都是一个非常好的工具库，应该受到重视，所以本入门教程里，我们会介绍整个SwiftDB的基本功能。

做为参考，可以在[这里](http://oyvindkg.github.io/swiftydb/)找到SwiftDB的文档，当然你得先看完本文再去看文档。可能你已经想要用SQLite了，但是还有些犹豫，那SwiftDB会给你一个定心丸。

上面的说完了，现在我们就一探这个靠谱工具库的究竟。

## 关于Demo

在今天的文章里，我们就真正的创建一个简单的笔记应用，能执行所有我们想要的操作:

- 笔记列表
- 创建新笔记
- 更新现在笔记
- 删除笔记

显然，SwiftDB会负责管理SQLite数据库里面的数据。所有上面这些列出来的操作都足以让你入门SwiftDB。

为了尽可能的保持咱们二边的一致(学习和讲解)，你可以下载这个我已经创建好的[初始工程](https://github.com/appcoda/SwiftyDB-Demo/blob/master/NotesDBStarter.zip?raw=true)，从这个工程开始学习。下载完之后用Xcode打开浏览一下工程，熟悉一下工程里的内容。如你所见，所有的都是基础的功能，没有数据相关的特性。能跑一下这个工程最好不过了，也好知道app里面大致是什么。
App是基于导航的，在第一个视图控制器里包含了一个展示所有已经保存笔记的tableview:

![](http://www.appcoda.com/wp-content/uploads/2016/03/t50_1_note_list.png)

点击每条笔记，可以编辑、修改笔记，我们还可以在每条笔记上左滑来删除笔记。

![](http://www.appcoda.com/wp-content/uploads/2016/03/t50_2_delete_note.png)

可以通过点击导航栏上的(+)加号来创建新笔记。为了有足够的操作练习的例子可用，下面列出了在编辑笔记时可用的操作(有已经在demo里实现的，也有新增的)：
1. 设置标题和正文
2. 更改字体
3. 更改字号
4. 更改文本颜色
5. 笔记本引入图片
6. 范围内移动图片，指定图片位置

所有上面列出的操作所产生的数据都会存储在数据库里。为了让上面最后二条更加明晰，我们把真正的图片文件存储在app的文档目录下，而数据库里只存储对应的图片名称和图片尺寸。不但如此，我们还得创建一个专门管理图片的类(稍后给出具体实现)。

![](http://www.appcoda.com/wp-content/uploads/2016/03/t50_3_edit_note.png)

最后还有件重要的事需要说明，即使你已经下载了初始工程，但是在下一节最后，你还是需要一个单独的工作目录。因为我们要用**CocoaPods**来下载SwiftDB库，或者任意其他工程里需要的依赖库。

准备开始吧！但首先，如果你打开了那个初始工程，先关了它。

## 安装SwiftDB

最先要做的就是下载SwiftDB库，然后放到我们的工程里使用。只是下载库文件再直接加入到工程里这样是不能用的，必须使用**[CocodPods](https://cocoapods.org/)**依赖管理器来安装SwiftDB库。过程非常简单，即使你之前没用过**CoacoPods**也能马上就搞定。你可以看一下上文CocoaPods的链接，参考一下。

### 安装 CocoaPods

首先要从安装CocoaPods开始。如果已安装过CocoaPods，直接跳过这步。如果没安过，先打开**终端(Terminal)**。输入下面命令来安装CocoaPods:

> sudo gem install cocoapods

回车，输入密码，下载的时候可以稍等一会儿休息下。下载完之后别关终端，一会儿还要用。

### 安装 SwiftyDB及相关依赖

使用**cd**命令进入到初始工程的根目录下(在终端里):

>  cd PATH_TO_THE_STARTER_PROJECT_DIRECTORY

现在是时候创建**Podfile**了，Podfile文件描述了我们将要下载到CocoaPods里面的库文件。最简单的方式就是就是使用下面的命令让CocoaPods为我们生成一个空Podfile::

> pod init

运行完命令，工程根目录会产生一个名为`Podfile`的文件。用编辑器打开(最好别用自带的TextEdit)，然后把下面的内容填入文件中:

```shell
use_frameworks!

target 'NotesDB' do
    pod "SwiftyDB"
end
```

![t50_4_podfile](http://www.appcoda.com/wp-content/uploads/2016/03/t50_4_podfile.png)


`pod "SwiftDB"`这一行代码是真正有用的。通过那行代码CocoaPods就会下载SwiftDB库和其所有的依赖库，同时还会在Xcode的工作目录中创建多个子目录。

编辑完Podfile，保存关闭。这时确保你已经在Xcode里关了之前打开的初始工程，回到终端输入下面的命令:

> pod install


![t50_5_pod_install_2](http://www.appcoda.com/wp-content/uploads/2016/03/t50_5_pod_install_2.png)


稍等片刻，然后准备开始吧。这次不要打开之前的初始工程，直接用Xcode打开`NotesDB.xcworkspace`。

![](http://www.appcoda.com/wp-content/uploads/2016/03/t50_6_folder_after_installation.png)

## 从SwiftyDB开始 – 我们的Model层

在`NotesDB`工程里有一个名为`Note.swift`的空文件。这个文件就是我们今天的入手点，现在我们要创建几个类用来表示笔记的实体。在理论层面上，接下来要做的就是iOS`MVC`模式的`Model`层的工作。

首先要做的就是引入SwiftDB库，如你所想，在文件的最开头加入如下代码:

```swift
import SwiftDB
```

现在，定义这个项目中最核心的类:

```swift
class Note: NSObject, Storable{
  
}
```

使用SwiftDB的时候有下面二个特殊规则，在上面的类定义中你也能看到:

1. 带有属性的类要用SwiftDB将其存储在数据库中，这个类必须继承`NSObject`。
2. 带有属性的类要用SwiftDB将其存储在数据库中，这个类必须适配`Storable`协议(是SwiftDB的协议)。

现在可以考虑在类中添加属性了，针对添加属性SwiftDB有一个新的规则:为了能在数据库中取出数据时加载整个`Note`对象，定义的属性`datatypes`(数据类型)必须是这个[列表](http://oyvindkg.github.io/swiftydb/#howToRetrieveObjects)中已有的类型，不能用简单的数据类型(数组字典)。如果有“不兼容”的数据类型，我们需要使用额外的方法来转换成对应的对应的数据(稍后详述)。SwiftDB默认这样的数据类型是不会被存入数据库的，也不会有对应的数据表字段。同时，其他任何类中我们想存入数据库的属性我们都需要区别对待。



最后还有一个规则，适配了`Storable`协议的的类必须实现`init`方法:

```swift
class Note: NSObject, Storable {
    
    override required init() {
        super.init()
        
    }
}
```

现在我们已经有了所有该有的信息，开始定义类属性吧。现在并不定义好所有的属性，因为还有一些属性后期需要讨论一下。所以，这里都是基本的属性:

```swift
class Note: NSObject, Storable {
    let database: SwiftyDB! = SwiftyDB(databaseName: "notes")
    var noteID: NSNumber!
    var title: String!    
    var text: String!    
    var textColor: NSData!    
    var fontName: String!    
    var fontSize: NSNumber!    
    var creationDate: NSDate!    
    var modificationDate: NSDate!
    
    ...
}
```

除了第一个，其他不需要额外的注释。第一句代码的作用就是，当对象初始化的时候，如果数据库(名为`notes.sqlite`)不存在就会创建一个，同时会自动创建一个数据表。数据表的属性与类中定义的属性数据类型相匹配。此外，如果数据库存在，就会直接打开它。。

你可能注意到了，上面我们定义的笔记的各种属性就是描述的要保存到笔记里面的各种元素(标题，正文，文本颜色，字体名字与字号，创建与修改日期)，但是没有任何笔记存储图片的相关信息。这是故意这么做的，因为我们需要创建新的图片类，只存储二个属性，图片数据和图片名称。


接下来，还是打开`Notes.swift`文件，在已有代码的上面或者下面，创建下面的类:

```swift
class ImageDescriptor: NSObject, NSCoding {
    var frameData: NSData!    
    var imageName: String!
}
```
