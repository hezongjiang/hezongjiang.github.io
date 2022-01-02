---
title: TensorFlow 2.x - MNIST 图像分类(2)
catalog: true
date: 2020-05-14 10:41:21
subtitle: TensorFlow入门
header-img: tensorflow.jpg
tags: 机器学习
---

在上一节{% post_link TensorFlow-2-x-MNIST-图像分类-1 《TensorFlow 2.x MNIST-图像分类(1)》 %}中，我们搭建了一个全连接的神经网络用于手写数字识别。本节将在上一节的基础上，将使用卷积神经网络 (Convolutional Neural Network, CNN) 来搭建网络识别数字。

![CNN](https://upload-images.jianshu.io/upload_images/2708793-207924bb7c1b030e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 一、为什么要用 CNN

在全连接神经网络中，每相邻两层之间的每个神经元之间都是相连的。当输入层的特征维度变得很高时，这时全连接网络需要训练的参数就会增大很多，计算速度就会变得很慢。

例如一张黑白的 28×28 的手写数字图片，输入层的神经元就有 784 个，若在中间只使用一层 15 个神经元的隐藏层，那么神经元的权重参数 w 就有 784 × 15 = 11760 个；若输入的是 28×28 带有颜色的 RGB 颜色的手写数字图片，那么参数 w 的个数还需要再乘以 3，而这仅仅是 28×28 像素的图片。

![28×28 像素的数字](https://upload-images.jianshu.io/upload_images/2708793-99dec7a45d0ded12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果需要识别一张高清图片呢？很容易看出使用全连接神经网络处理图像中的需要训练参数过多的问题。这就需要使用 CNN 来减少需要训练的参数 w。如果你还不了解 CNN，建议先看一下 [《卷积神经网络—CNN》](https://www.jianshu.com/p/d813215b27fc)，没错，也是我的文章。

## 二、加载数据

加载数据与上一节基本相同，也是使用 MNIST 数据集。

```
import tensorflow as tf
from tensorflow.keras import datasets, layers, models

class DataSource(object):
    def __init__(self):
        # mnist数据集存储的位置
        data_path = '/path/mnist/mnist.npz'
        (train_images, train_labels), (test_images, test_labels) = datasets.mnist.load_data(path=data_path)

        # 6万张训练图片，1万张测试图片
        train_images = train_images.reshape((60000, 28, 28, 1))
        test_images = test_images.reshape((10000, 28, 28, 1))
        # 像素值映射到 0 - 1 之间
        train_images, test_images = train_images / 255.0, test_images / 255.0

        self.train_images, self.train_labels = train_images, train_labels
        self.test_images, self.test_labels = test_images, test_labels
```

## 三、搭建网络

模型定义的前半部分主要使用 Keras.layers 提供的 Conv2D (卷积) 与MaxPooling2D (池化) 函数。

CNN 的输入层的 input_shape 为 (image_height, image_width, color_channels) 的张量。mnist 数据集是黑白的，因此只有一个 color_channel，即 color_channel = 1；彩色图片有 3 个颜色通道 (R,G,B)，所以 color_channel = 3。

对于 mnist 数据集，输入的张量维度就是 (28, 28, 1)，通过参数 input_shape 传给网络的第一层。
```
class CNN(object):
    def __init__(self):
        model = models.Sequential()
        # 输入层，用32个3*3的卷积核进行卷积操作，28*28为待训练图片的大小
        model.add(layers.Conv2D(filters=32, kernel_size=(3, 3), activation='relu', input_shape=(28, 28, 1)))
        # 添加池化层
        model.add(layers.MaxPooling2D((2, 2)))
        # 第2层卷积，用64个3*3的卷积核进行卷积操作
        model.add(layers.Conv2D(filters=64, kernel_size=(3, 3), activation='relu'))
        # 添加池化层
        model.add(layers.MaxPooling2D((2, 2)))
        # 第3层卷积，用64个3*3的卷积核进行卷积操作
        model.add(layers.Conv2D(filters=64, kernel_size=(3, 3), activation='relu'))
        # 压扁
        model.add(layers.Flatten())
        # 全连接层
        model.add(layers.Dense(64, activation='relu'))
        model.add(layers.Dense(10, activation='softmax'))
        # 打印模型的结构
        model.summary()

        self.model = model

```
每一个 Conv2D 和 MaxPooling2D 层的输出都是一个三维的张量 (height, width, channels)。height 和 width 会逐渐地变小。输出的 channel 的个数，是由 filters 参数控制，随着 height 和 width 的变小，channel 可以变大。

整个卷积过程就是把图片的宽高减小，图片的“厚度”增加。

1. 原始图片大小为 (28, 28, 1)；
2. 经过输入层卷积后变为 (26, 26, 32)，池化后变为 (13, 13, 32)；
3. 经过第二层卷积后变为 (11, 11, 64)，池化后变为  (5, 5, 64)；
4. 经过第三层卷积后变为 (3, 3, 64)。

模型的后半部分，是定义全连接层。layers.Flatten 会将三维的张量转为一维的向量。展开前张量的维度是 (3, 3, 64) ，转为一维 (576) 的向量后，再使用 layers.Dense 层，构造了 2 层全连接层，逐步地将一维向量的位数从 576 变为 64，再变为 10。

后半部分相当于是构建了一个输入层为 576，隐藏层为 64，输出层为 10 的普通的神经网络。最后一层的激活函数是 softmax，10 位恰好可以表达 0-9 十个数字。

## 四、开始训练并保存训练结果

```
class Train:
    def __init__(self):
        self.cnn = CNN()
        self.data = DataSource()

    def train(self):
        check_path = './ckpt/cp-{epoch:04d}.ckpt'
        # period 每隔5epoch保存一次
        save_model_cb = tf.keras.callbacks.ModelCheckpoint(check_path, save_weights_only=True, verbose=1, period=5)

        self.cnn.model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
        self.cnn.model.fit(self.data.train_images, self.data.train_labels, epochs=5, callbacks=[save_model_cb])

        test_loss, test_acc = self.cnn.model.evaluate(self.data.test_images, self.data.test_labels)
        print("准确率: %.4f，共测试了%d张图片 " % (test_acc, len(self.data.test_labels)))


if __name__ == "__main__":
    app = Train()
    app.train()
```

训练 4~5 次，准确率就能达到 99% 以上了。


在第五轮时，模型参数成功保存在了`./ckpt/cp-0005.ckpt`。接下来我们就可以加载保存的模型参数，恢复整个卷积神经网络，进行真实图片的预测了。

## 五、图片预测
这里我们从本地选择一张 28 * 28 的数字图片，然后输入给程序。看看能不能准确预测。
安装 PIL：`pip3 install pillow`。

```
import tensorflow as tf
from PIL import Image
import numpy as np

class Predict(object):
    def __init__(self):
        latest = tf.train.latest_checkpoint('./ckpt')
        self.cnn = CNN()
        # 恢复网络权重
        self.cnn.model.load_weights(latest)

    def predict(self, image_path):
        # 以黑白方式读取图片
        img = Image.open(image_path).convert('L')
        flatten_img = np.reshape(img, (28, 28, 1)) / 255
        x = np.array([1 - flatten_img])

        # 预测
        y = self.cnn.model.predict(x)

        # np.argmax()取得最大值的下标，即代表的数字
        print(image_path)
        print(y[0])
        print('-> Predict digit', np.argmax(y[0]))

if __name__ == "__main__":
    app = Predict()
    app.predict('/Users/Desktop/4.png')
```
最后，你可以在[我的 GitHub](https://github.com/hezongjiang/TensorflowDemos/blob/master/classify_images_number_cnn.py) 上看到完整的代码。

在下一节{% post_link TensorFlow-2-x-MNIST-图像分类-3 《TensorFlow 2.x MNIST-图像分类(3)》 %}中，我们将使用更复杂的图片来进行分类测试。