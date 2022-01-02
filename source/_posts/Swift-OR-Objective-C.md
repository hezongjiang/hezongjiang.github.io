---
title: Objective-C? OR Swift?
catalog: true
date: 2019-01-10 22:02:44
subtitle: 从OC到Swift
header-img:
tags:
---
# 1、Swift简介
WWDC 2014 上苹果再次惊世骇俗的推出了新的编程语言 **Swift**——雨燕, 这个消息会前没有半点风声的走漏。消息发布当时，会场一片惊呼，相信全球看直播的码农们当时也感觉脑袋被敲了一记闷棍吧。于是熬夜学习了Swift大法， 越看越想高呼“Swift大法好！”
 
程序员，最讲究的就是实事求是和客观，这篇文章就开始对比两种语言。
 
首先要强调的是，Swift绝对不是解释性语言，更不是脚本语言，它和Objective-C，C++一样，编译器最终会把它翻译成C语言，也就是说编译器最终面对的其实都是C语言代码（这是千真万确，不容置疑的！所以不要看它长的想脚本语言，其实它是比Java，C#要高效的多的C语言！），但是Swift的强大之处在于它站在所有语言的肩膀上，吸取所有语言的精华。
 
这个系列我们先谈谈几个最基本的语法变化：
Swift终于放弃了Objective-C那幺蛾子般的`[obj method:x1 with:x2]`的语法，终于跟随了大流，变成了`obj.method()`的顺眼模式。虽然对于Objective-C的程序员来说，这些[ ]看上去特显酷， 而且我们看上去也挺顺眼的，但是你们知道就是这个中括弧吓跑了多少C++，java，C#的程序员嘛？所以说这个小小的变化，可以让苹果的开发更平易近人，对有其他开发语言基础的人来说更友好，能让更多的人步入iOS开发。

但苹果不会这么自甘平庸，我们知道Objective-C里方法的调用有种语法是其他主流语言没有的，那就是标签。我们在使用Java，C++，C，C#等语言时，如果使用`rect.set(10, 20, 100, 500)`，虽然在写set方法的时候，IDE有提示四个形参的含义，但写完后，这句代码中10， 20， 100， 500是社么意思? 所以代码的可读性就变的很差， 而Objective-C很完美的解决了这个问题 ：

OC代码：
```
［rect setX:10 y:20 width:100 height:500];  
```
看看！多棒！Swift当然不会丢弃这么好的传统， 在Swift中是这个样子的
Swift代码：
```
rect.set(x:10, y:20, width:100, height:500)  
```

 
其实Swift中对类的定义和Java，C#几乎一样，再也不分头文件和.m文件了。
一个类的定义语法如下：
Swift代码 
```
class Weapon  
{  
    var name: String  
    var power: Int  
    init(name: String, power: Int)  
    {  
        self.name = name  
        self.power = power  
    }  
    func shoot()  {
        print("威力 + \(power)")
    }
}  
```
**注意**：Swift中的语句不需要分号结束！
其次，终于有构造函数和析构函数了！
Objective-C也有？
no no no!!!
Objective-C中没有构造函数，真正的构造函数是系统自动调用的，而不是强迫程序员去调用。以前要强迫程序员[ [Obj alloc ] init ]，现在终于终于终于系统自动调用了！
C代码 
```
Weapon weapon = Weapon(name:"人间大炮", power:100000000)  
```
 
我没有写错，对的！现在和Java，C#一样！虽然weapon是一个指针，但不要写那颗星号！！因为这颗星号吓死过好多人！“ 啥？指针？！！啊……”
 
C、 C++的程序员注意了，这个weapon对象不是分配在栈内存上的， 仍然是alloc出来的， 在堆上呢。

最期待的语法终于加入了！

对于override在Java,C++,Objective-C中都有问题，举个栗子：
OC代码
``` 
@interface Weapon  
- (void)shoot;  
@end  
@interface Gun : Weapon  
- (void)Shoot;  
@end  
```
 
在大项目中经常会遇到这个问题，程序员的本意是希望覆盖父类的shoot ,结果手潮。。。。写成了Shoot ， 这既没有语法错误，也没有逻辑错误，结果在
OC代码 
```
Weapon *currentWeapon = [Gun new];  
```
 `
[currentWeapon shoot] 
`调用的却是父类的shoot方法（因为子类根本没有覆盖啦，子类大小写不小心写错了）， 这种小错误如果出现在超大型项目种还真是很难找呢！！现在，Swift终于解决这个问题啦！ 子类覆盖父类方法的时候，一定要在方法前写上`override`
Swift代码：
```
override func shoot{  
}  
```
 
