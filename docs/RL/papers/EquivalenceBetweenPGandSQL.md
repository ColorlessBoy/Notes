# Equivalence Between Policy Gradient and Soft Q-Learning

[论文链接](https://arxiv.org/abs/1704.06440)

这篇文章更像是Schulman的个人数学笔记，他这个人比较偏好数学推导。但是数学推导很容易让人陷入其中。人们往往最后发现强化学习的数学并不完备，而大受打击。这篇文章是挂在Arxiv上的小文章，尝试揭示Policy Gradient和Soft Q-Learning的关系，算是有一定的启发性。 我按照个人的习惯进行了重新的推导。

## 熵正则化强化学习

首先，原始的强化学习问题是为了求解

$$
  \begin{aligned}
  \max_{\pi} \rho(\pi) =& (1 - \gamma) \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi)} \bigg[\sum^{\infty}_{t=0} \gamma^t r_t \bigg] \\
  =& \mathbb{E}_{s \sim p_{\gamma}(\cdot; \pi), a \sim \pi(\cdot \vert s)} [r(s, a)].
  \end{aligned}
$$

最直接地，从最大熵理论中得到启发，熵正则化强化学习问题变成了

$$
  \begin{aligned}
  \max_{\pi} \rho(\pi) =& \mathbb{E}_{s \sim p_{\gamma}(\cdot; \pi), a \sim \pi(\cdot \vert s)} [r(s, a)] - \alpha \mathbb{E}_{s \sim p_{\gamma}(\cdot; \pi)}[D_{KL}(\pi(\cdot \vert s) \Vert \tilde \pi(\cdot \vert s))] \\
  =& \mathbb{E}_{s \sim p_{\gamma}(\cdot; \pi), a \sim \pi(\cdot \vert s)} [r(s, a) - \alpha D_{KL}(\pi(\cdot \vert s) \Vert \tilde \pi(\cdot \vert s))] \\
  =& (1 - \gamma) \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi)} \bigg[\sum^{\infty}_{t=0} \gamma^t [r_t - \alpha D_{KL}(\pi(\cdot \vert s) \Vert \tilde \pi(\cdot \vert s))] \bigg] 
  \end{aligned}
$$

---

首先我们来研究一种定义方式

$$
  Q_\pi(s, a) = \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi)} \bigg[\sum^{\infty}_{t=0} \gamma^t [r_t - \alpha D_{KL}(\pi(\cdot \vert s_t) \Vert \tilde \pi(\cdot \vert s_t)) \vert s_0 = s, a_0 = a] \bigg].
$$

$$
  V_\pi(s) = \mathbb{E}_{a \sim \pi(\cdot \vert s)} [Q_\pi(s, a)].
$$

$$
  \rho(\pi) = \mathbb{E}_{s \sim p_0, a \sim \pi(\cdot \vert s)} [Q_\pi(s, a)].
$$

可见这是一个牵一发而动全身的改变，并且最优贝尔曼操作也因此变形为(矩阵形式，要求状态动作都是有限集)

$$
  TQ = R - \alpha D_{KL}(\pi\Vert\tilde \pi) + \gamma \mathbf{P}_{\pi} Q.
$$

