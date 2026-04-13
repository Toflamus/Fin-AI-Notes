# 泊松分布 (Poisson Distribution)

泊松分布是一个专门用来描述**在一段固定的时间或空间内，某个随机事件发生特定次数的概率**的数学工具。当我们知道某个事件平均会发生多少次，泊松分布就能帮我们预测它实际发生各种次数的可能性。

它通常用于描述那些"相对罕见"或"各自独立"的事件。

> 在量化金融中，泊松分布用于建模：
> - 单位时间内到达的订单数量（[PIN 模型](../Microstructure/PIN_VPIN.md) 中知情/噪音交易者的到达率 $\mu, \epsilon_b, \epsilon_s$）
> - 极端事件的发生频率（如闪电崩盘）
> - 做市商面临的订单到达过程

---

## 一、核心公式

泊松分布只有一个核心参数 $\lambda$ (Lambda)，代表在固定时间或空间内事件发生的**平均次数**（期望值）。

事件刚好发生 $k$ 次的概率：

$$\boxed{P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!}}$$

| 符号 | 含义 |
|------|------|
| $P(X=k)$ | 事件刚好发生 $k$ 次的概率 |
| $\lambda$ | 事件发生的**平均次数**（唯一参数） |
| $e$ | 自然对数的底数（常数，$\approx 2.71828$） |
| $k!$ | $k$ 的阶乘（例如 $3! = 3 \times 2 \times 1 = 6$） |

### 基本性质

| 性质 | 表达式 |
|------|--------|
| 期望 | $E[X] = \lambda$ |
| 方差 | $\text{Var}(X) = \lambda$ |
| 特征 | **期望 = 方差**（这是泊松分布的独特标志） |

---

## 二、从二项分布推导泊松分布

泊松分布不是凭空出现的，它是**二项分布 (Binomial Distribution) 在极端条件下的极限形式**。

### 2.1 二项分布的起点

假设进行 $n$ 次独立试验（如抛硬币），每次成功概率为 $p$。事件恰好成功 $k$ 次的概率由二项分布给出：

$$P(X = k) = \binom{n}{k} p^k (1-p)^{n-k} = \frac{n!}{k!(n-k)!} p^k (1-p)^{n-k}$$

### 2.2 引入极端条件

假设：
- 试验次数 $n \to \infty$（无穷多次试验）
- 单次成功概率 $p \to 0$（每次成功的概率极小）
- 但平均成功总次数保持为常数：$\lambda = np$，即 $p = \frac{\lambda}{n}$

### 2.3 完整推导

将 $p = \frac{\lambda}{n}$ 代入二项分布公式，并展开组合数：

$$P(X = k) = \frac{n(n-1)(n-2)\cdots(n-k+1)}{k!} \left(\frac{\lambda}{n}\right)^k \left(1 - \frac{\lambda}{n}\right)^{n-k}$$

重新组合，将含有 $n$ 的项分离出来：

$$P(X = k) = \frac{\lambda^k}{k!} \cdot \underbrace{\left(\frac{n}{n} \cdot \frac{n-1}{n} \cdot \frac{n-2}{n} \cdots \frac{n-k+1}{n}\right)}_{\text{第一组}} \cdot \underbrace{\left(1 - \frac{\lambda}{n}\right)^n}_{\text{第二组}} \cdot \underbrace{\left(1 - \frac{\lambda}{n}\right)^{-k}}_{\text{第三组}}$$

当 $n \to \infty$ 时，逐组取极限：

**第一组**：对于任意有限的 $k$，

$$\lim_{n \to \infty} \frac{n-i}{n} = 1 \quad \text{（对所有 } i = 0, 1, \ldots, k-1 \text{）}$$

所以第一组的乘积趋近于 **1**。

**第二组**：根据微积分中自然对数底数 $e$ 的重要极限定义，

$$\lim_{n \to \infty} \left(1 - \frac{\lambda}{n}\right)^n = e^{-\lambda}$$

