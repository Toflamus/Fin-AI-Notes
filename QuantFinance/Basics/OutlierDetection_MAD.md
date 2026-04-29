# 异常值检测与 MAD 去极值

因子预处理中的核心步骤：如何判断异常值，以及为什么 MAD 是量化金融中最常用的去极值方法。

---

## 1. MAD (Median Absolute Deviation) — 中位数绝对偏差

**MAD** 全称 **Median Absolute Deviation**。在统计学和数据处理中，它是衡量一组数值**离散程度 (Dispersion)** 的一个极其**鲁棒 (Robust)** 的指标。相比于常用的标准差 (Standard Deviation)，MAD 在面对包含极端离群点 (Outliers) 的数据时表现要稳定得多。

---

### 1.1 数学定义

对于一组观测值 $X = \{x_1, x_2, \dots, x_n\}$，其 MAD 定义为：

$$\boxed{\text{MAD} = \text{median}_i\left(\, |x_i - \tilde{x}| \,\right)}$$

其中：
- $\tilde{x} = \text{median}(X)$：这组数据的**中位数**
- $|x_i - \tilde{x}|$：每个观测点到中位数的**绝对距离**
- 最后对这些绝对距离再取**一次中位数**

### 1.2 计算步骤（逐步展开）

**Step 1** 求中位数：
$$\tilde{x} = \text{median}(\{x_1, x_2, \dots, x_n\})$$

**Step 2** 求每个点到中位数的绝对偏差：
$$d_i = |x_i - \tilde{x}|, \quad i = 1, 2, \dots, n$$

**Step 3** 对绝对偏差序列再取中位数：
$$\text{MAD} = \text{median}(\{d_1, d_2, \dots, d_n\})$$

---

### 1.3 为什么用 MAD 而不是标准差？——从理论到实战

#### 1.3.1 标准差的数学痛点

标准差定义：
$$\sigma = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (x_i - \bar{x})^2}$$

**平方和**是罪魁祸首：
- 如果数据里有一个观测点 $x_k$ 远离均值 $\bar{x}$，那么 $(x_k - \bar{x})^2$ 会以**平方级**放大它的影响
- 加入一个马斯克级的异常值，$\sigma$ 被**一次性拉飞**
- 更糟的是：被异常值膨胀的 $\sigma$ 反过来让**原本应该被标记为异常**的中等异常点被"**洗白**"——这就是**掩蔽效应 (Masking Effect)**

#### 1.3.2 MAD 的鲁棒性：击穿点理论 (Breakdown Point)

**击穿点 (Breakdown Point)** 是鲁棒统计的核心指标，定义为：**一个统计量在失去任何可解释性之前所能容忍的最大污染比例**。

| 统计量 | 击穿点 | 含义 |
|--------|-------|------|
| 均值 $\bar{x}$ | **0** | 哪怕只有 1 个点 $\to \infty$，均值就跟着 $\to \infty$ |
| 标准差 $\sigma$ | **0** | 同上 |
| 中位数 $\tilde{x}$ | **50%** | 数据污染不超过一半时，中位数仍有意义 |
| **MAD** | **50%** | 同上，继承自中位数的鲁棒性 |

**数学直觉**：
- 中位数是**排序统计量** (Order Statistic)
- 当少于 50% 的数据被污染时，中位数所在的秩位置**仍在未被污染的数据集中**
- MAD 是"对绝对偏差取中位数"，击穿点同样是 50%

**结论**：只要异常值比例 $<$ 50%，MAD 就能始终锁定在**代表主体分布的"平庸"数据**上。

#### 1.3.3 对比速查表

| 维度 | 标准差 (Std) | MAD |
|------|-------------|-----|
| 依赖的中心统计量 | 均值 (Mean) | 中位数 (Median) |
| 对极值 | **极度敏感**（平方级放大） | **几乎不受影响** |
| 击穿点 | **0%** | **50%** |
| 掩蔽效应 | **严重**（极值拉宽 $\sigma$，中等异常被洗白） | **不存在** |
| 分布假设 | 隐含正态假设 | 无 |

