---
title: iOS Cell小动画
catalog: true
date: 2017-12-04 23:06:06
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
这里让后出现的Cell有一个小动画，稍微延迟一点出现，如下图：

![](http://upload-images.jianshu.io/upload_images/2708793-4b10e181c78b643b.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码的实现非常简单，就只有一个动画：

```
func tableView(_ tableView: UITableView, willDisplay cell: UITableViewCell, forRowAt indexPath: IndexPath) {
    cell.transform = CGAffineTransform(translationX: 0, y: 30)
    UIView.animate(withDuration: 0.8) {
        cell.transform = .identity
    }
}
```