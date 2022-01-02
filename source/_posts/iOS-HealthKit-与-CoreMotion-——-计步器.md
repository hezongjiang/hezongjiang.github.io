---
title: iOS HealthKit 与 CoreMotion —— 计步器
catalog: true
date: 2019-01-13 16:35:15
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
## 一、HealthKit 简介
**HealthKit** 是 Apple 在 iOS 8 推出的一款与健康有关的 API ，它提供了一种优雅的方式来获取和存储用户的健康数据。在 iOS 8 Apple 加入了一个健康APP，用来整合不同来源的数据。这些数据包括个人的身高、体重、血型等基本信息，步行+跑步距离、步数等健身信息等。

在本篇 **HealthKit** 教程中，将会创建一个简单地记录用户信息的 App。主要功能：
1. 怎么样向用户请求允许来获得 **HealthKit** 的数据。
2. 怎样通过 **HealthKit** 获取用户数据，包括获取步数、行走路程、性别。
3. 怎么样将数据写回 **HealthKit** ，如设置体重。

## 二、HealthKit 准备工作
**1、隐私策略**
由于苹果一些重要的隐私策略改变，我们需要在info.plist文件中声明苹果健康的使用权限，所以在info.plist中添加以下key就可以了。请求写入和请求读取都需要添加！
```
// 请求写入
<key>NSHealthUpdateUsageDescription</key>
     NSHealthUpdateUsageDescription
<string>请求更新健康信息</string>
```
```
// 请求读取
<key>NSHealthShareUsageDescription</key>
<string>请求获取健康信息</string>
```

**2、开启HealthKit**

