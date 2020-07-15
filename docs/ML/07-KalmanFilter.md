# 卡尔曼滤波

## 隐马尔可夫链

简单定义一下隐马尔可夫链：

$$
    \tag{1} \label{HMM}
    \xrightarrow[]{p(\mathbf{x}_0)}(\mathbf{x}_0 \xrightarrow[]{p(\mathbf{z}_0 \vert \mathbf{x}_0)}\mathbf{z}_0)
    \xrightarrow[]{p(\mathbf{x}_1 \vert \mathbf{x}_0)}
    (\mathbf{x}_1 \xrightarrow[]{p(\mathbf{z}_1 \vert \mathbf{x}_1)} \mathbf{z}_1)
    \rightarrow \cdots .
$$

其中 $p(\mathbf{x}_0)$ 表示初始状态概率，$p(\mathbf{x}_{t+1} \vert \mathbf{x}_t)$ 表示隐变量状态转移概率，$p(\mathbf{z}\vert \mathbf{x})$ 表示实变量条件概率。

在隐马尔可夫链中，我们只能观测到变量 $(\mathbf{z}_0 , \mathbf{z}_1,\ldots)$ 的值，而无法获得隐变量的值。

这个模型有非常强的现实意义，通常动力学系统的运动是一个马尔可夫链，而我们无法获得准确的动力学模型的状态 $\mathbf{x}$，只能使用传感器来测量相应状态的一个相关量 $\mathbf{z}$。

## 马尔可夫模型建模

对于随机过程的假设集，我们通常设定为高斯分布：

$$
    \tag{2} \label{hmm_hypothesis}
    \mathcal{H} = \left\{ (p(\mathbf{x}_0) = \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0), p(\mathbf{x}_{t+1} \vert \mathbf{x}_{t}) = \mathcal{N}(\pmb{\mu}_1, \pmb{\Sigma}_1), p(\mathbf{z}\vert\mathbf{x}) = \mathcal{N}(\pmb{\mu}_2, \pmb{\Sigma}_2)) \right\}.
$$

我们希望从假设集中找出一个元组最接近真实的状态转移 $(p_{\mathcal{D}}(\mathbf{x}_0), p_{\mathcal{D}}(\mathbf{x}_{t+1}\vert \mathbf{x}_{t}), p_{\mathcal{D}}(\mathbf{z} \vert \mathbf{x}))$。

采样一条样本 $\tau = (\mathbf{z}_0, \mathbf{z}_1, \ldots, \mathbf{z}_t)$, 那么：

$$
    \tag{3} \label{trueprobability}
    p_{\mathcal{D}}(\tau) = \int p_{\mathcal{D}}(\mathbf{x}_0) p_{\mathcal{D}}(\mathbf{z}_{0}\vert \mathbf{x}_0)p_{\mathcal{D}}(\mathbf{x}_{1}\vert\mathbf{x}_0)\cdots p_{\mathcal{D}}(\mathbf{x}_{t} \vert \mathbf{x}_{t-1}) p_{\mathcal{D}}(\mathbf{z}_t \vert \mathbf{x}_t) \mathrm{d}\mathbf{x}_0\mathrm{d}\mathbf{x}_1\ldots\mathrm{d}\mathbf{x}_t
$$


$$
    \tag{4} \label{modelprobability}
    p_{\theta}(\tau) = \int p_{\theta}(\mathbf{x}_0) p_{\theta}(\mathbf{z}_{0}\vert \mathbf{x}_0)p_{\theta}(\mathbf{x}_{1}\vert\mathbf{x}_0)\cdots p_{\theta}(\mathbf{x}_{t} \vert \mathbf{x}_{t-1}) p_{\theta}(\mathbf{z}_t \vert \mathbf{x}_t) \mathrm{d}\mathbf{x}_0\mathrm{d}\mathbf{x}_1\ldots\mathrm{d}\mathbf{x}_t
$$

定义损失函数为：

$$
    \tag{5} \label{trueloss}
    L_{D}(\theta) = \mathbb{E}_{\tau\sim\mathcal{D}}\left[\ln\left( \frac{p_{\mathcal{D}}(\tau)}{p_{\theta}(\tau)} \right)\right].
$$

对应的经验损失函数为：

$$
    \tag{6} \label{empirical}
    L_{S}(\theta) = -\frac{1}{m} \sum^{m}_{i=1}\ln\left( {p_{\theta}(\tau)} \right).
$$

> 这个函数很难优化，所以需要其他方法，这里超出了我的知识范围，所以先放到一边。

## 卡尔曼滤波

卡尔曼滤波对应的假设集：

