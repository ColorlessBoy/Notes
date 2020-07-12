# Boosting

弱分类器到强分类器，这里有趣的东西可太多了，我先把主干算法Adaboost整理一下。

## Adaboost

>定义1：(**Weak hypothesis class**).
>We call a hypothesis class is a weak hypothesis class of data $(\mathbf{x}, y) \sim \mathcal{D}$ we means that
>
>\begin{equation}
>    \mathcal{H}_{weak} = \left\{ 
>        h_1, h_2, \ldots, h_N : \forall k \in [N], L_{(\mathbf{x}_i, y_i) \sim \mathbf{p}_k}(h_k) \le \frac{1}{2} - \gamma, (\mathbf{x}_i, y_i) \in S
>    \right\}.\tag{1}
>\end{equation}
>

这里的 $\mathbf{p}_k$ 是在样本上的某个分布，在后面会有提到, 某一种设定方式。
这里指的是某一类分类算法给出任意的输入样本， 只能返回一个比较弱的分类器。错误率小于50%。

Adaboost 假设我们被提供了一个弱分类器的集合 $\mathcal{H}_{weak}$, 我们将从下面这个假设集中学习一个强分类器：

\begin{equation}
    \mathcal{H}_{ada} = 
    \left\{ 
        \sum^{N}_{i=1} \alpha_i h_i : \alpha_i \in \mathbb{R} \wedge h_i \in \mathcal{H}_{weak} 
    \right\}.\tag{2}
\end{equation}

Adaboot解决的问题是: 数据在 $(\mathbf{x}, y) \sim \mathcal{D}$, 并且 $y \in \{-1, 1\}$，$h_i(\mathbf{x}) \in\{-1, 1\}$。 我们要从 $\mathcal{H}_{ada}$ 中选出一个最好的分类器，能够更好地拟合真实条件分布 $P_{\mathcal{D}}(y \vert \mathbf{x})$。

这里使用的损失函数为

\begin{equation}
    {L}_{\mathcal{D}}(h) = \mathbb{E}_{\mathcal{D}} \left[ e^{- y h(\mathbf{x})} \right].\tag{3}
\end{equation}

对应的经验损失函数为


\begin{equation}
    {L}_{\mathcal{S}}(h) = \frac{1}{m} \sum_{i=1}^{m} e^{-y_i h(\mathbf{x}_i)}.
    \tag{4}
\end{equation}

为什么是这个损失函数呢？因为它有很好的数学性质。
我们可以用坐标梯度下降来求解这个优化问题，并且设定每一步的步长为最优步长。
也就是每一步的优化，我们要通过一个子问题来求解当前的最优步长:

\begin{equation}
    \arg\min_{\eta} {L}_{\mathcal{S}}(\alpha_1, \ldots, \alpha_{k-1}, \alpha_k + \eta, \alpha_{k+1}, \ldots, \alpha_{N})\tag{5}
\end{equation}

\begin{align}
    & m {L}_{\mathcal{S}}(\alpha_1, \ldots, \alpha_{k-1}, \alpha_k + \eta, \alpha_{k+1}, \ldots, \alpha_{N}) \\
    =& \sum_{i=1}^{m} exp\left\{- y_i \sum_{j=1}^{N} \alpha_{j} h_{j}(\mathbf{x}_i) - y_i h_{k}(\mathbf{x}_i) \eta\right\} \\
    \frac{\partial}{\partial \eta} {L}_{\mathcal{S}}(\eta)
    =& \sum_{i = 1}^{m} -y_i h_k(\mathbf{x}_i) exp\left\{- y_i \sum_{j=1}^{N} \alpha_{j} h_{j}(\mathbf{x}_i) - y_i h_{k}(\mathbf{x}_i) \eta\right\} \\
    =& \sum_{y_i \ne h_k(\mathbf{x}_i)} exp\left\{- y_i \sum_{j=1}^{N} \alpha_{j} h_{j}(\mathbf{x}_i) + \eta\right\} \\
    & - \sum_{y_i = h_k(\mathbf{x}_i)} exp\left\{- y_i \sum_{j=1}^{N} \alpha_{j} h_{j}(\mathbf{x}_i) - \eta\right\}
