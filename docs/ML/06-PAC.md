# PAC: Principal Component Analysis

## 问题描述

在集合 $\mathbb{R}^{d}$ 存在未知分布 $\mathcal{D}$ , 它的样本用符号 $\mathbf{x}$ 表示。
我们希望找到一个在 $\mathbb{R}^{k}$ 上的分布 $p(\mathbf{y})$ 最接近分布 $p_{\mathcal{D}}(\mathbf{x})$，
这里的最接近，指的是 $p(\mathbf{y})$ 包含最多 $p_{\mathcal{D}}(\mathbf{x})$ 的 **信息**。

什么叫做保留最多的信息呢。更确切地说，我们希望找到一个编码器 $E:\mathbf{x} \rightarrow \mathbf{y}$ (其中 $y \in \mathbb{R}^k$ 以及 $k < d$),
以及一个解码器 $D:\mathbf{y} \rightarrow \mathbf{x}$, 使得 $D(E(\mathbf{x}))$ 最接近 $p_{\mathcal{D}}(\mathbf{x})$:


$$
    \tag{1} \label{loss_1}
    \min_{E, D} \left\{ Distance(p_{\mathcal{D}}(\mathbf{x}) \Vert p(\hat{\mathbf{x}} = E(D(\mathbf{x}))) \right\}
$$

这个问题描述还不是很合适，我需要再思考一下。

## PAC

对于样本 $\mathbf{x} \in \mathbb{R}^d$, 服从未知分布 $\mathcal{D}$，
PAC的优化目标是

$$
    \tag{2} \label{pac_true_loss}
    \min_{\mathbf{U}, \mathbf{V}} L_{\mathcal{D}}(\mathbf{U}, \mathbf{V}) 
    = \mathbb{E}_{\mathbf{x}\sim\mathcal{D}}
    \left\{\Vert \mathbf{x} - \mathbf{U}\mathbf{V}\mathbf{x} \Vert^2_2 \right\},
$$

其中 $\mathbf{U} \in \mathbb{R}^{d\times k}$, $\mathbf{V} \in \mathbb{R}^{k \times d}$ 并且 $k \le d$。

给定样本集 $S = \{\mathbf{x}_1, \mathbf{x}_2, \ldots, \mathbf{x}_m\}$，对应的经验误差函数为

$$
    \tag{3} \label{pac_empirical_loss}
    L_{S}(\mathbf{U}, \mathbf{V}) 
    = \frac{1}{m} \sum^{m}_{i=1}
    \Vert \mathbf{x}_i - \mathbf{U}\mathbf{V}\mathbf{x}_i \Vert^2_2.
$$

PAC的主要工作就是求解上面这个优化问题$\eqref{pac_empirical_loss}$，接下来的内容就是讨论这个问题的求解方法。

首先，我们定义一个矩阵 $\mathbf{E} \in \mathbb{R}^{d \times k}$，并且 $\mathbf{E}^T \mathbf{E} = \mathbf{I}^k$，
那么对于任意的矩阵 $\mathbf{U}$, 满足 $\{\mathbf{U} \mathbf{V} \mathbf{x}: \mathbf{x} \in S\} \subset \{\mathbf{E} \mathbf{y}: \mathbf{y} \in \mathbb{R}^{k}\}$。
并且我们注意到，对于任意的向量 $\mathbf{x}$, $\min_{\mathbf{y}} \Vert \mathbf{E}\mathbf{y} - \mathbf{x} \Vert^2_2$ 的最优解为 $\mathbf{y}^* = \mathbf{E}^T \mathbf{x}$。
综合上面这两条信息，我们可以得到

\begin{align*}
    &\min_{\mathbf{U}, \mathbf{V}} L_{S}(\mathbf{U}, \mathbf{V})\\
    \ge&
    \min_{\mathbf{E}} \frac{1}{m} \sum^{m}_{i=1} \min_{\mathbf{y}_i}
    \Vert \mathbf{x}_i - \mathbf{E}\mathbf{y}_i \Vert^2_2\\
    =& \min_{\mathbf{E}} \frac{1}{m} \sum^{m}_{i=1} 
    \Vert \mathbf{x}_i - \mathbf{E}\mathbf{E}^T\mathbf{x}_i \Vert^2_2\\
    =& \min_{\mathbf{E}} \frac{1}{m} \sum^{m}_{i=1} 
    (2 \Vert \mathbf{x}_i \Vert^2_2 - 2 \mathbf{x}^T_i \mathbf{E}\mathbf{E}^{T}\mathbf{x}_i)\\
    =& \min_{\mathbf{E}} \frac{1}{m} \sum^{m}_{i=1} 
    [2 \Vert \mathbf{x}_i \Vert^2_2 - 2 trace(\mathbf{E}^{T}\mathbf{x}_i\mathbf{x}^T_i\mathbf{E})]\\
    =& \min_{\mathbf{E}} \frac{2}{m} \sum^{m}_{i=1} 
    \Vert \mathbf{x}_i \Vert^2_2 - \frac{2}{m} trace\left(\mathbf{E}^{T} \sum^{m}_{i=1} \mathbf{x}_i\mathbf{x}^T_i\mathbf{E}\right).\\
    \tag{4} \label{proof_step1}
\end{align*}

因为 $\sum^{m}_{i=1} \mathbf{x}_i\mathbf{x}^T_i$ 是实对称矩阵，所以能够正交分解为 $\mathbf{F} \mathbf{D} \mathbf{F}^T$ 三个矩阵的乘积，其中 $\mathbf{F}^T \mathbf{F} = \mathbf{I}^d$ 以及 $\mathbf{D}$ 是对角矩阵，并且 $D_{11} \ge D_{22} \ge \ldots \ge D_{mm}$。所以原问题等价于优化下面这个问题：

\begin{align*}
    &\max_{\mathbf{E}, \mathbf{F}} trace\left( \mathbf{E}^{T}\mathbf{F} \mathbf{D} \mathbf{F}^{T} \mathbf{E} \right)\\
    =&
    \max_{\mathbf{H}: \mathbf{H}^T \mathbf{H} = \mathbf{I}^k} trace\left( \mathbf{H}^T \mathbf{D} \mathbf{H} \right)\\
    =&
    \max_{\mathbf{H}: \mathbf{H}^T \mathbf{H} = \mathbf{I}^k}
    \sum^{d}_{i=1} {D}_{ii} \sum^{k}_{j=1} {H}^{2}_{ij}.
    \tag{5} \label{proof_step2}
\end{align*}

接下来是一个矩阵的性质，我们可以扩展矩阵 $\mathbf{H}$ 的列，形成新的矩阵 $\tilde{\mathbf{H}}$, 满足 $\tilde{\mathbf{H}}^T \tilde{\mathbf{H}} = \mathbf{I}^d$。那么 $\tilde{\mathbf{H}} \tilde{\mathbf{H}}^T = \mathbf{I}^d$, 即 $\sum^{d}_{j=1} \tilde{H}^2_{ij} = 1 \Rightarrow \sum^{k}_{j=1} H^2_{ij} \le 1$。再有，$\sum^{d}_{i=1}\sum^{k}_{j=1} H_{ij} = k$。 综上$\eqref{proof_step2}$式还可以继续转化为

$$
    \tag{6} \label{proof_step3}
    \eqref{proof_step2} \le
    \max_{\pmb{0} \preceq \pmb{\beta} \preceq \pmb{1}, \Vert \pmb{\beta} \Vert_1 = k}
    \sum^{d}_{i=1} D_{ii} \beta_{i} = \sum^{k}_{i=1} D_{ii}.
$$

当 $\mathbf{E}$ 由 $\mathbf{F}$ 前 $k$ 列组成，那么

$$
    \tag{7} \label{proof_step4}
    trace\left( \mathbf{E}^{T}\mathbf{F} \mathbf{D} \mathbf{F}^{T} \mathbf{E} \right) = \sum^{k}_{i=1} D_{ii}，
$$

即问题$\eqref{proof_step2}$的最值为 $\sum^{k}_{i=1} D_{ii}$。

最后，我们发现 $\mathbf{E}$ 恰好是矩阵 $\sum^{m}_{i=1}\mathbf{x}^{T}_{i}\mathbf{x}_{i}$ 的最大$k$个特征值对应的特征向量。特征值大，在 $\mathbf{F} \mathbf{D} \mathbf{F}^T$ 中影响也大，即对应的部分是原矩阵的主成分。所以我们也叫这种方法为 **主成分分析** 。
