# Generations Of Boosting

这一篇涉及的是Boosting算法的进一步理解，算是一篇读书笔记，对我来说，我的收获很大。

几乎所有的机器学习算法构造的逻辑都是：先确定一个假设集，然后构造一个损失函数，然后在这个假设集里挑出那个使损失函数最小的元素。
在实际中，我们只能通过样本来优化这个损失函数，也就是说，我们只能优化对应的 **经验损失函数** 。对应的,原本的损失函数我们称为 **真实损失函数**。在统计机器学习的理论中，大部分工作都是在研究这两个损失函数之间的距离该怎么保障。

在我的所知里，所有的机器学习算法都可以按照这个逻辑进行，因为这个逻辑非常清晰明了，但是贝叶斯算法除外。贝叶斯框架下的算法很难写出一个真实损失函数和经验损失函数，导致了它的泛化误差比较难分析。关于贝叶斯方法我还有很多要说的，但是超过了这一篇读书笔记的主题，我就不再继续了。

Boosting算法是能够从损失函数的角度来理解，在[AdaBoost](05-AdaBoost.md)中，我们可以认为它对应的是一个自然指数损失函数的坐标梯度下降过程。不过这种理解不够具有泛化能力，我们无法将它推广到其他领域。这篇文章提供了一个新的损失函数来理解AdaBoost算法，同时也能推广出其他的Boosting算法。

## AdaBoost算法的复习与简单推广

我们首先回忆一下AdaBoost算法。AdaBoost算法是解决二分类问题的一个算法。它的样本空间 $(\mathbf{x}, y)$ 中的 $y \in \{-1, 1\}$ 。它的假设集如下

$$
    \tag{1}
    \mathcal{H}_{Ada} = \left\{ F_{\pmb{\lambda}} = \sum^{N}_{j=1}\lambda_j h_j, h_j \in \mathcal{H} \right\}.
$$

其中 $\mathcal{H}$ 是一个弱分类器的集合

$$
    \tag{2}
    \mathcal{H} = \left\{ h_1, \ldots, h_N \right\},
$$

它们的真实准确率都大于50%。

AdaBoost算法构造的损失函数是

$$
    \tag{3}
    L_S(\pmb{\lambda}) = \frac{1}{m}\sum^{m}_{i=1}\exp(-y_i F_{\pmb{\lambda}}(\mathbf{x}_i)).
$$

这里，我们可以用多种方法来优化这个损失函数：坐标梯度下降、随机坐标梯度下降以及梯度下降。

我们做一个简单的扩展，使得这个算法可以用于回归问题。

既然是回归问题，我们的样本空间 $(\mathbf{x}, y)$ 中的 $y\in\mathbb{R}$ 。
我们先不管弱分类器的定义是什么，总之我们拥有一个弱分类器，以及一个假设集，它的元素是弱分类器的线性组合。

那么对应的损失函数我们可以选择为：

$$
    \tag{4}
    L_S(\pmb{\lambda}) = \frac{1}{m}\sum^{m}_{i=1}(F_{\pmb{\lambda}}(\mathbf{x}_i) - y_i)^2.    
$$

类似的，我们想用坐标梯度下降的方法来优化这个损失函数，也就是说，我们每一步需要求解下面这个子问题：

$$
    \tag{5}
    \min_{\alpha_t, h_t} \frac{1}{m}\sum^{m}_{i=1}(F_{t-1}(\mathbf{x}_i) + \alpha_t h_t(\mathbf{x}_i) - y_i)^2.    
$$

首先我们可以求解得到：

$$
    \tag{6} \label{alphaOptimalStep}
    \alpha^*_t = \sum^{m}_{i=1}r_i \frac{h_t(\mathbf{x}_i)}{\Vert h_t \Vert^2_2},
$$

其中 $r_i = y_i - F_{t-1}(\mathbf{x}_i)$, 并且 $\Vert h_t \Vert_2 = \sqrt{\sum^{m}_{i=1}h^2_t(\mathbf{x}_i)}$.

我们计算损失函数的差值可得：

\begin{align*}
    & \frac{1}{m}\sum^{m}_{i=1}(F_{t-1}(\mathbf{x}_i) + \alpha_t h_t(\mathbf{x}_i) - y_i)^2 - 
    \frac{1}{m}\sum^{m}_{i=1}(F_{t-1}(\mathbf{x}_i)- y_i)^2\\
    =& \frac{1}{m}\sum^{m}_{i=1}[2 F_{t-1}(\mathbf{x}_i) - 2 y_i + \alpha_t h_t(\mathbf{x}_i)] \cdot \alpha_t h_t(\mathbf{x}_i)\\
    =& \frac{1}{m}\sum^{m}_{i=1}[\alpha_t h_t(\mathbf{x}_i) - 2 r_i] \cdot \alpha_t h_t(\mathbf{x}_i)
