# SVM算法

SVM算法本身就是一个线性函数的优化问题，
因为很多扩展而变得复杂起来。
学过优化的话就很好理解了：
SVM算法本质是将一个二分类问题建模成一个线性函数的优化问题，
它刚好又闭解，闭解的推导应该是属于优化的领域。

## 目标问题
在一切开始前，我们要先确定一个hypothesis集 $\mathcal{H}=\{h_{\theta}(\mathbf{x})\}$。 
现在给定一个样本集 $\mathcal{S} = \{(\mathbf{x}_1, y), (\mathbf{x}_2, y), \ldots, (\mathbf{x}_m, y)\}$，其中$\mathbf{x} \in \mathbb{R}^{d}$ 并且 $y \in \{-1, 1\}$。我们希望在 $\mathcal{H}$ 中找到一个分类函数 $h_\theta(\mathbf{x})$能把样本$\mathcal{S}$ 分得最开。
什么叫做分得最开呢？我们这里用样本与分类面的距离来评价分开的程度。

我们构建的hypothesis集是线性函数的集合：

$$
    \mathcal{H} = \{h_{\theta}(\mathbf{x}) = \mathbf{w}^T_{\theta} \mathbf{x} + b_{\theta}\}.\tag{1}
$$

那么，样本点 $(\mathbf{x}, y)$ 与超平面的欧式距离有闭解：

$$
    d(h_{\theta}, (\mathbf{x}, y)) = \frac{\vert \mathbf{w}^T_{\theta} \mathbf{x} + b_{\theta} \vert}{\Vert \mathbf{w}_{\theta} \Vert}.\tag{2}
$$

那我们可以构造损失函数是：

$$
    L_{\mathcal{S}} (\theta) = \min_{i \in [1, m]} 
    -\frac{y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta})}{\Vert \mathbf{w}_{\theta} \Vert}. \tag{3}
$$

当有分类错的点，这个损失函数为正，错误点距离超平面远，错得越离谱，损失函数越大。当分类全部正确时，超平面将点分得越开，损失函数越小。

> 这里我们用距离的下界来当作损失函数，相当于全民奔小康。如果用平均距离，会导致分类错误率高，但是平均距离小的分类器被选择，相当于劣币驱逐良币。我们学习分类器的本质还是用来分类的，不能忘了初心。

## Hard SVM算法
这一小节的算法假设 $\mathcal{H}$ 里存在经验误差等于0的分类器。上面的损失函数不容许任何一个点分类错误，所以对应的算法叫做 Hard SVM算法。
首先我们写出我们优化目标：

$$
    \max_{\theta} \min_{i\in[1, m]}
    \frac{y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta})}{\Vert \mathbf{w}_{\theta} \Vert}, \tag{4}
$$

这个优化目标的解有无限多组：如果 $(\mathbf{w}^*_{\theta}, b^*_{\theta})$ 是解，那么对任意$k > 0$, $(k \mathbf{w}^*_{\theta}, k b^*_{\theta})$ 都是这个优化目标的解。

我们只想求到一个解就好了:如果增加一个限制到分母上 $\Vert\mathbf{w}_{\theta}\Vert = 1$, 优化目标就变成了:

$$
    \max_{\theta} \min_{i\in[1, m]}
    y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta}),
    \quad s.t. \quad
    \Vert \mathbf{w}_{\theta} \Vert^2 = 1. \tag{5}
$$

这个问题等价的拉格朗日形式很难解：

$$
    \max_{\theta} min_{\pmb{\alpha} \ne 0} \left[\pmb{\alpha}(\Vert \mathbf{w}_{\theta} \Vert^2 - 1) + \min_{i\in[1, m]}
    y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta}) \right],
$$

> 拉格朗日形式等价的原因是：如果 $\Vert \mathbf{w}_{\theta} \Vert^2 \ne 1$，那么内层的优化$\min_{\alpha \ne 0}$就会等于无穷小，即$\max_{\theta,\Vert \mathbf{w}_{\theta} \Vert^2 \ne 1}$ 是无穷小，肯定小于$\max_{\theta,\Vert \mathbf{w}_{\theta} \Vert^2 = 1}$的值。

