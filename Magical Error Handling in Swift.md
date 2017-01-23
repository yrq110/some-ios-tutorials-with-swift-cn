# Magical Error Handling in Swift
## 奇妙的Swift错误处理

***

>* 原文链接 : [Magical Error Handling in Swift](https://www.raywenderlich.com/130197/magical-error-handling-swift)
* 原文作者 : [Gemma Barlow](https://www.raywenderlich.com/u/gemmakbarlow)
* 译者 : [yrq110](https://github.com/yrq110/)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/05/errorhandling-1-feature-250x250.png)

Error handling in Swift has come a long way since the patterns in Swift 1 that were inspired by Objective-C. Major improvements in Swift 2 made the experience of handling unexpected states and conditions in your application more straightforward. These benefits continue in Swift 3, but there are no significant updates to error handling made in the latest version of the language. (Phew!)

Just like other common programming languages, preferred error handling techniques in Swift can vary, depending upon the type of error encountered, and the overall architecture of your app.

This tutorial will take you through a magical example involving wizards, witches, bats and toads to illustrate how best to deal with common failure scenarios. You’ll also look at how to upgrade error handling from projects written in earlier versions of the language and, finally, gaze into your crystal ball at the possible future of error handling in Swift!

> Note: This tutorial assumes you’re familiar with Swift 3 syntax – particularly enumerations and optionals. If you need a refresher on these concepts, start with the What’s New in Swift 2 post by Greg Heo, and the other materials linked.

Time to dive straight in (from the the cauldron into the fire!) and discover the various charms of error handling in Swift 3!

## Getting Started

There are two starter playgrounds for this tutorial, one for each section. Download Avoiding Errors with nil – Starter.playground and Avoiding Errors with Custom Handling – Starter.playground playgrounds.

Open up the Avoiding Errors with nil starter playground in Xcode.

Read through the code and you’ll see several classes, structs and enums that hold the magic for this tutorial.

Take note the following parts of the code:

```swift
protocol Avatar {
  var avatar: String { get }
}
```
This protocol is applied to almost all classes and structs used throughout the tutorial to provide a visual representation of each object that can be printed to the console.
```swift
enum MagicWords: String {
  case abracadbra = "abracadabra"
  case alakazam = "alakazam"
  case hocusPocus = "hocus pocus"
  case prestoChango = "presto chango"
}
```
This enumeration denotes magic words that can be used to create a Spell.
```swift
struct Spell {
  var magicWords: MagicWords = .abracadabra
}
```
This is the basic building block for a Spell. By default, you initialize its magic words to .abracadabra.
Now that you’re acquainted with the basics of this supernatural world, you’re ready to start casting some spells.

## Why Should I Care About Error Handling?

> *“Error handling is the art of failing gracefully.”*    –Swift Apprentice, Chapter 22 (Error Handing)

Good error handling enhances the experience for end users as well as software maintainers by making it easier to identify issues, their causes and their associated severity. The more specific the error handling is throughout the code, the easier issues are to diagnose. Error handling also lets systems fail in an appropriate way so as not to frustrate or upset users.

But errors don’t always need to be handled. When they don’t, language features let you avoid certain classes of errors altogether. As a general rule, if you can avoid the possibility of an error, take that design path. If you can’t avoid a potential error condition, then explicit handling is your next best option.

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
