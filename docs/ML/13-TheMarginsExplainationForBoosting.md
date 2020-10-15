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

???证明
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

我们就得到了定理1，应该还是需要一些不等式放缩的步骤，这里书上省略了，我也不追究了。
总的来说，这个界对 $n$ 的大小有限制，$m$ 越大， $n$ 可以越大。这个界还是很奇怪的，
我觉得目前的表达方式并没有体现出它的本质：对任意的 $f$ 都有这个界，这个界也太松弛了吧。

## Infinite Base Hypothesis Spaces

我们使用假设集的 VC-维 来做类似的采样复杂度的界，下面的定理非常类似定理1。

!!!定理2
    样本空间 $\mathcal{X} \times \{-1, 1\}$ 服从未知分布 $\mathcal{D}$。样本集
    $\mathcal{S}$ 包含 $m$ 个样本空间采集的样本。如果 $\mathcal{H}$ 的 VC-维 是 $d$。
    对于 $m \ge d \ge 1$ 以及 $\delta > 0$，那么对于任意的 $f \in Co(\mathcal{H})$, 

    $$
        \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}} \left\{
            \forall \theta > \sqrt{8d \ln(em / d) / m}:
        \mathbf{Pr}_{(x, y) \sim \mathcal{D}} [ y f(x) \le 0]
        \le \mathbf{Pr}_{(x, y) \sim \mathcal{S}}[y f(x) \le \theta] + 
        O \left( \sqrt{
            \frac{d \ln(m / d) \ln(m \theta^2 / d)}{m\theta^2}
            + \frac{\ln(1 / \delta)}{m}
        }\right)
        \right\} > 1 - \delta.
    $$

相比于定理1，定理2的证明需要解决的关键问题是如何修改引理3。

!!!引理4
    定义

    $$
        \epsilon_n = \sqrt{\frac{32}{m} [\ln(n(n+1)^2) + dn \ln(em / d) + \ln(8/\delta)]}.
    $$

    我们有：

    $$
        \mathbf{Pr}_{\mathcal{S}\sim \mathcal{D}} \left\{
            \forall n \ge 1, \hat f \in \mathcal{A}_n, \theta \ge 0:
            \mathbf{Pr}_{(x, y) \sim \mathcal{D}}
            \left[y \hat f(x) \le \frac{\theta}{2}\right] 
            \le
            \mathbf{Pr}_{(x, y) \sim \mathcal{S}}
            \left[y \hat f(x) \le \frac{\theta}{2}\right] + \epsilon_n
        \right\} \ge 1 - \delta.
    $$

???证明
    对于样本空间 $\mathcal{Z} = \mathcal{X} \times \{-1, +1\}$，我们构造它的一个子集

    $$
        B_{\hat f, \theta} = \left\{
            (x, y) \in \mathcal{Z}: y \hat f(x) \le \theta/2    
        \right\}
    $$

    来表示样本点满足对应 $\hat f(x)$ 的 margin 最多为 $\theta/2$ 的点的集合。
    我们再定义 $B_{\hat f, \theta}$ 的集合：

    $$
        \mathcal{B}_n = \left\{
            B_{\hat f, \theta}: \hat f \in \mathcal{A}_n, \theta \ge 0    
        \right\}.
    $$

    接下来我们希望求解一下 $\mathcal{B}_n$ 的增长函数 $\Pi_{\mathcal{B}_n}(m)$
    （再我的笔记里，也叫它 $\tau_{\mathcal{B}_n}(m)$）。
    因为假设 $\mathcal{H}$ 的 VC维 是 $d$，所以根据 Sauer's 引理，当 $m \ge d \ge 1$时，
    我们可得：

    $$
        \vert \left\{
            \langle 
                h(x_1), \ldots, h(x_m)
            \rangle: h \in \mathcal{H}
        \right\} \vert \le (em / d)^d.
    $$

    我们可以很快地得到一个非常松弛的界

    $$
        \vert \left\{
            \langle 
                y_1 \hat f(x_1), \ldots, y_m \hat f(x_m)
            \rangle: \hat f \in \mathcal{A}_n
        \right\} \vert \le (em / d)^{dn}.
    $$

    这里因为 $n$ 是确定的，所以 $\hat f(x)$ 也是离散的点，所以我们可以使用类似引理3的
    技术，构造集合 $\Theta_n$, 得到：
    
    $$
        \Pi_{\mathcal{B}_n}(m) \le (n+1) (em/d)^{dn}.
    $$

    VC维有一个定理（原书定理2.6）：如果 $\mathcal{A}$ 是集合 $Z$ 子集的集合，那么有：

    $$
        \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}}
        \left\{
            \exists A \in \mathcal{A}: 
            \mathbf{Pr}_{z \sim \mathcal{D}}[z \in A] 
            \ge \mathbf{Pr}_{z \sim \mathcal{S}}[z \in A]
            + \epsilon
        \right\} \le 8 \Pi_{\mathcal{A}}(m) e^{-m \epsilon^2 / 32}.
    $$

    带入可得：

    $$
        \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}}
        \left\{
            \forall B_{\hat f, \theta} \in \mathcal{B}_n,
            \mathbf{Pr}_{z \sim \mathcal{D}}[z \in B_{\hat f, \theta}]
            \le
            \mathbf{Pr}_{z \sim \mathcal{S}}[z \in B_{\hat f, \theta}]
            + \epsilon_n
        \right\} \ge 1 - \delta/(n(n+1)).
    $$

    那么，我们再聚集一下 $\mathbf{B}_n$，对于任意的 $n \ge 1$，我们都有至少 $1 - \delta$ 
    的概率得到引理4。
    
