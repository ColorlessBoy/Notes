# The Margins Explanation for Boosting’s Effectiveness

这是一篇阅读笔记，关于《Boosting Fundations and algorithms》的第五章。这一章解释了
Margin Theory 以及它在Boosting中的应用。

首先我们需要了解一下Margin Theory最原始的动机，《Understanding Machine Learning》
所涉及的理论误差界分析的是一个二分类问题（$y \in \{-1, 1\}$）的如下的关系：

$$
    \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}}\{ \mathbb{E}_{(\mathbf{x}, y) 
    \sim \mathcal{D}}[\mathbf{1}_{h(\mathbf{x}) \ne y}] \le 
    \mathbb{E}_{(\mathbf{x}, y) \sim \mathcal{S}}[\mathbf{1}_{h(\mathbf{x}) \ne y}] 
    + \epsilon(m, \delta)\} \ge 1 - \delta,
$$

其中 $\mathcal{S}$ 是从 $\mathcal{D}$ 中独立同分布采用了 $m$ 个样本的样本集。上式要求
$h(\mathcal{x}) \in \{-1, 1\}$, 如果 $h(\mathcal{x}) \in [-1, 1]$，那么我们通常使用
$sign(h(\mathcal{x}))$ 来作为最终的二分类器。

但是，如果 $h(\mathcal{x}) \in [-1, 1]$ 那么上面的误差分析就不够精确，因为我们只关心 
$sign(h(\mathbf{x}))$ 是否等于 $y$，或者说 $yh(\mathbf{x})$ 是否小于0。我们回过头再
看：如果 $h(\mathbf{x}) \in [-1, 1]$，那么 $yh(\mathbf{x})$ 本身也反映着分类器 $h$ 
的某种置信程度：如果 $h(\mathbf{x})$ 远远大于0，那么 $h(\mathbf{x})$ 有很大的自信 
$y = 1$。如果我没有理解错，那么 $y h(\mathcal{x})$ 就是某种 **margin**，并且与 margin
有关的分析理论就叫做 **margin theory**。

我们今天用 margin theory 来研究一下 Boosting 的泛化误差，也就是 
$\mathbf{Pr}_{\mathcal{D}}(y h(\mathbf{x}) \le 0)$ 的上界。
当然下面的文章符号会有些许变化，所以我先介绍一下。

对于 Boosting 算法，我们需要一个 Base Hypothesis Set $\mathcal{H}$，而 Boosting 算法
会从 $\mathcal{H}$ 的凸包中选择一个分类器：

$$
Co(\mathcal{H}) = \left\{f(\mathbf{x}) = \sum^T_{t=1} \alpha_t h(\mathcal{x}) \vert  
\alpha_1, \ldots, \alpha_T \ge 0; \sum^T_{t=1} \alpha_t = 1; 
h_1, \ldots, h_T \in \mathcal{H}; T \ge 1\right\}.
$$

## Finite Base Hypothesis Space

就像题目说的，我们首先来研究一下 $\mathcal{H}$ 是有限的情况下，我们能够得到什么。

!!! 定理1
    样本空间 $\mathcal{X} \times \{-1, 1\}$ 服从未知分布 $\mathcal{D}$。样本集
    $\mathcal{S}$ 包含 $m$ 个样本空间采集的样本。如果 $\mathcal{H}$ 是有限的，
    那么存在 $1 - \delta$ 的可能满足：对于任意的 $f \in Co(\mathcal{H})$, 

    $$
       \mathbf{Pr}_{(\mathbf{x}, y) \sim \mathcal{D}} [y f(\mathbf{x}) \le 0]
        \le \mathbf{Pr}_{(\mathbf{x}, y) \sim \mathcal{S}} [y f(\mathbf{x}) \le 
        \theta] + O \left(\sqrt{\frac{\ln \vert \mathcal{H} \vert}{m \theta^{2}} 
        \ln\frac{m\theta^{2}}{\ln \vert \mathcal{H} \vert} 
        + \frac{\ln(1/\delta)}{m}}\right),
    $$

    其中要求 $\theta > \sqrt{\ln \vert \mathcal{H} \vert / (4m)}$。

首先，这个界和 AdaBoost 的轮次 $T$ 没有关系，所以高轮次的 AdaBoost 并不会引起
过拟合。

这个定理的证明思路还是非常巧妙的，我尝试整理一下吧。

证明过程中构造了一个经验分类器集合，它对应着 $Co(\mathcal{H})$：

$$
    \mathcal{A}_{n} = \left\{f(\mathbf{x}) = \frac{1}{n}\sum^{n}_{j=1}h_j(\mathbf{x})
        \vert h_1, \ldots, h_n \in \mathcal{H}\right\}.
