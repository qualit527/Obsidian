---
date: 2025-07-11
title: Pauli Frame
tags:
  - knowledge
---

## Pauli Frame

> [!NOTE] 背景问题
> 在容错量子计算中，什么时候进行 decoding？什么是进行错误恢复？

在最原始的情况下，一个纠错周期为 [^1]：

![[Pasted image 20250711170936.png|600]]
即，进行逻辑操作、执行 r 轮稳定子测量、decoding、错误恢复。这个过程有两个大问题：
- 量子计算机需要硬等 decoding 完毕，计算时间太久，idle 过程也会产生不少错误
- 手动执行错误恢复也耗费时间，且可能引入额外错误

幸运的是，我们可以在大部分时间内只在经典设备中追踪累计发生的 Pauli frame，而不着急手动恢复。根据 Clifford 层级：
$$
\mathcal{C}_{k} = \{ U\mathcal{C}_{1}U^{\dagger}\subset \mathcal{C}_{k-1}\}.
$$
$\mathcal{C}_{0}$ 的 Pauli 门在和 $\mathcal{C}_{1}$ 的 Clifford 门交换后仍然是 Pauli 门。又因为主流纠错码的稳定子提取线路只有 Clifford 门组成，因此只要不碰到计算所需的逻辑非 Clifford 门，我们都可以同时进行计算与 decoding，同步更新 Pauli frame。每个物理比特的 frame 只有四种情况：$\{I, X, Z, XZ\}$，只需要 2 个经典比特就可以存储。

![[Pasted image 20250711171646.png|600]]
具体来说，Pauli frame 在量子计算中的变化如下表：

![[Pasted image 20250711171740.png|700]]

以测量为例，假如现在保存的 frame 有 $F_{q_{1}}=X$，且对该物理比特的 $Z$ 基测量结果为 $+1$，则只需要将测量结果解释为 $-1$。

理论上，只维护物理比特的 frame 就足够。比如要进行 lattice surgery 的 MZZ 测量，就需要一方面根据目前的物理 frame 解释测量结果，一方面要根据测量线路更新物理 frame。但可能有时再维护一个 logical frame 比较方便：

![[Pasted image 20250711173144.png|600]]

## Clifford Frame

有时我们的 frame 还是可以与非 Clifford 门交换一下 [^2]。比如，Pauli 门与 T 门交换会变成 Clifford 门，而理论上还是可以在经典软件中追踪 Clifford 门的变化。设当前的 frame 是 $E$：
$$
C \cdot E = (CEC^{\dagger})\cdot C
$$
括号内的乘积项完全由 Clifford 门组成，因此可以经典模拟。
![[Pasted image 20250711174550.png|800]]
图中 [^3] 第一次出现 logical Pauli 后，经过第一个 T 门被转换为 Clifford frame，经过第二个 T 门后会超出 Clifford 群。因此，这种情况下只需要在第二个 T 门之前更新完毕即可。

## References

[^1]: Riesebos L, Fu X, Varsamopoulos S, et al. Pauli frames for quantum computer architectures[C]//Proceedings of the 54th Annual Design Automation Conference 2017. 2017: 1-6.
[^2]: Chamberland C, Iyer P, Poulin D. Fault-tolerant quantum computing in the Pauli or Clifford frame with slow error diagnostics[J]. Quantum, 2018, 2: 43.
[^3]: https://quantumcomputing.stackexchange.com/questions/32048/rigorous-understanding-that-one-should-correct-pauli-drift-before-non-clifford
