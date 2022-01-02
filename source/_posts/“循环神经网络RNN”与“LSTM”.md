---
title: “循环神经网络RNN”与“LSTM”
catalog: true
date: 2018-03-29 13:58:42
subtitle: 
header-img: nn.jpg
tags: 
- 机器学习
---
## 一、为什么需要 RNN（Recurrent Neural Network）？
普通的神经网络，都只能单独的取处理一个个的输入，前一个输入和后一个输入是完全没有关系的。但是，某些任务需要能够更好的处理序列的信息，即前面的输入和后面的输入是有关系的。

比如，当我们在理解一句话意思时，孤立的理解这句话的每个词是不够的，我们需要处理这些词连接起来的整个序列； 当我们处理视频的时候，我们也不能只单独的去分析每一帧，而要分析这些帧连接起来的整个序列。也就是说，我们输入到神经网络的数据之间是相互关联的。

而普通的全连接NN就存在着一个问题——无法对时间序列上的变化进行建模。然而，样本出现的时间顺序对于自然语言处理、语音识别、手写体识别等应用非常重要。对了适应这种需求，就出现了题主所说的另一种神经网络结构——循环神经网络RNN。
![图1 与时间轴相关的数据](https://upload-images.jianshu.io/upload_images/2708793-574d31097d3aa78f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 二、什么是 RNN
RNN就是**神经元的输出可以在下一个时间戳直接作用到自身**，简单的说就是让数据之间的关联也能被神经网络分析。

![图2 RNN 基本模型](https://upload-images.jianshu.io/upload_images/2708793-771572cd35d01b88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那我们如何让数据间的关联也被 NN 加以分析呢？ 最基本的方式，就是记住之前发生的事情。那我们让神经网络也具备这种记住之前发生的事的能力，再分析 Data0 的时候，我们把分析结果存入记忆，然后当分析 Data1的时候，NN会产生新的记忆，但是新记忆和旧记忆是有联系的，RNN 会把新、旧记忆调用过来，一起分析。如果继续分析更多的有序数据，RNN就会把之前的记忆都累积起来，一起分析。可以用下图来表示。

![图3 RNN 模型展开](https://upload-images.jianshu.io/upload_images/2708793-6ab2945d0ca8f62e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

实际上，图3就是图2在时间轴上的展开。**(t+1) 时刻网络的最终结果 O(t+1) 是该时刻输入和所有历史共同作用的结果**，这就达到了对时间序列建模的目的。所以**RNN可以看成一个在时间上传递的神经网络，它的深度是时间的长度**。

**但是，BUT，However，这样的神经网络是有问题的。**

之前我们说过，RNN 是在有顺序的数据上进行学习的，为了记住这些数据，RNN 会产生对先前发生事件的记忆。不过一般形式的 RNN 有时候比较健忘，为什么会这样呢?

 假如，我们的目标信息，距离当前时刻比较久（很久以前发生），这个信息原的记忆要进过长途跋涉才能抵达最后一个时间点。然后我们得到误差，而且在反向传递得到的误差的时候，他在每一步都会乘以一个自己的参数 W，如果这个 W 是一个小于 1 的数，比如0.9，这个0.9 不断乘以误差，误差传到初始时间点也会是一个接近于零的数，我们把这个问题叫做**梯度消失(Gradient vanishing)**。

反之如果 W 是一个大于1 的数，比如1.1，不断累乘，则到最后变成了无穷大的数，这种情况我们叫做 **梯度爆炸(Gradient exploding)**。
![图4](https://upload-images.jianshu.io/upload_images/2708793-01638a5fe02f01c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如何解决这个问题呢？于是就产生了 LSTM 全称叫 长短期记忆网络 Long Short-Term Memory networks。

## 三、LSTM（ Long Short-Term Memory networks）
LSTM 就是为了解决 RNN 中 梯度消失 和 梯度爆炸 而诞生的。LSTM 和普通 RNN 相比，多出了三个控制器，输入控制，输出控制，忘记控制。 LSTM RNN 内部的情况是这样。

![图5 LSTM](https://upload-images.jianshu.io/upload_images/2708793-d306f64c60f463f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

整个Block相当于隐藏层的一个神经元。


输入门（input gate）: 一个Sigmoid层，观察 h(t-1) 和 x(t)，对于元胞状态c(​t−1) 中的每一个元素输出一个0~1之间的数。1表示“完全保留该信息”，0表示“完全丢弃该信息”.

![Input Gate Signal](https://upload-images.jianshu.io/upload_images/2708793-24672c5b89f9c1fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Input](https://upload-images.jianshu.io/upload_images/2708793-dcdf528647b9d26a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
遗忘门（forget gate): 一个Sigmoid层决定我们要更新哪些信息，并由一个tanh层创造了一个新的候选值（结果在(-1, 1)(−1,1)范围）
![Forget Gate](https://upload-images.jianshu.io/upload_images/2708793-04393c4d6d697e93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

中间Cell
![Cell](https://upload-images.jianshu.io/upload_images/2708793-04df70d577cb9543.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


输出门（output gate）：控制哪些信息需要输出
![Output Gate](https://upload-images.jianshu.io/upload_images/2708793-547aed744fe34960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Hidden Layer Output](https://upload-images.jianshu.io/upload_images/2708793-ba0cf92c6f8771ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下图为两个block为中间隐藏层示意图，可以结合上面公式来看就非常容易理解了。

![图6 LSTM连接](https://upload-images.jianshu.io/upload_images/2708793-bb0b1dbf622727a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

典型的工作流为：在“输入门”中，根据当前的数据流来控制接受细胞记忆的影响；接着，在 “遗忘门”里，更新这个细胞的记忆和数据流；然后在“输出门”里产生输出更新后的记忆和数据流。LSTM 模型的关键之一就在于这个“遗忘门”， 它能够控制训练时候梯度在这里的收敛性（从而避免了 RNN 中的梯度 vanishing/exploding 问题），同时也能够保持长期的记忆性。

如果我们把LSTM的 Forget Gate全部置0（总是忘记之前的信息），Input Gate全部 置1，Output Gate全部置1（把cell state中的信息全部输出），这样LSTM就变成一个标准的RNN。

最后：使用 TensorFlow 的一个 LSTM小栗子。红线是一个正弦曲线，蓝色的虚线会去拟合红实线，随着时间的推移，LSTM 记住了红线的规律，从而逐渐拟合。完整的代码可以看[这里](https://github.com/Hearsayer/TensorflowDemos/blob/master/LSTM.py)。
![LSTM 小栗子](https://upload-images.jianshu.io/upload_images/2708793-601a7da563682c3f.gif?imageMogr2/auto-orient/strip)
