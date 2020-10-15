# Rademacher复杂度

## 前置知识
在实际生活中，我们能获得一系列的样本 $\mathcal{S} = \left\{ (x_1, y_1), (x_2, y_2), \dots, (x_m, y_m) \right\}$。其中 样本属于某个集合 $z = (x, y) \in Z = X \times Y$,
并且我们认为这些样本是服从某个未知的联合分布 $\mathcal{D}$，即 $(x, y) \sim \mathcal{D} $。 与之相关的未知条件分布 $P_{\mathcal{D}}(y \vert x)$ 是我们希望获得的：当我们知道了 $x$ 的值，我们就能知道 $y$ 的值最可能是多少。

我们人为地构造某个特定条件分布的集合(Hypothesis Set): $\mathcal{H} = \{P_{\theta}(y \vert x)\}$，这些条件分布被 $\theta$ 参数化。 机器学习算法目标就是根据样本从候选集中选择最接近真实分布 $P_{\mathcal{D}}(y \vert x)$ 的分布 $P_{\theta}(y \vert x)$。

讲到这里就太抽象了，必须要一个例子来说明一下了。
> 例子1：一个线性回归问题中，假设集是某个空间超平面的集合 $\mathcal{H}_{\theta} = \{h_\theta(x) = w^T_{\theta} x + b_{\theta}\}$, 这里的条件分布对应的是 $P_{\theta}\{y = h_\theta(x) \vert x\} = 1$。我们可以选取某个衡量两个分布差异的距离来作为损失函数，求这个损失函数的最小值，就可以得到在这个距离下，最接近真实条件分布的参数化条件分布, 即优化目标为：
>
> $$
> \min_{\theta} \mathbb{E}_{x \sim P_{\mathcal{D}}(x)} [ Dist(P_{\mathcal{D}}(\cdot \vert x) \Vert P_{\theta}(\cdot \vert x))]. \tag{1}
> $$
>
> 这里如果使用Wasserstein Distance, 并且使用花费函数 $c(a, b) = \Vert a - b\Vert^2$, 那么就是可以得到经典回归问题的优化目标：
>
> $$
> \min_{\theta} \mathbb{E}_{(x, y) \sim \mathcal{D}} [\Vert h_{\theta}(x) - y \Vert^2]. \tag{2}
> $$
>
> 如果 $y \in \{0, 1\}$, 也就是对于一个二分类问题，优化目标等价于0-1损失：
>
> $$
> \min_{h \in \mathcal{H}} L_{\mathcal{D}}(h) = \mathbb{E}_{(x, y) \sim \mathcal{D}} [\mathbb{1}_{h(x) \ne y}] = \mathbb{E}_{(x, y) \sim \mathcal{D}} [y \cdot h(x)]. \tag{3}
> $$

接下来的讨论都是围绕0-1分类问题而进行的。非常不幸的是，上面提到的真实损失函数我们都没办法直接进行优化，我们能做的是优化对应的经验损失函数：

$$
\min_{h \in \mathcal{H}} \mathbb{E}_{(x, y) \sim \mathcal{S}} [\mathbb{1}_{h(x) \ne y}] 
= \frac{1}{m} \sum^{m}_{i=1} l^{0-1}(h(x), y), 
\tag{4}
$$

其中 $l^{0-1}(h(x), y) = 1_{h(x) = y}$.

上面这种优化目标有一个名字叫做经验风险最小化(Empirical Risk Minimization, ERM)，(4)式对应的最优分类器 $h^*$ 可以写成 $ERM_{\mathcal{H}}(S)$。

## Representativeness

首先引入一个评价样本的标准:

> 定义1： $\mathcal{S}$ 被称为 $\epsilon-representative$，如果
>
> $$
> \sup_{h \in \mathcal{H}} \vert L_\mathcal{D}(h) - L_{\mathcal{S}}(h) \vert \le \epsilon.
> $$
>

如果 样本 $S$ 是 $\epsilon-representative$， 那么 

$$
\forall h \in \mathcal{H},
L_{\mathcal{D}}(ERM_{\mathcal{H}}(S)) 
\le L_{\mathcal S}(ERM_{\mathcal{H}}(S)) + \epsilon \\
\le L_S(h) + \epsilon \le 
L_{\mathcal{D}}(h) + 2 \epsilon,
$$

