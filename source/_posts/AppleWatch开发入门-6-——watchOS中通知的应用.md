---
title: AppleWatch开发入门(6)——watchOS中通知的应用
catalog: true
date: 2018-11-21 19:03:42
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
**AppleWatch开发入门(6)——watchOS中通知的应用**
**{% post_link AppleWatch开发入门-7-——AlertController AppleWatch开发入门(7)——AlertController %}**
**{% post_link AppleWatch开发入门-8-——动画 AppleWatch开发入门(8)——动画 %}**


# 一、引言
在 iOS 系统中，支持的通知有两种类型：本地通知和远程通知。本地通知多用于计时类通知，远程的又称推送，多用于一些提示动态的提示信息。这里有相关通知的一些知识总结：

而 iWatch 中的通知是和 iPhone 同步的，在 iPhone 上的 App 收到通知的同时，会默认也推送到 iWatch 上，基于 iWatch的穿戴性，对用户来说，它上面的通知信息将比 iPhone 更加及时。

# 二、WatchOS通知概览
首先，iWatch 上的通知分为两部分：short-look 和 long-lock。

short-look 可以理解为一个简单的通知预览，它会将通知发起的APP和主要标题等信息展示给你，让你一目了然，当用户抬起手腕，查看这个通知一定时间，这个短通知就会转换为 long-look 通知。short-look 的通知界面不能够自定义，系统已经设计好了模样，如下：

