---
date: 2025-07-18
title: QCCD Architecture
tags:
  - knowledge
---

# Definition

传统的离子阱架构允许对其中的任意两个量子比特施加两比特门，因此是全联通的。但该架构随着离子数量增加，量子操作更难变得复杂，扩展性不好。为此，QCCD 架构将传统架构分割为单独的区域，每个区域内部联通，不同区域之间需要通过“shuttle”操作搬运位于边缘的离子，牺牲部分联通性而获得扩展性。shuttle 操作分为三个步骤：

![[../attachments/Pasted image 20250718135507.png]]

一些名词定义：

- SWAP：同一个 trap 内部两个离子的交换门，**代价低廉**
- Shuttle：将一个离子移动到另一个 trap 的过程，**代价高昂**
- Ion reordering：同一个 trap 内，通过一系列 SWAP 和滑动对离子重新排序，是宏观的表述
- Junction：不同 trap 之间连接的交叉区域，允许转弯移动

QCCD 架构有不同的拓扑架构实现：

![[../attachments/Pasted image 20250718140056.png]]

从左到右分别是 linear-connected（L 型）、grid-like（G 型）、fully-connected（S 型）。

# Model

可以将离子阱的拓扑结构抽象成图，并且任意的 SWAP 和 shuttle 操作都不会改变图的拓扑：

![[../attachments/Pasted image 20250718140550.png|500]]

其中，白色圆圈默认不存放数据比特，用于调度中转。trap 内部的边权重较低，穿过 junction 的边权重较高。蓝色实线的边从上到下穿越一个 shuttle，蓝色虚线的边横穿了两个 shuttle。

## Mapping

指算法最开始时数据比特的排列分布，可以分为两级：

1. 哪些比特放到同一个 trap 中：均匀放置、近邻放置、启发式放置
2. trap 内部怎么排列：根据比特在算法的作用决定，与 trap 内部耦合更多的放在中间，与外部耦合更多的放在边缘

## Scheduling

目标：最小化执行时间、最大化保真度

![[../attachments/Pasted image 20250718141724.png|600]]

可以定义一个启发式 cost function，对每个双比特门，考虑所有相关的 SWAP 操作，计算操作成本 + 对双比特门执行的收益，并用类似贪心的方法选择策略。

**改进空间**：目前的 cost function 比较短视，只考虑对当前双比特门的贡献，没有考虑未来潜在收益。RL 中的 Q-learning 正好解决的是这个问题，可惜不知道符不符合体系结构会议的审美。

# Reference

[S-SYNC: Shuttle and Swap Co-Optimization in Quantum Charge-Coupled Devices | Proceedings of the 52nd Annual International Symposium on Computer Architecture](https://dl.acm.org/doi/full/10.1145/3695053.3731084?casa_token=A5TPX4o-0ewAAAAA%3AcooDaE8lt5RXSdq94Gmz7vL8Q9Y0eet6adG-y3R8v2V2c-1X7F7Iig_AnbFsNUcEIfDRt-pFinwt1Q)
