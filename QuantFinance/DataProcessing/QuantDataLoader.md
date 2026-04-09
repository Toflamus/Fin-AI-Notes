# 量化定制 DataLoader：截面分组与按日批次

量化深度学习中，数据加载器的设计与传统 CV/NLP 完全不同。本文记录定制版 `DataLoader` 与 PyTorch 原生 `torch.utils.data.DataLoader` 的核心区别，以及为什么"一天一个 Batch"是横截面选股的标准设计。

> 前置：[特征净化与降维](FeatureOptimization.md)

---

## 一、定制 DataLoader vs 原生 DataLoader

### 1. 核心批次策略：按时间分组 vs 无状态随机

| 维度 | 量化定制版 | PyTorch 原生 |
|------|-----------|-------------|
| 批次单位 | **同一天的所有股票** = 一个 Batch | 固定 `batch_size` 顺序/随机采样 |
| 实现方式 | `groupby('date')` + `iter_daily()` | `Dataset` + `Sampler` + `collate_fn` |
| 适用模型 | HIST、ALPHAS60 等截面排序模型 | 通用 CV/NLP |

**为什么必须按日分组？**

量化截面模型需要在 Batch 内提取**股票间的横向特征**（相关性、排名）。如果一个 Batch 混入多天数据，模型会把"周一的苹果"和"周三的特斯拉"放一起比较 → 引入未来函数和噪声。

### 2. `pin_memory` 的"魔改"

| | 量化定制版 | PyTorch 原生 |
|---|-----------|-------------|
| `pin_memory=True` 含义 | **直接把全部数据塞进 GPU 显存** | 加载到 CPU 锁页内存 (Page-Locked Memory) |
| 实现 | `torch.tensor(..., device='cuda')` | 训练时按 Batch 通过 PCIe 传输 |
| 优点 | 训练时零通信开销，速度极快 | 平衡内存/显存占用 |
| 缺点 | **极度消耗显存**，仅适用中小数据集 | 每个 Batch 有传输延迟 |

### 3. 数据流 API 设计模式

```python
# 量化定制版：迭代器只 yield 索引，需手动调用 get()
for slc in loader.iter_daily():
    features, labels = loader.get(slc)
    
# 原生：直接 yield 打包好的张量
for batch in dataloader:
    features, labels = batch
```

### 4. 架构耦合度

| | 量化定制版 | 原生 DataLoader |
|---|-----------|----------------|
| 设计 | All-in-One，存储+采样+打包全在一个类 | 解耦：Dataset + Sampler + collate_fn + DataLoader |
| 灵活性 | 业务场景下直观 | 通用，可组合性高 |

### 5. 多进程

| | 量化定制版 | 原生 |
|---|-----------|------|
| 并发 | 单线程同步 | `num_workers` 多进程异步预加载 |
| 适用 | 已处理好的结构化表格（DataFrame） | 大图片、音频、复杂 CPU 预处理 |

> 量化数据多为结构化表格，无需多进程加速；GPU 显存预加载策略带来的提速反而更显著。

---

## 二、为什么必须"一天一个 Batch"

### 1. 横截面视角的破坏与保留

| 策略 | 模型视角 | 后果 |
|------|---------|------|
| **一天一个 Batch** | 看到当日全市场 5000 只股票的全貌 | ✓ 可做 Cross-sectional Attention、横截面 BN |
| 几天混合一个 Batch | 跨日股票放一起比较 | ✗ 引入未来函数，金融逻辑无意义 |

### 2. 损失函数的逻辑

量化常用 IC / Rank IC，对应 Listwise Rank Loss 或 Correlation Loss：

- **同一截面**：直接算 Spearman 相关系数，排名公平
- **跨日混合**：模型会试图对不同日期的股票全局排序，被宏观波动主导，丧失选股能力

### 3. 显存物理限制

假设 5000 只股票 × 60 天 × 10 维特征：

- 一天一个 Batch：Batch Size = 5000，单卡舒适区
- 5 天一个 Batch：Batch Size = 25000，**直接 OOM**

### 4. 适用场景判断

| 任务类型 | 推荐策略 |
|---------|---------|
| 横截面选股（股票间排名/关联） | **必须按日分组** |
| 单股票时序预测（高频、择时）且模型无跨股票交互 | 可以混合多天，甚至有助于打破时间自相关，提升泛化 |