![short-look](http://upload-images.jianshu.io/upload_images/2708793-13d97ded30f15fd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

long-look 的界面我们是可以进行一定程度上的自定义的，并且可以添加按钮等逻辑操作。

long-look 也分为两种界面，静态界面和动态界面。这个也好理解，静态界面是我们在写程序时就定义好的界面，在通知发送到watch上时，界面会自动匹配通知内容进行显示。动态的界面则是当收到通知时，会先执行我们相应的配置代码，之后在进行通知界面的展示。一个 long-look界面大致如下：

![long-look](http://upload-images.jianshu.io/upload_images/2708793-4a2b25adec90e9be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 long-lock 中，界面定义为三个部分，头部标题栏，自定义视图栏和按钮交互区。头部的标题栏我们不能自定义，它是一个半透明的上面有App图标和名字的横栏。其下面是我们可以自定义的区域，我们可以在storyBoard中拉入文本和图片。最下面是一些交互按钮，其名称等配置信息在推送的文件中定义。

# 三、在模拟器上测试本地推送

无论本地推送还是远程推送，需要注意一点，iWatch 上的消息推送是依赖于 iPhone，也就是说只有当 iPhone 收到推送时，iWatch 才能收到通知。所以，应先让 iPhone 能收到本地通知。

为方便测试，直接在 AppDelegate 中发送本地推送。
```
import UserNotifications

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // 消息中心
        let center = UNUserNotificationCenter.current()
        center.requestAuthorization(options: [.badge, .alert, .sound]) { (isSuccess, error) in
        if !isSuccess { print(error) }
        }
        // 5秒后触发消息推送
        let timeTrigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
        let requestIdentifier = "requestIdentifier"

        // 消息推送的内容
        let content = UNMutableNotificationContent()
        content.title = "通知来了"
        content.subtitle = "这里是一个测试用例通知的子标题"
        content.body = "这里是用于 Apple Watch 通知 Demo 的内容详情。"
        content.badge = 1
        content.sound = UNNotificationSound.default
        content.categoryIdentifier = requestIdentifier

        // 添加按钮
        let join = UNNotificationAction(identifier: "join", title: "接受邀请", options: .authenticationRequired)
        let look = UNNotificationAction(identifier: "look", title: "查看邀请", options: .foreground)
        let cancel = UNNotificationAction(identifier: "cancel", title: "取消", options: .destructive)
        let input = UNTextInputNotificationAction(identifier: "input", title: "请输入", options: .foreground, textInputButtonTitle: "发送", textInputPlaceholder: "tell me loudly")
        let notificationCategory = UNNotificationCategory(identifier: requestIdentifier, actions: [join, look, cancel], intentIdentifiers: [], options: .customDismissAction)
        var set = Set<UNNotificationCategory>()
        set.insert(notificationCategory)
        center.setNotificationCategories(set)

        let request = UNNotificationRequest(identifier: requestIdentifier, content: content, trigger: timeTrigger)
        center.add(request, withCompletionHandler: nil)

        return true
    }
}
```
然后运行程序。**注意：只有 iPhone 在锁屏的状态下，并且 iWatch 在 Home 界面时，iWatch 才能收到通知。**

![Apple Watch 收到来自 iPhone 的消息推送](https://upload-images.jianshu.io/upload_images/2708793-ddd5ac4debb17b0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 三、在真机上模拟远程推送
在 watchOS 模拟器上，Xcode 为我们准备好了一种可以模拟测试推送的方式。如果我们创建项目时，选择了 NotifacationScene，Xcode 会默认为我们创建一个 apns 文件：

![PushNotificationPayload.apns](https://upload-images.jianshu.io/upload_images/2708793-526c5e15591e06b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个文件就是模拟推送的相关配置文件，如果没有，也可以手动创建：

![](http://upload-images.jianshu.io/upload_images/2708793-b50a91f5cfc22902.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

文件中的内容格式如下：
```
{    
    "aps": {    
        "alert": {            
        "body": "通知",           
        "title": "通知来了"        
        },        
        "category": "requestIdentifier"    
    },       
    "WatchKit Simulator Actions": [
    {            
        "title": "First Button",            
        "identifier": "firstButtonAction"                                           
    }],        
    "customKey": "Use this file to define a testing payload for your notifications. The aps dictionary specifies the category, alert text and title. The WatchKit Simulator Actions array can provide info for one or more action buttons in addition to the standard Dismiss button. Any other top level keys are custom payload. If you have multiple such JSON files in your project, you'll be able to select them when choosing to debug the notification interface of your Watch App."
}
```
这是一些 Json 格式的数据，其中 alert 是对推送内容的设置，body 会显示在 long-look 的标题栏，title 会显示在 short-look 的标题栏，Actions 数组中是对按钮就行配置，每一个按钮可以设置一个标题和 id，标题用于在推送界面显示，id 用于处理点击按钮后触发的逻辑。注意：这里的 category 一定要和上文代码中设置的 requestIdentifier 一致。



创建好这个，我们可以来试着测试一下推送的界面，选择推送工程，运行即可。




# 四、long-look 的静态界面和动态界面
上面提到过，long-look 分为静态界面和动态界面两种，当我们在 storyboard 中拉入 Notification Interface Controller 的时候，可以选择同时创建动态界面，勾选 Has Dynamic Interface：

![](http://upload-images.jianshu.io/upload_images/2708793-fd664a944998e5d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时，在 storyboard 中是如下模样：

![](http://upload-images.jianshu.io/upload_images/2708793-6b7460ad81fa2381.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们在创建一个文件，继承于 WKUserNotificationInterfaceController，并将 storyboard 中`Dynamic Interface`的`class`设置为我们创建的类：

![](http://upload-images.jianshu.io/upload_images/2708793-570a4859f924ceae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意，这里设置的是动态的 Interface，也就是上面右边的 controller。之后运行，你会发现效果并没有什么改变，那是因为系统默认会从静态界面加载推送界面，我们需要在 NotifacationController 代码中做一些操作：
```
//在NotificationController中重写下面方法
override func didReceive(_ notification: UNNotification, withCompletion completionHandler: @escaping (WKUserNotificationInterfaceType) -> Void) {
    completionHandler(.custom) 
}
```

# 五、触发推送点击事件
首先，我们多配置几个点击按钮，在apns文件中如下配置：
```
"WatchKit Simulator Actions": [                                   
{                                   
    "title": "第一",                                   
    "identifier": "one"                                                                     
},                                  
{                                   
    "title": "第二",                                   
    "identifier": "two"                                                                     
},                                   
{                                   
    "title": "第三",                                   
    "identifier": "three"                                                                      
}                                   
],
```
在我们watch App的InterfaceController中实现如下的方法：
```
import UserNotifications
//重写下面方法来响应点击事件
override func handleAction(withIdentifier identifier: String?, for notification: UNNotification) {

}
```

现在，当你手机收到通知时，看看你的watch。
