# Apache Parquet — 列式存储格式

Parquet 是开源的、面向列（Columnar）的数据文件格式，广泛应用于大规模数据处理与分析。在量化金融的高频数据和因子计算场景中几乎是标准选择。

---

## 核心优势

### 1. 极高的查询效率（I/O 优化）

列式存储只读取需要的列，跳过无关数据。

- 100 列的因子数据中，只需"时间戳"、"价格"和 2 个因子列
- CSV：必须扫描整个文件
- Parquet：精准读取 4 列，大幅减少磁盘 I/O 和内存占用

### 2. 卓越的压缩率

同一列数据类型完全相同（全是浮点数或全是字符串），可针对性采用高效编码和压缩算法（Snappy / Gzip）。

- Parquet 文件体积通常只有同等 CSV 的 **1/10 甚至更小**

### 3. 自带数据结构 (Schema-aware)

文件内部包含元数据（Metadata），明确记录每列的名称和严格数据类型。

- 不需要像 CSV 那样每次读取时推断类型
- 保证数据一致性，避免类型错误

---

## 对比：Parquet vs CSV

| 特性 | Parquet (列式) | CSV (行式) |
|------|---------------|------------|
| 读取速度（特定列） | 极快（仅读目标列） | 慢（必须扫描每行完整文本） |
| 文件体积 | 非常小（二进制 + 高压缩率） | 大（纯文本冗余存储） |
| 数据类型保留 | 严格保留（强类型，读取即用） | 容易丢失，每次需重新推断 |
| 人类可读性 | 不可读（二进制，需代码读取） | 强（记事本/Excel 可直接打开） |
| 多重索引 | 完美保留 MultiIndex | 不支持 |

---

## Python 使用

```python
import pandas as pd

# 写入 Parquet（默认 Snappy 压缩）
df.to_parquet('data.parquet')

# 读取 Parquet（比 read_csv 快数倍）
df = pd.read_parquet('data.parquet')

# 只读取特定列（列式存储的核心优势）
df = pd.read_parquet('data.parquet', columns=['timestamp', 'price', 'ofi_1'])

# 指定压缩算法
df.to_parquet('data.parquet', compression='gzip')  # 更高压缩率
df.to_parquet('data.parquet', compression='snappy') # 更快速度（默认）
```

---

## 适用场景

- **量化因子数据**：列多行多，频繁按列读取
- **高频行情数据**：海量 Tick/Snapshot 存储与快速时间切片
- **ML 特征矩阵**：训练 XGBoost 等模型时只取部分特征列
- **分布式计算**：Apache Spark、Hadoop 的首选底层格式

> **经验法则**：数据量开始变大、需要频繁按列读取/过滤时，就该从 CSV 切换到 Parquet。