![开启HealthKit](http://upload-images.jianshu.io/upload_images/2708793-a6a0373fa6f64174.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、HealthKit Let's Coding
**1、代码初始化**
新建一个`HealthKitManage`类，继承于`NSObject`，并且引入`HealthKit`库，懒加载`healthStore `
```
import HealthKit
class HealthKitManage: NSObject {
    private lazy var healthStore = HKHealthStore()
}
```
**2、授权**
申请获取读取用户的健康信息
```
/// 授权申请
private func authorizeHealthKit() {
    // 判读是否支持‘健康’
    if HKHealthStore.isHealthDataAvailable() {
        healthStore.requestAuthorization(toShare: dataTypesToWrite(), read: dataTypesRead()) { (isAuthorize, error) in
            print(isAuthorize)
            print(error)
        }
    } else {
        print("can't support!")
    }
}

/// 获取‘写’取权限，配置需要向用户的健康中写入的数据
private func dataTypesToWrite() -> Set<HKSampleType>? {
        
    guard let height = HKObjectType.quantityType(forIdentifier: .height),
        let weight = HKObjectType.quantityType(forIdentifier: .bodyMass),
        let temperature = HKObjectType.quantityType(forIdentifier: .bodyTemperature),
        let activeEnergy = HKObjectType.quantityType(forIdentifier: .basalEnergyBurned) else { return nil }
        
    var set = Set<HKQuantityType>()
        
    set.insert(height)
    set.insert(weight)
    set.insert(temperature)
    set.insert(activeEnergy)
        
    return set
}
    
/// 获取‘读’权限，可以读取用户健康中的哪些信息
private func dataTypesRead() -> Set<HKObjectType>? {
        
    guard let height = HKObjectType.quantityType(forIdentifier: .height),
        let weight = HKObjectType.quantityType(forIdentifier: .bodyMass),
        let temperature = HKObjectType.quantityType(forIdentifier: .bodyTemperature),
        let birthday = HKObjectType.characteristicType(forIdentifier: .dateOfBirth),
        let sex = HKObjectType.characteristicType(forIdentifier: .biologicalSex),
        let stepCount = HKObjectType.quantityType(forIdentifier: .stepCount),
        let distance = HKObjectType.quantityType(forIdentifier: .distanceWalkingRunning),
        let activeEnergy = HKObjectType.quantityType(forIdentifier: .basalEnergyBurned) else { return nil }
        
    var set = Set<HKObjectType>()
    set.insert(height)
    set.insert(weight)
    set.insert(temperature)
    set.insert(birthday)
    set.insert(sex)
    set.insert(stepCount)
    set.insert(distance)
    set.insert(activeEnergy)
        
    return set
}
```
**3、将`HealthKitManage`类，设计为单例**
```
static let shared = HealthKitManage()
private override init() {
    super.init()
    // 授权判断
    authorizeHealthKit()
}
```
**4、获取步数**
```
/// 获取步数
///
/// - Parameters:
///   - beginTime: 获取计步的开始时间
///   - endTime: 获取计步的结束时间
///   - complented: 完成回调，注意是在子线程回调
func getStepCount(beginTime: Date, endTime: Date, complented: @escaping ((_ step: Int) -> ())) {
        
    guard let step = HKObjectType.quantityType(forIdentifier: .stepCount) else { return }
    let timeSortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierEndDate, ascending: false)
    let predicate = HKQuery.predicateForSamples(withStart: beginTime, end: endTime)
    let query = HKSampleQuery(sampleType: step, predicate: predicate, limit: HKObjectQueryNoLimit, sortDescriptors: [timeSortDescriptor]) { (query, results, error) in
            
        guard let results = results as? [HKQuantitySample] else { return }
        var step = 0
        for quantitySample in results {
            let quantity = quantitySample.quantity
            let heightUnit = HKUnit.count()
            let usersHeight = quantity.doubleValue(for: heightUnit)
            step += Int(usersHeight)
        }
        complented(step)
    }
    healthStore.execute(query)
}
```
**5、获取行走路程**
```
/// 获取行走路程
///
/// - Parameters:
///   - beginTime: 获取路程的开始时间
///   - endTime: 获取路程的结束时间
///   - complented: 完成回调，注意是在子线程回调
func getDistance(beginTime: Date, endTime: Date, complented: @escaping (_ distance: Double) -> ()) {
        
    guard let distanceType = HKObjectType.quantityType(forIdentifier: .distanceWalkingRunning) else { return }
        let timeSortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierEndDate, ascending: false)
        let predicate = HKQuery.predicateForSamples(withStart: beginTime, end: endTime)
        let query = HKSampleQuery(sampleType: distanceType, predicate: predicate, limit: HKObjectQueryNoLimit, sortDescriptors: [timeSortDescriptor]) { (qurry, samples, error) in

        guard let quantitySamples = samples as? [HKQuantitySample] else { return }
            
        var totalDistance: Double = 0
        for quantitySample in quantitySamples {
            let quantity = quantitySample.quantity
            let distanceUnit = HKUnit.meterUnit(with: .kilo)
            let dis = quantity.doubleValue(for: distanceUnit)
            totalDistance += dis
        }
        complented(totalDistance)
    }
    healthStore.execute(query)
}
```
**6、获取性别**
```
/// 获取性别
func getGender() -> HKBiologicalSex {        
    return (try! healthStore.biologicalSex()).biologicalSex
}
```
**7、设置体重**
```
/// 设置体重
func setWeight() {
        
    let quantityType = HKQuantityType.quantityType(forIdentifier: .bodyMass)!
    let weight: Double = 0
    let quantity = HKQuantity(unit: HKUnit(from: .gram), doubleValue: weight)
    let weightSample = HKQuantitySample(type: quantityType, quantity: quantity, start: Date(), end: Date())
        
    healthStore.save(weightSample) { (success, error) in
        if success {
            print("体重设置成功")
        } else {
            print("体重设置失败")
        }
    }
}
```

## 四、CoreMotion
如果你修改`健康`中的步数，微信健康里的步数并不会改变，说明微信不是用 **HealthKit** 来获取步数的，有可能是用 **CoreMotion** 来实现的，这里的数据是无法修改的。
```
import CoreMotion

class ViewController: UIViewController {
    
    private lazy var pedometer = CMPedometer()

    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {

        if !CMPedometer.isStepCountingAvailable() { return }
        
        pedometer.queryPedometerData(from: Date(timeIntervalSinceNow: -3 * 24 * 3600), to: Date()) { (data, error) in
            
            guard let data = data else { return }           
            print(data)
        }
    }
}
```
**注意**：需要在Info.plist中申请权限`NSMotionUsageDescription`