$$
  \begin{aligned}
  TQ(s, a) =& \max_{\pi} R(s, a) - \alpha D_{KL}(\pi(\cdot \vert s) \Vert \tilde \pi(\cdot \vert s))\\ &+ \gamma \sum_{s'} p(s' \vert s, a) \sum_{a'} \pi(a' \vert s') Q(s', a').
  \end{aligned}
$$


其中$\tilde \pi(a \vert s)$通常是均匀分布。

---

上面这种定义方式会导致最优贝尔曼操作很难求解，所以人们定义了另一套符号。这套符号会看起来稍微绕一些，但是在计算中能够提供很大的方便。

$$
  Q_\pi(s, a) = \mathbb{E}_{\tau \sim \mathcal{MDP}(\pi)} \bigg[r_0 + \sum^{\infty}_{t=1} \gamma^t [r_t - \alpha D_{KL}(\pi(\cdot \vert s_t) \Vert \tilde \pi(\cdot \vert s_t)) \vert s_0 = s, a_0 = a] \bigg].
$$

$$
  V_\pi(s) = \mathbb{E}_{a \sim \pi(\cdot \vert s)} [Q_\pi(s, a)] - \alpha D_{KL}(\pi(\cdot \vert s) \Vert \tilde \pi(\cdot \vert s)).
$$

$$
  \rho(\pi) = \mathbb{E}_{s \sim p_0, a \sim \pi(\cdot \vert s)}[Q_\pi(s, a) - \alpha D_{KL}(\pi(\cdot \vert s) \Vert \tilde \pi(\cdot \vert s))].
$$

类似的，最优贝尔曼操作变为

$$
  TQ = \max_{\pi} R + \gamma \mathbf{P}_{\pi} (Q - \alpha D_{KL}(\pi \Vert \tilde \pi)).
$$

或者等价为

$$
  \begin{aligned}
  TQ(s, a) =& \max_{\pi} R(s, a) + \gamma \sum_{s'} p(s' \vert s, a) \sum_{a'} \pi(a' \vert s') \\
  & [ Q(s', a') - \alpha(\ln\pi(a' \vert s') - \ln\tilde\pi(a' \vert s')) ]. \\
  \end{aligned}
$$

---

可以证明最优贝尔曼操作是一个收缩映射，存在不动点。我们定义

$$
  \pi_Q = \arg\max_{\pi} \sum_{a} \pi(a \vert s) 
  [ Q(s, a) - \alpha(\ln\pi(a \vert s) - \ln\tilde\pi(a \vert s)) ].
$$

求解可得

$$
  \pi_Q(s, a) = \frac{\tilde\pi(a\vert s) \exp{[Q(s, a)/\alpha]}}{\sum_{a'} \tilde\pi(a' \vert s) \exp{[Q(s, a')/\alpha]}}.
$$

再定义

$$
  V_Q(s) = \alpha \ln \bigg\{\sum_{a} \tilde \pi(a \vert s) \exp[Q(s, a; \theta_Q)/\alpha]\bigg\}.
$$

那么有关系

$$
  \pi_Q(a \vert s) = \tilde\pi(a \vert s) \exp\{[Q(s, a) - V_Q(s)]/\alpha\};
$$

$$
  Q(s, a) = V_Q(s) + \alpha \ln \frac{\pi_Q(a \vert s)}{\tilde\pi(a \vert s)};
$$

$$
  V_Q(s) = \sum_{a}\pi_Q(a \vert s) Q(s, a) - \alpha D^Q_{KL}(s),
$$

其中 $D^{Q}_{KL}(s) = D_{KL}(\pi_Q(\cdot \vert s) \Vert \tilde\pi(\cdot \vert s))$;

$$
  TQ = R + \gamma \mathbf{P}_{\pi} V_Q.
$$

---

## Soft Q-Learning

我们使用$\theta_Q$来参数化$Q$函数，写为$Q(s, a; \theta_Q)$。
那么 Soft Q-Learning 使用的损失函数为

$$
  L(\theta_Q) = \mathbb{E}_{s\sim p, a \sim \pi(\cdot \vert s; \theta_Q)}\bigg[\frac{1}{2} \Vert Q(s, a; \theta_Q) - TQ(s, a;\theta) \Vert^2\bigg\vert_{\theta=\theta_Q}\bigg],
$$

那么

$$
\begin{aligned}
& \nabla_{\theta_Q} L(\theta_Q) \\
= &\mathbb{E}_{s\sim p, a \sim \pi(\cdot \vert s; \theta_Q)}\bigg\{[Q(s, a; \theta_Q) - TQ(s, a;\theta_Q)]\nabla_{\theta_Q} Q(s, a; \theta_Q) \bigg\}\\
=&\mathbb{E}_{s\sim p, a \sim \pi(\cdot \vert s; \theta_Q)}\bigg\{\bigg[V(s;\theta_Q) + \alpha \ln \frac{\pi(a \vert s; \theta_Q)}{\tilde \pi(a \vert s)} - r(s, a) \\
& - \gamma \sum_{s'} p(s'\vert s, a) V(s';\theta_Q) \bigg]
\nabla_{\theta_Q} \bigg[V(s;\theta_Q) + \alpha \ln\frac{\pi(a \vert s; \theta_Q)}{\tilde \pi(a \vert s)}\bigg] \bigg\}\\
=& \mathbb{E}_{s\sim p, a \sim \pi(\cdot \vert s; \theta_Q)} \bigg\{ - \alpha \hat Q(s, a; \theta_Q) \nabla_{\theta_Q}\ln \pi(a \vert s; \theta_Q) \\
& + \nabla_{\theta_Q} \frac{1}{2}\Vert V(s; \theta_Q) - \hat V(s; \theta) \Vert^2\vert_{\theta = \theta_Q}\bigg\}.
\end{aligned}
$$

（用到了 $\sum_a \nabla_{\theta} \pi(a \vert s; \theta) = 0$。）

其中

$$
\hat Q(s, a;\theta_Q) = r(s, a) + \gamma \sum_{s'}p(s' \vert s, a) V(s'; \theta_Q) - \alpha \ln\frac{\pi(a \vert s; \theta_Q)}{\tilde \pi(a \vert s)}.
$$

$$
\hat V(s; \theta) = \sum_{a} \pi(a \vert s; \theta) \bigg[r(s, a) + \gamma \sum_{s'}p(s' \vert s, a) V(s';\theta)\bigg] - \alpha D_{KL}(s; \theta).
$$

其中 $-\alpha \hat Q(s, a; \theta_Q) \nabla_{\theta_Q} \ln \pi(s, a; \theta_Q)$ 和策略梯度的形式非常类似，但实际上存在着某种偏差，再原论文中就没有再深入讨论了。我暂时也没有更深入地研究。

!!!Note
    $\hat Q(s, a; \theta_Q)$ 和第一次定义的$Q(s, a)$是统一的，不是后面定义的。也就是后面定义的$Q$函数少了起始项的交叉熵，这里$\hat Q(s, a)$补充回来了。

```bib
@article{schulman2017equivalence,
  title={Equivalence between policy gradients and soft q-learning},
  author={Schulman, John and Chen, Xi and Abbeel, Pieter},
  journal={arXiv preprint arXiv:1704.06440},
  year={2017}
}
```