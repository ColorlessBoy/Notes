# Generative Adversarial Imitation Learning

```bib
@article{ho2016generative,
    title={Generative Adversarial Imitation Learning},
    author={Ho, Jonathan and Ermon, Stefano},
    pages={4565--4573},
    year={2016}
}
```

[论文链接](https://arxiv.org/pdf/1606.03476.pdf)

!!!Abstract
    本文介绍了一个非常有名的模仿学习算法GAIL。我做这篇论文的笔记，主要是比较看中这篇文章对逆强化学习数学问题的描述与分析。虽然它本身的GAIL算法和GAN算法如出一辙，但是文章本身的分析对我理解逆强化学习问题帮助比较大。

## 定义符号

!!!定义
    （ **逆强化学习** ， Inverse Reinforcement Learning） 设马尔可夫决策问题$\mathcal{MDP}=\{\mathcal{S}, \mathcal{A}, p_0, p, \gamma, C\}$ （$C$是损失函数），以及专家策略$\pi_E$，那么逆强化学习需要求解如下问题：

    $$
    \tag{1}\label{equ:irl}
    \max_{c \in \mathcal{C}} \bigg(\min_{\pi \in \Pi} \rho(\pi)\bigg) - \rho(\pi_E).
    $$

    或者添加与策略的熵有关的正则项

    $$
    \tag{2}\label{equ:irl-entropy}
    \max_{c \in \mathcal{C}} \bigg(-\alpha H(\pi) + \min_{\pi \in \Pi} \rho(\pi) \bigg) - \rho(\pi_E).
    $$

    其中$H(\pi) = \mathbb{E}_{s \sim p_{\gamma}(\cdot; \pi)}[-\int_{a}\pi(a \vert s) \ln \pi(a \vert s) \mathrm{d} a ]$。

    或者添加与奖励函数有关的正则项

    $$
    \tag{3}\label{equ:irl-entropy-psi}
    \max_{c \in \mathcal{C}} -\psi(c) + \bigg(-\alpha H(\pi) + \min_{\pi \in \Pi} \rho(\pi) \bigg) - \rho(\pi_E).
    $$

    其中 $\psi(c)$ 是关于$c$的凸函数。

同时，我们定义符号

$$
\tag{4}
IRL_{\psi}(\pi_E) = \arg\max_{c \in \mathcal{C}} -\psi(c) + \bigg(-\alpha H(\pi) + \min_{\pi \in \Pi} \rho(\pi) \bigg) - \rho(\pi_E),
$$

以及

$$
\tag{5}
RL(c) = \arg\min_{\pi \in \Pi} -H(\pi) + \mathbb{E}_{\pi}[c(s, a)].
$$

## 推导逆强化学习的最优策略

首先回忆强化学习策略值函数为

$$
\begin{aligned}
\rho(\pi) =& \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi)}\bigg[(1 - \gamma) \sum^{\infty}_{t=0} \gamma^t c(s_t, a_t) \bigg]\\
=& \mathbb{E}_{s \sim p_{\gamma}(\cdot; \pi), a \sim \pi(\cdot \vert s; \pi)}[c(s, a)]\\
=& \sum_{s, a} p_{\gamma}(s, a; \pi) c(s, a).
\end{aligned}
\tag{6}
$$

其中 $p_{\gamma}(s; \pi) = (1 - \gamma) \sum^{\infty}_{t=0}\gamma^t p_t(s; \pi)$， 以及$p_{\gamma}(s, a; \pi) = p_{\gamma}(s; \pi) \pi(a \vert s)$。联合分布$p_{\gamma}(s, a; \pi)$也被叫做关于$\pi$的 **occupancy measures**。