\end{align*}

我们将 $\alpha^*_t$ 带入上式可得一个新的子问题

$$
    \tag{7} \label{}
    \min_{h_t} -\frac{1}{m}\left( \sum^{m}_{i=1} r_i \frac{h_t(\mathbf{x}_i)}{\Vert h_t \Vert_2} \right)^2.
$$

如果我们能保证 $h_t(x_i) r_i$ 是正数，那么我们可以获得一个等价的损失函数

$$
    \tag{8}
    \min_{h_t} \frac{1}{m}\sum^{m}_{i=1}
    \left( \frac{h_t(x_i)}{\Vert h_t \Vert_2} - r_i \right)^2.
$$

## 泛函梯度下降

在前面，我们认为AdaBoost是一种参数空间$\pmb{\lambda}$上的坐标梯度下降算法。这种理解虽然很有用并且优美，但是也阻碍了我们扩展这个算法。这里，我们将介绍另一种视角来理解AdaBoost，它更有用更优美，并且它很容易推广到其他损失函数上。这个视角就是 **泛函梯度下降**。

首先，我们不再以参数空间 $\pmb{\lambda}$ 的视角来看待假设集以及损失函数，我们认为假设集的元素就是一个个函数，我们的损失函数的自变量也是一个函数。比如，在AdaBoost中，损失函数为：

$$
    \tag{9}
    L_S(F) = \frac{1}{m}\sum^{m}_{i=1} \exp{(-y_i F(\mathbf{x}_i))}.
$$

在这个经验损失函数中, 我们只关心函数$F$在样本集上的取值，因此函数 $F$ 也可以看成一个在空间 $\mathbb{R}^{m}$ 上的向量 $f$。我们要在全空间 $\mathbb{R}^{m}$ 上做优化。那么，如果我们使用梯度下降算法，那么它的更新策略就变成了

$$
    \tag{10}\label{functionalGradientDescent}
    F \leftarrow F - \alpha \nabla L_S(F),
$$

其中 $\nabla L_S(F) = \left[\frac{\partial{L_S(F)}}{\partial{F(\mathbf{x}_1)}}, \frac{\partial{L_S(F)}}{\partial{F(\mathbf{x}_2)}}, \ldots, \frac{\partial{L_S(F)}}{\partial{F(\mathbf{x}_m)}}\right]^T$.

如果$F$是在$\mathbb{R}^{m}$上，一方面它在$\mathbb{R}^{m}$上没有限制，导致它可以完美拟合样本集，另一方面它又只在$\mathbb{R}^{m}$上有定义，导致它没有泛化性。

我们回归到原始的假设集，我们希望$F$是在弱假设集 $\mathcal{H}$ 的元素作为基的空间上，并且我们限制 $F$ 的更新规则是

$$
    \tag{11}
    F(\mathbf{x}_i) \leftarrow F(\mathbf{x}_i) + \alpha h, \quad s.t. h \in \mathcal{H}.
$$

那么，我们为了尽可能让上式接近 **泛函梯度下降**，那么我们需要让 $h$ 与 $-\nabla L_S(F)$ 尽可能同向。这里我们将它转化为最大化内积的问题，构造一个子问题：

$$
    \tag{12}
    \max_{h \in \mathcal{H}} - \nabla L_S(F) \cdot h
    = - \sum^{m}_{i=1} \frac{\partial{L_S(F)}}{\partial{F(\mathbf{x}_i)}} h(\mathbf{x}_i).
$$

我们称上面这种方法为 **AnyBoost**。

我们首先把AdaBoost的指数损失函数带入到上式可以得到AdaBoost有关的子问题：

$$
    \tag{13}\label{AdaBoostFunctional}
    \min_{h_t} \frac{1}{m}\sum^{m}_{i=1}y_i h_t(\mathbf{x}_i) \exp(-y_i F_{t-1}(\mathbf{x}_i)).
$$

沿用AdaBoost定义的符号，我们使用 $D_t(i) = \frac{exp(-y_i F_{t-1} (\mathbf{x}_i))}{\sum^{m}_{j=1}exp(-y_j F(\mathbf{x}_j))}$，以及 $\epsilon_t = P_{(\mathbf{x}, y) \sim D_t} [h_t(\mathbf{x}) \ne y]$，我们可以得到$\eqref{AdaBoostFunctional}$式的一个等价问题

$$
    \tag{14}
    \min_{h_t} \epsilon_t，
$$

这就是AdaBoost中每一步中，对样本加权后的分类问题。

这时，我们再来回头看看回归问题泛函形式的损失函数：

$$
    \tag{15}
    L(F) = \frac{1}{m} \sum^{m}_{i=1}(F(\mathbf{x}_i) - y_i)^2，
$$

它对应的梯度为

