# 线性回归问题

## 问题描述

在实际生活中，我们能获得一系列的样本 $\mathcal{S} = \left\{ (x_1, y_1), (x_2, y_2), \dots, (x_m, y_m) \right\}$。其中 样本属于某个集合 $z = (x, y) \in Z = X \times Y$,
并且我们认为这些样本是服从某个未知的联合分布 $\mathcal{D}$，即 $(x, y) \sim \mathcal{D}$。 与之相关的未知条件分布 $P_{\mathcal{D}}(y \vert x)$ 是我们希望获得的：当我们知道了 $x$ 的值，我们就能知道 $y$ 的值最可能是多少。

如果 $y \in \mathbb{R}$, 那么求解 $P_\mathcal{D}(y\vert x)$ 的问题也叫做回归问题。
通常我们构造一个假设集 $\mathcal{H}_{\theta} = \{P_{\theta}(y \vert x)\}$, 我们从假设集中选一个最接近 $P_\mathcal{D}(y \vert x)$ 的集合 $P_{\theta^*}(y \vert x)$。对应的优化目标是

$$
    \min_{\theta} L_{\mathcal{D}}(P_{\theta}) = \mathbb{E}_{ P_{\mathcal{D}}(x)} [Distance(P_{\mathcal{D}}(\cdot \vert x) \Vert P_{\theta}(\cdot \vert x))]. \tag{1}
$$

## 确定性的线性回归
我们可以简化这个问题：当我们知道了 $P_\mathcal{D}(y \vert x)$ 的值后，给定一个 $x$, 我们通常预测 $y \in \arg\max_{y} P_{\mathcal{D}}(y \vert x)$。那么，我们不一定需要求解最接近 $P_\mathcal{D}(y \vert x)$ 的条件分布，我们可以转而求解分布满足 $\arg\max_{y} P_{\theta}(y \vert x) \subset \arg\max_y P_{\mathcal{D}}(y \vert x)$。
那么，我们可以构造更激进的候选集来简化问题:

$$
\mathcal{H}_\theta = \{P_{\theta}(y \vert \mathbf{x}) = \mathbf{1}_{\{y = h_{\theta}(\mathbf{x})\}}, h_\theta(\mathbf{x}) = \mathbf{w}^T_{\theta} \mathbf{x} + b_\theta\}. \tag{2}
$$

我们使用Wasserstein距离(输运花费函数为 $c(a, b)$)，那么优化目标就变成了

$$
    \min_{\theta} L_{\mathcal{D}}(\theta) 
    = \mathbb{E}_{(x, y) \sim \mathcal{D}}[c(h_{\theta}(\mathbf{x}), y)]. \tag{3}
$$

如果 $c(a, b) = \Vert a - b \Vert^2_2$, 那么就是大家熟知的平方误差损失函数:

$$
L_{\mathcal{D}}(\theta) = \mathbb{E}_{(x, y) \sim \mathcal{D}}[\Vert h_{\theta}(\mathbf{x}) - y \Vert^2_2]. \tag{4}
$$

对于样本集 $S = \{(\mathbf{x}_1, y_1), (\mathbf{x}_2, y_2), \ldots, (\mathbf{x}_m, y_m)\}$, 对应的经验误差函数为 

$$
L_{S}(\theta) = \frac{1}{m} \sum^{m}_{i = 1} \Vert h_{\theta}(\mathbf{x}_i) - y_i \Vert^2_2. \tag{5}
$$

> 注意：我没有提到高斯分布，而且不需要提到高斯分布的假设。

## 随机性的线性回归

如果我们希望不要那么激进，那么可以构造其他假设集, 例如：

$$
\mathcal{H}_{\theta} = \{p_{\theta}(y \vert \mathbf{x}) \sim \mathcal{N}(\pmb{\mu}^T_{\theta} \mathbf{x}, \mathbf{x}^T\pmb{\Sigma}_{\theta}\mathbf{x})\}. \tag{6}
$$

这个集合怎么来的呢？其实和贝叶斯线性回归有关系。贝叶斯线性回归就是给参数 $\mathbf{w}$ (b被包含在 $\mathbf{w}$ 中)添加一个先验分布，通常给 $\mathbf{w}$ 先验分布为某个参数化的高斯分布 $\mathcal{N}(\pmb{\mu}_\theta, \pmb{\Sigma}_{\theta})$，即假设集为 

$$
\mathcal{H}_{\theta} = \{h(\mathbf{x}) = \mathbf{w}^T \mathbf{x}, \mathbf{w} \sim \mathcal{N}(\pmb{\mu}_{\theta}, \pmb{\Sigma}_{\theta})\}.\tag{7} \label{linearhypothesis}
$$

那么，$\mathbb{E}_{\mathbf{w}}[h(\mathbf{x})] = \mathbf{w}^T \pmb{\mu}_{\theta}$, 并且 $Var_{\mathbf{w}}(h(\mathbf{x})) = \mathbf{x}^T \pmb{\Sigma_{\theta}} \mathbf{x}$。

这时，损失函数是有点麻烦了:

