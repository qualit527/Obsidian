---
date: 2025-07-01
title: "{{title}}"
tags:
  - trick
---

> [!NOTE] Trick：相空间点算符的对角元处处非负

我们先求相空间点算符 $A_{0}$ 的对角元：
$$
A_0 = \frac{1}{d} \sum_{a,b \in \mathbb{Z}_d} \tau^{-ab} Z^a X^b, \quad 
\tau = e^{(d+1)\pi i/d}, \quad w = e^{2\pi i/d}.
$$
在计算基 $\{ \ket{j} \}_{j=0}^{d-1}$ 中有：
$$
X^b \ket{j} = \ket{j \oplus b}, \quad 
Z^a \ket{j} = w^{aj} \ket{j},
$$
其中 $\oplus$ 表示模 $d$ 加法。于是对角元为：
$$
\begin{align*}
\langle j|A_0|j\rangle
&= \frac{1}{d} \sum_{a,b} \tau^{-ab} \langle j| Z^a X^b |j\rangle \\
&= \frac{1}{d} \sum_{a,b} \tau^{-ab} w^{aj} \underbrace{\langle j | j \oplus b \rangle}_{= \delta_{b,0}} \\
&= \frac{1}{d} \sum_{a} w^{aj}.
\end{align*}
$$
有限几何级数给出：
$$
\frac{1}{d} \sum_{a=0}^{d-1} w^{aj} =
\begin{cases}
1, & j = 0, \\
0, & j \ne 0.
\end{cases}
$$
因此我们有：
$$
\boxed{\langle j|A_0|j\rangle \in \{0,1\}}
$$
显然非负。

定义平移算符和相空间点算符：
$$
T_{\mathbf u} = \tau^{-a_1 a_2} Z^{a_1} X^{a_2}, \quad 
A_{\mathbf u} = T_{\mathbf u} A_0 T_{\mathbf u}^\dagger.
$$

对角元为：
$$
\begin{align*}
\langle j | A_{\mathbf u} | j \rangle 
&= \langle j | Z^{a_1} X^{a_2} A_0 X^{-a_2} Z^{-a_1} | j \rangle \quad (\text{忽略全局相位 } \tau^{-a_1 a_2}) \\
&= w^{a_1 j} \langle j | X^{a_2} A_0 X^{-a_2} | j \rangle w^{-a_1 j} \\
&= \langle j \ominus a_2 | A_0 | j \ominus a_2 \rangle,
\end{align*}
$$
其中 $\ominus$ 表示模 $d$ 减法。因此：
$$
\langle j|A_{\mathbf u}|j\rangle =
\begin{cases}
1, & j \equiv a_2 \ (\mathrm{mod}\ d), \\
0, & \text{otherwise}.
\end{cases}
$$
得证。
