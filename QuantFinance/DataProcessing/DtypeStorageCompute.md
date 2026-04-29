# 量化数据 dtype 选型：Store in 32, Compute in 64

> 一条经验法则 + 必须留 64 位的例外清单。
> 起源：2026-04-27 institutional_ofi 项目在 8GB 服务器上跑全市场，因 dtype 默认 float64 导致内存压力。

## 核心原则

**存储 float32 / int8、计算 float64**（GPU 深度学习圈的标准做法，量化也适用）。

理由：
- 截面因子值、风格暴露、单期收益率都是**有界小量**（量级 ~1e-2 ~ 1e0），float32 的 7 位有效数字足够
- 行业哑变量、bool flag 是 0/1，int8 (1 字节) 足够
- 需要高精度的**计算环节**（np.linalg.lstsq、np.cov 等）默认升回 float64，所以"存 32 算 64"不损失精度
- 内存占用减半到八分之一，缓存 / IO 也快

---

## float64 必须用的场景

| 场景 | 为什么 | 例子 |
|---|---|---|
| **累乘链** | float32 单步误差 ~1e-7，连乘 252 步后 NAV 偏离明显 | `(1+ret).cumprod()`、净值曲线、年化回报 |
| **大量 PnL 累加** | catastrophic cancellation：百万笔正负相加 | tick PnL 累加、日内全市场成交额 |
| **小数除小数 t-stat** | 信号 ~1e-3，std ~1e-2，比值精度敏感 | Sharpe、IC t-stat、ICIR |
| **协方差矩阵求逆** | 条件数容易 ~1e10+，float32 直接奇异 | Markowitz 优化、风险平价、Barra 模型自身估计 |
| **Newton 迭代** | 收敛阈值 ~1e-8 | 期权隐含波动率求解 |
| **大量级计数（按厘）** | 全市场日成交额 ~10^14 厘，超 float32 尾数精度 | 全市场流量、成交额累计 |

---

## int64 必须用的场景

| 场景 | 为什么 | 例子 |
|---|---|---|
| **纳秒时间戳** | int32 只能存 ~24 天的纳秒 | 微观结构 event_time |
| **全局唯一 order_id** | 交易所一日订单数破亿 | A 股一日 ~5 亿条订单 |
| **大额累计计数** | A 股全年单股成交量超 int32 上限 (~21 亿股) | 单股年累计成交量 |
| **bitmask 多标签** | 标签数 > 8 时需要 int16/64 | CATEGORY_OFI_PLAN 的 `bucket_mask` 6 桶用 int8 够；扩到 30+ 标签要升级 |

---

## 哪些可以放心降级

| 数据 | 安全 dtype | 备注 |
|---|---|---|
| 已截面归一化的因子值（factor_zoo 的 `f_*`） | float32 | 量级 ~1e-2 |
| 单期收益（日 ret_o2o ~ -0.1 ~ +0.1） | float32 | 不做 cumprod 时安全 |
| Barra 风格暴露（zscore ~ -3 ~ +3） | float32 | Barra 自己发布精度 `decimal(6,3)` |
| 行业哑变量 (0/1) | int8 | 1 字节够 |
| 价格 / OHLC（4 位小数 A 股股价） | float32 | 5-6 位有效数字 |
| 单档挂单量 / 成交量 | int32 | 单笔不会超 21 亿 |

---

## 反模式（不要做）

- ❌ 把 NAV 净值序列存 float32 落 parquet —— 后续做最大回撤、年化时误差累积
- ❌ 把纳秒时间戳存 int32 —— 直接溢出
- ❌ 在 lstsq / inv 之前把 X 强制 float32 —— 病态矩阵直接奇异
- ❌ 为了 "省一点" 把 order_id 存 int32 —— 一天就溢出

---

## 实操速查

```python
# 因子表落盘前
df[factor_cols] = df[factor_cols].astype('float32')
df[industry_cols] = df[industry_cols].astype('int8')
df.to_parquet(path)

# 计算时不强转，让 numpy/scipy 默认升 float64
y = df[factor].to_numpy()  # 默认 float64
X = df[exog].to_numpy()    # 默认 float64
coef, *_ = np.linalg.lstsq(X, y, rcond=None)
```

参考：`Ideas/institutional_ofi/notebooks/03_factor_evaluation.ipynb` 是这条原则的落地示例。
