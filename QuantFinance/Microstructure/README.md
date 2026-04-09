# 市场微观结构 (Market Microstructure)

订单流微观分类、限价订单簿动力学与高频 Alpha 因子的系统性研究。

---

## 范式起点

在现代金融市场的微观结构理论与高频交易 (High-Frequency Trading, HFT) 领域，市场不再被视为一个抽象的宏观定价机制，而是被解构为**由海量异质性订单构成的复杂动力学系统**。

随着电子化交易与撮合引擎技术的极速演进，量化研究人员和交易员获取到的底层数据已经从传统的日线、分钟线，下沉至 **Level-2 快照**乃至 **Tick 级和逐笔委托 (Market-by-Order, MBO)** 数据。这些包含了**纳秒级时间戳**、精细价格梯度和微观容量的底层日志，构成了金融市场的"基本粒子"。

探讨订单流 (Order Flow) 的微观分类，本质上是在运用**显微镜穿透价格变动的表象**，揭示三件事情：

1. **微观市场参与者的异质性动机** — 谁在交易？为什么交易？
2. **信息不对称程度的动态演变** — 谁掌握着信息？谁是知情者？
3. **流动性供需的瞬时失衡** — 哪一方正在压迫对方？

对这些海量订单流进行极其细致的分类与解构，不仅是：

- 挖掘高频 Alpha（超额收益）因子的基石
- 构建预测模型的原料
- 优化算法执行（如降低市场冲击成本）的前提
- 进行微观市场系统性风险（如闪电崩盘预警）管理的核心手段

> **核心洞察**：微观结构中的每一个订单不仅携带了**方向与规模的物理属性**，更蕴含了**市场深度的博弈意图**。

基于此，量化研究界发展出了**多维度的订单流分类体系**，涵盖：

| 分类维度 | 核心问题 | 对应文档 |
|---------|---------|----------|
| 交易发起方方向 | 这笔成交是主动买还是主动卖？ | [Lee-Ready / BVC](TradeDirection_LeeReady_BVC.md) |
| 流动性微观力学 | 谁在提供流动性？谁在消耗？背后的均衡模型？ | [Kyle 模型推导](KyleModel_Derivation.md) |
| 信息毒性理论 | 这股订单流后面是否有"知情人"？ | [PIN / VPIN](PIN_VPIN.md) |
| 参与者行为轨迹 | 这是机构拆单、还是散户冲动、还是高频扫货？ | [Iceberg / Sweep](OrderTypes_Iceberg_Sweep.md) + [Boehmer 散户](RetailOrders_Boehmer.md) |
| 订单簿失衡因子 | 当前盘口的供需压迫力如何量化？ | [OFI / VOI](OFI_VOI.md) |

---

## 文件清单

| 文件 | 主题 |
|------|------|
| ⭐ [Summary.md](Summary.md) | **全模块总结**：五维度全景图、递进逻辑、实战建议 |
| [DataArchitecture_Physics.md](DataArchitecture_Physics.md) | 金融数据的"物理层次"架构：MBO→Tick→Snapshot→K线的因果影模型 |
| [TradeDirection_LeeReady_BVC.md](TradeDirection_LeeReady_BVC.md) | 交易方向推定：Lee-Ready / EMO / BVC 算法 |
| [KyleModel_Derivation.md](KyleModel_Derivation.md) | Kyle (1985) 单期连续拍卖模型的完整推导 |
| [PIN_VPIN.md](PIN_VPIN.md) | 信息毒性度量：PIN / VPIN 与体积时钟 |
| [OrderTypes_Iceberg_Sweep.md](OrderTypes_Iceberg_Sweep.md) | 冰山订单识别 + 扫单流分析 |
| [RetailOrders_Boehmer.md](RetailOrders_Boehmer.md) | Boehmer 算法：用子便士机制识别散户订单 |
| [OFI_VOI.md](OFI_VOI.md) | 订单流失衡 OFI 与交易量失衡 VOI 的对比与扩展 |

## 与其他笔记的关联

- [行情数据 L1/L2](../Basics/MarketData_L1_L2.md) — 数据层级基础
- [订单簿与成交价格推算](../Basics/OrderBook_Pricing.md) — 穿透订单簿、Micro-price、做市商博弈（Kyle 模型简版）
- [高频因子挖掘](../Strategies/HighFreqFactorMining.md) — Tick→K线聚合的因子构建范式