$$

对于任意的 $f = \sum^{\vert \mathcal{H} \vert}_{i=1} \alpha_i h_i$, 我们依据权重
$\alpha_i$ 从 $\mathcal{H}$ 中独立同分布采集 $n$ 个分类器，那么我们就可以获得一个
经验分类器 $\hat f = \frac{1}{n} \sum^{n}_{j = 1} h_j$，它是属于 $\mathcal{A}_n$
的，我们定义经验分类器服从的测度空间为 $\mathcal{A}_n(f)$。

不规范的说，主体的证明思路是：

$$
    \mathbf{Pr}_{(\mathbf{x}, y) \sim \mathcal{D}}
        \left[y f(\mathbf{x}) \le 0\right] 
    \le \mathbf{Pr}_{(\mathbf{x}, y) \sim \mathcal{D}}
        \left[y \hat f(\mathbf{x}) \le \frac{\theta}{2}\right] \\
    \le \mathbf{Pr}_{(\mathbf{x}, y) \sim \mathcal{S}}
        \left[y \hat f(\mathbf{x}) \le \frac{\theta}{2}\right]
    \le \mathbf{Pr}_{(\mathbf{x}, y) \sim \mathcal{S}}
        \left[y f(\mathbf{x}) \le \theta\right].
$$

其中 $\hat f$ 起到了很重要的桥梁的作用。

!!! 引理1
    根据霍夫听不等式，我们可得：对于任意的 $x, f$, 以及 $n \ge 1, \theta > 0$，满足

    $$
        \mathbf{Pr}_{\hat f \sim \mathcal{A}_n(f)}
        \left[
            \vert \hat f(x) - f(x) \vert \ge \frac{\theta}{2}
        \right] \le 2 e ^{-n\theta^2/8} = \beta_{n, \theta}.
    $$

自然我们可得一条新的引理

!!!引理2
    对于任意的 $(x, y) \sim \mathcal{D}, f$，以及 $n \ge 1, \theta > 0$，满足

    $$
        \mathbf{Pr}_{(x,y)\sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}
        \left[\vert y f(x) - y \hat f(x) \vert \ge \frac{\theta}{2} \right]
        \le \beta_{n, \theta}.
    $$
    
这条引理也非常简单，因为 $y \in \{-1, 1\}$，以及绝对值符合保证它可以被忽略，接下来就是
使用引理1即可。

那么我们先做第一步放缩：

$$
\begin{aligned}
    &\mathbf{Pr}_{(x, y) \sim \mathcal{D}}[y f(x) \le 0] \\
    =& \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}[y f(x) \le 0]\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}
        \left[y \hat f(x) \le \frac{\theta}{2}\right] \\
    & + \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}
        \left[
            y f(x) \le 0, y \hat f(x) > \frac{\theta}{2}
        \right]\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}
        \left[y \hat f(x) \le \frac{\theta}{2}\right] \\
    & + \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}
        \left[
            \vert y f(x) - y \hat f(x) \vert > \frac{\theta}{2}
        \right]\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f \sim \mathcal{A}_n(f)}
        \left[y \hat f(x) \le \frac{\theta}{2}\right] + \beta_{n, \theta}.
\end{aligned}
$$

类似的，我们可以获得

$$
\begin{aligned}
    &\mathbf{Pr}_{(x, y) \sim \mathcal{S}, \hat f \sim \mathcal{A}_n(f)}
    \left[ y \hat f(x) \le \frac{\theta}{2} \right]\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{S}, \hat f \sim \mathcal{A}_n(f)}
        \left[ y f(x) \le \theta \right]\\
    &+ \mathbf{Pr}_{(x, y) \sim \mathcal{S}, \hat f \sim \mathcal{A}_n(f)}
        \left[
            y \hat f(x) \le \frac{\theta}{2}, y f(x) \ge \theta
        \right] \\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{S}, \hat f \sim \mathcal{A}_n(f)}
        \left[ y f(x) \le \theta \right]\\
    &+ \mathbf{Pr}_{(x, y) \sim \mathcal{S}, \hat f \sim \mathcal{A}_n(f)}
        \left[
            \vert y \hat f(x) - y f(x) \vert \ge \frac{\theta}{2}
        \right] \\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{S}}[y f(x) \le \theta] + \beta_{n, \theta}.
\end{aligned}
$$

我们最后还差一条引理，来连接上面两个结论：

