# Magical Error Handling in Swift
## 奇妙的Swift错误处理

***

>* 原文链接 : [Magical Error Handling in Swift](https://www.raywenderlich.com/130197/magical-error-handling-swift)
* 原文作者 : [Gemma Barlow](https://www.raywenderlich.com/u/gemmakbarlow)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/05/errorhandling-1-feature-250x250.png)

Swift的错误处理是从Swift 1借鉴OC的模式开始，在那之后取得了很大的进展，在Swift 2中的更新使得app中异常状态和场景的处理更加方便，这个优点延续到了Swift 3，不过在最新版本中却没有什么重大的更新。

像其它变成语言一样，swift中的错误处理方法会根据所遇到的错误类型与app的整体结构有所不同。

这篇教程会将你带入一个有巫师、魔女、蝙蝠和蟾蜍的魔法世界中，展示如何较好处理一些常见的错误场景，同时也会学到如何将项目中老版本的错误处理方法更新成新版的语法，最后，通过注视你的水晶球展望Swift错误处理的未来!

> 注意: 这篇教程假定你对Swift 3的语法较为熟悉 - 特别是枚举和可选类型。若需要重温这些概念，可以先看看Greg Heo的[What’s New in Swift 2](https://www.raywenderlich.com/108522/whats-new-in-swift-2)或者其它资料。

是时候开始了(from the the cauldron into the fire!)，一同去发现Swift 3中错误处理的奇妙之处!

## 入门

这篇教程有两个开始项目，分别对应两个小节，[使用nil处理错误](https://koenig-media.raywenderlich.com/uploads/2016/07/Avoiding-Errors-with-nil-Starter.playground-Swift-3-Secondary-Update.zip)和[自定义错误处理方法](https://koenig-media.raywenderlich.com/uploads/2016/07/Avoiding-Errors-with-Custom-Handling-Starter.playground-Swift-3-Secondary-Update.zip)。

在Xcode中打开第一个项目。

浏览一下代码会看到一些类、结构体与枚举，它们在这篇教程中具有魔力。

注意这部分代码:

```swift
protocol Avatar {
  var avatar: String { get }
}
```
这个协议基本上被所有类和结构体继承了，为每个对象提供一个可视化的表现属性使其可以输出到控制台中。
```swift
enum MagicWords: String {
  case abracadbra = "abracadabra"
  case alakazam = "alakazam"
  case hocusPocus = "hocus pocus"
  case prestoChango = "presto chango"
}
```
这个枚举表示用来创建魔法的咒语。
```swift
struct Spell {
  var magicWords: MagicWords = .abracadabra
}
```
用于构建一个基本的咒语，默认的咒语是.abracadabra。

好了现在你已经熟悉了这个超自然的世界，作为五火球神教的一员已经准备好施放魔法了。

## 为何重视错误处理?

> *“错误处理是一门让错误变得优雅的艺术”*    –Swift Apprentice, 第22章 (错误处理)

良好的错误处理可以增强终端用户的体验，使软件维护者可以更容易的发现问题、问题起因和引起的严重性。对代码中的错误处理针对性越强，代码就越容易维护。同时错误处理可以在系统层面以合适的方式处理异常，以免给用户带来糟糕的体验。

不过错误并不总是需要被处理的，有时语言的特性就会避免一些常见的错误。比如说，若有些语法特性可以完全避免一个错误的发生可以直接使用这种设计，若无法避免一个潜在的错误条件(error condition)，针对性的处理是最好的选择。

## 使用nil避免错误

由于Swift优秀的可选处理能力，可以完全避免当期望一个值时但没有值存在时的错误条件。作为一个机智的开发者，可以利用这个特性在错误条件发生时返回nil，当遇到一个错误状态时应该不采取任何操作，这么做效果很好。

使用nil避免swift错误的两个典型示例是可失败构造器和guard语句。

### 可失败构造器

可失败构造器会阻止一个没有提供有效信息的对象的创建，在Swift提供这个构造器之前一般是用工厂模式来实现的。

在create:方法中可以看到这个模式的实现:
```swift
static func create(withMagicWords words: String) -> Spell? {
  if let incantation = MagicWords(rawValue: words) {
    var spell = Spell()
    spell.magicWords = incantation
    return spell
  }
  else {
    return nil
  }
}
```
上面的构造器尝试使用所提供的咒语来创建符咒，若咒语不是魔法咒语则返回nil。

观察一下创建符咒后的输出:

![](https://koenig-media.raywenderlich.com/uploads/2016/07/Spell.create.png)

第一次使用咒语"abracadabra"成功创建了符咒，不过"ascendio"并没有那么好的运气，第二次返回了nil。(魔女并不是总成功的)

![](https://koenig-media.raywenderlich.com/uploads/2016/05/errorhandling-2-650x256.png)

工厂模式是一种老式的编程方法，在Swift中有更佳的实现方法——使用可失败构造器来实现。

删除create(\_words:)方法，使用如下代码替换它:

```swiftd
init?(words: String) {
  if let incantation = MagicWords(rawValue: words) {
    self.magicWords = incantation
  }
  else {
    return nil
  }
}
```
在这里简化了代码，代码中未具体指明创建与返回Spell对象。

这几行现在出现了编译错误:
```swift
let first = Spell.create(withMagicWords: "abracadabra")
let second = Spell.create(withMagicWords: "ascendio")
```
需要改成新的构造器进行创建，使用如下代码替换上面的代码:

```swift
let first = Spell(words: "abracadabra")
let second = Spell(words: "ascendio")
```
这样一来就没有错误了，playground顺利编译通过。这样一改代码就更加简介了，不过还可以做的更好！:]

### Guard语句

guard是一个用来断言某些事情为true的快捷方法：比如说，判断一个值是否大于0或一个条件值是否可以被解绑，在验证失败的情况下可以执行一段代码。

guard语句是在Swift 2中提出的，一般是通过调用堆栈进行冒泡错误处理，错误在最后才会被处理。Guard语句允许从函数或方法中较早的退出，这样就比需要判断条件满足之后的逻辑才会执行的方法更加清晰，简明。

将可失败构造器改为如下使用guard语句的形式来初始化:

```swift
init?(words: String) {
  guard let incantation = MagicWords(rawValue: words) else {
    return nil
  }
  self.magicWords = incantation
}
```

这样修改之后就不需要为了判断另一种情况额外增加一行代码了，使在失败的情况下的处理更加显眼，因为它放在了构造器的最前端。并且“黄金路径”的缩进最少。“黄金路径”是指当每件事都如预期即没有错误发生时的执行路径。更少的缩进更易于阅读。

注意到第一个和第二个咒语的常量值并未改变，而代码则变得更加精练了。

## 自定义处理避免错误

至此我们已经整理了Spell构造器的代码并且使用nil避免了一些错误，接下来学习如何进行更加复杂的错误处理。

进行这一节的学习请先打开Avoiding Errors with Custom Handling – Starter.playground。

看看下面的代码，观察它的特点:
```swift
struct Spell {
  var magicWords: MagicWords = .abracadbra
 
  init?(words: String) {
    guard let incantation = MagicWords(rawValue: words) else {
      return nil
    }
    self.magicWords = incantation
  }
 
  init?(magicWords: MagicWords) {
    self.magicWords = magicWords
  }
}
```
这是在教程第一部分中修改后的Spell的构造器。注意下面的Avatar协议和一个为了方便起见使用的可失败构造器，
```swift
protocol Familiar: Avatar {
  var noise: String { get }
  var name: String? { get set }
  init(name: String?)
}
```
在playground的后面会有一些动物(比如蝙蝠和蟾蜍)实现这个Familiar协议。

> 注意: 这里的familiar并不是熟悉的意思，指的是魔女或者魔法师的宠物或仆从，通常具有类人的能力，想象一下哈利波特中的Hedwig和绿野仙踪中的飞猴。

> ![](https://koenig-media.raywenderlich.com/uploads/2016/04/Owl-480x320.jpg)
  
>这明显不是Hedwig(哈利波特中作为信使的宠物)，不过也挺可爱的，不是吗?

```swift
struct Witch: Magical {
  var avatar = "*"
  var name: String?
  var familiar: Familiar?
  var spells: [Spell] = []
  var hat: Hat?
 
  init(name: String?, familiar: Familiar?) {
    self.name = name
    self.familiar = familiar
 
    if let s = Spell(magicWords: .prestoChango) {
      self.spells = [s]
    }
  }
 
  init(name: String?, familiar: Familiar?, hat: Hat?) {
    self.init(name: name, familiar: familiar)
    self.hat = hat
  }
 
  func turnFamiliarIntoToad() -> Toad {
    if let hat = hat {
      if hat.isMagical { // 你见过魔女不带帽子就施放魔法吗 :]
        if let familiar = familiar {   // 检查魔女是否拥有仆从
          if let toad = familiar as? Toad {  // 若仆从是蟾蜍，则不需要魔法
            return toad
          } else {
            if hasSpell(ofType: .prestoChango) {
              if let name = familiar.name {
                return Toad(name: name)
              }
            }
          }
        }
      }
    }
    return Toad(name: "New Toad")  // 这是一只新的蟾蜍
  }
 
  func hasSpell(ofType type: MagicWords) -> Bool { // 检查魔女的魔法书中是否拥有某个咒语
    let change = spells.flatMap { spell in
      spell.magicWords == type
    }
    return change.count > 0
  }
}
```
最后是魔女，代码中会看到:

* 一个魔女是使用姓名和仆从或姓名、仆从和帽子初始化的。
* 一个魔女了解有限数量的咒语，并保存在Spell对象数组中。
* 一个魔女倾向于使用.prestoChango咒语将仆从变为蟾蜍，在turnFamiliarIntoToad()方法中。

应该注意到了turnFamiliarIntoToad()方法的长度和缩进的数量。若在函数中出错的话会返回一只全新的蟾蜍，对这个特定的咒语来说是一个令人困惑(也是错误的！)的输出结果，接下来会在下一节中使用自定义的错误处理来修改代码。

### 使用Swift错误进行重构

> “Swift提供了优秀的对运行时的错误抛出、捕获、传递和可恢复操作的支持。”  -----The Swift Programming Language (Swift 3)

不要与末日神殿相混淆，Swifr中的金字塔是一种反模式的结构，需要使用多层嵌套的语句来实现控制流。在上面的turnFamiliarIntoToad()方法中可以看到 - 注意需要使用六个花括号才能闭合语句，要读懂这种多层嵌套的代码需要下一些功夫。

![](https://koenig-media.raywenderlich.com/uploads/2016/05/errorhandling-3-650x288.png)

之前见过的guard语句与多个并列的可选绑定类型可以帮助简化金字塔型的代码，不过使用do-cathch的话可以通过在处理错误状态时进行控制流去耦合以解决这个问题。

do-catch机制通常伴随着下面这些关键字:

* throws
* do
* catch
* try
* defer
* Error

在实际操作do-catch时会抛出不同的自定义错误。首先，需要定义期望处理的状态，使用枚举列出可能发生错误的情况。

在Witch的定义前面添加如下代码:
```swift
enum ChangoSpellError: Error {
  case hatMissingOrNotMagical
  case noFamiliar
  case familiarAlreadyAToad
  case spellFailed(reason: String)
  case spellNotKnownToWitch
}
```
注意关于ChangoSpellError的两件事:

* 它符合Error协议，在Swift中定义错误时的必要条件。
* 在spellFailed的情况下可以自行通过一个关联值设置咒语失败时的原因。

> 注意: ChangoSpellError的命名是在咒语“Presto Chango!”施放之后 —— 当一个魔女将一个仆从转化为蟾蜍时所使用的魔法。

好了，准备好施放一些魔法了，在方法名中添加throws来标识调用这个方法后可能产生的错误:

```swift
func turnFamiliarIntoToad() throws -> Toad {
```

在Magical协议中更新下这个方法名:

```swift
protocol Magical: Avatar {
  var name: String? { get set }
  var spells: [Spell] { get set }
  func turnFamiliarIntoToad() throws -> Toad
}
```

现在将错误都用枚举列出来了，需要重新编写turnFamiliarIntoToad()方法，分开处理每种错误情况。

### 处理魔法帽错误

首先，修改如下语句以确保魔女带着那顶至关重要的帽子:

```swift
if let hat = hat {
```
改成下面这样:
```swift
guard let hat = hat else {
  throw ChangoSpellError.hatMissingOrNotMagical
}
```
> 注意: 不要忘了移除底部关联的花括号，否则会编译错误。

下面这行是帽子属性的布尔值验证:

```swift
if hat.isMagical {
```
应当使用额外的guard语句来执行这个验证，不过将验证语句打包在一起用一行搞定的话会更加简洁。将第一个guard语句如下整合在一起:
```swift
guard let hat = hat, hat.isMagical else {
  throw ChangoSpellError.hatMissingOrNotMagical
}
```
现在可以删掉if hat.isMagical { 验证了。

下一部分来拆分一下条件语句金字塔。

### 处理仆从错误

接下来检查魔女是否拥有仆从:
```swift
if let familiar = familiar {
```
修改成如下使用guard语句的形式，若没有仆从则抛出一个.noFamiliar错误:
```swift
guard let familiar = familiar else {
  throw ChangoSpellError.noFamiliar
}
```
目前先忽略其他情况下的错误，之后代码会进行处理。

### 处理蟾蜍错误

在之后的代码中若魔女调用turnFamiliarIntoToad()咒语则返回已存在的蟾蜍，这里最好有一个能指出她操作"失误"的error。修改下面的代码:

```swift
if let toad = familiar as? Toad {
  return toad
}
```
改成如下这样:
```swift
if familiar is Toad {
  throw ChangoSpellError.familiarAlreadyAToad
}
```
注意，把as?改为了is，这样简化了验证语句，没必要使用判断结果来判断是否符合协议。is关键句也常用于类型比较。

把else的相关语句都删掉，不需要了~

### 处理咒语错误

最后，调用hasSpell(\_ type:)确保魔女吟唱的是正常的咒语，将下面的代码:
```swift
if hasSpell(ofType: .prestoChango) {
  if let name = familiar.name {
    return Toad(name: name)
  }
}
```
改成这样:
```swift
guard hasSpell(ofType: .prestoChango) else {
  throw ChangoSpellError.spellNotKnownToWitch
}
 
guard let name = familiar.name else {
  let reason = "Familiar doesn’t have a name."
  throw ChangoSpellError.spellFailed(reason: reason)
}
 
return Toad(name: name)
```
现在可以移除最后这行了:

```swift
return Toad(name: "New Toad")
```
这个方法变得更加简洁了，可以使用了。我在下面的代码中添加了一些注释，可以稍微解释下:
```swift
func turnFamiliarIntoToad() throws -> Toad {
 
  // When have you ever seen a Witch perform a spell without her magical hat on ? :]
  guard let hat = hat, hat.isMagical else {
    throw ChangoSpellError.hatMissingOrNotMagical
  }
 
  // 检查魔女是否拥有仆从
  guard let familiar = familiar else {
    throw ChangoSpellError.noFamiliar
  }
 
  // 检查仆从是否为蟾蜍 - 若是的话为何多此一举？
  if familiar is Toad {
    throw ChangoSpellError.familiarAlreadyAToad
  }
  guard hasSpell(ofType: .prestoChango) else {
    throw ChangoSpellError.spellNotKnownToWitch
  }
 
  // 检查仆从是否拥有名称
  guard let name = familiar.name else {
    let reason = "Familiar doesn’t have a name."
    throw ChangoSpellError.spellFailed(reason: reason)
  }
 
  // 检查完毕后返回一个与魔女仆从同名的蟾蜍
  return Toad(name: name)
}
```
你可以从turnFamiliarIntoToad()中返回一个可选类型来指示出“咒语吟唱时出错了”，不过使用自定义错误的话可以更明确地指出错误状态并作出对应的操作。

## 使用自定义错误的其他好处

现在你拥有了一个可以抛出自定义Swift错误的方法，接下来要处理它们，使用的这种机制一般被称为do-catch语句，类似其他语言比如Java中的try-catch机制。

在playground的底部添加如下代码:

```swift
func exampleOne() {
  print("") // 在调试区域添加一个空行
 
  // 1
  let salem = Cat(name: "Salem Saberhagen")
  salem.speak()
 
  // 2
  let witchOne = Witch(name: "Sabrina", familiar: salem)
  do {
    // 3
    try witchOne.turnFamiliarIntoToad()
  }
  // 4
  catch let error as ChangoSpellError {
    handle(spellError: error)
  }
  // 5
  catch {
    print("Something went wrong, are you feeling OK?")
  }
}
```
函数做了下面几件事:

1. 创建魔女的仆从，一只叫Salem的猫。
2. 创建魔女Sabrina。
3. 尝试把猫转换为蟾蜍。
4. 捕获到ChangoSpellError错误并适当地处理错误。
5. 最后捕获其它错误并打印一条信息。

添加完上述代码并运行后会看到一个编译错误 - 接着处理一下。

handle(spellError:)方法还没有定义，在exampleOne()函数的上方添加如下代码:
```swift
func handle(spellError error: ChangoSpellError) {
  let prefix = "Spell Failed."
  switch error {
    case .hatMissingOrNotMagical:
      print("\(prefix) Did you forget your hat, or does it need its batteries charged?")
 
    case .familiarAlreadyAToad:
      print("\(prefix) Why are you trying to change a Toad into a Toad?")
 
    default:
      print(prefix)
  }
}
```
最后在playground的地步添加如下代码调用并运行:

```swift
exampleOne()
```
点击Xcode工作区左下角的箭头图标展开调试控制台，就可以看到playground的输出了:

![](https://koenig-media.raywenderlich.com/uploads/2016/04/Expand-Debug-Area-1.gif)

### 错误捕获

下面是一些在上述代码片段中所出现的语言特性的简要介绍。

`catch`

可以在Swift中使用模式匹配处理特定错误或其它错误类型的主题。

上面的代码中使用了catch: 一个捕获特定的ChangoSpell错误，另外一个处理其余的错误情况。

`try`

使用try结合do-catch机制明确地指出哪一行或哪一块代码会抛出错误。

可以用如下几种方式使用try:

* try: 标准用法，也是上面所用到的。
* try?: 忽略要处理的错误，若抛出错误则返回nil。
* try!: 类似强制解绑，try!可用于加载某些确定存在的媒体文件，需谨慎使用。

是时候使用try?语句了，将下面的代码剪切并粘贴到playground的底部:
```swift
func exampleTwo() {
  print("") // 在调试区域添加一个空行
 
  let toad = Toad(name: "Mr. Toad")
  toad.speak()
 
  let hat = Hat()
  let witchTwo = Witch(name: "Elphaba", familiar: toad, hat: hat)
 
  print("") // 在调试区域添加一个空行
 
  let newToad = try? witchTwo.turnFamiliarIntoToad()
  if newToad != nil { // Same logic as: if let _ = newToad
    print("Successfully changed familiar into toad.")
  }
  else {
    print("Spell failed.")
  }
}
```
注意与exampleOne的不同，在这里并不关心发生的错误类型，不过当错误发生时还是会捕获。由于未创建蟾蜍，因此newToad的值是nil。

### 错误传递

`throws`

在Swift中，若一个函数或方法抛出错误时需要使用throw关键字，抛出的错误会自动传递给调用的堆栈，不过让错误冒泡的距离离源头太远通常被认为是不好的。代码库中显著的错误传递会增加错误未被有效处理的可能性，throws是对代码中记录传递过程的授权 - 给编程者留下传递的记录，以便有效的处理错误。

`rethrows`

至此为此看到的所有例子都用的是throws，没见过它的兄弟rethrows吧

rethrows告诉编译器仅当它的函数参数抛出错误时才抛出错误，下面是个简单的例子(不需要添加到playground中):

```swift
func doSomethingMagical(magicalOperation: () throws -> MagicalResult) rethrows -> MagicalResult {
  return try magicalOperation()
}
```
这里的doSomethingMagical(\_:)会在magicalOperation给函数抛出一个错误的情况下抛出错误，若是则返回一个MagicalResult。 

### 操作错误处理的行为

`defer`

虽然自动传递在大多数情况下是适用的，不过也有一些时候想将操作应用的行为像操作向上调用堆栈中的错误一样。

defer语句是一种允许每当跳出一个当前域时执行"清空"操作的机制，比如在一个方法或函数返回时。它有助于管理需要整理的资源，尽管有时操作会不成功，这在处理错误上下文时很有效。

来实践一下，给Witch添加如下方法:
```swift
func speak() {
  defer {
    print("*cackles*")
  }
  print("Hello my pretties.")
}
```
在playground底部添加如下代码:
```swift
func exampleThree() {
  print("") // Add an empty line in the debug area
 
  let witchThree = Witch(name: "Hermione", familiar: nil, hat: nil)
  witchThree.speak()
}
 
exampleThree()
```
在调试控制台中，会看到魔女在说完话后会咯咯地笑(先输出"Hello my pretties"然后输出"*cackles*")。

有趣的是，defer语句会以相反的顺序输出我们缩写的print代码内容。

在speak()中添加第二个defer语句，这样魔女会在说完话后先尖叫(screeches)后咯咯地笑(cackles)。
```swift
func speak() {
  defer {
    print("*cackles*")
  }
 
  defer {
    print("*screeches*")
  }
 
  print("Hello my pretties.")
}
```
这些语句如期按顺序输出了吗?是的，这就是defer的魅力!
