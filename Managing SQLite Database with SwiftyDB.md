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