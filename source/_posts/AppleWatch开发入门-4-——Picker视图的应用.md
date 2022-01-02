---
title: AppleWatch开发入门(4)——Picker视图的应用
catalog: true
date: 2018-11-21 19:02:45
subtitle:
header-img: "AppleWatch.jpg"
tags: AppleWatch
---

本文章是一个系列，如果有兴趣可以看看以下文章：
**{% post_link AppleWatch开发入门-1-——界面布局 AppleWatch开发入门(1)——界面布局 %}**
**{% post_link AppleWatch开发入门-2-——控制器生命周期、界面跳转 AppleWatch开发入门(2)——控制器生命周期、界面跳转 %}**
**{% post_link AppleWatch开发入门-3-——Table视图的应用 AppleWatch开发入门(3)——Table视图的应用 %}**
**AppleWatch开发入门(4)——Picker视图的应用**
**{% post_link AppleWatch开发入门-5-——Menu的使用 AppleWatch开发入门(5)——Menu的使用 %}**
**{% post_link AppleWatch开发入门-6-——watchOS中通知的应用 AppleWatch开发入门(6)——watchOS中通知的应用 %}**
**{% post_link AppleWatch开发入门-7-——AlertController AppleWatch开发入门(7)——AlertController %}**
**{% post_link AppleWatch开发入门-8-——动画 AppleWatch开发入门(8)——动画 %}**

# 一、简介

Picker到底是个什么东西？如下图所示：

![](http://upload-images.jianshu.io/upload_images/2708793-581a2976deb0a630.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2708793-829dc9d96c0434b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Picker有3种模式：List、Stack、Sequence：
1、List：有点像tableview，一行一行展示
2、Stack：有点像图片浏览器，一页一页展示，翻动时有动画效果
3、Sequence：与Stack类似，只不过翻动时没有动画效果

# 二、创建Picker
在 storyboard 中拖入 Picker：

![](http://upload-images.jianshu.io/upload_images/2708793-926aaaec7ddf3092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为Picker有3种模式（List、Stack、Sequence），我们每种模式弄一个：

![](http://upload-images.jianshu.io/upload_images/2708793-1f54b0b20b6a8fb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、代码部分

工欲善其事，必先利其器，呃，必先看头文件：

```
@available(watchOS 2.0, *)
public class WKInterfacePicker : WKInterfaceObject {
      // 设置聚焦，有点像becomeFirstResponder
      public func focus()
      // 失去聚焦，有点像resignFirstResponder
      public func resignFocus()
      // 改变选中的索引
      public func setSelectedItemIndex(_ itemIndex: Int)
      // 设置Picker中的内容
      public func setItems(_ items: [WKPickerItem]?)
      public func setCoordinatedAnimations(_ coordinatedAnimations: [WKInterfaceObject]?)
      public func setEnabled(_ enabled: Bool)
}
@available(watchOS 2.0, *)
public class WKPickerItem : NSObject, NSSecureCoding {
    // 设置文字，仅当Picker是'List'时有效.
    public var title: String?
    // Caption to show for the item when focus style includes a caption callout.
    public var caption: String?
    // 设置title前面的小图片，仅当Picker是'List'时有效，图片大小固定是13×13pt。
    @NSCopying public var accessoryImage: WKImage?
    // 大图
    @NSCopying public var contentImage: WKImage?
}
```
具体实现
1、 Pick 中的数据源是 WKPickerItem，先懒加载` [WKPickerItem]`
```
lazy var items: [WKPickerItem] = {
        var its = [WKPickerItem]()
        for i in 0...4 {
            let item = WKPickerItem()
            item.title = "Title" + "\(i)"
            item.caption = "Caption" + "\(i)"
            item.accessoryImage = WKImage(image: UIImage(named: "ad_0" + "\(i)")!)
            item.contentImage = WKImage(image: UIImage(named: "ad_0" + "\(i)")!)
            its.append(item)
        }
        return its
    }()
```
这里图片资源大家根据实际需求添加，加载图片时要特别注意，否则可能加载不出，详细见：[AppleWatch开发（1）——界面布局](http://www.jianshu.com/p/7c223446040c) 这篇文章的末尾部分。

2、 连线
```
@IBOutlet var listPicker: WKInterfacePicker!
@IBOutlet var stackPicker: WKInterfacePicker!
@IBOutlet var sequencePicker: WKInterfacePicker!
```

3、 设置数据
```
override func awake(withContext context: Any?) {
        super.awake(withContext: context)
        
        listPicker.setItems(items)
        stackPicker.setItems(items)
        sequencePicker.setItems(items)
        sequencePicker.focus()
    }
```
这里要注意，在有多个Picker时，需要设置`focus()`，不然系统不知道你要滚动哪一个。
