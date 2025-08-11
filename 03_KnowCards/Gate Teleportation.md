---
date: 2025-06-26
title: "{{title}}"
tags:
  - definition
---

# Definition

![[Pasted image 20250626181344.png|1000]]

上图是经典的 qubit teleportation，实际上门隐形传态唯一的区别就是把输入的 bell 态 $\Phi$ 变成了：

$$
(I \otimes U)\ket{\Phi} 
$$

由此可以推导：

$$
\begin{aligned}
a\ket{0} + b\ket{1} \otimes (I \otimes U)(\ket{00}+\ket{11}) &= a\ket{00}\otimes U\ket{0} + a\ket{01} \otimes U\ket{1} + b\ket{10} \otimes U\ket{0} + b\ket{11}\otimes U\ket{1} \\[3pt]
\text{(After CNOT)} &\to a\ket{00}\otimes U\ket{0} + a\ket{01} \otimes U\ket{1} + b\ket{11} \otimes U\ket{0} + b\ket{10}\otimes U\ket{1} \\[3pt]
\text{(After H1)} &\to a(\ket{0}+\ket{1})(\ket{0}\otimes U\ket{0}+\ket{1}\otimes U\ket{1}) + b(\ket{0}-\ket{1})(\ket{0}\otimes U\ket{1}+\ket{1}\otimes U\ket{0}) \\[3pt]
&= \ket{00}(aU\ket{0}+bU\ket{1}) \to U\ket{\psi} \\[3pt]
&+ \ket{01}(aU\ket{1}+bU\ket{0}) \to UX\ket{\psi} = (UXU^{\dagger})U\ket{\psi} \\[3pt]
&+ \ket{10}(aU\ket{0}-bU\ket{1}) \to UZ\ket{\psi} = (UZU^{\dagger})U\ket{\psi} \\[3pt]
&+ \ket{11}(aU\ket{1}-bU\ket{0}) \to UXZ\ket{\psi} = (UXZU^{\dagger})U\ket{\psi} \\[3pt]
\end{aligned}
$$

其中，括号中是要执行的恢复操作。

# Clifford Hierarchy

值得注意的是，**隐形传态的层级正好对应了 Clifford 层级**（[相关文章](https://arxiv.org/abs/quant-ph/9908010)）：

$$
\mathcal{C}_{k} = \{ U\mathcal{C}_{1}U^{\dagger}\subset \mathcal{C}_{k-1}\}.
$$

其中，$\mathcal{C}_{1}$ 就是泡利群，$\mathcal{C}_{2}$ 是 Clifford 群，$\mathcal{C}_{3}$ 包含 T 门和 Toffoli 门。**要实现 $U\in \mathcal{C}_{k}$ 的隐形传态，只需要 $\mathcal{C}_{k-1}$ 中的门**。

例如，T 门（$Z_{\pi/8}$ 门）隐形传态需要 S 门和泡利门作修正：

![[Pasted image 20250626183758.png|600]]

S 门（$Z_{\pi/4}$ 门）隐形传态需要泡利门作修正：

![[Pasted image 20250626183828.png|800]]

**（电路等效性尚未验证）**

# Intuition

> [!NOTE] 门隐形传态和 [[Choi-Jamiolkowski Isomorphism#Definition|C-J Isomorphism]] 的等价性
> - 对 bell 态第二个维度作用 $U$ 正好就实现了 “纯信道” $\mathcal{E(\rho)}=U\rho U^{\dagger}$ 的 Choi 矩阵
> - 整个隐形传态的过程实现了 “通过 Choi 矩阵应用信道” 这个逆向过程

**证明**：隐形传态的过程相当于对输入态作 bell 基测量：
$$
\mathcal{E}(\rho)=\langle\Phi|(\rho \otimes J)|\Phi\rangle
$$

利用 [[Trace 的一些性质#贝尔测量的恒等式]] 有：

$$
\langle\Phi|(\rho \otimes J)|\Phi\rangle = \mathrm{Tr}_{A}[J(\rho^T\otimes I)] 
$$

这里 $J$ 是作用在 $\mathcal{H}_{A}\otimes \mathcal{H}_{B}$ 上的运算，$\rho$ 作用在 $\mathcal{H}_{A}$ 上。为了对齐维度，将 $\rho$ 拓展为 $\rho \otimes I$ 并 trace 掉 $\mathcal{H}_{A}$。

# One-bit Teleportation

待整理

[容错 - 一位传送和魔法状态的小工具电路之间的差异 - 量子计算 --- fault tolerance - Differences between one bit teleportation and gadget circuits for magic states - Quantum Computing Stack Exchange](https://quantumcomputing.stackexchange.com/questions/40900/differences-between-one-bit-teleportation-and-gadget-circuits-for-magic-states)

[arXiv:quant-ph/0002039v2 1 Aug 2000](https://arxiv.org/pdf/quant-ph/0002039)
