---
title: iOS 粒子效果—— CAEmitterLayer 粒子发射器
catalog: true
date: 2017-09-01 23:13:52
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
## 一、粒子发射器
在UIKit中，粒子系统由两部分组成：
1. **CAEmitterLayer** ：发射的层，主要控制粒子的形状和发射的位置（例如，在矩形内，或边缘）。这个层具有全局的乘法器，可以施加到系统内的CAEmitterCells。
2. **CAEmitterCell** ：单个粒子的原型。当散发出一个粒子，UIKit根据这个发射粒子和定义的基础上创建一个随机粒子。此原型包括一些属性来控制粒子的图片，颜色，方向，运动，缩放比例和生命周期。

**CAEmitterLayer** 继承自CALayer，提供了一个基于Core Animation的粒子发射系统，粒子用CAEmitterCell来初始化，一个单独的CAEmitterLayer可同时支持多个CAEmitterCell。

![雪花效果](http://upload-images.jianshu.io/upload_images/2708793-e7cf2f428e1022f2.gif?imageMogr2/auto-orient/strip)

## 二、CAEmitterLayer 与 CAEmitterCell 重要属性
1. **CAEmitterLayer** 属性
```
/// 发射位置
var emitterPosition: CGPoint

/// 发射源的尺寸大小，其实这个 emitterSize 结合 emitterPosition 构建了发射源的位置及大小的矩形区域 rect
var emitterSize: CGSize

/**
  发射模式，这个字段规定了在特定形状上发射的具体形式是什么：
  kCAEmitterLayerPoint：发射效果为在发射点发射粒
  kCAEmitterLayerOutline：整个边框都是发射点，即边框进行发射
  kCAEmitterLayerSurface：边框包含下的区域进行抛洒
  kCAEmitterLayerVolume：这个的效果和kCAEmitterLayerSurface很像
*/
var emitterMode: String

/**
  发射源的形状，这个字段规定了发射源的形状：
  kCAEmitterLayerPoint：发射源的形状是一个点，位置在上面position设置的位置
  kCAEmitterLayerLine：发射源的形状是一条线，位置在rect的横向的位于垂直方向中间那条
  kCAEmitterLayerRectangle：发射源是一个矩形，就是上面生成的那个矩形rect
  kCAEmitterLayerCuboid：发射源是一个立体矩形，这里要生效的话需要设置z方向的数据，如果不设置就同矩形状
  kCAEmitterLayerCircle：发射源是一个圆形，形状为矩形包裹的那个圆，二维的
  kCAEmitterLayerSphere：三维的圆形，同样需要设置z方向数据，不设置则通二维一样
*/
var emitterShape: String
```
2. **CAEmitterCell** 属性
```
/// 粒子产生速率
var birthRate: Float

/// 粒子存在时长
var lifetime: Float

/// 存在时长浮动范围，代表存在时间会在这个数字内浮动
var lifetimeRange: Float

/// 运动速度与运动速度浮动范围
var velocity: CGFloat
var velocityRange: CGFloat

/// 抛洒的角度
var emissionLongitude: CGFloat
var emissionRange: CGFloat

/// x/y/z方向的 加速度
var xAcceleration: CGFloat
var yAcceleration: CGFloat
var zAcceleration: CGFloat

/// 缩放大小、范围、速度
var scale: CGFloat
var scaleRange: CGFloat
var scaleSpeed: CGFloat

/// 旋转
var spin: CGFloat
var spinRange: CGFloat

/// 颜色，RGBA变化范围
var color: CGColor?
var redRange: Float
var greenRange: Float
var blueRange: Float
var alphaRange: Float
```

## 三、雪花效果实现
```
import UIKit

class SnowView: UIView {
    
    fileprivate lazy var snowEmitter = CAEmitterLayer()

    override init(frame: CGRect) {
        super.init(frame: frame)
        
        snowEmitter.emitterPosition = CGPoint(x: bounds.width * 0.5, y: 0)
        snowEmitter.emitterSize = bounds.size
        snowEmitter.emitterMode = kCAEmitterLayerOutline
        snowEmitter.emitterShape = kCAEmitterLayerLine
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    func snow() {
        
        let snowflake = CAEmitterCell()
        
        snowflake.name = "snow"
        // 粒子产生速率，越大，出现的越快
        snowflake.birthRate = 4
        // 存活时间
        snowflake.lifetime = 50
        // 粒子速度
        snowflake.velocity = 10
        // 粒子速度范围
        snowflake.velocityRange = 10
        // 粒子y方向的加速度分量
        snowflake.yAcceleration = 20
        // 周围发射角度
        snowflake.emissionRange = 0.5 * .pi
        // 子旋转角度范围
        snowflake.spinRange = 0.3 * .pi
        // 粒子图片
        snowflake.contents = UIImage(named: "snow")?.cgImage
        // 粒子缩放范围
        snowflake.scaleRange = 0.5
        // 粒子颜色
        snowflake.color = UIColor(red: 0.5, green: 0.5, blue: 0.5, alpha: 1).cgColor
        snowflake.redRange = 0.5
        snowflake.greenRange = 0.5
        snowflake.blueRange = 0.5
        
        snowflake.alphaSpeed = -0.01
        
        // 将粒子添加到粒子发射器上
        snowEmitter.emitterCells = [snowflake]
        layer.insertSublayer(snowEmitter, at: 0)
    }
}
```