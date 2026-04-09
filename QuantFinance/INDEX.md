# 量化金融 - 索引

## Basics/ — 基础概念
- [行情数据 L1 vs L2](Basics/MarketData_L1_L2.md) — Level-1/Level-2 数据的内容、频率与因子挖掘应用
- [组合优化五大维度](Basics/PortfolioOptimization.md) — 绝对收益、跟踪误差、流动性、滑点、尾部风险（含 VaR/CVaR）
- [t 统计量](Basics/T_Statistic.md) — 信号/噪声比，因子显著性验证，与信息比率的关系
- [IC、IR 与夏普比率](Basics/IC_IR_Sharpe.md) — 因子评价三指标及主动管理基本定律
- [IC/IR 计算详解](Basics/IC_IR_Calculation.md) — Pearson vs Spearman、Rank IC 的鲁棒性、Python 实现
- [t 与 IR 的数学推导](Basics/T_IR_Derivation.md) — 从 Student-t 分布到 $t = IR \times \sqrt{T}$ 的严格推导
- [t 检验判断流程](Basics/T_Test_HypothesisTesting.md) — 显著性水平、p-value、单/双侧检验、多重检验校正
- [群体性数据挖掘与 t 值通胀](Basics/MultipleTestingProblem.md) — 抽屉效应、因子动物园、Harvey 的 t>3.0 标准
- [异常值检测与 MAD 去极值](Basics/OutlierDetection_MAD.md) — Z-Score/IQR/MAD/Isolation Forest 对比，工业级去极值流程
- [订单簿与成交价格推算](Basics/OrderBook_Pricing.md) — 穿透订单簿、Micro-price、集合竞价、Kyle 模型与做市商博弈

## DataProcessing/ — 数据处理工作流（经验总结）
- [标准工作流](DataProcessing/Workflow.md) — 清洗→去极值→标准化→缺失值→中性化，附常用 Tricks
- [特征净化与降维](DataProcessing/FeatureOptimization.md) — NaN填充→ZCA正交化→PCA降维，全局vs截面，模型适配
- [协方差/PCA/ZCA 数学原理](DataProcessing/Covariance_PCA_ZCA_Math.md) — 从惯性张量到坐标变换，PCA vs ZCA 的本质区别
- [量化定制 DataLoader](DataProcessing/QuantDataLoader.md) — 截面分组、按日批次、与原生 DataLoader 的区别

## Models/ — 定价与风险模型
*暂无内容*

## Strategies/ — 交易策略
- [指数增强策略](Strategies/IndexEnhancement.md) — 带约束优化视角下的 Alpha 来源、风险模型与适用环境
- [高频因子挖掘](Strategies/HighFreqFactorMining.md) — Tick→变换→K线聚合→预测的完整范式，维度计算与维度匹配

## Tools/ — 工具与编程
*暂无内容*
