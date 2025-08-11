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

传统的 sliding window decoding 的思想是将 syndrome data 分割成固定长度的时间窗口，并按顺序对每个窗口解码。这样使得 decoder 处理的数据流可控，然而，这种方法是不可 scaling 的：

> [!Warning] Sliding window decoding is unscalable!
> 设每个 sliding window 包含 $n_{syn}$ 轮的 syndrome 数据，产生每轮数据的时间为 $\tau_{round}$，则生成 window 的时间为 $\tau_{gen}=n_{syn}\tau_{round}$。为了避免指数延迟，decoding 的速度需要满足 $\tau_{dec}<n_{syn}\tau_{round}$。
> 在乐观情况下——decoder 处理数据是线性复杂度，即 $\tau_{dec} = kn_{syn}N$，其中 $k$ 为常数、$N$ 为码长。代入上式得：
> $$
> N < \frac{\tau_{round}}{k}
> $$
> 该式意味着，为使得 decoder 能跟上 syndrome 的速度，使用的纠错码的大小有一个上界。

这个问题直到 2022 年 parallel window decoding 的提出才被理论上解决，又在 2025 年被进行了优化（speculative window decoding）。

- Parallel window：时空上不相邻的窗口是不存在依赖关系的，因此可以并行解码。在一维时间轴上，只需要串行的两层解码就可以处理所有的窗口。
- [[Predictive Window Decoding for Fault-Tolerant Quantum Programs|Speculative window]]：可以想办法移除窗口间的依赖，先在窗口边界处局部解码，之后就可以近似并行地解码所有窗口，如果正式解码后窗口边界的 prediction 和局部解码不符，则重启相邻窗口的解码。

**Insight**：只要给定足够数量的 decoder，就可以快速并行解决大量的解码任务，即使单个 decoder 的解码速度较慢。

> [!Warning] Speculation 的潜在问题
> 1. 只能预测到 $\text{weight}\leq_{2}$ 的边缘错误
> 2. 没有考虑到足够的信息，可能会增加发生 logical error 的概率
> 3. 错误的预测可能会影响整条逻辑链，不过从实验结果看基本只影响相邻的

> [!Question] 能否将改进的窗口策略应用到 BP（即 QLDPC decoding）？可能会多出什么问题？