> **类比**：一群人月薪 1 万，混进一个马斯克——均值和标准差被瞬间拉飞，但中位数和 MAD 几乎不变。

---

### 1.4 1.4826 尺度系数的严格推导

在实际应用中，为了让 MAD 能像标准差一样在正态分布下进行可比的衡量，我们会乘以一个系数（约 $1.4826$），得到**鲁棒标准差估计**：

$$\boxed{\hat{\sigma}_{\text{robust}} \approx 1.4826 \cdot \text{MAD}}$$

这个 $1.4826$ 不是拍脑袋的经验值，而是有严格的数学来源。下面给出完整推导。

#### Step 1：假设数据服从标准正态 $X \sim \mathcal{N}(0, \sigma^2)$

此时中位数 $\tilde{x} = 0$（由对称性），于是：
$$|X_i - \tilde{x}| = |X_i|$$

$$\text{MAD} = \text{median}(|X|)$$

#### Step 2：求 $|X|$ 的中位数

$|X|$ 的 CDF 为：
$$P(|X| \leq t) = P(-t \leq X \leq t) = \Phi(t/\sigma) - \Phi(-t/\sigma) = 2\Phi(t/\sigma) - 1$$

其中 $\Phi$ 是标准正态 CDF。

令此概率等于 $1/2$（找中位数）：
$$2\Phi(t/\sigma) - 1 = \frac{1}{2}$$

$$\Phi(t/\sigma) = \frac{3}{4}$$

#### Step 3：查表 / 推导 $\Phi^{-1}(3/4)$

$$t/\sigma = \Phi^{-1}(3/4) \approx 0.6745$$

这是标准正态的**上四分位数**（75% 分位点）。

因此：
$$\text{MAD} = t \approx 0.6745 \cdot \sigma$$

#### Step 4：反解得到尺度系数

$$\sigma = \frac{\text{MAD}}{0.6745} = \frac{1}{\Phi^{-1}(3/4)} \cdot \text{MAD}$$

**计算 $1/0.6745$**：

$$\frac{1}{0.6745} \approx 1.4826$$

更精确地：
$$\frac{1}{\Phi^{-1}(0.75)} = \frac{1}{0.67448975019608171...} = 1.48260221850560186...$$

所以：

$$\boxed{\hat{\sigma}_{\text{robust}} = 1.4826 \cdot \text{MAD}}$$

#### Step 5：一致性 (Consistency) 的含义

这个系数被称为**一致性系数 (Consistency Constant)**。含义是：

> **当数据真实服从正态分布时**，$1.4826 \cdot \text{MAD}$ 是 $\sigma$ 的一个**一致估计量 (Consistent Estimator)**——样本量 $n \to \infty$ 时它**依概率收敛**到真实的 $\sigma$。

换言之：在正态情况下，$1.4826 \cdot \text{MAD}$ 和样本标准差估计的是**同一个 $\sigma$**，但前者**对异常值免疫**。

这就是为什么在量化金融中看到的几乎所有"鲁棒 Z-Score"都用这个形式：

$$Z_{\text{robust}}(x_i) = \frac{x_i - \tilde{x}}{1.4826 \cdot \text{MAD}}$$

---

### 1.5 直观理解

- **均值 + 标准差**：衡量的是"**集体平均水平**"，容易被极少数的巨富（机构单）带偏
- **中位数 + MAD**：衡量的是"**典型普通人水平**"，无论数据里有多少个极端的尖峰，它依然只看那群**最占多数的底噪**

**对当前研究场景（幂律底噪 + 机构尖峰）**：

在 LOB 数据、订单流、撤单行为这类**"幂律底噪 + 机构尖峰"**的场景中：

- **散户底噪**：遵循幂律分布，量大但每个都很小
- **机构尖峰**：数量少但极端（14:56:59 大规模撤单、大单吃货）
- **标准差**：会被少数机构尖峰拉飞 → 对散户噪声水平的估计**严重失真**
- **MAD**：只要异常（尖峰）不超过 50%，就始终锁定在代表**散户底噪**的"平庸"数据上

