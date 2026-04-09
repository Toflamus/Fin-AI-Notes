# Kyle (1985) 模型的完整推导

## 背景与意义

在电子化限价订单簿 (LOB) 主导的现代金融交易所中，所有市场参与者被严格地映射为两种**微观物理角色**：**流动性的提供者**与**流动性的消耗者**。

这种分类不仅构成了各大交易所制定不对称手续费结构（**Maker-Taker Fee Model**：对提供流动性者予以返佣，对消耗流动性者收取费用）的商业基础，更是**资产价格发现与微观动力学演进的底层根源**。

### Maker 与 Taker 的定义

| 角色 | 中文 | 行为 | LOB 中的位置 |
|------|------|------|-------------|
| **Maker** | 流动性提供者（被动单） | 以**限价单 (Limit Order)** 形式提交意图，价格未与当前盘口形成交叉 | 静静地挂在最佳买卖价位 (L1) 或更深档位中排队，构筑"盘口厚度"与防御纵深 |
| **Taker** | 流动性消耗者（主动单） | 通过**市价单 (Market Order)** 或价格极具侵略性的穿透型限价单，直接与 Maker 挂单即时撮合 | 瞬间抽离并消耗系统的盘口流动性 |

当 **Taker 的激进订单流连续消耗 Maker 的静态订单**时，资产的微观均衡价格被迫发生位移——这一现象在微观结构中被称为**价格冲击 (Price Impact)**。

关于价格冲击与流动性深度的理论基石，源自 **Albert S. Kyle 于 1985 年**提出的经典单期连续拍卖模型。Kyle 模型天才般地构建了由：

- **掌握私有信息的知情交易者**
- **基于流动性需求进行随机买卖的噪音交易者**
- **负责出清市场但面临信息劣势的做市商**

组成的三方博弈论框架。

> 简版概述见：[订单簿与成交价格推算](../Basics/OrderBook_Pricing.md)（第 4 节）

---

## 1. 模型设定：三方博弈

| 参与者 | 信息状态 | 行为目标 |
|--------|---------|---------|
| **知情交易者 (Informed Trader)** | 拥有绝对私有信息，无成本观察真实价值 $v$ | 最大化预期利润 |
| **噪音交易者 (Noise Traders)** | 无信息，基于流动性需求随机买卖 | 无最优策略 |
| **做市商 (Market Maker)** | 信息劣势，无法区分订单来源 | 风险中性，竞争性出清，期望利润为零 |

---

## 2. 数学符号与先验分布

| 符号 | 含义 | 分布/性质 |
|------|------|----------|
| $v$ | 资产期末真实清算价值 | $v \sim N(p_0, \Sigma_0)$ |
| $p_0$ | 资产先验均值 | 公共已知 |
| $\Sigma_0$ | 基础价值的先验方差 | 公共已知 |
| $x$ | 知情交易者的订单需求量 | 决策变量 |
| $u$ | 噪音交易者的总净订单流 | $u \sim N(0, \sigma_u^2)$ |
| $y = x + u$ | 做市商观察到的总订单流 | 唯一可观察信号 |
| $p$ | 做市商设定的出清价格 | 决策变量 |
| $\lambda$ | Kyle's Lambda（价格冲击系数） | 待求 |

---

## 3. 做市商的定价规则

做市商无法从微观底层区分知情订单 $x$ 和噪音订单 $u$，只能观察总订单流 $y$。

依据**半强有效市场假说**与零利润竞争性条件，设定使期望利润为零的出清价格：

$$p = E[v \mid y]$$

假设需求线性，采用**线性定价规则 (Linear Pricing Rule)** 应对逆向选择风险：

$$\boxed{p = \mu + \lambda y}$$

- $\mu$：常数截距项
- $\lambda$：**Kyle's Lambda**，每单位净订单流对资产价格造成的不可逆冲击幅度
- $1/\lambda$：**市场绝对深度 (Market Depth)** 的严格数学定义

---

## 4. 知情交易者的最优策略

