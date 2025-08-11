---
date: 2025-07-30
---
- [[#Background|Background]]
	- [[#Background#Pauli Frame|Pauli Frame]]
	- [[#Background#Parallel Window Decoding|Parallel Window Decoding]]
- [[#Motivation|Motivation]]
	- [[#Motivation#Old|Old]]
	- [[#Motivation#New|New]]
- [[#Problems|Problems]]
- [[#Methodology|Methodology]]
	- [[#Methodology#一般情况：启发式优先级调度|一般情况：启发式优先级调度]]
	- [[#Methodology#紧急情况：并行窗口着色|紧急情况：并行窗口着色]]
		- [[#紧急情况：并行窗口着色#V1|V1]]
		- [[#紧急情况：并行窗口着色#V2|V2]]
- [[#Implementation|Implementation]]
	- [[#Implementation#总体架构|总体架构]]
	- [[#Implementation#开发文档|开发文档]]
		- [[#开发文档#1. 系统输入|1. 系统输入]]
		- [[#开发文档#2. 核心模块|2. 核心模块]]
			- [[#2. 核心模块#2.1. `class SimulationEngine`|2.1. `class SimulationEngine`]]
			- [[#2. 核心模块#2.2. `class Scheduler`|2.2. `class Scheduler`]]
			- [[#2. 核心模块#2.3. `class Timeline`|2.3. `class Timeline`]]
			- [[#2. 核心模块#2.4. `class SliceState`|2.4. `class SliceState`]]
			- [[#2. 核心模块#2.5. `class DecoderManager`|2.5. `class DecoderManager`]]
		- [[#开发文档#3. 主执行流程|3. 主执行流程]]
- [[#Experiment|Experiment]]
- [[#Concern|Concern]]
	- [[#Concern#硬件架构|硬件架构]]
		- [[#硬件架构#1. 输入缓冲与调度器 (Input Buffer & Scheduler)|1. 输入缓冲与调度器 (Input Buffer & Scheduler)]]
		- [[#硬件架构#2. 路由核心：解复用器 (The Routing Core: Demultiplexer)|2. 路由核心：解复用器 (The Routing Core: Demultiplexer)]]
		- [[#硬件架构#3. 更复杂的实现：片上网络 (Network-on-Chip, NoC)|3. 更复杂的实现：片上网络 (Network-on-Chip, NoC)]]
	- [[#Concern#总结|总结]]

## Background

### Pauli Frame

在 memory QEC 中，不需要执行逻辑门，因此用 Clifford 门执行的 syndrome measurement 的结果什么时候处理都可以，只需要一直重复 measurement - decoding - update Pauli frame 这个并行循环。

在 FTQC 中，non-Clifford 门的存在使得 [[../03_KnowCards/Pauli Frame|Pauli Frame]] 的记录无法穿过这种门，因此在执行 non-Clifford 门前必须把之前所有的 syndrome 解码完毕，并应用 Pauli 恢复。以 Surface code 为例，一般执行的通用门集是 Clifford+T，T 门通过 magic state distillation + [[../03_KnowCards/Gate Teleportation|Gate Teleportation]] 实现，那么就必须在 teleportation 的最后执行 S 门修正前同步解码结果，不然 Pauli frame 就要变成 Clifford frame，再经过下一个 T 门就无法用经典软件追踪了。

![[../attachments/Pasted image 20250730145915.png|600]]

**这给我们两个 insight：**
1. decoder 无需实时跟上 syndrome 流
2. decoder 必须在 T 门执行前处理完之前的 syndrome——T 门是 decoding bottleneck

两个 insight 看起来有“不和谐”之处：pauli frame 放松了对实时响应的要求，但 decoder 又必须跟上 T 门。decoder 的速度依然必须大于 syndrome measurement 的速度，否则未处理的 syndrome 会指数挤压（exponential backlog）。

### Parallel Window Decoding

Decoder 怎么接收数据？传统的 sliding window decoding 的思想是将 syndrome data 分割成固定长度的时间窗口，并按顺序对每个窗口解码。这样使得 decoder 处理的数据流可控，然而，这种方法是不可 scaling 的：

> [!Warning] Sliding window decoding is unscalable!
> 设每个 sliding window 包含 $n_{syn}$ 轮的 syndrome 数据，产生每轮数据的时间为 $\tau_{round}$，则生成 window 的时间为 $\tau_{gen}=n_{syn}\tau_{round}$。为了避免指数延迟，decoding 的速度需要满足 $\tau_{dec}<n_{syn}\tau_{round}$。
> 在乐观情况下——decoder 处理数据是线性复杂度，即 $\tau_{dec} = kn_{syn}N$，其中 $k$ 为常数、$N$ 为码长。代入上式得：
> $$
> N < \frac{\tau_{round}}{k}
> $$
> 该式意味着，为使得 decoder 能跟上 syndrome 的速度，使用的纠错码的大小有一个上界。

这个问题直到 2022 年 parallel window decoding 的提出才被理论上解决，又在 2025 年被进行了优化（speculative window decoding）。

- Parallel：时空上不相邻的窗口是不存在依赖关系的，因此可以并行解码。在一维时间轴上，只需要串行的两层解码就可以处理所有的窗口。
- [[../02_Info/Predictive Window Decoding for Fault-Tolerant Quantum Programs|Speculative]]：可以想办法移除窗口间的依赖，先在窗口边界处局部解码，之后就可以近似并行地解码所有窗口，如果正式解码后窗口边界的 prediction 和局部解码不符，则重启相邻窗口的解码。

三种 [[../03_KnowCards/Window Decoding|Window Decoding]] 的示意图如下：

![[../attachments/Pasted image 20250730152608.png]]

在空间上也可以执行 [[../03_KnowCards/Spatially Window Decoding 的窗口大小与错误率|Spatially Window Decoding]]，因此 FTQC window decoding 的最小单元是一个 $(d+w_{space},d+w_{space},d+w_{time})$ 的时空 slice。

**Insight**：只要给定足够数量的 decoder，就可以快速并行解决大量的解码任务，即使单个 decoder 的解码速度较慢。

## Motivation

### Old

我们以 surface code computation 为例。假设采用并行化的 window decoding，在时间轴上有许多并行的 window 需要分配 decoder 处理，在空间上，不同的 logical qubit 组成了二维的 block，也需要分配多个 decoder 进行处理。这种方法的经典资源成本和功耗会非常高，在 $\tau_{dec}<\tau_{gen}$ 的条件下也会产生性能浪费。

设想在下图的 block 下进行容错量子计算，包含对多个逻辑比特 patch 的操作。该怎么分配 decoder？

![[../attachments/Pasted image 20250730110744.png|300]]

1. One to one：每个 decoder 解码一个 patch，有多少逻辑比特就要多少个 decoder，并行性 max。问题：
	- 使得经典控制系统的尺寸和功耗高昂，且资源浪费
	- 如何识别不同 patch 之间的 correlated error？
2. One for all：常数个 decoder 解码所有 patch，要求延迟 $~O(1/N_{lq})$，导致算法不可扩展
3. M for N：
	- 假设 $M\leq N$ 且 $\tau_{dec}<\tau_{gen}$，需要一个调度方案以高效分配高速的 decoder。
	- 假设 $M> N$ 且 $\tau_{dec}\geq\tau_{gen}$，需要在合适的时机启用并行解码以弥补单个 decoder 的慢速。

高速 decoder 的 idea 被 [Managing Classical Processing Requirements for Quantum Error Correction](https://arxiv.org/abs/2406.17995v2) 抢发了。

这篇论文考虑的是编译层确定后，对 decoder 的调度（最少平均等待时间）。

- **没有考虑并行窗口解码**：如果调度器分配 decoder 给一个 logical qubit，就要让它处理掉该 qubit 目前为止的所有 syndrome，是部分串行化的。这样做有一个问题就是常用的 MWPM 和 UF 都是超线性复杂度的，一次解码多个 slice 的时间肯定是次优的。如果考虑并行窗口，每次关键解码（T 门前的 qubit）产生时，都可以用并行的数个 decoder 协同解决，但调度问题会增加一个维度，变得复杂。
- **没有做编译层优化**：线路中的 T 门是可以和 Clifford 门交换的，因此可以手动让量子算法的 T 门变得均匀。甚至可能可以考虑编译和解码层的运行时交互？
- **没有考虑关联错误**：
	- 时间关联错误需要我们自己实现——在调度时考虑 decoder 的局域性？加先验？
	- lattice surgery 会导致空间关联错误吗？

### New

[[../03_KnowCards/Parallel Window Decoding 是不可扩展的|Parallel Window Decoding 是不可扩展的]]

## Problems

验证和执行这个 idea 可能涉及到的问题：

1. Decoder 数量较少时，如何进行 spatially parallel window decoding？
	[Spatially parallel decoding for multi-qubit lattice surgery](https://arxiv.org/abs/2403.01353)
2. 如何调度 decoder？
	[Managing Classical Processing Requirements for Quantum Error Correction](https://arxiv.org/abs/2406.17995v2)
	[A Scalable Decoder Micro-architecture for Fault-Tolerant Quantum Computing](https://arxiv.org/pdf/2001.06598)
3. lattice surgery 会引入空间关联错误吗？（大坑，PRL/PRXQ）
	[Correlated decoding of logical algorithms with transversal gates](https://arxiv.org/abs/2403.03272)
	[Algorithmic Fault Tolerance for Fast Quantum Computing](https://arxiv.org/abs/2406.17653)
	[Error Correction of Transversal CNOT Gates for Scalable Surface-Code Computation](https://journals.aps.org/prxquantum/pdf/10.1103/PRXQuantum.6.020326)

## Methodology

经典计算资源给定的情况下（decoder num $M$, speed $r_{dec}/r_{gen}$）

编译器：

- 尽量使 T 门更稀疏
- 将操作时间线和空间关联情况发送给调度器：
	- 每个（logical qubit，d rounds）是一个时空 slice，即调度单位
	- 将每个 $\pi/8$ 门后面的 Clifford 修正标注为关键解码，具有 deadline，关键解码前的每个 slice 都有截止计时
	- 标注操作的空间关联情况

调度器： 将 A 个 qubit，各 B 个窗口分配给 M 个 decoder，$M = \sum_{i}A_{i}B_{i}$。设计概要：

- 一般情况：维护一个启发式优先级函数，考虑时空依赖的情况下，将 M 个资源分配给现有的优先级最高的 slices
- 紧急情况：产生的 syndrome 已经接近 Clifford 修正 $t_{c}$ 时（比如 $t_{c}-2$），启动紧急子任务，将与该 Clifford 修正相关的、尚未被解码的 slices 构成约束图，用无向边表示时空关联，返回一个并行窗口调度方案。

### 一般情况：启发式优先级调度

启发式函数的设计可能需要根据紧急情况的特性来设计，在实验中调整。以下是一些不同的思路：

1. 随机：顾名思义
2. 优先最长时间未解码：防止哪个 slice 变饥饿，但在可以使用并行窗口的前提下似乎不用担心饥饿问题
3. 优先最多最紧急关键解码：未雨绸缪

### 紧急情况：并行窗口着色

#### V1

~~（取名 Wavefront Coloring Algorithm）~~

- 建图：与即将执行的 Clifford 修正递归相关的、尚未被解码的 slices
- 将当前可解码的，度数最多的一个 slice 着色为 1
- 根据关联递归着色：
	- 与 slice $i$ 相连的 $j$，$j$ 可以取其邻居着色之外的、不小于其所在的时间点的最小值（前者保证依赖性，后者保证该时间点的 syndrome 已存在）

![[../attachments/487cae2532e6d5338369cb272f93aea5.jpg|400]]

时间复杂度：$O(V+E)$，与 slices 数量呈线性关系，可认为高效

适用范围：单个 decoder 的解码速度不超过 syndrome 生成速度

**待解决的问题**：
1. 若单个 decoder 的速度超过 syndrome 生成速度，给着色算法限制最大并行数量，还适用吗？还高效吗？

**好处**：
1. 可以在紧急时集中力量办大事，最大程度利用并行窗口的特点
2. 给 decoder 只分配原子任务（single qubit，$d$ rounds），防止其多项式复杂度带来的额外时间

> [!fail] V1 的问题
> 窗口的解码时间与缓冲区数量和大小有关，而时空窗口的缓冲区显然各不相同，因此解码任务是异步、动态的，而不是静态的。环境变得更加复杂，需要重新设计调度算法。

#### V2

当已生成 syndrome 的 slice 的 deadline 小于等于 4 时，启动紧急模式。每当产生了新的 syndrome 或现有的解码任务被释放，调度器都要运行新着色算法以分配 decoder。分配逻辑为：

1. 选取相互不具有依赖关系的、度最小的 slice，分配 decoder，更新周围块的依赖。（实现方法：把当前能够解码的 slice 按邻居数量排序，从小到大遍历，每选取一个，记录它的邻居，这些邻居不能被选取，直到把当前能够解码的 slice 遍历完毕）
2. 如果产生了新的 syndrome 且资源池有剩余算力，则查看新产生的 slice，如果其没有正在解码的依赖，且度数小于等于 2，则直接分配 decoder，更新周围块的依赖。（有依赖则只能等依赖块解码完毕；度数约束防止在依赖图的中心区域解码过久，使得其后继 slice 一直等待）
3. 如果现有的 slice 解码完毕，则更新周围块的依赖，查看是否有能分配 decoder 的，如有则分配并更新依赖。
4. 何为更新依赖：当分配一个 slice 任务时，其周围的邻接（有依赖关系的）slice 需要进入等待状态（不能被分配 decoder）；当结束一个 slice 任务时，释放其邻接 slice 的状态，并将这些 slice 的度数减 1。

![[../attachments/1e671bcd08af821fc60837b460e881fd.jpg|400]]

现有的问题：

如何保证紧急模式能优先使用 decoder？这样的动态调度好像可以预计算，可以预先估计在这个关键解码任务中需要并行使用多少 decoder 并预留？

拓展：这个 setting 在计算机科学中是否有抽象描述？

## Implementation

### 总体架构

![[../attachments/fb19a0ac2b172077954ec816e62baa07.jpg|800]]

1. （已有实现）编译器可以生成编译后的中间指令 LLI，即什么时候应该在哪些 slice 上执行什么操作。
2. （**需要实现**）将 LLI 解析成 timeline 表，每一项是一个 time，每个 time 包括所有 slice 的坐标、执行的操作（pi/4 门，pi/8 门，rotation 当前状态，idle），LLI 给出的 rotation 包括连续 3 个 time 的操作（patch_deformation, move_corner, move_patch），需要进行拆解。每个 time 的每个 slice 要有一个值计算 deadline，即离下一次 clifford 修正还有几步（下一次 clifford 修正为下一次 pi/8 门后跟的第一个 pi/4 门）。
3. （理论上有，但暂时不需要实现）LLI 被发送给量子计算机（QC），生成对应时刻的 syndrome，存在 timeline 表中。
4. （**需要实现**）调度器通过 curr_time 这个指针与 timeline 互动。curr_time 初始为 1，即刚刚生成一层 syndrome；每次 decoding 需要消耗时间，消耗的时间会是小数，由 decoder 的速率决定（速率和 decoder 数量都是预先输入的参数），一轮 decoding 后更新 curr_time，向下取整就是调度器能够拿到的 syndrome 的时间。
5. （**需要实现**）调度器有一个等待队列，队列中维护着计算出的对未来 syndrome 的 decoder 分配。
6. （**需要实现，重点**）调度器知道当前所处时间后，更新得到目前未被处理的 syndrome（以（time，coord）时空坐标为单位）。首先看最新 time 中有没有快到 deadline 的 slice（deadline 计数为 2），有的话调用并行窗口着色算法，优先将资源分配到这些 slice（需要考虑每一个 time 的 decoder 数量约束），并更新等待队列。执行完毕后（或没有 deadline slice）通过启发式优先级函数更新每个其他 slice 的优先级，并将当前 time 剩余的 decoder 资源分配到这些 slice 上。更新这一轮 decoding 结束后的 curr_time。
	注：只有并行窗口着色算法会对未来的 syndrome 预分配 decoder，普通优先级调度只会关注当前时间。
7. （在框架搭成后实现）并行窗口着色算法和优先级函数。
8. （**需要实现，重点**）Clifford 修正执行前必须确保该 slice 历史的 syndrome 已被 decoder 处理完毕，否则需要添加一层 idle（严谨的做法是只为这个 slice 添加 idle，先实现简化版本）（若这个 slice 已经是 idle 则跳过）。
9. （资源统计）统计由于 decoding 未完成而添加的 idle 层的数量，计算端到端逻辑错误率。

系统的输入是 decoder 数量和相对速率（t_decoding / t_syndrome）。

### 开发文档

#### 1. 系统输入

* **LLI 文件路径 (`lli_path`)**: `str` - 由编译器生成的指令文件，描述了在哪个时间、哪个空间坐标（slice）上执行何种操作。
* **解码器数量 (`num_decoders`)**: `int` - 可用的并行解码器总数（M）。
* **相对解码速率 (`relative_speed`)**: `float` - 单个解码器处理一个原子解码任务所需时间与一个 syndrome 测量周期时间的比值 (t_decoding / t_syndrome)。

#### 2. 核心模块

##### 2.1. `class SimulationEngine`

**描述**: 系统的顶层控制器，负责初始化所有组件、驱动主模拟循环并收集最终的统计数据。

| 属性                           | 类型          | 描述                       |
| :--------------------------- | :---------- | :----------------------- |
| `timeline`                   | `Timeline`  | 存储整个计算过程状态的 Timeline 实例。 |
| `scheduler`                  | `Scheduler` | 负责决策的核心调度器实例。            |
| `total_idle_layers_inserted` | `int`       | 统计因解码延迟而插入的 `idle` 层总数。  |

| 方法      | 参数  | 返回     | 描述                                                                            |
| :------ | :-- | :----- | :---------------------------------------------------------------------------- |
| `run()` | -   | `None` | 启动并完整执行整个模拟流程。详细步骤见 [[Idea - Wavefront Scheduling - A Compiler-Informed Decoding Scheduler for Real-time Quantum Error Correction#3. 主执行流程\|主执行流程]]。 |

##### 2.2. `class Scheduler`

**描述**: 系统的决策核心。根据当前时间和 `Timeline` 中的状态，决定如何将解码器资源分配给待处理的 syndrome。

| 属性                | 类型               | 描述                      |
| :---------------- | :--------------- | :---------------------- |
| `timeline`        | `Timeline`       | 对系统中心 `Timeline` 对象的引用。 |
| `decoder_manager` | `DecoderManager` | 管理解码器资源池的实例。            |
| `curr_time`       | `float`          | 浮点数，精确追踪当前模拟的虚拟时间。      |
| `waiting_queue`   | `List[Task]`     | 存储由并行窗口着色算法预分配的未来解码任务。  |

| 方法                                          | 参数                                                                 | 返回           | 描述                                                                                          |
| :------------------------------------------ | :----------------------------------------------------------------- | :----------- | :------------------------------------------------------------------------------------------ |
| `__init__(…)`                               | `timeline: Timeline`, `num_decoders: int`, `relative_speed: float` | `None`       | 构造函数，初始化所有属性。                                                                               |
| `run_one_cycle()`                           | -                                                                  | `float`      | **调度器主循环**。在一个调度周期内，决定所有要执行的解码任务，并返回这些任务所需的总时间。                                             |
| `check_sync_and_request_idle()`             | -                                                                  | `bool`       | **关键同步检查**。检查当前时间点需要执行 Clifford 修正的 slice，其所有历史 syndrome 是否都已解码。若否，返回 `True`，表示需要插入 `idle`。 |
| `update_time(duration)`                     | `duration: float`                                                  | `None`       | `self.curr_time += duration`，推进模拟时间。                                                        |
| `_run_coloring_algorithm(slices)`           | `slices: List[SliceState]`                                         | `List[Task]` | 实现**并行窗口着色**算法，处理紧急任务。                                                                      |
| `_run_heuristic_scheduling(slices, budget)` | `slices: List[SliceState]`, `budget: int`                          | `List[Task]` | 实现**启发式优先级函数**，处理一般任务。                                                                      |

##### 2.3. `class Timeline`

**描述**: 系统的中心数据仓库。负责解析 LLI 文件，并以结构化的方式存储每个时空片（slice）的状态。

| 属性     | 类型                                   | 描述                                                                |
| :----- | :----------------------------------- | :---------------------------------------------------------------- |
| `data` | `Dict[int, Dict[Tuple, SliceState]]` | 核心数据结构。键为时间步 `time`，值为一个字典，该字典的键为空间坐标 `coord`，值为 `SliceState` 对象。 |

| 方法                             | 参数                          | 返回           | 描述                                                                                                                         |
| :----------------------------- | :-------------------------- | :----------- | :------------------------------------------------------------------------------------------------------------------------- |
| `__init__(lli_path)`           | `lli_path: str`             | `None`       | 构造函数，调用内部的 `_parse_lli` 方法。                                                                                                |
| `_parse_lli()`                 | -                           | `None`       | **LLI 解析器**。负责读取 LLI 文件，填充 `data` 属性。关键职责包括：1. 拆解多步操作。2. **计算 `deadline`**: 从后向前扫描，为每个 slice 计算其到下一次关键 Clifford 修正的剩余时间步数。 |
| `get_slice_state(time, coord)` | `time: int`, `coord: Tuple` | `SliceState` | 获取特定时空片的状态对象。                                                                                                              |
| `insert_idle_time(at_time)`    | `at_time: int`              | `None`       | 在指定时间点 `at_time` 插入一个 `idle` 层。此操作会将 `at_time` 及之后的所有数据的时间戳向后顺延。                                                           |
| `is_complete(curr_time)`       | `curr_time: float`          | `bool`       | 检查整个计算任务是否已完成。                                                                                                             |

##### 2.4. `class SliceState`

**描述**: 一个数据类（Data Class），用于封装在特定时间和空间坐标点上的所有状态信息。

| 属性                | 类型                | 描述                                                                                                                                           |
| :---------------- | :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| `time`            | `int`             | 时间坐标。                                                                                                                                        |
| `coord`           | `Tuple[int, int]` | 空间坐标。                                                                                                                                        |
| `operation`       | `str`             | 该 slice 上执行的操作（如 'pi/8', 'idle'）。                                                                                                            |
| `deadline`        | `int`             | 距离下一次关键 Clifford 修正的剩余时间步数。                                                                                                                  |
| `neibor`          | `bool`*4          | 与时空邻居的依赖关系（上下左右）                                                                                                                             |
| `syndrome_data`   | `Any`             | 存储实际的 syndrome 测量结果（初期可为占位符）。                                                                                                                |
| `decoding_status` | `Enum`            | 解码状态：<br>`UNGENERATED`：还未生成 syndrome<br>`PENDING`：已生成 syndrome，等待解码<br>`ASSIGNED`：已分配 decoder<br>`OCCUPIED`：等待依赖解码，无法分配<br>`COMPLETED`：已解码完毕 |

##### 2.5. `class DecoderManager`

**描述**: 管理解码器资源池的抽象层。

| 属性               | 类型           | 描述              |
| :--------------- | :----------- | :-------------- |
| `total_decoders` | `int`        | 可用解码器总数。        |
| `relative_speed` | `float`      | 单个解码器相对速率。      |
| `busy_decoders`  | `List[Task]` | 记录当前正在执行任务的解码器。 |

| 方法                                   | 参数               | 返回      | 描述                                      |
| :----------------------------------- | :--------------- | :------ | :-------------------------------------- |
| `assign_task(task)`                  | `task: Task`     | `None`  | 分配一个解码任务。                               |
| `get_free_decoder_count()`           | -                | `int`   | 返回当前空闲的解码器数量。                           |
| `calculate_decoding_time(num_tasks)` | `num_tasks: int` | `float` | 根据此次分配的任务总数、解码器总数和相对速率，计算完成这些任务所需的虚拟时间。 |

#### 3. 主执行流程

`SimulationEngine.run()` 方法驱动的主循环逻辑如下：

1. **初始化**: 创建 `Timeline` 和 `Scheduler` 实例。`total_idle_layers_inserted` 置零。
2. **进入循环**: 只要模拟未结束（`!timeline.is_complete(…)`），就持续循环。
3. **同步检查**: 调用 `scheduler.check_sync_and_request_idle()`。
    * 如果返回 `True`，说明有解码任务落后，必须等待。
    * 此时，调用 `timeline.insert_idle_time()` 在当前时间点插入一个 `idle` 层。
    * `total_idle_layers_inserted` 计数加一。
    * 跳过本轮后续步骤，直接开始下一轮循环（因为时间表已改变，需重新评估）。
4. **调度周期**: 如果同步检查通过（返回 `False`），则调用 `scheduler.run_one_cycle()`。
    * 此方法内部会执行紧急和一般调度逻辑，并将任务分配给 `DecoderManager`。
    * 方法返回完成本轮所有解码任务所需的总时间 `duration`。
5. **时间推进**: 调用 `scheduler.update_time(duration)`，将模拟器的当前时间向前推进。
6. **循环结束**: 当模拟完成后，循环终止。
7. **报告结果**: 打印 `total_idle_layers_inserted` 等统计数据。

## Experiment

预期实验图：

1. 慢速 decoder：延迟时间 / LER 与 decoder 数量的关系，一定有一个软饱和点
2. 快速 decoder：同上，并计算与 one-to-one 相比带来的开销减少量
3. 慢速 - baseline 对比：随机 / 先入先出调度器（T 门等待时间更多、导致 LER 高）
4. 快速 - baseline 对比：Tannu 那一篇，那篇有时会一口气解码很多个 slice，由于解码算法是超线性的，这样一定是 sub-optimal 的
5. 待续

## Concern

1. 解码硬件不是物理上固定的吗？你又怎么调度？

答案是：我们通过**可编程的“数据开关”和“路由网络”**来实现。物理线路本身是接死的，但数据具体走哪条线路，是由你的调度器动态控制的。

想象一个复杂的铁路枢纽。所有的铁轨（物理线路）都是铺设好的，无法移动。但调度中心（你的调度器）可以通过控制**道岔（数据开关）**，让一列火车（syndrome 数据）从 A 站台开往 C 站台，而下一列火车开往 F 站台。

在硬件（如 FPGA 或 ASIC）中，这个“道岔”就是**多路复用器 (Multiplexer, MUX)** 和 **解复用器 (Demultiplexer, DEMUX)**。

下面是一个典型的实现架构，我们分步来看：

### 硬件架构

#### 1. 输入缓冲与调度器 (Input Buffer & Scheduler)

- **Syndrome 数据流**从量子芯片接口进入，首先被存放在一个**输入缓冲池 (Input Buffer / FIFO)** 中。这就像乘客在进入地铁站时，先在站台等候区排队。
    
- 你的**调度器 (Scheduler)** 是一个独立的逻辑单元。它的物理位置也是固定的。它的工作是：
    
    - 查看输入缓冲池，了解有哪些“乘客”（syndrome slices）在等待。
        
    - 监控所有解码器（Decoders）的状态，知道哪个“站台”是空闲的。
        
    - 根据其调度算法（比如负载均衡或基于复杂度的判断），做出决策：“将下一个 syndrome slice 发送给 3 号解码器”。

#### 2. 路由核心：解复用器 (The Routing Core: Demultiplexer)

这是回答你问题的核心。调度器的决策会变成一个二进制的**控制信号**，送到一个巨大的“数据开关”——**解复用器 (DEMUX)**。

- **DEMUX**有一个数据输入端和多个数据输出端。
    
- **输入端**连接着输入缓冲池，准备好了要被发送的 syndrome 数据。
    
- **每一个输出端**都有一条**固定的物理线路**，分别连接到**每一个解码器**的输入端。是的，线路是接死的，从 DEMUX 到每个 Decoder 都有一条专属通道。
    
- **控制信号**（来自你的调度器）决定了在某一时刻，输入的数据会从**哪一个**输出端流出。

**工作流程示意图：**

```
                               +---------------------+
                               |  Scheduler Logic    |  <-- 监控所有Decoder状态
                               +---------------------+
                                         |
                                         | (决策: "用Decoder 2")
                                         V
+-----------------+      +-----------------------------+      +----------------+
| Syndrome Data   |      |                             |----->|   Decoder 1    |
| from Buffer/FIFO|----->|     DEMULTIPLEXER (开关)    |      +----------------+
+-----------------+      | (由Scheduler的决策信号控制) |      +----------------+
                       |                             |----->|   Decoder 2    | <-- 数据流向这里
                       +-----------------------------+      +----------------+
                                       |                    +----------------+
                                       +------------------->|   Decoder 3    |
                                                            +----------------+
                                                                   ...
```

关键点：

你看到的 --> 都是永久存在的物理线路。调度器并没有改变布线，它只是通过控制 DEMUX，决定了数据流在这一瞬间激活了哪条路径。所有其他路径虽然物理上存在，但没有数据流通过。

#### 3. 更复杂的实现：片上网络 (Network-on-Chip, NoC)

当解码器数量非常多时（比如几十上百个），一个巨大的 DEMUX 会变得非常低效且难以布线。这时，工程师会采用更先进的架构：**片上网络 (Network-on-Chip, NoC)**。

这就像把互联网的原理微缩到了芯片上。

1. **打包数据 (Packetization):** 调度器不再只是生成一个简单的控制信号。它会将 syndrome 数据封装成一个**数据包 (Packet)**。这个包里除了数据本身，还有一个**目标地址**（例如：“致 3 号解码器”）。
    
2. **注入网络 (Injection):** 调度器将这个数据包“注入”到片上网络中。
    
3. **路由转发 (Routing):** NoC 由一系列微小的、固定的**路由器 (Routers)** 和它们之间的链路组成。每个路由器看到数据包，会读取其目标地址，然后像快递分拣员一样，把它转发给下一个更靠近目标的路由器。
    
4. **到达目的地 (Ejection):** 经过几次转发，数据包最终到达目标解码器门口的路由器，然后被提取出来送入解码器。

这种方式扩展性极强，你的调度器只需要负责“打包和贴标签”，而底层的 NoC 硬件会负责高效、并行的“快递投送”。

### 总结

“我的调度器是一个**高层逻辑决策单元**，它并不关心物理布线的细节。它的决策结果（比如一个目标地址）会被转换成底层的控制信号。这些信号去操控一个**固定的路由硬件**（如 DEMUX 或片上网络），来引导数据流向一个**物理位置固定**的解码器。我们动态调度的是**数据**，而不是硬件本身，从而完美地在静态的物理约束下实现了动态的负载管理和智能分配。”
