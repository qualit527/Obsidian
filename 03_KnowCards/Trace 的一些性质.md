---
date: 2025-06-26
title: "{{title}}"
tags:
  - definition
  - trick
---

# Definition

> [!NOTE] 定义 1：Trace
> $\mathrm{Tr}[A]=\sum_{i=0}^{d-1} \bra{i}X\ket{i}$，其中 $d$ 为 Hilbert 空间维度

很常用的基本性质：

- $\mathrm{Tr}[AB]=\mathrm{Tr}[BA]$
- $\mathrm{Tr}[aA+bB]=a\mathrm{Tr}[A]+b\mathrm{Tr}[B]$
- $\mathrm{Tr}[A\otimes B]=\mathrm{Tr}[A]\times \mathrm{Tr}[B]$

# Properties

## 半正定矩阵相关

> [!NOTE] 引理 1
> 正规算子的迹为特征值之和

**证明：** 所有 normal 算子都可以被酉对角化，即存在 unitary $U$ 使得：
$$
A=U\Lambda U^{\dagger}, \quad \Lambda =\text{diag}\{\lambda_{1},\dots ,\lambda_{{n}}\}
$$

而：

$$
\mathrm{Tr}[A]=\mathrm{Tr}[U\Lambda U^{\dagger}]=\mathrm{Tr}[\Lambda U^{\dagger}U]=\mathrm{Tr}[\Lambda]
$$

得证。

> [!NOTE] 推论 1
> 半正定矩阵的 trace 一定非负

**证明：** 半正定矩阵 $A$ 是 Hermitian 的，且满足 $\forall v, \ v^{\dagger}Av\geq {0}$，显然其所有特征值都非负（否则取负特征值对应的特征向量，结果就小于 0 了）。结合引理 1 得证。

> [!NOTE] 推论 2
> 若 $A\leq B$，则 $\mathrm{Tr}[A]\leq \mathrm{Tr}[B]$

**证明**：由 推论 1 显然而得。

## 贝尔测量的恒等式

> [!NOTE] 定理 1
> $$
> \langle\Phi|(A \otimes B)|\Phi\rangle=\frac{1}{d}\operatorname{Tr}\left(A B^T\right)
> $$

**证明**：
$$
\begin{aligned}
\langle\Phi|(A \otimes B)|\Phi\rangle &= \frac{1}{d} \sum_{i, j}\langle i i|(A \otimes B)|j j\rangle \\
&= \frac{1}{d} \sum_{i,j} \braket{i|A|j}\cdot \braket{i|B|j} \\
&= \frac{1}{d} \sum_{i,j} \braket{i|A|j}\cdot \braket{j|B^T|i} \\
&= \frac{1}{d} \sum_{i} \braket{i|A\sum_{j}\ket{j} \bra{j}B^T |i} \\
&= \frac{1}{d} \sum_{i} \braket{i|AB^T|i} \\
&= \frac{1}{d} \mathrm{Tr}[AB^T]
\end{aligned}
$$

其中第三个等号用到矩阵 $B$ 坐标的 identity。

该定理反过来可能也能用上。

## 偏迹

当复合系统可以被写成张量积时，偏迹的定义非常直观，就是 trace 掉对应子系统。但即使系统是混合态，偏迹仍是良定义的线性超算符：

> [!NOTE] 定义 2：Partial Trace
> 设 $\rho \in \mathcal{L}(\mathcal{H}_A \otimes \mathcal{H}_B)$，则对任意 $M \in \mathcal{L}(\mathcal{H}_A)$，偏迹 $\mathrm{Tr}_B[\rho]$ 定义为满足：
> $$
> \mathrm{Tr}[\rho \cdot (M \otimes I_B)] = \mathrm{Tr}[(\mathrm{Tr}_B[\rho]) \cdot M]
> $$
> 从测量的角度理解：将 $\rho$ 作用于复合系统的期望值，应等于将 $\mathrm{Tr}_{B}[]\rho]$ 作用于 $M$ 上的期望值。
>
> **计算公式：**
> $$
> \mathrm{Tr}_B[\rho] = \sum_i (\mathbb{I}_A \otimes \bra{i}) \ \rho \ (\mathbb{I}_A \otimes \ket{i})
> $$

例，最大纠缠态：

$$
\ket{\Phi^+} = \frac{1}{\sqrt{2}}(\ket{00} + \ket{11}), \quad \rho = \ket{\Phi^+} \bra{\Phi^+} 
$$
$$
\mathrm{Tr}_B[\rho] = \frac{1}{2} (\ketbra{0}{0} + \ketbra{1}{1}) = \frac{I}{2}
$$

可以理解为对 $\ket{\Phi^+}$ 的每一个分量都取偏迹，只剩两个分量非平凡。

> [!NOTE] 推论 3：无中生有
> 设 $X\in \mathcal{L}(\mathcal{H}_{a}\otimes \mathcal{H}_{B})$，则有：
> $$
> \mathrm{Tr}[X] = \mathrm{Tr}[\mathrm{Tr}_{A}[X]] = \mathrm{Tr}[\mathrm{Tr}_{B}[X]]
> $$
