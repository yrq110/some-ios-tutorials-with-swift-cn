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
这个协议基本上被所有类和结构体继承了，为每个对象提供一个可视化的表现使其可以输出到控制台中。
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

> *“错误处理是一门并不优雅的艺术”*    –Swift Apprentice, Chapter 22 (Error Handing)

良好的错误处理可以增强终端用户的体验，使软件维护者可以更容易的发现问题、问题起因和引起的严重性。对代码中的错误处理针对性越强，代码就越容易维护。同时错误处理可以在系统层面以合适的方式处理异常，以免给用户带来糟糕的体验。

不过错误并不总是需要被处理的，有时语言的特性就会避免一些常见的错误。比如说，若有些语法特性可以完全避免一个错误的发生可以直接使用这种设计，若在一个无法避免潜在错误的场景下，针对性的处理是最好的选择。

## Avoiding Swift Errors Using nil

Since Swift has elegant optional-handling capabilities, you can completely avoid the error condition where you expect a value, but no value is provided. As a clever programmer, you can manipulate this feature to intentionally return nil in an error condition. This approach works best where you should take no action if you reach an error state; i.e. where you choose inaction over emergency action.

Two typical examples of avoiding Swift errors using nil are failable initializers and guard statements.

### Failable Initializers

Failable initializers prevent the creation of an object unless sufficient information has been provided. Prior to the availability of these initializers in Swift (and in other languages!), this functionality was typically achieved via the Factory Method Pattern.

An example of this pattern in Swift can be seen in create::
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
The above initializer tries to create a spell using the magic words provided, but if the words are not magical you return nil instead.

Inspect the creation of the spells at the very bottom of this tutorial to see this behavior in action:

![](https://koenig-media.raywenderlich.com/uploads/2016/07/Spell.create.png)

While first successfully creates a spell using the magic words "abracadabra", "ascendio" doesn’t have the same effect, and the return value of second is nil. (Hey, witches can’t win all the time).

![](https://koenig-media.raywenderlich.com/uploads/2016/05/errorhandling-2-650x256.png)

Factory methods are an old-school programming style. There are better ways to achieve the same thing in Swift. You’ll update the Spell extension to use a failable initializer instead of a factory method.

Delete create(_words:) and replace it with the following:

```swift
init?(words: String) {
  if let incantation = MagicWords(rawValue: words) {
    self.magicWords = incantation
  }
  else {
    return nil
  }
}
```
Here you’ve simplified the code by not explicitly creating and returning the Spell object.

The lines that assign first and second now throw compiler errors:
```swift
let first = Spell.create(withMagicWords: "abracadabra")
let second = Spell.create(withMagicWords: "ascendio")
```
You’ll need to change these to use the new initializer. Replace the lines above with the following:

```swift
let first = Spell(words: "abracadabra")
let second = Spell(words: "ascendio")
```
After this, all errors should be fixed and the playground should compile without error. With this change your code is definitely tidier – but you can do even better! :]

### Guard Statements

guard is a quick way to assert that something is true: for example, if a value is > 0, or if a conditional can be unwrapped. You can then execute a block of code if the check fails.

guard was introduced in Swift 2 and is typically used to (bubble, toil and trouble) bubble-up error handling through the call stack, where the error will eventually be handled. Guard statements allow early exit from a function or method; this makes it more clear which conditions need to be present for the rest of the processing logic to run.

To clean up Spell‘s failable initializer further, edit it as shown below to use guard:

```swift
init?(words: String) {
  guard let incantation = MagicWords(rawValue: words) else {
    return nil
  }
  self.magicWords = incantation
}
```
With this change, there’s no need need for an else clause on a separate line and and the failure case is more evident as it’s now at the top of the intializer. Also, the “golden path” is the least indented. The “golden path” is the path of execution that happens when everything goes as expected, i.e. no error. Being least indented makes it easy to read.

Note that the values of first and second Spell constants haven’t changed, but the code is more more streamlined.

## Avoiding Errors with Custom Handling