知情交易者深知自己的订单会引发做市商的价格调整（即知晓 $\lambda$ 的存在），因此最大化预期终端利润：

$$E[\pi] = E[x(v - p)] = x(v - \mu - \lambda x)$$

对 $x$ 求一阶导数并令其为零：

$$\frac{\partial E[\pi]}{\partial x} = v - \mu - 2\lambda x = 0$$

解出**最优隐藏交易策略**：

$$\boxed{x = \frac{v - \mu}{2\lambda}}$$

> **直觉**：知情交易者不会一次性把所有信息优势变现，而是按 $\frac{1}{2\lambda}$ 的强度逐步投放订单——价格冲击越大，他越要克制自己。

---

## 5. 均衡解

做市商理性预期知情交易者的需求函数呈线性结构 $x = \alpha + \beta v$。将上面解出的最优策略代入，并与做市商的贝叶斯理性预期均衡联立，得到一组优雅的线性均衡解：

$$\boxed{\mu = p_0}$$

$$\boxed{\lambda = \frac{1}{2} \sqrt{\frac{\Sigma_0}{\sigma_u^2}}}$$

$$\boxed{\alpha = -p_0 \sqrt{\frac{\sigma_u^2}{\Sigma_0}}}$$

$$\boxed{\beta = \sqrt{\frac{\sigma_u^2}{\Sigma_0}}}$$

---

## 6. Kyle's Lambda 的金融物理意义

这一组优雅的数学公式揭示了**微观金融物理学中最深刻的定律之一**：

公式 $\lambda = \dfrac{1}{2} \sqrt{\dfrac{\Sigma_0}{\sigma_u^2}}$ 明确指出了两个看似矛盾、实则深刻的事实：

### 6.1 波动率与流动性的反向关系

$\lambda \propto \sqrt{\Sigma_0}$（与基础价值不确定性正相关）

**直觉链条**：
1. 当资产基础价值的先验波动性 $\Sigma_0$ 越大
2. → 做市商（Maker）面临**与知情交易者做对手盘而遭受损失**的逆向选择风险就越高
3. → 因此，他们**别无选择**，只能加宽买卖价差并设定极高的价格冲击 $\lambda$ 以自我保护
4. → 表现为**市场整体流动性枯竭**

**这就是为什么**：在重大新闻发布前后、财报披露窗口、政策变动时段，市场流动性会急剧下降——做市商主动撤离了。

### 6.2 噪音与流动性的正向关系

$\lambda \propto \dfrac{1}{\sqrt{\sigma_u^2}}$（与噪音交易量负相关）

**直觉链条**：
1. 当市场中的**非知情噪音交易量** $\sigma_u^2$ 极大时
2. → 这些噪音为知情交易者提供了**完美的"伪装 (Camouflage)"**
3. → 做市商面对巨量双边随机订单，其感知到的逆向选择风险被**稀释**
4. → 从而降低 $\lambda$ 值
5. → 表现为**市场流动性极佳**

**这就是为什么**：散户活跃度高的交易日，机构反而更容易低成本建仓——散户提供了完美的掩护。

### 6.3 核心洞察

> **任何 Taker 订单流在消耗流动性时，本质上都在向 Maker 支付由 $\lambda$ 决定的"信息风险溢价"**。

这个溢价不是手续费，也不是买卖价差，而是**做市商对"你可能是知情者"这种可能性的事前定价**。即使你完全没有信息，也得为这份"可能性"埋单。这正是高频交易的核心战场：如何让做市商相信你只是噪音，从而以更低的成本完成执行。

### 6.4 跨学科物理类比：$\lambda$ 的三重物理身份

跳出纯粹的金融术语，从物理和动态系统的角度审视，$\lambda$ 具有三层极其直观且深刻的物理含义。

#### 6.4.1 电路学类比：市场的"倒电容" (Inverse Capacitance)

在电学中，电容 $C$ 衡量系统储存电荷的能力，基本公式为 $\Delta V = \frac{1}{C} Q$。

在 Kyle 模型中，做市商的线性定价规则：

