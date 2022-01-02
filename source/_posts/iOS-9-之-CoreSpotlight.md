---
title: iOS 9 之 CoreSpotlight
catalog: true
date: 2017-09-07 23:12:26
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
## 一、CoreSpotlight 简介
iOS9 推出了 **CoreSpotlight** 框架，这个框架可以为 iOS 的搜索 **App 内部**的数据，能够使我们在 iPhone 上下拉出现得搜索框中，搜索我们使用的 **App 里面**的内容（当然App必须做了适配我们才能搜索到）。而在 iOS 9 之前这是一个私有的API，只有系统自己能搜索，并且无法搜索到我们 App 里面的内容。如下图：我们输入一个“海”字，就能搜索到哪些 App 中包含“海”的信息。
![CoreSpotlight 搜索](http://upload-images.jianshu.io/upload_images/2708793-12fe03aef8e27403.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

对于 **CoreSpotlight** 可以类比 NSUserDefault，都是全局的存储空间。不同的是 **CoreSpotlight** 是系统的存储空间，而
 NSUserDefault 是每个 App 私有的，其他 App 无法访问。另外对于存储的内容
 **CoreSpotlight** 存储的是 item，即 CSSearchableItem，而每个
 CSSearchableItem 又有许多属性，这些属性是通过
 CSSearchableItemAttributeSet 进行设置。

## 二、Core Spotlight 使用样例

![基本描述](http://upload-images.jianshu.io/upload_images/2708793-595d1935e0dbece5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)


**1. 基本使用**
```
// 创建索引属性
let set = CSSearchableItemAttributeSet(itemContentType: kUTTypeData as String)
set.title = "This is spotlight's title, and you can write long word in here"
set.contentDescription = "There are can write description for this search, yes, I'm long long description~~~~"
set.keywords = ["simple"]
let item = CSSearchableItem(uniqueIdentifier: "uniqueIdentifier", domainIdentifier: "domainIdentifier", attributeSet: set)
CSSearchableIndex.default().indexSearchableItems([item], completionHandler: nil)
```
![最基本使用](http://upload-images.jianshu.io/upload_images/2708793-fcaad411fe1f531c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

注意：这里没有设置Logo，会自动默认用 App 的 icon 作为 Logo。

**2. 图片与星评**
```
let set = CSSearchableItemAttributeSet(itemContentType: kUTTypeData as String)
set.title = "盗梦空间"
set.contentDescription = "《盗梦空间》是由克里斯托弗·诺兰执导，莱昂纳多·迪卡普里奥，玛丽昂·歌迪亚等主演的电影。影片剧情游走于梦境与现实之间，被定义为“发生在意识结构内的当代动作科幻片”。影片讲述由莱昂纳多·迪卡普里奥扮演的造梦师，带领约瑟夫·高登-莱维特、艾伦·佩吉扮演的特工团队，进入他人梦境，从他人的潜意识中盗取机密，并重塑他人梦境的故事。"
set.keywords = ["盗梦空间", "daomengkongjian"]
set.thumbnailData = UIImagePNGRepresentation(UIImage(named: "timg")!)
set.rating = 4.5
set.ratingDescription = "4.1分"
let item = CSSearchableItem(uniqueIdentifier: "uniqueIdentifier", domainIdentifier: "domainIdentifier", attributeSet: set)
CSSearchableIndex.default().indexSearchableItems([item], completionHandler: nil)
```
![图片与星评](http://upload-images.jianshu.io/upload_images/2708793-737bf3f19056c494.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

**3. 显示时间**
```
// 注意类型
let set = CSSearchableItemAttributeSet(itemContentType: kUTTypeMessage as String)
set.title = "盗梦空间"
set.contentDescription = "《盗梦空间》是由克里斯托弗·诺兰执导，莱昂纳多·迪卡普里奥，玛丽昂·歌迪亚等主演的电影。影片剧情游走于梦境与现实之间，被定义为“发生在意识结构内的当代动作科幻片”。影片讲述由莱昂纳多·迪卡普里奥扮演的造梦师，带领约瑟夫·高登-莱维特、艾伦·佩吉扮演的特工团队，进入他人梦境，从他人的潜意识中盗取机密，并重塑他人梦境的故事。"
set.keywords = ["盗梦空间", "daomengkongjian"]
set.contentCreationDate = Date()
let item = CSSearchableItem(uniqueIdentifier: "uniqueIdentifier", domainIdentifier: "domainIdentifier", attributeSet: set)
CSSearchableIndex.default().indexSearchableItems([item], completionHandler: nil)
```
![显示时间](http://upload-images.jianshu.io/upload_images/2708793-263b10802f7fdad9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

**4. 导航**
```
set.latitude = 123
set.longitude = 23
set.supportsNavigation = true
```
![导航](http://upload-images.jianshu.io/upload_images/2708793-846022009852b684.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

**5. 拨打电话**
```
set.phoneNumbers = ["18300000000", "17300000000"]
set.supportsPhoneCall = true
```
![显示电话](http://upload-images.jianshu.io/upload_images/2708793-119491abe37a1f83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/500)

注意：虽然 phoneNumbers 属性是一个数组，但是在测试中发现，当点击电话按钮时，只会拨打第一个电话，并且只在真机上有效，模拟器无效

**6. 删除索引**
```
func deleteSearchableItems(withIdentifiers identifiers: [String], completionHandler: ((Error?) -> Swift.Void)? = nil)

func deleteSearchableItems(withDomainIdentifiers domainIdentifiers: [String], completionHandler: ((Error?) -> Swift.Void)? = nil)

func deleteAllSearchableItems(completionHandler: ((Error?) -> Swift.Void)? = nil)
```

**7. 监听点击**
在 `AppDelegate` 中实现以下代理方法：
```
func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([Any]?) -> Swift.Void) -> Bool
```