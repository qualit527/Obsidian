---
date: 2025-06-27
title: "{{title}}"
tags:
  - knowledge
---

# Definition

![[Pasted image 20250627200657.png|800]]

这是通过 Steane code 构造 Transversal H 门进行魔态蒸馏的线路图，一眼看上去很复杂，可以逐部分分析。

# Encoding

这一部分完成的是任意量子态（这里是魔态 $\ket{A}$）的注入。之所以看着面生是因为之前 surface code 的编码线路太有规律了，而且平时注入的都是 $\ket{0}$ 态。CNOT 门的插入规律看下面的代码就能理解：

```python
Logical X = [[0 0 0 0 1 1 1 0 0 0 0 0 0 0]]
Logical Z = [[0 0 0 0 0 0 0 1 1 0 0 0 0 1]]

encoding_circuit = []
for i in range(k):
    for j in range(r, n-k):
        if logical_xs[i, j]:
            encoding_circuit.append(["cx", n-k+i, j])

for i in range(r):
    encoding_circuit.append(["h", i])
    for j in range(n):
        if i == j:
            continue
        if standard_gens_x[i, j] and standard_gens_z[i, j]:
            encoding_circuit.append(["cx", i, j])
            encoding_circuit.append(["cz", i, j])
        elif standard_gens_x[i, j]:
            encoding_circuit.append(["cx", i, j])
        elif standard_gens_z[i, j]:
            encoding_circuit.append(["cz", i, j])
```

所以图中的前两根线完成的是逻辑 X 算符的注入，其余的线定义了所有 X/Z 稳定子：

> [!NOTE] Steane 码的稳定子
> X 和 Z 稳定子是完全对称的：
> $$
> XIXIXIX , XXIIXXI , XIXXIXI, ZIZIZIZ, ZZIIZZI , ZIZZIZI
> $$

看不懂稳定子是怎么编码的？下面这张图清晰很多：

![[Pasted image 20250627203155.png|400]]

除去编码逻辑算符那一列，还剩三列，每一列都对应了一个 X 稳定子和一个 Z 稳定子的测量。

- 从下往上看：经过 H 门后执行的是 4-body CX，测量 X 稳定子；
- 从上往下看：直接执行的是 4-body CZ，测量 Z 稳定子。

# Hardmard Test

![[Pasted image 20250627204941.png|200]]

这一步的原理实际上与稳定子测量线路一样！稳定子测量线路要保证数据比特位于稳定子的 +1 本征空间内，而这一步需要检验编码后的“大逻辑比特”是否在 Hardmard 的 +1 本征空间内。因此，辅助比特在 $\ket{+}$ 基下制备与测量。

为保证线路容错性，需要制备 cat qubits 并执行 trasversal CNOT，如果测得结果是偶数，则检测通过。

# Decoding

![[Pasted image 20250627204957.png|200]]

这一步实际上是一个 teleportation，通过 $\ket{+}$ 基制备 + CNOT 门将逻辑信息从数据比特传至辅助比特上，根据数据比特 Z 基测量的结果执行泡利修正，推导也很简单：

$$
\begin{aligned}
\ket{+}\otimes (a\ket{0_{L}}+b\ket{1_{L}}) \quad &\underrightarrow{\text{CNOT}} \quad \ket{0}\otimes ((a\ket{0_{L}}+b\ket{1_{L}})) + \ket{1} \otimes (a\ket{1_{L}}+b\ket{0_{L}}) \\
&= (a\ket{0}+b\ket{1})\otimes \ket{0_{L}} + (b\ket{0}+a\ket{1})\otimes \ket{1_{L}}
\end{aligned}
$$

第一步利用了 Steane code 上 CNOT 门的横向性，第二步中的第二项需要执行泡利 X 修正。
