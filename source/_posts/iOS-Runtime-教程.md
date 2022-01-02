---
title: iOS Runtime 教程
catalog: true
date: 2019-03-18 14:02:23
subtitle:
header-img: timg.jpg
tags: iOS开发
---
关于 Runtime ，网上已经有很多很好的文章，这里只是我在网上看了些，然后自己总(co)结(py)一下，目的只是加深自己的印象。

## 一、什么是 Runtime
说 OC 是一门动态语言，其实就是因为在其背后默默工作着的就是 Runtime。

在 C 语言中，将代码转换为可执行程序，一般要经历三个步骤，即编译、链接、运行。在链接的时候，对象的类型、方法的实现就已经确定好了。

而在 Objective-C 中，却将一些在编译和链接过程中的工作，放到了运行阶段。也就是说，就算是一个编译好的 .ipa 包，在程序没运行的时候，也不知道调用一个方法会发生什么。这也为后来大行其道的「热修复」提供了可能。因此我们称 Objective-C 为一门动态语言。
这样的设计使 Objective-C 变得灵活，甚至可以让我们在程序运行的时候，去动态修改一个方法的实现。而实现这一切的基础就是 Runtime 。

## 二、Class 和 Object

先来看下 `Objective-C` 中对象的定义：
```
typedef struct objc_object *id;

struct objc_object {
    Class isa;
};
```
在这里 `id` 被定义为一个指向 `objc_object` 的指针。说明 `objc_object` 就是我们平时常用的对象的定义，它只包含一个 `isa` 指针。`isa` 指针指向什么呢？按住 `command` 键看看 `Class`。

进入 `objc.h` 中，可以看到 `Class` 被定义为指向 `objc_class` 的指针，定义如下：
```
typedef struct objc_class *Class;
```
其中 `objc_class` 是一个结构体，在 `runtime.h` 中的定义如下：
```
struct objc_class {
    Class isa;                                // 实现方法调用的关键
    Class super_class;                        // 父类
    const char * name;                        // 类名
    long version;                             // 类的版本信息，默认为0
    long info;                                // 类信息，供运行期使用的一些位标识
    long instance_size;                       // 该类的实例变量大小
    struct objc_ivar_list * ivars;            // 该类的成员变量链表
    struct objc_method_list ** methodLists;   // 方法定义的链表
    struct objc_cache * cache;                // 方法缓存
    struct objc_protocol_list * protocols;    // 协议链表
};
```

可以看到，一个类保存了自身所有的成员变量（ `ivars` ）、所有的方法（ `methodLists` ）、所有实现的协议（ `objc_protocol_list` ）。


根据上面的 `objc_object` 和 `objc_class` 的定义，可以看出，一个对象**唯一**保存的信息就是它的 `Class` 的地址。当我们调用一个对象的方法时，会先通过 `isa` 去找到对应的 `objc_class`，然后再在 `objc_class` 的 `methodLists` 中找到调用的方法，然后执行。

再说说 `objc_class` 中的 `cache`，因为调用方法的过程是个查找 `methodLists` 的过程，如果每次调用都去查找，效率会非常低。所以对于调用过的方法，会以 map 的方式保存在 `cache` 中，下次再调用就会快很多。

现在划重点，在 Objective-C 中，**类 也被设计为一个 对象**。

其实观察 `objc_class` 和 `objc_object` 的定义，会发现两者其实本质相同，都包含 `isa` 指针，只是 `objc_class` 多了一些额外的字段。所以，**类 也是一个 对象**，只是类保存了一些额外信息。

既然说类也是对象，那么类的类型是什么呢？这里就引出了另外一个概念 —— Meta Class（元类）。

在 Objective-C 中，每一个类都有对应的元类，也就是类的 `isa` 指向的类。而在元类的 `methodLists` 中，保存了类的方法链表，即所谓的「类方法」，所以「类方法」其实就是调用了元类的方法。这一段有些拗口。

实际上元类也有一个 `isa` 指针，元类也是一个对象。为了不让这种结构无限延伸下去，所有元类的 `isa` 指向基类（比如 NSObject ）的元类。而基类的元类的 `isa` 指向自己。这样就形成了一个闭环。

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gdxvx589j30nj0i2t94.jpg)

## 三、Method

下面介绍一下 Objective-C 中的方法调用。