!!!引理3
    定义 $\epsilon_n = \sqrt{\ln[n(n+1)^2 \vert \mathcal{H} \vert^n / \delta]/2m}$,
    其中 $m$ 是 $\mathcal{S}$ 的大小。那么，我们有

    $$
        \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}} \left \{
            \forall n \ge 1, \hat f \in \mathcal{A}_n, \theta>0:
            \mathbf{Pr}_{(x, y) \sim \mathcal{D}}
            \left[
                y \hat f(x) \le \frac{\theta}{2}
            \right]
            \le \mathbf{Pr}_{(x, y) \sim \mathcal{S}}
            \left[
                y \hat f(x) \le \frac{\theta}{2}    
            \right] + \epsilon_n
        \right\} \ge 1 - \delta.
    $$

!!!证明
    设 

    $$
        p_{\hat f, \theta} = 
        \mathbf{Pr}_{(x, y) \sim \mathcal{D}}
        \left[
            y \hat f(x) \le \frac{\theta}{2}
        \right],
    $$

    以及

    $$
        \hat p_{\hat f, \theta} = 
        \mathbf{Pr}_{(x, y) \sim \mathcal{S}}
        \left[
            y \hat f(x) \le \frac{\theta}{2}
        \right].
    $$

    很明显 $\mathbb{E}_{\mathcal{S} \sim \mathcal{D}}[\hat p_{\hat f,\theta}] = p_{\hat f, \theta}$,
    因此我们可以使用霍夫听不等式得到：

    $$
        \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}}
        \left[
            p_{\hat f, \theta} \ge \hat p_{\hat f, \theta} + \epsilon_n
        \right] \le e^{-2\epsilon^2_n m}.
    $$

    接下来的事情就比较巧妙了。
    我们注意到 $\hat f(x) = \frac{1}{n}\sum^n_{j=1}\hat h_j(x)$，所以如果
    $y \hat f(x) \le \frac{\theta}{2}$ 当且仅当 
    $y\sum^n_{j=1}\hat h_j(x) \le \frac{n \theta}{2}$。
    又因为 $h_j(x) \in \{-1, 1\}$，所以还等价于
    $y\sum^n_{j=1}\hat h_j(x) \le \lfloor{n \theta}/{2}\rfloor$。

    又因为 $\theta > 2$ 没必要考虑，所以启发我们定义一个集合

    $$
        \Theta_n = \left\{
            \frac{2i}{n}: i = 0, 1, \ldots, n
        \right\}.
    $$
    
    那么我们有

    $$
    \begin{aligned}
        &\mathbf{Pr}
            \left[
                \exists \hat f \in \mathcal{A}_n, \theta \ge 0: 
                p_{\hat f, \theta} \ge \hat p_{\hat f, \theta} + \epsilon_n
            \right] \\
        =&\mathbf{Pr}
            \left[
                \exists \hat f \in \mathcal{A}_n, \theta \in \Theta_n: 
                p_{\hat f, \theta} \ge \hat p_{\hat f, \theta} + \epsilon_n
            \right] \\
        =& \vert \mathcal{A}_n \vert \cdot \vert \Theta_n \vert \cdot e ^{-2\epsilon^2_n m}\\
        \le& \vert \mathcal{H} \vert^n \cdot (n + 1) \cdot e ^{-2\epsilon^2_n m}\\
        =& \frac{\delta}{n(n+1)}.
    \end{aligned}
    $$

    又因为 $\sum^{\infty}_{n=1} \frac{\delta}{n(n+1)} = \delta$，我们最终证明了定理3。

最终，我们证明了定理1：

$$  
\begin{aligned}
    &\mathbf{Pr}_{(x, y) \sim \mathcal{D}}[y f(x) \le 0]\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{D}, \hat f\sim \mathcal{A}_n(f)}
        \left[
            y \hat f(x) \le \frac{\theta}{2}
        \right] + \beta_{n, \theta}\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{S}, \hat f\sim \mathcal{A}_n(f)}
        \left[
            y \hat f(x) \le \frac{\theta}{2}
        \right] + \epsilon_n + \beta_{n, \theta}\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{S}}
        \left[
            y f(x) \le \theta
        \right] + \epsilon_n + 2\beta_{n, \theta}\\
    \le& \mathbf{Pr}_{(x, y) \sim \mathcal{S}}
        \left[
            y f(x) \le \theta
        \right] + 4e^{-n\theta^2/8} \\
    & + \sqrt{\ln[n(n+1)^2 \vert \mathcal{H} \vert^n / \delta]/2m}.
\end{aligned}
$$

当设定 

$$
    n = 
    \left\lceil 
        \frac{4}{\theta^2} \ln \left(\frac{4m\theta^2}{\ln\vert\mathcal{H}\vert}\right) 
    \right\rceil,
$$

我们就得到了定理1。当然，具体推到还是有点复杂，原书也省略了，我也不仔细追究了。
