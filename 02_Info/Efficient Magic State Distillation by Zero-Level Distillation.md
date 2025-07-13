---
title: Efficient Magic State Distillation by Zero-Level Distillation
tags: 
author: Tomohiro Itogawa, Yugo Takada, Yutaka Hirano, Keisuke Fujii
year: "2025"
journal: PRX Quantum
---

阅读状态：精读
PDF Link：[Efficient Magic State Distillation by Zero-Level Distillation](https://journals.aps.org/prxquantum/pdf/10.1103/thxx-njr6)

## 一、省流总结

### 阅读目的

- [ ] 写文献综述、查资料
- [x] 了解新领域
- [ ] 训练提升学术能力
- [ ] 借鉴文章具体方法
- [ ] 回答下列问题：

### 作者的写作目的

- [ ] 总结其他研究
- [ ] 建议未来方向
- [ ] 报告新发现
- [x] 发展理论/算法
- [ ] 表达特别的观点

### 主要贡献

> 写完笔记之后最后填，概述文章的内容，以后查阅笔记的时候先看这一段

#### 解决了什么问题？

#### 主要创新点在哪里？

## 二、文章结构

### Abstract

Magic state distillation (MSD) is an essential element for universal fault-tolerant quantum computing, which distills a high-fidelity magic state from noisy magic states using ideal (error-corrected) Clifford operations. For ideal Clifford operations, **it needs to be performed on the logical qubits and hence incurs a large spatiotemporal overhead**, which is one of the major bottlenecks for the realization of fault-tolerant quantum computers (FTQCs). Here we propose zero-level distillation, which **prepares a high-fidelity logical magic state at the physical level**, namely zero level, using physical qubits and nearest-neighbor two-qubit gates on a square lattice. We develop a zero-level distillation circuit and show that distillation can be made even more efficient than the conventional sophisticated approaches with logical level distillations. The key idea involves the Knill et al.-type distillation using the Steane code and its careful mapping to the square-lattice architecture with error detection. The distilled magic state on the Steane-code state is then teleported or converted to surface codes. We numerically find that the error rate of the logical magic state scales as approximately 100 × p2 in terms of the physical error rate p. For example, with a physical error rate of p = 10−4 (10−3 ), the logical error rate is reduced to p L = 10−6 (10−4 ), resulting in an improvement of 2 (1) orders of magnitude. This contributes to reducing both space and time overhead for early FTQC as well as full-fledged FTQC combined with conventional multilevel distillation protocols.

### Conclusion

We proposed zero-level distillation, which efficiently distills and prepares the logical magic state encoded in surface codes without requiring multiple logical qubits. All operations required can be implemented on the square lattice connectivity and the number of required physical qubits is substantially reduced and spatial overhead for one or two logical patches is sufficient.

According to the numerical simulation, zero-level distillation with teleportation successfully reduces the logical error rate pL of a logical magic state to pL = 100 × p2 . For example, when p = 10−3 and pL = 10−4 , the logical error rates result in p = 10−4 and pL = 10−6 , respectively, indicating 1 and 2 orders of magnitude improvement. The success rate is reasonably high even when p = 10−3 . The depth of the zero-level distillation circuit is only 25, hence it is compatible with the conventional multilevel distillation routines. In addition, we also developed zerolevel distillation based on the code conversion to further reduce the number of physical qubits employed.

### Motivation

> [!NOTE] 作者的研究目标？
> 探索低开销、有良好错误缩放的魔态蒸馏协议。

> [!NOTE] 作者需要解决的问题？
> 魔态蒸馏是实现 FTQC 的一大瓶颈。现存许多纠错码可以都过 Transversal gate 或 Lattice surgery 等方式实现 Clifford 门，但保障量子计算优势的非 Clifford 门需要通过魔态蒸馏并通过 [[Gate Teleportation]] 加入到线路中。
>
> 现存的魔态蒸馏协议要么物理比特开销很大，要么破坏了拓扑上局部连接的约束，需要提出考虑到此约束的低开销、高保真度的蒸馏协议。

> [!NOTE] 关于这个问题，其它学者提出了哪几类的解决方案，有何缺陷?
> - 传统逻辑比特级联方案：错误缩放较高、可以实现横向 T 门，但空间开销很大；
> - 目前物理比特方案：空间开销小，但涉及复杂连接，不适合 Surface code 和超导路线；

### Background

预备部分：使用 Steane code 蒸馏 $\ket{A}$ 态的量子线路：[[MSD Circuit For Steane Code]]

### Method(s)

### Evaluation

> 作者如何评估自己的方法？
> 实验的 setup 是什么样的？
> 感兴趣实验数据和结果有哪些？
> 有没有问题或者可以借鉴的地方？**量子比特排列？能否加入到线路优化中**
> 从结果（含曲线) 上看，作者是如何有力地证明他解决了问题?

### Structure

> 本文的结构怎样？对你写文章有什么参考作用？

## 三、阅读笔记

### 阅读挑战

1. 这篇论文的缺陷在哪儿?
2. 尝试一下，在你所阅读的论文与现实生活之间建立物理与逻辑联系，找出一个可以拓展的点?
3. 对于作者提出的解决方法，你有何看法和建议？

### 对我有什么用

1. 回答了我为什么要读的问题了吗？
2. 同意/反对
3. 准备正面/负面的引用吗？
4. 准备深度的讨论吗？

### 改进思路

## Reference

> (optional) 列出相关性高的文献，以便之后可以继续 track 下去。
