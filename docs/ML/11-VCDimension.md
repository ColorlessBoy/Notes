# VC Dimension

从 [没有免费午餐](10-NoFreeLunch.md) 得到启示：对于二分类问题，我们如果使用某个非常强大的假设集，那么学出来的分类器并没有什么意义。那么，对于一个二分类问题，我们的假设集应该满足什么性质才能保证算法是PAC可学的呢？这里我们介绍一个概念，VC维(由Vladimir Vapnik 和 Alexey Chervonenkis 提出)。

这一节中，我们把问题限制在二分类问题中，即样本空间 $\mathcal{Z} = \mathcal{X} \times \mathcal{Y}$, 其中 $\mathcal{Y} = \{-1, 1\}$。并且，我们构造的假设集 $\mathcal{H} = \{ h(\mathbf{x}) \}$ 是deterministc分布的集合。

我们定义一个样本集 $\mathcal{S} = (\mathbf{x}_1, \mathbf{x}_2, \ldots, \mathbf{x}_m)$。那么，我们结合 $\mathcal{S}$ 和 假设集 $\mathcal{H}$ 得到一个新的集合：

$$
    \tag{1} \label{restriction}
    \mathcal{H}_{\mathcal{S}} = \{(h(\mathbf{x}_1), h(\mathbf{x}_2), \ldots, h(\mathbf{x}_m)), h \in \mathcal{H}, \mathbf{x}_i \in \mathcal{S}\}.
$$

这个集合是 $\mathcal{S}$ 上的一个假设集，并且是有限的，$\vert \mathcal{H}_{\mathcal{S}} \vert \le 2^{m}$。如果 $\vert \mathcal{H}_{\mathcal{S}} \vert = 2^{m}$, 我们称 $\mathcal{H}$ **shatters** $\mathcal{S}$。

!!! 定义
    (**VC Dimension**) 给定假设集 $\mathcal{H}$，它的VC维是
    $$
        \max \vert \mathcal{S} \vert, \quad \vert \mathcal{H}_{\mathcal{S}} \vert = 2 ^{\vert \mathcal{S} \vert}.
    $$
