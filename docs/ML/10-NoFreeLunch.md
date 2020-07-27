# No Free Lunch

没有免费午餐是机器学习里的一个非常经典的结论，那么，它究竟指的是什么呢？
我们稍微严谨地讨论一下。
首先我们思考一个问题：有没有一个万能的学习方法，对所有同类型的问题都能解的最好呢？
我们在解决一个机器学习问题的时候， 我们需要通过一个先验知识构造一个假设集，然后从假设集里挑出一个最接近真实目标分布的近似分布来。那么，如果这个万能的学习方法存在，则意味着这个假设集非常大，包含所有可能的分布。

上面这段话比较抽象，我们用一个例子来讲解：现在我们要解决的是一个二分类问题，$\mathbf{x}\in\mathbb{R}^d$ 并且 $y \in \{0, 1\}$。$P(y \vert \mathbf{x})$ 对应某个未知的分布 $\mathcal{D}$。那么，是否存在一个万能的学习方法，无论 $\mathcal{D}$ 是什么，这个方法是概率渐进正确(PAC)的。

**没有免费午餐** 定理告诉我们，这是不可能的，样本再多也不可能。

定理的证明，且不说 $\mathcal{D}$ 是任意的分布，$P_{\mathcal{D}}(y \vert \mathbf{x})$ 是deterministic分布集合中的一个，也是不可能的。

> deterministic 分布指的是 $P_{\mathcal{D}}(y = 1 \vert \mathbf{x}) = 1$ 或者 $P_{\mathcal{D}}(y = 0 \vert \mathbf{x}) = 1$。

我们限定所面对的二分类机器学习问题中，$P_{\mathcal{D}}(y \vert \mathbf{x})$ 是 deterministic 分布, 并且 $p_{\mathcal{D}}(\mathbf{x})$ 是均匀分布。我们假设集 $\mathcal{H}$ 也是所有 $deterministic$ 分布的集合。那么 $P_{\mathcal{D}}(y \vert \mathbf{x}) \in \mathcal{H}$。
接下来我要证明，无论样本有多少个，我们都会存在过拟合的风险。

首先，我们讨论一个问题：我们认为真实数据集合为有限集合 $\mathcal{Z}$, 其中 $\vert \mathcal{Z} \vert = 2m$，我们从中采集m个样本 $\mathcal{S} = \{(\mathbf{x}_1, y_1), (\mathbf{x}_2, y_2), \ldots, (\mathbf{x}_m, y_m)\}$, 这里设样本集一共可能有 $K$ 种。另外，真实分布$\mathcal{D}$ 会有 $T = 2^{2m}$ 种, 即 $\mathcal{D} \in \{\mathcal{D}_1, \mathcal{D}_2, \ldots, \mathcal{D}_{T}\}$。 这里我们用 $0-1$ 误差函数来衡量分类函数。

关键的第一步，我们将要证明: 对于任意的机器学习算法 $A$,

$$
    \tag{1} \label{step1}
    \max_{i} \mathbb{E}_{\mathcal{S} \sim \mathcal{D}_i} [L_{\mathcal{D}_i}(A(\mathcal{S})] \ge \frac{1}{4}.
$$

!!! 证明
    提醒，对于有限集合$\mathcal{Z}$, 真实的 $p_{\mathcal{D}}(\mathbf{x})$ 是均匀分布。另外定义：大小为 m 的样本集$\mathcal{S}$ 一共可能有 $K$ 种。

    \begin{align*}
        \tag{2} \label{step2}
        &\max_{i} \mathbb{E}_{\mathcal{S} \sim \mathcal{D}_i} [L_{\mathcal{D}_i}(A(\mathcal{S})] \\
        =& \max_{i} \frac{1}{K} \sum^{K}_{j=1} L_{\mathcal{D}_i}(A(\mathcal{S}_j)) \\
        \ge& \frac{1}{T} \sum^{T}_{i=1} \frac{1}{K} \sum^{K}_{j=1} L_{\mathcal{D}_i}(A(\mathcal{S}_j)) \\
        =& \frac{1}{K} \sum^{K}_{j=1} \frac{1}{T} \sum^{T}_{i=1} L_{\mathcal{D}_i}(A(\mathcal{S}_j)) \\
        \ge& \min_{j}\frac{1}{T} \sum^{T}_{i=1} L_{\mathcal{D}_i}(A(\mathcal{S}_j)).
    \end{align*}

    因为样本集 $\mathcal{S}$ 会有重复的元素，我们定义没有出现在样本集中的元素对应的集合是 $\mathcal{Z}' = \{(\mathbf{x}_1, y_1), \ldots, (\mathbf{x}_p, y_p)\}$，其中 $p \ge m$。我们可得
    
    \begin{align*}
        \tag{3} \label{step3}
        &L_{\mathcal{D}_i}(A(\mathcal{S}_j)) \\
        =& \frac{1}{2m} \sum^{}_{(\mathbf{x}, y) \in \mathcal{Z}} 1 \{A(\mathcal{S}_j)(\mathbf{x}) \ne y\}\\
        \ge& \frac{1}{2p} \sum^{}_{(\mathbf{x}, y) \in \mathcal{Z}'} 1 \{A(\mathcal{S}_j)(\mathbf{x}) \ne y\} \\
        \ge& \frac{1}{2} \min_{(\mathbf{x}, y) \in \mathcal{Z}'} 1 \{A(\mathcal{S}_j)(\mathbf{x}) \ne y\}.
    \end{align*}
    
    我们带入 $\eqref{step2}$, 可得 $\forall A$,
    
    \begin{align*}
        \tag{4} \label{step4}
        &\max_{i} \mathbb{E}_{\mathcal{S} \sim \mathcal{D}_i} [L_{\mathcal{D}_i}(A(\mathcal{S})] \ge 
        \frac{1}{2} \min_{(\mathbf{x}, y) \in \mathcal{Z}'} \frac{1}{T} \sum^{T}_{i=1}1 \{A(\mathcal{S}_j)(\mathbf{x}) \ne y\}
        \ge \frac{1}{4}.
    \end{align*}

!!! 马尔可夫不等式及其变种

    马尔可夫不等式是：

    $$
    P[Z \ge a] \le \mathbb{E}[Z] / a.
    $$
    
    并且，如果 $Z\in[0,1]$ $\mathbb{E}[Z]=\mu$, 那么 $\forall a \in (0, 1)$，

    $$
        P[Z>1-a]=1-\mathbb{P}[Z\le 1-a]=1-P[{1-a\ge Z}] \ge1-\frac{\mathbb{E}[1-Z]}{a}=1-\frac{1-\mu}{a}.
    $$
    
那么，我们可得：$\exists \mathcal{D}$, 满足

$$
    \tag{5} 
    P_{S \sim \mathcal{D}}\left[L_{\mathcal{D}}(A(S)) \ge \frac{1}{8}\right] \ge \frac{1}{7}.
$$

进一步扩展地说，如果 $\mathcal{Z}$ 中的 $\mathbf{x}$ 空间是连续的, 不再是有限的，那么样本 $m$ 再大，我们也不可能成功地学习。
