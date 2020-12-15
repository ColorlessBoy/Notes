# OpenAI Baseline: ACKTR & A2C

这是一篇OpenAI的[博客](https://openai.com/blog/baselines-acktr-a2c/)，介绍了一个添加了置信域的Actor-Critic框架的算法。它同时附带了一个Advantage Actor Critic(A2C)算法，对标DeepMind的Asynchronous Advantage Actor Critic算法。它自称A2C算法和A3C算法的表现差不多。（我不禁吐槽，异步更新算法显然比同步更新算法的适用范围更广才对。）接着阅读本文的由头，我们一次性搞清楚三个算法：A3C、A2C和ACKTR算法。

!!!Note
    不过我本人对使用随机策略梯度法构造的Actor-Critic框架的算法保持鄙视态度，因为对数函数使得这个两玩家的算法的纳什均衡点表现得很奇怪。我比较看好基于最优贝尔曼等式构造的Actor-Critic框架的算法，因为它的纳什均衡点就是最优策略。

## A3C

在这篇论文里，作者自称提出了一个不使用经验池（Replay Buffer），而是通过分布式计算的方式，来保证每次使用的样本的多样性（我是比较吐槽这种说法的）。算法本身确实没有什么可以称道的地方，就是并行优化版本的策略梯度优化算法，并且是异步更新。

## ACKTR

这篇论文里，作者使用自然梯度法（类似TRPO）来更新策略函数，但是使用的是A3C和A2C的算法框架。与TRPO不同的是，它使用了A Kronecker-factored approximate Fisher matrix for convolutional layers中的技术来估计Fisher Matrix而不是共轭梯度法。所以这也是一篇应用文而已。

```bib
@inproceedings{mnih2016asynchronous,
  title={Asynchronous methods for deep reinforcement learning},
  author={Mnih, Volodymyr and Badia, Adria Puigdomenech and Mirza, Mehdi and Graves, Alex and Lillicrap, Timothy and Harley, Tim and Silver, David and Kavukcuoglu, Koray},
  booktitle={International conference on machine learning},
  pages={1928--1937},
  year={2016}
}
@inproceedings{wu2017scalable,
  title={Scalable trust-region method for deep reinforcement learning using kronecker-factored approximation},
  author={Wu, Yuhuai and Mansimov, Elman and Grosse, Roger B and Liao, Shun and Ba, Jimmy},
  booktitle={Advances in neural information processing systems},
  pages={5279--5288},
  year={2017}
}
@inproceedings{grosse2016kronecker,
  title={A kronecker-factored approximate fisher matrix for convolution layers},
  author={Grosse, Roger and Martens, James},
  booktitle={International Conference on Machine Learning},
  pages={573--582},
  year={2016}
}
```
