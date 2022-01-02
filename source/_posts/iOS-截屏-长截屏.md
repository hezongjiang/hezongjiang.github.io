---
title: iOS 截屏&长截屏
catalog: true
date: 2017-08-18 23:21:32
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
**截屏**在 iOS 开发中经常用到，本篇文章讲的是**监听用户截屏操作**，并且**获取截屏图片**，如果当前是UIScrollView或者UIWebView，则为获取整个scrollView的截图内容，效果图如下：

![截屏](http://upload-images.jianshu.io/upload_images/2708793-f0540b36e04a3a57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 一、监听用户截屏
iOS7 开始提供一个新的监听方法：UIApplicationUserDidTakeScreenshotNotification。只要像往常一样监听系统通知即可。
```
// 监听屏幕截图
NotificationCenter.default.addObserver(self, selector: #selector(notify(_:)), name: NSNotification.Name.UIApplicationUserDidTakeScreenshot, object: nil)
```
**注意：**截图完成后，如果你想从通知的`userInfo`中获取截屏图片，那就错了，你会发现`userInfo`中是空的，所以需要用代码实现截屏操作，以此获取图片。

## 二、截屏
这里给`UIView`添加一个分类，这样，只要是一个`UIView`的子类，都可以截图了。

```
import UIKit

extension UIView {
    
    /// 截屏
    ///
    /// - Parameters:
    ///   - frame: 需要截取的范围，若为nil，则为视图的大小
    func screenshot(frame: CGRect? = nil) -> UIImage? {
        
        if let scrollView = self as? UIScrollView { // 如果是UIScrollView

            return scrollViewScreenShot(scrollView, frame: frame)

        } else if let webView = self as? UIWebView { // 如果是UIWebView

            let scrollView = webView.scrollView
            return scrollViewScreenShot(scrollView, frame: frame)

        } else {
            let shotFrame: CGRect = (frame == nil) ? self.bounds : frame!
            UIGraphicsBeginImageContextWithOptions(shotFrame.size, true, 0) 
            guard let currentContext = UIGraphicsGetCurrentContext() else { return nil }

            currentContext.translateBy(x: -shotFrame.origin.x, y: -shotFrame.origin.y)
            let path = UIBezierPath(rect: shotFrame)
            path.addClip()
            layer.render(in: currentContext)
            let image = UIGraphicsGetImageFromCurrentImageContext()
            UIGraphicsEndImageContext()
            return image
        }
    }
    
    /// 截取scrollView的内容
    ///
    /// - Parameters:
    ///   - scrollView: scrollView
    ///   - frame: 截取范围
    /// - Returns: 截取的图片
    private func scrollViewScreenShot(_ scrollView: UIScrollView, frame: CGRect?) -> UIImage? {
        
        let shotFrame: CGRect = (frame == nil) ? CGRect(origin: CGPoint(), size: scrollView.contentSize) : frame!
        
        UIGraphicsBeginImageContextWithOptions(shotFrame.size, false, 0)
        
        let savedContentOffset = scrollView.contentOffset
        let savedFrame = scrollView.frame
        
        scrollView.contentOffset = CGPoint()
        scrollView.frame = CGRect(origin: CGPoint(), size: scrollView.contentSize)
        
        guard let currentContext = UIGraphicsGetCurrentContext() else { return nil }
        
        currentContext.translateBy(x: -shotFrame.origin.x, y: -shotFrame.origin.y)
        
        let path = UIBezierPath(rect: shotFrame)
        path.addClip()
        
        scrollView.layer.render(in: currentContext)
        
        let image = UIGraphicsGetImageFromCurrentImageContext()
        
        UIGraphicsEndImageContext()
        
        scrollView.contentOffset = savedContentOffset
        scrollView.frame = savedFrame
        
        return image
    }
}
```

[这是本文的Demo，你不点一下吗？](https://github.com/Hearsayer/ScreenShot)