---
title: AppleWatch开发入门(7)——AlertController
catalog: true
date: 2018-11-21 19:03:58
subtitle:
header-img: "AppleWatch.jpg"
tags: AppleWatch
---

本文章是一个系列，如果有兴趣可以看看以下文章：
**{% post_link AppleWatch开发入门-1-——界面布局 AppleWatch开发入门(1)——界面布局 %}**
**{% post_link AppleWatch开发入门-2-——控制器生命周期、界面跳转 AppleWatch开发入门(2)——控制器生命周期、界面跳转 %}**
**{% post_link AppleWatch开发入门-3-——Table视图的应用 AppleWatch开发入门(3)——Table视图的应用 %}**
**{% post_link AppleWatch开发入门-4-——Picker视图的应用 AppleWatch开发入门(4)——Picker视图的应用 %}**
**{% post_link AppleWatch开发入门-5-——Menu的使用 AppleWatch开发入门(5)——Menu的使用 %}**
**{% post_link AppleWatch开发入门-6-——watchOS中通知的应用 AppleWatch开发入门(6)——watchOS中通知的应用 %}**
**AlertController AppleWatch开发入门(7)——AlertController**
**{% post_link AppleWatch开发入门-8-——动画 AppleWatch开发入门(8)——动画 %}**

# 一、简介
AlertController是wiatch中的警告框，它有3种Style：
* alert
* actionsheet
* sldeBySldeButtonsAlert

3中Style分别如下图所示：

![Alert Style](http://upload-images.jianshu.io/upload_images/2708793-1b6e9e6a815d8eda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Actionsheet Style](http://upload-images.jianshu.io/upload_images/2708793-f5799399df3a7f10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![sldeBySldeButtonsAlert Style](http://upload-images.jianshu.io/upload_images/2708793-8024d687626056ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里需要注意的是：
* `Alert`和`Actionsheet`按钮数量不限，而`sldeBySldeButtonsAlert`只能有两个按钮，如果设置了多个按钮，则无法弹出。
* `Alert`取消按钮只能有一个，如果设置多个，也只有一个的效果。

# 二、创建3种AlertController
1. 界面布局与连线
如下图：
![](http://upload-images.jianshu.io/upload_images/2708793-7012038820d073eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 详细代码
```
@IBAction func click1() {
    alertView(style: .actionSheet)
}
@IBAction func click2() {
    alertView(style: .alert)
}
@IBAction func click3() {
    alertView(style: .sideBySideButtonsAlert)
}

func alertView(style: WKAlertControllerStyle) {
    // 创建一个取消按钮
    let action1 = WKAlertAction(title: "取消按钮", style: .cancel) {
        print("点击了取消按钮")
    }
    // 创建一个默认按钮
    let action2 = WKAlertAction(title: "默认按钮", style: .default) {
        print("点击了默认按钮")
    }
    // 创建一个警告按钮
    let action3 = WKAlertAction(title: "警告按钮", style: .destructive) {
        print("点击了警告按钮")
    }
    // 弹出alert，设置标题和信息，并把上面创建的按钮加入到alert中
    presentAlert(withTitle: "消息标题", message: "在这里可以输入消息的内容", preferredStyle: style, actions: [action1, action2, action3])
}
```
至此，已经大功告成，不过要注意，`sideBySideButtonsAlert `只能有两个按钮，所以在`click3`中要进行特殊处理。