即 $L_\mathcal{D}(ERM_{\mathcal{H}}(S)) \le \min_{h \in \mathcal{H}} L_{\mathcal{D}}(h) + 2 \epsilon$。也就是说，通过ERM标准求到的分类器是能够接近最优值的，并且 $\epsilon$ 能够体现两者的接近程度。如果 $\epsilon$ 足够小，那么 $S$ 足以代表真实分布 $\mathcal{D}$。

接下来同样是纯数学的东西。我们要解决的问题是：给定任意一个样本 $S$, 我们要估计它的代表程度(representiveness)。
为了讨论方便，我们首先定义一个新的集合:

$$
\mathcal{F} = l \circ \mathcal{H} = 
\{f_h(x, y) = l(h(x), y): h \in \mathcal{H}\}, \tag{5}
$$

这样，真实误差可写成 $L_\mathcal{D}(f) = \mathbb{E}_{(x, y) \sim \mathcal{D}} [ f(x, y)]$，经验误差可以写成 $L_S(f) = \frac{1}{m} \sum^{m}_{i=1} f(z_i)$。那么样本 $S$的代表性可以重新写成

$$
Rep_{\mathcal{D}}(\mathcal{H}, S) = 
\sup_{f \in \mathcal{F}} (L_{\mathcal{D}}(f) - L_{S}(f)). \tag{6}
$$

其实(6)式只是定义1换了一种写法，我们没有办法真的计算它。接下来我们将介绍如何它对应地经验公式，以及经验公式和(6)式的gap。

## 样本集代表性的经验估计: Rademacher复杂度

首先介绍一个直觉上的经验估计：将 $S$ 均分成两个样本集 $S_1$ 与 $S_2$。均分的意思是$S$中的样本$(x, y)$ 只会出现在 $S_1$ 与 $S_2$ 中的一个, 而且一定会出现在其中一个样本集中。
那么，$S$代表性的经验估计是：

$$
\sup_{f \in \mathcal{F}} (L_{S_1}(f) - L_{S_2}(f)). \tag{7}
$$

这个经验估计并不是符合所有人的直觉的，所以你如果感觉莫名其妙，也不要慌，跟着我继续。
我们可以构造一个随机变量 $\sigma \in \{-1, 1\}$，它有0.5的概率取$1$, 有0.5的概率取$-1$，那么一个更好的经验估计可以写成

$$
R(\mathcal{F} \circ S) = 
\frac{1}{m} \mathbb{E}_{\sigma_i \sim U(\{-1, 1\})} \left[ \sup_{f \in \mathcal{F}} \sum^{m}_{i = 1} \sigma_i f(z_i) \right], \tag{8}
$$

这个式子就叫做 Rademacher复杂度。这个估计比(7)式复杂得多，是$2^m$指数倍，计算复杂的问题我们放到后面解决。我们先来解决一下这个Rademacher复杂度和样本表达性之间的关系。

> 定理1：
>
> $$
>     \mathbb{E}_{S\sim\mathcal{D}^m} [Rep_{\mathcal{D}}(\mathcal{F}, S)] \le 2 \mathbb{E}_{S \sim \mathcal{D}^m}[R(\mathcal{F} \circ S)].
> $$

证明很巧妙，书上有。
根据定理1，我们可以得到几个引理：

- 首先 $\mathbb{E}_{S\sim\mathcal{D}^m} [L_{\mathcal{D}}(ERM_{\mathcal{H}}(S)) - L_{S}(ERM_{\mathcal{H}}(S))] \le 2 \mathbb{E}_{S \sim \mathcal{D}^m} R(l \circ \mathcal{H} \circ S)$;

- 其次 $\mathbb{E}_{S\sim\mathcal{D}^m} [L_{\mathcal{D}}(ERM_{\mathcal{H}}(S)) - L_{S}(h^*)] \le 2 \mathbb{E}_{S \sim \mathcal{D}^m} R(l \circ \mathcal{H} \circ S)$,其中 $h^* = \min L_{\mathcal{D}}(h)$;

- 最后 $P \left\{ L_{\mathcal{D}}(ERM_{\mathcal{H}}(S)) - L_{S}(h^*) \le \frac{2}{\delta} \mathbb{E}_{S \sim \mathcal{D}^m} R(l \circ \mathcal{H} \circ S) \right\} \le \delta$。(马尔可夫不等式)

为了接下来的分析，需要先介绍一个不等式：McDiarmid's Inequality。

