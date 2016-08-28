#Building iOS Apps with Xamarin and Visual Studio
##在Xamarin与Visual Studio中构建iOS应用

***

>* 原文链接 : [Building iOS Apps with Xamarin and Visual Studio](https://www.raywenderlich.com/134049/building-ios-apps-with-xamarin-and-visual-studio)
* 原文作者 : [Bill Morefield](https://www.raywenderlich.com/u/bmorefield)
* 译者 : [yrq110](https://github.com/yrq110)

***

> PS：本篇教程使用的是C#，并非Swift。

在创建iOS app时，开发者通常会选择苹果提供的语言与集成开发环境: Objective-C / Swift与Xcode，然而这并不是唯一的选择，可以使用多种语言与框架进行iOS app的创建。

[Xamarin](https://xamarin.com/)是一个比较流行的选择，一个跨平台的框架，允许你使用C#和Visual Studio进行iOS, Android, OS X与Windows app的开发。Xamarin的一个主要优势在于可以在iOS与Android间共用一个app代码。

Xamarin跟其他跨平台框架比有一个很大的优点: 使用Xamarin的话你的工程会编译成本地代码(native code)，并可以使用内部的本地API，这意味着一个书写良好的Xamarin app与Xcode中的app是没有区别的。想了解更多细节可以看看这篇文章[Xamarin vs. Native App Development](http://willowtreeapps.com/blog/xamarin-vs-native-app-development/) 。

Xamarin在之前存在一个巨大的缺点: 价格。每个平台每年1000美元的高昂价格也许会使你放弃日常的拿铁与卡布奇诺去支付它...没有咖啡的编程是很危险的。因为这个高昂的价格，直到最近Xamarin都还是主要吸引一些有着庞大预算的企业项目的注意。

不过最近发生了变化，Microsoft收购了Xamarin并宣布它将支持全版本的Visual Studio，包括免费的[Community Edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)，方便了个人开发者与小型组织。

免费? 是个值得庆祝的价格!

<img src="https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/dollar-941246_1280.jpg" width="50%" height="50%" />

更多的咖啡钱!

***

除了开销外, Xamarin的其它优点有:
* 利用现有的C#库与工具来构建移动app。
* 在不同平台间复用app代码。
* 在ASP.Net后端与面向用户的app间共享代码。

Xamarin同时提供了多种工具来选择，取决于你的需求。使用[Xamarin Forms](https://www.xamarin.com/forms)来最大化跨平台的代码复用，无需针对平台来指定功能与特殊的自定义接口，非常有效。 

若你的app需要某些平台独有的特性或设计，使用Xamarin.iOS、Xamarin.Android与其他针对平台的模块来直接使用本地API与框架来进行交互。这些模块提供灵活创建自定义用户接口的功能，同时也允许跨平台共享代码。

在这篇教程中将会使用**Xamarin.iOS**来创建一个用来显示用户照片的iPhone app。

无需任何iOS或Xamarin开发经验，不过需要你掌握一些C#的基本知识使知识的收益最大化。

## 入门

为了使用Xamarin和Visual Studio开发iOS应用，理论上需要两台机器:

1. **一台Windows**  用来运行Visual Studio与写项目代码。
2. **一台安装了Xcode的Mac**  作为构建主机(build host)。可以不是一个专门用来构建的机子，只要在Windows测试与开发时保持网络畅通就行。

如果这两台机子离的比较近就再好不过了，当你在Windows上构建运行时，Mac上的iOS模拟器就会加载对应的项目。

我知道你会说，“我没两台机子咋整?!”

* **对于仅使用Mac的用户**  Xamarin为OS X提供了一个IDE，不过在这篇教程中只关注Visual Studio。因此若你想跟着教程走可以在Mac上运行一个Windows虚拟机，可以使用[VMWare Fusion](https://www.vmware.com/products/fusion)或免费、开源的[VirtualBox](https://www.virtualbox.org/)来创建虚拟机。

若使用Windows虚拟机，需要确保Windows与Mac间保持正常的网络连接。通常会在Windows中ping Mac的IP地址测试一下。

* **对于仅使用Windows的用户**  现在去买个Mac，我会等你！:]也可以使用像[MacinCloud](http://www.macincloud.com/)或[Macminicolo](https://macminicolo.net/)这种提供远程访问服务的Mac主机来构建。

这篇教程中假定你分别使用Mac与Windows电脑，别担心，在Mac上使用Windows虚拟机也是可以的。

## 安装Xcode与Xamarin

若还没准备好Xcode的话先在你的Mac上[下载并安装Xcode](https://itunes.apple.com/us/app/xcode/id497799835?uo=8&at=11ld4k)，跟安装其它App Store上的app一样，不过大小上G了，需要花一些时间。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/danish-butter-cookies-1032894_1280.jpg)

Perfect time for a cookie break!
***

Xcode安好后，在Mac上下载Xamarin Studio，需要提供你的邮箱地址来下载。

下载完成后打开安装包双击**Xamarin.app**安装，接受条款一路continue。

安装器会搜索已存在的工具与当前平台版本，并列出一个当前开发环境的详细列表。确保选择的是**Xamarin.iOS**，点击**Continue**。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/05/xamarin-installer.png)

接着是安装项目的确认列表，点击**Continue**执行，会出现一个Summary与一个启动Xamarin Studio的选项，点击**Quit**完成安装。

## 安装Visual Studio与Xamarin

这篇教程中可以使用任何版本的Visual Studio，包括免费的[Community Edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)。有些特性在Community Edition中没有，不过这并不影响你开发出复杂的app来。

查看Windows系统的Visual Studio[最低硬件需求](https://www.visualstudio.com/en-us/downloads/visual-studio-2015-system-requirements-vs.aspx#1)，为了有一个好的开发体验，至少要3G的内存。

若还未安装Visual Studio的话，在[Community Edition](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)网站点击绿色的**Download Community 2015**按钮下载**Community Edition**安装器。

运行安装器开始安装，选择自定义安装选项，在列表中展开**Cross Platform Mobile Development**，选择C#/.NET (Xamarin v4.0.3)(v4.0.3是教程在撰写时的版本，未来也许会变化)。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/vs-installer-354x500.png)

点击Next等待安装完成，会花点儿时间 :]

若你已准备好Visual Studio了不过还没有Xamarin工具，打开Windows系统的**Programs and Features**找到Visual Studio 2015，访问其安装程序，选择**修改**，会在Cross Platform Mobile Development下面看到**C#/.NET (Xamarin v4.0.3)**，选择它点击**更新**进行安装。

安了一大堆东西啊，现在搞定所有需要的东西了!

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Install_Powers.png)

## 创建App

打开Visual Studio选择**File\New\Project**，在Visual C#下展开**iOS**，选择**iPhone**，右侧选择**Single View App**模板。这个模板会创建一个具有单视图控制器的app。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/NewProject-461x320.png)

**Name**和**Solution Name**都输入**ImageLocation**，选择一个app的保存路径点击**OK**创建工程。

Visual Studio需要你准备好作为Xamarin构建主机的Mac:

1. 在Mac上打开**System Preferences**选择**Sharing**。
2. 开启**Remote Login**。
3. 选择**Allow access**下的**Only these users**，添加一个用户用来访问Mac上的Xcode和Xamarin。

  ![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/build-host-setup-629x500.png)

4. 关闭设置回到Windows电脑。

回到Visual Studio中，会被询问是否将Mac作为构建主机。选择Mac点击**Connect**，输入用户名与密码点击**Login**。

可以通过工具栏确认连接状态。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/Connected_Indicator.png)

从解决平台的下拉菜单中选择iPhone Simulator，这时在构建主机中会自动选择一个模拟器，可以通过点击当前模拟器旁边的小箭头来改变模拟器设备。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/Change_Simulator-1.png)

点击绿色的**Debug**箭头或者快捷键**F5**来构建并运行程序。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Build_and_Run.png)

app会编译并执行，不过你在Windows上看不到它的运行情况，需要在Mac上看，这就是为何两台机子距离较近比较好 :]

在最近的[Evolve会议](https://evolve.xamarin.com/)上，Xamarin宣布[iOS Simulator Remoting](https://blog.xamarin.com/live-from-evolve-new-xamarin-previews/)很快就能实现app运行在苹果的iOS 模拟器就好像是运行在Windows上的模拟器一样，不过现在你需要操作Mac上的模拟器。

能看到一个显示启动界面的模拟器，然后出现一个空的视图。恭喜！Xamarin设置成功。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/05/template-app-running-1-272x500.png)

点击红色停止按钮(快捷键Shift+F5)来停止app。

## 创建Collection View

app会在一个Collection View中显示用户照片的缩略图，Collection View是一个使用网格显示一些物件的iOS控件。

在**Solution Explorer**中打开**Main.storyboard**，编辑app的故事板，其中包含app的“场景”。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/Main_Storyboard.png)

打开**Toolbox**在文本框中输入**collection**过滤出部件。将**Data Views**下面的**Collection View**对象拖拽到空视图上面。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Drag_Collection_View-650x456.png)

