# 全局索引

## 量化金融 (Quantitative Finance)

> 详见 [QuantFinance/INDEX.md](QuantFinance/INDEX.md)

- [行情数据 L1 vs L2](QuantFinance/Basics/MarketData_L1_L2.md) — 数据层级、内容差异与因子挖掘应用
- [组合优化五大维度](QuantFinance/Basics/PortfolioOptimization.md) — 绝对收益、跟踪误差、流动性、滑点、尾部风险
- [t 统计量](QuantFinance/Basics/T_Statistic.md) — 信号/噪声比，因子验证与夏普比率的联系
- [IC、IR 与夏普比率](QuantFinance/Basics/IC_IR_Sharpe.md) — 因子评价三指标及主动管理基本定律
- [IC/IR 计算详解](QuantFinance/Basics/IC_IR_Calculation.md) — Pearson vs Spearman、Rank IC 鲁棒性与代码实现
- [t 与 IR 的数学推导](QuantFinance/Basics/T_IR_Derivation.md) — Student-t 分布推导及 $t = IR \times \sqrt{T}$ 的金融含义
- [t 检验判断流程](QuantFinance/Basics/T_Test_HypothesisTesting.md) — 显著性水平、p-value、单/双侧检验、多重检验校正
- [群体性数据挖掘与 t 值通胀](QuantFinance/Basics/MultipleTestingProblem.md) — 抽屉效应、因子动物园、为什么 t>3.0
- [异常值检测与 MAD 去极值](QuantFinance/Basics/OutlierDetection_MAD.md) — MAD 鲁棒性、四种检测方法对比、去极值代码
- [订单簿与成交价格推算](QuantFinance/Basics/OrderBook_Pricing.md) — 穿透订单簿、Micro-price、Kyle 模型、做市商博弈
- [指数增强策略](QuantFinance/Strategies/IndexEnhancement.md) — Alpha来源、风险约束、适用环境
- [高频因子挖掘](QuantFinance/Strategies/HighFreqFactorMining.md) — Tick→K线聚合→预测的完整范式

## 人工智能 (Artificial Intelligence)

> 详见 [AI/INDEX.md](AI/INDEX.md)

- [归一化层 BN/LN/RMSNorm](AI/DeepLearning/Normalization.md) — 张量维度、公式与适用场景

## 交叉领域 (AI x Quant)

> 详见 [Cross/INDEX.md](Cross/INDEX.md)

- [Batch Size：量化 vs LLM](Cross/Batch_Quant_vs_LLM.md) — 截面交互 vs 独立加速的根本差异

## 工具与编程 (Tools & Programming)

- [SQL 教程](Tools/SQL/Tutorial.md) — 从基础语法到窗口函数、性能优化，含 ClickHouse 语法与 OFI 因子实战案例
- [Parquet 列式存储](Tools/DataFormats/Parquet.md) — 列式 vs 行式、压缩优势、Python 用法