$$\Delta p = \lambda \Delta y$$

| 电学量 | 金融对应 | 符号 |
|-------|---------|------|
| 电荷量 $Q$ | 订单流（向系统注入的交易量） | $\Delta y$ |
| 电势差 / 电压变化 $\Delta V$ | 价格变动 | $\Delta p$ |
| **弹性率** $1/C$ (Elastance) | **价格冲击系数** | $\lambda$ |
| **电容** $C$ | **市场深度** | $1/\lambda$ |

**类比的深层含义**：

一个具有高流动性、"深度"极好的市场，就像一个性能卓越的**超级电容器 (Supercapacitor)**——你可以向里面倾注海量的交易订单（大量电荷），而标的资产的价格（电压）几乎不会发生剧烈的波动，系统依然能够保持稳定。

反之，在一个缺乏流动性的"浅水"市场中（$\lambda$ 很大，电容极小），微小的订单流就会引发巨大的价格飙升或暴跌——这在电学中对应的就是**电压击穿 (Voltage Breakdown)**。

#### 6.4.2 信号处理类比：系统信噪比的具象化 (Signal-to-Noise Ratio)

根据连续时间模型的推导结果：

$$\lambda = \sqrt{\frac{\Sigma_0}{\sigma_u^2}}$$

| 信号处理量 | 金融对应 | 符号 |
|-----------|---------|------|
| **信号强度** (Signal) | 资产真实价值的先验方差（信息池的总能量） | $\Sigma_0$ |
| **噪声强度** (Noise) | 噪声交易者订单量的方差（系统背景噪声） | $\sigma_u^2$ |

$\lambda$ 本质上是该动态系统的**信噪比 (SNR) 的平方根**。

做市商在这个系统中扮演着类似**卡尔曼滤波器 (Kalman Filter)** 的角色：
- 做市商只能看到总输入（总订单流 $y$）
- 不知道里面有多少是真正携带信息的信号（内幕交易），有多少是纯粹的无序噪声
- 当系统的信噪比很高时（内幕信息极具破坏力，且缺乏足够的噪声作为缓冲），做市商为了防止自身被"信息降维打击"，会主动调大 $\lambda$
- 这相当于**增加了系统的摩擦力或阻尼**，使得系统对输入的响应变得极其敏感，哪怕是一点点订单流也会让做市商大幅调整价格预期以进行防御

> 这与 [正态投影定理](../Basics/NormalProjectionTheorem.md) 中推导的增益系数 $\lambda = \frac{\text{Cov}(V,Y)}{\text{Var}(Y)}$ 完全一致——它就是卡尔曼增益在 Kyle 设定下的具体表达。

#### 6.4.3 动力学类比：对抗外部冲击的"刚度" (Stiffness)

如果把价格体系看作一个**弹簧振子系统**，$\Delta p = \lambda \Delta y$ 在结构上与胡克定律完全一致：

$$F = kx \quad \longleftrightarrow \quad \Delta p = \lambda \Delta y$$

| 力学量 | 金融对应 |
|-------|---------|
| 外力 $F$ | 净买入/净卖出（订单流冲击） |
| 形变 $x$ | 价格偏离均衡位置的位移 |
| **弹簧刚度** $k$ | **价格冲击系数** $\lambda$ |

- $\lambda$ 越小（市场越深） → 弹簧越软 → 系统吸收外力形变的能力越强
- $\lambda$ 越大（市场越浅） → 弹簧越硬 → 强行施加外力会导致做功（交易成本/滑点）急剧增加

正是因为这个刚度（摩擦力）的存在，**限制了内幕交易者的行为**。内幕交易者通过观测当前的 $\lambda$，必须像系统缓慢释放热量一样，进行极为精密的"受控释放"，将私人信息平滑、连续地融入到价格的布朗运动中，而不是瞬间打出所有底牌。

#### 6.4.4 三重物理身份总结

在物理与工程的视角下，$\lambda$ 并非只是一个简单的金融回归参数：

