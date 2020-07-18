# 分类问题

## 问题描述

在现实生活中，我们能获得一系列的样本 $\mathcal{S} = \left\{ (\mathbf{x}_1, y_1), (\mathbf{x}_2, y_2), \ldots, (\mathbf{x}_m, y_m) \right\}$。 其中 $\mathbf{x} \in \mathbb{R}^d$ 并且 $y$ 属于某个有限的离散集合，例如： $y \in \{0, 1\}$, $y \in \{-1, 1\}$ 或者 $y \in \{1, 2, 3, \ldots, k\}$。我们认为这些样本服从某个未知的联合分布, 即 $(\mathbf{x}, y) \sim \mathcal{D}$。我们希望得到对应的未知条件分布 $P_{\mathcal{D}} (y \vert \mathbf{x})$, 用来指导我们在知道 $\mathbf{x}$ 的情况下预测 $y$ 值。如果 $y$ 所在集合的大小为2，那我们就称这个问题为二分类问题。

通常我们会构造一个假设集：

$$
    \tag{1} \label{hypothesis1}
    \mathcal{H} = \left\{ p_{\theta}(y \vert \mathbf{x}) \right\}.
$$

SVM 就是用来解决二分类问题的方法，它对应的假设集为

$$
    \tag{2} \label{svmhypothesis}
    \mathcal{H}_{\theta} = \left\{ h(\mathbf{x}) = sign(\mathbf{w}^T_{\theta} \mathbf{x} + b_{\theta}) \right\},
$$

这里对应的分布是 $P\{y = h(\mathbf{x}) \} = 1$。

## 逻辑回归
逻辑回归解决的问题是 $y \in \{-1, 1\}$.
它使用的假设集复合了线性函数与sigmoid函数:

$$
    \tag{3} \label{logichypothesis}
    \mathcal{H}_{\theta} = \left\{ P_{\theta}(y = 1 \vert \mathbf{x}) = \frac{1}{1 + exp(\mathbf{w}^T_{\theta} \mathbf{x} + b_\theta)} \right\}.
$$

我们依旧用KL距离来衡量两个分布之间的距离：

$$
    \tag{4} \label{trueLossOfLogic}
    L_{\mathcal{D}}(\theta) = \mathbb{E}_{P_{\mathcal{D}}(\mathbf{x})}\left\{\mathbb{E}_{P_\mathcal{D}(y \vert \mathbf{x})}\left[\ln \frac{P_{\mathcal{D}}(y \vert \mathbf{x})}{P_{\theta}(y \vert \mathbf{x})}\right]\right\}.
$$

对应的经验损失函数:

$$
    \tag{5} \label{empiricalLossOfLogic}
    L_{S}(\theta) = - \frac{1}{m} \sum^{m}_{i=1} \ln P_{\theta}(y_i \vert \mathbf{x}_i).
$$

因为sigmoid函数有一个特别的性质，所以比较流行吧：

\begin{align*}
    &P_{\theta}(y = -1 \vert \mathbf{x}) \\
    =& 1 - \frac{1}{1 + exp(\mathbf{w}^T_{\theta} \mathbf{x} + b_\theta)} \\
    =& \frac{exp(\mathbf{w}^T_{\theta} \mathbf{x} + b_\theta)}{1 + exp(\mathbf{w}^T_{\theta} \mathbf{x} + b_\theta)} \\
    =& \frac{1}{1 + exp[-(\mathbf{w}^T_{\theta} \mathbf{x} + b_\theta)]}\\
    \tag{6}
\end{align*}

综上，我们可以把经验损失函数改写成：

$$
    \tag{7} 
    L_{S}(\theta) = \frac{1}{m} \sum^{m}_{i=1} \ln \{1 + exp[y(\mathbf{w}^T_{\theta} \mathbf{x} + b_\theta)]\}.
$$

我这里不纠结正负号的问题，因为不涉及本质。我要吐槽的是，只是因为 Sigmoid 函数的特别性，而且只是能够简化损失函数，就受到很多的关注，这种事情就很离谱。对于二分类问题，线性函数加 Sigmoid 代表的分布 $P_\theta(y \vert \mathbf{x})$ 实在是太局限了。

## 多分类问题

各种教材中，多分类问题总是纠结在 ''一对一策略'' 和 ''一对多策略''，其实就是钻了牛角尖，想用线性函数来解决多分类问题。
虽然这种方法在理论分析中，可以直接搬用二分类问题的分析结果，但是实际中是非常局限，我相信现在绝大多数人没有用线性函数来解决多分类问题了。很多人惊讶神经网络的高效，我惊讶的是，当年人们会用超平面解决多分类问题。

对于k分类问题，我们更希望使用某个函数 $f: \mathbf{x} \rightarrow \pmb{\mathcal{P}}$, 其中 $\pmb{\mathcal{P}}$ 是一个离散概率分布，即 $\pmb{\mathcal{P}} \in \mathbb{R}^{k}$, $\pmb{\mathcal{P}} \succeq \pmb{0}$ 并且 $\sum^{k}_{i=1} \mathcal{P}_i = 1$。那么对应的假设集就变成了:

