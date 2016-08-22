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