$$
    \tag{7} \label{kalman_hypothesis}
    \mathcal{H} = \{ (p(\mathbf{x}_0) = \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0), p(\mathbf{x}_{t+1} \vert \mathbf{x}_{t}) = \mathcal{N}(\mathbf{A}\mathbf{x}_t + \mathbf{b}, \pmb{\Sigma}_1), \\
    p(\mathbf{z}\vert\mathbf{x}) = \mathcal{N}(\mathbf{C} \mathbf{x}_t + \mathbf{d}, \pmb{\Sigma}_2)) \}.
$$

而且我们已经找到了对应的最优模型:

\begin{cases}
    \tag{8} \label{kalman_model}
    p(\mathbf{x}_0) = \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0),\\
    p(\mathbf{x}_{t+1} \vert \mathbf{x}_{t}) = \mathcal{N}(\mathbf{A}\mathbf{x}_t + \mathbf{b}, \mathbf{Q}),\\
    p(\mathbf{z}\vert\mathbf{x}) = \mathcal{N}(\mathbf{C} \mathbf{x} + \mathbf{d}, \mathbf{R}).
\end{cases}

**卡尔曼滤波要解决的问题是** ：给定一条采样 $(\mathbf{z}_0, \mathbf{z}_1, \ldots, \mathbf{z}_t, \ldots)$, 估计分布 $p(\mathbf{x}_t \vert \mathbf{z}_t, \mathbf{z}_{t-1}, \ldots, \mathbf{z}_0)$。
这同样也是对 **滤波** 做的非常优雅的 **数学定义**，即给定一系列''观测''，估计当前系统的''真实状态''。

> 这里的符号确实有点问题，会让人产生巨大的疑问：$\mathbf{z}$ 究竟指的是样本的值 和 还是值背后的随机变量。这里我希望表述成抽象的随机变量。容许我再啰嗦一下。
> HMM模型可以认为定义了一个概率测度空间 
>
> $$
> HMM = \{\tau = (\mathbf{x}_0, \mathbf{z}_0, \mathbf{x}_1, \mathbf{z}_1, \ldots) \sim p(\tau)\},
> $$
>
> 每个元素是一条随机变量的序列，这个序列的概率由初始状态概率分布、隐变量状态转移概率分布和实变量条件概率分布决定。
> 现在我们对这个概率测度空间做聚合简化 
>
> $$
> HMM_t = \{\tau_t = (\mathbf{z}_0, \mathbf{z}_1, \ldots, \mathbf{x}_t, \mathbf{z}_t), \tau_t \sim p(\tau)\}.
> $$
>
> 上面卡尔曼滤波要解决的问题就是求解这个概率测度空间 $HMM_t$ 对应的某个条件概率分布 $p(\mathbf{x}_t \vert \mathbf{z}_t, \mathbf{z}_{t-1}, \ldots, \mathbf{z}_0)$。这里可以用很多监督学习的方法，但是卡尔曼滤波告诉我们可以利用马尔可夫链和高斯分布的性质，用一种更简单的迭代更新的方式来求解这个问题。

首先，我们使用贝叶斯公式可得：

\begin{align*}
    \tag{9} \label{step1}
    &p(\mathbf{x}_t \vert \mathbf{z}_t, \mathbf{z}_{t-1},\ldots, \mathbf{z}_0)\\
    =& \frac{p(\mathbf{z}_t \vert \mathbf{x}_t) p(\mathbf{x}_t \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0)} {\int p(\mathbf{z}_t \vert \mathbf{x}_t) p(\mathbf{x}_t \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0) \mathrm{d}\mathbf{x}_t}.
\end{align*}

首先利用高斯分布的 **共轭性质**，我们可以确定目标条件分布也是高斯分布，我们不妨定为$\mathcal{N}(\pmb{\mu}_t, \pmb{\Sigma}_t)$。我们再来研究这个$\eqref{step1}$式，我们已经有了$p(\mathbf{z}_t \vert \mathbf{x}_t)$，所以我们面对的第一个问题就是求解条件分布 $p(\mathbf{x}_t \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0)$，我们使用概率密度的性质可得

\begin{align*}
    \tag{10} \label{step2}
    & p(\mathbf{x}_t \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0)\\
    =& \int p(\mathbf{x}_t \vert \mathbf{x}_{t-1}) p(\mathbf{x}_{t-1} \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0) \mathrm{d} \mathbf{x}_{t-1}.
\end{align*}

这里我们发现 $p(\mathbf{x}_{t-1} \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0)$ 递归了，我们不妨假设我们已经求出来了这个条件概率服从$\mathcal{N}(\pmb{\mu}_{t-1}, \pmb{\Sigma}_{t-1})$, 我们又已知 $p(\mathbf{x}_{t} \vert \mathbf{x}_{t-1}) \sim \mathcal{N}(\mathbf{A}\mathbf{x}_{t-1}+\mathbf{b}, \mathbf{Q})$，那么使用高斯分布的一个结论，我们可以快速求得$\eqref{step2}$式。

