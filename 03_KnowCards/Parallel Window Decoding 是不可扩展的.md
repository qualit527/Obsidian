---
date: 2025-08-08
title: Parallel Window Decoding 是不可扩展的
tags:
  - insight
---
[[Window Decoding|Parallel Window Decoding]] 的提出带有很强烈的动机：串行滑动窗口是不可扩展的——不管 decoder 优化得有多快，都一定有一个 code size 使得 decoder 跑得比 syndrome 慢（$r_{decode} < r_{syndrome}$）。

而并行窗口的提出使得只要能保证
$$
\frac{M*r_{decode}}{2} > r_{syndrome}
$$
即可，$M$ 是阻塞操作前需要处理的窗口数量，共运行两轮，每轮并行使用 $M/2$ 个 decoder。后续 [[../02_Info/Predictive Window Decoding for Fault-Tolerant Quantum Programs|SWIPER]] 又把这个约束优化成了
$$
M*r_{decode} - \frac{\epsilon}{t} > r_{syndrome}
$$
即只需用并行的 $M$ 个 decoder 运行一轮。

但是，他们都没有讨论**需要的 decoder 数量 $M$ 到底是多少**。

SWIPER 里面有一张带有迷惑性的示意图：

![[../attachments/Pasted image 20250808211207.png|600]]

看上去，为了执行阻塞操作（T teleportation 后的 S 门修正），只需要将相邻的这 3 个窗口解码完毕，就可以更新完执行 T 门所需的 [[Pauli Frame]]。而事实是需要：
1. 执行 T 门的 logical qubit 的所有历史窗口
2. 与该 qubit 在阻塞操作前有过多比特测量的所有 qubit（关联 qubit），在执行该测量前的所有历史窗口
3. 第二点中的每个关联 qubit 自身，在执行与 T qubit 关联测量前的其他关联 qubit，的所有历史窗口

可以看到这是一个递归的定义，画成示意图为：

![[../attachments/367d559b0f6cf56ef0becacbef1e0bdf.jpg|300]]

纵向是时间轴，横向是空间布局，阴影连接了多比特测量。图中涉及的所有 patch 都需要在顶端的 Clifford 修正执行前解码完毕。（不然，试想右下角的 patch 发生了一个未被解码的 logical pauli，其经过两个多比特门传到了当前执行 T 门的 patch，使得我们对当前 patch 的 pauli frame 推断有误）

已知 decoder 的速度比 syndrome 慢，那么示意图中时间上的深度一定是存在的。为了并行窗口解码，涉及到的 decoder 数量是多少呢？在最坏情况下：
- 空间：所有 logical qubit 都与当前 qubit 递归相关
- 时间：算法的连续大部分时间都没有执行 T 门，而随着 code size 的 scaling，单个 decoder 的速度太慢，几乎完全落后

因此，最坏情况下所需的 decoder 数量 $M_{worst} = N_{logical}T_{slices}$，即量子算法编译后的时空体积。

至此我们说明了，随着 code size 的增大，并行窗口虽然解决了 decoder 速度上不可扩展的问题，但随之带来的是经典硬件资源的不可扩展。这带给我们动态调度 decoder 而不是被动等 T 门到来再执行并行窗口的动机。