选择collection view，应该会看到视图的每条边上都有**空心圆**，若看到的是**T型**图案，再次点击切换到空心圆。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/resize-collection-view-521x500.gif)

点击并拖动每个圆到视图的边界直接出现蓝色线条，这时松开鼠标边界就会与这个位置对齐。

现在需要设置collection view的Auto Layout Constraints，这会告知app当设备旋转时如何调整视图的尺寸。在故事板上方的工具栏中，点击CONSTRAINTS标签旁的绿色按钮添加信号，这会给collection view自动添加constrains。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/Add_Constraints-650x112.png)

生成的constrains基本上没问题，因此你无需修改它们。在**Properties**窗口，切换到**Layout**子视图然后滚动到下面的**Constraints**区域。

由边界生成的两个constraints是正确的，不过**高度**和**宽度**constraints却有问题，点击每栏后面的**X**删除Width与Height constraints。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Delete_Constraints-304x500.png)

注意若collection view变成了橘黄色，则出现的指示器表示constraints需要被修复。

点击collection view选择它，若看到的依旧是圆圈则再次点击将图标换成绿色**T型**图案，点击并拖动collection view**顶部**边界的T到名为**Top Layout Guide**的绿色矩形，释放鼠标创建一个与视图上边界关联的constraint。

最后，点击并拖动collection view**左侧**的T到左边界直接出现**蓝色虚线**，释放鼠标创建一个与视图左边界关联的constraint。