这里就要用到这一小节开头的假设，这一小节的算法假设 $\mathcal{H}$ 里存在经验误差等于0的分类器。我们可以增加一个限制到分子上，优化目标就变成了：

$$
    \max_{\theta} \frac{1}{\Vert \mathbf{w}_\theta \Vert}
    \quad s.t.\quad 
    \min_{i \in [1, m]} y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta}) = 1.  \tag{6}
$$

这里的变形就差最后一步了，我们想去掉 $\min_{i\in[1,m]}$ 操作，即把优化目标还可以简化成：

$$
    \min_{\theta} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2
    \quad s.t.\quad 
    \forall i \in [1, m],\quad  y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta}) \ge 1. \tag{7}
$$

这一步的等价原因是：
如果这个问题的最优解 $(\mathbf{w}^*_\theta, b^*_{\theta})$ 
导致 $\min_{i \in [1, m]} y_i(\mathbf{w^*}^T_{\theta} \mathbf{x}_i + b^*_{\theta}) = \gamma > 1$, 
那么一定存在解$(\mathbf{w}^*_\theta/\gamma, b^*_{\theta}/\gamma)$ 比 $(\mathbf{w}^*_\theta, b^*_{\theta})$ 更优。

对应的无约束拉格朗日形式是

$$
\min_{\theta} \max_{\pmb{\alpha} \succeq 0} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + \sum^{m}_{i=1}\alpha_i [1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta})]. \tag{8}
$$

现在来求解一下这个优化问题。

$$
\begin{align*}
    &\min_{\theta} \max_{\pmb{\alpha} \succeq 0} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + \sum^{m}_{i=1}\alpha_i [1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta})] \\
    =& \max_{\pmb{\alpha} \succeq 0}\min_{\theta}  \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + \sum^{m}_{i=1}\alpha_i [1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta})] \\
    &(\mathbf{w}^*_{\theta} = \sum^{m}_{i=1}\alpha_i y_i \mathbf{x}_i,\quad \sum^{m}_{i=1} \alpha_i y_i = 0.)\\
    =& \max_{\pmb{\alpha} \succeq 0} \sum^{m}_{i=1} \alpha_i - \frac{1}{2} \sum^m_{i=1} \sum^{m}_{j=1} \alpha_i\alpha_j y_i y_j \mathbf{x}^T_i \mathbf{x}_j \\
    =& \max_{\pmb{\alpha}\succeq 0} \mathbf{1}^T \pmb{\alpha} - \frac{1}{2} \mathbf{\alpha}^T \mathbf{D}_y \mathbf{X}^T \mathbf{X} \mathbf{D}_y \pmb{\alpha}. \tag{9}
\end{align*}
$$

其中 $\mathbf{D}_y=diag(y_1, y_2, \ldots, y_m)$，$\mathbf{X} = [\mathbf{x}_1, \mathbf{x}_2, \ldots, \mathbf{x}_m]$, 因此可得：

$$
\begin{cases}
    \pmb{\alpha}^* = (\mathbf{D}_y \mathbf{X}^T \mathbf{X} \mathbf{D}_y)^{-1} \mathbf{1}, \\
    \mathbf{w}^*_{\theta} = \mathbf{X}\mathbf{D}_y \pmb{\alpha}^*, \\
    \mathbf{b}^*_{\theta} = y_i - \sum^{m}_{i=1} \alpha_j y_j \mathbf{x}^T_j \mathbf{x}_i \quad
    (\alpha_i \ne 0 \overset{KKT}{\Rightarrow} \mathbf{w^*}^T_\theta \mathbf{x}_i + \mathbf{b}^*_\theta = y_i), \\
    \Vert \mathbf{w}^*_\theta\Vert = \Vert \mathbf{X D}_y \pmb{\alpha}^* \Vert^2 = \mathbf{1}^T (\mathbf{D}_y \mathbf{X}^T \mathbf{X} \mathbf{D}_y)^{-1} \mathbf{1} = \Vert \pmb{\alpha}^* \Vert_1.
\end{cases}
$$

