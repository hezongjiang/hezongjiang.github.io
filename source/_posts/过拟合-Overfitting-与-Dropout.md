---
title: 过拟合(Overfitting) 与 Dropout
catalog: true
date: 2018-03-08 20:47:35
subtitle:
header-img: timg.jpeg
tags:
- 机器学习
---
## 一、过拟合(Overfitting)
Overfitting 也被称为过度学习，过度拟合。 它是机器学习中常见的问题。 举个Classification（分类）的例子。

![过拟合](http://upload-images.jianshu.io/upload_images/2708793-76b8536ec6f1bb06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中黑色曲线是正常模型，绿色曲线就是overfitting模型。尽管绿色曲线很精确的区分了所有的训练数据，但是并没有描述数据的整体特征，对新测试数据的适应性较差。

## 二、解决方法
**方法一**： 增加数据量，大部分过拟合产生的原因是因为数据量太少了。如果我们有成千上万的数据，线也会慢慢被拉直，变得没那么扭曲。

**方法二**：运用正规化。L1、L2 Regularization 等等。 我们简化机器学习的公式为 `y = W x`。在过拟合中，`W` 的值往往变化得特别大或特别小。为了不让W变化太大，我们在计算误差上做些手脚。原始的 cost 误差是 cost = (预测值-真实值)的平方. 如果 W 变得太大, 我们就让 cost 也跟着变大, 变成一种惩罚机制. 这一种形式的正规化：{% post_link L1-L2-正规化-Regularization L1-L2-正规化-Regularization %}。

**还有一种**：专门用在神经网络的正规化的方法，叫作 Dropout。在训练的时候，随机忽略掉一些神经元和神经联结，是这个神经网络变得“不完整”。用一个不完整的神经网络训练一次。到第二次再随机忽略另一些, 变成另一个不完整的神经网络。有了这些随机 Drop 掉的规则，我们可以想象其实每次训练的时候，我们都让每一次预测结果都不会依赖于其中某部分特定的神经元。像L1、L2正规化一样，过度依赖的 W，也就是训练参数的数值会很大，L1、L2会惩罚这些大的参数。Dropout 的做法是从根本上让神经网络没机会过度依赖。

## 三、建立 Dropout 层
[戳这里查看全部代码](https://github.com/hezongjiang/TensorflowDemos/blob/master/dropout.py)

首先导入框架
```
import tensorflow as tf
from sklearn.datasets import load_digits
from sklearn.cross_validation import train_test_split
from sklearn.preprocessing import LabelBinarizer
```
```
keep_prob = tf.placeholder(tf.float32)
...
...
Wx_plus_b = tf.nn.dropout(Wx_plus_b, keep_prob)
```
这里的keep_prob是保留概率，即我们要保留的结果所占比例，它作为一个placeholder，在run时传入， 当keep_prob=1的时候，相当于100%保留，也就是dropout没有起作用。 下面我们分析一下程序结构，首先准备数据，

```
digits = load_digits()
X = digits.data
y = digits.target
y = LabelBinarizer().fit_transform(y)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.3)
```
其中X_train是训练数据, X_test是测试数据。 然后添加隐含层和输出层
```
# add output layer
l1 = add_layer(xs, 64, 50, 'l1', activation_function=tf.nn.tanh)
prediction = add_layer(l1, 50, 10, 'l2', activation_function=tf.nn.softmax)
```
loss函数（即最优化目标函数）选用交叉熵函数。交叉熵用来衡量预测值和真实值的相似程度，如果完全相同，交叉熵就等于零。

```
cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction), reduction_indices=[1]))  # loss
```
train方法（优化算法）采用梯度下降法
```
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```
最后开始train，总共训练500次。
```
sess.run(train_step, feed_dict={xs: X_train, ys: y_train, keep_prob: 0.5})
#sess.run(train_step, feed_dict={xs: X_train, ys: y_train, keep_prob: 1})
```
结果如下：
![drop_out = 0.5](http://upload-images.jianshu.io/upload_images/2708793-1a65f27af65c2303.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![drop_out = 1](http://upload-images.jianshu.io/upload_images/2708793-f62f7ca0436aca06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
