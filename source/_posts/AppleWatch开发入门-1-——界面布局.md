---
title: AppleWatch开发入门(1)——界面布局
catalog: true
date: 2018-11-18 19:01:05
subtitle:
header-img: "AppleWatch.jpg"
tags: AppleWatch
---

本文章是一个系列，如果有兴趣可以看看以下文章：
**AppleWatch开发入门(1)——界面布局**
**{% post_link AppleWatch开发入门-2-——控制器生命周期、界面跳转 AppleWatch开发入门(2)——控制器生命周期、界面跳转 %}**
**{% post_link AppleWatch开发入门-3-——Table视图的应用 AppleWatch开发入门(3)——Table视图的应用 %}**
**{% post_link AppleWatch开发入门-4-——Picker视图的应用 AppleWatch开发入门(4)——Picker视图的应用 %}**
**{% post_link AppleWatch开发入门-5-——Menu的使用 AppleWatch开发入门(5)——Menu的使用 %}**
**{% post_link AppleWatch开发入门-6-——watchOS中通知的应用 AppleWatch开发入门(6)——watchOS中通知的应用 %}**
**{% post_link AppleWatch开发入门-7-——AlertController AppleWatch开发入门(7)——AlertController %}**
**{% post_link AppleWatch开发入门-8-——动画 AppleWatch开发入门(8)——动画 %}**

# 一、简介

在 iPhone 开发中，最基本的布局方式是通过 frame 将控件的位置和大小固定在屏幕上，后来，由于手机屏幕的尺寸有了略微变化，有了 autoresizing 布局框架，可以设置子视图随父视图的改变做一些相应的变化，再后来，iPhone 尺寸与分辨率也越来越多，适配各个屏幕也成为了 iOS 开发者遇到的新的问题，幸运的是，autolayout 机制的出现，大大减少在适配方面的成本。但是在 iWatch 的布局方式中，**需要抛弃 iPhone 中的布局思路**，接受一套不太一样的布局框架。

首先，iWatch 的屏幕不大，目前有 38mm、40mm、42mm 和 44mm 四个尺寸，无法在这个有限的空间里做非常复杂的界面效果，因此，在开发中应便于使用、一目了然。iWatch 上的布局方式采用的是一种平面堆放的方式，没有 frame，也没有约束，控件的布局方式只是一个挨着一个的平面堆放，也不可重叠。但在 iWatch 中提供了 Group 这样一种布局方式，可以让我们在布局中体现自由与个性的方面。

# 二、创建项目工程
创建工程就不做详细描述了，直接下一步就行。

![WX20181121-091446@2x.png](https://upload-images.jianshu.io/upload_images/2708793-565bd24f021cfd81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

                 
# 三、基础的堆放布局
打开 `WatchKit APP ` 中的 `storyboard`，这就是整个 iWatch 布局的地方。在 iWatch 开发中，目前只能用 storyboard 来开发。

iWatch 的布局采用的是最基础的堆放方式，从上到下依次排开，例如，我们添加三个 Label，效果如下：

![](https://upload-images.jianshu.io/upload_images/2708793-50572be7c36cdef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过拖动控件改变顺序，注意，这里仅可改变其上下位置。

这种方式的布局高度并没有限制，我们可以一直往下排列，在 iWatch上则会出现上下滑动的效果。

# 四、Group 进行复杂界面布局
通过上面的布局方式，我们只能进行纵向的排列布局，这并不能达到我们的需求，WatchKit 中提供那一套布局的模型：Group。

可以这样理解，Group 默认透明，它的主要作用是提供布局，可以将屏幕分成了几各分区，我们可以设置各个分区的排列方式，例如水平或者垂直，通过这样的思路，完成复杂的 iWatch 界面布局。

直接拖一个 Group 到界面上，然后将需要水平布局的控件放在 Group 里面：

![Group 布局](https://upload-images.jianshu.io/upload_images/2708793-75b9988b4c9b2aa5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后运行程序

![](https://upload-images.jianshu.io/upload_images/2708793-9af726303e9050c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**扩展：所谓Group**
Group在界面布局上，不仅可以起到分区屏幕的作用，其还可以设置一些属性来使布局更加漂亮。在storyBoard右侧的设置菜单中，我们可以对这些属性进行操作：
* Layout：设置布局模式，分为 水平布局 和 垂直布局 两种；
* Insert：可以设置内容区域偏移量，通过这个属性，我们可以使其中填充的控件四周留白；
* Spacing：其中填充的控件的间距；
* Background：设置Group的背景图案；
* Mode：设置背景图案的填充方式；
* Animate：出现时带动画；
* Color：设置Group的背景颜色；
* Radius：设置Group的圆角度。

# 四、布局中控件的位置和尺寸设置
对于控件的尺寸，有三种模式，控件的 Width 和 Height 都是通过这三个模式设置的：
在 iPhone 中，通过 frame 或者约束来控制控件的位置和尺寸，而在 iWatch 中则简单很多，尺寸和位置都是固定的模式，我们只需要做一些设置即可。
1. 控件尺寸的控制
**Size To Fit Content**：控件尺寸与内容相关，例如，Label 中字数的多少决定了 Label 的尺寸。
![Size To Fit Content](http://upload-images.jianshu.io/upload_images/2708793-521619e55206a654.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Relative to Container**：控件尺寸按照容器尺寸的比例设置。例如设置为 0.5，则当前控件的尺寸就是容纳其 Group 的一半。
![Relative to Container](http://upload-images.jianshu.io/upload_images/2708793-a1e625bf48c1f2a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**Fixed**：设置一个固定的值。
![Fixed](http://upload-images.jianshu.io/upload_images/2708793-b0b837ca508cb30a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 控件位置的控制
因为 iWatch 的界面十分简洁，对于控件的位置设置，是通过水平和垂直两个维度来设置的，通过设置每个维度的属性来控制其在容纳它的父控件中的位置：
![位置控制](https://upload-images.jianshu.io/upload_images/2708793-a0cd2b6faaa4a4f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
# 五、图片设置
图片的使用有大坑。

关于图片素材，你可以发现，在 Extension 和 App 文件夹中各有一个 Assets.xcasssets 组，只有将素材放入这里面，iWatch才能使用。

![Assets.xcasssets](https://upload-images.jianshu.io/upload_images/2708793-81826d82ae72177e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

加载图片的方法有一下几种，这几种方法也有大坑。
```
@available(watchOS 2.0, *)
open class WKInterfaceImage : WKInterfaceObject, WKImageAnimatable {

    open func setImage(_ image: UIImage?)

    open func setImageData(_ imageData: Data?)

    open func setImageNamed(_ imageName: String?)

    open func setTintColor(_ tintColor: UIColor?)
}
```

图片使用一定要特别注意：
**1. 在 Interface.storyboard 中，只能加载 WatchKit App 中 Assets.xcassets 的图片。**
**2. `func setImageNamed(_ imageName: String?)`此方法只能加载 WatchKit App 中 Assets.xcassets 的图片。**
**3. `func setImage(_ image: UIImage?)`此方法只能加载 WatchKit Extension 中 Assets.xcassets 的图片。**
**4. `func setImageData(_ imageData: Data?)`此方法只能加载 WatchKit Extension 中 Assets.xcassets 的图片。**


![iWatch 图片加载方法](https://upload-images.jianshu.io/upload_images/2708793-ec48a1f8213cb54d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
