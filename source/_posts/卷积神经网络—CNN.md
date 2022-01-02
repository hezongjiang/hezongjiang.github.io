---
title: 卷积神经网络—CNN
catalog: true
date: 2018-03-15 20:12:33
subtitle:
header-img: cnn.jpeg
tags:
- 机器学习
---
## 一、为什么需要CNN
在全连接神经网络中，每相邻两层之间的每个神经元之间都是有边相连的。当输入层的特征维度变得很高时，这时全连接网络需要训练的参数就会增大很多，计算速度就会变得很慢，例如一张黑白的 28×28 的手写数字图片，输入层的神经元就有784个，如下图所示：

![全连接](https://upload-images.jianshu.io/upload_images/2708793-9bf202a891b30d47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

若在中间只使用一层15个神经元的隐藏层，那么参数 w 就有 784 × 15 = 11760 多个；若输入的是 28×28 带有颜色的RGB格式的手写数字图片，那么参数 w 的个数还需要再乘以 3。这很容易看出使用全连接神经网络处理图像中的需要训练参数过多的问题。

而在卷积神经网络（Convolutional Neural Network,CNN）中，卷积层的神经元只与前一层的部分神经元节点相连，即它的神经元间的连接是非全连接的，且同一层中某些神经元之间的连接的权重 w 和偏移 b 是共享的（Shared Weights），这样大量地减少了需要训练参数的数量。

## 二、CNN的卷积层
在卷积层中有几个重要的概念：
**1、Local receptive fields（感受视野）**
假设输入的是一个 28×28 的的二维神经元，我们定义5×5 的 一个 Local receptive fields（感受视野），即 隐藏层的神经元与输入层的 5×5 个神经元相连，这个 5×5 的区域就称之为 Local Receptive Fields，如下图所示：
![Local receptive fields](https://upload-images.jianshu.io/upload_images/2708793-4049d62e332d5343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**2、Shared weights（共享权值）**
我们知道，隐含层的每一个神经元都连接 5x5 个图像区域，也就是说每一个神经元存在 5x5=25 个连接权值参数。那如果对下一层的每个神经元来说，这 25 个参数是相同的呢？也就是说下一层的每个神经元用的是同一个卷积核去卷积图像。这样我们就只有多少个参数？？只有 25 个参数啊！不管你隐层的神经元个数有多少，两层间的连接我只有 25 个参数啊！这就是卷积神经网络的主打卖点。

![](https://upload-images.jianshu.io/upload_images/2708793-e10ead4323c94e00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**但是**，你就会想，这样提取特征也忒不靠谱吧，这样你只提取了一种特征啊？我们需要提取多种特征对不？假如一种滤波器，也就是一种卷积核就是提出图像的一种特征，那么我们需要提取不同的特征，怎么办，加多几种滤波器不就行了吗？对了。所以假设我们加到 100 种滤波器，每种滤波器的参数不一样，表示它提出输入图像的不同特征。这样每种滤波器去卷积图像就得到对图像的不同特征的放映，我们称之为Feature Map。所以 100 种卷积核就有 100 个 Feature Map。这 100 个 Feature Map 就组成了一层神经元。到这个时候明了了吧。我们这一层有多少个参数了？100 种卷积核 x 每种卷积核共享 25 个参数= 100 x 25 = 2500，也就是2500个参数。
![](https://upload-images.jianshu.io/upload_images/2708793-38ab7530eb04044b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
需要注意的一点是，上面的讨论都没有考虑每个神经元的偏置部分。所以权值个数需要加1 。这个也是同一种滤波器共享的。
因此在CNN的卷积层，我们需要训练的参数大大地减少到了 (5×5+1)×100=2600个。
![图片卷积](https://upload-images.jianshu.io/upload_images/2708793-e03633a7da7da8cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、“卷积”操作
当给一张新的图时，CNN并不能准确地知道这些 Features 到底要匹配原图的哪些部分，所以它会在原图中每一个可能的位置进行尝试。这样在原始整幅图上每一个位置进行匹配计算。这个我们用来匹配的过程就被称为卷积操作，这也就是卷积神经网络名字的由来。

这个卷积操作背后的数学知识其实非常的简单。要计算一个滤波器和其在原图上对应的某一小块的结果，只需要简单地将两个小块内对应位置的像素值进行乘法运算，然后将整个小块内乘法运算的结果累加起来，最后再除以小块内像素点总个数即可。具体过程如下：
![卷积操作](https://upload-images.jianshu.io/upload_images/2708793-fcdb65a287fee659.gif?imageMogr2/auto-orient/strip)

最后整张图算完，大概就像下面这个样子：
![卷积完成](https://upload-images.jianshu.io/upload_images/2708793-be8b2a70dbf7e2b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后换用其他滤波器进行同样的操作，最后得到的结果就是这样了：
![多卷积和](https://upload-images.jianshu.io/upload_images/2708793-2ceb023b5ecd755c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、池化(Pooling)
当输入经过卷积层时，若感受视野比较小，步长stride比较小，得到的feature map （特征图）还是比较大，可以通过池化层来对每一个 feature map 进行降维操作，输出的深度还是不变的，依然为 feature map 的个数。


池化可以将一幅大的图像缩小，同时又保留其中的重要信息。池化背后的数学顶多也就是小学二年级水平。它就是将输入图像进行缩小，减少像素信息，只保留重要信息。通常情况下，池化都是 2×2 大小，比如对于 max-pooling 来说，就是取输入图像中 2×2 大小的块中的最大值，作为结果的像素值，相当于将原始图像缩小了 4 倍。(注：同理，对于average-pooling来说，就是取 2×2 大小块的平均值作为结果的像素值。)

![池化](https://upload-images.jianshu.io/upload_images/2708793-c6006174376411ef.gif?imageMogr2/auto-orient/strip)


所以，池化的实质就是把图片变模糊了，相当于给图片打了马赛克，如下图：
![马赛克](https://upload-images.jianshu.io/upload_images/2708793-47fdbcd5f6b43d0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过加入池化层，可以很大程度上减少计算量，降低机器负载。

到现在为止，已经讲了从输入到池化的过程，图片也从原来的 28 × 28 变成了 12 × 12，如下图

![输入->卷积->池化](https://upload-images.jianshu.io/upload_images/2708793-a7a268b6c29571bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 五、激活函数Relu (Rectified Linear Units)
这里，我们用Relu作为激活函数：
![Relu激活函数](https://upload-images.jianshu.io/upload_images/2708793-e0374cbc4b1421a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 六、全连接层(Fully connected layers)
到这里，卷积神经网络基本介绍完毕，总结（敲黑板），卷积神经网络的目的就是降低网络复杂度，减少权值W和偏置值b的个数，它不是一个完整的网络，所以，在最后还须加入普通的神经网络（全连接层）。
![卷积神经网络总体图](https://upload-images.jianshu.io/upload_images/2708793-9a81462bcdc5e270.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考：
http://blog.csdn.net/v_JULY_v/article/details/51812459
https://www.jianshu.com/p/fe428f0b32c1#comment-21577884
http://blog.csdn.net/u012641018/article/details/52238169
http://blog.csdn.net/cxmscb/article/details/71023576
http://blog.csdn.net/u010555688/article/details/24848367