!!!定理
    设两个集合

    $$
    \tag{7}
    \mathcal{D}_1 = \{p_{\gamma}(s, a; \pi) : \pi \in \Pi\},
    $$

    以及

    $$
    \tag{8}
    \mathcal{D}_2 = \bigg\{p(s, a):  \sum_a p(s, a) = (1 - \gamma) p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a') p(s', a')\bigg\}.
    $$

    那么 $\mathcal{D}_1$ 和 $\mathcal{D}_2$ 之间存在一个双射关系。

???证明

    1. 对于任意的$p_{\gamma}(s, a; \pi) \in \mathcal{D}_1$满足$p_{\gamma}(s, a; \pi) \in \mathcal{D}_2$， 即 $\mathcal{D}_1 \subseteq \mathcal{D}_2$。任意$p_{\gamma}(s, a; \pi) \in \mathcal{D}_1$， 可以被带入$\mathcal{D}_2$的条件

        $$
        \begin{aligned}
        & (1 - \gamma) p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a') p_{\gamma}(s, a; \pi) \\
        =& (1 - \gamma) p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a') (1 - \gamma) \sum^{\infty}_{t=0} \gamma^t p_t(s; \pi) \\
        =& (1 - \gamma) p_0(s) + (1 - \gamma) \sum^{\infty}_{t=0} \gamma^{t+1} p_{t+1}(s; \pi) \\
        =& (1 - \gamma) \sum^{\infty}_{t=0} \gamma^{t} p_{t}(s; \pi).
        \end{aligned}
        \tag{9}
        $$

    2. 接着证明$\mathcal{D}_2 \subseteq \mathcal{D}_1$。对于任意的$p(s, a) \in \mathcal{D}_2$， 人为定义和它相关的分布

        $$
        p(s) = \sum_{a} p(s, a),
        \tag{10}
        $$

        以及策略

        $$
        \pi_p(a \vert s) = \frac{p(s, a)}{\sum_{a'} p(s, a')}.
        \tag{11}
        $$

        那么

        $$
        p(s) = (1 - \gamma)p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a') \pi_p(a' \vert s') p(s').
        \tag{12}
        $$

        写成矩阵的形式可得

        $$
        \mathbf{p}^T = (1 - \gamma)\mathbf{p}^T_0 + \gamma \mathbf{p}^T \mathbf{P}^{\pi_p}.
        \tag{13}
        $$

        其中$\mathbf{P}^{\pi_p}_{ij} = \sum_{a'} p(s_j \vert s'_i, a') \pi(a' \vert s'_i)$。求解可得

        $$
        \mathbf{p}^T = (1 - \gamma)\mathbf{p}^T_0 (I - \gamma \mathbf{P}^{\pi_p})^{-1} = (1 - \gamma) \mathbf{p}^T_0 \sum^{\infty}_{t=0} (\gamma \mathbf{P}^{\pi_p})^{t}.
        \tag{13}
        $$

        写回到泛化形式可得

        $$
        p(s) = (1 - \gamma) \sum^{\infty}_{t=0} \gamma^t p_t(s; \pi_p) = p_{\gamma}(s; \pi_p).
        \tag{14}
        $$

!!!性质
    $\mathcal{D}_2$是一个凸集合。

???证明
    对于任意的$p_A(s, a) \in \mathcal{D}_2$和$p_B(s, a) \in \mathcal{D}_2$，

    $$
    \begin{aligned}
    &\sum_a [\lambda p_A(s, a) + (1 - \lambda) p_B(s, a)] \\
    =& \lambda \bigg[(1 - \gamma)p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a') p_A(s', a')\bigg] \\
    & + (1 - \lambda) \bigg[(1 - \gamma)p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a') p_B(s', a')\bigg] \\
    =& (1 - \gamma) p_0(s) + \gamma \sum_{s', a'} p(s \vert s', a')[\lambda p_A(s', a') + (1 - \lambda)p_B(s', a')].
    \end{aligned}
    $$

    所以 $\lambda p_A(s, a) + (1 - \lambda) p_B(s, a)] \in \mathcal{D}_2$。综上$\mathcal{D}_2$是一个凸集。

!!!定理
    如果$\psi(c)$是一个关于$c$的凸函数，并且$\mathcal{C}$是一个凸集，那么
    
    $$
    \begin{aligned}
    RL \circ IRL_{\psi}(\pi_E) =& \arg\min_{\pi\in\Pi}\max_{c \in \mathcal{C}} -\alpha H(\pi) + \sum_{s, a}[p_{\gamma}(s, a;\pi) - p_{\gamma}(s, a;\pi_E)]c(s, a) - \psi(c)\\
    =& \arg\min_{\pi\in\Pi} -\alpha H(\pi) + \psi^*(p_{\gamma,\pi} - p_{\gamma,\pi_E}).
    \end{aligned}
    \tag{15}
    $$

???证明
    首先原问题为
    
    $$
    \begin{aligned}
    RL \circ IRL_{\psi}(\pi_E) =& \arg\min_{\pi\in\Pi}\max_{c \in \mathcal{C}} -\psi(c) + \bigg(-\alpha H(\pi) + \min_{\pi \in \Pi} \rho(\pi) \bigg) - \rho(\pi_E)\\
    =& \arg\min_{\pi\in\Pi}\max_{c \in \mathcal{C}} \min_{\pi \in \Pi} -\alpha H(\pi) + \sum_{s, a} [p_{\gamma}(s, a; \pi) - p_{\gamma}(s, a; \pi_E)]c(s, a) -\psi(c)\\
    :=& \arg\min_{\pi\in\Pi}\max_{c \in \mathcal{C}} \min_{\pi \in \Pi} L(c, \pi). 
    \end{aligned}
    \tag{16}
    $$

    我们如果能够证明

    $$
    \max_{c \in \mathcal{C}} \min_{\pi \in \Pi} L(c, \pi) 
    = \min_{\pi \in \Pi} \max_{c \in \mathcal{C}} L(c, \pi),
    \tag{17}
    $$

    那么我们就完成了定理的证明。

    1. 第一步是构造一个等价问题

        $$
        \max_{c \in \mathcal{C}} \min_{p \in \mathcal{D}_2} L(c, p) =  -\alpha H(p) + \sum_{s, a} [p(s, a) - p_{\gamma}(s, a; \pi_E)]c(s, a) -\psi(c),\\
        \tag{18}
        $$

        其中

        $$
        H(p) = - \int_{s, a} p(s, a) \ln \frac{p(s, a)}{\sum_{a'} p(s, a')}\mathrm{d} s \mathrm{d}a.
        \tag{19}
        $$

        已知$p(s, a) \in \mathcal{D}_2$是策略$\pi_p(s, a) = p(s, a) / \sum_{a'}p(s, a')$的 **occupancy measure**。所以我们可以证明$H(p) = H(\pi_p)$，

        $$
        \begin{aligned}
        H(p) =& - \int_{s, a} p(s, a) \ln \frac{p(s, a)}{\sum_{a'} p(s, a')}\mathrm{d} s \mathrm{d}a \\
        =& - \int_{s, a} p_{\gamma}(s; \pi_p) \pi_p(a \vert s) \ln \pi_p(a \vert s) \mathrm{d} a \mathrm{d}s \\
        =& H(\pi_p).
        \end{aligned}
        $$

        所以

        $$
        L(c, \pi_p) = L(c, p).
        \tag{20}
        $$

        同时也就很容易得到结论

        $$
        \max_{c \in \mathcal{C}} \min_{\pi \in \Pi} L(c, \pi_p) = 
        \max_{c \in \mathcal{C}} \min_{p \in \mathcal{D}_2} L(c, p).
        \tag{21}
        $$

    2. 我们接着证明第二步强对偶性。

        $$
        \max_{c \in \mathcal{C}} \min_{p \in \mathcal{D}_2} L(c, p) =
        \min_{p \in \mathcal{D}_2} \max_{c \in \mathcal{C}} L(c, p).
        \tag{22}
        $$

        我们如果能够得到$L(c, p)$关于$c$是一个凹函数（concave），并且关于$p$是一个凸函数即可，再结合条件$\mathcal{C}$和$\mathcal{D}_2$都是凸集。我们可以利用鞍点的性质就可以得到强对偶性。

        所需要的条件只差一个$L(c, p)$关于$p$是凸函数的证明，即证明$-H(p)$是一个凸函数，

        $$
        -H(p) = \sum_{s, a} p(s, a) \ln \frac{p(s, a)}{\sum_{a'}p(s, a')}.
        \tag{23}
        $$

        !!!辅助定理

            $$
            \sum_{i} a_i \log \frac{a_i}{b_i} \ge \bigg(\sum_i a_i\bigg) \ln \frac{\sum_i a_i}{\sum_i b_i}.
            \tag{24}
            $$

        ???证明
            根据Jensen Inequality：设$\alpha_i \ge 0$，$\sum_i \alpha_i = 1$ 以及 $f$是凸函数，那么

            $$
                \sum_i \alpha_i f(t_i) \ge f\bigg(\sum_i \alpha_i t_i\bigg).
                \tag{25}
            $$

            令 $\alpha_i = \frac{b_i}{\sum_j b_j}$， $t_i = \frac{a_i}{b_i}$ 以及$f(t) = t \ln t$可得

            $$
            \sum_i \frac{b_i}{\sum_j b_j} \frac{a_i}{b_i} \ln \frac{a_i}{b_i}
            \ge \sum_{i} \frac{a_i}{\sum_j b_j} \ln \sum_i \frac{a_i}{\sum_j b_j}.
            \tag{26}
            $$

            化简可得引理。
        
        所以对于任意的$p_A(s, a)$和$p_B(s, a)$可得

        $$
        \begin{aligned}
        &H[\lambda p_A + (1 - \lambda)p_B]\\
        =&[\lambda p_A(s, a) + (1 - \lambda) p_B(s, a)] \ln \frac{\lambda p_A(s, a) + (1 - \lambda)p_B(s, a)}{\sum_{a'} [\lambda p_A(s, a') + (1 - \lambda) p_B(s, a')]} \\
        \le& \lambda p_A(s, a) \ln \frac{\lambda p_A(s, a)}{\sum_{a'}\lambda p_A(s, a)} + (1 - \lambda) p_B(s, a) \ln \frac{(1 - \lambda) p_B(s, a)}{\sum_{a'} (1 - \lambda) p_B(s, a)} \\
        =& \lambda H(p_A) + (1 - \lambda) H(p_B).
        \end{aligned}
        \tag{27}
        $$
    
    3. 再使用对偶空间可得

    $$
        \min_{p \in \mathcal{D}_2} \max_{c \in \mathcal{C}} L(c, p)
        = \min_{\pi \in \Pi} \max_{c \in \mathcal{C}} L(c, p).
        \tag{28}
    $$

    综合以上三个结论，我们得到强对偶性，即定理的结论。

!!!推论
    当$\psi(c)$是一个常数，并且$\mathcal{C} = \mathbb{R}^{\vert \mathcal{S} \times \mathcal{A}\vert}$时，$p^* = p_E$。

???证明
    当$\psi(c)$是一个常数时，求解问题变成

    $$
    \min_{p \in \mathcal{D}_2}\max_{c \in \mathcal{C}} -\alpha H(p) + \sum_{s, a}[p(s, a) - p_{\gamma}(s, a;\pi_E)]c(s, a).
    \tag{29}
    $$

    它的等价问题为

    $$
    \min_{p \in \mathcal{D}_2} -\alpha H(p),\quad s.t.\quad
    p(s, a) = p_{\gamma}(s, a;\pi_E), \forall (s, a) \in \mathcal{S} \times \mathcal{A}.
    \tag{30}
    $$

## GAIL

总结一下，到目前为止逆强化学习（模仿学习，Imitation Learning）所需要求解的问题为

$$
\min_{\pi \in \Pi} \max_{c \in \mathcal{C}} -\alpha H(\pi) + \sum_{s, a}[p_{\gamma, \pi}(s, a) - p_{\gamma, \pi_E}(s, a)]c(s, a) - \psi(c).
\tag{31}
$$

GAIL将其实例化为

$$
\min_{\pi \in \Pi} \max_{c \prec \pmb{0}} -\alpha H(\pi) + \sum_{s, a}[p_{\gamma, \pi}(s, a) - p_{\gamma, \pi_E}(s, a)]c(s, a) - \psi_{GA}(c),
\tag{32}
$$

其中

$$
\psi_{GA}(c) = \sum_{s, a} p_{\gamma, \pi_E}(s, a)[-c(s, a) - \log(1 - e^{c(s,a)})].
\tag{33}
$$

带入$\psi_{GA}$可以得到一个优化问题

$$
\min_{\pi \in \Pi} \max_{c \prec \pmb{0}} -\alpha H(\pi) + \sum_{s, a}[p_{\gamma, \pi}(s, a) c(s, a) + p_{\gamma, \pi_E}(s, a) \ln(1 - e^{c(s, a)})].
\tag{34}\label{equ:gail-problem1}
$$

关于$c$的优化问题的闭解为

$$
c^*(s, a) = \ln \frac{p_{\gamma}(s, a; \pi)}{p_{\gamma}(s, a; \pi) + p_{\gamma}(s, a; \pi_E)}.
\tag{35}
$$

那么，优化问题(\ref{equ:gail-problem1})就变成了

$$
\begin{aligned}
\min_{\pi \in \Pi} -\alpha H(\pi) + \sum_{s, a}\bigg[&p_{\gamma, \pi}(s, a) \ln \frac{p_{\gamma}(s, a; \pi)}{p_{\gamma}(s, a; \pi) + p_{\gamma}(s, a; \pi_E)} \\
+& p_{\gamma, \pi_E}(s, a) \ln \frac{p_{\gamma}(s, a; \pi_E)}{p_{\gamma}(s, a; \pi) + p_{\gamma}(s, a; \pi_E)}\bigg].
\end{aligned}
\tag{36}
$$

这个优化问题本质上是最小化分布$p_{\gamma}(s, a;\pi)$和$p_{\gamma}(s, a; \pi_E)$的Jensen-Shannon divergence。也就是

$$
D_{JS}(p_{\gamma,\pi} \Vert p_{\gamma, \pi_E}) := D_{KL}(p_{\gamma,\pi} \Vert (p_{\gamma,\pi} + p_{\gamma,\pi_E})/2) + D_{KL}(p_{\gamma, \pi_E} \Vert (p_{\gamma, \pi} + p_{\gamma, \pi_E})/2).
$$

我们使用一个函数$D_w(s, a)$来间接求解$c^*(s, a)$

$$
L(w) = \sum_{s, a}\bigg[p_{\gamma, \pi}(s, a) \ln D_w(s, a)
+ p_{\gamma, \pi_E}(s, a) \ln (1 - D_w(s, a))\bigg].
\tag{37}
$$