| 物理身份 | 公式对应 | 直觉 |
|---------|---------|------|
| 系统的**倒电容** (Elastance) | $\Delta p = \lambda \Delta y \sim \Delta V = \frac{1}{C}Q$ | 市场容纳资金流的"蓄水池"大小 |
| 系统的**信噪比平方根** | $\lambda = \sqrt{\Sigma_0 / \sigma_u^2}$ | 做市商的卡尔曼滤波增益 |
| 系统的**弹簧刚度** (Stiffness) | $\Delta p = \lambda \Delta y \sim F = kx$ | 价格体系对抗外部冲击的"硬度" |

> 它用极其优美的数学形式，将"流动性"和"信息毒性"这两个原本抽象的金融概念，转化为了**可精确度量的物理属性**。

---

## 7. 撤单流与微观信息操纵

考察流动性不能仅盯成交数据。在 HFT 环境中，**撤单 (Cancellation) 往往比实际成交携带更致密的前瞻性信息**。

### 7.1 高频做市商的撤单行为

- 高频做市商通过向订单簿两端高频发送限价单并极速撤销来维持风险敞口中性
- 订单存活周期处于**亚秒级**
- 撤单流的微观动力学演变直接反映市场情绪的脆弱点

### 7.2 高撤单/成交比的两种微观状态

**状态 1：理性逃跑（防御性撤单）**
- 跨市场套利信号或相关资产波动 → 做市商察觉到极高的逆向选择风险
- 为避免被即将到来的单向趋势流"碾压 (Run over)"
- 集体瞬间撤销深层挂单 → 制造**流动性真空**

**状态 2：恶意操纵（晃骗 / 分层）**
- **Spoofing / Layering**：在远离最优价格的深层档位挂出极其庞大的虚假限价单
- 制造流动性严重失衡的视觉假象
- 诱导其他基于微观动量跟踪的算法提前入场接盘
- 在毫秒级内撤销所有虚假挂单，反向收割利润

### 7.3 实战意义

精准剥离并分析撤单流是现代高频量化风控模型不可或缺的组件。撤单流的高质量重构需要 **MBO (Market-by-Order)** 数据源，能够追踪每一个独立 Order ID 的生命周期。

---

## 8. 单次拍卖模型的完整结论

在前述的单次拍卖（第 1-6 节推导的模型）中，核心结果汇总：

| 均衡参数 | 表达式 | 含义 |
|---------|--------|------|
| $\lambda$ | $\dfrac{1}{2}\sqrt{\dfrac{\Sigma_0}{\sigma_u^2}}$ | 价格冲击系数 / 市场深度的倒数 |
| $\beta$ | $\sqrt{\dfrac{\sigma_u^2}{\Sigma_0}}$ | 内幕交易者的交易强度 |
| $\mu$ | $p_0$ | 定价截距 = 先验均值 |
| $\alpha$ | $-\beta p_0$ | 策略截距 |

### 8.1 求解均衡参数的完整代数过程

将 $\beta = \frac{1}{2\lambda}$ 代入 $\lambda$ 的表达式：

$$\lambda = \frac{\beta \Sigma_0}{\beta^2 \Sigma_0 + \sigma_u^2} = \frac{\frac{1}{2\lambda} \Sigma_0}{\frac{1}{4\lambda^2} \Sigma_0 + \sigma_u^2}$$

通分化简分母：

$$\lambda = \frac{\frac{\Sigma_0}{2\lambda}}{\frac{\Sigma_0 + 4\lambda^2 \sigma_u^2}{4\lambda^2}}$$

交叉相乘：

$$\lambda \cdot \frac{\Sigma_0 + 4\lambda^2 \sigma_u^2}{4\lambda^2} = \frac{\Sigma_0}{2\lambda}$$

$$\frac{\Sigma_0 + 4\lambda^2 \sigma_u^2}{4\lambda} = \frac{\Sigma_0}{2\lambda}$$

两边乘以 $4\lambda$：

$$\Sigma_0 + 4\lambda^2 \sigma_u^2 = 2\Sigma_0$$

