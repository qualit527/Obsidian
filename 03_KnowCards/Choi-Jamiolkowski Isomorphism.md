---
date: 2025-06-26
title: "{{title}}"
tags:
  - definition
---

## Definition

> [!NOTE] 从信道到 Choi 矩阵
> $$
> J = (I \otimes \mathcal{E})(\ket{\Omega} \bra{\Omega}), \quad \text{where} \ \ket{\Omega} = \sum_{i}\ket{i}\ket{i}
> $$
> 也可以写成
> $$
> J = \sum_{i, j}\ket{i} \bra{j} \otimes  \mathcal{E}(\ket{i} \bra{j})
> $$

> [!NOTE] 从 Choi 矩阵到信道
> $$
> \mathcal{E}(\rho) = \mathrm{Tr}_{A}[J(\rho^T \otimes I)]
> $$

**证明**：
$$
\begin{aligned}
\mathrm{Tr}_{A}[J(\rho^T \otimes I)] &= \mathrm{Tr}_{A} [(I \otimes \mathcal{E})(\ket{\Omega} \bra{\Omega})(\rho^T \otimes  I)] \\[3pt]
&= \mathrm{Tr}_{A} \left[ \sum_{i, j}\ket{i} \bra{j}\rho^T \otimes  \mathcal{E}(\ket{i} \bra{j}) \right]  \qquad \text{（展开纠缠态）}\\[3pt]
&= \sum_{i, j, m} \braket{ m | i } \braket{ j | \rho^T | m } \mathcal{E}(\ket{i} \bra{j} ) \qquad \text{（偏迹定义）} \\[3pt]
&= \sum_{i, j} \braket{ j | \rho^T |i } \mathcal{E}(\ket{i} \bra{j} ) \\[3pt]
&= \mathcal{E}\left( \sum_{i, j} \rho_{ij}\ket{i} \bra{j} \right) \\[3pt]
&= \mathcal{E}(\rho)
\end{aligned}
$$

其中倒数第二个等号使用了 $\braket{ j | \rho^T |i } = \braket{ i | \rho|j }$，这两种表示都是取 $\rho$ 在计算基底下第 $i$ 行第 $j$ 列的元素。

> [!NOTE] 从 Kraus 算符到 Choi 矩阵
> 定义 vec 符号：
> $$
> |E_{k}\rangle\rangle = \text{vec}(E_{k}) = \sum_{i, j} \braket{i|E_{k}|j} \ \ket{i} \otimes \ket{j},
> $$
> 则 Choi 矩阵可写为：
> $$
> J = \sum_{k} |E_{k}\rangle\rangle \langle\langle E_{k}|
> $$

**证明**：由 vec 的如下性质得证：
$$
|E\rangle\rangle = (I \otimes E)\ket{\Omega} 
$$

> [!NOTE] 从 Choi 矩阵到 Kraus 算符
> 见 [[Choi 矩阵的形式 & 信道的秩#信道的 Kraus 秩（Choi 秩）|信道的 Kraus 秩（Choi 秩）]]

## Intuition

C-J 同构和 teleportation 有很强的关联，见 [Choi-Jamiolkowski 同构背后的直觉](https://physics.stackexchange.com/questions/270032/whats-the-intuition-behind-the-choi-jamiolkowski-isomorphism))。用文字描述，从信道到 Choi 矩阵相当于对 bell 态的第二个维度作用该信道；从 Choi 到信道相当于通过 [[Gate Teleportation]] 实现了该矩阵。

![[Pasted image 20250626174623.png|300]] ![[Pasted image 20250626174711.png|300]]

## Application

由于量子信道是一个 CPTP map，由 CP 性质得到其对应的 Choi 矩阵是半正定的，因此很适合用 [[SDP]] 求解相关的问题。

> [!NOTE] 量子信道（Choi 矩阵）的 SDP 表示
> $$
> \begin{aligned}
> \text{Variable} \quad & J \in \mathbb{C}^{d_{A}d_{B}\times d_{A}d_{B}} \\[3pt]
> \text{s.t.} \quad & J\geq 0 \qquad \text{(Completely Positive)} \\[3pt]
> & \mathrm{Tr}_{B}[J] = I_{A} \qquad \text{(Trace Preserving)}.
> \end{aligned}
> $$

其中 TP 性质的等价性证明用到了 [[Trace 的一些性质#偏迹]] 的“无中生有”trick。
