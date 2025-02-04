---
title: 分清一堆 AI 技术词
catalog: true
date: 2025-02-04 18:35:10
subtitle:
header-img:
tags:
- 机器学习
---
<meta name="referrer" content="no-referrer"/>
说起近期的热门科技词汇，AIGC 当之无愧位列其中，但你真的了解 AIGC 吗？

![](https://upload-images.jianshu.io/upload_images/2708793-0601cbcad67e50b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最近我们的生活突然被改变了，发现 AI 可以帮忙生成文字、图片、音频视频等等内容了，而且让人难以分清，背后的创作者到底是人还是 AI。这些 AI 生成的内容被叫做 AIGC，它是 AI Generated Conten，即「AI生成内容」的简写。

![aigc](https://upload-images.jianshu.io/upload_images/2708793-c606deff09a6200c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

像 ChatGPT 生成的文章，Github Copilot 生成的代码等等，都属于AIGC。

而海外更流行的是另外一个词：Generative AI 即「生成式 AI」。

从字面上来看，生成式 AI 和 AIGC 之间的关系很好理解：生成式 AI 所生成的内容就是 AIGC，所以 ChatGPT、Github Copilot 都属于生成式 A。

那么生成式 AI 和机器学习、监督学习、无监督学习、强化学习、深度学习、大语言模型等等词汇之间又是什么关系呢？

这个很难一言以蔽之，但通过一张图，就可以直观理解它们之间的关系了。

![](https://upload-images.jianshu.io/upload_images/2708793-b92ba2c45c447b19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1、AI
AI 也叫人工智能，是计算机科学下的一个学科，旨在让计算机系统去模拟人类的智能，从而解决问题和完成任务。

早在二十世纪中期，AI 就被确定为了一个学科领域，在此后数 10 年间经历过多轮低谷与繁荣。

### 2、机器学习

![](https://upload-images.jianshu.io/upload_images/2708793-dcf3961fa46c9f60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

机器学习是 AI 的一个子集，它的核心在于不需要人类做显示编程，而是让计算机通过算法自行学习和改进，去识别模式，做出预测和决策。

比如我们给电脑大量玫瑰和向日葵的图片，让电脑自行识别模式，总结规律，从而能对没见过的图片进行预测和判断，这种就是机器学习。

机器学习领域下有多个分支，包括监督学习、无监督学习、强化学习。

![](https://upload-images.jianshu.io/upload_images/2708793-7dab14ad072605b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.1、监督学习

在监督学习里，机器学习算法会接受有标签的训练，数据标签就是期望的输出值，所以每个训练数据点都既包括输入特征，也包括期望的输出值。

算法的目标是学习输入和输出之间的映射关系，从而在给定新的输入特征后，能够准确预测出相应的输出值

经典的监督学习任务包括分类——也就是把数据划分为不同的类别，以及回归——也就是对数值进行预测。

比如拿一堆猫猫狗狗的照片和照片，对应的猫狗标签进行训练，然后让模型根据没见过的照片预测是猫还是狗，这就属于分类。

![](https://upload-images.jianshu.io/upload_images/2708793-997e1ba113cac9c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

拿一些房子特征的数据，比如面积卧室树是否带阳台等，和相应的房价作为标签进行训练，然后让模型根据没见过的房子的特征预测房价，这就属于回归。

![](https://upload-images.jianshu.io/upload_images/2708793-c7ed02d1b29a5051.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.2、无监督学习

和监督学习不同的是，他学习的数据是没有标签的，所以算法的任务是自主发现数据里的模式和规律。

经典的无监督学习任务包括聚类，也就是把数据进行分组。比如拿一堆新闻文章，让模型根据主题或内容的特征，自动把相似文章进行组织。

![](https://upload-images.jianshu.io/upload_images/2708793-a0078b431280536e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.3、强化学习

而强化学习则是让模型在环境里采取行动，获得结果反馈，从反馈里学习，从而能在给定情况下采取最佳行动，来最大化奖励或是最小化损失。

所以就跟训小狗似的，刚开始的时候小狗会随心所欲做出很多动作，但随着和训犬师的互动，小狗会发现某些动作能够获得零食，某些动作没有临时，某些动作甚至会遭受惩罚，通过观察动作和奖惩之间的联系，小狗的行为会逐渐接近训犬师的期望。

![](https://upload-images.jianshu.io/upload_images/2708793-823100457c8c16cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

强化学习可以应用在很多任务上，比如说让模型下围棋，获得不同行动导致的奖励或损失反馈，从而在一局局游戏里优化策略，学习如何采取行动达到高分。

![](https://upload-images.jianshu.io/upload_images/2708793-eb12037d4513846d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那问题来了，深度学习属于这三类里的哪一类呢？

### 3、深度学习

深度学习不属于里面的任何一类，深度学习是机器学习的一个方法，**核心在于使用人工神经网络**，模仿人脑处理信息的方式，通过层次化的方法提取和表示数据的特征。

![](https://upload-images.jianshu.io/upload_images/2708793-f785d15237ef6749.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

神经网络是由许多基本的计算和储存单元组成，这些单元被称为神经元，这些神经元通过层层连接来处理数据，并且深度学习模型通常有很多层，因此称为深度。

比如要让计算机识别小猫的照片，在深度学习中，数据首先被传递到一个输入层，就像人类的眼睛看到图片一样，然后数据通过多个隐藏层，每一层都会对数据进行一些复杂的数学运算，来帮助计算机理解图片中的特征。

例如小猫的耳朵眼睛等等，最后计算机会输出一个答案，表明这是否是一张小猫的图片。

![](https://upload-images.jianshu.io/upload_images/2708793-ac1febf153dc6d6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

神经网络可以用于监督学习，无监督学习，强化学习，所以深度学习不属于他们的子集。

生成式 AI 是深度学习的一种应用，它利用神经网络来识别现有内容的模式和结构，学习生成新的内容，内容形式可以是文本、图片、音频等等。

![](https://upload-images.jianshu.io/upload_images/2708793-756652d96da70c85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而大语言模型也叫 LLM（Large Language Model），也是深度学习的一种应用，专门用于进行自然语言处理任务。

![](https://upload-images.jianshu.io/upload_images/2708793-dc1fde2e6e5e7a67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


大语言模型里面的大字，说明模型的参数量非常大，可能有数百亿甚至到万亿个，而且训练过程中也需要海量文本数据集，所以能更好的理解自然语言，以及生成高质量的文本

![](https://upload-images.jianshu.io/upload_images/2708793-a57ad59f5ed50df1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大语言模型的例子有非常多，比如 GPT 可以进行文本的理解和生成。以 GPT3 为例，它会根据输入提示以及前面生成过的，通过概率计算，逐步生成下一个词或 token 来输出文本序列。

但不是所有的生成式 AI 都是大语言模型，也不是所有的大语言模型是为生成式 AI。

![](https://upload-images.jianshu.io/upload_images/2708793-03f563204fff01a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前半句很好理解，生成图像的扩散模型就不是大语言模型，它并不输出文本。

不是所有大语言模型都是生成式 AI。这是因为有些大语言模型由于其架构特点，不适合进行文本生成。

谷歌的 BERT 模型就是一个例子，它的参数量和训练数据很大，属于大语言模型。应用方面，BERT 理解上下文的能力很强，因此被谷歌用在搜索上，用来提高搜索排名和信息摘录的准确性，它也被用于情感分析、文本分类等任务。但 BERT 不擅长文本生成，特别是连贯的长文本生成，此类模型不属于生成式 AI 的范畴。


### 4、总结

到这里，我们可以很好的区分，AIGC 和 AI、机器学习、监督学习、无监督学习、强化学习、深度学习、神经网络、大语言模型这些概念了。这些概念共同构成了生成式AI的核心要素。