这样，编译器一看方法前写了`override`， 它就会在父类中去查找是否有`shoot`方法， 如果你写错成`override func Shoot`, 那编译器就立刻能发现报错啦！
 
# 2、对比
 
有人可能不同意关于Swift会取代Objective-C的论点，在这里我想强调两点：
 
1. Swift其实就是Objective-C的文本变种，对于这门全新的语言，苹果做的工作其实远没有我们想像的艰巨。LLVM编译器做工作只是 先把swift翻译成Objctive-C代码，然后再把Objective-C代码翻译成C语言代码，然后再把C语言代码翻译成汇编，最终翻译成机器 码。至于为什么编译器厂商这么绕，不直接把自己的语言翻译成汇编和机器码，那是由于现有的语言编译器（Objective-C, C )已经非常成熟，而高级语言间的文本转换开发成本和维护成本都极其小。Swift为什么要翻译成Objective-C，是由于Swift仍然需要 Objective-C中辛苦构建的ARC，GCD 等环境。
 
2. 既然Swift代码最终会被LLVM翻译成Objective-C， 那Swift语言还有什么意义？想想ARC刚出来的时候大家的反应吧，很多人和今天的反应一样，认为我是资深的objective-c马仔了，我深谙内存 管理之道，不停的写[obj release ]，[obj autoRelease] 很牛，只有那些初学者才会用ARC呢。结果就是不到一年，ARC统治了整个马仔界，因为我们马仔关注的应该是业务逻辑，而不应该把精力分散在语法等低级问 题上，语法消耗我们的时间越少，这门语言就越成功。
 
既然Swift其实就是Objective-C， 对入门者而言远比Objective-C好学，对资深开发者来说又能节约很多无谓的低级重复的机械代码（这些代码在LLVM翻译成Objective-C 时，编译器自动帮你写上）。我是想不出任何一点Swift不替换Objective-C的理由呢。
 
好吧，争论放置一边不表，我们从头来看Swift到底进化到什么程度。

#### 2.1 初始化
变量如果有初始化就不需要类型
Swift代码：
```
var n = 22  
```
对于编译器而言，既然你都初始化为22了，它当然明白n是int , 你都打回车了， 它当然知道这是语句的结束，所以LLVM毫无压力的把它翻译成
OC代码：
```
int n = 22;  
```
当然对于多个语句放一行，那编译器就没有办法了， 你还是要用分号来结束语句。如果没有初始化，你也可以手工指定变量类型
Swift代码：
```
var n = 22; var hero: Hero  
```
所以看上去是无类型变量，实质上还是强类型的（ 编译器给你做了 ）.
如果是常量的话， 用`let`
C代码
```
let PI = 3.1415926  
```
这里的PI 就是常量， 现在想想，以前的强类型高级语言真是傻到无语啊，let PI = 3.1415926 , PI  都这么明显是个double, 为啥还要程序员再写double ?!
 
#### 2.2  函数的定义与调用
* 如果有返回值， 返回类型用符号 “ -> ” 表示
Swift代码
```
func  add( p1: int, p2 : int ) -> int  {  
        return p1+p2  
}  
```

* 如果想定义一个求和函数，但是有多少个数相加，这个是不确定的，咋办？
在OC中，你可能想到用数组，但是在Swift中：
```
func sum(nums: Double...) -> Double {
        var result = 0.0
        for num in nums {
            result += num
        }
        return result
    }
```
调用如下：
```
sum(nums: 1, 2, 3, 4) // 结果是10
```
实际上，`sum`函数参数`nums`就是一个数组，只不过这个数组不需要你来定义，你只需要拿过来直接用就可以了，在Java中也用这也的语法，这只是个语法糖，表面上没有数组，但底层仍是用数组实现，大大简化了我们程序员的工作。

