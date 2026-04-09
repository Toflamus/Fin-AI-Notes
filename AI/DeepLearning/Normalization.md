# 归一化层：BatchNorm、LayerNorm、RMSNorm

深度学习中三种主流的归一化方法。它们的区别在于**在输入张量的哪几个维度上计算统计量**，以及计算公式的简化程度。

---

## 张量维度约定

假设输入是三维张量 $X$，形状 $B \times S \times F$：

| 下标 | 维度 | NLP 中的含义 |
|------|------|-------------|
| $b \in [1, B]$ | Batch | 第 $b$ 个句子 |
| $s \in [1, S]$ | Sequence | 句子中的第 $s$ 个词 |
| $f \in [1, F]$ | Feature | 词向量的第 $f$ 个特征 |

任意标量数据点：$x_{b, s, f}$

---

## 1. BatchNorm — 跨样本求统计量

### 核心思想

跨越所有 Batch，对**同一个特征（通道）**进行归一化。

### 严谨公式

为第 $f$ 个特征计算一个**全局均值** $\mu_f$ 和方差 $\sigma_f^2$，**坍缩 $B$ 和 $S$ 维度**：

$$\mu_f = \frac{1}{B \times S} \sum_{b=1}^{B} \sum_{s=1}^{S} x_{b, s, f}$$

$$\sigma_f^2 = \frac{1}{B \times S} \sum_{b=1}^{B} \sum_{s=1}^{S} (x_{b, s, f} - \mu_f)^2$$

$$y_{b, s, f} = \gamma_f \cdot \frac{x_{b, s, f} - \mu_f}{\sqrt{\sigma_f^2 + \epsilon}} + \beta_f$$

> **大白话**：把所有句子里、所有词的"第 $f$ 个特征值"全部抽出来求均值和方差。统计量 $\mu, \sigma$ 的形状为 $[1, 1, F]$。

### 优缺点

- ✓ 极大缓解内部协变量偏移 (Internal Covariate Shift)，CNN 中表现极好
- ✗ **极度依赖 Batch Size**——Batch 太小时统计量剧烈抖动
- ✗ 不适合变长序列：不同句子长度不同，长序列位置可能全是 Padding

---

## 2. LayerNorm — 单样本内跨特征

### 核心思想

不跨样本，**只在单个样本的单个时间步内**对所有特征维度进行归一化。

### 严谨公式

为第 $b$ 个句子的第 $s$ 个词计算专属的 $\mu_{b,s}$ 和 $\sigma_{b,s}^2$，**仅坍缩 $F$ 维度**：

$$\mu_{b, s} = \frac{1}{F} \sum_{f=1}^{F} x_{b, s, f}$$

$$\sigma_{b, s}^2 = \frac{1}{F} \sum_{f=1}^{F} (x_{b, s, f} - \mu_{b, s})^2$$

$$y_{b, s, f} = \gamma_f \cdot \frac{x_{b, s, f} - \mu_{b, s}}{\sqrt{\sigma_{b, s}^2 + \epsilon}} + \beta_f$$

> **大白话**：死死盯住第 $b$ 句话的第 $s$ 个词（比如"我"字），把它的 $F$ 个特征值加起来求平均。完全不关心其他词，也不关心其他句子。统计量形状为 $[B, S, 1]$。

### 优缺点

- ✓ 完全不受 Batch Size 影响
- ✓ 非常适合 Transformer 等变长序列模型
- ✗ CV 任务中效果不如 BatchNorm（忽略了全局样本统计规律）

---

## 3. RMSNorm — LayerNorm 的极简版

### 核心思想

研究者发现 LayerNorm 的成功主要在于**缩放 (Scaling)**，而非**平移 (Mean Centering)**。RMSNorm 直接去掉了计算均值的步骤。

### 严谨公式

仅坍缩 $F$ 维度，但**不减均值**：

$$\text{RMS}_{b, s} = \sqrt{\frac{1}{F} \sum_{f=1}^{F} x_{b, s, f}^2 + \epsilon}$$

$$y_{b, s, f} = \gamma_f \cdot \frac{x_{b, s, f}}{\text{RMS}_{b, s}}$$

> **大白话**：把特征向量里的每个值平方后求平均，再开根号。用这个标量值缩放该词的所有特征。**没有 $\beta$ 偏置**。

### 优缺点

- ✓ **速度快**：省去均值计算，计算量减少 10%~30%
- ✓ **效果好**：在大语言模型中表现与 LayerNorm 几乎一致，甚至更稳定
- ✓ 被现代 LLM 广泛采用：LLaMA、Gemma、Mistral 等

---

## 4. 三者对比总结

| 特性 | BatchNorm | LayerNorm | RMSNorm |
|------|-----------|-----------|---------|
| 坍缩维度 (PyTorch `dim`) | `(0, 1)` 即 $B, S$ | `(2)` 即 $F$ | `(2)` 即 $F$ |
| 统计量形状 | $[1, 1, F]$ | $[B, S, 1]$ | $[B, S, 1]$ |
| 是否依赖 Batch Size | **是**（小 Batch 崩溃） | 否 | 否 |
| 是否计算均值（中心化） | 是 | 是 | **否** |
| 是否有偏置 $\beta$ | 是 | 是 | 通常无 |
| 主流应用 | 计算机视觉 (CNN) | NLP/Transformer (BERT, GPT-2) | 现代大模型 (LLaMA, Gemma) |

### 物理意义对照

| | 物理意义 |
|---|---------|
| BatchNorm | "每种特征"的全局统计算法 |
| LayerNorm | "每个词自身"的内部特征分布 |
| RMSNorm | "每个词自身"的内部能量大小 |

> **演化逻辑**：BatchNorm（CV 时代）→ LayerNorm（解决变长序列与 Batch 依赖）→ RMSNorm（去掉冗余的中心化，追求极致效率）。每一步都是对前者的精简优化。
