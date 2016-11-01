# Building Your First Web App in Swift Using Vapor
## 使用Vapor构建你的第一个Web应用

***

>* 原文链接 : [Building Your First Web App in Swift Using Vapor](http://www.appcoda.com/server-side-swift-vapor/)
* 原文作者 : [SAHAND EDRISIAN](http://www.appcoda.com/author/sahandedrisian/)
* 译者 : []()

***

In WWDC 2015, Apple announced that Swift would be open source. Shortly after that, in December 2015, Swift’s codebase was public on [GitHub](https://github.com/apple/swift).

Open sourcing the Swift codebase introduces developers to a multitude of opportunities and expand the use of Swift worldwide.

As expected, developers swiftly explored the various new applications of Swift (no pun intended). One such application is using swift code to power a web app which we will explore in this tutorial.

## Why Should I Care?

There are many benefits to having a strictly typed language on the server. If you are creating a backend for your iOS app, it’s easier to stay consistent by sticking to one language and style.

There are three excellent server-side Swift services out there: Perfect, Kitura, and Vapor. In this tutorial, I will explore the interesting world of Swift on the server with Vapor.

I will teach you how to install Vapor and Swift 3, learn the basics of Server-Side Swift, and deploy your website/backend on Heroku.

## Prerequisites

In this tutorial, we will be using a lot of bash commands, so it is important to have a basic knowledge of bash and the Terminal. Vapor runs on Swift 3, thus Xcode 8 is required.

Vapor and any Swift (Swift Package Manager) project needs to run on Swift 3, so I recommend you to learn more about the changes in Swift 3 in this AppCoda article.

Towards the end of the tutorial, I will explain how to host your Vapor server on Heroku, a popular cloud hosting provider, so it is recommended to have some experience using it.

Now let’s get started!

## Installing Vapor

Vapor requires Swift 3, and the latest version of Xcode 8 beta. You can download the new Xcode GM here.

First, you have to choose the latest command line tools. Open you Xcode preferences.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s1.png)

Next, navigate to Locations.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s3-1024x653.png)

Lastly, choose the latest Xcode 8 command line toolset.

![](http://www.appcoda.com/wp-content/uploads/2016/09/s4-1024x666.png)