先来看一下 Method 在头文件中的定义：
```
typedef struct objc_method *Method;

struct objc_method {
    SEL method_name;
    char * method_types;
    IMP method_imp;
};
```
Method 被定义为一个 `objc_method` 指针，在这个结构体中，包含一个 `SEL` 和一个 `IMP`，同样来看一下它们的定义：
```
// SEL
typedef struct objc_selector *SEL;

// IMP
typedef id (*IMP)(id, SEL, ...);
```
先说一下 `SEL`，**SEL 实际上就是一个保存方法名的字符串**。

由于一个 Method 只保存了方法的方法名，并最终要根据方法名来查找方法的实现，所以，**OC 中不支持方法重载**，因为方法重载时，两个方法拥有相同的方法名，不同的形参列表，而 `SEL` 只保存了方法名，所以方法重载会被认为是相同的方法。

> Swift 支持方法重载

再来说 `IMP`。可以看到它是一个「函数指针」。简单来说，「函数指针」就是用来找到函数地址，然后执行函数。

这里要注意， `IMP` 指向的函数的前两个参数是默认参数， `id` 和 `SEL` 。这里的 `SEL` 好理解，就是函数名。而 `id`，对于实例方法来说， `self` 保存了当前对象的地址；对于类方法来说，`self` 保存了当前对应类对象的地址。后面的省略号即是参数列表。

到这里， Method 的结构就很明了了。 Method 建立了 `SEL` 和 `IMP` 的关联，当对一个对象发送消息时，会通过给出的 `SEL` 去找到  `IMP`，然后执行。

在 Objective-C 中，所有的方法调用，都会转化成向对象发送消息。发送消息主要是使用 `objc_msgSend` 函数。
```
id objc_msgSend(id self, SEL op, ...);
```
可以看到参数列表和 `IMP` 指向的函数参数列表是相对应的。 Runtime 会将方法调用做下面的转换，所以一般也称 Objective-C 中的调用方法为「发送消息」。

不过实际调用有点变态……
```
// 发送消息给无参数，无返回值的方法
((void (*)(id, SEL)) objc_msgSend)(self, NSSelectorFromString(@"functionName"));

// 发送消息给有参数，有返回值的方法
((NSString *(*)(id, SEL, NSString *)) objc_msgSend)(self, NSSelectorFromString(@"functionName"), @"parameter");
```
`objc_msgSend` 会默认传入 `id` 和 `SEL`，分别对应两个隐含参数， `self` 和 `_cmd`，其中 `_cmd` 指向方法本身。

**小结：当向一个对象发送消息时，会去这个类的 `methodLists` 中查找相应的 `SEL` ，如果查不到，则通过 `super_class` 指针找到父类，再去父类的 `methodLists` 中查找，层层递进。最后仍然找不到，才走抛异常流程。**

## 四、消息转发
刚才提到如果找不到对应的消息，会抛出异常，这里让我想到一道经典面试题：**从一个对象收到一个它无法响应的方法到崩溃之间发生了什么？**

**答：当一个方法找不到的时候，会走拦截调用和消息转发流程。**

