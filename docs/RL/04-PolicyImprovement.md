# Policy Improvement

我们估计出策略的好坏还远远不够，我们需要的是不断优化策略，最终找到最优的策略。
但是，我们该往哪个方向来改变策略，才能使策略越来越好呢？

强化学习里存在两种方法来优化策略：

- 把策略函数 $\pi$ 的评价函数 $\rho(\pi)$ 看成一个非凸函数，我们通过优化的方法直接优化这个函数；
- 我们不直接求解最优的策略函数，我们转而去求解最优策略函数对应的最优 $Q$ 函数。

两种方法就分成了强化学习算法的两个种类，一种是 **policy-based** 的算法，另一种是 **value-based** 的算法。大家会很奇怪：我们怎么做到直接求解最优策略函数对应的最优 $Q$ 函数呢？我们接下来就逐渐解开这个疑问。

首先介绍MDPs中非常重要的算子 **最优贝尔曼操作** (Optimal Bellman Operator)。

!!!定义
    (最优贝尔曼操作)。给定任意的函数（或者叫做向量）$Q \in \mathbb{R}^{\vert \mathcal{S} \times \mathcal{A} \vert}$, 我们可以对它做如下操作：$\forall (s, a) \in \mathcal{S} \times \mathcal{A}$，
    
    
    $$
        \tag{1} 
        T Q (s, a) = \max_{\pi} T_{\pi} Q (s, a) = \max_{\pi} \bar{r}(s, a) + \int_{s', a'} p(s' \vert s, a) \pi(a' \vert s') Q(s', a') ds' da'.
    $$
    
!!! 定理
    最优贝尔曼操作是一个 $\gamma$-收缩映射。

这里我们提供一个不是很严格的证明。

!!! 证明
    设 $\pi_1 \in {\arg\max}_{\pi} T_{\pi} Q_1$ 以及 $\pi_2 \in {\arg\max}_{\pi} T_{\pi} Q_2$。
    
    \begin{align*}
        &TQ_1 - TQ_2 \\
        =& T_{\pi_1} Q_1 - T_{\pi_2} Q_2 \\
        \preceq& T_{\pi_1} Q_1 - T_{\pi_1} Q_2 \\
        =& \gamma \int_{s', a'} p(s' \vert s, a) \pi_1(a' \vert s') [Q_1(s', a') - Q_2(s', a')] ds' da'.
    \end{align*}
    
    类似的，我们可以得到 $TQ_2 - TQ_1 \preceq \gamma \int_{s', a'} p(s' \vert s, a) \pi_2(a' \vert s') [Q_2(s', a') - Q_1(s', a')] ds' da'.$

    \begin{align*}
        &\Vert TQ_1 - TQ_2 \Vert\\
        \le& \min\{\gamma \Vert \int_{s', a'} p(s' \vert s, a) \pi_1(a' \vert s') [Q_1(s', a') - Q_2(s', a')] ds' da' \Vert, \\
            &\gamma \Vert \int_{s', a'} p(s' \vert s, a) \pi_2(a' \vert s') [Q_2(s', a') - Q_1(s', a')] ds' da'\Vert\} \\
        \le& \gamma \Vert Q_1 - Q_2 \Vert.
    \end{align*}
    
根据Banach不动点定理，我们可以得到最优贝尔曼操作存在唯一的不动点 $Q^*$ 满足：

$$
    \tag{2}
    Q^* = T Q^*.
$$

我们也叫上面的等式为 **最优贝尔曼等式** (Optimal Bellman Equation)。

!!!定理
    设 $\pi^* = {\arg\max}_{\pi} \rho(\pi)$, 

    $$
       \tag{3}
       Q^* = Q_{\pi^*}.
    $$
            
接下来，我们要证明这个我认为是MDPs中最为重要的定理。这个证明非常巧妙，换句话说不是那么直观，大家要注意了。

!!!证明
    我们的证明分为两步：

    - 当 $Q \succeq TQ$ 时， $Q \succeq Q_{\pi^*}$;
    - 当 $Q \preceq TQ$ 时， $Q \preceq Q_{\pi^*}$.

    在证明中，我们将使用矩阵形式的最优贝尔曼操作 $T \mathbf{Q} = {\arg\max}_{\pi} \bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi} \mathbf{Q}$。 

    首先是第一步：对于任意的 $\pi$, 我们可得：

    \begin{align*}
        \mathbf{Q} \succeq& T(\mathbf{Q}) 
        \succeq \bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi}\mathbf{Q} \\
        \succeq& \bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi}
                (\bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi}\mathbf{Q})\\
        \vdots& \\
        \succeq& \lim_{K \rightarrow \infty} \sum^{K}_{t=0}(\gamma \mathbf{P}^T_{\pi})^t \bar{\mathbf{r}}
                + (\gamma \mathbf{P}^T_{\pi})^{K+1} \mathbf{Q} \\
        =& \mathbf{Q}_{\pi}.
    \end{align*}
    
    因为上式对任意的 $\pi$ 都成立，所以也对 $\pi^*$ 成立，即 $Q \succeq Q_{\pi^*}$。

    接着是第二步，我们设 $\pi_Q = {\arg\max}_{\pi} \bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi}\mathbf{Q}$。
    那么，我们有：
    
    \begin{align*}
        \mathbf{Q} \preceq& T(\mathbf{Q}) 
        = \bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi_Q}\mathbf{Q} \\
        \preceq & \bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi_Q}
                (\bar{\mathbf{r}} + \gamma \mathbf{P}^T_{\pi_Q}\mathbf{Q})\\
        \vdots& \\
        \preceq & \lim_{K \rightarrow \infty}\sum^{K}_{t=0}(\gamma \mathbf{P}^T_{\pi_Q})^t \bar{\mathbf{r}}
                + (\gamma \mathbf{P}^T_{\pi_Q})^{K+1} \mathbf{Q} = \mathbf{Q}_{\pi_Q}\\
        \preceq& \mathbf{Q}_{\pi^*}.
    \end{align*}
    
    综合上面两步，我们可得: 当 $Q = T(Q)$ 时，$Q = Q_{\pi^*}$。
    又因为只有点 $Q^*$ 满足 $Q = T(Q)$ 所以我们有 $Q^* = Q_{\pi^*}$。
    
至此，我们将求解最优策略的问题转化为了求解最优贝尔曼等式的问题。
类似的，我们有两种方法来求解这个问题：

- 不动点迭代算法：$Q_{t+1} = (1 - \alpha) Q_{t} + TQ_{t}$;
- 构造损失函数法：$L(Q) = \Vert TQ - Q \Vert^2_2$。
    
