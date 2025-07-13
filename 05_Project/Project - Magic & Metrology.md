
> [!Question] 猜想：信道的每个 Kraus 的每行只有一个元素 $\iff$ Parallel 策略是否存在 QFI gap

> [!NOTE] 引理 1：
> $$
> wI - M \geq 0 \iff w \geq \lambda_{max}(M)
> $$

证明见 [[半正定与最大特征值]]。

> [!NOTE] 引理 2：相空间点算符的对角元处处非负。

证明见 [[Phase-Point Operator 的对角元只有 1 和 0]]。

> [!Success] 定理 1：若待测信道对应的性能算符对输出维度作偏迹后 $\mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h)$ 对角，则对该信道的并行测量策略无量子优势。

**证明**：我们考虑如下原始优化问题与其在 CPWP（完全正且 Wigner 保持）限制下的对偶形式 [[SDP]]：

无约束情形（原问题）：

$$
\begin{aligned}
\text{minimize} \quad & w \\
\text{subject to} \quad & w I \geq \mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h)
\end{aligned}
$$

CPWP 限制下的优化问题：

$$
\begin{aligned}
\text{minimize} \quad & w_{\mathrm{cpwp}} \\
\text{subject to} \quad & w_{\mathrm{cpwp}} I \geq \mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h) + \sum_{j=1}^{d^{2N}} a_j M_j \\
& a_j \geq 0, \quad j = 1, \dots, d^{2N}
\end{aligned}
$$

其中每个 $M_j$ 是所有 $A_{\mathbf{u}}$ 元素的两两张量积形式组成的正交基底中的一项，可写为

$$
M_j = A_{\mathbf{u}}^{\otimes N}.
$$

根据引理 1，原问题的最优值记作：

$$
w^* := \lambda_{\max}\left( \mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h) \right),
$$

设该最大特征值对应的特征向量为 $\ket{e_w}$。由于 $\mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h)$ 是对角矩阵，其特征向量 $\ket{e_w}$ 属于计算基。

我们将 CPWP 问题的最优值记作：

$$
w^*_{\mathrm{cpwp}} := \lambda_{\max}\left( \mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h) + \sum_j a_j M_j \right).
$$

由于最大特征值的单调性，我们有：

$$
\begin{aligned}
w^*_{\mathrm{cpwp}} 
&\geq \bra{e_w} \left( \mathrm{Tr}_{2,4,\dots,2N} \, \Omega_\phi(h) + \sum_j a_j M_j \right) \ket{e_w} \nonumber \\
&= w^* + \sum_j a_j \bra{e_w} M_j \ket{e_w} \nonumber \\
&= w^* + \sum_j a_j (M_j)_{ww}.
\end{aligned}
$$

由引理 2（即 $A_{\mathbf{u}}$ 的对角元非负性）与 $a_j \geq 0$ 可知，上式第二项非负，因此有

$$
w^*_{\mathrm{cpwp}} \geq w^*.
$$

为了实现最小化，必须有 $\sum_j a_j (M_j)_{ww} = 0$，这要求所有 $a_j = 0$，因而：

$$
\boxed{w^*_{\mathrm{cpwp}} = w^*}.
$$

> [!NOTE] 推论 1：并行策略下对角哈密顿量的测量不具有量子优势
> 在奇数维度的 qudit 系统中，若采用并行量子计量策略，且待测信道的哈密顿量在计算基下为对角形式，则无论该策略是否使用 magic resource，其最优量子费舍信息（QFI）值相同。

**证明**：目标为证明 $\mathrm{Tr}_{2,4,\dots,2N}\Omega_{\phi}(h)$ 是对角的。
$$
H(\phi) = \text{diag}\{f_{1}(\phi), f_{2}(\phi), f_{3}(\phi)\}
$$

待测信道为：

$$
U(\phi) = e^{-iH(\phi)t} = \text{diag}\{e^{-itf_{1}(\phi)},e^{-itf_{2}(\phi)},e^{-itf_{3}(\phi)}\} = \text{diag}\{g_{1}(\phi),g_{2}(\phi),g_{3}(\phi)\}
$$

将单一 Kraus 算子转为 Choi 向量，并求导：

