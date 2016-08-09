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

再输出之前，一个发票会生成所需要填写的一些变量值，有些是可以在视图控制器中手动设置的，有些事自动生成的，还有一些是硬编码，写死的。在demo app中需要手动设置的值有:

* 接收者信息，上面表格中的灰色区域。
* 发票条目, 每个条目都包含两部分: 所提供服务的描述与价格。为了方便起见，这里不考虑增值税。使用底部的加号按钮来添加条目。

自动生成的值有:

* 发票编码(导航栏的标题数字)。
* 发票总价(底部工具栏的左侧显示).

最后是发票值中的硬编码部分:

* 发送者信息。
* 发票开具日期(设为空了如果你需要的话自行设置)。
* 支付方法。
* 发票的logo。

关于发票的条目，工程中有一个叫AddItemViewController的视图控制器可以用来输入简单的数据，有两个文本输入框和一个保存按钮，点击保存按钮就会添加条目到发票中，并返回到前一视图、:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_3_add_item_viewcontroller.png)

所有发票条目都添加进了一个字典数组中，每个字典包含两个值：描述与价格。将这个数组作为CreatorViewController中tableview的数据源，列出所有的条目。每当保存一个发票时，手动添加与自动生成的数据都会添加到字典中，并将一些数据返回到了InvoiceListViewController中，返回了如下这些值:

* 发票编码(字符串)
* 接收者信息(字符串)
* 总价(字符串)
* 发票条目(字典数组).


对于现有的代码乏善可陈，你需要做的就是看懂每个视图控制器中的流程与实现的细节。我想提的一点就是AppDelegate.swift文件，这里有三个方法：一个用于快速访问应用的delegate，一个用于得到文件路径，一个用于将总价用包含货币符号的字符串表示。这些方法已经在开始工程中用到了，之后还会使用它们。在AppDelegate中有一个currencyCode属性，默认设置的是“eur”(欧元)，可以根据自行情况设置。

点击InvoiceListViewController中列表中的一个发票，这个发票数据会传给PreviewViewController，在这里有一个预览发票的webview，预览被渲染成HTML文档的效果，有一个输出成PDF的按钮。这些函数在开始工程中没有，需要之后来实现它们，数据都是现成的可以直接使用。

##HTML模板文件
