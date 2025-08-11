---
date: 2025-07-18
title: Syndrome Extraction Circuit for QLDPC Codes
tags:
  - insight
---

> [!question] 问题 1：给定一个 QLDPC code 的 stabilizers，如何写出对应的 syndrome extraction circuit？

首先，这是一个构造问题，而不是可不可解的问题。下图就是对稳定子的通用测量线路：

![[../attachments/Pasted image 20250718145456.png|400]]

- $X$ 型稳定子：$X$-basis 制备，数据 to 校验的 CNOT
- $Z$ 型稳定子：$Z$-basis 制备，校验 to 数据的 CNOT（等价于 $X$-basis 制备 + CZ 门）

所以理论上，只需要给每个稳定子分配一个 ancilla，稳定子连接 $c$ 个数据比特，就在上面连 $c$ 个 CNOT。

那么设计的瓶颈在哪里呢？

> [!question] 问题 2：给定一个 syndrome extraction circuit，如何使线路深度尽量低？

测量线路的深度显然会影响测量的错误率，因此稳定子测量的并行化是一个重点优化目标。

**例 1**： Surface code

![[../attachments/Pasted image 20250718150049.png|600]]

这个测量线路可以被完全并行化：

![[../attachments/Pasted image 20250718150247.png|700]]

**例 2**：Bivariate Bicycle code

![[../attachments/Pasted image 20250718150346.png|700]]

其中，BB code 的数据比特可拆成 6 个部分：

$$
\begin{array}{llll}
G_A: & H_A^X=\left[A_2+A_3 \mid B_3\right] & \text { and } & H_A^Z=\left[B_3^T \mid A_2^T+A_3^T\right] \\
G_B: & H_B^X=\left[A_1 \mid B_1+B_2\right] & \text { and } & H_B^Z=\left[B_1^T+B_2^T \mid A_1^T\right] .
\end{array}
$$

每部分内部是可以完全并行的。这个测量线路据说是 IBM 暴搜出来的。

这两个例子的 code 都具有某种程度的对称性，stabilizer 的连接并不复杂。但对于一般的 QLDPC code，可能需要图论或机器学习的方法尽量使测量并行化。

此外，还有一个比较大的问题。

> [!question] 给定一个 syndrome extraction circuit，如何使错误的传播尽量少？

换句话说是如何使测量线路尽量满足 distance preserving？这里我的了解还不深，但是一定有一定程度的 trade-off：

- 数据比特共享 ancilla：同一个数据比特的错误会传到多个 ancilla，反之亦然 —— ancilla 少，但错误率高
- 每个数据比特单独 ancilla（Shor-sytle）：错误率低，但 ancilla 多好几倍
- Flag qubit：暂时不了解

还有 hook error 的影响：

![[../attachments/Pasted image 20250718151458.png|500]]

在 surface code 中，可以通过调度并行测量的顺序避免 hook error 带来的 distance 减半的影响：

![[../attachments/Pasted image 20250718151519.png|500]]

对于一般的 QLDPC，有什么办法减缓 hook error 带来的错误传播？

# References

- Flag qubit: [Quantum Error Correction with Only Two Extra Qubits | Phys. Rev. Lett.](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.121.050502)
- CSS 的测量深度：[Constant-Overhead Quantum Error Correction with Thin Planar Connectivity | Phys. Rev. Lett.](https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.129.050504)
- 测量线路的错误表征：[Scalable Noise Characterization of Syndrome-Extraction Circuits with Averaged Circuit Eigenvalue Sampling | PRX Quantum](https://journals.aps.org/prxquantum/abstract/10.1103/PRXQuantum.6.010334)