现在，你的constraints设置应像下面这样:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/Constraints.png)


## 设置Collection View Cell

也许你注意到了collection view中的包含一个红色感叹号的正方形轮廓，这就是一个colletion view cell，表示collection view中的单个条目。

来设置这个**cell的尺寸**，选择collection view，滚动到**Layout**选项卡的顶部，将**Cell Size**下的**Width**和**Height**设为**100**。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/cell-size.png)

接着点击cell上的**红色圆圈**，此时会弹出一个提示说你还未设置cell的重用修饰符，选择cell到Widget选项卡中，滚动到底部的**Collection Reusable View**，在**Identifier中**输入**ImageCellIdentifier**，这样错误提示就会消失了。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/Set_Reuse_Identifier-480x202.png)

继续向下滚动到**Interaction Section**，选择**Predefined**(预设)中的blue作为背景色。

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/Set_Cell_Background_Color-427x320.png)

这样scene应如下所示:

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/05/collection-cell-with-color-470x500.png)

滚动到**Widget**顶部将**Class**设为**PhotoCollectionImageCell**。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Set_Cell_Class.png)

Visual Studio会使用这个名称自动创建一个继承自**UICollectionViewCell**的类，并生成**PhotoCollectionImageCell.cs**文件。


## 创建Collection View数据源

你需要手动创建一个类，作为`UICollectionViewDataSource`为collection view提供数据。

右击`Solution Explorer`中的`ImageLocation`，选择Add/Class，命名为`PhotoCollectionDataSource.cs`点击`Add`。

打开新建的`PhotoCollectionDataSource.cs`，在文件顶部添加如下代码:

```c#
using UIKit;
```
这允许你访问iOS的`UIKit`框架。


如下改变类的定义:
```c#
public class PhotoCollectionDataSource : UICollectionViewDataSource
{
}
```

还记得之前定义过collection view cell的重用标识符吗？在这个类中会使用它,在类中进行如下定义:

```c#
private static readonly string photoCellIdentifier = "ImageCellIdentifier";
```

`UICollectionViewDataSource`类中包含两个需要你实现的抽象成员。在类中添加如下代码:

```c#
public override UICollectionViewCell GetCell(UICollectionView collectionView, 
    NSIndexPath indexPath)
{
    var imageCell = collectionView.DequeueReusableCell(photoCellIdentifier, indexPath)
       as PhotoCollectionImageCell;
 
    return imageCell;
}
 
public override nint GetItemsCount(UICollectionView collectionView, nint section)
{
    return 7;
}
```

`GetCell()`负责提供collection view中显示的cell。

`DequeueReusableCell`重用那些不再需要的cell，比如说超出了屏幕。若没有可用的重用cell，会自动创建一个新的cell。

`GetItemsCount`告诉collection view需要显示7个条目。

接着需要添加一个collection view与ViewContrller类的关联，这样view contrller。回到`Main.storyboard`，选择collection view，然后选择`Widget`选项卡，在`Name`栏输入collectionView。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Set_CollectionView_Name-480x160.png)

Visual Studio会使用这个名字自动在ViewContrller类中创建一个静态变量。

打开`ViewController.cs`，在类中添加如下代码:

```c#
private PhotoCollectionDataSource photoDataSource;
```

在`ViewDidLoad()`底部，添加如下代码用来初始化数据源并与collection view连接。

```c#
photoDataSource = new PhotoCollectionDataSource();
collectionView.DataSource = photoDataSource;
```

这样的话`photoDataSource`就会给collection view提供数据了。

构建并运行，应该能在collection view中看到七个蓝色方块。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/05/cells-no-photo-app-272x500.png)

