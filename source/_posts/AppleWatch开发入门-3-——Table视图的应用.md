---
title: AppleWatch开发入门(3)——Table视图的应用
catalog: true
date: 2018-11-20 19:02:20
subtitle:
header-img: "AppleWatch.jpg"
tags: AppleWatch
---

本文章是一个系列，如果有兴趣可以看看以下文章：
**{% post_link AppleWatch开发入门-1-——界面布局 AppleWatch开发入门(1)——界面布局 %}**
**{% post_link AppleWatch开发入门-2-——控制器生命周期、界面跳转 AppleWatch开发入门(2)——控制器生命周期、界面跳转 %}**
**AppleWatch开发入门(3)——Table视图的应用**
**{% post_link AppleWatch开发入门-4-——Picker视图的应用 AppleWatch开发入门(4)——Picker视图的应用 %}**
**{% post_link AppleWatch开发入门-5-——Menu的使用 AppleWatch开发入门(5)——Menu的使用 %}**
**{% post_link AppleWatch开发入门-6-——watchOS中通知的应用 AppleWatch开发入门(6)——watchOS中通知的应用 %}**
**{% post_link AppleWatch开发入门-7-——AlertController AppleWatch开发入门(7)——AlertController %}**
**{% post_link AppleWatch开发入门-8-——动画 AppleWatch开发入门(8)——动画 %}**

# 一、简介

iWatch 中的 Table 和 iPhone 中的 UITableView 还是有很大的区别，在开发之前，我们应该明白 WatchOS 中 Table 的局限性和特点。
1. Table只有行的概念，没有分区的概念，没有头尾视图的概念。
2. 可以通过创建多个Table，来实现分区的效果。
3. 因为 iWatch 上是通过 Gruop 进行布局适应的，所以没有行高等设置。
4. Table 没有代理，所有行的数据都是采用静态配置的方式。
5. 点击 Table 中的行触发的方法，是通过重写 Interface 中的方法来实现的。

# 二、创建一个Table
在 storyboard 中拖入 Table，如下：

![](http://upload-images.jianshu.io/upload_images/2708793-3bf50de52787c21f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 Table 上放两个 Label：

![](http://upload-images.jianshu.io/upload_images/2708793-a93f316c080c7bc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个 Table 中包含一个 TableRowController，实际上我们 Table 上的控件都是通过 TableRowController 进行管理的，因此如果我们需要在代码中控制 TableRow 上的内容，我们需要新建一个类作为 Table 的 TableRowController，继承自 NSObject:

![](http://upload-images.jianshu.io/upload_images/2708793-439265b54532f5a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将 storyboard 中的 TableRowController 修改为我们创建的类，并指定 Identifier：

![选中刚创建的TableRowController](https://upload-images.jianshu.io/upload_images/2708793-b8d2dcf0924c6adb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中 TableRowController 时，注意不要选中 Group，同时 Group 也可以调整控件的布局方式。


![修改为自己新建的类](https://upload-images.jianshu.io/upload_images/2708793-adc231313fc82ca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![指定 Identifier](https://upload-images.jianshu.io/upload_images/2708793-622eb49c3c90a30b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后，将两个 Label 关联到 TableRowController 中：

```
import WatchKit
class TableRowController: NSObject {
    @IBOutlet var titleLabel: WKInterfaceLabel!
    @IBOutlet var numberLabel: WKInterfaceLabel!
}
```
 
将 Table 关联到 InterfaceController 中：
````
class InterfaceControllerMain: WKInterfaceController {        
    @IBOutlet var table: WKInterfaceTable!
}
````
 
下面，我们开始在interface中对Table做相关配置，WatchOS的Table配置非常简单易用，例如我们如下配置：
```
override func awake(withContext context: Any?) {
    super.awake(withContext: context)    
    let dict = [["name" : "中国银行", "money" : "￥20000"],
    			["name" : "中国邮政储蓄", "money" : "￥1100"],
    			["name" : "中国建设银行", "money" : "￥1000"],
                ["name" : "中国农业银行", "money" : "￥5000"],
                ["name" : "中国工商银行", "money" : "￥2000"],
                ["name" : "中国招商银行", "money" : "￥4010"]]
        
    table.setNumberOfRows(dict.count, withRowType: "TableRow")
        
    for (idx, info) in dict.enumerated() {
        let cell = table.rowController(at: idx) as! TableRowController
        cell.titleLabel.setText(info["name"])
        cell.numberLabel.setText(info["money"])
    }
}
```
 
这样一个展示银行卡余额的界面我们就创建完成了，效果如下：

![](http://upload-images.jianshu.io/upload_images/2708793-5e45268a24cdc7b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、关于Table的点击事件
上面我们提到，Table 没有所谓代理方法，点击 cell 的时候，我们也是通过两种方式进行逻辑跳转的。
一种是在 storyboard 中，通过拖拽 TableRowController 拉线跳转，这时如需传值，我们需在 Interface 中重写如下方法：
```
open func contextForSegue(withIdentifier segueIdentifier: String, in table: WKInterfaceTable, rowIndex: Int) -> Any?
```


另一种方式，重写如下方法，来处理Table的点击事件：
```
open func table(_ table: WKInterfaceTable, didSelectRowAt rowIndex: Int)
```
无论哪种方式，我们都可以通过参数 table 和 rowIndex 来确认点击的具体是那个table和哪一行，进行传值和处理我们的逻辑。             
