# Game Theory, Online Learning and Boosting

!!!Abstract
    本文摘自`Boosting: Foundations and algorithms`的第六章。这一章介绍了Game Theory、Online Learning 和 Boosting 之间的关系，非常有意思。GAN和RL的相关算法实际上也是在求解一个博弈游戏，所以这一章的思想可能非常具有启发性。

## Game Theory

!!!定义
    对于一个 **博弈游戏** $\mathbf{M}$矩阵，两个玩家可以选择它的行$i$和列$j$，得到一个游戏数值$\mathbf{M}(i, j)$。当两个玩家选择两个分布$P$和$Q$后，会得到一个游戏数值

    $$
        M(P, Q) = \sum_{i,j} P(i)\mathbf{M}(i, j)Q(j) = P^T \mathbf{M} Q.
    $$

!!!定义
    对于一个博弈游戏$\mathbf{M}$，它的 **轮次玩法** (sequential play)则对应两个游戏目的

    $$
        \min_{P}\max_{Q} \mathbf{M}(P, Q)
    $$

    和

    $$
        \max_{Q}\min_{P} \mathbf{M}(P, Q).
    $$

冯诺依曼的`Minmax定理`证明了：在上述的矩阵情况下，这两个游戏的结果是等价的。但是，当 $\mathbf{M}$ 未知或者太大的时候，我们该如何求解最优策略 $P^*$ 和 $Q^*$ 就成了一个问题。我们希望通过 **不断地玩游戏** 来求解最优策略（Learning in Repeated Game Playing）。

!!!定义
    对于一个博弈游戏$\mathbf{M}$，它的 **重复玩法** （repeated play）设定如下。博弈游戏$\mathbf{M}$未知，但是满足$\mathbf{M}(i, j) \in [0, 1]$。并且存在两个玩家（学习者learner和环境environment）。游戏被分为$t = 1, \ldots, T$轮次，每个轮次可以做一下事情：

    1. 学习者选择一个混合策略$P_t$；
    2. 环境在可能已知$P_t$的情况下选择一个$Q_t$；
    3. 学习者只能观测到损失$\mathbf{M}(i, Q_t)$；
    4. 学习者的损失是$\mathbf{M}(P_t, Q_t);

    学习者的累计损失为

    $$
    \sum^{T}_{t=1} \mathbf{M}(P_t, Q_t),
    $$

    并且它的目标是最小化

    $$
    \min_{P} \sum^{T}_{t=1} \mathbf{M}(P, Q_t).
    $$

    即最小化事后诸葛亮（hindsight）损失。

介绍一个乘子权重法（multiplicative weights, MW）来求解这个问题。

!!!算法
    乘子权重算法（multiplicative weights, MW）分为两个部分来不断更新$P_t$：

    $$
        P_{t+1} = \arg\min_{P} \sum_i P(i) \mathbf{M}(i, Q_t) + \frac{1}{\eta} D_{KL}(P \Vert P_t).
    $$

    即
    $$
        P_{t+1}(i) = \frac{P_t(i) \exp(-\eta \mathbf{M}(i, Q_t))}{Z_t},
    $$

    其中

    $$
        Z_t = \sum^{m}_{t=1} P_t(i) \exp(-\eta \mathbf{M}(i, Q_t)).
    $$

这种算法下获得的$\{P_t\}^{T}_{t=1}$满足如下定理。

!!!定理
    MW算法获得的混合策略满足

    $$
    \sum^{T}_{t=1}\mathbf{M}(P_t, Q_t) \le \min_P \bigg[a_{\eta} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + c_\eta D_{KL}(P \Vert P_1)\bigg],
    $$

    其中 $a_\eta = \frac{\eta}{1 - e^{-\eta}}$，
    $c_\eta = \frac{1}{1 - e^{-\eta}}$。
    