$$
\begin{aligned}
\ket{C_{\phi}} &= \text{vec}(U(\phi)) = \sum_{i} g_{i}(\phi) \ket{i}\ket{i} \\
\ket{\dot{C_{\phi}}} &= \sum_{i} g_{i}'(\phi)\ket{i}\ket{i}
\end{aligned}
$$

对于 N-fold 的 probe state，整个复合待测信道的 Choi 向量为：

$$
\begin{align}
    \ket{N_\phi} &:= \ket{C_\phi}^{\otimes N} = \sum_{\mathbf{i}} G_{\mathbf{i}}(\phi) \ket{\mathbf{i}}_{\text{odd}} \otimes \ket{\mathbf{i}}_{\text{even}} \\
    \text{with } \quad G_{\mathbf{i}}(\phi) &:= \prod_{k=1}^N g_{i_k}(\phi), \quad \mathbf{i} = (i_1, \dots, i_N)
\end{align}
$$

对其求导得：

$$
\begin{equation}
    \ket{\dot{N}_\phi} := \partial_\phi \ket{N_\phi} = \sum_{\mathbf{i}} \dot{G}_{\mathbf{i}}(\phi) \ket{\mathbf{i}}_{\text{odd}} \otimes \ket{\mathbf{i}}_{\text{even}}
\end{equation}
$$

其中，

$$
\begin{equation}
    \dot{G}_{\mathbf{i}}(\phi) = \left( \sum_{m=1}^N \frac{\dot{g}_{i_m}(\phi)}{g_{i_m}(\phi)} \right) G_{\mathbf{i}}(\phi)
\end{equation}
$$

我们分析的是纯信道，Choi rank=1（[[Choi 矩阵的形式 & 信道的秩]]），因此扰动量 $h$ 只需要取标量，则 Choi 矩阵的任意分解可写为：

$$
\begin{aligned}
    \ket{\dot{\tilde{N}_\phi}} &:= \ket{\dot{N}_\phi} - i h \ket{N_\phi} \\
    &= \sum_{\mathbf{i}} (\dot{G}_{\mathbf{i}}(\phi) - i h G_{\mathbf{i}}(\phi)) \ket{\mathbf{i}}_{\text{odd}} \otimes \ket{\mathbf{i}}_{\text{even}} \\
    &= \sum_{\mathbf{i}} \tilde{G}_{\mathbf{i}}(\phi) \ket{\mathbf{i}}_{\text{odd}} \otimes \ket{\mathbf{i}}_{\text{even}}
\end{aligned}
$$

定义 [LHYY] 中的性能算符：

$$
\begin{equation}
    \Omega_\phi(h) := \ket{\dot{\tilde{N}_\phi}}\bra{\dot{\tilde{N}_\phi}} 
\end{equation}
$$

对输出维度的子系统 ($2,4,\dots,2N$) 求导，我们得到：

$$
\begin{align}
    \mathrm{Tr}_{2,4,\dots,2N} \Omega_\phi(h) 
    &= \sum_{\mathbf{i}, \mathbf{j}} \tilde{G}_{\mathbf{i}}(\phi) \tilde{G}_{\mathbf{j}}^*(\phi)
    \braket{\mathbf{j} | \mathbf{i}}_{\text{even}} \ket{\mathbf{i}}_{\text{odd}} \bra{\mathbf{j}}_{\text{odd}} \\
    &= \sum_{\mathbf{i}} \left| \tilde{G}_{\mathbf{i}}(\phi) \right|^2 \ket{\mathbf{i}}_{\text{odd}} \bra{\mathbf{i}}_{\text{odd}}
\end{align}
$$

显然 $\mathrm{Tr}_{2,4,\dots,2N} \Omega_\phi(h)$ 中只包含对角元，证毕。

> [!NOTE] 推论 2：当待测信道的每个 Kraus 算符每一行的非零元数量不超过 1，则奇数维 qudit 系统中的并行策略无量子优势。

我们只需要证明待测信道满足条件时，对应的性能算符是对角的。

未来研究方向：

1. 什么情况下能够减小特征值（反命题）
2. 能否量化减小特征值的程度（信道的量子度？）
