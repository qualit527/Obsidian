---
date: 2025-06-29
title: "{{title}}"
tags:
  - knowledge
  - definition
aliases: []
---

## Choi 矩阵的数值形式

我们可以分析 [[Choi-Jamiolkowski Isomorphism#Definition|Choi 矩阵]] 的数值形式，先从外积表示转为张量积表示：

$$
\begin{aligned}
J &= (I \otimes \mathcal{E})(\ket{\Omega} \bra{\Omega}) \\
&= (I \otimes \mathcal{E})\left( \sum_{i=0}^{d-1}\ket{i}\ket{i} \sum_{j=0}^{d-1}\bra{j}\bra{j} \right) \\
&= (I \otimes \mathcal{E})\left( \sum_{ij}\ket{i} \bra{j} \otimes \ket{i} \bra{j}   \right) \\
&= \sum_{ij} E_{ij}\otimes \mathcal{E}(E_{ij})
\end{aligned}
$$

其中 $E_{ij}$ 表示坐标 $(i,j)$ 为 1，其余为 0 的矩阵。结合张量积的运算方式（左边矩阵的每一个元素乘以右边的矩阵），整个 Choi 矩阵实际上是信道 $\mathcal{E}$ （$\mathbb{C}^{n\times n}\to \mathbb{C}^{m\times m}$）作用于各个 $E_{ij}$ 上结果的分块矩阵，以 $d=2$ 为例：

![[265a1e14b0b62085da5ce8658e54c2a.jpg|600]]

## 信道的 Kraus 秩（Choi 秩）

我们可以从上面这个分块矩阵（$\mathbb{C}^{mn\times mn}$）中拆出信道 $\mathcal{E}$ 对于每个 $E_{ij}$ 的作用矩阵：

$$
\Phi(E_{kl}) = P_{k} \cdot J \cdot  P_{L}^* = P_{k} \cdot \sum_{i}\lambda_{i}\ket{v_i} \bra{v_{i}} \cdot P_{L}^* = \sum_{i} P_{k}\ket{v'_{i}} \bra{v'_{i}} P_{L}^*
$$

第一个等号中的 $P_{k}\in \mathbb{C}^{m\times mn}$ 的作用是提取矩阵的第 $k$ 行块，其转置的作用自然是提取第 $k$ 列块，形式如下（$m=n=2$ 时）：

$$
P_{0}=
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0
\end{bmatrix} \qquad
P_{1}=
\begin{bmatrix}
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

后面的等式是谱分解，且把非负特征值吸收进了特征向量中。

Choi 矩阵特征向量的维度是 $mn\times 1$，因此 $P_{k}\ket{v'_{i}}$ 的维度是 $m\times 1$。我们总可以找到作用在 $\mathbb{C}^n$ 单位向量 $\ket{e}$ 上的 $m\times n$ 矩阵 $V_{i}$ 满足：

$$
V_{i}\ket{e_{k}} = P_{k}\ket{v'_{i}}
$$

有点抽象？还看这个 $m=n=2$ 的例子：

![[ed22262a242d9479055606f210ce5aa.jpg|700]]

实际上 $V_{i}$ 就是特征向量 $\ket{v'_{i}}$ 按列优先堆叠成的矩阵！代入回信道单元作用的式子：

$$
\Phi(E_{kl}) = \sum_{i} P_{k}\ket{v'_{i}} \bra{v'_{i}} P_{L}^* = \sum_{i} V_{i}\ket{e_{k}} \bra{e_{l}} V_{i}^{\dagger} = \sum_{i} V_{i}E_{kl}V_{i}^{\dagger}
$$

再利用信道的线性性，把所有单元算符加到一起：

$$
\Phi(\rho) = \sum_{kl}\sum_{i}V_{i}E_{kl}V_{i}^{\dagger} = \sum_{i}V_{i}\rho V_{i}^{\dagger}
$$

我们就得到了信道的 Kraus 表示。

能写出 Choi 矩阵的一种谱分解，就能写出一套 Kraus 算符。根据 $V_{i}$ 与 $\ket{v'_{i}}$ 的对应关系，我们知道这种写法的 Kraus 算符数量就等于 Choi 矩阵的秩（Kraus 算符数量可以任意大，但 Choi 秩是其最小数量）。自然地，纯 unitary 信道的秩为 1。
