[![header](../../../assets/header21.jpg)](https://yuenshome.github.io)

<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>

# 深度学习正则化系列3：约束优化的范数惩罚

《Deep Learning》Chapter 7
Regularization for Deep Learning
翻译水平有限，如有错误请留言，不胜感激！
<h1 class="entry-title">7.2 约束优化的范数惩罚</h1>
考虑参数范数惩罚的代价函数正则化：

$$
\widetilde{J}(\theta; X, y) = J(\theta; X, y) + \alpha \Omega(\theta).
$$

回顾第 4.4 小节中，由于原来的目标函数有一套惩罚，通过构造一个广义的拉格朗日函数，可以使约束函数最小化。每个惩罚相当于一个乘积，乘积由两项构成：一项是被称为Karush–Kuhn–Tucker（KKT）乘子的系数，另一项是用来代表限制条件是否满足的函数。如果我们想要约束 $\Omega(\theta)$ 小于某个常数 $k$ ，我们可以构造一个广义拉格朗日函数：

$$
\mathcal{L} (\theta, \alpha; X, y) = J(\theta; X, y) + \alpha (\Omega(\theta) - k).
$$<!--more-->

约束问题的解由以下公式给出：

$$
\theta^* = \arg_{\theta} \min \max_{\alpha,\alpha \leq 0} \mathcal{L}(\theta, \alpha).
$$

正如第 4.4 小节所述，要计算得到 $\theta^*$ 值需要同时修改 $\theta$ 与 $\alpha$ 的值，第 4.5 小节提供了一个带 $L^2$ 正则化限制的线性回归的例子。许多不同的过程是可能的——某些情况可以使用梯度下降求解，而其它时候因为梯度为零而需要使用分析解，但是在所有过程中，当 $\Omega (\theta) &gt; k$ 时， $\alpha$ 必须增加，而当 $\Omega (\theta) &lt; k$ 时 $\alpha$ 必须减小。所有的正值 $\alpha$ 会促使 $\Omega (\theta)$ 收缩，还有最佳值 $\alpha^*$ 也会促使 $\Omega (\theta)$ 收缩，但再怎么收缩减少也不会使 $\Omega (\theta)$ 小于 $k$ 。

为了进一步分析约束的作用，我们可以将 $\alpha^*$ 的值固定并把问题变为 $\theta$ 的函数的问题：

$$
\theta^* = \arg_{\theta} \min \mathcal{L}(\theta, \alpha^*) = \arg_\theta \min J(\theta; X, y) + \alpha^* \Omega(\theta).
$$

这与使 $widetilde{J}$ 最小化的正则训练问题完全相同。因此，我们可以认为参数范数惩罚是对权重施加约束，如果 $\Omega$ 是$L^2$ 范数，则权重被约束在 $L^2$ 球中。如果 $\Omega$ 是 $L^1$ 范数，则权重被限制在位于有限 $L^1$ 范数的区域中。通常我们不知道（通过使用系数$\alpha^*$ 的权重衰减的）约束区域的大小，因为得到了 $\alpha^*$ 的值并不能直接得到 $k$ 的值。原则上，可以求解这种交叉关系（fork），但是 $k$ 和 $\alpha^*$ 之间的关系取决于 $J$ 的形式。虽然不知道约束区域的确切大小，但可以通过增加或减少 $\alpha$ 来粗略地控制它，以便增长或缩小约束区域，较大的 $\alpha$ 将导致较小的约束区域，而较小的 $\alpha$ 将导致较大的约束区域。

有时我们可能希望使用明确的约束，而不是惩罚。如第 4.4 节所述，我们可以修改算法，如随机梯度下降沿 $J(\theta)$ 下坡，然后投射 $\theta$ 回到满足 $\Omega (\theta) &lt; k$ 的最近点。如果我们知道 $k$ 的什么值是适当的并且不想花时间搜索与该 $k$ 对应的 $\alpha$ 值，这可能是有用的。

使用显式约束和重投影而不是用惩罚实施约束的另一个原因是惩罚可以导致非凸优化过程陷入对应于小θ的局部最小值。当训练神经网络时，这通常表现为训练有几个“死掉的神经元单位”的神经网络。这些“死掉的神经元”是对网络学习的函数的行为没有多大贡献的单位，因为进入或离开这些神经元的权重都很小。当训练对权重的范数进行惩罚时，这些配置可以是局部最优的，即使可以通过使权重更大来显着减少 $J$ 。通过重投影实现的显式约束在这些情况下的效果更好，因为这种方法不会促使权重逼近原点。通过重投影实现的显式约束只有当权重变大并且试图离开约束区域时才具有效果。

最后，使用重投影的显式约束可能是有用的，因为它们对优化过程施加了一些稳定性。当使用大学习率时，可能遇到正反馈回路，其中大的权重诱导大的梯度，然后引起对权重的大的更新。如果这些更新一致地增加权重的大小，则 $\theta$ 迅速地从原点移开，直到出现数值溢流。具有重投影的显式约束可以防止此反馈循环，因为权重的持续增大没有限制。Hinton等人 （2012c）建议使用约束结合高学习率，这样可允许快速探索参数空间，同时保持一些稳定性。

特别地， Hinton 等人 （2012c）推荐由 Srebroand Shraibman （2005）引入的策略：约束神经网络层的权重矩阵的每列的范数，而不是约束整个权重矩阵的 Frobenius 范数。分别约束每列的范数可防止任何一个隐藏单元具有非常大的权重。如果我们在拉格朗日函数中将这个约束转换为惩罚，它将类似于 $L^2$ 权重衰减，但是对于每个隐藏单元的权重具有单独的 KKT 乘子。这些 KKT 乘子中的每一个将被单独地动态地更新，以使每个隐藏单元服从约束。在实践中，列规范限制总是作为具有重投影的显式约束来实现。
