# Batch Size 在量化与语言模型中的本质区别

`batch_size` 这个看似一致的概念，在量化截面模型和大语言模型 (LLM) 中的物理意义和底层逻辑完全不同。一个决定模型的视野，另一个只是工程加速器。

> 相关：[量化定制 DataLoader](../QuantFinance/DataProcessing/QuantDataLoader.md) | [Normalization](../AI/DeepLearning/Normalization.md)

---

## 一、样本间的"交互性" vs "独立性"

| | 量化截面 Batch | LLM Batch |
|---|---------------|-----------|
| Batch 内样本关系 | **相互关联**，需要横向比较 | **绝对独立**，互不可见 |
| Attention 范围 | 跨股票（Cross-sectional Attention） | 仅在单个序列内部 |
| 例子 | 5000 只股票同 Batch，A 的预测受 B/C 影响 | 32 段无关文本同 Batch，A 与 B 完全隔离 |

---

## 二、张量维度的含义

### 量化截面模型

```
[Num_Stocks, Sequence_Length, Features]
   ↑ 占据 batch_size 的位置，但代表「同空间下的多个实体」
```

### 语言模型

```
[Batch_Size, Sequence_Length, Embedding_Dim]
     ↑ 几篇独立文章
                  ↑ 一篇文章的词数（真正的上下文交互发生在这里）
```

> **关键洞察**：LLM 中真正的"上下文交互"发生在 `Sequence_Length` 维度上，而不是 `Batch_Size` 维度上。

---

## 三、归一化方式的根本分歧

| | 量化/CV | LLM |
|---|--------|-----|
| 偏好 | BatchNorm | LayerNorm / RMSNorm |
| 原因 | 跨样本统计去除大盘波动是合理的 | 跨句子统计会污染语义 |
| Batch=1 时的行为 | 崩溃 | 与 Batch=100 完全等价 |

详见 → [BN/LN/RMSNorm 详解](../AI/DeepLearning/Normalization.md)

---

## 四、为什么 LLM 还需要 Batch Size？

既然 LLM 内部 Batch 互不相干，为什么不一句一句训练？答：纯粹为了**工程优化**。

### 1. 硬件榨汁机（GPU 并行度）

- 单句子无法填满 GPU 上万个计算核心
- 多句子拼成 Batch 用矩阵乘法一起算，榨干显存带宽和算力
- 提升的是**吞吐量 (Throughput)**，不是模型能力

### 2. 梯度平滑

- 每次只看一句话 → 学习方向像无头苍蝇剧烈震荡
- Batch 内多句子求梯度均值 → 朝更稳定的方向收敛

---

## 五、核心结论

| 视角 | 量化截面 Batch | LLM Batch |
|------|---------------|-----------|
| 决定的是 | **模型的视野**（今天有哪些股票同台竞技） | **并行加速 + 梯度稳定** |
| 真正的视野在哪里 | Batch 维度（横截面） | Sequence Length（上下文） |
| 改变 batch_size | 改变模型可见范围 | 不改变单样本计算结果（仅影响速度） |

> **工程暗示**：如果在量化项目中借鉴 LLM 的分布式训练经验时，要特别警惕——LLM 的 batch 是无状态加速器，量化的 batch 是有状态视野窗。直接套用 gradient accumulation、bucket sampling 等技巧时，可能破坏截面分组的语义。