→ **MAD 是在满是噪音和干扰的 LOB 数据中，寻找"真实底噪水平"的一把精密直尺**。

在 [EntropyFactors_Summary](../Research/AShare_HFFactorEngineering.md)、[ClusterValidation_NoLabel](../../Research/OrderAttribution_AShare_2026Q2/notes/ClusterValidation_NoLabel.md) 的验证流程中，所有需要估计"**背景水平**"的地方都应使用 $1.4826 \cdot \text{MAD}$ 而不是 $\sigma$。

---

### 1.6 工业级去极值流程

```python
import numpy as np

def robust_winsorize(x: np.ndarray, n_mad: float = 5.0,
                     outlier_threshold: float = 0.03) -> np.ndarray:
    """
    MAD 鲁棒去极值：
    - 若异常比例 < threshold：按 n_mad * MAD 剪裁
    - 若异常比例超标：退化到分位数兜底
    """
    median = np.median(x)
    mad = np.median(np.abs(x - median))
    robust_sigma = 1.4826 * mad  # ← 一致性尺度系数

    # 判断异常值比例（以 n_mad * MAD 为阈值）
    outlier_ratio = np.mean(np.abs(x - median) > n_mad * mad)

    if outlier_ratio > outlier_threshold:
        # 异常值太多（>3%），说明分布远离正态，用分位数兜底
        lower, upper = np.percentile(x, [3, 97])
    else:
        # 正常情况，用 n × MAD 作为边界
        lower = median - n_mad * mad
        upper = median + n_mad * mad

    return np.clip(x, lower, upper)
```

**参数选择经验**：
- $n = 3$：非常严格（类比正态下的 $\pm 2\sigma$ 左右）
- $n = 5$：量化因子预处理常用（类比 $\pm 3.4\sigma$，覆盖 > 99.9%）
- $n = 10$：极宽容，仅剔除真正的灾难点

---

## 2. 异常值检测的四种方法

### 2.1 Z-Score（标准分法）— 基于正态假设

$$Z = \frac{x - \mu}{\sigma}$$

- 判定标准：$|Z| > 3$ 为异常值（偏离均值 3 个标准差，占比 < 0.3%）
- **局限**：均值和标准差本身就被极值污染 → 掩蔽效应严重

### 2.2 IQR（四分位距法 / 箱线图法）— 非参数

1. 计算 $Q_1$（25 分位）和 $Q_3$（75 分位）
2. $\text{IQR} = Q_3 - Q_1$
3. 正常范围：$[Q_1 - 1.5 \cdot \text{IQR}, \; Q_3 + 1.5 \cdot \text{IQR}]$

- 不依赖分布假设，适用于偏态数据
- 箱线图（Boxplot）就是基于此方法

### 2.3 MAD（绝对中位差法）— 鲁棒统计 ★

- 正常范围：$[\text{Median} - n \cdot \text{MAD}, \; \text{Median} + n \cdot \text{MAD}]$
- 经验取值：$n = 3 \sim 5$（量化中常用 5）
- **量化因子预处理的首选方法**

### 2.4 机器学习方法 — 多维数据

当单一维度判定失效（如需联合考察流动性、波动率、动量的异常情况）：

| 方法 | 原理 | 适用 |
|------|------|------|
| 孤立森林 (Isolation Forest) | 随机切分空间，切分次数少的点 = 边缘点 = 异常 | 通用多维 |
| DBSCAN | 低密度区域的非核心点 = 噪声/异常 | 有聚簇结构的数据 |

---

## 3. 方法对比

| 方法 | 分布假设 | 抗极值能力 | 维度 | 复杂度 |
|------|---------|-----------|------|--------|
| Z-Score | 正态 | 弱（掩蔽效应） | 单维 | 低 |
| IQR | 无 | 中等 | 单维 | 低 |
| **MAD** | **无** | **强** | **单维** | **低** |
| Isolation Forest | 无 | 强 | 多维 | 中 |
| DBSCAN | 无 | 强 | 多维 | 中 |

> **实践建议**：因子截面去极值优先用 MAD；多维联合异常检测用 Isolation Forest。