> 多元高斯分布的边缘分布与条件分布：
> 已知分布：
>
> \begin{cases}
>     \tag{11} \label{step3}
>     p(\mathbf{x}) = \mathcal{N}(\pmb{\mu}, \pmb{\Lambda}^{-1}), \\
>     p(\mathbf{y} \vert \mathbf{x}) = \mathcal{N}(\mathbf{A}\mathbf{x} + \mathbf{b}, \pmb{L}^{-1}),
> \end{cases}
> 
> 我们可得：
>
> \begin{cases}
>     \tag{12} \label{step4}
>     p(\mathbf{y}) = \mathcal{N}(\mathbf{A}\pmb{\mu} + \mathbf{b}, \mathbf{L}^{-1} + \mathbf{A}\pmb{\Lambda}^{-1}\mathbf{A}^T), \\
>     p(\mathbf{x} \vert \mathbf{y}) = \mathcal{N}(\pmb{\Sigma}[\mathbf{A}^T \mathbf{L}(\mathbf{y} - \mathbf{b}) + \pmb{\Lambda\mu}], \pmb{\Sigma}),\\
     \pmb{\Sigma} = (\pmb{\Lambda} + \mathbf{A}^T \mathbf{L} \mathbf{A})^{-1}.
> \end{cases}
>
> (参考自：PRML(2006年出版)第93页)

那么我们求解得到 $\eqref{step2}$:

$$
    \tag{13} \label{step5}
    p(\mathbf{x}_t \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0)
    = \mathcal{N}(\mathbf{A}\mu_{t-1} + \mathbf{b}, \mathbf{A} \pmb{\Sigma}_{t-1} \mathbf{A}^T + \mathbf{Q}).
$$

回到求解 $\eqref{step1}$, 我们已知 $p(\mathbf{z} \vert \mathbf{x}) \sim \mathcal{N}(\mathbf{C}\mathbf{x} + \mathbf{d}, \mathbf{R})$, 我们再规定 $p(\mathbf{x}_t \vert \mathbf{z}_{t-1},\ldots, \mathbf{z}_0) = \mathbf{N}(\hat{\pmb{\mu_t}}, \hat{\pmb{\Sigma}_t})$,
那么我们带入公式可得：

$$
    \tag{14} \label{step6}
    p(\mathbf{x}_t \vert \mathbf{z}_t, \mathbf{z}_{t-1}, \ldots, \mathbf{z}_{0})
    = \mathcal{N}(\pmb{\Sigma}[\mathbf{C}^{T}\mathbf{R}^{-1}(\mathbf{z}_t - \mathbf{d}) + \hat{\pmb{\Sigma}}^{-1}_{t}\hat{\pmb{\mu}}_t], \pmb{\Sigma}),
$$

其中 $\pmb{\Sigma} = (\hat{\pmb{\Sigma}}_t^{-1} + \mathbf{C}^T \mathbf{R}^{-1} \mathbf{C})^{-1}$。

综上，我们可以得要一个卡尔曼滤波的公式：起始时 $\pmb{\mu}_0, \pmb{\Sigma}_0$，接着每获得一个观测值 $z_t$ 我们使用下面的迭代公式

\begin{cases}
    \tag{15} \label{kalmanfilter}
    \hat{\pmb{\Sigma}}_t = \mathbf{A} \pmb{\Sigma}_{t-1} \mathbf{A}^T + \mathbf{Q};\\
    \hat{\pmb{\mu}_t} = \mathbf{A} \pmb{\mu}_{t-1} + \mathbf{b}; \\
    \pmb{\Sigma}_t = (\hat{\pmb{\Sigma}}^{-1}_{t} + \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C})^{-1};\\
    \pmb{\mu}_t = \pmb{\Sigma} [\mathbf{C}^{T} \mathbf{R}^{-1} (\mathbf{z}_t - \mathbf{d}) + \hat{\pmb{\Sigma}}^{-1}_{t}\hat{\pmb{\mu}}_{t}]. \\
\end{cases}

这里这个公式是直接带入PRML书中的公式，我们可以化简一下，就成了我们常见的卡尔曼滤波的公式了。


