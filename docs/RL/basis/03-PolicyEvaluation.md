# Policy Evaluation

要解决马尔科夫决策问题，我们首先要解决：给定一个策略函数 $\pi$, 那么对应的奖励值 $\rho(\pi)$ 是多少。
我们把这个问题叫做 **策略估计** (Policy Evaluation)。

我们首先根据 $Q_\pi$ 的定义可以简化 $\rho(\pi)$ 为：

$$
    \tag{1}
    \rho(\pi) = \mathbb{E}_{(s, a) \sim p_0(s) \pi(a \vert s)}[Q_\pi(s, a)].
$$

因此，我们要对策略的好坏进行估计，我们可以转而先求出函数 $Q_\pi(s, a)$。

我们定义 $\bar{r} = \mathbb{E}_{r \sim R} [r]$。接着，我介绍函数 $Q$ 的一个特性：

$$
    \tag{2}
    Q_{\pi} (s, a) =  \bar{r}(s, a) + \gamma \int_{s_1, a_1} p(s_1 \vert s, a) \pi(a_1 \vert s_1) Q_{\pi}(s_1, a_1) \mathrm{d} s_1 \mathrm{d} a_1.   
$$

???证明
    \begin{align*}
        & Q_\pi(s, a) \\
        =& \bar{r}(s, a) + \gamma \int_{s_1, a_1} p(s_1 \vert s, a) \pi(a_1 \vert s_1)\bar{r}(s_1, a_1) \mathrm{d} s_1 \mathrm{d} a_1 \\
        & + \gamma^2 \int_{s_1, a_1}\int_{s_2, a_2} p(s_1 \vert s, a) \pi(a_1 \vert s_1) \\
        & \cdot p(s_2 \vert s_1, a_1) \pi(a_2 \vert s_2) \bar{r}(s_2, a_2) \mathrm{d} s_2 \mathrm{d} a_2 \mathrm{d} s_1 \mathrm{d} a_2 \\
        & \vdots \\
        =& \bar{r}(s, a) + \gamma \int_{s_1, a_1} p(s_1 \vert s, a) \pi(a_1 \vert s_1) Q(s_1, a_1) \mathrm{d} s_1 \mathrm{d} a_1.
    \end{align*}

我们已经非常接近马尔可夫决策过程的重要的概念 **贝尔曼操作**(Bellman Operator) 和 **贝尔曼等式** (Bellman Equation)。

