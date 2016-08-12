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

输出之前，一个发票会生成所需要填写的一些变量值，有些是可以在视图控制器中手动设置的，有些事自动生成的，还有一些是硬编码，写死的。在demo app中需要手动设置的值有:

* 接收者信息，上面表格中的灰色区域。
* 发票条目, 每个条目都包含两部分: 所提供服务的描述与价格。为了方便起见，这里不考虑增值税。使用底部的加号按钮来添加条目。

自动生成的值有:

* 发票编号(导航栏的标题数字)。
* 发票总价(底部工具栏的左侧显示).

最后是发票值中的硬编码部分:

* 发送者信息。
* 发票开具日期(设为空了如果你需要的话自行设置)。
* 支付方法。
* 发票的logo。

关于发票的条目，工程中有一个叫AddItemViewController的视图控制器可以用来输入简单的数据，有两个文本输入框和一个保存按钮，点击保存按钮就会添加条目到发票中，并返回到前一视图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_3_add_item_viewcontroller.png)

所有发票条目都添加进了一个字典数组中，每个字典包含两个值：描述与价格。将这个数组作为CreatorViewController中tableview的数据源，列出所有的条目。每当保存一个发票时，手动添加与自动生成的数据都会添加到字典中，并将一些数据返回到了InvoiceListViewController中，返回了如下这些值:

* 发票编号(字符串)
* 接收者信息(字符串)
* 总价(字符串)
* 发票条目(字典数组)


对于现有的代码乏善可陈，你需要做的就是看懂每个视图控制器中的流程与实现的细节。我想提的一点就是AppDelegate.swift文件，这里有三个方法：一个用于快速访问应用的delegate，一个用于得到文件路径，一个用于将总价用包含货币符号的字符串表示。这些方法已经在开始工程中用到了，之后还会使用它们。在AppDelegate中有一个currencyCode属性，默认设置的是“eur”(欧元)，可以根据自行情况设置。

点击InvoiceListViewController中列表中的一个发票，这个发票数据会传给PreviewViewController，在这里有一个预览发票的webview，预览被渲染成HTML文档的效果，有一个输出成PDF的按钮。这些函数在开始工程中没有，需要之后来实现它们，数据都是现成的可以直接使用。

##HTML模板文件

接着我们要使用HTML模板来初始化一个发票，然后最终的HTML代码会输出成PDF文件。这里的主要逻辑就是先用占位符去填充HTML文件中的一些位置，然后再用真实的数据去替换，因此需要产生一个自定义的HTML表单来生成最终的结果。鉴于这篇教程的目的我们不会去创建任何发票的HTML模板，使用一个现成的模板(特别感谢作者)，并稍微修改了一下。

在开始工程中有三个HTML文件:

* invoice.html
* last_item.html
* single_item.html

第一个文件中包含产生除了条目外的整个发票代码。还有两个与条目有关的模板: single_item.html用来显示除了最后一个条目外的条目，last_item.html仅用来表示最后一个条目，这是因为最后一行的下面是有边框线的。

HTML模板文件中的占位符是被#号包含的特殊关键字。比如下面这段中有发票编号、签发日期与支付日期的占位符:

````html
<td> Invoice #: #INVOICE_NUMBER<br>
#INVOICE_DATE#<br>
#DUE_DATE# </td>
````

>注意: 虽然支付日期(due date)用了占位符，不过我们并不使用它，只用一个空字符串替换它，若你需要的话请自行更改。

可以在这三个HTML文件中发现很多占位符:

* \#LOGO_IMAGE\#
* \#INVOICE_NUMBER\#
* \#INVOICE_DATE\#
* \#DUE_DATE\#
* \#SENDER_INFO\#
* \#RECIPIENT_INFO\#
* \#PAYMENT_METHOD\#
* \#ITEMS\#
* \#TOTAL_AMOUNT\#
* \#ITEM_DESC\#
* \#PRICE\#

