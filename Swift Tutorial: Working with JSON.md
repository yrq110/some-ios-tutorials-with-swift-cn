#Swift Tutorial: Working with JSON
##Swift指南：JSON的使用
[原文地址](https://www.raywenderlich.com/120442/swift-json-tutorial)
> 更新至Xcode 7.1与Swift 2   12-21-2015

[JavaScript Object Notation](http://www.json.org/)或简称JSON，是一种与Web端服务进行数据传递的通用方法。它具有易用与可读性强的特点，这也是它为何会这么火的原因。

考虑一下下面这段JSON：

~~~~
[
  {
    "person": {
      "name": "Dani",
      "age": "24"
    }
  },
  {
    "person": {
      "name": "ray",
      "age": "70"
    }
  }
]
~~~~

在Objective-C中，对JSON的解析与反序列化是很明确的：
~~~~
NSArray *json = [NSJSONSerialization JSONObjectWithData:JSONData options:kNilOptions error:nil];
NSString *age = json[0][@"person"][@"age"];
NSLog(@"Dani's age is %@", age);
~~~~
在Swift中，由于Swift的可选类型与类型安全，同样的操作显得更加繁琐：
~~~~
var json: Array!
do {
  json = try NSJSONSerialization.JSONObjectWithData(JSONData, options: NSJSONReadingOptions()) as? Array
} catch {
  print(error)
}
 
if let item = json[0] as? [String: AnyObject] {
  if let person = item["person"] as? [String: AnyObject] {
    if let age = person["age"] as? Int {
      print("Dani's age is \(age)")
    }
  }
}
~~~~
在上面的代码中，当你使用JSON中的每一个object之前都需要通过可选绑定进行检查。这会保证你代码的安全性，但是若你的JSON越复杂，你的代码就会变得越丑。

使用Swift2.0中的guard语句可以帮助避免if语句的嵌套：
~~~~
guard let item = json[0] as? [String: AnyObject],
  let person = item["person"] as? [String: AnyObject],
  let age = person["age"] as? Int else {
    return;
}
print("Dani's age is \(age)")
~~~~
还是太冗长，不是吗？你怎样才能使它更简洁？

在这篇JSON指南中，你会学到一种更轻松的解析JSON的方法——使用开源库[Gloss](https://github.com/hkellaway/Gloss)

你将会使用Gloss去解析一个包含US App商店Top25应用信息的JSON文档。你将会发现在Objective-C中也能轻易的使用！

##入门
下载这篇指南的[开始playground](http://www.raywenderlich.com/wp-content/uploads/2015/11/TopApps-Starter.zip)

由于用户界面在这篇JSON指南中并不是很重要，因此你将只使用playgrounds进行学习。

在Xcode中打开Swift.playground看一看。

>你可能会发现playground工程的导航默认是关闭的。使用`command`+`1`来显示它。

在开始playground文件中有一些source与resource文件，会使你更加专注于使用Swift进行JSON的解析。看一看它的结构与概要：
* Resources文件夹绑定了你的Swift代码中要访问的资源文件
  * topapps.json：包含需要解析的JSON字符串
* Sources文件夹包含你的主playground可以访问的其他Swift远吗文件。在Sources文件夹中添加额外的.swift辅助文件会使你的playground变得更加简洁与易读。
  * App.swift：
  * DataManager.swift：
