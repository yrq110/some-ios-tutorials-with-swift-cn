#How to Generate PDF using HTML Templates and UIPrintPageRenderer in iOS
##如何使用HTML模板与UIPrintPageRenderer生成PDF

[原文地址](http://www.appcoda.com/pdf-generation-ios/)

你曾被要求过生成包含app内容的PDF文档吗？

你想过怎么去做吗，或者说之前从没做过？

好吧，从提问开始可能有点非主流，不过上述的问题正是在这篇教程中需要讨论的。生成一个iOS app的PDF文档的想法听起来挺无可救药的，实际上并不是。作为一个机智的开发者，总会找到最适合自己的方法并且是以最小的开支去实现目标。我不得不承认手工绘制一个PDF文档挺痛苦的，而且最后可能成为一项非生产性的任务。计算点，添加边，设置颜色，偏移等等，也许会很有趣，不过如果需要绘制的文档很复杂的话事情就会变得很麻烦。

目标是通过一个完全不同的方法来创建PDF文档，比手工绘制更加方便。这个方法是基于HTML模板的，大致可以简化为如下步骤:
* 创建打印PDF所需的表单与内容模板
* 使用HTML模板产生实际内容(也可用web view显示出来)
* 打印成PDF
