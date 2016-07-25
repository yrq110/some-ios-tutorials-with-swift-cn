#Documenting Your Swift Code in Xcode Using Markdown
##使用Markdown文档化Xcode中的Swift代码

[原文地址](http://www.appcoda.com/swift-markdown/)

在Xcode 7所包含的所有特性中，有一个新的革命毋庸置疑：以一个更好的方式去书写代码文档。开发者可以使用强大的Markdown语句写出含有多种文本样式的文档，结合特定的关键字标识出特殊部分(比如参数、函数结果等等)，会有意想不到的结果。新的启用Markdown的文档风格有如下优点：提供了更高等级的自定义文本能力，更加灵活，当然也更加有趣。不过如果你对老式的文档风格感兴趣，可以看看我们之前的[教程](http://www.appcoda.com/documenting-source-code-in-xcode/)。

撰写代码文档对每个开发者来说都是一项重要的任务，虽然看上去这会拖慢开发的进度，不过确实是开发中的一部分。我不觉得对每一个属性、函数、类、结构体与其他项目中存在的项撰写合适的、便于理解的文档是件容易的事情。可以通过以下要点来书写文档：

* 描述属性、函数于类的作用、目的、细节等，高亮函数特定的条件、情况或者需求。
* 高亮方法的输入与输出(参数与返回值)。
* 在初始实现后，当你重新审视代码时，函数与属性的作用一目了然。
* 使其他开发者理解如何使用你的代码或库。
* 使用工具来产生专业的手册(比如说Jazzy).

可以通过三种方式来预览与访问在Xcode中书写的代码文档:
* 按住Option/Alt点击属性名、方法等、类名等，快速预览窗会在名称附近弹出。
* 将光标放置在属性名、方法名或类名上，打开快速帮助。
* 使用可以生成手册的第三方工具。比如说Jazzy是个不错的工具，之后会说它。使用它的话，你的文档就可以编译成网页，整个文档可以组成一个脱机的网站，保存在工程文件夹中的一个子文件夹。

代码文档不是一成不变，它需要是灵活的，根据实体中(属性、方法、类、结构体、枚举)产生的修改来改变。若当你实现了一个新实体时没有添加文档则之后你也不会添加的。因此，养成一个在适当的时机添加代码文档的习惯吧，分配一些时间去做，这是值得的。

##Markdown语句基础

为了充分利用新的文档风格，了解一下Markdown语句的相关知识是很重要的，如果你已经知道了就跳过这一节吧。可以在网上找到很多关于Markdown的资料，比如说[这儿](https://daringfireball.net/projects/markdown/syntax)和[这儿](https://confluence.atlassian.com/bitbucketserver/markdown-syntax-guide-776639995.html)有一些不错的阅读资料。

你可以找到其他有关Markdown语句的资源，我这里只要求最低要求的基础语句，提供完整全面的引导教程不是我所关注的，将注意力放在最常用标注语句的表现上。

你可能知道(或者你刚学了) Markdown语句是通过使用特殊的字符来格式化文本、添加资源(例如链接或图像)与标注特殊区域或文本(例如有序或无序列表，代码块)。这些字符很好记，可以通过反复使用或查询下面的列表来加深记忆。当你习惯了Markdown语句后会发现很有趣，通过使用合适的表及其可以产生不同格式的文档：比如HTML网页、PDF文档等等。说到HTML，Markdown支持内联的HTML，意味着你可以直接在文本中加入HTML标签，然后会进行适当的渲染。不过HTML并不是Markdown的本质，因此只关注Markdown本身的语句。

下面这个列表包含了常用的Markdown语句:

*   #text#: 将文本设置为header，与HTML中的`<H1>`等价。两个\#字符相等于`<H2>`header，直到`<H6>`同理，后面的\#字符是可选的。
* \*\*text\*\*: 返回星号中间加粗的文本。
* \*text\*: 返回星号中间斜体的文本。
* \* text: 创建一个无序列表项(注册在星号与文本间有一个空格)，也可以使用“+”或“-”符号。
* 1\. text: 创建一个有序列表项 (数字列表)。
* \[linked text\]\(http://some-url.com\): 当文本转换为一个链接。
* \> text: 创建一个块引用
* 使用4个空格或tab标记书写的代码块，与HTML中的`<pre>``</pre>`标签，如果想添加缩写需要添加额外的4个空格或1个tab字符。
* 如果你不想被空格和tab搞混，可以将代码写在“\`” 字符中。比如说\`var myProperty\`会显示成: `var myProperty`。
* 另一种创建代码块的方式是添加4个(“\`”)符号，然后开始代码的书写，在代码的最后一行添加另外4个(“\`”)。
* 使用反斜杠禁用Markdown语句中特殊字符的渲染。例如：/**this/**就不会显示成加粗的字体。

上述是需要了解的Markdown语句中最重要的元素，其实还有更多，给你介绍的这些足够入门了，想了解更多细节的话请自己去查阅更多的资料。

对Markdown产生兴趣后，寻找一个免费的编辑器(在线的或者是Mac软件)，来上手尝试一下。如果你找的编辑器支持实时预览的话，就可以观察到你的输入会直接转换成HTML，立即看到输出。

>编者的话: 可以尝试如下编辑器StackEdit (在线), Typora, Macdown, Focused 与 Ulysses。

##使用Markdown

在Swift中对各类实体编写文档需要遵守一些特性的规则，可以对属性(变量与常量)、方法、函数、类、结构体、结构体、枚举、协议、扩展与其他代码结构或实体编写文档。文档块需要在实体的声明或头文件后来编写，使用3个斜杠或将代码块包含在如下的结构中:
````
/**
 
*/
````
双斜杠(//)会被Xcode无视，不会渲染任何文档。在你的代码块(一个方法的body)中使用双斜杠，而不是在编写整个实体的文档时。 

来看一个使用Markdown语句的简单例子，在下面的代码中文档化一个简单的属性。建议你在Xcode中新建一个Playground，在里面把玩示例。
````swift
/// This is an **awesome** documentation line for a really *useful* variable.
var someVar = "This is a variable"
````
效果如下:

![](http://www.appcoda.com/wp-content/uploads/2016/05/t52_1_quickhelp1.png)

注意 “awesome” 被显示成了加粗的状态，因为它被双星号所包含，"useful"被星号包含因此被显示成了斜体。

来看看另一个例子，这次使用一个函数:

````swift
/**
    It calculates and returns the outcome of the division of the two parameters.
 
    ## Important Notes ##
    1. Both parameters are **double** numbers.
    2. For a proper result the second parameter *must be other than 0*.
    3. If the second parameter is 0 then the function will return nil.
 
*/
func performDivision(number1: Double, number2: Double) -> Double! {
    if number2 != 0 {
        return number1 / number2
    }
    else {
        return nil
    }
}
````
把上述代码复制粘贴进playground中，按住Option点击函数名的话，会看到显示出的快速帮助:

![](http://www.appcoda.com/wp-content/uploads/2016/05/t52_2_quickhelp2.png)

这里使用了两个新的Markdown元素，header与有序列表，在有序列表中的列表项应用了加粗与斜体格式。注意到使用Markdown的特殊字符来丰富文档中的文本样式是很轻松的一件事。在Quick Help Inspector中如下所示:

![](http://www.appcoda.com/wp-content/uploads/2016/05/t52_3_help_inspector1.png)

接下来，要在函数的文档中添加一个代码块。“\`”符号除了可以创建代码块外，也常常用于在行内标记函数名
````swift
/**
    It doubles the value given as a parameter.
 
    ### Usage Example: ###
    ````
    let single = 5
    let double = doubleValue(single)
    print(double)
    ````
 
    * Use the `doubleValue(_:)` function to get the double value of any number.
    * Only ***Int*** properties are allowed.
*/
func doubleValue(value: Int) -> Int {
    return value * 2
}
````
效果如下:

![](http://www.appcoda.com/wp-content/uploads/2016/05/t52_4_quickhelp3.png)

最后让我们来给枚举添加一些文档，然后在另一个函数的函数体中使用它。在这里有意思的是枚举中的每一个元素都有对应的文档:
````swift
/**
    My own alignment options.
 
    ````
    case Left
    case Center
    case Right
    ````
*/
enum AlignmentOptions {
    /// It aligns the text on the Left side.
    case Left
 
    /// It aligns the text on the Center.
    case Center
 
    /// It aligns the text on the Right side.
    case Right
}


func doSomething() {
    var alignmentOption: AlignmentOptions!
 
    alignmentOption = AlignmentOptions.Left
}
````
现在，每当使用枚举中的元素时Xcode就会显示对应元素的文档:

![](http://www.appcoda.com/wp-content/uploads/2016/05/t52_5_quickhelp4.png)

##使用关键字

使用Markdown语句只是文档化代码的其中一个方面，显然丰富文本的格式是很棒的，可以产生一个不错的文档，不过如果只干到这儿的话会落下一些东西的，来看看文档化代码的另一个方面————使用关键字。

每当使用一个关键字时Xcode会自动识别它，用一个默认的格式去渲染文档(用第三方库时同理)。在标注文档中通用的代码结构来区分它们时使用关键字是很有效的。比如说，有用于高亮方法参数的、返回值的、类作者的、函数版本的关键字，还有很多，不用全部都使用，有些是很不常用的。熟悉一些常用的关键字，其他的可以用的时候再去查阅。

说了这么多，是时候来通过一些例子来练习一下关键字的使用了，第一个关键字用来标注一个方法或函数接受的参数:

````swift
/**
    This is an extremely complicated method that concatenates the first and last name and produces the full name.
 
    - Parameter firstname: The first part of the full name.
    - Parameter lastname: The last part of the fullname.
*/
func createFullName(firstname: String, lastname: String) {
    let fullname = "\(firstname) \(lastname)"
    print(fullname)
}
````
效果如下:

![](http://www.appcoda.com/wp-content/uploads/2016/05/t52_6_keywords1.png)
