---
title: iOS Core Location 之 定位 与 地理编码
catalog: true
date: 2017-08-29 23:15:24
subtitle:
header-img: ios.jpg
tags:
- iOS开发
---
## 一、定位功能简介
要实现地图、导航功能，往往需要先熟悉定位功能。在 iOS 中通过 **Core Location** 框架进行定位操作。**Core Location** 自身可以单独使用，和地图开发框架 **MapKit** 完全是独立的。但是往往地图开发要配合定位框架使用。在 **Core Location** 中主要包含了定位、地理编码（包括反编码）功能。

## 二、CLLocationManager 主要方法和属性
要实现定位功能，需要了解 **Core Loaction** 中 **CLLocationManager** 类，首先看一下这个类的一些主要方法和属性：

1. 类方法
```
/// 是否启用定位服务，通常如果用户没有启用定位服务可以提示用户打开定位服务
class func locationServicesEnabled() -> Bool

///  定位服务授权状态，返回枚举类型
class func authorizationStatus() -> CLAuthorizationStatus
```
2. 属性
```
/// 定位精度，枚举类型
var desiredAccuracy: CLLocationAccuracy 

/// 位置信息更新最小距离，只有移动大于这个距离才更新位置信息，默认为kCLDistanceFilterNone：不进行距离限制
var distanceFilter: CLLocationDistance

/// 位置更新可能会自动暂停，默认是会自动停止定位，建议设置为false
var pausesLocationUpdatesAutomatically: Bool
```
3. 对象方法
```
/// 开始定位追踪，开始定位后将按照用户设置的更新频率执行locationManager(_:didUpdateLocations:)方法
func startUpdatingLocation()

/// 停止定位追踪
func stopUpdatingLocation()

/// 开始导航方向追踪
func startUpdatingHeading()

/// 停止导航方向追踪
func stopUpdatingHeading()

/// 开始对某个区域进行定位追踪，如果用户进入或者走出某个区域会调用代理方法反馈相关信息
func startMonitoring(for region: CLRegion)

/// 停止对某个区域进行定位追踪
func stopMonitoring(for region: CLRegion)

/// 请求应用使用时的定位服务授权，注意使用此方法前在要在info.plist中配置NSLocationWhenInUseUsageDescription
func requestWhenInUseAuthorization()

/// 请求应用的定位服务授权，注意使用此方法前在要在info.plist中配置NSLocationAlwaysUsageDescription
func requestAlwaysAuthorization()
```