\begin{align*}
    \tag{16} \label{step7}
    &\pmb{\mu}_t = \pmb{\Sigma}_t [\mathbf{C}^{T} \mathbf{R}^{-1} (\mathbf{z}_t - \mathbf{d}) + \hat{\pmb{\Sigma}}^{-1}_{t}\hat{\pmb{\mu}}_{t}] \\
    =& \hat{\pmb{\mu}}_{t} + \pmb{\Sigma}_t [\mathbf{C}^{T} \mathbf{R}^{-1} (\mathbf{z}_t - \mathbf{d}) + \hat{\pmb{\Sigma}}^{-1}_{t}\hat{\pmb{\mu}}_{t} - \pmb{\Sigma}^{-1} \hat{\pmb{\mu}}_t] \\
    =& \hat{\pmb{\mu}}_{t} + \pmb{\Sigma}_t [\mathbf{C}^{T} \mathbf{R}^{-1} (\mathbf{z}_t - \mathbf{d})  - \mathbf{C}^{T} \mathbf{R}^{-1} \mathbf{C} \hat{\pmb{\mu}}_t] \\
    =& \hat{\pmb{\mu}}_{t} + \pmb{\Sigma}_t \mathbf{C}^{T} \mathbf{R}^{-1} [(\mathbf{z}_t - \mathbf{d})  - \mathbf{C} \hat{\pmb{\mu}}_t] \\
\end{align*}

现在还是又多次求逆的操作，我们需要继续简化。

令 $K_t = \pmb{\Sigma}_t \mathbf{C}^T \mathbf{R}^{-1}$, 接着我们化简$K_t$的计算(我也是根据结果凑出来的，卡尔曼太厉害了):

\begin{align*}
    \tag{17} \label{step8}
    & \mathbf{K}_t = \pmb{\Sigma}_t \mathbf{C}^T \mathbf{R}^{-1}\\
    =& (\hat{\pmb{\Sigma}}^{-1}_{t} + \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C})^{-1}\mathbf{C}^T \mathbf{R}^{-1}\\
    =&  (\hat{\pmb{\Sigma}}^{-1}_{t} + \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C})^{-1}\mathbf{C}^T \mathbf{R}^{-1}\\
    & \cdot (\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T +\mathbf{R})(\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T +\mathbf{R})^{-1}\\
    =&  (\hat{\pmb{\Sigma}}^{-1}_{t} + \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C})^{-1}\\
    & \cdot (\mathbf{C}^T \mathbf{R}^{-1}\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T + \mathbf{C}^T)(\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T +\mathbf{R})^{-1}\\
    =&  (\hat{\pmb{\Sigma}}^{-1}_{t} + \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C})^{-1} \cdot (\mathbf{C}^T \mathbf{R}^{-1}\mathbf{C} + \hat{\pmb{\Sigma}}^{-1}_{t})\\
    & \hat{\pmb{\Sigma}}_t \mathbf{C}^T(\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T +\mathbf{R})^{-1}\\
    =& \hat{\pmb{\Sigma}}_t \mathbf{C}^T(\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T +\mathbf{R})^{-1}.
\end{align*}

我们已经获得了 $\mathbf{K}_t$ 我们如何不用矩阵求逆操作就求出 $\pmb{\Sigma}_t$ 呢？

\begin{align*}
    \tag{18} \label{step9}
    &\pmb{\Sigma}_t = \pmb{\Sigma}_t \hat{\pmb{\Sigma}}_t^{-1} \hat{\pmb{\Sigma}}_t \\
    =& \pmb{\Sigma}[\hat{\pmb{\Sigma}}_t^{-1} + \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C} - \mathbf{C}^T \mathbf{R}^{-1}\mathbf{C}) \hat{\pmb{\Sigma}}_t \\
    =& (\mathbf{I} - \pmb{\Sigma}_t \mathbf{C}^T \mathbf{R}^{-1} \mathbf{C}) \hat{\mathbf{\Sigma}}_t \\
    =& (\mathbf{I} - \mathbf{K}_t \mathbf{C}) \hat{\mathbf{\Sigma}}_t.
\end{align*}

综上，我们获得了最传统的卡尔曼滤波的更新公式：

\begin{cases}
    \tag{19} \label{kalmanfilter2}
    \hat{\pmb{\Sigma}}_t = \mathbf{A} \pmb{\Sigma}_{t-1} \mathbf{A}^T + \mathbf{Q};\\
    \hat{\pmb{\mu}_t} = \mathbf{A} \pmb{\mu}_{t-1} + \mathbf{b}; \\
    \pmb{K}_t = \hat{\pmb{\Sigma}}_t \mathbf{C}^T(\mathbf{C}\hat{\pmb{\Sigma}}_t \mathbf{C}^T +\mathbf{R})^{-1};\\
    \pmb{\Sigma}_t = (\mathbf{I} - \mathbf{K}_t \mathbf{C}) \hat{\mathbf{\Sigma}}_t;\\
    \pmb{\mu}_t = \hat{\pmb{\mu}}_{t} + \mathbf{K}_t[(\mathbf{z}_t - \mathbf{d})  - \mathbf{C} \hat{\pmb{\mu}}_t]. \\
\end{cases}