**第三组**：因为 $\frac{\lambda}{n} \to 0$，

$$\lim_{n \to \infty} \left(1 - \frac{\lambda}{n}\right)^{-k} = 1^{-k} = 1$$

将三组极限结果相乘：

$$P(X = k) = \frac{\lambda^k}{k!} \cdot 1 \cdot e^{-\lambda} \cdot 1$$

$$\boxed{P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!}}$$

### 2.4 这个推导的意义

> **当事件发生的概率极小，但观测的样本量极大时，二项分布就演变成了泊松分布。**

这解释了为什么泊松分布适用于"罕见事件"的建模——它是大量微小概率事件叠加的自然结果。

---

## 三、泊松过程：三个严苛的数学前提

在时间轴上，要让一个随机过程真正服从泊松分布（这被称为**泊松过程**），它必须严格满足以下三个数学条件。

假设在极短的时间间隔 $\Delta t$ 内：

### 条件一：独立增量 (Independent Increments)

在**互不重叠的时间段**内，事件发生的次数是**完全相互独立的**。

昨天发生多少次，完全不影响今天发生的概率。

### 条件二：平稳性 (Stationarity)

在极短的时间 $\Delta t$ 内，发生 1 次事件的概率与时间间隔的长短**成正比**，比例常数为 $\lambda$（发生率）。

用数学语言表达，即存在高阶无穷小 $o(\Delta t)$ 使得：

$$P(\text{发生 1 次}) = \lambda \Delta t + o(\Delta t)$$

### 条件三：普通性 (Orderliness / Non-multiplicity)

在极短的时间 $\Delta t$ 内，**同时发生 2 次及以上**事件的概率极小，可以忽略不计：

$$P(\text{发生} \geq 2 \text{次}) = o(\Delta t)$$

> **直觉**：事件是"一个接一个"发生的，不会在同一瞬间"扎堆"出现。

---

## 四、从微分方程推导泊松公式

### 4.1 先解 $P_0(t)$：$t$ 时间内发生 0 次的概率

如果在 $t + \Delta t$ 时间内发生了 0 次，意味着在前 $t$ 时间内发生 0 次，**并且**在新增的 $\Delta t$ 内也发生 0 次：

$$P_0(t + \Delta t) = P_0(t) \cdot (1 - \lambda \Delta t - o(\Delta t))$$

整理并求导：

$$\frac{P_0(t + \Delta t) - P_0(t)}{\Delta t} = -\lambda P_0(t) - \frac{o(\Delta t)}{\Delta t}$$

当 $\Delta t \to 0$ 时，$\frac{o(\Delta t)}{\Delta t} \to 0$，我们得到微分方程：

$$\frac{dP_0(t)}{dt} = -\lambda P_0(t)$$

这是一个标准的**一阶线性常微分方程**。解法：分离变量后积分，利用初始条件 $P_0(0) = 1$（时间为 0 时，发生 0 次的概率当然是 1）：

$$\boxed{P_0(t) = e^{-\lambda t}}$$

### 4.2 建立通用的递推微分方程

要想在总时间 $t + \Delta t$ 内刚好得到 $k$ 次事件，基于普通性条件（$\Delta t$ 内最多发生 1 次），只有两种互斥的情况：

**情况 A（保持不变）**：前 $t$ 时间内已经发生了 $k$ 次，新增的 $\Delta t$ 内发生了 **0 次**。

$$\text{概率：} P_k(t) \cdot (1 - \lambda \Delta t)$$

**情况 B（增加一次）**：前 $t$ 时间内只发生了 $k-1$ 次，新增的 $\Delta t$ 内刚好发生了 **1 次**。

$$\text{概率：} P_{k-1}(t) \cdot (\lambda \Delta t)$$

两种情况相加：

$$P_k(t + \Delta t) = P_k(t)(1 - \lambda \Delta t) + P_{k-1}(t)(\lambda \Delta t)$$

展开并移项：

$$P_k(t + \Delta t) - P_k(t) = -\lambda P_k(t) \Delta t + \lambda P_{k-1}(t) \Delta t$$