???证明
    首先是一个引理：对于任意的随机决策$P$满足
    
    $$
    D_{KL}(P \Vert P_{t+1}) - D_{KL}(P \Vert P_t) \le \eta \mathbf{M}(P, Q_t) - (1 - e^{-\eta}) \mathbf{M}(P_t, Q_t).
    $$

    这个引理证明很简答

    $$
    \begin{aligned}
    & D_{KL}(P \Vert P_{t+1}) - D_{KL}(P \Vert P_t) \\
    =& \sum_i P(i) \ln \frac{P(i)}{P_{t+1}(i)} - \sum_i P(i) \ln \frac{P(i)}{P_t(i)} \\
    =& \sum_i P(i) \ln \frac{P_t(i)}{P_{t+1}(i)} \\
    =& \sum_i P(i) \ln \frac{Z_t}{\exp(-\eta \mathbf{M}(i, Q_t))} \\
    =& \eta \sum_{i} P(i) \mathbf{M}(i, Q_t) + \ln \bigg[\sum_i P_t(i) \exp(-\eta \mathbf{M}(i, Q_t))\bigg] \\
    \le& \eta \mathbf{M}(P, Q_t) + \ln \bigg[\sum_i P_t(i)(1 - (1 - e^{-\eta})\mathbf{M}(i, Q_t))\bigg] \\
    =& \eta \mathbf{M}(P, Q_t) + \ln \bigg[(1 - (1 - e^{-\eta})\mathbf{M}(P_t, Q_t))\bigg] \\
    \le& \eta \mathbf{M}(P, Q_t) - (1 - e^{-\eta}) \mathbf{M}(P_t, Q_t).
    \end{aligned}
    $$

    其中用到了不等式

    $$
    e^{-\eta q} = e^{q(-\eta) + (1 - q)\cdot 0} \le q e^{-\eta} + (1 - q) e^0 = 1 - (1 - e^{-\eta}) q
    $$

    和对于$x < 1$，

    $$
    \ln(1 - x) \le -x.
    $$

    那么，将$t=1\ldots, T$所有项累加之后，可得

    $$
    (1 - e^{-\eta}) \sum^{T}_{t=1} \mathbf{M}(P_t, Q_t) \le 
    \eta \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + D_{KL}(P \Vert P_1) - D_{KL}(P\Vert P_{t+1}).
    $$
    
    因为$P$是任意的，并且$D_{KL}(P \Vert P_{t+1}) \ge 0$，所以可得上面定理的结论。

如果取$P_1$是均匀分布，那么

$$
\sum^{T}_{t=1}\mathbf{M}(P_t, Q_t) \le \min_P \bigg[a_{\eta} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + c_\eta \ln m\bigg],
$$

其中$m$是$\mathbf{M}$的行数。

因为当$\eta \le 0$时，$1 - \eta \le e^{-\eta}$，所以$a_{\eta} \ge 1$。当$\eta$趋向0的时候，$c_{\eta}$又趋向于无穷大。所以目前我们无法保证这个Bound是有意义的，要进一步地分析。

!!!推论
    当$\eta$取

    $$
    \ln \bigg(1 + \sqrt{\frac{2\ln m}{T}}\bigg),
    $$

    可以保证

    $$
    \frac{1}{T} \sum^{T}_{t=1} \mathbf{M}(P_t, Q_t) \le
    \min_{P} \frac{1}{T} \sum^T_{t=1} \mathbf{M}(P, Q_t) + \sqrt{\frac{2\ln m}{T}} + \frac{\ln m}{T}.
    $$

???证明
    
    $$
    \begin{aligned}
    &\sum^{T}_{t=1}\mathbf{M}(P_t, Q_t) \\
    =& \min_{P} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + (a_\eta - 1) T + c_\eta \ln m \quad(\mathbf{M} \in [0, 1])\\
    =& \min_{P} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + \bigg[\bigg(\frac{\eta}{1 - e^{-\eta}} - 1 \bigg)T + \frac{\ln m}{1 - e^{-\eta}}\bigg]\\
    \le& \min_{P} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + \bigg[\bigg(\frac{e^{\eta} - e^{-\eta}}{2(1 - e^{-\eta})} - 1 \bigg)T + \frac{\ln m}{1 - e^{-\eta}}\bigg] \quad(\eta \le (e^{\eta}- e^{-\eta})/2)\\
    =& \min_{P} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + \bigg[\frac{e^{\eta} - 1}{2} T + \frac{\ln m}{e^{\eta} - 1} + \ln m\bigg] \\
    \le& \min_{P} \sum^{T}_{t=1} \mathbf{M}(P, Q_t) + \sqrt{\frac{2\ln m}{T}} + \frac{\ln m}{T}. \quad \bigg(\eta = \ln\bigg(1 + \sqrt{\frac{2\ln m}{T}}\bigg)\bigg)\\
    \end{aligned}
    $$

