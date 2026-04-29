# 逆向选择风险 (Adverse Selection)

微观结构中的核心概念，解释"为什么挂限价单的人天然处于信息劣势"。

> **一句话**：当你**提供**流动性（挂限价单）时，愿意和你成交的人往往是**因为他知道你不知道的事情**。

> 相关：[Kyle 模型完整推导](../Microstructure/KyleModel_Derivation.md)（信息不对称博弈的数学基石）| [PIN / VPIN](../Microstructure/PIN_VPIN.md)（知情交易概率）| [Maker-Taker 模型与返佣](MakerTaker_Rebates.md)（Maker 的"返佣补偿"正是对冲逆向选择）| [集合竞价与机构 14:56:59 大逃亡](../Markets/ClosingAuction_AShare_InstBehavior.md)

---

## 一、定义

**逆向选择 (Adverse Selection)** 源于信息经济学（Akerlof 1970 的"二手车市场"模型）。在交易微观结构中：

> 当你**提供流动性**（作为 Maker 挂限价单）时，你的**交易对手往往是拥有更多信息（Alpha）的人**。你成交的瞬间，就是**信息差兑现**的瞬间。

---

## 二、直白解释

**你的视角**：我挂 10.00 元买入，是因为我觉得它值 10.00 元。

**现实的微观结构**：有人愿意立刻把货卖给你，往往是因为他**知道**这股票马上就要跌到 9.90 元了。

→ 你以为对手和你一样在"博弈"，实际上对手是**单方面降维打击**。

---

## 三、三个经典场景

### 场景 1：做市商 vs 知情交易者

- 做市商挂 10.00 买 / 10.01 卖
- 99% 的时间里，撮合的是噪音交易者（散户），做市商稳赚 1 分价差
- **1% 的时间**，一个知情交易者（掌握重大消息）突然来扫货
- 做市商卖出后股价暴涨到 10.50，瞬间亏 49 分
- **长期结果**：逆向选择的亏损吞噬了 99% 噪音交易的盈利

这就是 [Kyle (1985)](../Microstructure/KyleModel_Derivation.md) 模型的核心——做市商必须**加宽价差**或**根据订单流调整报价**来补偿逆向选择。

### 场景 2：收盘集合竞价的散户陷阱

- 你在 14:57 挂买单，**无法撤销**
- 某机构通过大宗数据或内幕信息预判**收盘后会有重大利空**
- 利用你的买单作为**离场窗口**
- **结果**：你成交了，但你买入的那一刻，该资产的真实内在价值已经**低于成交价**
- 你得到的不是"便宜"，而是**"信息落后带来的亏损"**

详见 [集合竞价与机构 14:56:59 大逃亡](../Markets/ClosingAuction_AShare_InstBehavior.md)。

### 场景 3：VPIN 飙升时的做市商撤单

- VPIN 飙升 → 订单流毒性 → 做市商发现自己一直在**被单边吃单**
- 继续挂单 = 持续被逆向选择宰割
- 做市商的理性反应：**撤单、扩价差、或彻底退出**
- 这就是 **2010 年 Flash Crash** 的触发机制——做市商在 VPIN 飙升时集体退出 → 流动性真空 → 价格崩盘

详见 [PIN / VPIN](../Microstructure/PIN_VPIN.md)。

---

## 四、数学刻画（Glosten-Milgrom 模型）

最经典的逆向选择定价模型。设：

- 股票真实价值 $V \in \{V_H, V_L\}$，$V_H > V_L$
- 市场先验：$P(V = V_H) = \theta$
- 交易者构成：
  - 比例 $\mu$ 是**知情交易者**（知道 $V$ 真实值）
  - 比例 $1 - \mu$ 是**噪音交易者**（随机买卖）

### 做市商的报价推导

**卖价** $P_{\text{ask}}$ 应该满足：**收到一笔买单后的条件期望 $E[V \mid \text{Buy}]$**。

根据贝叶斯公式：

$$P(\text{Buy} \mid V = V_H) = \mu \cdot 1 + (1 - \mu) \cdot \frac{1}{2}$$

（知情者在 $V = V_H$ 时 100% 会买；噪音者 50% 概率买）

$$P(\text{Buy} \mid V = V_L) = \mu \cdot 0 + (1 - \mu) \cdot \frac{1}{2} = \frac{1 - \mu}{2}$$

用贝叶斯定理：

