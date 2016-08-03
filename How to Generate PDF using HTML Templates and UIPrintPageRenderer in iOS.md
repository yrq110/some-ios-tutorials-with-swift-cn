#How to Generate PDF using HTML Templates and UIPrintPageRenderer in iOS
##如何使用HTML模板与UIPrintPageRenderer生成PDF

[原文地址](http://www.appcoda.com/pdf-generation-ios/)

你曾被要求过生成包含app内容的PDF文档吗？

你想过怎么去做吗，或者说之前从没做过？

好吧，从提问开始可能有点非主流，不过上述的问题正是在这篇教程中要讨论的。生成一个iOS app的PDF文档的想法听起来挺无可救药的，实际上并不是。作为一个机智的开发者，总会找到最适合自己的方法并且是以最小的开支去实现目标。我不得不承认手工绘制一个PDF文档挺痛苦的，而且最后可能成为一项非生产性的任务。计算点，添加边，设置颜色，偏移等等，也许会很有趣，不过如果需要绘制的文档很复杂的话事情就会变得很麻烦。

目标是通过一个完全不同的方法来创建PDF文档，比手工绘制更加方便。这个方法是基于HTML模板的，大致可以简化为如下步骤:
1. 创建生成PDF所需的表单与content模板
2. 使用HTML模板产生content(也可用web view显示出来)
3. 输出PDF

在最后一步，iOS会替你搞定所有麻烦事。

##开始工程

使用一个发票生成器工具demo来进行教程中的演示。开始前请先下载[开始工程](https://github.com/appcoda/Print2PDF/blob/master/Starter_Project.zip?raw=true)然后用Xcode打开。

在工程中已经完成了一些必要的工作，InvoiceListViewController用来显示app中创建与保存的的发票列表，在这里通过右上角的加号按钮可以创建一个新的发票。点击任一列表项会进入发票信息的预览界面，可以输出成PDF格式的文档，注意，这部分功能还没有在开始工程中实现，需要我们在接下来完成它。还有，向左拖动列表项会出现删除选项，如下图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_1_invoice_list_viewcontroller.png)

前面说过，可以通过点击加号按钮创建新发票，会跳转到另一个视图控制器-CreatorViewController，如下图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_2_creator_viewcontroller.png)
