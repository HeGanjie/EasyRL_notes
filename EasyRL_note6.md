# EasyRL task6 笔记


### 简单总结

1. 深度确定性策略梯度
  * 随机性/确定性？
    * 对随机性的策略来说，输入某一个状态 s，采取哪个 action 是根据概率随机抽取一个动作
    * 而对于确定性的策略来说，它没有概率的影响。当神经网络的参数固定下来了之后，输入同样的 state，必然输出同样的 action
  * 输出离散/连续动作
    * 要输出离散动作的话，我们就是加一层 softmax 层来确保说所有的输出是动作概率，且所有的动作概率加和为 1。
    * 要输出连续动作的话，一般可以在输出层这里加一层 tanh；我们拿到这个输出后，就可以根据实际动作的范围再做一下缩放，再输出给环境
  * 与 DQN 的区别
    * Deterministic 表示 DDPG 输出的是一个确定性的动作，可以用于连续动作的一个环境 
    * Policy Gradient 代表的是它用到的是策略网络，REINFORCE 算法每隔一个 episode 就更新一次，DDPG 网络是每个 step 都会更新一次 policy 网络
    * 经验回放这一块跟 DQN 是一样的，但 target network 这一块的更新跟 DQN 有点不一样
    * DDPG 直接在 DQN 基础上加了一个策略网络来直接输出动作值，所以 DDPG 需要一边学习 Q 网络，一边学习策略网络  
    * 属于 Actor-Critic 结构
  * Actor-Critic
    * 策略网络扮演的就是 actor 的角色，它负责对外输出动作
    * Q 网络就是评论家(critic)，它会在每一个 step 都对 actor 输出的动作打分，估计 actor 的 action 未来能有多少收益（Q 值，即 Q_w(s,a)）。 Actor 就需要根据环境目前的状态来做出一个 action；
    * Actor 根据评论家的打分来调整自己的策略，也就是更新 actor 的神经网络参数 θ， 争取下次可以做得更好。Critic 则是要根据环境的反馈 reward 来调整自己的打分策略，也就是要更新 critic 的神经网络的参数 w。Critic 的最终目标是让 Actor 的动作获得环境尽可能多的奖励，从而最大化未来的总收益
    * 需要注意的是：这里的 actor 是不管环境的，它只关注评论家
  * 优化目标
    * actor: 用来优化策略网络的梯度就是要最大化这个 Q 值，所以构造的 loss 函数就是让 Q 取一个负号求解让 Q 值最大的那个 action
    * critic: 真实的 reward r 和下一步的 Q 即 Q' 来去拟合未来的收益 Q_target
  * 目标网络：为了稳定这个 Q_target，DDPG 分别给 Q 网络和策略网络都搭建了 target network
  * 经验回放：跟 DQN 是一模一样；因为 DDPG 使用了经验回放这个技巧，所以 DDPG 是一个 off-policy 的算法
  * Exploration vs. Exploitation：如果 agent 使用同策略来探索（例如不使用经验回放？），在一开始的时候，它会很可能不会尝试足够多的 action 来找到有用的学习信号。为了让 DDPG 的策略更好地探索，我们在训练的时候给它们的 action 加了噪音
  * 常见问题：是已经学习好的 Q 函数开始显著地高估 Q 值；解决方案：双延迟深度确定性策略梯度(Twin Delayed DDPG，简称 TD3)
    * 截断的双 Q 学习(Clipped Dobule Q-learning)：通过最小化均方误差来同时学习两个 Q-function，两个 Q-function 都使用一个目标，且给出较小的值会被作为 Q-target
    * 延迟的策略更新(“Delayed” Policy Updates)：以较低的频率更新动作网络，较高频率更新评价网络，通常每更新两次评价网络就更新一次策略
    * 目标策略平滑(Target Policy smoothing) ：TD3 在目标动作中加入噪音，通过平滑 Q 沿动作的变化，使策略更难利用 Q 函数的误差

### 项目三

感觉把[航海士提供的策略梯度项目](https://gitee.com/lovelylin/EasyRL/tree/master/%E7%AD%96%E7%95%A5%E6%A2%AF%E5%BA%A6)的 env 换成 `Pendulum-v0` 环境，然后把 model.forward 和 algorithm.learn 调整一下就行，但是因为我还没学习过 pytorch 和 paddle，所以暂时搁置了，后面再来尝试