$$
    L_{\mathcal{D}}(\theta) = \mathbb{E}_{p_{\mathcal{D}}(\mathbf{x})}[Distance(p_{\mathcal{D}}(\cdot \vert \mathbf{x})\Vert p_{\theta}(\cdot \Vert \mathbf{x}))]. \tag{8}
$$

在这种情况下什么距离函数可以求呢？那就是Kullback-Leibler距离：

$$
    L_{\mathcal{D}}(\theta) = \mathbb{E}_{p_{\mathcal{D}}(\mathbf{x})}\mathbb{E}_{p_{\mathcal{D}}(y \vert \mathbf{x})}\left[\ln\frac{p_{\mathcal{D}}(y \vert \mathbf{x})}{p_{\theta}(y \vert \mathbf{x})}\right], \tag{9}
$$

我们略去与 $\theta$ 无关的项，那么对应的简化过的损失函数就是交叉熵：

$$
    L_{\mathcal{D}}(\theta) = -\mathbb{E}_{(\mathbf{x}, y) \sim \mathcal{D}}\left[\ln {p_{\theta}(y \vert \mathbf{x})}\right]. \tag{9}
$$

对于样本集 $S = \{(\mathbf{x}_1, y_1), (\mathbf{x}_2, y_2), \ldots, (\mathbf{x}_m, y_m)\}$, 对应的经验误差函数为 

$$
    L_{S}(\theta) = - \sum^{m}_{i=1} \ln p_{\theta}(y_i \vert \mathbf{x}_i), \tag{10}
$$

带入高斯分布的概率密度公式可得：

$$
\begin{align*}
    &L_{S}(\theta) = -\sum^{m}_{i=1}
    \ln\left(\frac{1}{\sqrt{2 \pi} \sigma_i} \exp\left\{-\frac{1}{2\sigma_i^2} (y_i - \bar{y}_i)^2\right\}\right)\\
    =& \sum^{m}_{i=1} \ln \sigma_i + \frac{1}{2\sigma_i^2} (y_i - \bar{y}_i)^2 + const \\
    =& \sum^{m}_{i=1}\left[ \frac{1}{2}\ln(\mathbf{x}_i^T \pmb{\Sigma}_{\theta} \mathbf{x}_i) + \frac{(y_i - \mathbf{x}^T_i \pmb{\mu}_{\theta})^2}{2\mathbf{x}_i^T \pmb{\Sigma}_{\theta} \mathbf{x}_i}\right] + const.
\end{align*}
$$

经验公式也可以写成

$$
L_S(\pmb{\mu}, \pmb{\Sigma})
    = \sum^{m}_{i=1} \left[\frac{1}{2}\ln(\mathbf{x}_i^T \pmb{\Sigma} \mathbf{x}_i) + \frac{(y_i - \mathbf{x}^T_i \pmb{\mu})^2}{2\mathbf{x}_i^T \pmb{\Sigma} \mathbf{x}_i}\right].
$$

求解：

$$
\begin{align*}
&\nabla_{\pmb{\mu}} L_S(\pmb{\mu}, \pmb{\Sigma})
= \sum^m_{i=1} \frac{(y_i - \mathbf{x}^T_i \pmb{\mu})\mathbf{x}_i}{\mathbf{x}_i^T \pmb{\Sigma} \mathbf{x}_i} \\
\Rightarrow& \sum^m_{i=1} \frac{y_i \mathbf{x}_i}{\mathbf{x}_i^T \pmb{\Sigma}^* \mathbf{x}_i} 
= \sum^m_{i=1} \frac{\mathbf{x}_i\mathbf{x}^T_i }{\mathbf{x}_i^T \pmb{\Sigma}^* \mathbf{x}_i} \pmb{\mu}^* \\
\Rightarrow& \pmb{\mu}^* = 
\left(\sum^m_{i=1} \frac{\mathbf{x}_i\mathbf{x}^T_i }{\mathbf{x}_i^T \pmb{\Sigma}^* \mathbf{x}_i}\right)^{-1} 
\sum^m_{i=1} \frac{y_i \mathbf{x}_i}{\mathbf{x}_i^T \pmb{\Sigma}^* \mathbf{x}_i}.
\end{align*}
$$

##贝叶斯线性回归

贝叶斯线性回归的假设集正如上面$\eqref{linearhypothesis}$式所描述的。
不过贝叶斯线性回归做的是在我们求出最优的 $\mathbf{w} \sim \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0)$ 后，
我们怎么样能做得更好。当然 $\mathbf{w} \sim \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0)$ 通常直接被认为给定一个分布，
人们叫它先验知识。我觉得加入上一节 [随机线性回归](#_4) 的先验模型学习更合理。

我们在贝叶斯回归时，实际上是在做什么？

现在我们已经有了分布 $p(\mathbf{w})$ 和 $p(y \vert \mathbf{x}, \mathbf{w})$，在贝叶斯框架中相当于我们已经获得了全部的
和概率相关的信息。现在我们给定一个样本 $S = \{(\mathbf{x}_1, y_1), (\mathbf{x}_2, y_2), \ldots, (\mathbf{x}_m, y_m)\}$, 
我们希望求得条件概率 $p(\mathbf{w} \vert S)$ , 来指导我们选择参数 $\mathbf{w}$。

令 $\mathbf{X} = [\mathbf{x}_1, \mathbf{x}_2, \ldots, \mathbf{x}_m]$, $\mathbf{y} = [y_1, y_2, \ldots, y_m]^T$, 我们已知

\begin{cases}
    \tag{11} \label{step1}
    p(\mathbf{w}) = \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0)\\
    p({\mathbf{y}} \vert \mathbf{X}, \mathbf{w}) = \mathcal{N}(\mathbf{X}^T \mathbf{w}, \mathbf{X}^T \pmb{\Sigma}_0 \mathbf{X})
