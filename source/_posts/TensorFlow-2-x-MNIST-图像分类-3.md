---
title: TensorFlow 2.x - MNIST 图像分类(3)
catalog: true
date: 2020-05-15 10:41:25
subtitle: TensorFlow入门
header-img: tensorflow.jpg
tags: 机器学习
---


这一节将使用 [TensorFlow 官方教程](https://tensorflow.google.cn/tutorials/keras/classification?hl=zh_cn) 的一个例子——Basic classification: Classify images of clothing——基础分类：衣物图片分类，训练一个神经网络模型来分类像运动鞋和衬衫这样的衣服的图像。

有了{% post_link TensorFlow-2-x-MNIST-图像分类-1 《TensorFlow 2.x MNIST-图像分类(1)》 %}和{% post_link TensorFlow-2-x-MNIST-图像分类-2 《TensorFlow 2.x MNIST-图像分类(2)》 %}这两节的铺垫，本节将显得比较简单。

## 一、加载数据

在之前的文章中，我们都是使用 MNIST，但是 MNIST 太简单了，只需要几分钟的训练，准确率就可以达到 99% 以上，并且大部分图片可能只需要几个像素点就可以区分开来。真实世界千变万化，我们需要更真实的图片。

本次使用的数据集是 Fashion MNIST。该数据集由 70,000 张黑白图片构成，每张图片大小为 28x28，图片数量和大小与经典 MNIST 完全相同，但 Fashion MNIST 是由十类服饰图片构成，MNIST 是手写数字。Fashion MNIST 更有挑战性，更适合用来验证算法。Fashion-MNIST 的目的是要代替 MNIST。

![Fashion-MNIST](https://upload-images.jianshu.io/upload_images/2708793-c729995af3629e9c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

废话不多说，先下载四个数据集：

（1）训练集图片：[train-images-idx3-ubyte.gz](http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/train-images-idx3-ubyte.gz)，60,000张，26 MBytes。
（2）训练集标签：[train-labels-idx1-ubyte.gz](http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/train-labels-idx1-ubyte.gz)，60,000张，29 KBytes。
（3）测试集图片：[t10k-images-idx3-ubyte.gz](http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/t10k-images-idx3-ubyte.gz)，10,000张，4.3 MBytes。
（4）测试集标签：[t10k-labels-idx1-ubyte.gz](http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/t10k-labels-idx1-ubyte.gz)，10,000张，5.1 KBytes。

下载后，加载数据：

```
import tensorflow as tf
from tensorflow import keras
import matplotlib.pyplot as plt
import os
import gzip
import numpy as np

def load_data(data_folder):
    files = ['train-labels-idx1-ubyte.gz', 'train-images-idx3-ubyte.gz',
             't10k-labels-idx1-ubyte.gz', 't10k-images-idx3-ubyte.gz']
    paths = []
    for f_name in files:
        paths.append(os.path.join(data_folder, f_name))

    with gzip.open(paths[0], 'rb') as lb_path:
        y_train = np.frombuffer(lb_path.read(), np.uint8, offset=8)

    with gzip.open(paths[1], 'rb') as img_path:
        x_train = np.frombuffer(img_path.read(), np.uint8, offset=16).reshape(len(y_train), 28, 28)

    with gzip.open(paths[2], 'rb') as lb_path:
        y_test = np.frombuffer(lb_path.read(), np.uint8, offset=8)

    with gzip.open(paths[3], 'rb') as img_path:
        x_test = np.frombuffer(img_path.read(), np.uint8, offset=16).reshape(len(y_test), 28, 28)

    return (x_train, y_train), (x_test, y_test)

(train_images, train_labels), (test_images, test_labels) = load_data(os.path.dirname(__file__) + '/data/fashion')
```
这里的数据加载方法与 TensorFlow 官方教程上的不同，因为官方教程加载数据时，直接下载，而下载链接需要翻墙，如果你能科学上网，可以使用如下官方教程，代替上一段代码：
```
(train_images, train_labels), (test_images, test_labels) = keras.datasets.fashion_mnist.load_data()
```
## 二、数据预处理

数据格式如下：
```
train_images.shape # (60000, 28, 28)
len(train_labels) # 60000
train_labels # ([9, 0, 0, ..., 3, 0, 5], dtype=uint8)
test_images.shape # (10000, 28, 28)
len(test_labels) # 10000
```

然后我们查看第一张图片：
```
plt.figure()
plt.imshow(train_images[0])
plt.show()
print(train_labels[0])  # 9
```

![Ankle boot](https://upload-images.jianshu.io/upload_images/2708793-7ec317b0b9329c37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图片大小 28x28，每个像素值取值范围 0-255。标签是整数，取值范围 0-9，与实际的服饰类别对应关系如下。
Label | Class 
:-:|:-:
0 | T-shirt/top （T恤） | 
1 | Trouser（裤子）| 
2 | Pullover（套衫） | 
3 | Dress （裙子） | 
4 | Coat（外套） | 
5 | Sandal（凉鞋） | 
6 | Shirt（汗衫） | 
7 | Sneaker（运动鞋） | 
8 | Bag（包） | 
9 | Ankle boot（短靴） | 

每个图像都映射到一个标签。上图的标签是 9，所以图片中应该是 Ankle boot (短靴)，虽然没看出哪里短。

```
class_names = ['T-shirt/top', 'Trouser', 'Pullover', 'Dress', 'Coat', 'Sandal', 'Shirt', 'Sneaker', 'Bag', 'Ankle boot']
```

图片的每个像素值在 0-255 之间，需要转为 0-1。训练集和测试集都需要经过相同的处理。

```
train_images = train_images / 255.0
test_images = test_images / 255.0
```

显示前 25 张图片与标签的对应关系：

```
# show first 25 images and labels
plt.figure(figsize=(10, 10))
for i in range(25):
    plt.subplot(5, 5, i + 1)
    plt.xticks([])
    plt.yticks([])
    plt.imshow(train_images[i], cmap=plt.cm.binary)
    plt.xlabel(class_names[train_labels[i]])
plt.show()
```

![](https://upload-images.jianshu.io/upload_images/2708793-6c119b414227ef89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三、搭建神经网络

搭建神经网络分为两步，首先配置模型的每一层，然后编译模型。

**配置模型层**

神经网络的基本构成是网络层 (layer)，大部分深度学习网络都由多个简单的 layers 构成。

```
model = keras.Sequential([
    keras.layers.Flatten(input_shape=(28, 28)),
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dense(10, activation='softmax')
])
```

网络的输入层 `tf.keras.layers.Flatten` 将输入的图片从 28 × 28 的二维数组（28 × 28 像素）转为 784 的一维数组，这一层的作用仅仅是将每一行值平铺在一行，没有参数需要学习。

接下来是 2 层 `tf.keras.layers.Dense`，即全连接层 (Fully Connected, FC)，第一层 Dense 有 128 个神经元。第二层有 10 个神经元，经过 softmax 后，返回了和为 1 长度为 10 的概率数组，每一个数分别代表当前图片属于分类 0-9 的概率。

**编译模型**

在开始训练前，模型需要编译 (Compile) 并设置一些参数：

1. Loss function - 损失函数，训练时评估模型的正确率，希望最小化这个函数，往正确的方向训练模型。
2. Optimizer - 优化器算法，更新模型参数的算法。
3. Metrics - 指标，用来监视训练和测试步数，下面的例子中使用 accuracy，即图片被正确分类的比例。

```
model.compile(optimizer='adam',
              loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True),
              metrics=['accuracy'])
```

## 四、训练模型

训练神经网络，通常有以下几个步骤。

1. 传入训练集数据，`train_images` 和 `train_labels`。
2. 训练模型，关联图片和标签。
3. 使用测试集 `test_images` 预测，并用标签 `test_labels ` 进行验证。

**Feed the model**

调用 `model.fit` 方法开始训练。
```
model.fit(train_images, train_labels, epochs=10)
```
```
Epoch 1/10
1875/1875 [==============================] - 5s 3ms/step - loss: 0.5025 - accuracy: 0.8243
..............................
Epoch 7/10
1875/1875 [==============================] - 5s 3ms/step - loss: 0.2696 - accuracy: 0.9001
Epoch 8/10
1875/1875 [==============================] - 5s 3ms/step - loss: 0.2576 - accuracy: 0.9035
Epoch 9/10
1875/1875 [==============================] - 6s 3ms/step - loss: 0.2491 - accuracy: 0.9075
Epoch 10/10
1875/1875 [==============================] - 6s 3ms/step - loss: 0.2396 - accuracy: 0.9112
```
经过 10 轮训练后，准确率达到了 91.12%。

**评估准确率**

使用测试集数据看看模型训练的结果：

```
test_loss, test_acc = model.evaluate(test_images,  test_labels, verbose=2)
print('\nTest accuracy:', test_acc)  # Test accuracy: 0.8865000009536743
```

测试集的准确率低于训练集，训练集和测试集准确率之间的差距代表模型过拟合 (overfitting)。即对于训练中没有见过的新数据，模型表现差。如果你对过拟合不熟悉，可以看看[《过拟合(Overfitting) 与 Dropout》](https://www.jianshu.com/p/b3d98ce0bc1e)，没错，也是我的文章。

**预测**

使用 predict 函数进行预测，并显示图片，打印预测结果及标签。

```
# 预测
predictions = model.predict(test_images)

idx = 20
plt.figure()
plt.imshow(train_images[idx])  # 显示图片
plt.show()
print(predictions[idx])
predict = int(np.argmax(predictions[idx]))
print('predict: ' + class_names[predict])  # 预测结果
print('label: ' + class_names[test_labels[idx]])  # 标签
```
看一下第一个预测结果。

```
[3.1960328e-09 1.2830121e-11 1.0000000e+00 1.5766790e-18 1.2614347e-16
 7.3473108e-24 1.2678178e-11 2.2807704e-21 2.7815026e-11 3.0496564e-14]
predict: Pullover
label: Pullover
```

结果是一个长度为 10 的数组，数组的每个值代表模型认为是对应标签的概率。上述例子中 9 号标签的概率最大 1.0000000e+00，即模型认为该图片是 9 号标签的概率是 100%。

![Pullover](https://upload-images.jianshu.io/upload_images/2708793-0c5551083b3b497d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

预测结果和标签结果相同，都是 Pullover。