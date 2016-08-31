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

你可能注意到了，上面我们定义的笔记的各种属性就是描述的要保存到笔记里面的各种元素(标题，正文，文本颜色，字体名字与字号，创建与修改日期)，但是没有任何笔记存储图片的相关信息。这是故意这么做的，因为我们需要创建新的图片类，只存储二个属性，图片尺寸和图片名称。


接下来，还是打开`Notes.swift`文件，在已有代码的上面或者下面，创建下面的类:

```swift
class ImageDescriptor: NSObject, NSCoding {
    var frameData: NSData!    
    var imageName: String!
}
```

注意表示尺寸的数据类型是用`NSData`而不是`CGRect`来表示的。这样做很有必要，后期向数据库存储数据的时候就相对简单些。针对类要适配`NSCoding`协议这个事，在你明白如何做的同时，也要明白为什么要这样做。

回到`Note`类，如下所示，在类中定义一个`ImageDescriptor`数组:

```swift
class Note: NSObject, Storable {
    ...    
    
    var images: [ImageDescriptor]!
    
    ...
}
```

但是现在有一个问题就是，我们应该怎么把用`ImageDescriptor`对象来表示的`image`图片数组转换成`imagesData`的`NSData`对象？方法就是是使用`NSKeyedArchiver`类把`images`数组打包成一个`NSDate`对象。之后会给出详细的实现代码，现在我们需要继续回去写`ImageDescripter`类中其他的代码。

如你所知，可以被打包归档(其他语言也叫`序列化`)的类，其类中的所有属性也必须是可序列化的，在我们定义的类满足条件，`ImageDescripter`类中定义的二种数据类型(`NSData`和`String`)都可以被序列化。但这还不够，还要对其进行`encode(编码)`和`decode(解码)`，以便能在合适的时候能打包和解包，这就是为什么类需要适配`NSCoding`协议。通过这个协议我们实现下面的几个方法(其中一个是`init`方法)，就可以正确的编码和解码类中的二个属性了:

```swift
class ImageDescriptor: NSObject, NSCoding {
    ...
    
    required init?(coder aDecoder: NSCoder) {
        frameData = aDecoder.decodeObjectForKey("frameData") as! NSData
        imageName = aDecoder.decodeObjectForKey("imageName") as! String
    }
    
    func encodeWithCoder(aCoder: NSCoder) {
        aCoder.encodeObject(frameData, forKey: "frameData")
        aCoder.encodeObject(imageName, forKey: "imageName")
    }
}
```

想更多了解`NSCoding`协议和`NSKeydArchiver`类，可以看下**[这里](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Protocols/NSCoding_Protocol/)**和**[这里](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSKeyedArchiver_Class/)**，在这里对此不做过多讨论。

除了上边那些，再定义一个便捷的`init`方法。方法很简单，不需要注释:

```swift
class ImageDescriptor: NSObject, NSCoding {
    ...    
    
    init(frameData: NSData!, imageName: String!) {
        super.init()
        self.frameData = frameData
        self.imageName = imageName
    }
}
```

到此，我们和SwiftDB的第一个照面就打完了。即使没做太多的SwiftDB相关工作，但这部分也非常重要，三点原因:

1. 创建一个需要使用SwiftDB功能的类。
2. 了解使用SwiftDB时的一些规则。
3. 知道更多关于使用SwiftDB时对存入数据库的数据类型的限制。

`提示`: 如果现在在Xcode中有些错误，再构建(Command - B)一次工程去掉这些错误。

## 主键和忽略属性

