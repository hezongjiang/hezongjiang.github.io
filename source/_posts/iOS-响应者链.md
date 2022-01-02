---
title: iOS 响应者链
catalog: true
date: 2019-02-19 17:06:06
subtitle:
header-img: Demo.png
tags:
- iOS开发
---

## 一、概述
iOS 响应者链(Responder Chain)是支撑 App 界面交互的重要基础，点击、滑动、旋转、摇晃等都离不开其背后的响应者链链。

简单的说（虽然不准确），响应者链的作用就是让 APP 知道用户点击里了哪里，然后应该哪个控件做出反应。专业点说，响应者链就是由多个响应者组合起来的链条，就叫做响应者链。它表示了每个响应者之间的联系，并且可以使得一个事件可选择多个对象处理。

## 二、事件的产生和传递
当一个触摸事件产生的时候，程序是如何找到第一响应者的呢？也就是说程序怎么知道点击了哪个控件呢？

![事件的产生与传递](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfj5ivg1j30gk08q74s.jpg)

当点击了屏幕会产生一个触摸事件，消息循环(runloop)会接收到触摸事件放到消息队列里，UIApplication 会从消息队列里取事件分发下去，接着需要找到去响应这个事件的最佳视图，也就是 Responder，所以开始的第一步应该是找到 Responder，那么又是如何找到的呢？那就不得不引出 UIView 的 2 个方法： 
```
// 返回此次触摸事件初始点所在的视图
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event 

// 返回视图是否包含指定的某个点
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event 
```
通过在显示视图层级中依次对视图调用这个 2 个方法来确认该视图是不是能响应这个点击的点，**首先会调用 `hitTest`，然后在 `hitTest` 中调用 `pointInside`，最终 `hitTest` 返回的那个 `view` 就是最终的响应者Responder**。

![image.png](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfjo8oq0j30du0f23zj.jpg)

以上图为例，假设点击了 `View3`。

首先调用 UIWindow 的 `hitTest:withEvent:`，在这个方法中调用 `pointInside:withEvent:` 方法判断点击是否在 UIWindow 的范围内，显然 `pointInside` 返回 `YES`；

然后遍历 window 的子视图，来到 RootView，调用 RootView 的 `hitTest` 和 `pointInside`，因为点击发生在 RootView 中所以继续遍历它的子视图。

从 View2 开始的，调用 View2 的 `hitTest` 和 `pointInside`，由于触摸点在 View2 的范围内，所以 `pointInside` 返回 `YES`；然后继续遍历 View2 的子视图，从 View4 开始，因为点击不发生在 View4， 所以 `pointInside` 返回 `NO`，而 View4 没有子视图，所以 `hitTest` 返回 `null`；然后继续在 View2 的另外一个子视图 View3 中调用 `hitTest` 和 `pointInside`，因为点击的就是 View3 所以 `pointInside` 返回 `YES`，且 View3 没有子视图，所以 `hitTest` 返回了自己 `View3`；接着 `View2` 的 `hitTest` 也返回 `View3`，RootView 的 `hitTest` 也返回 View3，直到 UIWindow 也返回 View3，自此我们找到了响应视图：View3。

![打印过程](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfjw70shj30xc0fqdvd.jpg)

**小结：**
1. 寻找事件的响应视图是通过调用视图的 `hitTest` 和 `pointInside` 完成的。
2. `hitTest` 的调用顺序是从 UIWindow 开始，对视图的每个子视图依次调用。
3. 遍历直到找到响应视图，然后逐级返回最终到 UIWindow 返回此视图，哪怕还有未遍历到的视图，也不会去遍历了。

## 三、响应者

所有继承 UIResponder 的控件都有一个 nextResponder 属性，此属性会返回在 Responder Chain 中的下一个事件处理者，如果每个 Responder 都不处理事件，那么事件将会被丢弃。所以继承自 UIResponder 的子类便会构成一条响应者链，所以我们可以打印下以 View3 为开始的响应者链是什么样的：

![image.png](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfk2p381j30xc08bn3b.jpg)

可以看到响应者链一直延伸到 AppDelegate，View3 的下一个是 View2，也就是 View3 的父视图，View2 下一个是 RootView，也是父视图，而 RootView 的下一个则是 Controller。

**所以下一个响应者的规则是：**如果有父视图，则 nextResponder 指向父视图，如果是控制器根视图则指向控制器，控制器如果在导航控制器中，则指向导航控制器的相关显示视图，最后指向导航控制器，如果是根控制器则指向 UIWindow，UIWindow 的 nexResponder 指向 UIApplication，最后指向 AppDelegate，而他们实现这一套指向都是靠重写 nextReponder 实现的。

继续接着上面的例子，当点击了 View3，先是由 UIWindow 通过 `hitTest` 返回所找到响应者 View3；接着会执行 View3 的 `touchesBegan`，然后是通过 `nextResponder` 依次是 View2、RootView，完全是按照 `nextResponder` 链条的调用顺序，`touchesEnded` 也是同样的顺序。

![touches 执行顺序](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfk7fqc4j30xc092gu4.jpg)

上面是 View3 不处理点击事件的情况，接下来我们为 View3 添加一个点击事件处理，看看又会是什么样的调用过程：

![touches 执行顺序](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfkh2rnnj30xc08x46f.jpg)

可以看到 `touchesBegan` 顺着 `nextResponder` 链条调用了，但是 View3 处理了事件，去执行了相关是事件处理方法，而 `touchesEnded` 并没有得到调用。

**小结：**
1. 找到最适合的响应视图后事件会从此视图开始沿着响应链   `nextResponder` 传递，直到找到处理事件的视图,如果没有处理的事件会被丢弃。 

2. 如果视图有父视图则 `nextResponder` 指向父视图，如果是根视图则指向控制器，最终指向 `AppDelegate`, 他们都是通过重写 `nextResponder` 来实现。

![响应者链](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gfkpvoogj30go09mjsz.jpg)


以上是视图可以正常点击的情况，但在一些情况下，视图无法点击，这时候响应者链会如何传递呢？

**无法点击是的情况总结：**
1. `alpha = 0`、子视图的 `frame` 超出父视图、`userInteractionEnabled = NO`、`hidden = YES` 这些情况下，视图会被忽略，不会调用 `hitTest`。 
2. 若父视图被忽略，后其所有子视图也会被忽略。换句话说，若父视图不可点击，则其子视图一定不能点击。

## 四、应用示例
1、点击透传 

RootView 有 2 个重叠在一起的子视图 View1 和 View2，View2 覆盖在 View1 上面，如何做到点击 View1 触发 View2 的处理逻辑？ 
很简单，设置 View2 的 `userInteractionEnabled = NO` 即可。

2、限定点击区域 

给定一个显示为圆形的视图，实现只有在点击区域在圆形里面才视为有效。 
我们可以重写View的pointInside方法来判断点击的点是否在圆内，也就是判断点击的点到圆心的距离是否小于等于半径就可以。
```
@implementation CircleView
- (void)awakeFromNib {
    [super awakeFromNib];
    self.layer.cornerRadius = self.frame.size.width / 2.0f;
}

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event {
    const CGFloat radius = self.frame.size.width / 2.0f;
    CGFloat xOffset = point.x - radius;
    CGFloat yOffset = point.y - radius;
    CGFloat distance = sqrt(xOffset * xOffset + yOffset * yOffset);
    return distance <= radius;
}
@end
```