我们综合上述结论，类似于有限假设集，我们可以得到定理2，不过要设置：

$$
    n = \left\lceil
        \frac{4}{\theta^2} \ln \left(
            \frac{m \theta^2}{8d \ln(em / d)}
        \right)
    \right\rceil.
$$

综上所述，$\tilde O(1 / \sqrt{m})$ 是一个非常松弛的界，如果我们有 consistent 假设，
也就是所有经验误差值可以降到0，那么我们能够获得一个更好的界 $\tilde O(1 / m)$。

## 基于 Rademacher 复杂度的分析

一些 Rademacher 的前置知识可以翻阅前面的笔记。接下来用到了一系列 Rademacher 复杂度的
数学性质。

我们先引入一个函数集合

$$
    \mathcal{M} = \left\{
        (x, y) \mapsto y f(x) \vert f \in co(\mathcal{H})
    \right\}.
$$

受限我们可得： $R_S(\mathcal{M}) = R_S(co(\mathcal{H})) = R_S(\mathcal{H})$。

我们引入一个分段线性函数（$0-1$损失函数的变形）：

$$
\phi(u) = 
\begin{cases}
    1 & u \le 0,\\
    1 - u/\theta & 0\le u \le \theta,\\
    0 & u \ge \theta.
\end{cases}
$$

因为 $\phi(u)$ 是 $1/\theta$-Lipschitz 连续函数，如果假设集 $\mathcal{H}$ 的 VC-维
是 $d$, 我们就能得到：

$$
    R_S(\phi \circ \mathcal{M}) \le \frac{1}{\theta} R_S(\mathcal{M})
    \le \frac{1}{\theta} \sqrt{\frac{2d}{m} \ln(em/d)}.
$$

我们带入到 Rademacher 复杂度的界可以得到：

$$
    \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}} \left\{
        \forall f \in co(\mathcal{H}):
        \mathbb{E}_{(x, y) \sim \mathcal{D}}[\phi(y f(x))]
        - \mathbb{E}_{(x, y) \sim \mathcal{S}}[\phi(y f(x))] \le
        R_S(\phi \circ \mathcal{M}) + 3 \sqrt{\frac{2}{m}\ln(2 / \delta)}
    \right\} \ge 1 - \delta.
$$

因为 $\pmb{1}\{u \le 0\} \le \phi(u) \le \pmb{1}\{u \le \theta\}$，所以我们有

$$
    \mathbf{Pr}_{(x, y) \sim \mathcal{D}}[y f(x) \le 0]
    = \mathbb{E}_{(x, y) \sim \mathcal{D}}[\pmb{1}\{y f(x) \le 0\}]
    \le \mathbb{E}_{(x, y) \sim \mathcal{D}} [\phi(y f(x))].
$$

以及

$$
    \mathbb{E}_{(x, y) \sim \mathcal{S}} [\phi(y f(x))]
    \le \mathbb{E}_{(x, y) \sim \mathcal{S}}[\pmb{1}\{y f(x) \le 0\}]
    = \mathbf{Pr}_{(x, y) \sim \mathcal{S}}[y f(x) \le 0].
$$

因此我们可得：

$$
    \mathbf{Pr}_{\mathcal{S} \sim \mathcal{D}} \left\{
        \forall f \in co(\mathcal{H}):
        \mathbf{Pr}_{(x, y) \sim \mathcal{D}}[y f(x) \le 0]
        - \mathbf{Pr}_{(x, y) \sim \mathcal{S}}[y f(x) \le \theta] \le
        \frac{2}{\theta} \sqrt{\frac{2d}{m} \ln(em/d)} 
        + 3 \sqrt{\frac{2}{m}\ln(2 / \delta)}
    \right\} \ge 1 - \delta.
$$

## Boosting 对 Margin 分布的影响

前面的分析广泛适用于使用假设集 $co(\mathcal{H})$ 的算法。这一节，我们分析一下 Boosting
有没有什么特别之处，总的来说就是通过给小margin样本更大的权重，来显著增大整个训练集的margin。

