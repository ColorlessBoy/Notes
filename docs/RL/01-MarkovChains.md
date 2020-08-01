# 马尔科夫链

马尔可夫链是一个非常流行的随机过程中的数学模型。
它非常简洁、符合直觉性以及足够强大，让人一度觉得它可以描述整个世界的事件。
这里大家应该能想到当年经典力学发展到顶峰的时候，人们一度以为：
只要对现有世界建模够准，就能够精准地预测未来。
正如经典力学地局限性，马尔科夫链也不是一个万能的随机过程模型。
但是，我们人类的大脑是怎么建模这个世界的呢？
如果我们的大脑就是把这个世界简化成马尔可夫链呢？
那么是不是意味着智能足够在马尔可夫模型建模得世界上产生？
这个假设鼓励了我们对强化学习的研究,我相信强化学习是人工智能的重要一环。

## 定义

我们目前只涉及离散的随机过程，时间被分割成自然数序列 $[0, 1, 2, \ldots]$，
每个时间点有一个随机变量 $S_t$, 那么因为时间维度的特殊性，
这就形成了一个过程 $(S_0, S_1, S_2, \ldots)$。
这个随机过程的样本空间是一个概率空间，即每个样本服从某个概率分布 $(s_0, s_1, s_2, \ldots) \sim p$。
我们当然可以把每个样本张成一个维度非常高的向量，退化成某种监督学习或者无监督学习的问题。
但是，当我们先验性地给数据赋予 **过程** 这种先验结构信息，我们也就增加了更多的操作空间。
**过程** 就是某种概率图，概率逐层传递，即我们认为 $S_t$ 的随机变量由 $(S_0, S_1, \ldots, S_{t-1})$ 决定：

$$
    \tag{1}
    p(S_t) = p(S_t \vert S_{t-1} = s_{t-1}， S_{t-2} = s_{t-2}, \ldots, S_{0} = s_{0}).
$$

而马尔可夫链就是更进一步简化版的随机过程。

!!!定义
    (**马尔科夫链**). 离散随机过程叫做马尔可夫链，表示：

    $$
        \tag{2}
        p(S_t \vert S_{t-1} = s_{t-1}， S_{t-2} = s_{t-2}, \ldots, S_{0} = s_{0}) 
        = p(S_t \vert S_{t-1} = s_{t-1}).
    $$

    也就是说，当前时刻的随机变量只与前一时刻的随机变量有关。

我们假设 $S_t$ 随机变量都是在集合 $\mathcal{S}$ 上，那么马尔可夫链就是一个测度空间，

$$
    \tag{3} 
    \mathcal{MC} = \{\Omega = \{\tau = (s_0, s_1, \ldots, s_t, \ldots) \vert s_t \in \mathcal{S}\}, \sigma(\Omega), \mu\},
$$

其中 $\mu(\tau) = p_0(s_0) \prod^{\infty}_{t = 0}p(s_{t+1} \vert s_t)$。 

所以，表示一个马尔科夫链可以简化为

$$
    \tag{4}
    \mathcal{MC} = (\mathcal{S}, p_0, p).
$$

如果 $\mathcal{S}$ 是可数的，那么我们构造一个状态转移矩阵$\mathbf{P}$:


\begin{equation*}
    \tag{5}
    \mathbf{P} =
    \begin{bmatrix}
        p(s1 \vert s1) & p(s2 \vert s1) & \cdots & p(sn \vert s1) & \cdots \\
        p(s1 \vert s2) & p(s2 \vert s2) & \cdots & p(sn \vert s2) & \cdots \\
        \vdots & \vdots & \ddots & \vdots & \ddots \\
        p(sn \vert sn) & p(s2 \vert sn) & \cdots & p(sn \vert sn) & \cdots \\
        \vdots & \vdots & \ddots & \vdots & \ddots
    \end{bmatrix},
\end{equation*}

注意 $\mathbf{P} \pmb{1} = \pmb{1}$。

## 稳定分布(Stationary Distribution)

在这一节，我们分享一个马尔科夫链中的重要结论。

!!! 定理
    (**Perron-Frobenius theorem**)
    If a matrix $\mathbf{A}$ is irreducible[^1], then

    - $\mathbf{A}$ has a positive real eigenvalue $\lambda_{\max}$
        such that all other eigenvalues of $\mathbf{A}$ satisfy
        $\vert \lambda \vert < \lambda_{\max}$.
    - Furthermore, $\lambda_{\max}$ has algebraic[^2]
        and geometric[^3] multiplicity one, and has an eigenvector $\mathbf{x}$ with $\mathbf{x} \succ 0$.
    - Any nonnegative eigenvector is a multiper of $\mathbf{x}$.

[^1]: $\mathbf{A}$ is called irreducible if for any i, j there is $k_{ij}$ such that $(\mathbf{A}^k)_{ij} > 0$.
[^2]: Algebraic multiplicity is the number of times an eigenvalue appears in a characteristic polynomial of a matrix.
[^3]: Geometric multiplicity is the nullity of $\mathbf{I} - \lambda \mathbf{A}$.

因为状态转移矩阵 $\mathbf{P} \mathbf{1} = \mathbf{1}$。
如果马尔科夫链是 $irreducible$ 以及 $aperiodical$的（满足上面定理的要求），那么
 1 是 $\mathbf{P}$ 的特征值。

接下来我们要证明它的最大特征值是1。
首先它的特征值 $\lambda$ 满足

$$
    \mathbf{P} \mathbf{z} = \lambda \mathbf{z},
$$

其中 $z_k$ 是 $\mathbf{z}$ 中的最大分量。

那么

$$
    \sum^{}_{j} p_{kj} z_j = \lambda z_k,
$$

所以

$$
    \vert \lambda z_k \vert = \vert \sum^{}_{j} p_{kj} z_j \vert \le \sum^{}_{j} p_{kj} \vert z_j \vert \le \sum^{}_{j} p_{kj} \vert z_k \vert \le \vert z_k \vert,
$$

所以 $\vert \lambda \vert \le 1$。

因为 $\mathbf{P}^T$ 的特征值和 $\mathbf{P}$ 相同, 所以 $\mathbf{P}^T$ 的最大特征值是1, 它也存在唯一的特征值 $\mathbf{\pi}$ 满足 $\mathbf{P}^T \mathbf{\pi} = \mathbf{\pi}$。

我们称 $\mathbf{\pi}$ 是马尔可夫链的**稳定分布**。我们为什么叫它是稳定分布呢？
因为对于任意的初始分布 $\mathbf{p}_0$, 我们定义$\mathbf{p}_{\infty} = \lim_{k \rightarrow \infty} (\mathbf{P}^T)^k \mathbf{p}_0$, 那么
$\mathbf{p}_{\infty} = \mathbf{P}^T \mathbf{p}_{\infty}$。
因为特征值的唯一性，我们可以得到 $\mathbf{p}_{\infty} = \mathbf{\pi}$。
