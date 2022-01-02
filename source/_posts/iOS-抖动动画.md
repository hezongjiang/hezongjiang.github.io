---
title: iOS 抖动动画
catalog: true
date: 2018-09-28 23:09:31
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
```
/// 抖动方向枚举
public enum ShakeDirection {
    /// 水平抖动
    case horizontal
    /// 垂直抖动
    case vertical
}

extension UIView {
    /// 扩展UIView增加抖动方法
    ///
    /// - Parameters:
    ///   - direction: 抖动方向（默认是水平方向）
    ///   - times: 抖动次数（默认6次）
    ///   - interval: 每次抖动时间（默认0.1秒）
    ///   - delta: 抖动偏移量（默认4）
    ///   - completion: 抖动动画结束后的回调
    public func shake(direction: ShakeDirection = .horizontal, times: Int = 6, interval: TimeInterval = 0.1, delta: CGFloat = 4, completion: ((Bool) -> ())? = nil) {
        
        UIView.animate(withDuration: interval, animations: {
            switch direction {
            case .horizontal:
                self.layer.setAffineTransform( CGAffineTransform(translationX: delta, y: 0))
            case .vertical:
                self.layer.setAffineTransform( CGAffineTransform(translationX: 0, y: delta))
            }

        }) { (_) in
            
            if (times == 0) { // 最后一次抖动，将位置还原，并调用完成回调函数
                UIView.animate(withDuration: interval, animations: {
                    self.layer.setAffineTransform(CGAffineTransform.identity)
                }, completion: completion)
            } else { // 不是最后一次抖动，继续播放动画（总次数减1，偏移位置反向）
                self.shake(direction: direction, times: times - 1,  interval: interval, delta: delta * -1, completion:completion)
            }
        }
    }
}
```