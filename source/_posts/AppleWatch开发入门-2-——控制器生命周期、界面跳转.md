---
title: AppleWatch开发入门(2)——控制器生命周期、界面跳转
catalog: true
date: 2018-11-19 19:01:53
subtitle:
header-img: "AppleWatch.jpg"
tags: AppleWatch
---

本文章是一个系列，如果有兴趣可以看看以下文章：
**{% post_link AppleWatch开发入门-1-——界面布局 AppleWatch开发入门(1)——界面布局 %}**
**AppleWatch开发入门(2)——控制器生命周期、界面跳转**
**{% post_link AppleWatch开发入门-3-——Table视图的应用 AppleWatch开发入门(3)——Table视图的应用 %}**
**{% post_link AppleWatch开发入门-4-——Picker视图的应用 AppleWatch开发入门(4)——Picker视图的应用 %}**
**{% post_link AppleWatch开发入门-5-——Menu的使用 AppleWatch开发入门(5)——Menu的使用 %}**
**{% post_link AppleWatch开发入门-6-——watchOS中通知的应用 AppleWatch开发入门(6)——watchOS中通知的应用 %}**
**{% post_link AppleWatch开发入门-7-——AlertController AppleWatch开发入门(7)——AlertController %}**
**{% post_link AppleWatch开发入门-8-——动画 AppleWatch开发入门(8)——动画 %}**

# 一、引言
在前面，讨论了关于 iWatch 开发中框架与界面布局相关，主要的逻辑，终究还是要通过代码来实现的，在我们创建了项目之后，就会生成 InterfaceController 这个文件，它就是我们 storyboard 中的入口视图控制器。此外，在 iWatch 开发中，目前只能用 storyboard 来开发。

# 二、代码交互与控制器声明周期
storyboard 中的控件我们可以通过拖拽的方式关联到文件中，Action 和 Outlet 两种关联方式基本可以达到我们修改控件和处理业务逻辑的需求。

WKInterfaceController 类似于iOS中的 ViewController，是 iWatch 中主要用于展示界面的 Controller，我们的控件也都是基于这个容器中显示。在模板中，系统为我们提供了三个函数，这三个函数体现了watch一个界面的声明周期，如下：
```
    // 初始化界面时触发，通过context可以实现界面的传值    
    override func awake(withContext context: Any?) {
        super.awake(withContext: context)    
    }    
    // 界面即将展现时触发 类似于iOS中的ViewWillApear    
    override func willActivate() {        
        // This method is called when watch view controller is about to be visible to user        
        super.willActivate()   
    }    
    // 界面消失后触发，类似于iOS中的ViewDidDisAppear    
    override func didDeactivate() {        
        // This method is called when watch view controller is no longer visible        
        super.didDeactivate()   
    }
```
# 三、界面跳转与传值
与iOS类似，watchOS的界面跳转也有两种方式：modal 和 push。同样，我们也可以通过 storyboard 或者代码来进行跳转。

1、通过**代码**跳转与传值

我们创建两个InterfaceController和与之对应的类，界面如下：

![](http://upload-images.jianshu.io/upload_images/2708793-b015d9d92df1f436.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过代码跳转，**一定**要给第二个控制器设置一个id标识符：

![第二个控制器的属性界面，需要设置Identifier](http://upload-images.jianshu.io/upload_images/2708793-60a6910d31435fcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在按钮触发的方法中，如下跳转：

```
// modal
 @IBAction func presentMyController()   {     
    // 这里的context是传值的上下文，在被跳转控制器的awakeWithContext方法中会将这个值取到
    // InterfaceControllerTwo 是在上文提到的 Identifier
    presentController(withName: "InterfaceControllerTwo", context: "我是传的值")    
}
// push
@IBAction func pushMyController() {
    pushController(withName: "InterfaceControllerTwo", context: nil)
}
```

2、在 storyboard 中设置跳转关系

也可以直接在 storyboard 中设置界面的跳转，右键拖拽触发跳转事件的控件，拖拽到将要跳转的控制器，会出现 push 和 model 菜单，如下图。

![](http://upload-images.jianshu.io/upload_images/2708793-eb3766902fb7f143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过这种方式进行的跳转，在执行跳转之前，会执行如下这个函数：

```
// 这个方法的返回值就是 context 上下文传递的值。
override func contextForSegue(withIdentifier segueIdentifier: String) -> Any? {
    return "我是值"
}
```