\end{align}

令  $L_S(\pmb{\alpha}) = Z = \sum_{i=1}^{m} exp\left\{ -y_i \sum_{j=1}^{N} \alpha_j h_j(\mathbf{x}_i) \right\}$,
某一个样本集 $S$ 上的分布 $\mathbf{p}_{k}(S) = \left\{ p_k(\mathbf{x}_1, y_1), p_k(\mathbf{x}_2, y_2), \ldots, p_k(\mathbf{x}_m, y_m)\right\}$, 满足 $\mathbf{p}_{k}(\mathbf{x}_i, y_i) = \frac{1}{Z}exp\left\{ -y_i \sum_{j=1}^{N} h(\mathbf{x}_i) \right\}$, 以及 $\epsilon_k = \mathbb{E}_{(\mathbf{x}, y) \sim \mathbf{p}_k} \left[ \pmb{1}_{h_k(\mathbf{x})\ne y} \right]$那么 

\begin{align}
    \frac{\partial}{\partial \eta} {L}_{\mathcal{S}}(\eta)
    &= \sum_{y_i \ne h_k(\mathbf{x}_i)}^{} p_{k}(\mathbf{x}_i, y_i) \exp(\eta)
    - \sum_{y_i = h_k(\mathbf{x}_i)}^{} p^{k}(\mathbf{x}_i, y_i) \exp(-\eta) \\
    &= \epsilon_k \exp(\eta) - (1 - \epsilon_k) \exp(-\eta), \\
    \Rightarrow& \eta^* = \frac{1}{2} \ln\left( \frac{1 - \epsilon_k}{\epsilon_k} \right).
\end{align}


综上所述，Adaboost算法的流程是：

- Step1: 初始化 $\pmb{\alpha} = \pmb{0}$, $k = 1$, $\mathcal{H}_{weak} = \{\}$;
- Step2: 求解$\mathbf{p}_{k}(S)$, 从样本 $\{(\mathbf{x}, y)\} \sim \mathbf{p}_{k}$ 中学习出一个弱分类器 $h_k$, $\mathcal{H}_{weak} = \mathcal{H} \cup \{h_k\}$;
- Step3: 求解$\epsilon_k$, $\eta_k$;
- Step4: 更新 $\pmb{\alpha}$ 和 $k$, 返回到第二步。


## 误差分析
首先 $\epsilon_k \le \frac{1}{2} - \gamma$, 那么

\begin{align}
    &\frac{L_{\mathbf{p}_k(S)}(\pmb{\alpha} + \eta \pmb{\epsilon}_k)}{L_{(\mathbf{x}, y) \sim \mathbf{p}_{k-1}(S)}(\pmb{\alpha})} \\
    =& (1 - \epsilon_k) \exp(-\eta) + \epsilon_k \exp(\eta) \\  
    =& 2 \sqrt{\epsilon_k(1 - \epsilon_k)}\\ 
    \le& 2\sqrt{\left( \frac{1}{2} - \gamma \right)\left( \frac{1}{2} + \gamma \right)} \\
    \le& \sqrt{1 - 4\gamma^2} \le \exp(-2\gamma^2).
\end{align}

如果取 $\pmb{\alpha}_0 = \pmb{0}$, 那么  $L_{S}(\pmb{\alpha}_0) = 1$, 所以可得
 $L_{\mathbf{p}_T(S)}(\pmb{\alpha}_T) \le \exp(-2\gamma^2T)$。

我们当然比较关心 $0-1$ 误差函数：$l^{0-1}(h, (\mathbf{x}, y)) = sign\{- y h(\mathbf{x})\}$,
它小于等于误差函数  $l^{exp}(h, (\mathbf{x}, y)) = \exp(-y h(\mathbf{x}))$, 所以

\begin{align}
    &{L}^{0-1}_{\mathcal{D}}(h) \le {L}_{\mathcal{D}}^{\exp}(h) \\
    ?=& \mathbb{E}_{S \sim \mathcal{D}^m} \left[ {L}_{\mathbf{p}_T(S)}^{exp}(h) \right] \\
    \le& \exp(-2\gamma^2 T).
\end{align}



