# EasyRL task3 笔记


### 简单总结

0. 复习了一下策略相关的算法的特点：
* 基于策略的Agent：它直接去学习 policy，就是说你直接给它一个 state，它就会输出这个动作的概率。然后在这个 policy-based agent 里面并没有去学习它的价值函数
* 然后另外还有一种 Agent 是把这两者结合。把 value-based 和 policy-based 结合起来就有了 Actor-Critic agent。这一类 Agent 就把它的策略函数和价值函数都学习了，然后通过两者的交互得到一个更佳的状态
* 基于策略迭代的强化学习方法：agent会制定一套动作策略（确定在给定状态下需要采取何种动作），并根据这个策略进行操作。强化学习算法直接对策略进行优化，使制定的策略能够获得最大的奖励；基于价值迭代的强化学习方法，agent不需要制定显式的策略，它维护一个价值表格或价值函数，并通过这个价值表格或价值函数来选取价值最大的动作
* 基于价值迭代的方法只能应用在不连续的、离散的环境下（如围棋或某些游戏领域），对于行为集合规模庞大、动作连续的场景（如机器人控制领域），其很难学习到较好的结果（此时基于策略迭代的方法能够根据设定的策略来选择连续的动作)


1. 策略梯度
* 蒙特卡洛：不需要马尔可夫决策过程的状态转移函数和奖励函数，每回合只采样一次轨迹，回合结束时才学习
* 梯度上升：用神经网络拟合奖励函数，更新 θ 从而最大化期望奖励
* 演员：与环境进行交互的 agent
* 评论员：估计出优势函数的网络，优势函数是控制基线的，目的是让采样更合理
* 独热向量：只有一个是 1，其余都是 0 的向量
* on-policy：因为在做 policy gradient 时，我们需要有一个 agent、一个 policy 和一个 actor。这个 actor 先去跟环境互动去搜集资料，搜集很多的 τ，根据它搜集到的资料按照 policy gradient 的式子去更新 policy 的参数。所以 policy gradient 是一个 on-policy 的算法

![4.28](https://datawhalechina.github.io/easy-rl/img/ch4/4.28.png)

2. 近端策略优化(Proximal Policy Optimization，简称 PPO) 
* 是 policy gradient 的一个变形，它是现在 OpenAI 默认的强化学习算法
* 重要性采样(Importance Sampling，IS)：使用另外一种数据分布，来逼近所求分布的一种方法；我们可以在important sampling中将 p 替换为任意的 q，但是本质上需要要求两者的分布不能差的太多
* Proximal Policy Optimization (PPO)：避免在使用 important sampling 时，两种分布采样出来的 action 差别过大，导致 important sampling 结果偏差较大而采取的算法
* 基于off-policy的优势：使用off-policy的importance sampling后，我们不用 θ 去跟环境做互动，假设有另外一个 policy θ′，它就是另外一个actor。它的工作是他要去做demonstration，θ′的工作是要去示范给 θ 看。它去跟环境做互动，告诉 θ 说，它跟环境做互动会发生什么事。然后，借此来训练θ。我们要训练的是 θ ，θ′只是负责做 demo，负责跟环境做互动，所以 sample 出来的东西跟 θ 本身是没有关系的。所以你就可以让 θ′ 做互动 sample 一大堆的data，θ 可以update 参数很多次。然后一直到 θ train 到一定的程度，update 很多次以后，θ′再重新去做 sample，这就是 on-policy 换成 off-policy 的妙用
* KL divergence：本质来说，KL divergence是一个function，其度量的是两个action （对应的参数分别为θ 和 θ′）间的行为上的差距，而不是参数上的差距
* PPO-Clip: 实现起来比 KL 散度简单，而且效果似乎还不错

### 问题

1. 不太确定策略梯度是否有模型；第三章说：免模型强化学习方法没有获取环境的状态转移和奖励函数，而是让智能体与环境进行交互，采集大量的轨迹数据，智能体从轨迹中获取信息来改进策略，从而获得更多的奖励；而图片里又有“策略模型”
2. 感觉在介绍算法之前，最好先全面介绍一下算法的属性。比如是基于策略还是价值，还是说都有；是 on-policy 还是 off-policy；是否有模型；是基于蒙特卡洛、动态规划还是时序差分；跟之前的算法相比优势在哪里；不然总感觉不清晰