$$4\lambda^2 \sigma_u^2 = \Sigma_0$$

$$\lambda^2 = \frac{\Sigma_0}{4\sigma_u^2}$$

$$\boxed{\lambda = \frac{1}{2}\sqrt{\frac{\Sigma_0}{\sigma_u^2}}}$$

代回 $\beta = \frac{1}{2\lambda}$：

$$\boxed{\beta = \sqrt{\frac{\sigma_u^2}{\Sigma_0}}}$$

### 8.2 $\Sigma_1 = \frac{1}{2}\Sigma_0$ 的推导

交易后的剩余方差（条件方差）：

$$\Sigma_1 = \text{Var}(v \mid y) = \Sigma_0 - \frac{[\text{Cov}(v, y)]^2}{\text{Var}(y)}$$

代入已知结果：

$$\text{Cov}(v, y) = \beta \Sigma_0 = \sqrt{\frac{\sigma_u^2}{\Sigma_0}} \cdot \Sigma_0 = \sqrt{\sigma_u^2 \Sigma_0}$$

$$\text{Var}(y) = \beta^2 \Sigma_0 + \sigma_u^2 = \frac{\sigma_u^2}{\Sigma_0} \cdot \Sigma_0 + \sigma_u^2 = 2\sigma_u^2$$

$$\Sigma_1 = \Sigma_0 - \frac{(\sqrt{\sigma_u^2 \Sigma_0})^2}{2\sigma_u^2} = \Sigma_0 - \frac{\sigma_u^2 \Sigma_0}{2\sigma_u^2} = \Sigma_0 - \frac{\Sigma_0}{2} = \frac{1}{2}\Sigma_0$$

> **结论**：单次博弈中，内幕交易者恰好将**一半**的私人信息融入了价格。

### 8.3 内幕交易利润的推导

将最优策略 $x^* = \beta(v - p_0)$ 代入利润公式：

$$E[\pi] = E[x(v - p)] = E[\beta(v - p_0)(v - p_0 - \lambda(\beta(v - p_0) + u))]$$

$$= E[\beta(v - p_0)^2(1 - \lambda\beta)] - E[\beta(v - p_0)\lambda u]$$

由于 $v$ 和 $u$ 独立，第二项为 0。又因为 $\lambda\beta = \frac{1}{2}$：

$$E[\pi] = \beta(1 - \frac{1}{2})E[(v - p_0)^2] = \frac{1}{2}\beta\Sigma_0 = \frac{1}{2}\sqrt{\frac{\sigma_u^2}{\Sigma_0}} \cdot \Sigma_0 = \frac{1}{2}\sqrt{\Sigma_0 \sigma_u^2}$$

$$\boxed{E[\pi] = \frac{1}{2}\sqrt{\Sigma_0 \sigma_u^2}}$$

---

## 9. 模型二：序列拍卖均衡 (Sequential Auction)

当交易日被分割为 $N$ 个离散的时间段 $t_n \in [0, 1]$ 时，内幕交易者面临**动态优化**问题。

### 9.1 设定

每期的价格变动与交易量：

$$\Delta p_n = \lambda_n(\Delta x_n + \Delta u_n)$$

$$\Delta x_n = \beta_n(v - p_{n-1})\Delta t_n$$

> 为什么策略一定是这种线性形式？因为这是一个线性二次型 (LQ) 动态规划问题——线性状态转移 + 二次收益函数 → 最优策略必然是状态变量的线性函数。详细的数学证明见 [LQ 动态规划](../Basics/LQ_DynamicProgramming.md)。

### 9.2 第一步：后向归纳求最优策略

假设在第 $n$ 期，内幕交易者对未来所有期的预期利润函数是**二次形式**（由 LQ 结构保证）：

$$E[\pi_n \mid p_{n-1}, v] = \alpha_{n-1}(v - p_{n-1})^2 + \delta_{n-1}$$

在第 $n$ 期，最大化当期利润 + 未来期望利润之和：