其中 $\mathbf{b}^*_\theta$涉及到support集$\{(\mathbf{x}, y):\mathbf{w^*}_\theta^T \mathbf{x} + b^*_{\theta} = y \}$，
不在这个集合上的点 $(\mathbf{x}_i, y_i)$ 满足 $\alpha_i = 0$，也就是说 $(\mathbf{w}^*_\theta, b^*)$ 只是support集元素的放射组合，与不在supoort集上的样本无关。

## Soft SVM算法

在上一小节，算法的要求太强了，样本数据需要严格线性可分。
我们可以引入一个松弛因子 $\pmb{\xi}\in\mathbb{R}^{m}_{+}$, 
将优化目标修改成：

$$
\begin{align*}
    &\min_{\theta, \pmb{\xi}\succeq 0} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + C \Vert\pmb{\xi}\Vert^p, \\
    &\ s.t.\quad 
    \forall i \in [1, m],\quad  y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta}) \ge 1 - \xi_i.\tag{10}
\end{align*}
$$

这里 $\xi_i$可以观察得到最小值为$\max(0, 1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta}))$。
那么优化目标就变成了

$$
\begin{align*}
    &\min_{\theta} \sum^m_{i=1} \Vert \max(0, 1- y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i + b_{\theta})) \Vert^p + \frac{1}{2C} \Vert \mathbf{w}_\theta \Vert^2. \\
\end{align*}
$$

因为$l_{hinge}(x) = max(0, 1- x)$, 所以这个优化目标也可以看成是hinge损失函数加二范数正则项。

回到优化问题(10)。 $p=1$时，我们写出它的等价拉格朗日无约束形式：

$$
    \min_{\theta, \pmb{\xi}\succeq 0} \max_{\pmb{\alpha}\succeq \pmb{0}, \pmb{\beta} \succeq \pmb{0}} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + C \sum^{m}_{i=1} {\xi}_i + \sum^m_{i=1}\alpha_i[1 - \xi_i - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i - b_\theta)] - \sum^{m}_{i=1}\beta_i \xi_i.
$$

整理一下可得：

$$
    \min_{\theta, \pmb{\xi}\succeq 0} \max_{\pmb{\alpha}\succeq \pmb{0}, \pmb{\beta} \succeq \pmb{0}} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + \sum^{m}_{i=1} (C - \alpha_i - \beta_i){\xi}_i + \sum^m_{i=1}\alpha_i[1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i - b_\theta)].
$$

接下来我们来求解这个优化问题:

$$
\begin{align*}
    &\min_{\theta, \pmb{\xi}\succeq 0} \max_{\pmb{\alpha}\succeq \pmb{0}, \pmb{\beta} \succeq \pmb{0}} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + \sum^{m}_{i=1} (C - \alpha_i - \beta_i){\xi}_i + \sum^m_{i=1}\alpha_i[1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i - b_\theta)] \\
    =&\max_{\pmb{\alpha}\succeq \pmb{0}, \pmb{\beta} \succeq \pmb{0}} \min_{\theta, \pmb{\xi}\succeq 0} \frac{1}{2} \Vert \mathbf{w}_\theta \Vert^2 + \sum^{m}_{i=1} (C - \alpha_i - \beta_i){\xi}_i + \sum^m_{i=1}\alpha_i[1 - y_i(\mathbf{w}^T_{\theta} \mathbf{x}_i - b_\theta)] \\
    &(\mathbf{w}^*_{\theta} = \sum^{m}_{i=1} \alpha_i y_i \mathbf{x}_i, \sum^{m}_{i=1}\alpha_i y_i = 0, C = \alpha_i + \beta_i) \\
    =& \max_{C \succeq \pmb{\alpha}\succeq \pmb{0}} \sum^{m}_{i=1} \alpha_i - \frac{1}{2} \sum^{m}_{i=1, j=1} \alpha_i \alpha_j y_i y_j \mathbf{x}^T_i \mathbf{x}_j\quad(s.t. \sum^{m}_{i=1} \alpha_i y_i = 0). 
 \end{align*}
$$

> SMO优化算法来求解该问题。




