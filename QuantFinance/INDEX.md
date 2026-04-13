# 量化金融 - 索引

## Basics/ — 基础概念
- [行情数据 L1 vs L2](Basics/MarketData_L1_L2.md) — Level-1/Level-2 数据的内容、频率与因子挖掘应用
- [组合优化五大维度](Basics/PortfolioOptimization.md) — 绝对收益、跟踪误差、流动性、滑点、尾部风险（含 VaR/CVaR）
- [泊松分布](Basics/PoissonDistribution.md) — 二项分布极限推导、泊松过程三前提、微分方程完整推导
- [正态分���与 t 分布](Basics/Normal_vs_T_Distribution.md) — PDF/CDF 公式、厚尾效应、t→正态的��限性质
- [正态投影定理](Basics/NormalProjectionTheorem.md) — 联合正态下的条件期望、信噪比增益系数、概率论性质汇总
- [LQ 动态规划](Basics/LQ_DynamicProgramming.md) — 贝尔曼方程、二次型传递、线性最优策略的数学来源
- [t 统计量](Basics/T_Statistic.md) — 信号/噪声比，因子显著性验证，与信息比率的关系
- [IC、IR 与夏普比率](Basics/IC_IR_Sharpe.md) — 因子评价三指标及主动管理基本定律
- [IC/IR 计算详解](Basics/IC_IR_Calculation.md) — Pearson vs Spearman、Rank IC 的鲁棒性、Python 实现
- [t 与 IR 的数学推导](Basics/T_IR_Derivation.md) — 从 Student-t 分布到 $t = IR \times \sqrt{T}$ 的严格推导
- [t 检验判断流程](Basics/T_Test_HypothesisTesting.md) — 显著性水平、p-value、单/双侧检验、多重检验校正
- [群体性数据挖掘与 t 值通胀](Basics/MultipleTestingProblem.md) — 抽屉效应、因子动物园、Harvey 的 t>3.0 标准
- [异常值检测与 MAD 去极值](Basics/OutlierDetection_MAD.md) — Z-Score/IQR/MAD/Isolation Forest 对比，工业级去极值流程
- [限价单](Basics/LimitOrders.md) — 限价买单/卖单、与市价单的区别、在 LOB 中的角色
- [Maker-Taker 模型与返佣](Basics/MakerTaker_Rebates.md) — 返佣条件、Post-Only 模式、各市场对比
- [复权](Basics/PriceAdjustment.md) — 不复权/前复权/后复权对比、未来函数陷阱、量化避坑指南
- [流动性](Basics/Liquidity.md) — 四大定量指标(ADV/Spread/Amihud/Impact)、流动性过滤器构建
- [Tick：跳价单位与逐笔数据](Basics/TickSize.md) — Tick Size（价格分辨率）vs Tick Data（时间心电图）
- [滑点](Basics/Slippage.md) — 流动性滑点/延迟滑点、平方根法则、策略中的四种应对方法
- [多元 OLS 回归](Basics/OLS_MultipleRegression.md) — 正规方程推导、高斯-马尔可夫假设、BLUE、几何投影解释
- [Ridge 与 LASSO](Basics/Ridge_LASSO.md) — L2/L1 正则化、闭式解 vs 迭代、菱形 vs 圆形的几何直觉
- [平方根冲击定律](Basics/SquareRootImpactLaw.md) — 公式推导、与Kyle模型对比、参与率阈值、真实指数δ∈[0.4,0.8]
- [订单簿与成交价格推算](Basics/OrderBook_Pricing.md) — 穿透订单簿、Micro-price、集合竞价、Kyle 模型与做市商博弈

## Microstructure/ — 市场微观结构 ★
- [模块总览](Microstructure/README.md) — 微观结构研究的全景图
- ⭐ [Summary 全模块总结](Microstructure/Summary.md) — 五维度全景、递进逻辑、实战路径
- [金融数据物理层次架构](Microstructure/DataArchitecture_Physics.md) — MBO→Tick→Snapshot→K线的"因果影"模型
- [交易方向推定 Lee-Ready / BVC](Microstructure/TradeDirection_LeeReady_BVC.md) — 报价规则、剔除测试、批量订单分类
- [Kyle (1985) 模型完整推导](Microstructure/KyleModel_Derivation.md) — 三方博弈、最优策略、均衡解、Kyle's Lambda
- [信息毒性 PIN / VPIN](Microstructure/PIN_VPIN.md) — 体积时钟革命与 2010 闪电崩盘预警
- [冰山订单与扫单流](Microstructure/OrderTypes_Iceberg_Sweep.md) — MBO 追踪、生存分析、流动性真空
- [Boehmer 散户订单识别](Microstructure/RetailOrders_Boehmer.md) — 子便士机制与 PFOF 微观签名
- [OFI 与 VOI](Microstructure/OFI_VOI.md) — 事件驱动 vs 静态存量、DeepLOB 多级架构

## Markets/ — 交易市场制度与规则
- [A 股做市商现状](Markets/MarketMaker_AShare.md) — 覆盖范围、主要券商、特殊性
- [中美做市商异同](Markets/MarketMaker_AvsUS.md) — 报价驱动 vs 订单驱动、PFOF、准入机制六大差异
- [中美量化策略异同](Markets/QuantStrategy_AvsUS.md) — T+1 vs T+0、指增/底仓T+0/量价因子 vs HFT/Long-Short/另类数据
- [A 股量化危机 2024](Markets/QuantCrisis_AShare_2024.md) — 因子拥挤→DMA 杠杆→流动性螺旋→监管重拳→行业转型

## Research/ — 文献调研与研究笔记
- [A 股高频因子工程](Research/AShare_HFFactorEngineering.md) — OFI 变体(磁吸效应/条件OFI)、大单主买主卖画像、T+1 下三种落地路径
- [OFI 因子数学构建方法](Research/OFI_Construction.md) — L1→MLOFI→PCA集成→非线性加权→霍克斯/OU→物理动能→神经网络

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
