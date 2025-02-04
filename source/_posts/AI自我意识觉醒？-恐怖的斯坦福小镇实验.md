---
title: AI自我意识觉醒？--恐怖的斯坦福小镇实验
catalog: true
date: 2025-02-04 19:04:21
subtitle:
header-img:
tags:
- 机器学习
---
<meta name="referrer" content="no-referrer"/>
当世界上所有的人还沉浸在 ChatGPT 带来的震惊中时，2023 年 4 月谷歌和斯坦福大学的几个研究人员做了一个很有意思的实验。

他们在一个虚拟人生的游戏中，放入了 25 个 AI 小人，并为这些 AI 设定了相应的角色和社会关系，比如小林是一个药店的店长，而他老婆是一位大学教授，他们还有一个上大学的儿子，喜欢音乐，诸如此类。

![](https://upload-images.jianshu.io/upload_images/2708793-43c3fe81beed54de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

小镇麻雀虽小，五脏俱全，有药店，有学校，有商店，有酒吧，有咖啡馆，当然还有每个人自己的家，AI 小人可以吃饭睡觉刷牙洗澡，买东西，去咖啡馆，或者在街上散步。

![](https://upload-images.jianshu.io/upload_images/2708793-14b41658d297473a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

与我们见过的常规游戏不同的是，在这个游戏中**没有**预先设定的剧本，AI小人每天的生活由他们自己决定，出乎意料的是这些AI就根据这点初始设定，在游戏中有滋有味的生活起来了。

一段时间以后，它们交了新的朋友，互相交换小镇的新闻，甚至成功举办了一个情人节派对，在派对上，一对互相暗恋的 AI 还获得了亲密接触的机会。这个实验被称为斯坦福小镇实验。

![](https://upload-images.jianshu.io/upload_images/2708793-e16f667b2cd6ff99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在斯坦福小镇上到底发生了什么，我们之前认为没有动机，没有规划，没有行为能力的 AI 似乎活了，它们学会了安排自己一天的生活，维护和发展社交关系，并能够自己组织起一项活动，AI 获得了心智，开始觉醒了吗？

要回答这个问题，需要先从一个概念开始说起，AI Agent，AI 代理。

## 1、AI Agent——AI 代理

大家都用过 ChatGPT，知道 ChatGPT 拥有超越常人的知识，通过非常巨大的预训练数据，了解了非常多的东西，可以在一问一答中和它有效的交流。

但只会聊天的 AI 实际的作用并不大，它的先验知识停留在过去的某一个时间点，它无法搜索互联网获得最新的知识，无法买机票、订酒店、叫外卖，也无法对最新的八卦提出看法，陪你科学的吃瓜，最重要的是没有规划能力，也缺乏长期的记忆，今天你跟它掏心掏肺的聊了半天，明天就不记得你是哪根葱了，这让它的实际应用处处受限，离我们所设想的通用人工智能似乎还差很远。

所以自从 GPT 出现以后，就有很多人在想如何利用 GPT 的先验知识，让AI做出更多的事情。

2023 年 3 月底，一个叫做 AutoGPT 的开源项目首先做出了尝试，他在 ChatGPT 上封装了一套框架，补全了 ChatGPT 所缺的这些东西，于是一个全新的概念出现了，这就是 AI Agent，这些框架也叫 AI 智能体。

一个什么样的 AI 才可以叫它智能体呢，首先他必须有独立的记忆，他要记住自己是谁，记住过去发生的事情，并形成自己的记忆流。

![](https://upload-images.jianshu.io/upload_images/2708793-dde38539126215f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后必须有规划，如果有一个目标，应该可以根据过去的经验来，把这个目标分解成一个一个的任务，然后按顺序去执行它，比如先赚他一个亿。

![](https://upload-images.jianshu.io/upload_images/2708793-3a0fabcd260adcda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后得有行动能力，具有在现实世界中实现它的渠道。

![](https://upload-images.jianshu.io/upload_images/2708793-1a62bcfce5a4d313.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这三样都是大模型所缺少的，所以几乎所有的 Agent 的框架都是在这三个层面，补全了大模型的不足。

![](https://upload-images.jianshu.io/upload_images/2708793-025157bcb410901d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你了解过任何一个 Agent 框架，就会发现他们一般分为四个模块。

最中间核心的肯定是大模型的调用模块，通过调用大模型的 API 来实现决策，在大模型模块之外有三个模块，分别是记忆、计划和工具。

![](https://upload-images.jianshu.io/upload_images/2708793-d92422bd5f9d270d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

「记忆模块」负责存储记忆。比如一个医疗 AI，最好能记住你过去和他聊过的一切信息，这样才能更好的判断你现在的身体状况。

![](https://upload-images.jianshu.io/upload_images/2708793-4340e51ded13ca45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

「计划模块」负责把目标分解成需要实现的一个个任务，比如让 AI 策划一个生日派对，它至少应该知道去网上选蛋糕，准备礼物，制作邀请函，并检索出你的好友一个一个发给他们。

![](https://upload-images.jianshu.io/upload_images/2708793-2d177467278b6efb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而「工具模块」则是去调用很多网站的 API 去完成一个一个任务的过程，比如在网上找蛋糕店，搜索评论，选择最好的，然后调用支付接口下单，调用图片生成 AI 制作邀请函等等。

![](https://upload-images.jianshu.io/upload_images/2708793-48fac9b41455afbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就像积木一样，可以搭建到大模型之上，就是在大模型外面包个壳，让它实现一些更复杂的任务。

但是斯坦福小镇的居民，也是这样简单粗暴的拼出来的，只不过斯坦福小镇在记忆和规划层面，做了更多精巧的设想。

## 2、斯坦福小镇

#### 2.1、记忆

首先我们来看记忆，斯坦福小镇居民的记忆来自于他们日常的活动，包括他自己所做的事情，以及他观察到周围环境中所发生的事情。

比如一个叫做伊莎贝拉的 AI，在咖啡馆工作一段时间内，可能会发生这样四件事情：
* 伊莎贝拉正在摆放糕点
* 玛瑞亚在喝咖啡的同时准备化学考试
* 伊莎贝拉和玛瑞亚正在商量在哈布斯咖啡店策划情人节派对
* 冰箱里什么都没有

![](https://upload-images.jianshu.io/upload_images/2708793-383a0594e2db6ca9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这四件事情就会按照顺序，记录到一个叫做记忆流的列表里，但这种记忆流的记录方式有很大的问题，很快就会积累非常多的记忆，这些记忆不光不便于检索，并且会有很多无关紧要的事情。

如果有人问伊莎贝拉，你最近在忙什么呢？伊莎贝拉应该怎么回答？

她的记忆里充满了像冰箱空了，或者刷牙洗澡睡觉之类的记忆，总不能把这些都一股脑地告诉对方吧，所以我们需要一个很好的记忆检索系统，能够把最关键的记忆提取出来。

![](https://upload-images.jianshu.io/upload_images/2708793-8a9fb2d2da986340.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

斯坦福小镇中，采用了一个检索函数来解决这一问题，他们从三个角度为一条记忆打分，时效性，重要性和相关性。

![](https://upload-images.jianshu.io/upload_images/2708793-55917638897bf7f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

离现在越近的事情肯定应该更容易被想起来，这就是「时效性」。

失恋这种记忆肯定比每天刷牙洗脸例行公事更让人刻骨铭心，这就是「重要性」。

如果问你与音乐相关的问题，你所有和音乐相关的记忆肯定更先被提出来，这就是「相关性」。

按照这三点给每一条记忆加权组合评分，就可以挑选出最有用的记忆了。

接下来 Agent 需要把这些记忆放到提示词中，连同对方的问题一起扔给 chat GPT，chat GPT会根据这些情况给出一个合理的回答，Agent 收到这个回答以后，把这句话说给对方听。

![](https://upload-images.jianshu.io/upload_images/2708793-8d1954b851a39743.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以现在你明白 Agent 为什么叫 Agent 了吧。

#### 2.2、社交

接下来一个更难的问题，AI 是如何感知到自己的社交关系，并对周围的人做出评价的呢？比如谁跟自己更亲密一些，谁和自己很疏远？如果仅仅是简单的记忆列表，是不足以得出这些结论来的。

这需要 AI 具有一种更高的能力——反思。

在斯坦福小镇中，AI 的反思是用一种非常巧妙的方式实现的，AI会把最近的100条记忆全部都列出来，然后扔给chat GPT，直接问他，基于这些事情，我们可以回答哪些主题的高层次问题。

说白了就是把记忆直接抛给 GPT 看，我都干了这么些事了，你有啥问题要问吗？

![](https://upload-images.jianshu.io/upload_images/2708793-e4d9af97cd06831d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

GPT 就会搜肠刮肚的想出一些问题：比如你和张三的关系怎么样啊？然后 Agent 就用这个问题去自己的记忆流中搜索，到这个时候就跟第一步一样了，加权相加以后，最重要的几条记忆就会被检索出来。

然后 Agent 再把这几条记忆扔给 GPT：
* 张三请我吃饭了
* 张三给我买东西
* 张三看见我大老远就跑过来

![](https://upload-images.jianshu.io/upload_images/2708793-9c8491ed596754af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后问 GPT，你觉得我和张三咋样，GPT 根据这些信息回复：我觉得张三挺喜欢你的。

![](https://upload-images.jianshu.io/upload_images/2708793-25ad2f0fab0b3c59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


原来张三喜欢我哦，Agent 把这条回答拿过来作为反思，放到自己的小本本里记忆留中，这就是 AI Agent 反思的过程。

有意思的是，因为反思本身也是记忆流的一部分，所以 Agent 还可以把这些反思作为记忆，再次提交给 GPT 做更高层次的反思，最终形成一颗反思树。

#### 2.3、计划

然后我们看计划，与一般的 AI Agent 的不同，小镇居民需要知道自己的一天应该怎么度过，所以他需要有一天的计划，这个计划是怎么产生的呢？是通过一种递归的方式，一步一步拆解完成的。

首先 Agent 会向大模型提供自己的基本信息：
* 自己是谁
* 自己是做什么工作的
* 自己的兴趣是什么
* 自己最近的目标是什么

![](https://upload-images.jianshu.io/upload_images/2708793-66cd552169334b2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


让大模型帮自己制定一天的计划，然后把这个计划先保存在自己的记忆流中，接下来取出计划的第一部分，然后提交给大模型，让大模型继续把计划细化，就这样反复几步，一天的计划就被细化好了。Agent 按照这个计划来安排一天的行动就可以了。

假设张三的计划是出去买自行车，但是一出门就碰到了自己暗恋的小美，然后小美说要不跟我去小树林散散步怎么样。

张三应该怎么回答？不行，我要去买自行车，没时间去散步，再见。

所以计划是比不上变化的，这种情况怎么处理呢？什么时候应该坚持计划？什么时候应该做出变化？

很简单，还是交给 GPT 来决策，Agent 会老老实实交代自己的背景：
* 我是谁
* 我是干啥的
* 我和小美是什么关系
* 我计划去干什么
* 然后小美跟我说了啥

最后问 GPT 我应该咋回答，GPT 会说这还不赶紧去散步，还要啥自行车啊，Agent 于是就改变计划和小美去散步了。

这就是小镇居民基本的决策和行动模式了。不管结构多复杂，核心的结构还是去大模型做决策，内事不决问模型，外事不决还是问模型，并没有什么黑科技，更没有什么意识觉醒，就是这样很 low 的方式，在 AI 真的精心的设计下，仍然呈现出了惊人的效果，甚至让我们看到了一种虚拟社会的雏形。

同样是调用大模型，只要想法和创意足够好，我们当然也可以做出类似的东西，，所以 AI Agent 不在于实现架构有多么高深，而在于对现实问题的理解和创意，只要想法够好，解决的问题足够痛点，就可以做出足够好的应用。
