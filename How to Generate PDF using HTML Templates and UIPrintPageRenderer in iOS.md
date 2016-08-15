#How to Generate PDF using HTML Templates and UIPrintPageRenderer in iOS
##如何使用HTML模板与UIPrintPageRenderer生成PDF

***

>* 原文链接 : [How to Generate PDF using HTML Templates and UIPrintPageRenderer in iOS](http://www.appcoda.com/pdf-generation-ios/)
* 原文作者 : [GABRIEL THEODOROPOULOS](http://www.appcoda.com/author/gabrielth/)
* 译者 : [yrq110](https://github.com/yrq110)

***

你曾被要求过生成包含app内容的PDF文档吗？

你想过怎么去做吗，或者说之前从没做过？

好吧，从提问开始可能有点非主流，不过上述的问题正是在这篇教程中要讨论的。生成一个iOS app的PDF文档的想法听起来挺无可救药的，实际上并不是。作为一个机智的开发者，总会找到最适合自己的方法并且是以最小的开支去实现目标。我不得不承认手工绘制一个PDF文档挺痛苦的，而且最后可能成为一项非生产性的任务。计算点，添加边，设置颜色，偏移等等，也许会很有趣，不过如果需要绘制的文档很复杂的话事情就会变得很麻烦。

目标是通过一个完全不同的方法来创建PDF文档，比手工绘制更加方便。这个方法是基于HTML模板的，大致可以简化为如下步骤:

1. 创建生成PDF所需的表单与content模板
2. 使用HTML模板产生content(也可用web view显示出来)
3. 导出PDF

在最后一步，iOS会替你搞定所有麻烦事。

**目录**
* [开始工程](#开始工程)
* [HTML模板文件](#html-template)
* [组合内容](#组合内容)
* [预览HTML](#html-preview)
* [准备导出](#准备导出)
* [生成PDF](#pdf-output)
* [绘制自定义页眉和页脚](#绘制自定义页眉和页脚)

##开始工程

使用一个发票生成器工具demo来进行教程中的演示。开始前请先下载[开始工程](https://github.com/appcoda/Print2PDF/blob/master/Starter_Project.zip?raw=true)然后用Xcode打开。

在工程中已经完成了一些必要的工作，InvoiceListViewController用来显示app中创建与保存的的发票列表，在这里通过右上角的加号按钮可以创建一个新的发票。点击任一列表项会进入发票信息的预览界面，可以导出成PDF格式的文档，注意，这部分功能还没有在开始工程中实现，需要我们在接下来完成它。还有，向左拖动列表项会出现删除选项，如下图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_1_invoice_list_viewcontroller.png)

前面说过，可以通过点击加号按钮创建新发票，会跳转到另一个视图控制器-CreatorViewController，如下图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_2_creator_viewcontroller.png)

导出之前，一个发票会生成所需要填写的一些变量值，有些是可以在视图控制器中手动设置的，有些事自动生成的，还有一些是硬编码，写死的。在demo app中需要手动设置的值有:

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

点击InvoiceListViewController中列表中的一个发票，这个发票数据会传给PreviewViewController，在这里有一个预览发票的webview，预览被渲染成HTML文档的效果，有一个导出成PDF的按钮。这些函数在开始工程中没有，需要之后来实现它们，数据都是现成的可以直接使用。

<a name="html-template"></a>
##HTML模板文件

接着我们要使用HTML模板来初始化一个发票，然后最终的HTML代码会导出成PDF文件。这里的主要逻辑就是先用占位符去填充HTML文件中的一些位置，然后再用真实的数据去替换，因此需要产生一个自定义的HTML表单来生成最终的结果。鉴于这篇教程的目的我们不会去创建任何发票的HTML模板，使用一个现成的模板(特别感谢作者)，并稍微修改了一下。

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

如你所见，准备多个HTML模板是为了创建自定义导出的表格，这并不难。进行完整个过程后你会发现，基于这些模板生成实际内容并提取成PDF文件是很简便并高效的。

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
模板代码在适当的时候会被修改以产生一个包含实际内容的发票，下面我们要利用上述方法的结果做一些事情。

<a name="html-preview"></a>
##预览HTML

在创建发票的实际内容后，是时候来验证之前所做的工作是否达标了，因此，在这一节我们的目标是将HTML字符串加载到PreviewViewController中的web view中看看效果。注意这只是一个可选的步骤，在实际的app中没必要在导出PDF之前去使用一个web view去预览HTML，这里这么做是为了demo app的完整性。

打开PreviewViewController.swift文件，来到类顶部，首先声明一些新属性:
```swift
class PreviewViewController: UIViewController {
 
    ...
 
    var invoiceComposer: InvoiceComposer!
 
    var HTMLContent: String!
 
}
```
第一个对象的类是之前我们创建好的，这里进行一个简单的初始化即可。HTMLContent字符串变量在之后被用来存放时间的HTML代码。

接着按如下步骤创建一个新方法:
1. 初始化invoiceComposer对象。
2. 调用renderInvoice(...)方法生成发票的HTML代码。
3. web view加载HTML。
4. 将返回的HTML字符串分配给HTMLContent属性。

具体实现:
```swift
func createInvoiceAsHTML() {
    invoiceComposer = InvoiceComposer()
    if let invoiceHTML = invoiceComposer.renderInvoice(invoiceInfo["invoiceNumber"] as! String,
                                                       invoiceDate: invoiceInfo["invoiceDate"] as! String,
                                                       recipientInfo: invoiceInfo["recipientInfo"] as! String,
                                                       items: invoiceInfo["items"] as! [[String: String]],
                                                       totalAmount: invoiceInfo["totalAmount"] as! String) {
 
        webPreview.loadHTMLString(invoiceHTML, baseURL: NSURL(string: invoiceComposer.pathToInvoiceHTMLTemplate!)!)
        HTMLContent = invoiceHTML
    }
}
```
没啥难的地方，注意一下renderInvoice(...)方法的输入参数，当我们得到返回的实际HTML字符串后回家再加载web view中。

是时候调用新方法了:
```swift
override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
 
    createInvoiceAsHTML()
}
```

若想看看效果，运行app新建一个发票，在列表中选中点击，看看在web view中出现的发票，如下所示:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_5_invoice_webview.png)

##准备导出

已经搞定一半的工作了，现在来着手将一个发票生成PDF的过程。为了达到目的需要使用一个特殊的类: UIPrintPageRenderer，如果你没听说过它不要紧，这个类是可用于渲染打印内容的一种可靠方式，官方文档在[这里](https://developer.apple.com/library/ios/documentation/iPhone/Reference/UIPrintPageRenderer_Class/)，浏览一下查看更多的信息。

UIPrintPageRenderer类提供了多种绘制方法，一般情况下我们不需要去重写它。那些绘制方法只能被UIPrintPageRenderer的子类中的方法所重写，这些额外的努力是值得的，可以使我们灵活的控制导出的实际页面的内容: 比如页眉和页脚。在demo app中我们需要创建并使用它的子类，因为要绘制自定义的页眉和页脚(同时)。 

回到Xcode中创建一个新的类，需要注意两件事:
1. 设置为UIPrintPageRenderer的子类.
2. 命名为CustomPrintPageRenderer.

准备好了后(已经在工程导航栏中看到CustomPrintPageRenderer.swift文件)，为下面的步骤做一些简单的准备。首先，指定A4页面的宽度和高度(单位为像素)，记住我们是想将发票提取成PDF，并且PDF是可以被正常打印出来的，所以限制页面大小是很重要的。

```swift
class CustomPrintPageRenderer: UIPrintPageRenderer {
 
    let A4PageWidth: CGFloat = 595.2
 
    let A4PageHeight: CGFloat = 841.8
 
}
```

上述值描述了精确的普通A4纸的宽度与高度。

指定纸张框架与CustomPrintPageRenderer类所绘制的打印区域是很必要的。在init()方法中进行这一步，使用上面定义的两个属性:
```swift
override init() {
    super.init()
 
    // Specify the frame of the A4 page.
    let pageFrame = CGRect(x: 0.0, y: 0.0, width: A4PageWidth, height: A4PageHeight)
 
    // Set the page frame.
    self.setValue(NSValue(CGRect: pageFrame), forKey: "paperRect")
 
    // Set the horizontal and vertical insets (that's optional).
    self.setValue(NSValue(CGRect: pageFrame), forKey: "printableRect")
}
```

在上面的代码段中，将纸张框架与打印区域设为了相同的值。然而也许你想给打印区域外面增加一些留白(小于纸张边界)，这样会得到更好的打印效果。这时，可以使用下面这行替换掉上述方法的最后一行代码:
```swift
   self.setValue(NSValue(CGRect: CGRectInset(pageFrame, 10.0, 10.0)), forKey: "printableRect")
```
上述代码在水平和垂直坐标方向各增加了10个像素的偏移，注意，这一节中所做的设置即使没有创建UIPrintPageRenderer的子类也是有效果的，换句话说，不论何时都不要忘记设置打印对象的纸张与打印区域。

<a name="pdf-output"></a>
##生成PDF 

说“打印成PDF”的意思实际上是绘制一些内容到PDF的图形上下文中去，一旦这一步发生后，那么绘制的内容就会被发送到一个打印机或者被保存到一个文件中。我们仅考虑第二种情况，将HTML内容绘制到一个PDF上下文中，接着将得到的结果保存到NSData对象中，最后将这个对象保存到一个文件(后缀.pdf文件)中。这个过程变化很多，不过步骤很简单。

打开InvoiceComposer.swift文件，实现exportHTMLContentToPDF(...)这个仅接受一个参数(提取成PDF所需的HTML内容)的新方法。在实现这个方法之前有必要介绍另一个与打印相关的概念——print formatter(UIPrintFormatter类). 苹果文档中有如下描述:

>UIPrintFormatter is an abstract base class for print formatters: objects that lay out custom printable content that can cross page boundaries. Given a print formatter, the printing system can automate the printing of the type of content associated with the print formatter.

这意味着对于我们来说只需要将HTML内容作为打印格式(print formatter)添加到打印页面的渲染器中，iOS的打印系统就会负责页面实际的布局与打印的内容。建议你看看[官方的这个页面](https://developer.apple.com/reference/uikit/uiprintformatter?language=objc)，说的很详细。有一点需要说明的是虽然UIPrintFormatter是一个抽象类，不过iOS SDK提供了可以利用的具体的子类，比如说UIMarkupTextPrintFormatter，我们需要用这个类来添加HTML内容到打印页面的渲染器对象中去，更多细节与其它的子类可以在上面的官方链接中找到。

说完了，是时候实现新方法了:
```swift
func exportHTMLContentToPDF(HTMLContent: String) {
    let printPageRenderer = CustomPrintPageRenderer()
 
    let printFormatter = UIMarkupTextPrintFormatter(markupText: HTMLContent)    
    printPageRenderer.addPrintFormatter(printFormatter, startingAtPageAtIndex: 0)
 
    let pdfData = drawPDFUsingPrintPageRenderer(printPageRenderer)
 
    pdfFilename = "\(AppDelegate.getAppDelegate().getDocDir())/Invoice\(invoiceNumber).pdf"
    pdfData.writeToFile(pdfFilename, atomically: true)
 
    print(pdfFilename)
}
```
逐步分析下:

* 首先初始化一个CustomPrintPageRenderer对象，使用它进行实际的绘制(也就是打印(printing))。
* 接着初始化一个UIMarkupTextPrintFormatter对象，输入HTML内容作为初始化的参数。
* 将page formatter添加到打印页面的渲染器对象中，addPrintFormatter(...)方法的第二个参数用来指定打印格式执行时的起始页面，这里我们设为0，并且只有1个页面。
* 下面进行实际的PDF绘制，drawPDFUsingPrintPageRenderer(...)是个自定义方法，后面会定义它。绘制PDF的结果会保存到pdfData对象中，pdfData实际上是个NSData对象。
* 将PDF数据保存到一个文件中。首先指定文件路径，同时给予一个合适的文件名(这里使用的是invoiceNumber属性)，然后将PDF数据写入该文件。
* 最后一步可选，打印出文件路径对在Finder中找到新创建的PDF文件是很有帮助的。运行app时，文件路径就会在控制器中打印出来，可以打开PDF进行预览并验证结果。

在一个更加复杂的app中可以使用多种打印格式对象，可以为每个格式指定不同的起始页面，不过现在在demo中不需要那么做，清晰地说明问题即可。

现在来实现自定义方法，执行绘制一个实际PDF上下文的工作。如你所见，下面用到了Core Graphics技术去实现，整个过程不长而且能望码生义:

```swift
func drawPDFUsingPrintPageRenderer(printPageRenderer: UIPrintPageRenderer) -&gt; NSData! {
    let data = NSMutableData()
 
    UIGraphicsBeginPDFContextToData(data, CGRectZero, nil)
 
    UIGraphicsBeginPDFPage()
 
    printPageRenderer.drawPageAtIndex(0, inRect: UIGraphicsGetPDFContextBounds())
 
    UIGraphicsEndPDFContext()
 
    return data
}
```
首先初始化一个可变的数据对象，将导出的PDF数据写入其中。第二行命令代码去创建PDF的图形上下文。接着创建一个新的PDF页面，实际的绘制操作发生在下面这行:
```swift
printPageRenderer.drawPageAtIndex(0, inRect: UIGraphicsGetPDFContextBounds())
```
使用这一行，打印页面的渲染器对象就会在PDF上下文的框架中绘制其内容。注意在drawPageAtIndex(...)调用打印页面渲染器对象的绘制方法时，自定义的页眉与页脚也会被自动绘制出来。

最后关闭PDF图形上下文，返回绘制的最终数据到调用此方法的exportHTMLContentToPDF方法中去。

上述方法打印了一个单页面，若需要打印多个页面，则可以将PDF页面的创建与渲染放入一个循环中，若你想扩展这个demo app或想将多个页面打印到一个PDF文件中的话留心一下这一点。

这时与PDF导出有关的所有工作就告一段落了，下一节将要给你展示如果在打印页面中添加自定义的页眉和页脚，在那之前，先操作一下，看看导出PDF的效果。

打开PreviewViewController.swift文件，定位到exportToPDF(...) IBAction方法中，添加如下代码，这样的话点击工具栏上的PDF按钮时就可以预览将发票提取成PDF后的效果了:

```swift
@IBAction func exportToPDF(sender: AnyObject) {
    invoiceComposer.exportHTMLContentToPDF(HTMLContent)
}
```
现在可以测试一个app，为了快捷我建议使用模拟器。选择一个发票来预览，点击右上角PDF按钮:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_6_pdf_button.png)

如此而为就会打印出PDF，当所有工作完成时，导出文件的路径会打印在控制器中。复制这个路径(不包括文件名)，打开一个新的Finder窗口，使用Shift-Command-G组合键，粘贴路径，则会看到刚刚生成的PDF文件，文件名为发票的编号。

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_7_pdf_in_finder.png)

双击它在Preview app中预览(或选择你喜欢的app):

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_8_pdf_preview-771x1024.png)

##绘制自定义页眉和页脚

现在来稍微扩展一下这个示例，给打印页面添加自定义的页眉和页脚，毕竟这是为何我们最初要创建UIPrintPageRenderer的子类。由于是自定义的，那么这意味着并不是HTML模板的一部分，不与HTML内容一同被渲染。我们想要实现的是在页面的右上角添加“Invoice”字样(作为页眉)，在页面底部添加短句“Thank you!”(页脚)与上方的水平线，目标如下图:

![](http://www.appcoda.com/wp-content/uploads/2016/07/t54_9_pdf_with_header_footer-771x1024.png)

在实现页眉与页脚的细节之前，必须指定期望的页眉与页脚的高度。打开CustomPrintPageRenderer.swift文件，在init()方法中添加如下两行(注意两个属性都来自UIPrintPageRenderer类):

```swift
override init() {
    ...
 
    self.headerHeight = 50.0
    self.footerHeight = 50.0
}
```

从页面的页眉开始入手。重写UIPrintPageRenderer类的方法:

```swift
override func drawHeaderForPageAtIndex(pageIndex: Int, inRect headerRect: CGRect) {
 
}
```
将在函数体内做的步骤如下所述:

1. 指定在页眉中绘制的文本(单词“Invoice”)。
2. 指定文本格式的属性，例如字体、颜色与字母间距离。
3. 计算包含执行格式化后文本的矩形尺寸，指定距页面右侧边界的偏移。
4. 决定文本的绘制起点。
5. 绘制文本(最终)。

来实现一下上述的步骤，有一些注释可以帮助理解:
```swift
override func drawHeaderForPageAtIndex(pageIndex: Int, inRect headerRect: CGRect) {
    // Specify the header text.
    let headerText: NSString = "Invoice"
 
    // Set the desired font.
    let font = UIFont(name: "AmericanTypewriter-Bold", size: 30.0)
 
    // Specify some text attributes we want to apply to the header text.
    let textAttributes = [NSFontAttributeName: font!, NSForegroundColorAttributeName: UIColor(red: 243.0/255, green: 82.0/255.0, blue: 30.0/255.0, alpha: 1.0), NSKernAttributeName: 7.5]
 
    // Calculate the text size.
    let textSize = getTextSize(headerText as String, font: nil, textAttributes: textAttributes)
 
    // Determine the offset to the right side.
    let offsetX: CGFloat = 20.0
 
    // Specify the point that the text drawing should start from.
    let pointX = headerRect.size.width - textSize.width - offsetX
    let pointY = headerRect.size.height/2 - textSize.height/2
 
    // Draw the header text.
    headerText.drawAtPoint(CGPointMake(pointX, pointY), withAttributes: textAttributes)
}
```

上述代码中有一点我没提到，就是getTextSize(...)方法的使用。如你所想，这个自定义方法用来计算并返回文本框架的尺寸。之所以把它写成一个额外的方法是因为在接下来绘制页脚时也需要用到它。

这是getTextSize(...)方法:
```swift
func getTextSize(text: String, font: UIFont!, textAttributes: [String: AnyObject]! = nil) -&gt; CGSize {
    let testLabel = UILabel(frame: CGRectMake(0.0, 0.0, self.paperRect.size.width, footerHeight))
    if let attributes = textAttributes {
        testLabel.attributedText = NSAttributedString(string: text, attributes: attributes)
    }
    else {
        testLabel.text = text
        testLabel.font = font!
    }
 
    testLabel.sizeToFit()
 
    return testLabel.frame.size
}
```
上面这个方法是个计算文本框架尺寸的通用方法。通过使用一个临时Label，赋予一个简单文本与字体或带属性的文本，然后使用sizeToFit()方法来计算其准确的尺寸。

来着手进行页面页脚的绘制，大致步骤跟页眉差不多，无需多言了。注意一下在文本的上方有一条水平线，并且文本颜色不同，文本的字符间距也不同:
```swift
override func drawFooterForPageAtIndex(pageIndex: Int, inRect footerRect: CGRect) {
    let footerText: NSString = "Thank you!"
 
    let font = UIFont(name: "Noteworthy-Bold", size: 14.0)
    let textSize = getTextSize(footerText as String, font: font!)
 
    let centerX = footerRect.size.width/2 - textSize.width/2
    let centerY = footerRect.origin.y + self.footerHeight/2 - textSize.height/2
    let attributes = [NSFontAttributeName: font!, NSForegroundColorAttributeName: UIColor(red: 205.0/255.0, green: 205.0/255.0, blue: 205.0/255, alpha: 1.0)]
 
    footerText.drawAtPoint(CGPointMake(centerX, centerY), withAttributes: attributes)
}
```
这段代码创建了“Thank you!”，不过没有添加水平线，需要使用更多的代码来画线:
```swift
override func drawFooterForPageAtIndex(pageIndex: Int, inRect footerRect: CGRect) {
    ...
 
    // Draw a horizontal line.
    let lineOffsetX: CGFloat = 20.0
    let context = UIGraphicsGetCurrentContext()
    CGContextSetRGBStrokeColor(context, 205.0/255.0, 205.0/255.0, 205.0/255, 1.0)
    CGContextMoveToPoint(context, lineOffsetX, footerRect.origin.y)
    CGContextAddLineToPoint(context, footerRect.size.width - lineOffsetX, footerRect.origin.y)
    CGContextStrokePath(context)
}
```
现在就有一条水平线了！

在这一节结束前，关于页眉和页脚还有一点我想说一下，若你注意到的话会发现两个文本都是NSString对象，而不是String对象，这里有个特别的原因: drawAtPoint(...)方法是属于NSString类的绘制方法，如果使用String对象的话，需要先转换一下类型，如下所示:
```swift
(text as! NSString).drawAtPoint(...)
```
再次运行app，预览导出的PDF，这次就有页眉和页脚了。

可以在[这里](https://github.com/appcoda/Print2PDF)下载到完整的Xcode工程。