$$
    \tag{16}
    \frac{\partial{L_S(F)}}{\partial{F(\mathbf{x}_i)}} = \frac{2}{m}(F(\mathbf{x}_i) - y_i),
$$

并且它对应的子问题是

$$
    \tag{17}
    \max_{h_t} \frac{2}{m} \sum^{m}_{i=1} h_t(\mathbf{x}_i) (y_i - F(\mathbf{x}_i)).
$$

备注，每一步的最优步长已经在 $\eqref{alphaOptimalStep}$ 中算出来了。


## 逻辑回归与条件概率

我们用逻辑回归的损失函数来替换AdaBoost所使用的指数损失函数：

$$
    \tag{18}
    L_S(F) = \sum^{m}_{i = 1}\ln[1 + exp(-y_i F(\mathbf{x}_i))],
$$

那么它的梯度就是

$$
    \tag{19}
    \frac{\partial{L(F)}}{\partial{F(\mathbf{x}_i)}} = 
    \frac{-y_i}{1 + \exp(y_i F(\mathbf{x}_i))},
$$

那么我们需要求解的子问题是:

$$
    \tag{20}
    \max_{h_t \in \mathcal{H}} \sum^{m}_{i=1}\frac{y_i h(\mathbf{x}_i)}{1 + \exp(y_i F(\mathbf{x}_i))}.
$$

这个子问题相相当于给每个样本增加权重 $\frac{1}{1 + \exp{(y_i F(\mathbf{x}_i))}}$ 的分类问题。 这个权重是在[0,1]区间内的，而原本的AdaBoost的权重 $\exp(-y_iF(\mathbf{x}_i)$ 是没有上界的。

接着，我们需要计算最优的更新步长 $\alpha_t$。
我们依旧是构造一个子问题来求解最优步长 $\alpha_t$:

$$
    \tag{21}
    \min_{\alpha_t} \sum^{m}_{i=1} \frac{\exp{(-y_i \alpha h(\mathbf{x}_i))} - 1}{1 + \exp(y_i F(\mathbf{x}_i))}
$$

???证明
    这里的证明我直接从原书中拿过来了，所以符号有一点点不同，不过不影响理解：

    \begin{aligned}
    \Delta \mathcal{L} & \doteq \mathcal{L}(F+\alpha h)-\mathcal{L}(F) \\
    &=\sum_{i=1}^{m} \ln \left(1+e^{-y_{i}\left(F\left(x_{i}\right)+\alpha h\left(x_{i}\right)\right)}\right)-\sum_{i=1}^{m} \ln \left(1+e^{-y_{i} F\left(x_{i}\right)}\right) \\
    &=\sum_{i=1}^{m} \ln \left(\frac{1+e^{-y_{i}\left(F\left(x_{i}\right)+\alpha h\left(x_{i}\right)\right)}}{1+e^{-y_{i} F\left(x_{i}\right)}}\right) \\
    &=\sum_{i=1}^{m} \ln \left(1+\frac{e^{-y_{i}\left(F\left(x_{i}\right)+\alpha h\left(x_{i}\right)\right)}-e^{-y_{i} F\left(x_{i}\right)}}{1+e^{-y_{i} F\left(x_{i}\right)}}\right) \\
    & \leq \sum_{i=1}^{m} \frac{e^{-y_{i}\left(F\left(x_{i}\right)+\alpha h\left(x_{i}\right)\right)}-e^{-y_{i} F\left(x_{i}\right)}}{1+e^{-y_{i} F\left(x_{i}\right)}} \\
    &=\sum_{i=1}^{m} \frac{e^{-y_{i} \alpha h\left(x_{i}\right)}-1}{1+e^{y_{i} F\left(x_{i}\right)}}
    \end{aligned}

我们定义 $D_t(i) = \frac{1/(1 + exp(y_i F(\mathbf{x}_i)))}{\sum^{m}_{j=1} 1/(1 + \exp(y_i F(\mathbf{x}_i)) }$, 以及误差值 $\epsilon_t = P_{(\mathbf{x}, y) \sim D_t}[y \ne h(\mathbf{x})]$, 那么和AdaBoost中最优步长推导一样，我们可以得到:

$$
    \tag{22}
    \alpha^* = \frac{1}{2} \ln\left( \frac{1-\epsilon}{\epsilon} \right).
$$

上面的这个算法也叫作 **AdaBoost.L**。从它中间构造的 $D_t$ 可得：在离群点数据中，它比AdaBoost更鲁邦，敏感度更低。
虽然我们在推导中经过了一次放缩，最后的的 $\alpha^*$ 也是原问题的最优解的。(具体分析在书中第200页）。







































!!!cite
    [Boosting:Fundations and Algorithms](https://www.amazon.com/Boosting-Foundations-Algorithms-Adaptive-Computation/dp/0262526034),ch7.