\end{cases}

要求解 $p(\mathbf{w} \vert \mathbf{X}, \mathbf{y})$, 我们需要引入下面这个结论：

> **多元高斯分布的边缘分布与条件分布**：
>
> 已知分布：
>
> \begin{cases}
>     \tag{12} \label{step3}
>     p(\mathbf{x}) = \mathcal{N}(\pmb{\mu}, \pmb{\Lambda}^{-1}), \\
>     p(\mathbf{y} \vert \mathbf{x}) = \mathcal{N}(\mathbf{A}\mathbf{x} + \mathbf{b}, \pmb{L}^{-1}),
> \end{cases}
> 
> 我们可得：
>
> \begin{cases}
>     \tag{13} \label{step4}
>     p(\mathbf{y}) = \mathcal{N}(\mathbf{A}\pmb{\mu} + \mathbf{b}, \mathbf{L}^{-1} + \mathbf{A}\pmb{\Lambda}^{-1}\mathbf{A}^T), \\
>     p(\mathbf{x} \vert \mathbf{y}) = \mathcal{N}(\pmb{\Sigma}[\mathbf{A}^T \mathbf{L}(\mathbf{y} - \mathbf{b}) + \pmb{\Lambda\mu}], \pmb{\Sigma}),\\
     \pmb{\Sigma} = (\pmb{\Lambda} + \mathbf{A}^T \mathbf{L} \mathbf{A})^{-1}.
> \end{cases}
>
> (参考自：PRML(2006年出版)第93页)

带入上面的结论，我们可得：

$$
\tag{14} \label{step5}
p(\mathbf{w} \vert {\mathbf{y}}, \mathbf{X}) = \mathcal{N}(\pmb{\mu}, \pmb{\Sigma}),
$$

其中 $\pmb{\Sigma} = (\pmb{\Sigma}_0 + \mathbf{X} \mathbf{X}^T \pmb{\Sigma}_0 \mathbf{X}\mathbf{X}^T)^{-1}$，
并且 $\pmb{\mu} = \pmb{\Sigma} [\mathbf{X}(\mathbf{X}^T \pmb{\Sigma}_0 \mathbf{X})^{-1} \mathbf{y} + \pmb{\Sigma}_0^{-1} \pmb{\mu}_0]$。

这个公式还不够简练，求逆的操作比较多，我参考了[卡尔曼滤波](07-KalmanFilter.md)的公式来化简这个求解。

> 我抽象了一下卡尔曼滤波中的一个结论。
> 已知分布$p(\mathbf{x}) = \mathcal{N}(\pmb{\mu}_0, \pmb{\Sigma}_0)$ 和 $p(\mathbf{y} \vert \mathbf{x}) = \mathcal{N}(\mathbf{A}\mathbf{x} + \mathbf{b}, \mathbf{C})$，我们可得: $p(\mathbf{x} \vert \mathbf{y}) = \mathcal{N}(\pmb{\mu}_1, \pmb{\Sigma}_1)$，满足
>
> \begin{cases}
>     \tag{15} \label{kalmanfilter}
>     \mathbf{K} = \pmb{\Sigma}_0 \mathbf{A}^T (\mathbf{A} \pmb{\Sigma}_0 \mathbf{A}^T + \mathbf{C})^{-1};\\
>     \pmb{\Sigma}_1 = (\mathbf{I} - \mathbf{K} \mathbf{A}) \mathbf{\Sigma}_0;\\
>     \pmb{\mu}_1 = \pmb{\mu}_0 + \mathbf{K} [(\mathbf{y} - \mathbf{b}) - \mathbf{A} \pmb{\mu}_0]
> \end{cases}
>

最后，我们可以获得贝叶斯线性回归的求解公式：

\begin{cases}
    \tag{16} \label{step6}
    \pmb{K} = {\pmb{\Sigma}}_0 \mathbf{X}(2 \mathbf{X}^{T} {\pmb{\Sigma}}_0 \mathbf{X})^{-1};\\
    \pmb{\Sigma} = (\mathbf{I} - \mathbf{K} \mathbf{X}^T) {\mathbf{\Sigma}}_0;\\
    \pmb{\mu} = {\pmb{\mu}}_{0} + \mathbf{K}[\mathbf{y}  - \mathbf{X}^{T} {\pmb{\mu}}_0]. \\
\end{cases}