![消息转发流程](https://tva1.sinaimg.cn/large/006y8mN6gy1g8gdy768mrj30go091dgt.jpg)

图中已经很明确的展示了消息转发的流程，下面分析一下：

**第一步：**

如果调用了一个对象的方法，在执行方法之前，首先会调用该类的 `+resolveInstanceMethod:` 方法进行判断，如果返回 `YES`， 则表示能接受消息，`NO` 表示不能接受消息并进入第二步。

例如，调用一个对象的 `testFunction` 方法，但实际上没有这个方法；我们在对应的类中添加 `+resolveInstanceMethod:` 方法，如下：

```
+ (BOOL)resolveInstanceMethod:(SEL)sel{
    //判断是否为外部调用的方法
    if ([NSStringFromSelector(sel) isEqualToString:@"testFunction"]) {

        [RuntimeTool addMethodWithClass:[self class] withMethodSel:sel withImpMethodSel:@selector(addDynamicMethod)];
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}
```

其中`RuntimeTool`是使用运行时动态添加方法的工具类，这里只要知道在运行时，用`addDynamicMethod`重写了`testFunction`，简单的说，就是调用 `testFunction` 时，实际调用的是 `addDynamicMethod`。这样即使没有实现也不会抛异常，因为消息已经转发出去了。

>注意，如果转发出去的消息仍没有实现，还是会抛异常。

同理，如果调用的是类方法，则在方法执行前会调用该类的 `+resolveClassMethod:`。

**第二步：**

如果第一步返回 `NO`，并且实现了 `-forwardingTargetForSelector:`，Runtime 这时会调用这个方法，在这个方法中，可以把这个消息转发给其他对象。

例如，把消息转发给 `ForwardMessage`，只要在这个类中实现了 `testFunction` 方法，那么也会执行，不过执行消息的对象改变了。

```
- (id)forwardingTargetForSelector:(SEL)aSelector{
  if ([NSStringFromSelector(aSelector) isEqualToString:@"testFunction"]) {
        return [[ForwardMessage alloc] init];
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

**第三步：**

如果第二部返回 `nil`，那么首先它会发送`-methodSignatureForSelector:`消息，获得函数的参数和返回值类型。如果`-methodSignatureForSelector:`返回`nil`，Runtime则会发出`-doesNotRecognizeSelector:`消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime就会创建一个`NSInvocation`对象并发送`-forwardInvocation:`消息给目标对象。

## 五、Category
我们来看一下 Category 在头文件中的定义：
```
typedef struct objc_category *Category;

struct objc_category {
    char * category_name;
    char * class_name;
    struct objc_method_list * instance_methods;
    struct objc_method_list * class_methods;
    struct objc_protocol_list * protocols;
}   
```
Category 是一个指向 `objc_category` 结构体的指针，这个结构体中包含对象方法列表、类方法列表、协议列表。所以， Category 支持添加对象方法、类方法、协议，但没有成员变量列表，所以**不能保存成员变量**。

> 注意：在 Category 中可以添加属性，会生成 getter、setter 的声明，但不会生成对应的成员变量、 getter 、setter 的实现。因此，调用 Category 中调用属性时会挂掉。

但是，可以通过「关联对象」的方式来添加可用的属性。
```
static NSString *name;

@implementation UIViewController (Name)

- (void)setName:(NSString *)n {
    objc_setAssociatedObject(self, &name, n, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

- (NSString *)name {
    return objc_getAssociatedObject(self, &name);
}

@end
```

## 六、Method Swizzling 方法交换
一般来说，交换方法主要想修改系统的方法实现。

比如说有一个项目，已经开发了2年，忽然项目负责人添加一个功能，点击每个 Button 时，都需要记录点击的是哪个——埋点，总不可能把项目里所有的 Button 都替换吧，这时方法交换就起到了很好的作用。
```
+ (void) {
    // 获得两个需要交换的方法
    Method originalMethod = class_getInstanceMethod(class, @selector(originalSelector));
    Method swizzledMethod = class_getInstanceMethod(class, @selector(swizzledSelector));

    // 尝试添加需要交换的方法，若添加失败，则说明已经有该方法，直接接交换
    if (!class_addMethod((class), @selector(originalSelector), 
                         method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod))) {             
        method_exchangeImplementations(originalMethod, swizzledMethod);         
    } else {
        // 如果添加成功，说明没有该方法，则把刚添加的方法替换为我们需要的方法
        class_replaceMethod((class), @selector(swizzledSelector),                                 
                            method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));            
    }
}

- (void)swizzledMethod {
    NSLog("调用自定义的方法");
    // 这里特别注意，必须调用交换的方法，因为上面已经把方法交换了，所以不会死循环，如果调用 originalMethod，反而会死循环
    [self swizzledSelector]; 
}
```
> `load` 方法原则上只会调用一次，但也有可能会被手动调用，所以方法里最好加上 `dispatch_once`。

## 七、获取属性

```
- (void)ropertyList {
    unsigned int count;
    objc_property_t *propertyList = class_copyPropertyList([self class], &count);
    for (unsigned int i = 0; i < count; i++) {
        const char *propertyName = property_getName(propertyList[i]);
        NSLog(@"property---->%@", [NSString stringWithUTF8String:propertyName]);
    }
    free(propertyList);
}
```
获取到属性后，还可以做到字典转模型，实际上，第三方字典转模型框架的核心就是上面一段代码。

参考：https://www.jianshu.com/p/361c9136cf3a
