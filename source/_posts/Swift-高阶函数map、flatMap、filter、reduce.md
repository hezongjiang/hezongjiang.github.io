---
title: Swift - 高阶函数map、flatMap、filter、reduce
catalog: true
date: 2017-12-29 21:36:17
subtitle:
header-img: swift.jpeg
tags:
- iOS开发
---
Swift 提供了如下几个高阶函数：`map`、`flatMap`、`filter`、`reduce`。使用高阶函数进行函数式编程不仅可以简化我们的代码，而且当数据比较大的时候，高阶函数会比传统实现更快。

## 一、map 函数
`map` 方法获取一个闭包表达式作为其唯一参数。然后它会遍历整个数组，并对数组中每一个元素执行闭包中的操作，并返回该元素所映射的值。
简单说就是，数组中每个元素通过某个方法进行转换，最后返回一个**新**的数组。

**示例1**：将数组中的数字，全部转换为字符串
```
let prices = [1, 2, 3, 4]
let priceStr = prices.map { (n) -> String in
    return "价格:\(n)"
}
print(priceStr) // ["价格:1", "价格:2", "价格:3", "价格:4"]
```
注：在闭包中，可以用`$0`表示第一个参数，`$1`表示第二个参数，以此类推；若返回值只有一句，则可以省略`return`，所以上面的代码可以简化为：
```
let prices = [1, 2, 3, 4]
let priceStr = prices.map { "价格\($0)" }
print(priceStr) // ["价格:1", "价格:2", "价格:3", "价格:4"]
```

**示例2**：对一个数组里面的数据进行平方操作
```
let values = [5, 6, 7]
let squares = values.map { $0 * $0 }
print(squares) // [25, 36, 49]
```

## 二 flatMap 函数
`flatMap` 方法同 `map` 方法比较类似，只不过它返回后的数组中不存在 nil（自动把 nil 给剔除掉），同时它会把 Optional 解包。
```
// 示例1：
let names = ["Jack", "Rose", nil, "AnyBody"]
let names2 = names.flatMap { $0 }
print(names2) // 输出时，会自动剔除nil，结果：["Jack", "Rose", "AnyBody"]

// 示例2:
let names = ["Jack", "Rose", "AnyBody"]
let names2 = names.flatMap { (name) -> String? in
    return name
}
print(names2) // 结果与示例1相同

// 示例3:
let names = ["Jack", "Rose", "AnyBody"]
let names2 = names.flatMap { (name) -> String in
    return name
}
print(names2) // 结果: ["J", "a", "c", "k", "R", "o", "s", "e", "A", "n", "y", "B", "o", "d", "y"]
```
**特别注意：**如果 `faltMap` 闭包的返回值**不是**可选型(示例3)，可能会有意想不到的结果。

同时，`flatMap` 还能把数组中存有数组的数组（二维数组、N维数组）一同打开变成一个新的数组。
```
let array = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
let arr1 = array.map { $0 }   // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
let arr2 = array.flatMap { $0 } // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

小结：`map` 和 `flatMap` 区别，`map` 函数值对元素进行变换操作。 但不会对数组的结构造成影响。 而 `flatMap` 会影响数组的结构。

## 三、filter 函数

`filter` 方法用于过滤元素，即筛选出数组元素中满足某种条件的元素。
```
// 筛选出金额大于 25 的元素。
let prices = [20, 30, 40]
let result = prices.filter { $0 > 25 }
print(result) // [30, 40]
```

## 四、reduce 函数
`reduce` 方法把数组元素组合计算为一个值，并且会接受一个初始值，这个初始值的类型可以和数组元素类型不同。

示例1：计算数组元素的乘积
```
let prices = [20, 30, 40]
let result = prices.reduce(1) { $0 * $1 }
print(result) // 24000
```
上面的方法的第二行还可以简写成：
```
let result = prices.reduce(1, *)
```

示例2：将数组转成字符串，每个元素用顿号`、`隔开。
```
let array = ["Apple", "Orange", "Grape"]
let str = array.reduce("") { $0 == "" ? $1 : $0 + "、" + $1 }
print(str) // Apple、Orange、Grape
```
也可以简写为：
```
let str = array.joined(separator: "、") // Apple、Orange、Grape
```