## 三、后台定位的配置
如果希望应用程序在后台或者锁屏情况下还能使用定位功能，则必须在`TARGETS`的`Capabilities`选项中打开`Background Modes`并勾选`Location updates`：
![勾选Location updates](http://upload-images.jianshu.io/upload_images/2708793-95ac61c0add8689c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、定位功能实现
```
import CoreLocation

typealias LocationResult = (_ location: CLLocation?, _ status: CLAuthorizationStatus, _ error: Error?) -> ()

class LocationManager: NSObject {

    /// 单例
    static let shared = LocationManager()
    
    fileprivate override init() { super.init() }
    
    /// 是否只定位一次
    fileprivate var atOnce: Bool = false
    
    /// 定位结果回调
    fileprivate var resultBlock: LocationResult?
    
    /// 定位授权状态
    fileprivate var authorizationStatus: CLAuthorizationStatus = .notDetermined
    
    /// 定位管理者
    fileprivate lazy var locationManager: CLLocationManager = {
       
        let locationManager = CLLocationManager()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.pausesLocationUpdatesAutomatically = false
        
        // 请求授权
        if #available(iOS 8.0, *) {
            
            guard let infoDic = Bundle.main.infoDictionary else { return locationManager }
            
            let whenInUse = infoDic["NSLocationWhenInUseUsageDescription"]
            let always = infoDic["NSLocationAlwaysUsageDescription"]
            
            if let backgroundModes = infoDic["UIBackgroundModes"] as? [String], backgroundModes.contains("location"), #available(iOS 9.0, *) {
                
                locationManager.allowsBackgroundLocationUpdates = true
            }
            
            if always != nil {
                locationManager.requestAlwaysAuthorization()
            } else if whenInUse != nil {
                locationManager.requestWhenInUseAuthorization()
            } else {
                print("错误提示：在 iOS8.0 以后，想要使用用户位置，要主动请求授权。应该在 info.plist 配置 NSLocationWhenInUseUsageDescription 或者 NSLocationAlwaysUsageDescription")
            }
        }
        return locationManager
    }()
    
    /// 获取当前位置
    ///
    /// - Parameters:
    ///   - atOnce: 是否只定位一次，若为 true，则定位一次后停止定位
    ///   - resultBlock: 位置信息
    func startUpdatingLocation(atOnce: Bool, resultBlock: @escaping LocationResult) -> () {
        
        self.atOnce = atOnce
        self.resultBlock = resultBlock
        
        if CLLocationManager.locationServicesEnabled() {
            locationManager.startMonitoringSignificantLocationChanges()
            locationManager.startUpdatingLocation()
        }else {
            self.resultBlock?(nil, CLAuthorizationStatus.denied, nil)
        }
    }
    
    /// 停止定位
    func stopUpdatingLocation() {
        locationManager.stopUpdatingLocation()
        locationManager.stopMonitoringSignificantLocationChanges()
    }
}

// MARK: - CLLocationManagerDelegate
extension LocationManager: CLLocationManagerDelegate {

    /// 定位发生错误
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        resultBlock?(nil, authorizationStatus, error)
    }

    /// 定位信息改变
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        
        guard locations.count > 0, let location = locations.last else {
            resultBlock?(nil, authorizationStatus, nil)
            return
        }
        resultBlock?(location, authorizationStatus, nil)
        if atOnce { manager.stopUpdatingLocation() }
    }

    /// 定位授权状态改变
    func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
        
        authorizationStatus = status
        locationManager(locationManager, didUpdateLocations: [])
    }
}
```

## 五、地理编码
除了提供位置跟踪功能之外，在定位服务中还包含 **CLGeocoder** 类，用于处理地理编码和逆地理编码（又叫反地理编码）功能。

* **地理编码**：根据给定的位置（通常是地名）确定地理坐标(经、纬度)。
* **反地理编码**：可以根据地理坐标（经、纬度）确定位置信息（街道、门牌等）。

**CLGeocoder** 最主要的两个方法就是：
```
/// 地理编码
func geocodeAddressString(_ addressString: String, completionHandler: @escaping CoreLocation.CLGeocodeCompletionHandler)

/// 反地理编码
func reverseGeocodeLocation(_ location: CLLocation, completionHandler: @escaping CoreLocation.CLGeocodeCompletionHandler)
```
具体使用方法如下：
1. 输入地名，获取 经纬度 和 当地天气
```
geocoder.geocodeAddressString("广州") { (placemarks, error) in
            
    guard let placemarkName = placemarks?.first.name else { return }
    
    // OpenWeatherMap——查询天气接口API
    let urlString = "https://api.openweathermap.org/data/2.5/weather?q=\(placemarkName)&appid=594a923fe158c2c45c54aca585f2707b"

    guard let url = URL(string: urlString) else { return }
            
    let sessionDataTask = URLSession.shared.dataTask(with: url) { (data, response, error) in
        guard let data = data, let json = try? JSONSerialization.jsonObject(with: data) else { return }
        print(json)
    }
    sessionDataTask.resume()
}
```
2. 输入经纬度，获取城市信息
```
let location = CLLocation(latitude: 23.125178, longitude: 113.280637)
        
geocoder.reverseGeocodeLocation(location) { (placemarks, error) in
            
    guard let placemark = placemarks?.first else { return }
    print(placemark.addressDictionary?["City"])
 }
```
**注意**：以上两个地理编码方法，需要在有网的时候才能获取结果，其实质就是去网络获取信息