棒 – 有点app的感觉啦!

![](https://cdn5.raywenderlich.com/wp-content/uploads/2016/06/Blue_Squares-230x320.png)

## 显示照片

即便蓝色方块比较酷，接着来更新数据源，检索设备中的照片将其显示在collection view中。需要使用Photos框架来访问Photos app所管理的照片与视频资源。

开始，添加在cell上添加一个用来显示图像的视图。打开Main.storyboard选择collection view cell。在Widget选项卡中向下滚动，把背景色改为默认。

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/Set_Default_Cell_Background_Color.png)

打开Toolbox，搜索Image View,，然后把一个Image View拖拽到collection view Cell上。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Drag_Image_View.png)

初始的image view大小会比cell大很多，来重新设定尺寸，选择image view来到Properties\Layout选项卡中。在View区域将X与Y设为0，Width与Height设为100。

![](https://cdn1.raywenderlich.com/wp-content/uploads/2016/06/Set_Image_View_Size.png)

切换到image view的Widget选项卡，将Name设为cellImageView。Visual Studio会自动创建一个名为cellImageView的字段。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/Set_Image_View_Name.png)

滚动到**View**区域，将**Mode**改为**Aspect Fill**，这会使图像拉伸以适应边界尺寸。

![](https://cdn4.raywenderlich.com/wp-content/uploads/2016/06/Set_Image_View_Mode.png)

这个字段并不是public的，其它类无法访问它。需要提供一个方法来设置图像。

打开PhotoCollectionImageCell.cs在类中添加如下方法:

```c#
public void SetImage(UIImage image)
{
    cellImageView.Image = image;
}
```

现在来检索照片以更新**PhotoCollectionDataSource**。

在**PhotoCollectionDataSource.cs**顶部添加如下行:

```c#
using Photos;
```

给**PhotoCollectionDataSource**添加如下字段:

```c#
private PHFetchResult imageFetchResult;
private PHImageManager imageManager;
```
**imageFetchResult**字段会保存一个照片实体对象们的有序列表，从**imageManager**来得到照片列表。

在**GetCell()**方法上面添加这个构造函数:

```c#
public PhotoCollectionDataSource()
{
    imageFetchResult = PHAsset.FetchAssets(PHAssetMediaType.Image, null);
    imageManager = new PHImageManager();
}
```

这个构造函数得到了Photos app中的一个所有图像资源的列表，将结果储存进了imageFetchResult字段中。接着设置imageManager，给app用来查询每幅图像的详细信息之用。

在构造函数下面添加析构函数释放imageManager对象。

```c#
~PhotoCollectionDataSource()
{
    imageManager.Dispose();
}
```

为了让GetItemsCount与GetCell方法使用这些资源并返回images而不是空的cells，如下修改GetItemsCount():

```c#
public override nint GetItemsCount(UICollectionView collectionView, nint section)
{
    return imageFetchResult.Count;
}
```

如下修改GetCell():

```c#
public override UICollectionViewCell GetCell(UICollectionView collectionView, 
    NSIndexPath indexPath)
{
    var imageCell = collectionView.DequeueReusableCell(photoCellIdentifier, indexPath) 
        as PhotoCollectionImageCell;
 
    // 1
    var imageAsset = imageFetchResult[indexPath.Item] as PHAsset;
 
    // 2w
    imageManager.RequestImageForAsset(imageAsset, 
        new CoreGraphics.CGSize(100.0, 100.0), PHImageContentMode.AspectFill,
        new PHImageRequestOptions(),
         // 3
         (UIImage image, NSDictionary info) =>
        {
           // 4
           imageCell.SetImage(image);
        });
 
    return imageCell;
}
```

这里是上面方法中变化部分的分析:

1. **indexPath**包含一个collection view所返回item的引用值。这个Item属性是一个索引值，通过索引得到资源并将其转换为**PHAsset**类型。
2. 使用**imageManager**通过期望的尺寸与内容模式请求得到资源图像。
3. 许多iOS框架的方法对于请求使用延迟执行来使它们有足够的时间去完成，比如**RequestImageForAsset**，在完成时会调用一个委托方法。。当请求完成时，会调用使用image与information的委托方法。
4. 最后将图像设置在cell中。

构建并运行，会看到一个请求访问权限的提示。

![](https://cdn2.raywenderlich.com/wp-content/uploads/2016/06/Permission_Prompt-333x500.png)

选择**OK**，然而app...没反应。太令人失望了!

![](https://cdn3.raywenderlich.com/wp-content/uploads/2016/06/Why_no_work-248x320.png)

iOS考虑到访问的用户照片是敏感信息，会提示用户是否给予权限。app必须注册以在用户授权时获得通知，这样就可以重载视图了。下面来做这些工作。

## Registering for Photo Permission Changes