$$\max_{\Delta x} E\left[(v - p_n)\Delta x + \alpha_n(v - p_n)^2 + \delta_n\right]$$

设 $S_{n-1} = v - p_{n-1}$（当前获利空间），代入价格转移方程 $v - p_n = S_{n-1} - \lambda_n \Delta x - \lambda_n \Delta u_n$：

$$E\left[(S_{n-1} - \lambda_n \Delta x - \lambda_n \Delta u_n)\Delta x + \alpha_n(S_{n-1} - \lambda_n \Delta x - \lambda_n \Delta u_n)^2 + \delta_n\right]$$

利用 $E[\Delta u_n] = 0$ 和 $E[(\Delta u_n)^2] = \sigma_u^2 \Delta t_n$，取期望：

$$(S_{n-1} - \lambda_n \Delta x)\Delta x + \alpha_n\left[(S_{n-1} - \lambda_n \Delta x)^2 + \lambda_n^2 \sigma_u^2 \Delta t_n\right] + \delta_n$$

对 $\Delta x$ 求导令其为 0。先展开目标函数的各项：

**本期利润项**：$(S_{n-1} - \lambda_n \Delta x)\Delta x = S_{n-1}\Delta x - \lambda_n(\Delta x)^2$

对 $\Delta x$ 求导：$S_{n-1} - 2\lambda_n \Delta x$

**未来利润项**：$\alpha_n(S_{n-1} - \lambda_n \Delta x)^2 = \alpha_n[S_{n-1}^2 - 2S_{n-1}\lambda_n \Delta x + \lambda_n^2(\Delta x)^2]$

对 $\Delta x$ 求导：$\alpha_n[-2S_{n-1}\lambda_n + 2\lambda_n^2 \Delta x] = -2\alpha_n \lambda_n S_{n-1} + 2\alpha_n \lambda_n^2 \Delta x$

（噪声项 $\alpha_n \lambda_n^2 \sigma_u^2 \Delta t_n + \delta_n$ 不含 $\Delta x$，导数为 0）

**两部分合并**，令导数为 0：

$$\underbrace{S_{n-1} - 2\lambda_n \Delta x}_{\text{本期利润的导数}} + \underbrace{(-2\alpha_n \lambda_n S_{n-1} + 2\alpha_n \lambda_n^2 \Delta x)}_{\text{未来利润的导数}} = 0$$

整理 $S_{n-1}$ 和 $\Delta x$ 的系数：

$$S_{n-1}(1 - 2\alpha_n \lambda_n) = \Delta x(2\lambda_n - 2\alpha_n \lambda_n^2)$$

$$(1 - 2\alpha_n \lambda_n)S_{n-1} = 2\lambda_n(1 - \alpha_n \lambda_n)\Delta x$$

解出最优交易量：

$$\boxed{\Delta x_n^* = \frac{1 - 2\alpha_n \lambda_n}{2\lambda_n(1 - \alpha_n \lambda_n)} \cdot (v - p_{n-1})}$$

即 $\beta_n \Delta t_n = \dfrac{1 - 2\alpha_n \lambda_n}{2\lambda_n(1 - \alpha_n \lambda_n)}$。

> 交易强度 $\beta$ 由**未来的获利潜力参数 $\alpha_n$** 和**当前价格冲击 $\lambda_n$** 共同决定。如果 $\alpha_n$ 很大（未来利润权重高），$\beta_n$ 会更小——内幕交易者更克制，防止信息过早泄露。

### 9.3 第二步：做市商的贝叶斯更新

做市商使用[正态投影定理](../Basics/NormalProjectionTheorem.md)更新价格和剩余不确定性。

**定价系数**（推导过程与单次拍卖完全类似，只需注意时间步长）：

$$\lambda_n = \frac{\beta_n \Sigma_{n-1}}{\beta_n^2 \Sigma_{n-1} \Delta t_n + \sigma_u^2}$$

**方差递推**（剩余信息量的衰减）：

根据正态分布条件方差的更新公式 $\Sigma_n = \text{Var}(v) - \frac{\text{Cov}(v, y_n)^2}{\text{Var}(y_n)}$：