!!!推论

    $$
    \frac{1}{T} \sum^{T}_{t=1} \mathbf{M}(P_t, Q_t) \le 
    \min_{P}\max_{Q} \mathbf{M}(P, Q) + \Delta_T,
    $$

    其中$\Delta_T = \sqrt{\frac{\ln m}{T}} + \frac{\ln m}{T}$.

从上面的引理就能得到冯诺依曼的Minmax定理

!!!定理

    $$
    \min_{P}\max_{Q} \mathbf{M}(P, Q) = \max_{Q} \min_{P} \mathbf{M}(P, Q).
    $$

!!!证明
    首先我们证明 $\min_{P} \max_{Q} \mathbf{M}(P, Q) \ge \max_{Q} \min_{P} \mathbf{M}(P, Q)$。因为对于任意的$\hat P$和$Q$ 满足 $\mathbf{M}(\hat P, Q) \ge \min_{P} \mathbf{M}(P, Q)$。所以$\max_{Q} \mathbf{M}(\hat P, Q) \ge \max_{Q} \min_{P} \mathbf{M}(P, Q)$。因为$\hat P$是任意的，所以我们得到了本段开头的结论。

    难度在于如何求证$\min_{P} \max_{Q} \mathbf{M}(P, Q) \le \max_{Q} \min_{P} \mathbf{M}(P, Q)$。

    设MW算法中，环境获得的$Q_t = \arg\max_{Q} \mathbf{M}(P_t, Q)$。同时，设$\bar P = \frac{1}{T} \sum^{T}_{t=1} P_t$ 和 $\bar Q = \frac{1}{T} \sum^{T}_{t=1} Q_t$。那么

    $$
    \begin{aligned}
        &\min_{P}\max_{Q} P^T \mathbf{M} Q \le \max_{Q} \bar P^T \mathbf{M} Q \\
        =& \max_{Q} \frac{1}{T} \sum^T_{t=1} P^T_t \mathbf{M} Q 
        \le \frac{1}{T} \sum^T_{t=1}\max_Q P^T_t \mathbf{M} Q \\
        =& \frac{1}{T} \sum^T_{t=1} P^T_t \mathbf{M} Q_t
        \le \min_{P} \frac{1}{T} \sum^T_{t=1} P^T \mathbf{M} Q_t + \Delta_T \\
        =& \min_P P^T \mathbf{M} \bar Q + \Delta_T \le \max_Q \min_P P^T \mathbf{M}Q + \Delta_T.
    \end{aligned}
    $$

从上面的证明中可知MW算法能够获得一个不错的策略$\bar P = \frac{1}{T} \sum^T_{t=1} P_t$ 能够保证

$$
    \max_{Q} \mathbf{M}(\bar P, Q) \le \max_{Q} \min_{P} \mathbf{M}(P, Q) + \Delta_T.
$$

同理$\bar Q = \frac{1}{T} \sum^T_{t=1} Q_t$ 满足 

$$
    \min_{P} \mathbf{M}(P, \bar Q) \ge \min_{P} \max_{Q} \mathbf{M}(P, Q) - \Delta_T.
$$

## Online Learning

在一般的机器学习算法中，数据是从训练集中随机采样出来的`batch`，算法借此学习到一个模型后，将在测试集中进行训练。而在在线学习中，整个数据模型发生了改变。样本是一个个依次输入到算法中的，然后算法给出一个预测标签，接着算法接受一个真实标签，这时预测标签和真实标签的误差既是训练损失，也是测试损失。在线学习和其他机器学习最大的不同就是算法对样本的来源没有任何限制，它既可以是随机的，也可以是全知全能的敌对数据（adversarial)。

