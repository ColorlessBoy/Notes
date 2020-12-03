# 马尔可夫决策过程(Markov Decision Processes)

马尔科夫链只是砖头，真正强大的是这些砖头搭起来的模型，马尔可夫决策过程。
马尔可夫决策过程被用在很多领域，强化学习只是其中之一。
可以说强化学习只是马尔可夫决策问题(Markov Decision Problem)的一系列求解方法的总和。

## 定义

我们这里介绍一个最基本的马尔可夫决策过程。

首先我们引入两个集合

- 状态(state)集合：$\mathcal{S} = \{s1, s2, \ldots, sn, \ldots\}$;
- 动作(action)集合：$\mathcal{A} = \{a1, a2, \ldots, an, \ldots\}$.

首先，这两个集合的元素$s$, $a$都是向量，这里我偷懒，没有加粗表示。
有了这两个集合，我们可以从数学上定义一个函数，策略(policy)函数：


$$
    \tag{1} \label{policyfuncion}
    \pi:\mathcal{S} \rightarrow \{\Delta(\mathcal{A})\},
$$

其中 $\Delta(\mathcal{A})$ 表示在集合 $\mathcal{A}$ 上的概率分布。
因为这个动作概率分布只和当前的状态有关，所以这个策略函数也是非常特殊的，
我把它叫做马尔可夫策略。

!!!定义
    (马尔可夫决策过程).马尔可夫决策过程是一个函数，将**策略函数空间**映射到一个**马尔科夫链**。
    
    $$
        \tag{2} \label{MDPs}
        \mathcal{MDP}: \{\pi\} \rightarrow \{\mathcal{MC} 
        = \{\mathcal{S} \times \mathcal{A}, p_0[(s_0, a_0)], p[(s_{t+1}, a_{t+1}) \vert (s_t, a_t)] \}\}.
    $$
   
当然，在一些简单问题上，策略函数空间和马尔科夫链都可以张成向量空间，把模型转换为普通数学模型。但是即便是简单问题，这种做法依旧把把很多结构信息给丢掉了。  

值得一提的是，马尔可夫决策过程包含的马尔科夫链的状态集合是 $\mathcal{S} \times \mathcal{A}$，它的状态转移概率满足：

$$
    \tag{3}
    p_\pi [(s_{t+1}, a_{t+1}) \vert (s_{t}, a_{t})] = 
    \pi(a_{t+1} \vert s_{t+1}) p(s_{t+1} \vert s_t, a_t).
$$

到此，我们其实已经定义完了马尔可夫决策过程，但是从广义来看，我们只是走完了一半的路程。
现在我们需要一个 **目标** ，或者说一个 **问题** 。
要获得一个目标或者问题，我们首先要评价一个马尔科夫链的好坏。
有了马尔可夫链的好坏，我们就能评价对应的策略函数的好坏。
最后，我们就能够获得一个问题，如何求最优的策略函数。

首先，我们引入一个最简单的评价标准，马尔可夫奖励函数：

$$
    \tag{4} \label{RewardFunction}
    R(s, a) = \Delta(\mathbb{R}),
$$

这里，奖励值只和马尔可夫链的当前状态有关，每个状态对应一个随机变量。
通常，在其他文献中，这里的随机变量只会取到某一个值，并且要求所有的值都满足有上界 $R_{\max}$，即从一个随机变量退化成一个一维的普通变量。
我们这里把它泛化，要求所有随机变量的期望满足有上界 $R_{\max}$。
奖励函数的含义是：当出现状态 $(s, a)$ 时，奖励函数给予我们这个状态正面评价的程度。

奖励函数把马尔科夫链映射成了的一个样本

$$
    \tag{5}
    \tau = (s_0, a_0, s_1, a_1, \ldots, s_t, a_t, \ldots)
$$

映射成了一个数列

$$
    \tag{6} 
    (r_0 \sim R(s_0, a_0), r_1 \sim R(s_1, a_1), \ldots, r_t(s_t, a_t), \ldots),
$$

并且它是一个隐马尔可夫链。

但是我们如何评价一个数列是好是坏？既然这时一个正面评价数列，那么一个最直觉的想法是：我们希望积累的正面评价越大越好。
我这里直接介绍马尔可夫决策过程中最常用的一个积累函数(accumulated reward function):

$$
    \tag{7}
    V_{\gamma}(r_0, r_1, \ldots, r_t, \ldots) = 
    \lim_{T \rightarrow \infty}
    \frac{\sum_{t = 0}^{T-1} \gamma^t r(s_t, a_t)}{\sum^{T-1}_{t=0} \gamma^t},
$$

其中 $\gamma \in [0, 1]$。当 $\gamma < 1$ 时，我们可以将上式简化为

$$
    \tag{8}
    V_{\gamma}(r_0, r_1, \ldots, r_t, \ldots) = 
    (1 - \gamma) \sum^{\infty}_{t = 0} \gamma^t r(s_t, a_t).
$$


至此，我们最终获得了一个策略函数的评价函数：

$$
    \tag{9}
    \rho(\pi) = \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi), r \sim R} 
    \left\{(1 - \gamma) \sum^{\infty}_{t=0} \gamma^t r(s_t, a_t)\right\}.
$$

!!!定义
    (马尔可夫决策问题, Markov Decision Problem). 对于一个马尔可夫决策过程，我们需要求解它的最优策略:
    
    $$
        \tag{10}
        \pi^* \in {\arg\max}_{\pi} \rho(\pi).
    $$
    
总结一下，一个马尔可夫决策问题可以用下面这个元组表示：

$$
    \tag{11} \label{MDProblems}
    \mathcal{MDP} = \{\mathcal{S}, \mathcal{A}, p_0(s), p(s' \vert s, a), R(s, a), \lambda\}.
$$

这里还要补充两个在强化学习中非常常用的函数。

- 第一个是 **状态价值函数**:

    $$
        \tag{12}
        V_{\pi}(s) = \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi), r \sim R} 
            \left[\sum^{\infty}_{t = 0} \gamma^t r(s_t, a_t) \big\vert s_0 = s\right].
    $$
    
    它可以理解为：马尔科夫链 $\mathcal{MDP}(\pi)$ 中，样本开头 $s_0 = s$ 的子空间（依旧是个马尔科夫链）的奖励期望。

- 第二个是 **状态动作价值函数**:


    $$
        \tag{13}
        Q_{\pi}(s, a) = \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi), r \sim R} 
        \left[\sum^{\infty}_{t = 0} \gamma^t r(s_t, a_t) \big\vert s_0 = s, a_0 = a\right].
    $$

    它可以理解为：马尔科夫链  $\mathcal{MDP}(\pi)$ 中，样本开头 $(s_0, a_0) = (s, a)$ 的子空间（依旧是个马尔科夫链）的奖励期望。

这里需要理解一下：$p_0$ 在 $s$ 上的概率为0，但是这个子空间依旧存在；同样的 $(s, a)$ 开头的样本在 $\mathcal{MDP}(\pi)$ 中的概率可能为0，但是对应的子空间依旧存在。