$$\text{Cov}(v, y_n) = \beta_n \Delta t_n \cdot \Sigma_{n-1}$$

$$\text{Var}(y_n) = \beta_n^2 (\Delta t_n)^2 \Sigma_{n-1} + \sigma_u^2 \Delta t_n$$

$$\lambda_n = \frac{\text{Cov}(v, y_n)}{\text{Var}(y_n)} = \frac{\beta_n \Sigma_{n-1} \Delta t_n}{\beta_n^2 (\Delta t_n)^2 \Sigma_{n-1} + \sigma_u^2 \Delta t_n}$$

（分子分母同除 $\Delta t_n$ 得到上面的简化形式）

方差更新：

$$\Sigma_n = \Sigma_{n-1} - \lambda_n \cdot \text{Cov}(v, y_n) = \Sigma_{n-1} - \lambda_n \beta_n \Delta t_n \Sigma_{n-1}$$

$$\boxed{\Sigma_n = (1 - \beta_n \lambda_n \Delta t_n)\Sigma_{n-1}}$$

> 每过一期，$\Sigma_n$ 都比 $\Sigma_{n-1}$ 小（因为 $1 - \beta \lambda \Delta t < 1$）。这代表随着交易进行，**私人信息逐渐被"挤"进价格中**，秘密不再是秘密。

### 9.4 递推闭环

三组方程构成联立差分方程系统：

$$\beta_n \Delta t_n = \frac{1 - 2\alpha_n \lambda_n}{2\lambda_n(1 - \alpha_n \lambda_n)} \quad \text{(内幕交易者最优策略)}$$

$$\lambda_n = \frac{\beta_n \Sigma_{n-1}}{\beta_n^2 \Sigma_{n-1} \Delta t_n + \sigma_u^2} \quad \text{(做市商定价)}$$

$$\Sigma_n = (1 - \beta_n \lambda_n \Delta t_n)\Sigma_{n-1} \quad \text{(信息衰减)}$$

### 9.5 序列拍卖的结论

1. **内幕交易者不会一次性打完所有子弹**，而是策略性地分批交易
2. $\Sigma_n$ 随时间单调递减——私人信息**逐渐**融入价格
3. 相比单次拍卖，分批交易使内幕交易者获得**更高的总利润**（通过在噪声中伪装自己）

---

## 10. 模型三：连续拍卖均衡 (Continuous Auction)

当 $N \to \infty$（$\Delta t \to 0$），序列拍卖的差分方程变为**微分方程**。这是 Kyle (1985) 论文最精华的部分。

### 10.1 连续时间的微分形式

交易规则和定价规则转化为：

$$dx(t) = \beta(t)[v - p(t)]dt$$

$$dp(t) = \lambda(t)[dx(t) + du(t)]$$

内幕交易者最大化整个区间 $[0, 1]$ 的总利润：

$$E[\pi(0)] = \int_0^1 \beta(t) \Sigma^*(t) \, dt$$

其中 $\Sigma^*(t) = E[(v - p(t))^2]$ 为 $t$ 时刻的剩余信息方差。

### 10.2 核心推演：$\lambda$ 必须是常数

Kyle 运用了一个极其巧妙的**反证法**来证明 $\lambda(t)$ 在连续均衡中必须恒定：

**假设 1**：如果 $\lambda(t)$ 随时间**下降**（市场深度变大）
- 内幕交易者会先疯狂交易破坏价格（Destabilize）
- 等市场变深后再反向交易
- → 获取**无限利润**（套利机会）
- → 违背均衡假设

**假设 2**：如果 $\lambda(t)$ 随时间**上升**
- 内幕交易者会希望**把所有信息立刻释放完毕**
- → 退化为单次拍卖
- → 不是连续均衡

为了使系统处于均衡，排除无限套利的可能：

$$\boxed{\lambda(t) = \lambda = \text{常数}}$$

### 10.3 求解连续参数

$dp$ 的瞬间方差由噪声主导，为 $\lambda^2 \sigma_u^2 dt$。