> 定理2：(McDiarmid's Inequality)。
> 如果 $i \in [1, m]$, 并且
>
> $$
> f(x_1, \ldots, x_m) - f(\ldots, x_{i-1}, x'_i, x_{i+1},...) \in [a_i, b_i], 
> $$ 
>
> 那么
>
> $$
> P\{f - \mathbb{E} f \ge \epsilon\} \le exp\left(\frac{-2 \epsilon^2}{\sum^m_{i = 1}(a_i - b_i)^2}\right),
> $$
>
> $$
> P\{f - \mathbb{E} f \le \epsilon\} \le exp\left(\frac{-2 \epsilon^2}{\sum^m_{i = 1}(a_i - b_i)^2}\right),
> $$
>
> $$
> P\left\{f - \mathbb{E} f \le \sqrt{\frac{1}{2} \sum^m_{i=1}(a_i - b_i)^2 \log(1/\delta)}\right\} \ge 1 - \delta,
> $$
>
> $$
> P\left\{f - \mathbb{E} f \le (b - a) \sqrt{\frac{m}{2} \log(1/\delta)}\right\} \ge 1 - \delta,
> $$
> 
> $$
> P\left\{\vert f - \mathbb{E} f\vert \le \sqrt{\frac{1}{2} \sum^m_{i=1}(a_i - b_i)^2 \log(2/\delta)}\right\} \ge 1 - \delta,
> $$
> 
> $$
> P\left\{\vert f - \mathbb{E} f\vert \le (b - a) \sqrt{\frac{m}{2} \sum^m_{i=1}\log(2/\delta)}\right\} \ge 1 - \delta.
> $$

> 定理3：(霍夫听不等式)
> 对于m个随机变量$X_1 \in [a_1, b_1], X_2 \in [a_2, b_2], \ldots, X_m \in [a_m, b_m]$, 它们均值的随机变量$\bar X$满足
> 
> $$
> P(\bar X - \mathbb{E}[\bar X] \ge \epsilon) \le 
> exp\left( -\frac{2\epsilon^2 m^2}{ \sum^m_{i=1} (b_i - a_i)^2} \right).
> $$

这时，我们可以获得与数据直接有关的边界，而不只是期望。
> 定理4：(数据相关界)。如果 $l(h(x), y) \in [a, b]$，那么
> 
> $$
> P_{S\sim\mathcal{D}^m} \{\forall h \in \mathcal{H}, L_{\mathcal{D}}(h) - L_{S}(h) \le 2 \mathbb{E}_{S' \sim \mathcal{D}^m} R(l \circ \mathcal{H} \circ S') + (b - a) \sqrt{2 \ln(1/\delta) / m}\} \ge 1 - \delta.
> $$
> 
> $$
> P_{S\sim\mathcal{D}^m} \left\{\forall h \in \mathcal{H}, L_{\mathcal{D}}(h) - L_S(h) \le 2 R(l \circ \mathcal{H} \circ S) + 3(b - a) \sqrt{2\ln(2/\delta)/m}\right\} \ge 1 - \delta
> $$
> 
> $$
> P_{S\sim\mathcal{D}^m} \left\{\forall h \in \mathcal{H}, L_{\mathcal{D}}(ERM_{\mathcal{H}}(S)) - L_S(h) \le 2 R(l \circ \mathcal{H} \circ S) + 4(b - a) \sqrt{2\ln(2/\delta)/m}\right\} \ge 1 - \delta
> $$
> 

证明：
对于第一个不等式，直接使用McDiarmid's Inequality, 注意对应的不是$l(h(x), y)$的界而是是 $[(a - b)/m, (b - a)/m]$；

对于第二个不等式，

$$
\begin{cases}
    P_{S \sim \mathcal{D}^m} \left\{Rep_{\mathcal{D}}(\mathcal{F}, S) \le \mathbb{E}_{S'} Rep_{\mathcal{D}}(l \circ \mathcal{H} \circ S') + (b - a) \sqrt{2\ln(2/\delta)/m}\right\} \ge 1 - \delta/2;\\
    \mathbb{E}_{S \sim \mathcal{D}^m} Rep_\mathcal{D} (\mathcal{F}, S) \le 2 \mathbb{E}_{s' \sim \mathcal{D}^m}R(l \circ \mathcal{H} \circ S'); \\
    P_{S \sim \mathcal{D}^m} \left\{E_{S'} R(l \circ \mathcal{H} \circ S' \le R(l \circ \mathcal{H} \circ S) + (b - a) \sqrt{2 \ln(2/\delta) / m}\right\} \ge 1 - \delta/2;
\end{cases}
$$


对于第三个不等式, 首先定义 $h_S = ERM_{\mathcal{H}}(S)$，

$$
\begin{cases}
\forall h \in \mathcal{H}, L_\mathcal{D}(h_S) - L_{\mathcal{D}}(h) \le L_\mathcal{D}(h_S) - L_{S}(h_S) + L_S(h) - L_\mathcal{D}(h);\\
P_{S\sim\mathcal{D}^m}\left\{L_{\mathcal{D}}(h_S) - L_S(h_S) \le 2R(l\circ \mathcal{H} \circ S) + 3(b-a) \sqrt{2\ln(3/\delta)/m}\right\} \ge 1 - 2\delta/3;\\
P_{S\sim\mathcal{D}^m}\left\{L_{\mathcal{D}}(h) - L_S(h) \le (b-a) \sqrt{\ln(3/\delta)/(2m)}\right\} \ge 1 - 1\delta/3;(霍夫听不等式)
\end{cases}
$$

这三个不等式层层递进，其中第三个等式是我们最想得到的结果。

## 计算Rademacher复杂度

对于Rademacher复杂度，我们可以更抽象的认为是衡量一个向量集合的复杂度的：

$$
A \subset \mathbb{R}^m, R(A) = \frac{1}{m} \mathbb{E}_{\pmb{\sigma}} \left( \sup_{\pmb{a} \in A} \sum^m_{i=1} \sigma_i a_i \right). \tag{9}
$$

> 引理1：对于集合 $A$, 常数 $c$ 和 某个向量 $\mathbf{a}_0$，那么
> $R(\{c \mathbf{a} + \mathbf{a}_0:\mathbf{a} \in A\}) = \vert c \vert R(A).$

> 引理2：对于集合A，对应的一个集合 $A' = \{\sum^{N}_{j=1} \alpha_i \mathbf{a}^{(j)}: N \in \mathbb{N}, \forall j, \mathbf{a}^{(j)} \in A, \alpha_j \ge 0, \Vert \mathbf{\alpha} \Vert_1 = 1\}$, 那么 
> $R(A') = R(A)$。

> 引理3：(Massart Lemma). 如果 $A = \{\mathbf{a}_1, \mathbf{a}_2, \ldots, \mathbf{a}_n : \mathbf{a} \in \mathbb{R}^m\}$，定义 $\bar{\mathbf{a}} = \frac{1}{n} \sum^{n}_{i=1} \mathbf{a}_i$, 那么
>
> $$
> R(A) \le \max_{\mathbf{a} \in A} \Vert \mathbf{a} - \bar{\mathbf{a}}\Vert_2 \frac{\sqrt{2\ln(n)}}{m}.
> $$
>

证明：不失一般性，我们可以假设 $\bar{\mathbf{a}} = 0$, 并且 $A' = \lambda A(\lambda > 0)$, 那么

$$
\begin{align*}
    &m R(A') 
    = \mathbb{E}_{\pmb{\sigma}} \left[ \max_{\mathbf{a} \in A'} \mathbf{\sigma}^T \mathbf{a} \right]
    = \mathbb{E}_{\pmb{\sigma}} \left[ \ln \left( \max_{\mathbf{a} \in A'} e^{\pmb{\sigma}^T \mathbf{a}} \right) \right]\\
    \le& \mathbb{E}_{\pmb{\sigma}} \left[ \ln \left( \sum_{\mathbf{a} \in A'} e^{\pmb{\sigma}^T \mathbf{a}} \right) \right]
    \le \ln\left[ \mathbb{E}_{\pmb{\sigma}} \left( \sum_{\mathbf{a} \in A'} e^{\pmb{\sigma}^T \mathbf{a}} \right) \right]\\
    =& \ln \sum_{\mathbf{a} \in A'} \prod^{m}_{i=1} \mathbb{E}_{\sigma_i}e^{\sigma_i a_i}
    = \ln \sum_{\mathbf{a} \in A'} \prod^{m}_{i=1} \frac{1}{2}(e^{a_i} + e^{-a_i}) \\
    \le& \ln \sum_{\mathbf{a} \in A'} \prod^{m}_{i=1} e^{a^2_i/2} = \ln \sum_{\mathcal{a} \in A'} e^{\Vert \mathbf{a}\Vert^2/2} \le \ln \left( n \max_{\mathbf{a} \in A'}e^{\Vert \mathbf{a}\Vert^2/2}  \right)\\
    =& \ln n + \max_{\mathbf{a} \in A'} {\Vert \mathbf{a}\Vert^2/2}
\end{align*}
$$

又因为 $R(A') = \lambda R(A)$, 所以

$$
mR(A) \le \frac{1}{\lambda} \left( \ln n + \max_{\mathbf{a} \in A} {\Vert \lambda\mathbf{a}\Vert^2/2} \right)
\le \sqrt{2 \ln n \cdot \max_{\mathbf{a} \in A} \Vert \mathbf{a}\Vert^2}.
$$

> 引理4：收缩性。
> 令 $\phi(\mathbf{a}) = (\phi_1(a_1), \phi_2(a_2), \ldots, \phi_m(a_m))$, 并且 $\phi_i$ 都是 $\rho$-Lipshcitz 连续函数，那么
> $R(\phi\circ A)  \le \rho R(A)$.

## 线性假设集的Rademacher复杂度

> 例2：假设集$\mathcal{H}_2 = \{h(\mathbf{x}) = \mathbf{w}^T \mathbf{x}:\Vert \mathbf{w} \Vert_2 \le 1\}$ 的Rademacher复杂度满足
> 
> $$
> R(\mathcal{H}_2 \circ S) \le \frac{\sqrt{ \sum^{m}_{i=1} \Vert \mathbf{x}_i \Vert^2_2}}{m}\le\max_i \frac{\Vert \mathbf{x_i} \Vert_2}{\sqrt{m}}
> $$

证明：

$$
\begin{align*}
    &m R(\mathcal{H} \circ S) \\
    =& \mathbb{E}_{\pmb{\sigma}} \left( \sup_{\mathbf{w}, \Vert \mathbf{w} \Vert_2 \le 1}  \sum^{m}_{i=1} \sigma_i \mathbf{w}^T \mathbf{x}_i\right) \\
    =& \mathbb{E}_{\pmb{\sigma}} \left( \big\Vert \sum^m_{i=1} \sigma_i \mathbf{x}_i \big\Vert_2 \right)
    \le \left( \mathbb{E}_{\pmb{\sigma}} \big\Vert \sum^m_{i=1} \sigma_i \mathbf{x}_i \big\Vert^2_2 \right)^{1/2} \\
    =& \left( \sum_{i \ne j} \mathbb{E}[\sigma_i \sigma_j] \mathbf{x}^T_i \mathbf{x}_j + \sum_{i} \mathbb{\mathbf{\sigma}^2_i}\Vert \mathbf{x}_i \Vert^2_2 \right)^{1/2} \\
    =& \sqrt{\sum^m_{i=1} \Vert \mathbf{x}_i \Vert^2_2}.
\end{align*}
$$

> 例3：假设集 $\mathcal{H}_1 = \{h(\mathbf{x}) = \mathbf{w}^T \mathbf{x}:\Vert \mathbf{w} \Vert_1 \le 1, \mathbf{x} \in \mathbb{R}^n\}$ 的Rademacher复杂度满足
> 
> $$ 
> R(\mathcal{H}_1 \circ S) \le \max_i \Vert \mathbf{x}_i \Vert_\infty \sqrt\frac{2 \ln(2n)}{m}.
> $$
>
这个证明有点巧妙，换句话说就是有点绕，大家小心了。
证明：
首先构造集合 $V = \{\mathbf{v}_1, \mathbf{v}_2, \ldots, \mathbf{v}_n, -\mathbf{v}_1, -\mathbf{v}_2, \ldots, -\mathbf{v}_n: \mathbf{v}_i = (x_{1,i}, x_{2, i}, \ldots, x_{m, i}) \in \mathbb{R}^m\}$.

$$
\begin{align*}
    &R(\mathcal{H}_1\circ S) \le \mathbb{E}_{\pmb{\sigma}} \left( \big\Vert \sum^m_{i=1} \sigma_i \mathbf{x}_i \big\Vert_\infty \right)\\
    =& \mathbb{E}_{\pmb{\sigma}}\left( \sup_j \vert \mathbf{v}^T_j \sigma \vert \right) = m R(V)\\
    \le& m \max_j \Vert \mathbf{v}_j \Vert_2 \frac{\sqrt{2\ln(2n)}}{m}\\
    \le& m \max_{i} \Vert \mathbf{x}_i \Vert_\infty \frac{\sqrt {2\ln(2n)}}{m}\\
    =& \max_{i} \Vert \mathbf{x}_i \Vert_\infty \sqrt\frac{ {2\ln(2n)}}{m}\\
\end{align*}
$$

## SVM算法的界

> 定理5：对于SVM要解决的问题，我们增加限定 $\Vert \mathbf{x} \Vert_2 \le R$ 以及假设集的限定 $\mathcal{H} = \{\mathbf{w}: \Vert\mathbf{w}\Vert \le B\}$, 样本点损失函数 $l:\mathcal{H} \times Z \rightarrow \mathbb{R}$, $l(a, y)$ 关于a是$\rho$-Lipschitz连续，并且 $l(a, y)\in[0, c]$, 那么
>
> $$
> P\left\{\forall \mathbf{w} \in \mathcal{H}, L_\mathcal{D}(\mathbf{w}) \le L_{S}(\mathcal{w}) + \frac{2 \rho BR}{\sqrt{m}} + c \sqrt\frac{2 \ln(1 / \delta)}{m}  \right\} \ge 1 - \delta.
> $$

证明：
其实我们只需要证明 $R(l \circ \mathcal{H}) \le \rho BR/\sqrt{m}$。前面的比例不变性、例子2证明了这些事情。

可以将 $\mathbf{x}$ 维度增加一维，那么Hard-SVM问题可以简化成

$$
\arg\min_{\mathbf{w}} \Vert\mathbf{w}\Vert^2_2,
\quad s.t. \quad \forall i, y_i \left\langle \mathbf{w}, \mathbf{x}_i \right\rangle \ge 1.
$$

如果存在一个 $\mathbf{w}_0$ 满足 $P_{(\mathbf{x}, y) \sim \mathcal{D}}\{y \left\langle \mathbf{w}_0, \mathbf{x} \right\rangle \ge 1\} = 1$, 并且 $\Vert \mathbf{x} \Vert \le R$，那么我们通过经验误差最小化求得的 $\mathbf{w}_S$ 满足

$$
P_{(\mathbf{x}, y) \sim \mathcal{D}} [y \ne sign(\mathbf{w}_S^T \mathbf{x})] \le 
P_{(\mathbf{x}, y) \sim \mathcal{D}} [l^{ramp}(\mathbf{w}^T_S \mathbf{x}, y)]\\
\le \frac{2R\Vert \mathbf{w}_0 \Vert_2}{\sqrt{m}}
+ (1 + R\Vert\mathbf{w}_0\Vert_2) \sqrt{\frac{2\ln(1/\delta)}{m}}. \tag{10}
$$

上面这个界还需要知道 $\mathbf{w}_0$, 最好能知道真实的最优分类器 $\mathbf{w}^*$ 但是这些都是不能得到的，我们需要一个新的界。这个界用到了结构风险最小化的思想。

对于正整数i，构造 $B_i = 2^8$, 以及对应假设集 $\mathcal{H}_i = \{\mathbf{w}: \Vert \mathbf{w} \Vert \le B_i\}$, 并且 $\delta_i = \frac{\delta}{2 i^2}$。那么，有

$$
P \left\{\forall \mathbf{w}\in \mathcal{H}_i, 
L_ \mathcal{D}( \mathbf{w})  \le L_S( \mathbf{w}) + \frac{2 B_i R}{\sqrt{m}} + \sqrt\frac{2 \ln(2 /\delta_i)}{m}\right\} \ge 1 - \delta_i.
$$

那么，和事件的概率是 $1 - \delta$。

对于任意一个 $\mathbf{w}$，它对应的假设集编号是 $i = \lceil \log_2(\Vert \mathbf{w} \Vert)\rceil$, $B_i\le 2\Vert \mathbf{w} \Vert$, 并且 $\frac{2}{\delta_i} = \frac{(2i)^2}{\delta} \le \frac{(4 \log_2(\Vert \mathbf{w} \Vert))^2}{\delta}$, 那么

$$
L_\mathcal{D}(\mathbf{w}) \le L_S(\mathbf{w}) 
+ \frac{4 \Vert \mathbf{w} \Vert R}{\sqrt{m}} + 
\sqrt\frac{4[\ln(4\log_2(\Vert \mathbf{w} \Vert)) + \ln(1 / \delta)]}{m}.
$$