$$
    \tag{8} \label{khypothesis}
    \mathcal{H} = \left\{ P_{\theta}(y \vert \mathbf{x}) \right\}.
$$

这个假设集就相当合理了。

我们依旧可以用 KL 距离来构造一个损失函数:

$$
    \tag{9} \label{ktrueloss}
    L_{\mathcal{D}}(\theta) = \mathbb{E}_{p_{\mathcal{D}}(\mathbf{x})}
    \left\{  
        \mathbb{E}_{P_{\mathcal{D}}(y \vert \mathbf{x})}
        \left[ \ln \frac{P_{\mathcal{D}}(y \vert \mathbf{x})}{P_{\theta}(y \vert \mathbf{x})} \right].
    \right\}
$$

对应的经验误差函数为:

$$
    \tag{10} \label{kempiricalloss}
    L_{S}(\theta) = -\frac{1}{m} \sum^{m}_{i=1} \ln P_{\theta}(y_i \vert \mathbf{x}_i).
$$

在神经网络中，一般的输出为 $\{z_1, z_2, \ldots, z_k\}$ 并不满足概率分布的要求。
这里最直觉的方法是：我们预测 $y = {\arg\max}_i z_i$，
这种方法相当于把假设集又简化成了确定性分类函数的集合，
一方面降低了它的表达能力, 
另一方面当我们做出错误预测时，$P_{\theta} = 0$, 我们无法再使用 $\eqref{kempiricalloss}$ 式作为我们的损失函数。

通常我们通过一层 [Softmax 层](https://pytorch.org/docs/master/generated/torch.nn.Softmax.html?highlight=softmax#torch.nn.Softmax) 实现归一化：

$$
    \tag{11} \label{sigmoid}
    p(y = i \vert \mathbf{x}) = \frac{exp(z_i)}{\sum^{k}_{j=1} exp(z_j) }.
$$

接下来我们来讲一下 ${\arg\max}_i z_i$ 和 Softmax 的联系。

首先我们从另一个角度来理解一下 ${\arg\max}_i z_i$ 操作。 给定一个序列 $\{z_1, z_2, \ldots z_k\}$, 我们实际想优化：

$$
    \tag{12} \label{argmax}
    \max_{P_\theta(z)} \mathbb{E}_{P_\theta(z_i)}[z_i] = \sum^{k}_{i=1} z_i P_{\theta}(z_i).
$$

这个问题的解自然是 $P_{\theta}(z = \max z_i) = 1$。

那么，Softmax 对应的优化问题是增加了一个熵的惩罚项：

$$
    \tag{13} \label{softmax}
    \max_{P_\theta(z)} \sum^{k}_{i=1} z_i P_{\theta}(z_i) - \alpha \sum^{k}_{i=1} P_{\theta}(z_i) \ln P_{\theta}(z_i).
$$

或者可以写成

$$
    \tag{14} \label{softmax2}
    \max_{\mathbf{P}} \sum^{k}_{i=1} z_i P_i - \alpha \sum^{k}_{i=1} P_i \ln P_i,
    \quad \forall i, P_i > 0, \sum^{k}_{i=1} P_i = 1.
$$

写成拉格朗日对偶形式可得

$$
    \tag{15} \label{softmax3}
    \max_{\mathbf{P}} \min_{\lambda \ne 0} \sum^{k}_{i=1} z_i P_i - \alpha \sum^{k}_{i=1} P_i \ln P_i + \lambda \left(\sum^{k}_{i=1} P_i - 1\right),
$$

对应的对偶形式是

$$
    \tag{16} \label{softmax4}
    \max_{\mathbf{P}} \min_{\lambda \ne 0} \sum^{k}_{i=1} z_i P_i - \alpha \sum^{k}_{i=1} P_i \ln P_i + \lambda \left(\sum^{k}_{i=1} P_i - 1\right).
$$

首先对内部子问题进行优化可得：

$$
    \tag{17} \label{softmax5}
    \alpha \ln P^*_i = z_i - \alpha + \lambda
    \Rightarrow P^*_i = \exp \left\{ \frac{z_i - \alpha + \lambda}{\alpha} \right\}.
$$

假设我们获得了 $\lambda^*$，那么根据概率的性质可得：


$$
    \tag{18} \label{softmax6}
    \sum^{k}_{i = 1} \exp\left\{ \frac{z_i - \alpha + \lambda^*}{\alpha} \right\} = \sum^{k}_{i=1} P^*_i = 1, \\
    \Rightarrow exp \left\{ \frac{-\alpha + \lambda^*}{\alpha} \right\} = \frac{1}{\sum^{k}_{i=1}exp \left\{ z_i / \alpha \right\}}.
$$

带入到$\eqref{softmax5}$式，我们可得：

$$
    \tag{19} \label{softmax7}
    P^*_i = \frac{exp(z_i/\alpha)}{\sum^{k}_{i=1}exp(z_i/\alpha)}.
$$

因子 $\alpha$ 可以包含到神经网络的前面层中，所以 Softmax 层固定 $\alpha=1$。