处理数据库的时候推荐使用`primary keys(主键)`，这些键能帮你唯一确定一条数据表中的记录，同时各种操作也依赖主键(如更新一条特定的数据记录)。可以在[这里](http://databases.about.com/cs/administration/g/primarykey.htm)查阅主键的含义。

在SwiftDB中用类中的一个或者多个属性为数据表指定主键非常简单。SwiftDB提供了`PrimaryKeys`协议，所有想要在数据库里各数据表中用主键标识唯一类对象的类都可以实现这个协议。

方法非常简单而且很标准，下面直接开始:

在`NotesDB`工程里找到`Extensions.swift`文件，工程导航栏中点击打开这个文件，加入下面的代码:

```swift
extension Note: PrimaryKeys {
    class func primaryKeys() -> Set<String> {
        return ["noteID"]
    }
}
```

在这个demo里，我们要`noteID`在sqlite数据库中各数据里做为唯一的主键。如果需要更多的主键，只需要写在一行用逗号隔开即可。

还有，并不是所有的属性都需要存储在数据库里，需要显式的告知SwiftDB不要包含那些属性。例如，在`Note`类中有二个属性不需要存储在数据库中(可能是属性可能存储在数据中，也可能是我们不想存入数据库):`images`数组和`database`数据库实例对象。如何显式排除这二个属性？通过实现另一个SwiftDB提供的叫`IgnoredProperties`的协议来做:

```swift
extension Note: IgnoredProperties {
    class func ignoredProperties() -> Set<String> {
        return ["images", "database"]
    }
}
```

如果还有更多的属性不想存入数据库，只需要把属性加入上面的列表即可。例如，还有下面这个属性:

```swift
var noteAuthor:String!
```

我们不想把这个属性存入数据库，只需要把这个属性加进`IgnoredProperties`协议实现里即可:

```swift
extension Note: IgnoredProperties {
    class func ignoredProperties() -> Set<String> {
        return ["images", "database", "noteAuthor"]
    }
}
```

`提示`:如果发现有错误，`MARKDOWN_HASH6211c316cc840902a4df44c828a26fbeMARKDOWN_HASH`库引入到`MARKDOWN_HASH1dbda56f2122b1744ebf59bb64bbffdfMARKDOWN_HASH`文件中。



## 保存新笔记

现在已经完成了`Note`类的最基本实现，是时候开始demo应用的功能实现了。目前在新定义的类中没添加任何方法;我们按缺失的功能一步一步的来实现。

首先要有笔记，因此，app需要知道如何用SwiftDB和我们新建的二个类来保存笔记。这些主要都是在`EditNoteViewController`类中实现的，是时候打开相应的文件了。写代码之前，我把我认为很重要的属性都列在这儿了:

- `imageView`:这是个数组，保存了被入笔记中的图片。不要忘了这个数组;之后有了这个数组写代码就便捷多了。
- `currentFontName`:保存当前文本区使用的字体名称。
- `currentFontSize`:当前文本区字体大小。
- `editNoteID`:`noteID`(主键)，用于更新笔记，之后会用到。

由于公用的功能已经在初始工程里写好了，所以下面就是来实现里面缺失的`saveNote`方法了。先做两件事: 第一，笔记如果没有标题或者正文不允许保存。第二，忽略保存笔记的时候出现的键盘。

```swift
func saveNote() {
    if txtTitle.text?.characters.count == 0 || tvNote.text.characters.count == 0 {
        return
    }
    
    if tvNote.isFirstResponder() {
        tvNote.resignFirstResponder()
    }   
}
```

继续初始化一个新`Note`对象，并为对象内属性赋值。图片需要区别对待，之后会马上处理。

```swift
func saveNote() {
    ...
    
    let note = Note()
    note.noteID = Int(NSDate().timeIntervalSince1970)
    note.creationDate = NSDate()
    note.title = txtTitle.text
    note.text = tvNote.text!
    note.textColor = NSKeyedArchiver.archivedDataWithRootObject(tvNote.textColor!)
    note.fontName = tvNote.font?.fontName
    note.fontSize = tvNote.font?.pointSize
    note.modificationDate = NSDate()    
}
```

一些注释:

- `noteID`属性期望任意的整型数字来做为主键。你可以创建或者生成任意的整型值，要足够长确保唯一。这里我们直接使用当前时间戳的一部分做为主键，但是在真正的应用程序里这样做并不是很好的方法，因为时间戳里包含了太多的数字。在我们的demo程序里还好，只是简单的用int值。
- 当我们第一次保存笔记的时候，同时设置笔记创建时间和修改时间为当前时间戳(用`NSDate`对象表示)。
- 唯一需要我们做一下特殊的转换的就是，从`NSData`对象中取出当前文本区的文本颜色。文本颜色的对象值是通过`NSKeyedArchiver`类打包过的。

现在重点看一下怎样保存图片。我们需要写一个新方法来填充图片数组。方法里做二个事情:保存真正的图片到应用程序的就文稿(documents)目录中，同时为每个图片创建相应的`ImageDecripter`对象。这些对象都会被放在`images`数组里。



为创建这个新的方法，我们需要迂回一下，再次回到`Note.swift`文件。先看具体实现，后面再具体讨论细节。(原文排版有误)

```swift
func storeNoteImagesFromImageViews(imageViews: [PanningImageView]) {
	if imageViews.count > 0 {
		if images == nil {
			images = ImageDescriptor
		} else {
			images.removeAll()
		}


    	for i in 0..<imageViews.count {
        	let imageView = imageViews[i]
         	let imageName = "img_\(Int(NSDate().timeIntervalSince1970))_\(i)"


        	images.append(ImageDescriptor(frameData: imageView.frame.toNSData(), imageName: imageName))

        	Helper.saveImage(imageView.image!, withName: imageName)
    	}

    	imagesData = NSKeyedArchiver.archivedDataWithRootObject(images)
	} else {
    	imagesData = NSKeyedArchiver.archivedDataWithRootObject(NSNull())
}
```



这里解释为什么要写上面这个方法:

1. 最开始检查`images`数组是否已经初始化。如果为空，初始化数组，非空则移除内部所有数据。第二步在我们更新一个已存在的笔记的时候非常有用。
2. 针对每个图片创建唯一的名称。图片名称类似于:“img_12345679\_1”。
3. 传入图片尺寸和图片名称参数给我们自定义的构造方法来初始化一个`ImageDescripter`对象。`toNSData()`方法已经做为`CGRect`的扩展实现，可以`Extensions.swift`中找到。作用就是转换尺寸为`NSData`对象。`ImageDecripter`初始化完成后，直接放入`images`数组中。
4. 保存真正的图片到应用程序的文稿目录中。`saveImage(_:withName:)`类方法在`Helper.swift`中实现，那里还有更多有用的方法。
5. 最后所有的图片都处理完成，就把`images`数组通过打包转换成`NSData`对象，并赋值给`imagesData`属性。最后一行代码才是`ImageDescripeter`类适配`NSCoding`协议的真正意义所在。

`else`分支看似无用，其实是必需的。默认情况笔记中加图片时`imagesData`就是空值。但是SQLite并不能识别“nil”值是空值。SQLite只认`NSNull`，所以我们需要转为`NSData`。

回到`EditNoteViewController.swift`文件，加入下面的我们刚刚创建的方法:

```swift
func saveNote() {
    ...
    
    note.storeNoteImagesFromImageViews(imageViews)
}
```

现在回到`Note.swift`文件中，实现真正向数据库中存入笔记数据的方法。此时有些重要的东西需要你了解一下: SwiftDB提供了选项来执行任意关系型数据库的同步或者异步操作。使用哪种方式取决于你的app的类型。我建议使用异步的方式，因为这样在处理数据库操作的时候不会对主线程加锁，也不会锁定UI而影响用户体验。但是重申一下，用什么方法，这一切都取决于你自己。

这里我们使用异步的方式把笔记数据保存到数据库。如你所见，SwiftDB里的方法包含了一个返回操作结果的闭包。返回的结果对象的更详细信息可以看[这里，我建议你现在就看。

现在看看我们一直在讨论的这个新方法:

```swift
func saveNote(shouldUpdate: Bool = false, completionHandler: (success: Bool) -> Void) {
    database.asyncAddObject(self, update: shouldUpdate) { (result) -> Void in
        if let error = result.error {
            print(error)
            completionHandler(success: false)
        }
        else {
            completionHandler(success: true)
        }
    }
}
```

上面的代码很简单，这个方法同时也用来更新笔记。通过事先设置`shouldUpdate`这个布尔值，`asyncDataObject(...)`方法根据这个值来确定是创建一个新的数据库记录还是更新已有的笔记记录。

还有，你看方法中第二个参数，是完成时的处理函数。我需要用合适的参数在笔记是否保存完成后调用这个方法。建议你在后台有异步的任务执行的时候一直用完成的回调处理函数。这样的话，在你后台任务执行完成的时候，你就能通知调用者的函数，那时就可以做任意的数据转换，或者数据备份。

上面你所看到的在其他关系型数据库也能看到。我们应该总是对结果进行错误检查，无论是否有错误我们都应该检查。上面的例子如果发生错误，调用完成处理函数的时候就需要传`false`值，意思就是笔记保存失败。反之就是传`true`表示保存操作成功执行。

再次回到`EditNoteViewController`类，完成`saveNote()`方法。我们调用一下刚写好的方法，如果保存成功则弹出一个窗口，如果发生错误就显示一条错误信息。



```swift
func saveNote() {
    ...
    
    let shouldUpdate = (editedNoteID == nil) ? false : true
    
    note.saveNote(shouldUpdate) { (success) -> Void in
        dispatch_async(dispatch_get_main_queue(), { () -> Void in
            if success {
                self.navigationController?.popViewControllerAnimated(true)
            }
            else {
                let alertController = UIAlertController(title: "NotesDB", message: "An error occurred and the note could not be saved.", preferredStyle: UIAlertControllerStyle.Alert)
                alertController.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: { (action) -> Void in
                    
                }))
                self.presentViewController(alertController, animated: true, completion: nil)
            }
        })
    }
}
```

注意上面实现中的`shouldUpdate`变量。由`editedNoteID`是否为空来决定其值。意思是笔记是否是被更新。

这时候你可以跑一下程序并试着保存一下笔记。如果你是按教程一步一步做到这步，那保存笔记应该不会出任何问题。