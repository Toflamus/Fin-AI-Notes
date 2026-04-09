# IC 与 IR 的计算方法：从公式到代码

本文详细拆解 IC（信息系数）和 IR（信息比率）的底层计算逻辑，包括皮尔森与斯皮尔曼两种相关系数的区别，以及为什么金融中几乎总是使用 Rank IC。

> 前置知识：[IC、IR 与夏普比率](IC_IR_Sharpe.md)

---

## 1. IC 的计算流程

### 单期 IC（截面因子测试）

在 $t$ 时刻，假设你有 $N$ 个标的（股票、期货合约等）：

- 因子值向量：$X_t = [x_1, x_2, \ldots, x_N]$（如每个标的的 OFI 值）
- 下一期实际收益率：$Y_{t+1} = [y_1, y_2, \ldots, y_N]$

该期的 IC 就是 $X_t$ 与 $Y_{t+1}$ 之间的相关系数：

$$IC_t = \text{Correlation}(X_t, Y_{t+1})$$

> 注意是**截面相关**：同一时刻，不同标的之间的因子值 vs 收益率。

### 多期 IR

对一段时间（如 1000 个高频窗口）逐期计算 IC，得到时间序列 $[IC_1, IC_2, \ldots, IC_T]$，然后：

$$IR = \frac{\mu_{IC}}{\sigma_{IC}}$$

> 年化 IR（用于报告）：$IR_{\text{annual}} = IR \times \sqrt{\text{Periods per year}}$。高频内部比对时，直接用原始序列计算即可。

---

## 2. 皮尔森相关系数 (Pearson Correlation) — Normal IC

衡量两个变量之间的**线性相关程度**。假设数据服从正态分布。

### 公式

$$r = \frac{\sum_{i=1}^{n}(X_i - \bar{X})(Y_i - \bar{Y})}{\sqrt{\sum_{i=1}^{n}(X_i - \bar{X})^2 \cdot \sum_{i=1}^{n}(Y_i - \bar{Y})^2}}$$

- 分子：$X$ 和 $Y$ 的协方差（衡量共同变动方向）
- 分母：$X$ 和 $Y$ 各自标准差的乘积（归一化）
- 范围：$[-1, +1]$

### 弱点

- 对**异常值极其敏感**：一个极端数据点可以严重扭曲结果
- 只能捕捉**线性**关系

---

## 3. 斯皮尔曼相关系数 (Spearman Rank Correlation) — Rank IC

衡量两个变量之间的**单调相关程度**（不要求线性）。核心思想：**不用原始值，而用数据在样本中的排序（Rank）**。

### 计算流程

1. 将 $X$ 的所有元素按大小排序，得到位置序号 $R(X)$
2. 将 $Y$ 的所有元素按大小排序，得到位置序号 $R(Y)$
3. 计算排序差值：$d_i = R(X_i) - R(Y_i)$
4. 代入公式（无大量并列排名时）：

$$\rho = 1 - \frac{6 \sum_{i=1}^{n} d_i^2}{n(n^2 - 1)}$$

### 计算示例

假设 5 个标的：

| 标的 | OFI 原值 | 收益率原值 | OFI Rank | 收益率 Rank | $d_i$ | $d_i^2$ |
|------|----------|-----------|----------|-------------|-------|---------|
| A | 120 | 0.8% | 4 | 4 | 0 | 0 |
| B | 50 | 0.2% | 2 | 2 | 0 | 0 |
| C | 200 | 1.5% | 5 | 5 | 0 | 0 |
| D | 80 | 0.5% | 3 | 3 | 0 | 0 |
| E | 10 | 0.1% | 1 | 1 | 0 | 0 |

$$\rho = 1 - \frac{6 \times 0}{5 \times (25 - 1)} = 1.0 \quad \text{(完美单调关系)}$$

---

## 4. 为什么金融中几乎都用 Rank IC？

金融数据存在严重的"肥尾"现象。例如某时刻因为一笔巨大市价单，OFI 爆表产生极端异常值：

| 对比维度 | 皮尔森 (Normal IC) | 斯皮尔曼 (Rank IC) |
|----------|-------------------|-------------------|
| 对离群值 | **极度敏感**，一个极值可摧毁当期 IC | **鲁棒**，无论离群值多大，Rank 最多是第 1 名或倒数第 1 名 |
| 捕捉关系 | 仅线性关系 | 任何单调关系（含非线性） |
| 分布假设 | 要求近似正态 | 无分布假设 |
| 金融适用性 | 容易被高频噪声扭曲 | **业界标准选择** |

> **核心直觉**：Rank 操作相当于一次自动的"去极值"处理。不管异常值是 100 还是 100000，在排序后都只是"第 1 名"，影响被严格限制住了。

---

## 5. Python (Pandas) 实现

### 单期 IC 计算

```python
import pandas as pd

# df 包含 N 个标的在 t 时刻的因子值和 t+1 时刻的收益率
# 列: 'OFI_factor', 'Next_Return'

# Normal IC (Pearson)
normal_ic = df['OFI_factor'].corr(df['Next_Return'], method='pearson')

# Rank IC (Spearman) — 业界标准
rank_ic = df['OFI_factor'].corr(df['Next_Return'], method='spearman')
```

### 多期 IC 序列与 IR

```python
# 假设 panel_df 包含列: 'date', 'instrument', 'OFI_factor', 'Next_Return'
# 每个 date 是一个截面

# 逐期计算 Rank IC
ic_series = panel_df.groupby('date').apply(
    lambda g: g['OFI_factor'].corr(g['Next_Return'], method='spearman')
)

# IR = IC均值 / IC标准差
ir = ic_series.mean() / ic_series.std()

# 年化 IR（假设日频，252 个交易日）
ir_annual = ir * (252 ** 0.5)

# 可视化 IC 时间序列
ic_series.plot(title='Rank IC Time Series')
```

### IC 的统计检验

```python
import numpy as np
from scipy import stats

# t 检验：IC 均值是否显著不为 0
t_stat = ic_series.mean() / (ic_series.std() / np.sqrt(len(ic_series)))
p_value = 2 * (1 - stats.t.cdf(abs(t_stat), df=len(ic_series)-1))

print(f"IC Mean: {ic_series.mean():.4f}")
print(f"IC Std:  {ic_series.std():.4f}")
print(f"IR:      {ir:.4f}")
print(f"t-stat:  {t_stat:.2f}, p-value: {p_value:.4f}")
```

> 注意 $t \approx IR \times \sqrt{N}$，与 [t 统计量](T_Statistic.md) 中的公式一致。
