---
title: TensorFlow 2.x - MNIST 图像分类(1)
catalog: true
date: 2020-05-13 10:41:11
subtitle: TensorFlow入门
header-img: tensorflow.jpg
tags: 机器学习
---

## 一、手写数字识别

手写数字识别是一个入门级的简单项目，相当于 HelloWorld，简单十来行代码就能搞定，但是其中的原理并不简单。代码不是重点，思想才是重点。

先来看看这个简单项目。

使用 TensorFlow 来完成，那么得先安装 TensorFlow，除此之外，还需要安装 numpy 用于支持维度数组与矩阵运算，以及 matplotlib 绘图工具。
```
$ pip3 install tensorflow
$ pip3 install numpy
$ pip3 install matplotlib
```

然后，可以在 [googleapis](https://storage.googleapis.com/tensorflow/tf-keras-datasets/mnist.npz) 下载所需要用到的数据集 MNIST。MNIST 是一个入门级的计算机视觉数据集，它包含各种手写数字图片。数据集里的每张图片大小为 28 * 28 像素。

![MNIST](https://upload-images.jianshu.io/upload_images/2708793-0db1532cb8d9257e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 二、加载数据

下载之后，我们先看看数据

```
import tensorflow as tf
datapath  = '/data/mnist/mnist.npz'
(train_images, train_labels), (test_images, test_labels) = tf.keras.datasets.mnist.load_data(datapath)

plt.imshow(train_images[0])
plt.show()
```
可以看到第一张图片是 5。

![](https://upload-images.jianshu.io/upload_images/2708793-88f1a38ee888ac98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是怎么把图片输入到程序中呢？

mnist 数据集中，每张图片大小为 28 * 28 像素，因此可以用 28 * 28 矩阵数组来表示一张图片。

![](https://upload-images.jianshu.io/upload_images/2708793-4083b552363683d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中纯白色部分用在矩阵中用 0 来表示，纯黑色部分用 1 表示，介于白色和黑色之间的部分用 0 ~ 1 之间的小数表示，这样就把一张图片抽象成一个矩阵。

这就完全是一个数学问题了。现在就是给 28 * 28 个 0 ~ 1 之间的数，然后对这些数做一些操作，使之输出一个数。

但这还不够，程序中读取到的是 0 ~ 255 之间的图片颜色，我们需要对应到 0 ~ 1，方便程序处理。

```
train_images = tf.keras.utils.normalize(train_images, axis=1)
test_images = tf.keras.utils.normalize(test_images, axis=1)
```

## 三、搭建网络

开始搭建我们神经网络：
```
model = tf.keras.models.Sequential()
# add input layer
model.add(tf.keras.layers.Flatten(input_shape=(28, 28)))
model.add(tf.keras.layers.Dense(128, activation=tf.nn.relu))
model.add(tf.keras.layers.Dense(128, activation=tf.nn.relu))
# add output layer
model.add(tf.keras.layers.Dense(10, activation=tf.nn.softmax))
```

这里我们使用 Sequential 创建顺序执行执行神经网络。

其中输入层 `tf.keras.layers.Flatten(input_shape=(28, 28))` 的主要的功能就是将 (28, 28) 像素的图像，即对应的 2 维的数组，再转成 28 * 28 = 784 的一维的数组。

第二、三层是 Dense，这个可以理解成全连接层，这个层存在 128 的神经元。激活函数使用简单的 relu，relu 相比其他激活函数计算简单。

最后一层是是输出层，激活函数使用 softmax。由于最后分类结果是 10 种，因此最后一层的神经元个数为 10 个，分别代码 0 ~ 9 十个数字，这十个神经元的输出则分别代表该图像属于某个类别的概率。

> softmax用于多分类过程中，它将多个神经元的输出，映射到（0,1）区间内，可以看成概率来理解，从而来进行多分类。

![softmax](https://upload-images.jianshu.io/upload_images/2708793-bcc162b85c680e86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、编译模型

接下来编译我们模型，可以指定 loss 函数类型，训练模型就是找到最合适方程，可以在 optimizer 指定找到最合适函数的方式。

```
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
```

在模型编译过程中有 3 个重要的参数：optimizer 优化器；loss 损失函数；metrics 计量标准是准确率，也就是正确归类的图像的概率。

因此我们通过 loss 损失函数、optimizer 优化器 以及 accuracy 准确率 这三个参数对模型调整。

**1. 损失函数**

神经网络最开始的输出是无序的，因此每次训练时，都略微调整神经元的权重值 w，这样，神经网络的输出也将发生变化。当输出值与真实值越接近，我们就说损失值减小了；当输出值与真实值越远，就说损失值增大了，所以自然而然的过度到了损失函数。

这样我们需要一个函数，用来衡量任意一个神经元的权重值 w，到底是好还是坏，这个函数叫做*损失函数*。

通过损失函数，我们可以从 w 的可行域中找到 w 取什么值，会是最不坏的情况，而这个过程会是一个优化的过程。

**2. 优化器**

其实整个训练过程就是一个优化过程。当我们有了损失函数之后，我们需要找到损失值最小的点。有点像高中数学问题，已知一个函数，求这个函数的最小值。

优化的过程就像是盲人下山的问题，盲人想通过最快的速度到达山底，就相当于我们找函数最小值，到达山底代表我们找到最小值，盲人踩着小碎步往下走，可是他该往哪个方向走最快呢？

是的，应该找最陡的地方，即哪个地方向下最陡，就往那边走，如果这是一个凸山（类比凸函数），他会找到一个最低点，当他每次都往最抖的方向走，我们优化的过程也是一样，寻找斜率最大值，不断的逼近函数的最小值。

但这存在着一个问题：**局部最优**和**全局最优**。

![](https://upload-images.jianshu.io/upload_images/2708793-656bc8f0b1233bb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果盲人走到了两座山之间的峡谷中呢？而这个峡谷并不是山底啊，这该怎么办？

常见的做法是给他一定的震荡，即并不是每次都朝着最陡峭的地方前进，而是大部分时间是朝着最陡峭的地方前进，小部分时间朝着并不陡峭的地方前进，这样就有可能避免走到峡谷中。

全局最优固然是最好，但是很多时候，你手中的都是一个局部最优解，这也是无可避免的。不过你可以不必担心，因为虽然不是全局最优，但是神经网络也能让你的局部最优足够优秀，而且想获取全局最优可能需要耗费大量的资源，而准确率可能只提高了一点点，所以应从多方面来考虑。

## 四、训练模型

整个训练过程就是：**喂数据 -> 误差反向传播 -> 调整参数**，再重复上述流程。简单点，用一个词来说就是**不断试错**。

比如我们输入数字 7 的图形，通过之前的操作，神经元实际收到的是 784 个像素值，经过 3 层神经网络的传播计算，理想情况下，输出层的第 7 个神经元的输出值为 1，而其他输出 0。

但最开始肯定不是这样的，最开始输出层肯定是无序的，关键思想在于整个神经网络参数调整有一个方向，即使损失最小。

我们通过不断的喂数据给神经网络，神经网络每次都略微调整神经元的权重值 w，得到输出结果，同时也告诉神经网络正确的结果，这两者之间的损失值，通过优化函数来减小，最终使神经网络能正确预测。

```
model.fit(train_images, train_labels, epochs=3)
```
描述复杂，代码只有一行……

其中 `x_train` 是训练集，需要喂给神经网络，`y_train` 是对应的标签，告诉神经网络正确的结果，`epochs` 是需要将所有训练样本训练几次。


## 五、评估模型

```
val_loss, val_acc = model.evaluate(test_images, test_labels)
print(val_loss)  # 0.0878981873
print(val_acc)  # 0.9722999930
```

当训练完成后，我们使用测试数据对模型进行评估，注意，训练模型时使用的是训练集，现在测试时使用测试数据来验证我们训练的模型是否准确。

可以看到，仅仅半分钟的训练准确率就达到 97.2%，这个不算高，稍微好一点的模型，准确率都在 99% 以上。

## 六、保存模型

把训练好的模型保存到磁盘，否则每次都要重新训练。
```
model.save('epic_num_reader.model')
save_model = tf.keras.models.load_model('epic_num_reader.model')
```

## 七、预测

```
i = 100  # 随便选一张图片
plt.imshow(test_images[i], cmap=plt.cm.binary)
plt.show()  # 显示这张图片
predictions = save_model.predict(test_images)  # 使用模型进行预测
print(np.argmax(predictions[i]))  # 打印预测结果
```

预测的结果是一个长度为 10 的数组来表示，这种编码称之为One hot（独热编码）。

独热编码使用 N 位代表 N 种状态，任意时候只有其中一位有效。

在手写数字中，用长度为 10 的数组来代表 0 ~ 9 这十个数字，例如 [1,0,0,0,0,0,0,0,0,0] 数组只在第零位有 1 代表数字 `0`，[0,1,0,0,0,0,0,0,0,0] 只在第一位有 1 代表数组 `1`，以此类推。

最后，你可以在[我的 GitHub](https://github.com/hezongjiang/TensorflowDemos/blob/master/classify_images_number.py) 上看到完整的代码。

下一节{% post_link TensorFlow-2-x-MNIST-图像分类-2 《TensorFlow 2.x MNIST-图像分类(2)》 %}
我们将使用卷积神经网络 CNN 来改造神经网络，将准确率提升到 99% 以上。