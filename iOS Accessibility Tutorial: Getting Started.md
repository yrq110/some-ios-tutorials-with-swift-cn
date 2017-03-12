# iOS Accessibility Tutorial: Getting Started
## iOS Accessibility教程：入门
***

>* 原文链接 : [iOS Accessibility Tutorial: Getting Started](https://www.raywenderlich.com/142058/ios-accessibility-tutorial)
* 原文作者 : [Vincent Ngo](https://www.raywenderlich.com/u/jomoka)
* 译者 : [yrq110](https://github.com/yrq110)

***

![](https://koenig-media.raywenderlich.com/uploads/2016/10/Accessibility-feature-1.png)

Developers constantly strive to make amazing, intuitive and immersive apps for users. But are developers really building for every possible user? Can everyone truly use your app to its full ability?

An accessible approach to designing apps, products or any kind of services lets people use those products — regardless of ability. This includes people with vision, motor, learning, or hearing disabilities.

Apple continues to provide developers with new and updated tools to design with accessibility in mind. Take a moment to watch this inspiring introductory video from Apple (8 mins).

In this iOS accessibility tutorial you will start off with a finished recipe app and transform it to become more accessible. You will learn the following:
* How to use VoiceOver.
* How to use the Accessibility Inspector.
* How to use the accessibility elements with UIKit.
* How to build a better user experience for people with disabilities.
This iOS accessibility tutorial requires Xcode 8 and Swift 3. It assumes you already know the basics of Swift development. If you’re new to Swift, you should first check out our book Swift Apprentice.

> Note: You’ll need a physical device to work with VoiceOver — this accessibility feature is not supported in the simulator.

## Getting Started

The app you will be enhancing is a simple recipe app, which shows you the difficulty level of each recipe and lets you rate what you made.

Start by downloading the starter project. Open Recipe.xcodeproj in the Recipe folder. Before you can run on a device, you need to configure signing.

Click on the Recipe project in the navigator, then select the target with the same name. Select the General tab and in the Signing section select your Team from the drop down.

![](https://www.raywenderlich.com/?attachment_id=143051)

Now build and run the app to become familiar with its features. The root controller is a table view of recipes containing an image, description, and difficulty rating. Click into a recipe for a larger picture with the recipe’s ingredients and instructions.

To make things more exciting, you can also cross off the items in the list to make sure you are on top of your cooking game! If you love or hate what you made, you can even toggle the like/dislike emoji.

![](https://koenig-media.raywenderlich.com/uploads/2016/08/showRecipeApp.gif)

Spend a few minutes becoming familiar with the code in the starter. Here are some highlights:
* RecipeInstructionViewController.swift contains the controller code for the detail view that shows a large image of the dish along with ingredients and instructions for cooking. You use a UISegmentedControl to toggle between recipes and instructions in the table view backed by InstructionViewModel.
* InstructionViewModel.swift acts as the data source for RecipeInstructionsViewController. It includes descriptions for ingredients and instructions as well as state information for the check boxes.
* RecipeCell.swift is the cell for the root controller’s recipe list. It handles display of the recipe’s difficulty level, name and photo based on the passed Recipe model object.
* Recipe.swift is the model object representing a recipe. It contains utility methods for loading an array of recipes that is used throughout the app.
* InstructionCell.swift defines a cell that contains a label and checkbox for use in instructions and ingredient lists. When the box is checked, it adds a strikethrough to the text.
* RecipeListViewController.swift manages the root table view which displays the list of all recipes available using an array of Recipe objects as the data source.
* Main.storyboard contains all storyboard scenes for the app. You will notice that all UI components are standard UIKit controls and views. UIKit controls and views are already accessible – making your job easier.
Now that you have an understanding of how the app works, you’ll begin to add some accessibility features.

## Why Accessibility?
Before you get started with the code, it’s important to understand the benefits of accessibility.
* Designing your app for accessibility makes it easier to write functional tests, whether you’re using the KIF framework, or UI Testing in Xcode.
* You’ll also broaden your market and user base by making your app usable by a larger group.
* If you work for any government agencies, you’re required to implement 508 compliance, which states that any software or technology must be accessible to all users.
* Plus, it plain feels good to know you’re making a small but noticeable difference in someone’s life! :]