!!!定义
    (贝尔曼操作)。对于任意的函数（或者向量）$Q \in \mathbb{R}^{\vert \mathcal{S} \times \mathcal{A} \vert}$，
    我们可以做如下操作: $\forall (s, a) \in \mathcal{S}\times\mathcal{A}$,

    $$
        \tag{3} 
        T_{\pi} Q (s,a) = \bar{r}(s, a) + \gamma \int_{s', a'} p(s' \vert s, a) \pi(a' \vert s') Q(s', a') \mathrm{d} s' \mathrm{d} a'.
    $$
    
    这里的 $T$ 是某种变换， $T_{\pi}Q$ 对应 $Q$ 经过 $T$ 变换后的新的函数。 

要引出贝尔曼等式，我们需要先了解一个数学定理Banach不动点定理。

!!!定理
    (Banach不动点定理)。如果 $U$ 是一个Banach空间，并且 $T:U\rightarrow U$ 是一个收缩映射，它的收缩因子是 $\gamma \in (0, 1)$。那么

    - 在空间 $U$ 中，存在唯一的的不动点 $\mathbf{v}^*$ 满足 $T\mathbf{v}^* = \mathbf{v}^{*}$;
    - 在空间 $U$ 中，对于任意的点 $\mathbf{v}^0$, 以及递推公式 $\mathbf{v}^{n+1} = T\mathbf{v}^{n}$，那么 $\lim_{n \rightarrow \infty} \mathbf{v}^{n} = \lim_{n \rightarrow \infty} T^{n} \mathbf{v}^0 = \mathbf{v}^{*}.$. 也就是说，任意点经过多次T的收缩映射，我们能够收敛到不动点 $\mathbf{v}^*$.

    这里收缩映射的意思是：对于任意的 $\mathbf{v}, \mathbf{u} \in U$，满足 $\Vert T\mathbf{v} - T\mathbf{u} \Vert \le \gamma \Vert \mathbf{v} - \mathbf{u} \Vert$。

???证明

    首先证明 $\{\mathbf{v}_n\}$ 是一个柯西序列：

    \begin{align*}
        \forall m \ge 1,\quad &\Arrowvert \mathbf{v}^{n+m} - \mathbf{v}^n \Arrowvert\\
        \le& \sum^{m-1}_{k=0} \Arrowvert \mathbf{v}^{n+k+1} - \mathbf{v}^{n+k} \Arrowvert\\
        =& \sum^{m-1}_{k=0} \Arrowvert T^{n+k} \mathbf{v} ^1 - T^{n+k} \mathbf{v}^{0} \Arrowvert\\
        \le& \sum^{m-1}_{k=0} \lambda^{n+k} \Arrowvert \mathbf{v}^1 - \mathbf{v}^{0} \Arrowvert\\
        =& \frac{\lambda^n (1-\lambda^m) }{(1-\lambda)} \Arrowvert \mathbf{v}^1 - \mathbf{v}^0 \Arrowvert.
    \end{align*}

    又因为空间 $U$ 是完全集，所以 $\mathbf{v}^\infty$ 也在空间 $U$ 中。

    接着，我们证明 $\mathbf{v}^{\infty}$ 是收缩映射 $T$ 的固定点。

    \begin{align*}
        0 \le& \Arrowvert T \mathbf{v}^\infty - \mathbf{v}^\infty \Arrowvert \\
        \le& \Arrowvert T \mathbf{v}^\infty - \mathbf{v}^n \Arrowvert + \Arrowvert \mathbf{v}^n - \mathbf{v}^\infty \Arrowvert \\
        =& \Arrowvert T\mathbf{v}^\infty - T\mathbf{v}^{n-1} \Arrowvert + \Arrowvert \mathbf{v}^n - \mathbf{v}^\infty \Arrowvert\\
        \le& \lambda \Arrowvert \mathbf{v}^\infty - \mathbf{v}^{n-1} \Arrowvert + \Arrowvert \mathbf{v}^n - \mathbf{v}^\infty \Arrowvert
        \overset{n \rightarrow \infty}{\longrightarrow} 0.
    \end{align*}
    
    最后，我们证明收缩映射的不动点唯一。我们假设收缩映射存在两个不同的不动点 $\mathbf{u}$ 和 $\mathbf{v}$，那么我们可得：
    
    $$
        \Arrowvert \mathbf{u}^* - \mathbf{v}^* \Arrowvert = \Arrowvert T\mathbf{u}^* - T\mathbf{v}^* \Arrowvert
        \le \lambda \Arrowvert \mathbf{u}^* - \mathbf{v}^* \Arrowvert
        \Rightarrow \mathbf{u}^* = \mathbf{v}^*.
    $$
    
这下，我们终于可以介绍马尔可夫决策过程中的非常重要的等式，**贝尔曼等式**:

!!!定理
    (贝尔曼等式)。$Q_\pi$ 是贝尔曼变换唯一的不动点：对于任意的$(s, a)$满足

    $$
        \tag{4}
        Q_\pi(s, a) = T_{\pi}Q_{\pi} (s, a).
    $$

??? 证明
    这里，我们只需要证明贝尔曼操作是收缩映射。
    
    \begin{align*}
        &\sum^{}_{s, a}\Vert T_{\pi} Q_1 (s,a) - T_{\pi} Q_2(s,a) \Vert \\
        =& \gamma \Big\Vert \int_{s_1, a_1} p(s_1 \vert s, a) \pi(a_1 \vert s_1) [Q_1(s_1, a_1) - Q_2(s_1, a_1)]\mathrm{d} s_1 \mathrm{d} a_1 \Big\Vert.
    \end{align*}
    
    这里如果写成矩阵的形式，我们从马尔可夫链的状态转移矩阵 $\mathbf{P}_\pi$ 的特征值不大于1可得
    
    $$
        \Vert T_{\pi} \mathbf{Q}_1 - T_{\pi} \mathbf{Q}_2 \Vert
        = \gamma \Vert \mathbf{P}^T_{\pi} (\mathbf{Q}_1 - \mathbf{Q}_2 \Vert
        \le \gamma \Vert \mathbf{Q}_1 - \mathbf{Q}_2 \Vert.
    $$
    
根据Banach不动点理论，我们可以够造出两种算法来进行策略估计：

- 迭代法：$Q_{t+1}(s, a) = \alpha Q_{t}(s, a) + (1 - \alpha) T_\pi Q_{t} (s, a)$, 这里要求 $\alpha < 1$;
- 构造损失函数: $L(Q) = \Vert Q - TQ \Vert^2_p$， 然后优化这个损失函数。(这里的 $p$ 泛指范数，不局限于二范数。)

这两种方法在优化领域也有非常有意思的讨论。

!!!example
    梯度下降法(GD)是最基础的一个优化算法，它的更新策略是

    \begin{equation*}
        x_{t+1} = x_{t} - \alpha \nabla_x f(x).
    \end{equation*}

    这个算法在解决下面这个凸优化问题非常有效

    \begin{equation*}
        \min f(x).
    \end{equation*}

    我们可以从不动点的角度来理解这个算法：

    \begin{equation*}
        x_{t+1} = T x_{t}, \quad
        GD(x) = x - \frac{1}{L} \nabla_x f(x).
    \end{equation*}

    如果 $f(x)$ 是 $L$-smooth 的话， 映射函数 $GD$ 是一个非扩张映射。

    那么原始的梯度下降算法就是一个不动点迭代算法

    \begin{equation*}
        x_{t+1} = \alpha x_t + (1 - \alpha) GD(x).
    \end{equation*}

    当然，我们也可以构造一个新的损失函数

    \begin{equation*}
        L(x) = \frac{1}{2} \Vert GD(x) - x \Vert^2_2
        = \frac{1}{2} \left\Vert \frac{1}{L} \nabla_x f(x) \right\Vert^2_2.
    \end{equation*}

    我们对这个损失函数进行梯度下降优化，我们套娃式地获得了一个新的优化算法，它地更新规则是

    \begin{equation*}
        x_{t+1} = x_t - \frac{\alpha}{L} \nabla^2_{xx} f(x_t) \nabla_x f(x_t).
    \end{equation*}

从上面这个例子可以看出来，在优化领域，不动点迭代算法非常流行，但是构造损失函数的方法看起来是在套娃，还要计算二阶梯度，增加了计算的复杂程度。
但是，在强化学习领域，我们会看到，构造损失函数法发挥着非常重要的作用。