!!! 定理3
    在 AdaBoost 中定义 $\gamma_t = \frac{1}{2} - \epsilon_t$, 我们可以保证：
    训练集中margin最多为 $\theta$ 的样本个数占总样本个数的比例为：

    $$
        \prod^T_{t=1} \sqrt{(1 + 2 \gamma_t)^{1 + \theta} (1 - 2 \gamma_t)^{1 - \theta}}.
    $$ 

???证明
    我们沿用定义 $f(x) = \frac{\sum^T_{t=1} \alpha_t h_t(x)}{\sum^T_{t=1} \alpha_t}$，
    那么 $y f(x) \le \theta$ 当且仅当 
    $y \sum^T_{t=1} \alpha_t h_t(x) \le \theta \sum^T_{t=1} \alpha_t$，
    进一步当且仅当

    $$
        \exp\left(
        -y \sum^T_{t=1} \alpha_t h_t(x)
        +\theta \sum^T_{t=1} \alpha_t    
        \right) \ge 1.
    $$

    因此

    $$
        \pmb{1} [y f(x) \le \theta] \le
        \exp\left(
        -y \sum^T_{t=1} \alpha_t h_t(x)
        +\theta \sum^T_{t=1} \alpha_t    
        \right).
    $$

    所以，我们可得

    $$
    \begin{aligned}
        &\mathbf{Pr}_{(x, y) \sim S} [y f(x) \le \theta]
        = \frac{1}{m} \sum^m_{i=1} \pmb{1}[y_i f(x_i) \le \theta]\\
        \le& \frac{1}{m}
            \exp\left(
            -y \sum^T_{t=1} \alpha_t h_t(x)
            +\theta \sum^T_{t=1} \alpha_t    
            \right)\\
        =& \frac{1}{m} \exp\left(\theta \sum^T_{t=1} \alpha_t \right)
            \sum^m_{i=1} \exp\left(-y_i \sum^T_{t=1} \alpha_t h_t(x_i)\right)\\
        =& \exp\left(\theta \sum^T_{t=1} \alpha_t \right) \prod^T_{t=1} Z_t.
    \end{aligned}
    $$

    我们回忆一下
    
    $$
    \begin{aligned}
        D_{T+1}(i) 
        &= D_1(i) \times \frac{e^{-y_i \alpha_1 h_1(x_i)}}{Z_1} \times \cdots \times
        \frac{e^{-y_i\alpha_T h_T(x_i)}}{Z_T}\\
        &= D_1(i) \frac{\exp{\left(-y_i \sum^T_{t=1} \alpha_t h_t(x_i)\right)}}{\prod^T_{t=1} Z_t}.
    \end{aligned}
    $$

    所以 
    
    $$
        {\prod^T_{t=1} Z_t} \sum^m_{i=1} D_{T+1}(i) 
        = \sum^m_{i=1} D_1(i) \exp{\left(-y_i \sum^T_{t=1} \alpha_t h_t(x_i)\right)}.
    $$

    又因为 

    $$
        \alpha_t = \frac{1}{2} \ln\left(\frac{1 - \epsilon}{\epsilon}\right)
        = \frac{1}{2}\ln\left(\frac{1 + 2\gamma_t}{1 - 2 \gamma_t}\right),
    $$

    以及

    $$
        Z_t = \sqrt{1 - 4\gamma^2_t},
    $$

    我们可以直接得到定理3。

如果 $\epsilon_t \le \frac{1}{2} - \gamma$, 那么界就变成了
$\left(\sqrt{(1 + 2 \gamma)^{1 + \theta} (1 - 2 \gamma)^{1 - \theta}}\right)^T$。
如果 $\sqrt{(1 + 2 \gamma)^{1 + \theta} (1 - 2 \gamma)^{1 - \theta}} < 1$，
那么随着 AdaBoost 的轮次增加，$yf(x) \le \theta$ 的可能指数下降到0。
我们整理一下得到：

$$
    \theta < \Upsilon(\gamma) = \frac{-\ln(1 - 4\gamma^2)}
    {\ln\left(\frac{1 + 2 \gamma}{ 1 - 2 \gamma}\right)}.
$$

因为 $0 \le \gamma \le \frac{1}{2}$，所以我们得到 $\gamma \le \Upsilon(\gamma) \le 2 \gamma$。
而且，当 $\gamma$ 接近0时，$\Upsilon(\gamma) \approx \gamma$。所以经过一定轮数，
$\Upsilon(\gamma)$ 给了一定的界保障了 margin。 $\gamma_t$ 越大，margin越大。这里通常
将 $\gamma_t$ 称为 edge。这里有点博弈的意味：更强的基础分类器能保证 edge 更大，但是也会
导致基础分类器的复杂度增加，有会对算法的效果有负面影响。