最后两个占位符只在single_item.html和last_item.html文件中有，#ITEMS#占位符被替换后每个item会使用其他2各HTML模板文件来创建各自的代码。

如你所见，准备多个HTML模板是为了创建自定义输出的表格，这并不难。进行完整个过程后你会发现，基于这些模板生成实际内容并提取成PDF文件是很简便并高效的。

##组合内容

在熟悉了demo app与发票模板后，是时候来开始实现剩下所有关键点了。一开始，在第一个视图控制器(InvoiceListViewController)中使用HTML模板来生成一个发票的实际HTML内容，搞定这个后在PreviewViewController已存在的web view中加载生成的HTML代码并显示出来，验证一下结果。

这一部分最主要与关键的任务就是在发票的HTML模板中使用真实值替换掉占位符。这些数据是对应在InvoiceListViewController中所选择的发票数据，传递给PreviewViewController。后面你会看到替换占位符是个很简单的任务，在那之前让我们来创建一个新的类，用于生成真实的HTML内容与PDF格式的输出。创建一个新的Cocoa Touch类，使其继承自NSObject类，命名为InvoiceComposer，一路Next完成创建。

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_4_create_invoice_composer.png)

打开已存在的InvoiceComposer.swift文件，从声明一些属性开始 (包含常量与变量):

````swift
class InvoiceComposer: NSObject {
 
    let pathToInvoiceHTMLTemplate = NSBundle.mainBundle().pathForResource("invoice", ofType: "html")
 
    let pathToSingleItemHTMLTemplate = NSBundle.mainBundle().pathForResource("single_item", ofType: "html")
 
    let pathToLastItemHTMLTemplate = NSBundle.mainBundle().pathForResource("last_item", ofType: "html")
 
    let senderInfo = "Gabriel Theodoropoulos<br>123 Somewhere Str.<br>10000 - MyCity<br>MyCountry"
 
    let dueDate = ""
 
    let paymentMethod = "Wire Transfer"
 
    let logoImageURL = "http://www.appcoda.com/wp-content/uploads/2015/12/blog-logo-dark-400.png"
 
    var invoiceNumber: String!
 
    var pdfFilename: String!
 
}
````

前三个属性(pathToInvoiceHTMLTemplate, pathToSingleItemHTMLTemplate, pathToLastItemHTMLTemplate)指定了HTML模板文件的路径，当我们需要打开模板代码并修改时使用这个属性就会很方便。

如我之前所说，这个demo没有提供设置全部发票参数的选项(senderInfo, dueDate, paymentMethod, logoImageURL)，这些会被设置为硬编码，不过在一个真正的app中这些值应该能被用户设置与修改。最后一个要说的是我已经设置好了发票logo所使用的图片链接。你可以随便修改上述的属性(比如在senderInfo中填入你自己的信息)。

最后的invoiceNumber属性会保存发票的编号，并且在编辑发票的过程中一直是可见的。pdfFilename属性包含PDF文件的路径，这是之后需要的，现在不用管，只需声明即可。

除了上述这些属性，还需要添加默认的init()方法:

````swift
class InvoiceComposer: NSObject {
 
    ...
 
    override init() {
        super.init()
    }
}
````

现在来创建一个处理替换HTML模板文件中占位符的新方法，命名为renderInvoice，这里是需要的参数:

````swift
func renderInvoice(invoiceNumber: String, invoiceDate: String, recipientInfo: String, items: [[String: String]], totalAmount: String) -> String! {
 
}
````
这些参数是在demo中创建一个发票时需要手动设置的，也是生成发票时所需要的数据。返回的字符串值是包含实际内容的最终的HTML代码。