$$P(V = V_H \mid \text{Buy}) = \frac{P(\text{Buy} \mid V_H) \cdot \theta}{P(\text{Buy} \mid V_H) \cdot \theta + P(\text{Buy} \mid V_L) \cdot (1 - \theta)}$$

代入：

$$P(V = V_H \mid \text{Buy}) = \frac{\left(\mu + \frac{1-\mu}{2}\right) \theta}{\left(\mu + \frac{1-\mu}{2}\right) \theta + \frac{1-\mu}{2}(1 - \theta)}$$

做市商的理性卖价：

$$P_{\text{ask}} = P(V_H \mid \text{Buy}) \cdot V_H + P(V_L \mid \text{Buy}) \cdot V_L$$

**同理**，买价：

$$P_{\text{bid}} = P(V_H \mid \text{Sell}) \cdot V_H + P(V_L \mid \text{Sell}) \cdot V_L$$

### 价差公式

$$\boxed{\text{Spread} = P_{\text{ask}} - P_{\text{bid}} = \frac{2\mu \cdot \theta(1-\theta)(V_H - V_L)}{\mu^2 \theta(1-\theta) + [\mu + (1-\mu)/2]^2 \theta + [(1-\mu)/2]^2 (1-\theta)}$$

（具体形式复杂，关键是以下定性结论）

### 关键结论

- 知情者比例 $\mu \uparrow$ → 价差 $\uparrow$（逆向选择成本补偿）
- 信息不确定性 $(V_H - V_L) \uparrow$ → 价差 $\uparrow$
- $\mu \to 0$ → 价差 $\to 0$（无逆向选择 = 无价差）
- $\mu \to 1$ → 价差 $\to V_H - V_L$（全是知情者 = 不可能有做市）

---

## 五、对交易行为的影响

### 5.1 为什么做市商要撤单？

做市商面对**毒性订单流**（高 VPIN）时的理性反应：
1. 加宽价差（被动应对）
2. 撤销限价单（主动退出）
3. 把资金切换到更"干净"的股票

### 5.2 为什么机构在收盘集合竞价前撤单？

- 14:57 后无法撤单 = 无法对逆向选择做任何反应
- 3 分钟黑箱 = 知情者独享信息优势
- 详见 [集合竞价机构行为](../Markets/ClosingAuction_AShare_InstBehavior.md)

### 5.3 为什么 PFOF（美股）会付钱给散户订单？

美股做市商**付钱**给券商（Payment for Order Flow）来**换取散户订单流**——因为散户是**非知情噪音交易者**，做市商面对散户时逆向选择风险极低，可以稳定赚价差。

反过来，机构的订单流做市商**避之不及**（逆向选择风险太高）。

详见 [Boehmer 散户订单识别](../Microstructure/RetailOrders_Boehmer.md)。

### 5.4 为什么 Maker 返佣？

交易所给 Maker 返佣，本质上是**补偿 Maker 承担的逆向选择风险**。详见 [Maker-Taker 模型与返佣](MakerTaker_Rebates.md)。

---

## 六、如何度量逆向选择？

### 6.1 已实现价差 (Realized Spread)

$$\text{Realized Spread} = 2 \cdot d \cdot (P_t - P_{t+\tau})$$

其中 $d = +1$ 为买、$-1$ 为卖，$\tau$ 是几分钟后。

- 大的正 Realized Spread = Maker 成交后价格回归（低逆向选择）
- 负 Realized Spread = Maker 成交后价格继续同向走（高逆向选择）

### 6.2 Effective Spread 拆解

有效价差 = 已实现价差 + **逆向选择成本**：

$$\text{Effective Spread} = \text{Realized Spread} + \text{Price Impact (Adverse Selection)}$$

### 6.3 PIN / VPIN

知情交易概率直接衡量逆向选择强度。详见 [PIN / VPIN](../Microstructure/PIN_VPIN.md)。

### 6.4 Kyle's $\lambda$

价格冲击系数，等价于"每单位订单流带来的永久价格变动"——这部分变动来自逆向选择。详见 [Kyle 模型推导](../Microstructure/KyleModel_Derivation.md)。

---

## 七、一句话总结

> **逆向选择的本质**：
> 你挂限价单 = 卖出**无偿的看涨/看跌期权**给市场上所有信息优势方。
>
> 愿意行权的人，恰恰是**知道自己会赢**的人。
>
> 这张期权的成本就是价差、返佣、或是收盘 3 分钟黑箱里的"便宜货"。
