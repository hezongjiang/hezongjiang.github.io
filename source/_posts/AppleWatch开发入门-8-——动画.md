---
title: AppleWatch开发入门(8)——动画
catalog: true
date: 2018-11-21 19:04:16
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
**{% post_link AppleWatch开发入门-7-——AlertController AppleWatch开发入门(7)——AlertController %}**
**AppleWatch开发入门(8)——动画**

# 一、简介
动画一直是 iOS 系统的一大亮点，CoreAnimation 和粒子效果的支持，开发者可以很容易的做出效果炫酷的动画特效。在 watchOS 中，由于性能和屏幕尺寸的限制，对于动画，并没有强大的框架支持，但是这并不是说开发者就没办法在watch上添加动画的特效了。

# 二、移动动画
在 watch 中的移动动画不像 iPhone 上那么复杂，目前只支持水平、垂直移动，并且移动的距离是受限的。水平移动的距离是一个枚举：`left`、`center`、`right`，也就是说你只能把控件移动到屏幕的左边、中间、右边，而不能控制具体移动到哪个位置；垂直方向移动的距离也一样是枚举：`top`、`center`、`bottom`，这两个枚举如下：
```
@available(watchOS 2.0, *)
public enum WKInterfaceObjectHorizontalAlignment : Int {
    case left
    case center
    case right
}
@available(watchOS 2.0, *)
public enum WKInterfaceObjectVerticalAlignment : Int {
    case top
    case center
    case bottom
}
```
执行动画的方法如下：
```
// 第一个参数是动画时间，第二个参数是闭包，里面写如何移动
@available(watchOS 2.0, *)
open func animate(withDuration duration: TimeInterval, animations: @escaping () -> Void)
```
代码实现非常简单：
```
@IBAction func startHorizontalAnimate() {
    // 水平移动到右边
    animate(withDuration: 2) {
        self.image.setHorizontalAlignment(.right)
    }
}
```

![水平移动](https://upload-images.jianshu.io/upload_images/2708793-12bc76dcce1f4da5.gif?imageMogr2/auto-orient/strip)

```
@IBAction func startVerticalAnimate() {
    // 垂直移动到底部
    animate(withDuration: 2) {
        self.image.setVerticalAlignment(.bottom)
    }
}
```

![垂直移动](https://upload-images.jianshu.io/upload_images/2708793-f2f5e6f90cb9f09c.gif?imageMogr2/auto-orient/strip)

这里比较有趣，我只是把图片往下移动了，结果 button 自动移上去。



# 三、帧动画
和 iOS 类似，watchOS 中的帧动画也是通过 UIImage 对象的合集来展示的。只是设置和用法略有不同。
首先，watchOS 中帧动画的操作被单独封装成了一个协议，当然，WKInterfaceImage 类已经遵守了这个协议的：
```
public protocol WKImageAnimatable : NSObjectProtocol {    
    //从默认帧开始播放动画    
    public func startAnimating()    
    //播放一个指定范围的帧动画 NSRange是帧的范围，durtion是播放一遍的时间，repeatCount是重复播放次数，0为无限循环    
    public func startAnimatingWithImages(in imageRange: NSRange, duration: TimeInterval, repeatCount: Int)
    //停止播放动画    
    public func stopAnimating()
}
```
创建帧动画的步骤与一些注意：
1、关联一个视图中的WKInterfaceImage对象
2、所有帧动画的图片帧必须有统一的格式：比如image1.png，image2.png等等
3、给WKInterfaceImage对象设置帧前缀：
```
image.setImageNamed("image")
```
**注意**：这里使用的方法和设置图片的方法一样，但是参数有别，图片的设置需要完整的图片名，动画帧前缀的设置只要设置帧图片的前缀，并且素材帧必须放入 WatchKit App 这个 Target 中，才可以使用。
4、开始动画
```
// range的location是图片的起始张，length是图片的总数
let range = NSRange(location: 1, length: 18)
image.startAnimatingWithImages(in: range, duration: 2, repeatCount: 0)
```

![帧动画](https://upload-images.jianshu.io/upload_images/2708793-49cc7df1f7752a85.gif?imageMogr2/auto-orient/strip)