两边除以 $\Delta t$，令 $\Delta t \to 0$：

$$\boxed{\frac{dP_k(t)}{dt} = -\lambda P_k(t) + \lambda P_{k-1}(t)}$$

> 这就是泊松过程的**核心递推微分方程**。它告诉我们：状态 $k$ 的变化率 = 从状态 $k-1$ 转移进来的速率 $-$ 从状态 $k$ 转移出去的速率。

### 4.3 用积分因子法解出 $P_1(t)$

将 $k = 1$ 以及已知的 $P_0(t) = e^{-\lambda t}$ 代入递推方程：

$$\frac{dP_1(t)}{dt} + \lambda P_1(t) = \lambda e^{-\lambda t}$$

这是一个标准的**一阶线性常微分方程**，形式为 $y' + py = q$。求解方法是寻找**积分因子 (Integrating Factor)**。

等式两边同时乘以积分因子 $e^{\lambda t}$：

$$e^{\lambda t} \frac{dP_1(t)}{dt} + \lambda e^{\lambda t} P_1(t) = \lambda e^{-\lambda t} \cdot e^{\lambda t}$$

根据乘积求导法则 $[u \cdot v]' = u'v + uv'$，等式左边恰好是 $(P_1(t) \cdot e^{\lambda t})$ 的导数；等式右边两个指数项相乘抵消，只剩下 $\lambda$：

$$\frac{d}{dt}\left[P_1(t) \cdot e^{\lambda t}\right] = \lambda$$

对等式两边关于 $t$ 积分：

$$P_1(t) \cdot e^{\lambda t} = \int \lambda \, dt = \lambda t + C$$

**确定常数 $C$**：初始条件 $P_1(0) = 0$（时间为 0 时，发生 1 次的概率必然是 0）：

$$0 \cdot e^0 = 0 + C \implies C = 0$$

因此：

$$P_1(t) \cdot e^{\lambda t} = \lambda t$$

$$\boxed{P_1(t) = (\lambda t) e^{-\lambda t}}$$

### 4.4 解出 $P_2(t)$，发现阶乘的踪迹

将 $k = 2$ 和 $P_1(t) = \lambda t e^{-\lambda t}$ 代入递推方程：

$$\frac{dP_2(t)}{dt} + \lambda P_2(t) = \lambda(\lambda t e^{-\lambda t}) = \lambda^2 t e^{-\lambda t}$$

使用完全相同的积分因子法，两边同乘 $e^{\lambda t}$：

$$\frac{d}{dt}\left[P_2(t) \cdot e^{\lambda t}\right] = \lambda^2 t$$

两边积分：

$$P_2(t) \cdot e^{\lambda t} = \int \lambda^2 t \, dt = \lambda^2 \cdot \frac{1}{2} t^2 + C$$

初始条件 $P_2(0) = 0$，所以 $C = 0$。

$$P_2(t) = \frac{(\lambda t)^2}{2} e^{-\lambda t}$$

> 看！分母上的 **2** 出现了，它其实就是 $2!$ 的雏形。它来源于对变量 $t$ 的积分——$\int t \, dt = \frac{t^2}{2}$。

$$\boxed{P_2(t) = \frac{(\lambda t)^2}{2!} e^{-\lambda t}}$$

### 4.5 数学归纳：推导一般的 $P_k(t)$

**归纳假设**：假设我们已经求出了 $P_{k-1}(t)$ 的公式如下：

$$P_{k-1}(t) = \frac{(\lambda t)^{k-1}}{(k-1)!} e^{-\lambda t}$$

**归纳步骤**：将它代入求 $P_k(t)$ 的微分方程：

$$\frac{dP_k(t)}{dt} + \lambda P_k(t) = \lambda \cdot \frac{(\lambda t)^{k-1}}{(k-1)!} e^{-\lambda t}$$