!!!定义
    在线学习的整体模型如下所示。设样本在有限集合$\mathcal{X}$中，以及一个针对二分类的有限假设集$\mathcal{H} = \{h: \mathcal{X} \rightarrow \{-1, 1\}\}$。并且假设样本服从一个未知的标签函数$c:\mathcal{X} \rightarrow \{-1, 1\}$。（这个目标可以是任意的标签函数，是随机分布也没有关系，这里假设为一个确定性的标签函数是为了简化后面的过程。）接着

    1. 环境以任意方式给学习者选定一个样本$x_t \in \mathcal{X}$；
    2. 学习者选定一个针对$x_t$的估计标签$\hat y_t \in \{-1, 1\}$；
    3. 学习者观测到了真实标签$c(x_t)$；
    
    经验累计误差为

    $$
        \mathbb{E}\bigg[\sum^{T}_{t=1} \mathbf{1}[\hat y_t \ne c(x_t)]\bigg]
        = \sum^T_{t=1} \mathbf{Pr}[\hat y_t \ne c(x_t)].
    $$

    在线学习的目标是最小化事后诸葛亮累计误差

    $$
        \min_{h \in \mathcal{H}} \sum^T_{t=1} \mathbf{1}[h(x_t) \ne c(x_t)].
    $$

将在线学习转化为博弈论问题：$\mathbf{M}$是一个$\vert \mathcal{H} \vert$行，$\vert \mathcal{X} \vert$列的在线博弈问题，满足

$$
\mathbf{M}(h, x) = \mathbf{1}[h(x) \ne c(x)].
$$

如果使用MW算法来求解Online Learning问题，那么就得到了一个求解Online Learning问题的算法`Weighted Majority Algorithm`。

$$
P_{t+1}(h) = \frac{P_t(h) e^{-\eta \cdot \mathbf{1}[h(x_t) \ne c(x_t)]}}{Z_t}.
$$

这个算法可以保证收敛率为 $O \bigg(\sqrt{\frac{\ln \vert \mathcal{H} \vert}{T}}\bigg)$.

## Boosting

Boosting的算法流程是：

1. Booster构造一个基于训练集$X$的分布$\mathcal{D}_t$给弱学习器；
2. 弱学习器提供一个分类函数$h_t \in \mathcal{H}$满足经验误差小于$\frac{1}{2} - \gamma$，即$\mathbf{Pr}_{x \sim \mathcal{D}_t} [h_t (x) \ne c(x)] \le \frac{1}{2} - \gamma$。

AdaBoost提供了一个算法来构造$\mathcal{D}_t$以及组合$\{h_t\}^T_{t=1}$的方式，使得最终的分类器达到理想效果。

将Boost转化为博弈游戏$\mathbf{M}(h, x) = \mathbf{1}[h(x) \ne c(x)]$。

那么，根据冯诺依曼的Minmax定理，可得

$$
\begin{aligned}
&\min_{P} \max_{x \in X} \mathbf{M}(P, x) \\
=& \min_{P} \max_{Q} \mathbf{M}(P, Q)
= \nu \\
=& \max_{Q} \min_{P} \mathbf{M}(P, Q) \\
=& \max_{Q} \min_{h \in \mathcal{H}} \mathbf{M}(h, Q)
\end{aligned}
$$

根据定义

$$
\mathbf{M}(h, Q) = \mathbf{Pr}_{x \sim Q} [h(x) \ne c(x)],
$$

所以

$$
\nu \le \mathbf{M}(h, Q^*) = \mathbf{Pr}_{x \sim Q^*}[h(x) \ne c(x)] \le \frac{1}{2} - \gamma.
$$

又因为Adaboost开头的$\gamma$-弱学习假设可知

$$
\mathbf{Pr}_{h \sim P^*}[h(x) \ne c(x)] = \mathbf{M}(P^*, x) \le \nu \le \frac{1}{2} - \gamma < \frac{1}{2}.
$$