为了保证在交易结束时 ($t = 1$) **所有信息都被完全吸纳**（$\Sigma(1) = 0$），价格方差的积分必须等于初始方差：

$$\int_0^1 \lambda^2 \sigma_u^2 dt = \Sigma_0 \implies \lambda^2 \sigma_u^2 = \Sigma_0$$

$$\boxed{\lambda = \sqrt{\frac{\Sigma_0}{\sigma_u^2}}}$$

> 注意：连续模型的 $\lambda$ 是单次拍卖 $\lambda$ 的**两倍**。

由有效市场方程推导出其他参数：

| 参数 | 表达式 | 时间依赖 |
|------|--------|---------|
| 剩余信息 | $\Sigma(t) = (1 - t)\Sigma_0$ | 随 $t$ 线性递减 |
| 交易强度 | $\beta(t) = \dfrac{\sigma_u}{\sqrt{\Sigma_0}} \cdot \dfrac{1}{1 - t}$ | 随 $t \to 1$ 爆发至 $\infty$ |
| 价格冲击 | $\lambda = \sqrt{\dfrac{\Sigma_0}{\sigma_u^2}}$ | **恒定** |

### 10.4 连续拍卖的深刻结论

#### 结论 1：完美的信息融入 (Perfect Price Discovery)

$$\Sigma(t) = (1 - t)\Sigma_0$$

当 $t \to 1$ 时，$\Sigma(1) = 0$。

> 在交易结束的瞬间，内幕交易者的私人信息被**百分之百地完全纳入**了价格体系。

#### 结论 2：价格服从布朗运动

在做市商眼里，价格波动率 $\lambda^2 \sigma_u^2$ 是常数。这意味着半强式有效的价格动态呈现**无漂移的布朗运动 (Brownian Motion)**：

> 尽管存在具备定向信息的内幕人，市场依然在**表面上表现为随机游走**。

#### 结论 3：交易强度的末期爆发

由于 $\beta(t) \propto \dfrac{1}{1-t}$，当 $t \to 1$ 时，$\beta(t) \to \infty$。

> 内幕交易者在交易**初期非常耐心**（隐藏在噪声中慢慢买/卖），但在**临近结束时**，为了榨干私人信息最后的剩余价值，交易量**急剧放大**。

#### 结论 4：连续博弈使利润翻倍

连续均衡下的总期望利润：

$$E[\pi] = \sqrt{\Sigma_0 \sigma_u^2}$$

这恰好是单次拍卖利润 $\frac{1}{2}\sqrt{\Sigma_0 \sigma_u^2}$ 的**两倍**。

> 通过跨期策略性地伪装自己，内幕交易者能够榨取更多的经济租金。分批、隐忍、末期爆发——这是最优信息利用策略。

---

## 11. 三个模型的对比总结

| 维度 | 单次拍卖 | 序列拍卖 ($N$ 期) | 连续拍卖 ($N \to \infty$) |
|------|---------|-----------------|-------------------------|
| $\lambda$ | $\frac{1}{2}\sqrt{\Sigma_0 / \sigma_u^2}$ | 每期不同 $\lambda_n$ | **恒定** $\sqrt{\Sigma_0 / \sigma_u^2}$ |
| 信息释放 | 恰好**一半** ($\Sigma_1 = \frac{1}{2}\Sigma_0$) | **逐期递减** | **线性递减** $\Sigma(t) = (1-t)\Sigma_0$ |
| 交易策略 | 一次性出手 | 分批，越来越急 | 初期耐心，末期爆发 |
| 内幕利润 | $\frac{1}{2}\sqrt{\Sigma_0 \sigma_u^2}$ | 介于单次与连续之间 | $\sqrt{\Sigma_0 \sigma_u^2}$（单次的**2 倍**） |
| 价格过程 | 一次跳跃 | 离散跳跃序列 | **布朗运动** |
| 终态 | 保留 50% 信息 | 保留部分信息 | **$\Sigma(1) = 0$，完全释放** |
