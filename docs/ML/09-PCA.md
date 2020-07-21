# PAC Learning Framework

概率性地近似地正确的(PAC, Probably Approximately Correct)学习框架是非常重要的框架，
它尝试解决一些本质性的问题：什么学习是有效的？什么学习是困难的？我们需要多少的样本？有没有普遍适用的模型？

很多人粗暴地把这些归结为''大数定理''，其实也没什么不对，不过我们希望掀开''大数定理''的帘子，再进一步分析一下。

## 问题描述
首先定义样本空间 $\{\mathcal{X}, \mathcal{Y}, \mathcal{Z} = \mathcal{X} \times \mathcal{Y}\}$, 
我们通常认为样本服从某个未知分布，即 $\mathcal{Z} \sim \mathcal{D}$。监督学习的大部分问题正是求解对应的一个条件分布 $P_{\mathcal{D}}(\mathbf{y} \vert \mathbf{x})$。当我们已知 $\mathbf{x}$, 条件分布 $P_{\mathcal{D}}(\mathbf{y} \vert \mathbf{x})$ 可以帮助我们推断 $\mathbf{y}$ 的值。我们用符号 $\mathcal{Z}_{\mathcal{D}}$ 来代表样本空间 $\mathcal{Z}$ 加上概率分布 $\mathcal{D}$，我们就称它为 **样本测度空间**。

在统计机器学习中，我们只能够采样来窥视样本测度空间 $\mathcal{Z}_{\mathcal{D}}$ 的样子, 我们通常把独立同分布的样本组成的合集称为样本集:

$$
    \tag{1} \label{samples}
    \mathcal{S} = \{(\mathbf{x}_1, \mathbf{y}_1), (\mathbf{x}_2, \mathbf{y}_2), \ldots, (\mathbf{x}_m, \mathbf{y}_m)\}.
$$

为了求解 $P_{\mathcal{D}}(\mathbf{y}\vert \mathbf{x})$，我们会先构造一个假设集:

$$
    \tag{2} \label{hypothesisSet}
    \mathcal{H}_{\theta} = \left\{ P_{\theta}(\mathbf{y} \vert \mathbf{x}) \right\}.
$$

这个假设集包含了我们的先验知识，并且可以用数学公式表示，使得我们能够用数学工具来求解。

接着，我们会再使用一些度量概率分布之间距离的函数来构造损失函数，那么绝大部分监督学习需要解决的问题就变成了下面这个数学问题：

$$
    \tag{3} \label{generalObjective}
    \min_{\theta} Distance(p_{\mathcal{D}}(\mathbf{y} \vert \mathbf{x}) \Vert p_{\theta}(\mathbf{y} \vert \mathbf{x})).
$$

比如，当样本空间 $\mathcal{Y} = \{0, 1\}$ 时，我们称这个问题为 **二分类问题**，对于这个问题，我们可以用 KL 距离来度量两个分布的差别：

$$
    \tag{4} \label{classificationObjective}
    \min_{\theta} L_{\mathcal{D}}(\theta) = \mathbb{E}_{\mathbf{x} \sim p_{\mathcal{D}}(\mathbf{x})} 
    \left\{ 
        \mathbb{E}_{{y} \sim P_{\mathcal{D}}({y} \vert \mathbf{x})}
        \left[ 
            \ln \left(
            \frac{P_{\mathcal{D}}({y}\vert \mathbf{x})}{P_{\theta}({y}\vert \mathbf{x})}
            \right) 
        \right] 
    \right\}.
$$

这个损失函数要求 $P_{\theta}(\mathbf{y} \vert \mathbf{x})$ 在 $y = 0$ 和 $y = 1$ 上的概率都大于0，我们可以用 Softmax 函数来保证。

我们也可以构造一个更简单的假设集，只使用退化的概率分布，即 $P_{\theta}(y = 1 \vert \mathbf{x}) = 1$ 或者 $P_{\theta}(y = 0 \vert \mathbf{x}) = 1$，我们表示为 $\mathcal{H}_{\theta} = \left\{ h_{\theta}(\mathbf{x}) \right\}$，这里的 $h_\theta(\mathbf{x})$ 表示概率为1的 $y$ 值。
那么，我们也可以用 Wasserstein 距离来衡量来构造损失函数，其中花费函数为 $c(a, b) = \pmb{1}_{\left\{ a \ne b \right\}}$:

$$
    \tag{5} \label{classificationObjective2}
    \min_{\theta} L_{\mathcal{D}}(\theta) = \mathbb{E}_{\mathbf{x} \sim p_{\mathcal{D}}(\mathbf{x})} 
    \left\{ 
        \mathbb{E}_{{y} \sim P_{\mathcal{D}}({y} \vert \mathbf{x})}
        \left[ 
            c(y, h(\mathbf{x})
        \right] 
    \right\}.
$$

我们很容易可以得到化简的损失函数 $L_{\mathcal{D}}(\theta) = \mathbb{E}_{(\mathbf{x}, y) \sim \mathcal{D}}[c(y, h(\mathbf{x})]$。

再回到本小节开头我们说的，我们只能获得一个样本集 $\mathcal{S} = \{(\mathbf{x}_1, y_1), (\mathbf{x}_2, y_2), \ldots, (\mathbf{x}_m, {y}_m)\}$, 所以我们也只能优化经验损失函数：

$$
    \tag{6} \label{classificationEmpiricalLoss}
    \min_{\theta} L_{\mathcal{S}}(\theta) = \frac{1}{m} \sum^{m}_{i=1} c(y_i, h_{\theta}(\mathbf{x}_i)).
$$

另外，我们把最小化经验误差函数的原则称为 **经验风险最小化** (ERM, Empirical Risk Minimization)。
终于进入我们关心的问题，ERM学出来的 $h_S$ 究竟有多好？能不能定量分析呢？

## PAC

!!!定义1
    (**PAC Learnability**).
    如果假设集是PAC Learnable的，则表示存在函数 $m_{\mathcal{H}}(\epsilon, \delta)$, 和某个学习算法 $A$ 满足：
    当样本集 $\mathcal{S}$ 大小大于 $m_{\mathcal{H}}(\epsilon, \delta)$ 时，
    
    $$
        \tag{7} \label{pacLearnability}
        \mathcal{P}_{\mathcal{S} \sim \mathcal{D}} \left\{ 
            L_{\mathcal{D}}(A(\mathcal{S})) \le 
            \min_{\theta} L_{\mathcal{D}}(\theta) + \epsilon
        \right\} \ge 1 - \delta.
    $$
    
    其中 $\epsilon$ 表示精度，$\delta$ 表示置信度。

    需要备注的是： $L_{\mathcal{D}}(\theta)$ 大于等于0；
    如果 $\min_{\theta} L_{\mathcal{D}}(\theta) > 0$，那么我们称 $\mathcal{H}$ 是 **Agnostic PAC Learnable** 的。
    Agnostic 这个词是想说，我们构造的假设集不够充分，真实分布 $\mathcal{D}$ 超出了 $\mathcal{H}$ 的认知。

我们可以先在一个非常非常特殊的例子做分析。

对于二分类问题，假设集是 $\mathcal{H}_{\theta} = \{h_{\theta_1}(\mathbf{x}), h_{\theta_2}(\mathbf{x}), \ldots, h_{\theta_n}(\mathbf{x})\}$，并且存在 $h_{\theta^*}$ 满足 $L_{\mathcal{D}}(\theta^*) = 0$ (The Realizability Assumption)。那么，任意给定一个样本集 $\mathcal{S}$，我们会发现至少有一个 $h_i(\theta_i)$ 满足 $L_{\mathcal{S}}(\theta_i) = 0$，并且我们只会从中随便挑选一个分类函数。接下来，我就要证明，随便挑，挑到''最差''的也没关系。

首先从假设集 $\mathcal{H}$ 中分离出一个坏子集 $\mathcal{H}_{bad} = \{h \in \mathcal{H}, L_{\mathcal{D}}(h) > \epsilon\}$。
我们再介绍一个坏的样本集的集合 $M_{bad} = \{S \sim \mathcal{D}: \exists h \in \mathcal{H}_{bad}, L_{S}(h) = 0\} = \cup_{h\in\mathcal{H}_{bad}} \{ \mathcal{S}: L_{\mathcal{S}}(h) = 0 \}$。那么，


$$
    \tag{8} \label{step1}
    \mathcal{P}_{S\sim\mathcal{D}}(\{S: L_{\mathcal{D}}(h_S) \ge \epsilon\})
    \le \mathcal{P}_{S\sim\mathcal{D}}(M_{bad})
    \le \sum^{}_{h \in \mathcal{H}_{bad}} \mathcal{P}_{S\sim\mathcal{D}}(\{S: L_{S}(h) = 0\}).
$$

对于任意的 $h \in \mathcal{H}_{bad}$，我们可得


\begin{align*}
    \tag{9} \label{step2}
    &\mathcal{P}_{S\sim\mathcal{D}}(\{S: L_{S}(h) = 0\})\\
    =& \prod^{m}_{i=1} \mathcal{P}_{(\mathbf{x}_i, y_i)\sim\mathcal{D}}(c(y_i, h(\mathbf{x}_i)))\\
    \le& (1 - \epsilon)^{m} \le e^{-\epsilon m}.
\end{align*}

带入 $\eqref{step1}$ 可得


\begin{align*}
    \tag{10} \label{step3}
    &\mathcal{P}_{S\sim\mathcal{D}}(\{S: L_{\mathcal{D}}(h_S) \ge \epsilon\})\\
    \le& \vert \mathcal{H}_{B} \vert \mathcal{P}_{S\sim\mathcal{D}}(\{S: L_{S}(h) = 0\})\\
    \le& \vert \mathcal{H} \vert e^{-\epsilon m} = n e^{-\epsilon m}.
\end{align*}

换句话说，在这个例子里，采样复杂度为 $m \ge \frac{\ln(n/\delta)}{\epsilon}$。

!!! note
    (**Bayes Optimal Predictor**).
    对于上文提到的二分类问题，是存在最优的deterministc分类器，我们叫它贝叶斯最优分类器：

    $$
    f_{\mathcal{D}}(\mathbf{x}) = 
    \begin{cases}
        1, &\mathcal{P}_{D}[y = 1 \vert \mathbf{x}] \ge 1/2;\\
        0, &otherwise.
    \end{cases}
    $$

## Uniform Convergence

!!! 定义2
    ($\pmb{\epsilon}$-**representative sample**). 我们评价一个样本集是 $\epsilon$-representative sample，
    指的是：
    
    $$
        \tag{11} \label{representative}
        \forall h \in \mathcal{H}, \vert L_{S}(h) - L_{\mathcal{D}}(h) \vert \le \epsilon.
    $$

凭直觉，样本够多的时候，总能成为 $\epsilon$-representative 样本.

当样本是 $\epsilon/2$-representative，那么

$$
    \tag{12} 
    L_{\mathcal{D}}(ERM(S)) \le \min_{h \in \mathcal{H}} L_{\mathcal{D}}(h) + \epsilon.
$$

证明很简单：
对于任意的 $h \in \mathcal{H}$，

$$
    \tag{13} 
    L_{\mathcal{D}}(ERM(S)) \le L_{S}(ERM(S)) + \epsilon/2 \le L_{S}(h) + \epsilon/2 
    \le L_{\mathcal{D}}(h) + \epsilon.
$$

接下来我们可以介绍 Uniform Convergence 了。

!!! 定义3
    (**Uniform Convergence**).
    我们形容假设集 $\mathcal{H}$ 有 Uniform Convergence 性质，表示存在函数 $m^{UC}_{\mathcal{H}}(\delta, \epsilon)$，满足当样本数大于 $m^{UC}_{\mathcal{H}}(\delta, \epsilon)$ 时，

    $$
        \tag{14} \label{uniformConvergence}
        \mathcal{P}_{S \sim \mathcal{D}}
        \left\{ S\ is\ \epsilon\ representative \right\} \ge 1 - \delta.
    $$

很容易，我们知道PCA learnable的采样复杂度函数满足 $m^{PCA}_{\mathcal{H}}(\delta, \epsilon) \le m^{UC}_{\mathcal{H}}(\delta, \epsilon/2)$。

!!! 定理1
    (**Hoeffding's Inequality**)
    对于 $m$ 个随机变量 $\mathcal{X}_1 \in [a_1, b_1], \mathcal{X}_2 \in [a_2, b_2], \ldots, \mathcal{X}_{m} \in [a_m, b_m]$,
    它们的均值随机变量 $\bar{\mathcal{X}}$ 满足:

    $$
        \tag{15} \label{Hoeffding}
        \mathcal{P}\left\{ \vert \bar{\mathcal{X}} - \mathbb{E} \bar{\mathcal{X}} \vert \ge \epsilon \right\}
        \le 2 \exp\left\{ - \frac{2 m^2 \epsilon^2}{\sum^{m}_{i=1}(b_i - a_i)^2} \right\}.
    $$

我们发现在二分类问题的损失函数$\eqref{classificationEmpiricalLoss}$中，$c(y, h(\mathbf{x})) \in [0, 1]$ 可以看成随机变量，
那么 $L_{S}(h)$ 就是 $m$ 个随机变量的均值，并且 $L_{\mathcal{D}}(h)$ 是 $L_{S}(h)$ 的期望，我们可以直接带入霍夫听不等式，可得：

$$
    \tag{16}
    \forall h \in \mathcal{H},
    \mathcal{P}_{\mathcal{S} \sim \mathcal{D}} \left\{ \vert L_{\mathcal{D}}(h) - L_{S}(h) \vert \ge \epsilon \right\}
    \le 2 \exp\left\{ {-2 m \epsilon^2} \right\}.
$$

那么，可得

$$
    \tag{17}
    \mathcal{P}_{\mathcal{S} \sim \mathcal{D}} \left\{\exists h \in \mathcal{H},\vert L_{\mathcal{D}}(h) - L_{S}(h) \vert \ge \epsilon \right\}
    \le 2 \vert \mathcal{H} \vert \exp\left\{ {-2 m \epsilon^2} \right\}.
$$

这个结论有一定的泛化性，在很多其他问题中，只要损失函数对应于 $c(y, h(\mathbf{x}))$ 的项都在 $[0, 1]$ 范围内，都可以满足上面的结果。
整理一下，我们可以得到如下定理：

!!! 定理2
    假设集有限的话，它一定是 **agnostic PAC learnable**，并且
    
    $$
        \tag{18}
        m^{UC}_{\mathcal{H}} (\epsilon, \delta) \le
        \left\lceil
            \frac{\log(2\vert\mathcal{H}\vert / \delta)}{2\epsilon^2}
        \right\rceil,
        \quad
        m^{PAC}_{\mathcal{H}} (\epsilon, \delta) \le
        \left\lceil
            \frac{2\log(2\vert\mathcal{H}\vert / \delta)}{\epsilon^2}
        \right\rceil.
    $$