来实现这个方法并进行第一项重要的任务。在下面的代码段中有两个关键的地方:首先是将invoice.html文件中的模板内容加载在字符串变量中以便修改。另一个是替换掉除了发票条目外的占位符，注释会帮助你理解:
```swift
func renderInvoice(invoiceNumber: String, invoiceDate: String, recipientInfo: String, items: [[String: String]], totalAmount: String) -> String! {
    // Store the invoice number for future use.
    self.invoiceNumber = invoiceNumber
 
    do {
        // Load the invoice HTML template code into a String variable.
        var HTMLContent = try String(contentsOfFile: pathToInvoiceHTMLTemplate!)
 
        // Replace all the placeholders with real values except for the items.
        // The logo image.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#LOGO_IMAGE#", withString: logoImageURL)
 
        // Invoice number.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#INVOICE_NUMBER#", withString: invoiceNumber)
 
        // Invoice date.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#INVOICE_DATE#", withString: invoiceDate)
 
        // Due date (we leave it blank by default).
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#DUE_DATE#", withString: dueDate)
 
        // Sender info.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#SENDER_INFO#", withString: senderInfo)
 
        // Recipient info.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#RECIPIENT_INFO#", withString: recipientInfo.stringByReplacingOccurrencesOfString("\n", withString: "<br>"))
 
        // Payment method.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#PAYMENT_METHOD#", withString: paymentMethod)
 
        // Total amount.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#TOTAL_AMOUNT#", withString: totalAmount)
 
    }
    catch {
        print("Unable to open and use HTML template files.")
    }
 
    return nil
}
```
替换占位符的方法如上面那样简单，使用stringByReplacingOccurrencesOfString(...)字符串方法即可，将placeholder作为第一个参数，实际值作为第二个参数。替换大量的占位符可能有些乏味，不过毕竟这样做不难。

注意到所有工作都在一个do-catch语句中进行，从初始化文件路径的HTMLContent字符串开始，若出错了则返回nil，目前不会返回包含实际值的HTML内容，后面再说。

来关注发票条目的设置，由于它们的数量是可变的，因此我们使用一个循环来处理。对于除了最后一个的所有条目，打开single\_item.html模板替换其中的占位符。对于最后一个条目，使用last\_item.html模板，因为底部存在一条线。最终的HTML代码会添加到另一个字符串中(后面的allItems变量)。当这个字符串与所有条目的详细信息整合在一起后，会被用来替换HTMLContent字符串中的#ITEMS#占位符，在方法的最后会返回这个字符串。

在`do`的body中添加代码，如下:
```swift
func renderInvoice(invoiceNumber: String, invoiceDate: String, recipientInfo: String, items: [[String: String]], totalAmount: String) -> String! {
     ...
 
    do {
        ...       
 
        // The invoice items will be added by using a loop.
        var allItems = ""
 
        // For all the items except for the last one we'll use the "single_item.html" template.
        // For the last one we'll use the "last_item.html" template.
        for i in 0..&lt;items.count {
            var itemHTMLContent: String!
 
            // Determine the proper template file.
            if i != items.count - 1 {
                itemHTMLContent = try String(contentsOfFile: pathToSingleItemHTMLTemplate!)
            }
            else {
                itemHTMLContent = try String(contentsOfFile: pathToLastItemHTMLTemplate!)
            }
 
            // Replace the description and price placeholders with the actual values.
            itemHTMLContent = itemHTMLContent.stringByReplacingOccurrencesOfString("#ITEM_DESC#", withString: items[i]["item"]!)
 
            // Format each item's price as a currency value.
            let formattedPrice = AppDelegate.getAppDelegate().getStringValueFormattedAsCurrency(items[i]["price"]!)
            itemHTMLContent = itemHTMLContent.stringByReplacingOccurrencesOfString("#PRICE#", withString: formattedPrice)
 
            // Add the item's HTML code to the general items string.
            allItems += itemHTMLContent
        }
 
        // Set the items.
        HTMLContent = HTMLContent.stringByReplacingOccurrencesOfString("#ITEMS#", withString: allItems)
 
        // The HTML code is ready.
        return HTMLContent
    }
    catch {
        print("Unable to open and use HTML template files.")
    }
 
    return nil
}
```
