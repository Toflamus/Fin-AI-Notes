# 量化因子数据处理工作流

从原始数据到可用因子的标准化流水线。本文档持续更新，记录实践中积累的经验与 tricks。

---

## 标准工作流

```
原始数据 → ① 清洗 → ② 去极值 → ③ 标准化 → ④ 缺失值处理 → ⑤ 中性化 → 干净因子
```

### Step 1: 数据清洗 (Data Cleaning)

去除明显无效的数据，保证输入质量。

- **过滤无效报价**：`ask_price > 0 AND bid_price > 0`
- **剔除停牌/涨跌停股票**：这些股票无法交易，纳入截面会扭曲因子分布
- **剔除上市不足 N 天的新股**：价格行为异常（通常 N = 60 或 120）
- **剔除 ST / *ST 股票**：基本面异常，不参与正常因子比较
- **时间段过滤**：去除集合竞价时段（9:15-9:25）、盘前盘后的异常数据

```python
# 典型过滤
df = df[df['is_suspended'] == False]
df = df[df['days_listed'] > 60]
df = df[~df['name'].str.contains('ST')]
```

### Step 2: 去极值 (Winsorization / Clipping)

将极端值压缩到合理范围，防止少数离群点主导模型。

**推荐方法：MAD 去极值**（详见 [异常值检测与 MAD](../Basics/OutlierDetection_MAD.md)）

```python
def winsorize_mad(x, n=5, max_outlier_ratio=0.03):
    median = np.median(x)
    mad = np.median(np.abs(x - median))
    
    if np.mean(np.abs(x - median) > n * mad) > max_outlier_ratio:
        lower, upper = np.percentile(x, [3, 97])
    else:
        lower = median - n * mad
        upper = median + n * mad
    
    return np.clip(x, lower, upper)
```

**其他方法：**
- 百分位 Winsorize：直接按 1%/99% 分位截断
- 3σ 法：`clip(x, μ-3σ, μ+3σ)`（对极值不够鲁棒，慎用）

### Step 3: 标准化 (Standardization / Normalization)

将因子值映射到可比较的尺度，消除量纲差异。

**Z-Score 标准化（截面）** — 最常用

$$x_{\text{std}} = \frac{x - \mu}{\sigma}$$

```python
# 截面标准化：每个时间截面内独立计算
df['factor_std'] = df.groupby('date')['factor'].transform(
    lambda x: (x - x.mean()) / x.std()
)
```

**Rank 标准化** — 更鲁棒

```python
# 将因子转为截面排名百分比 [0, 1]
df['factor_rank'] = df.groupby('date')['factor'].transform(
    lambda x: x.rank(pct=True)
)
```

> **Trick**: Rank 标准化天然去极值，不受离群点影响，与 Rank IC 天然配合。如果后续要算 Rank IC，可以直接在这一步完成排序。

### Step 4: 缺失值处理 (Missing Value Imputation)

- **行业均值填充**：用同行业同期的因子均值填充
- **截面中位数填充**：比均值更鲁棒
- **前值填充 (ffill)**：适用于低频因子（如财务数据），但要注意设上限防止过期数据
- **直接剔除**：缺失比例过高（>30%）的因子或个股直接丢弃

```python
# 行业均值填充
df['factor'] = df.groupby(['date', 'industry'])['factor'].transform(
    lambda x: x.fillna(x.median())
)
```

> **Trick**: 填充顺序很重要——先做去极值再填充，否则极端值会污染填充用的均值/中位数。

### Step 5: 中性化 (Neutralization)

剥离因子中不想要的系统性暴露（如市值效应、行业效应），提取纯 Alpha 信号。

**方法：截面回归取残差**

$$\text{Factor}_i = \beta_0 + \beta_1 \cdot \ln(\text{MarketCap}_i) + \sum_j \gamma_j \cdot \text{Industry}_{ij} + \epsilon_i$$

残差 $\epsilon_i$ 就是中性化后的因子。

```python
import statsmodels.api as sm

def neutralize(df_cross_section, factor_col, neutralize_cols):
    """截面中性化：对 market_cap 和 industry dummy 回归取残差"""
    y = df_cross_section[factor_col]
    X = sm.add_constant(df_cross_section[neutralize_cols])
    model = sm.OLS(y, X, missing='drop').fit()
    return model.resid

# 每个截面独立中性化
df['factor_neutral'] = df.groupby('date').apply(
    lambda g: neutralize(g, 'factor_std', ['ln_cap', 'industry_dummy'])
).reset_index(level=0, drop=True)
```

**常见中性化维度：**
- 市值中性（必选）：几乎所有因子都与市值相关
- 行业中性（必选）：消除行业间系统性差异
- 可选：波动率中性、流动性中性

---

## 常用数据处理 Tricks

### Trick 1: 处理顺序不能乱

```
去极值 → 标准化 → 填充 → 中性化
```

- 先去极值再标准化，否则极值拉偏均值和标准差
- 先填充再中性化，否则回归时丢失样本

### Trick 2: 截面操作 vs 时序操作

| 操作类型 | 含义 | 典型场景 |
|---------|------|---------|
| 截面 (Cross-sectional) | 同一时刻，跨所有股票 | 标准化、中性化、IC 计算 |
| 时序 (Time-series) | 同一股票，跨时间 | 动量计算、滚动波动率、ffill |

> **大坑**：截面操作一定要 `groupby('date')`，时序操作一定要 `groupby('instrument')`。混淆会导致未来函数或跨股票污染。

### Trick 3: 防止未来函数 (Look-ahead Bias)

- 财务数据用**报告发布日**而非报告期末日
- 时序滚动窗口用 `min_periods` 参数，避免初始期用到未来数据
- `resample` 时用 `closed='left', label='left'`

### Trick 4: Parquet 格式存储中间结果

每一步处理后的数据保存为 Parquet（详见 [Parquet 列式存储](../../Tools/DataFormats/Parquet.md)），方便复现和断点续跑：

```python
df.to_parquet(f'factor_{step_name}_{date}.parquet')
```

### Trick 5: 日志与数据质量监控

每一步处理后记录关键统计量，方便排查问题：

```python
def log_stats(df, col, step_name):
    print(f"[{step_name}] {col}: "
          f"mean={df[col].mean():.4f}, std={df[col].std():.4f}, "
          f"null%={df[col].isna().mean()*100:.1f}%, "
          f"min={df[col].min():.4f}, max={df[col].max():.4f}")
```