* 还有一点个人觉得非常重要，就是Swift中函数有默认参数了：
下面摘抄SDWebImage中的一小段。我们不需要仔细看它究竟在做什么，只需要知道，第一个方法仅仅是在调用第二个方法，而第二个方法也是仅仅在调用第三个方法，而第三个方法也仅仅是在调用第四个方法，依次类推，直到最后一个方法，它所调用的才是真正的实现方法。
```
-(void)sd_setImageWithURL:(NSURL *)url {
        [self sd_setImageWithURL:url placeholderImage:nil options:0 progress:nil completed:nil];
}
-(void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder {
        [self sd_setImageWithURL:url placeholderImage:placeholder options:0 progress:nil completed:nil];
}
-(void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options {
        [self sd_setImageWithURL:url placeholderImage:placeholder options:options progress:nil completed:nil];
}
-(void)sd_setImageWithURL:(NSURL *)url completed:(SDWebImageCompletionBlock)completedBlock {
        [self sd_setImageWithURL:url placeholderImage:nil options:0 progress:nil completed:completedBlock];
}
-(void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder completed:(SDWebImageCompletionBlock)completedBlock {
        [self sd_setImageWithURL:url placeholderImage:placeholder options:0 progress:nil completed:completedBlock];
}
-(void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options completed:(SDWebImageCompletionBlock)completedBlock {
        [self sd_setImageWithURL:url placeholderImage:placeholder options:options progress:nil completed:completedBlock];
}
```
为什么要这么复杂的定义这么多方法？
答：因为OC中的方法参数没有默认值。
如果是Swift应该怎么做？如下：
```
func sd_setImage(url: URL, placeholderImage: UIImage? = nil, options: Int = 0, progress: String = "0")
{
}
```
 只需要定义这一个方法，在调用时，系统会提供4个方法：
```
let url = URL(string: "")!
let image = UIImage(named: "")
sd_setImage(url: url)
sd_setImage(url: url, placeholderImage: image)
sd_setImage(url: url, placeholderImage: image, options: 2)
sd_setImage(url: url, placeholderImage: image, options: 2, progress: "100%")
```
这些都是正确的。
啥？有3个方法我都没写啊？
这就是Swift中函数的默认参数，定义方法时，在形参类型后加` = XXX`的意思就是，调用该方法时，这个参数可以不用传入，用方法中的默认值`XXX`，当然你也可以传入这个参数，覆盖方法中的默认值。

* 类的定义
不再分头文件和m文件了！如果不想让其它类访问，加上private，这点和Java，C#一模一样
Swift代码：
```
class Person  
{  
    var name:String  
    var age = 0  
    init(name:String, age:int)  
    {  
        self.name = name;  
        self.age = age;  
    }  
    func description()->String  
    {  
        return "Name:\( self.name ) ; Age: \( age )"; 
    }  
}  
```
注意终于有构造函数了！！init 是系统自动调用的， 不需要程序员手工调用。所以它写起来和普通函数也有区别，前边不能加func。 编译器为什么要这样做？因为如果init也允许前面加上func, 万一程序员不小心把init函数名写错了， 写成func Inot( ),编译器就完全不知道它是程序员想写的构造函数。现在构造函数前不加func , 如果你写成Inot( ) 。 编译器一看前面没有func知道你要写构造，可函数名又不是init, 编译器就知道你不小心写错了就可以立刻报错啦！！

*  可选类型
专门设计了一个叫”optional value（可选变量）”的变量
语法：
Swift代码：
```
var n : UInt ? = 5   // 或者
var n ? = 5  
```
这里的？表示n 是个可选变量， 也就是说 n 有可能不存在， 什么情况下n不存在呢？
如果你这样写：
C代码
```
var n : UInt ?  
```
此外，需要注意的是swift语法中，nil 并不是0 , 而是一个NilType类型的变量
所以上面提到的那个问题就可以很容易解决了
Swift代码：
```
func FindScoreByName( source:DataSource, name:String) -> Int?  //返回的是可选变量  
{  
    var score : UInt ?; //此时 score 的变量没有分配内存，也就是说score为nil  
    if( source.HasStudent( name: name))  {
        score = source[ name ]. score;  //这里score才分配内存；
    }  
    return score; //如果没有找到学生信息， score的内存一直没被分配  
}  
```
**注意：**
`Int`和`Int?`是两种不同的类型，例如：
```
var a: Int = 5
var b: Int? = 10
let c = 20.0
// 下一句会报错，因为Int和Int?是两种不同的类型，Swift是强类型语言（OC是弱类型语言），类型不同的数，不能直接相加减，只有类型相同才行，
//a = b
// 下一句也会报错，因为a是Int类型，而c编译器会自动推导其为Double类型，类型不同，不能相加减
//let d = a + c
```
也许你会问，“那咋整？！”
这就需要强制类型转换，例如
```
let d = Double(a) + c
```
这里只需要**注意：**是把需要转换的数放括号里，而**不是**像OC那样，把类型放括号里