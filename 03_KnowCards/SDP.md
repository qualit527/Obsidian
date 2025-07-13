---
date: 2025-06-26
title: "{{title}}"
tags:
  - definition
---

# Definition

SDP 问题在量子信息中常写成以下的形式：

$$
\begin{array}{l}
\begin{aligned}
&\underline{\textbf{Primal}}\\
\text{maximize} \quad & \mathrm{Tr}[AX] \\
\text{s.t.} \quad & \Phi(X) \leq B, \\
& X\geq 0.
\end{aligned}
\\[1em]
\begin{aligned}
&\underline{\textbf{Dual}}\\
\text{minimize} \quad & \mathrm{Tr}[BY] \\
\text{s.t.} \quad &\Phi ^{\dagger}(Y) \geq A, \\
&Y\geq 0.
\end{aligned}
\end{array}
$$

> [!NOTE] 为什么这是一个凸优化问题？
> - 目标函数 trace 是线性函数，在 primal 满足凹性，在 dual 满足凸性
> - $Y>0$ 不但是凸的，还是一个凸锥：对于 $\lambda_{1},\lambda_{2}>0,\ \lambda_{1}X_{1}+\lambda_{2}X_{2}\geq 0$
> - $\Phi(X) \leq B$ 其中 $\Phi(X)$ 是线性的，加到一起相当于仿射函数的半正定条件，也是凸的（但不是凸锥）

凸优化问题的好处是只有一个局部最小值，方便用数值方法直接求解。作为对比，现代的**绝大部分神经网络建模的问题都是非凸的**，非常不利于理论分析。

> [!NOTE] 推论 1：弱对偶性
> primal 的最优解一定不超过 dual 的最优解

**证明**：由下面的不等式得证：
$$
\mathrm{Tr}[AX]\leq \mathrm{Tr}[\Phi^{\dagger}(Y)X]=\mathrm{Tr}[Y\Phi(X)]\leq \mathrm{Tr}[YB]
$$

其中第一个和最后一个不等式用到了 [[Trace 的一些性质#Properties]] 的推论 2；第二个不等式用到伴随超算符的定义（可以用算符的伴随理解：$\langle y, Ax \rangle = \bra{y}A\ket{x} = \langle A^{\dagger}y, x \rangle$，超算符也保留了这个作用）。

我们可以为 SDP 有一个几何图像，primal 问题是一个碗口向下的抛物线，dual 问题是一个碗口向上的，前者的最高点不会高于后者的最低点。如果这两个点重合，那么就满足了**强对偶性**，这两个问题的曲线构成了一个马鞍面，最优解就是鞍点。

**（未完待续）**
