---
date: 2025-07-11
title: Window Decoding
tags:
  - knowledge
---
在容错量子计算中，decoder 需要处理每个编码块重复测量产生的 syndrome。比如，有 $l$ 个 surface code 逻辑比特参与计算，每次纠错进行 $3d$ 次测量，则校验节点的数量为 $3ld^3$。

由于 [[Pauli frame]] 只能与 Clifford 门进行交换，因此 decoder 需要在执行非 Clifford 门前将其更新完毕。在 surface code 上，常用 Clifford+T 实现通用线路，其中的 T 门通过 [[Gate Teleportation]] 实现，decoder 需要在应用修正 S 门前更新好 frame。因此，传统解码方案对 decoder 时延的要求很高。

可以借鉴经典通信中的滑动窗口方案，将需要处理的校验节点划分为小窗口，并在相邻的两个窗口间设立 buffer 以保证 syndrome 的一致性。
![[Pasted image 20250711154926.png]]

> [!NOTE] 如何通过 buffer 确保 syndrome 的一致性
> 图 d 的深蓝色区域是当前窗口需要确定解决的 syndrome，称为 commit region。浅蓝色是 buffer region，需要通过该区域给下一个窗口（绿色）的第一行确定额外信息，buffer 的长度为 d 是为了给 decoder 提供足够的信息。
>
> 我们需要考虑穿过 buffer 的错误，只有它们会对下一个窗口的译码结果产生影响：
> - 错误的终点在下一个窗口的第一行：这个错误可以在当前窗口直接提交，不需要下一个窗口关心，因此消除下一个窗口的 syndrome；
> - 错误的终点在下一个窗口内部：当前窗口只能提交错误的一部分，需要在下一个窗口上人工新建 syndrome 以保证其他部分的匹配。

目前存在三种窗口策略（图 e）：
- Sliding window：所有窗口串行分布，只有完成了第 $n-1$ 个窗口才能执行第 $n$ 个
- Parallel window：窗口可以隔一分层，如 1 和 3 之间没有逻辑关系，可以并行执行
	还有一个 aligned parallel window，强制使 T 门成为底层窗口以避免额外的等待
- [[Predictive Window Decoding for Fault-Tolerant Quantum Programs|Speculative window]]：先只用窗口邻接处的 1~2 行作预处理，预测相邻窗口的依赖，因此可以几乎同时进行

> [!Warning] Speculation 的潜在问题
> 1. 只能预测到 $\text{weight}\leq_{2}$ 的边缘错误
> 2. 没有考虑到足够的信息，可能会增加发生 logical error 的概率
> 3. 错误的预测可能会影响整条逻辑链，不过从实验结果看基本只影响相邻的

> [!Question] 能否将改进的窗口策略应用到 BP（即 QLDPC decoding）？可能会多出什么问题？
