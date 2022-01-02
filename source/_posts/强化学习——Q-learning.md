---
title: 强化学习——Q-learning
catalog: true
date: 2018-04-15 11:48:36
subtitle:
header-img: Reinforcement.jpg
tags:
- 机器学习
---
## 一、什么是Q-learning
Q_learning是强化学习中的一个决策算法，如果你还不知道什么是强化学习，可以参看{% post_link 强化学习 强化学习 %}这篇文章。

## 二、Q-Learning 决策
假设我们的行为准则已经学习好了，现在我们处于状态 s(tate) 1，有两个行为 a(ction) 1、a(ction) 2，在这种 s1 状态下，a2 带来的潜在奖励要比 a1 高（如下表所示），这里的潜在奖励我们可以用一个有关于 s 和 a 的 Q 表格代替，在Q表格中，Q(s1, a1) = -2，要小于 Q(s1, a2) = 1，所以我们判断要选择 a2 作为下一个行为。（注意，此时Q表里的数据是随机设置的，称为估计值，而Q_learning的**目的**就是更新Q表，使之变成一个准确的Q表。）

|       Q   |   a1  | a2 |
| :--------: | :----------: |:---------:|
| s1 | -2  | 1  |
| s2 | -4  |   2   |

当选择 a2 后，状态 s1 结束，我门到达状态 s2，我们还是有两个同样的选择，重复上面的过程，在行为准则 Q 表中寻找 Q(s2, a1)、Q(s2, a2) 的值，并比较他们的大小，选取较大的一个。接着根据 a2 我们到达 s3 并在此重复上面的决策过程。

## 三、Q-Learning 更新
根据 Q 表的估计，因为在 s1 中, a2 的值比较大，通过之前的决策方法，我们在 s1 采取了 a2，并到达 s2，这时，我们开始更新用于决策的 Q 表，接着我们分别看看两种行为哪一个的 Q 值大。

比如说 Q(s2, a2) 的值比 Q(s2, a1) 的大，所以我们把大的值乘上一个衰减值 gamma (比如0.9) 并加上到达 s2 时所获取的奖励 R(eward)，因为会获取实实在在的奖励 R，我们将这个作为我现实中 Q(s1, a2) 的值，但是我们之前是根据 Q 表估计 Q(s1, a2) 的值。所以有了现实和估计值，我们就能更新Q(s1, a2)，根据估计与现实的差距，将这个差距乘以一个学习效率 alpha 累加上旧的 Q(s1, a2) 的值，变成新的值。

![Q 表更新方式](https://upload-images.jianshu.io/upload_images/2708793-055034e4da8d984e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但时刻记住，我们虽然用 maxQ(s2) 估算了一下 s2 状态，但还没有在 s2 做出任何的行为，s2 的行为决策要等到更新完了以后再重新另外做。这就是 Off-Policy 的 Q learning 是如何决策和学习优化决策的过程。

## 四、Q-Learning 整体算法
```
Initialize Q arbitrarily // 随机初始化Q表
Repeat (for each episode): // 每一次从开始到结束是一个episode
    Initialize S // S为初始位置的状态
    Repeat (for each step of episode):
        Choose a from s using policy derived from Q(ε-greedy) //根据当前Q和位置S，使用一种策略，得到动作A，这个策略可以是ε-greedy等
        Take action a, observe r // 做了动作A，到达新的位置S'，并获得奖励R，奖励可以是1，50或者-1000
        Q(S,A) ← Q(S,A) + α*[R + γ*maxQ(S',a))-Q(s,a)] // 在Q中更新S
        S ← S'
    until S is terminal //即到游戏结束为止
```
这一段代码概括了我们之前所有的内容。这也是 Q_learning 的算法, 每次更新我们都用到了 Q 现实和 Q 估计，而且 Q learning 的重点就是在 Q(s1, a2) 现实中，也包含了一个 Q(s2) 的最大估计值，将对下一步的衰减的最大估计和当前所得到的奖励当成这一步的现实。

最后我们来说说这套算法中一些参数的意义。Epsilon Greedy 是用在决策上的一种策略，比如 epsilon = 0.9 时, 就说明有 90% 的情况我会按照 Q 表的最优值选择行为，10% 的时间使用随机选行为，这样做的目的是让其有机会跳出局部最优。alpha是学习率，来决定这次的误差有多少要被学习的，alpha是一个小于1 的数。gamma 是对未来 reward 的衰减值。


*以上内容参考[莫凡Python](https://morvanzhou.github.io/tutorials/machine-learning/)*。