$$\frac{dP_k(t)}{dt} + \lambda P_k(t) = \frac{\lambda^k t^{k-1}}{(k-1)!} e^{-\lambda t}$$

再次同乘积分因子 $e^{\lambda t}$，凑成导数形式：

$$\frac{d}{dt}\left[P_k(t) \cdot e^{\lambda t}\right] = \frac{\lambda^k t^{k-1}}{(k-1)!}$$

对等式两边积分。右边只有 $t^{k-1}$ 是变量，其他都是常数：

$$P_k(t) \cdot e^{\lambda t} = \frac{\lambda^k}{(k-1)!} \int t^{k-1} \, dt$$

根据幂函数的积分公式 $\int x^n \, dx = \frac{x^{n+1}}{n+1}$：

$$P_k(t) \cdot e^{\lambda t} = \frac{\lambda^k}{(k-1)!} \cdot \frac{t^k}{k} + C$$

初始条件 $P_k(0) = 0$（当 $k \geq 1$ 时），所以 $C = 0$。

**见证奇迹的时刻**：分母上的 $(k-1)! \times k$ 刚好等于 $k!$：

$$P_k(t) \cdot e^{\lambda t} = \frac{\lambda^k t^k}{(k-1)! \cdot k} = \frac{(\lambda t)^k}{k!}$$

把 $e^{\lambda t}$ 移到右边：

$$\boxed{P_k(t) = \frac{(\lambda t)^k e^{-\lambda t}}{k!}}$$

### 4.6 与核心公式完美闭环

在上面的公式中：
- $\lambda$ 仅仅是**单位时间内的发生率**
- $t$ 是**观测的总时间**
- $\lambda t$ 就是在整个观测时间 $t$ 内，事件发生的**总期望次数**（平均发生次数）

如果用一个新的符号 $\Lambda = \lambda t$ 来代表总期望次数，公式就变成：

$$P(X = k) = \frac{\Lambda^k e^{-\Lambda}}{k!}$$

这**完全对应**了最开始介绍的那个核心公式。

---

## 五、推导的总结：三条通往泊松分布的道路

| 道路 | 出发点 | 关键数学工具 | 揭示了什么 |
|------|--------|-------------|-----------|
| **二项分布取极限** | 离散的 $n$ 次试验 | 重要极限 $\lim(1-\frac{x}{n})^n = e^{-x}$ | 泊松分布是"大量微小概率事件叠加"的自然结果 |
| **泊松过程三前提 → 微分方程** | 连续的时间轴 + 三个公理 | 一阶线性 ODE + 积分因子法 | $e^{-\lambda}$ 来自微分方程的解，$k!$ 来自反复对 $t^{k-1}$ 积分 |
| **概率生成函数 (PGF)** | 母函数 $G(s) = E[s^X]$ | 指数函数的泰勒展开 | （高级方法，此处不展开） |

> **核心洞察**：指数 $e$ 和阶乘 $k!$ 不是人为规定的，而是在特定微观假设下（独立、平稳、无并发），宏观随机现象的**必然数学结果**。

---

## 六、泊松过程的三个前提的局限性

在现实中，以下情况会**违反**泊松过程的前提：

| 被违反的条件 | 现实例子 | 后果 |
|------------|---------|------|
| **独立增量** | 地震的余震（前一次地震增大了后续地震的概率） | 事件存在**时间聚集性**，需用 Hawkes 过程等自激模型 |
| **平稳性** | 交通流量的早晚高峰（$\lambda$ 随时间变化） | 需用**非齐次泊松过程** $\lambda(t)$ |
| **普通性** | 大型活动散场时人群同时涌出（瞬间大量事件并发） | 需用**复合泊松过程**或批到达模型 |

> **金融中的违反**：在 [VPIN](../Microstructure/PIN_VPIN.md) 模型中，订单到达率 $\mu$ 被假设为泊松过程。但在闪电崩盘等极端事件中，**独立性和平稳性同时被打破**——订单到达呈现爆发式聚集，这正是 VPIN 的体积时钟试图解决的问题。
