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

要求解一个假设集 $\mathcal{H}$ 的VC维，我们需要找到一个样本集 $\mathcal{S}$ 满足：它
的 $\mathcal{H}_{\mathcal{S}}$ 能完全表达样本集的所有可能的分类函数；另一方面，我们还
要证明，所有大于 $\mathcal{S}$ 的样本集 $\mathcal{S}'$ 对应的 $\mathcal{H}_{\mathcal{S}'}$
不能完全包含 $\mathcal{S}'$ 的分类函数。

## Growth Function and Sauer's Lemma

!!! 定义
    (**Growth Function**) 对于任意一个假设集 $\mathcal{H}$，我们的定义它的成长函数为
    $$
        \tau_{\mathcal{H}}(m) = \max_{\mathcal{S} \subset \mathcal{X}, \vert 
        \mathcal{S} \vert = m} \vert \mathcal{H}_{S} \vert.
    $$

首先，如果 $VCdim(\mathcal{H}) = d$，那么当 $m \le d$ 时，$\tau_{\mathcal{H}}(m) =
2^m$。并且，根据 Sauer-Shelah-Perles 引理，我们可知，$\tau_{\mathcal{H}}(m)$ 的增长
速度是多项式的，而不是指数的。

!!! 引理
    (**Sauer-Shelah-Perles** 引理) 如果 $VCdim(\mathcal{H}) \le d$, 那么
    $\tau(m) \le \sum^d_{i=1} C^i_m$。特别地，当 $m > d+1$ 时，$\tau_{\mathcal{H}}(m) 
    \le (em / d)^d$。

## VC维与Rademacher复杂度的联系

我们已知，对于有限假设集 $\mathcal{H}$, 使用 $0-1$ 误差时，我们有界

$$
    R(\mathcal{H} \circ \mathcal{S}) \le 
    \sqrt{\frac{2 \ln \vert \mathcal{H} \vert}{m}}.
$$

其实 VC-维 给了我们一个无限假设集的有限界，所以我们很快可以得出：如果假设集 $\mathcal{H}$
的 VC维 是 $d$，并且 $m \ge d \ge 1$，那么：
$$
    R(\mathcal{H} \circ \mathcal{S}) \le 
    \sqrt{\frac{2 \ln \vert \Pi_{\mathcal{H}}(\mathcal{S}) \vert}{m}}
    \le 
    \sqrt{\frac{2 d \ln(em/d) }{m}}.
$$