也就是说，在样本集的样本可以被如下加权分类器完美分类

$$
c(x) = sign\bigg(\sum_{h \in \mathcal{H}} P^*(h) h(x)\bigg).
$$

从minimax游戏可以看出来，这个分类器的margin至少是$2\gamma$，并且edge和margin的关系一目了然。

Adaboost的原理是求解问题

$$
    \max_{Q} \min_{h \in \mathcal{H}} \mathbf{M}(h, Q),
$$

所以Boosting要求解一个$Q^*$分部而不再是$P^*$分部。我们如果要使用MW算法我们需要对问题$\mathbf{M}$做修改，而使用对偶游戏$\mathbf{M}'$

$$
\mathbf{M}' = \mathbf{1} - \mathbf{M}^T.
$$

那么，Boosting算法的每一步都要完成如下步骤：

1. 将分布$Q_t = \mathcal{D}_t$传给弱学习器；
2. 获得弱学习器返回的分类函数$h_t$满足$\mathbf{Pr}_{x\sim\mathcal{D}_t}[h_t(x) = c(x)] \ge \frac{1}{2}+\gamma$；
3. 获得与$Q_t$期望等价的分类函数$h_t$：$\mathbf{M}'(x, Q_t) = \mathbf{M}'(x, h_t) = \mathbf{1}\{h_t(x) = c(x)\}$；
4. 计算新的分布$Q_{t+1} = {Q_t e^{-\eta \mathbf{1}\{h_t(x) = c(x)\}}}/{Z_t}$。

算法最终返回的分类器为

$$
H(x) = sign\bigg(\sum^T_{t=1} h_t(x)\bigg).
$$

这个算法也被称为 $\epsilon$-boosting 或者 $\epsilon$-AdaBoost。

根据前面对博弈游戏的分析我们可得

$$
\frac{1}{2} + \gamma \le \frac{1}{T} \sum^T_{t=1} \mathbf{M}'(P_t, h_t) \le \min_{x \in X} \frac{1}{T} \sum^T_{t=1} \mathbf{M}'(x, h_t) + \Delta_T,
$$

也就是说

$$
\min_{x \in X} \frac{1}{T} \sum^T_{t=1} \mathbf{M}'(x, h_t) \ge \frac{1}{2} + \gamma - \Delta_T
$$

即当$T$够大时，分类器能够拟合目标函数$c(x)$。

另一个关于博弈游戏的结论可得

$$
\frac{1}{2} + \gamma \le \frac{1}{T} \sum^T_{t=1}\mathbf{M}'(P_t, h_t) \le a_{\eta} \min_{x \in X} \frac{1}{T} \sum^T_{t=1} \mathbf{M}'(x, h_t) + \frac{c_{\eta}\ln m}{T}.
$$

设 $\eta = 2 \alpha$，可得

$$
\begin{aligned}
&\min_{x \in X} \frac{1}{T} \sum^T_{t=1}\mathbf{M}'(x, h_t) \ge \frac{1}{a_{\eta}}
\bigg[\frac{1}{2}+\gamma - \frac{c_{\eta}\ln m}{T}\bigg]\\
=& \frac{1}{2} + \gamma - \bigg(1 - \frac{1 - e^{-2\alpha}}{2\alpha}\bigg)\bigg(\frac{1}{2} + \gamma\bigg) - \frac{\ln m}{2\alpha T}\\
\ge& \bigg(\frac{1}{2} + \gamma\bigg) - \alpha\bigg(\frac{1}{2} + \gamma\bigg) - \frac{\ln m}{2\alpha T}. \quad(e^{-z} \le 1 - z + z^2/2)
\end{aligned}
$$

当$\alpha$足够小，并且$T$足够大时，$\alpha$-Adaboost能够达到AdaBoost的性能。

```bib
@article{schapire2013boosting,
  title={Boosting: Foundations and algorithms},
  author={Schapire, Robert E and Freund, Yoav},
  journal={Kybernetes},
  year={2013},
  publisher={Emerald Group Publishing Limited}
}
```