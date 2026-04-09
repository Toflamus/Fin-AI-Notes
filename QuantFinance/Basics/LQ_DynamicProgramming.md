# 线性二次型动态规划 (LQ Dynamic Programming)

纯数学视角：为什么在 Kyle 模型的序列拍卖中，内幕交易者的最优策略是线性的、价值函数是二次的？这源于**线性二次型 (Linear-Quadratic, LQ)** 问题的经典结构。

> 应用场景：[Kyle 模型完整推导](../Microstructure/KyleModel_Derivation.md)（序列拍卖与连续拍卖）

---

## 一、马尔可夫决策过程的三要素

将内幕交易者的决策抽象为**随机最优控制 (Stochastic Optimal Control)** 问题。

### 1.1 状态变量 (State Variable)

$$S_n = v - p_{n-1}$$

"剩余获利空间"。由于 $v$ 是常数（已被内幕交易者观测），系统的随机性全来自 $p_{n-1}$ 的演变。

### 1.2 控制变量 (Control Variable)

$$\Delta x_n$$

内幕交易者决定这一期的交易量。

### 1.3 状态转移方程 (State Transition)

根据做市商的线性定价规则 $p_n = p_{n-1} + \lambda_n(\Delta x_n + \Delta u_n)$：

$$S_{n+1} = S_n - \lambda_n \Delta x_n - \lambda_n \Delta u_n$$

这是一个**线性**的状态转移方程（$S_{n+1}$ 是 $S_n$ 和 $\Delta x_n$ 的线性组合，加上随机干扰 $\Delta u_n$）。

---

## 二、贝尔曼方程 (Bellman Equation)

设 $W_n(S_n)$ 为从第 $n$ 期开始的**价值函数 (Value Function)**——在已知当前获利空间为 $S_n$ 时的最大期望收益：

$$W_n(S_n) = \max_{\Delta x_n} E\left[\underbrace{(v - p_n)\Delta x_n}_{\text{本期利润}} + \underbrace{W_{n+1}(S_{n+1})}_{\text{未来期望总利润}} \;\middle|\; S_n\right]$$

代入 $v - p_n = S_n - \lambda_n(\Delta x_n + \Delta u_n)$：

$$W_n(S_n) = \max_{\Delta x_n} E\left[(S_n - \lambda_n \Delta x_n - \lambda_n \Delta u_n)\Delta x_n + W_{n+1}(S_n - \lambda_n \Delta x_n - \lambda_n \Delta u_n)\right]$$

---

## 三、二次型价值函数的来源：倒向归纳

### 3.1 最后一期（第 N 期）：二次项的"种子"

在最后一期，没有未来，目标纯粹是最大化当期利润：

$$\pi_N = (v - p_N)\Delta x_N$$

代入定价规则，取关于 $\Delta u_N$ 的期望（$E[\Delta u_N] = 0$）：

$$E[\pi_N \mid v, p_{N-1}] = (v - p_{N-1})\Delta x_N - \lambda_N (\Delta x_N)^2$$

> **关键观察**：利润 = "价格差 $\times$ 交易量"。由于价格差本身包含了交易量的一次项（因为交易会影响价格），两者相乘便产生了**第一个二次项** $-\lambda(\Delta x)^2$。

对 $\Delta x_N$ 求导令其为 0：

$$\frac{dE[\pi_N]}{d\Delta x_N} = (v - p_{N-1}) - 2\lambda_N \Delta x_N = 0$$

$$\implies \Delta x_N^* = \frac{1}{2\lambda_N}(v - p_{N-1})$$

将最优 $\Delta x_N^*$ 代回期望利润：

$$E[\pi_N^*] = \frac{(v - p_{N-1})^2}{2\lambda_N} - \frac{(v - p_{N-1})^2}{4\lambda_N} = \frac{1}{4\lambda_N}(v - p_{N-1})^2$$

**结论**：最后一期的最优利润确实是状态变量 $(v - p_{N-1})$ 的**二次函数**。

令 $\alpha_{N-1} = \frac{1}{4\lambda_N}$，则：

$$W_N(S_{N-1}) = \alpha_{N-1} \cdot S_{N-1}^2$$

### 3.2 倒数第二期（第 N-1 期）：二次结构的传递

在第 $N-1$ 期，目标变为：

$$\max_{\Delta x_{N-1}} E\left[\underbrace{(v - p_{N-1})\Delta x_{N-1}}_{\text{本期利润}} + \underbrace{\alpha_{N-1}(v - p_{N-1})^2}_{\text{第 N 期最优利润}}\right]$$

代入价格转移方程 $v - p_{N-1} = (v - p_{N-2}) - \lambda_{N-1}(\Delta x_{N-1} + \Delta u_{N-1})$：

- **本期利润项**展开后包含 $\Delta x_{N-1}$ 的一次项和二次项
- **未来利润项**由于是 $(v - p_{N-1})$ 的平方，展开后会通过**平方运算保持二次形式**并向前传导

整个目标函数仍然是关于 $\Delta x_{N-1}$ 的**二次多项式**，且系数包含 $(v - p_{N-2})$。

### 3.3 数学归纳：结构保持性

通过上述两步可以归纳出核心定理：

> **如果第 $n$ 期的价值函数 $W_n(S_n)$ 是 $S_n^2$ 的形式，那么第 $n-1$ 期的价值函数 $W_{n-1}(S_{n-1})$ 也一定是 $S_{n-1}^2$ 的形式。**

**原因**：

1. **状态转移是线性的**：$S_n$ 是 $S_{n-1}$ 和 $\Delta x_{n-1}$ 的线性组合
2. **本期收益是二次的**：$(S_n - \lambda \Delta x)\Delta x$ 展开后包含 $(\Delta x)^2$ 项
3. **二次型在线性算子下的复合仍是二次型**：$(S_{n-1} - \text{linear term})^2$ 展开后仍是 $S_{n-1}^2$ 的形式

因此每一期的价值函数都维持：

$$W_n(S_n) = \alpha_n S_n^2 + \delta_n$$

其中 $\delta_n$ 是由噪声方差 $\sigma_u^2$ 产生的常数项，**不影响**关于 $\Delta x$ 的最优化求解。

---

## 四、为什么最优策略一定是线性的？

这是 LQ 控制理论的**核心结论**：

| 条件 | 本模型中的体现 |
|------|--------------|
| 状态转移方程是**线性**的 | $S_{n+1} = S_n - \lambda_n \Delta x_n - \lambda_n \Delta u_n$ |
| 收益函数是**二次**的 | 利润 = 一次项 × 控制量 → 产生二次项 |
| 噪声是**正态**的 | $\Delta u_n \sim N(0, \sigma_u^2 \Delta t_n)$ |

在 LQ 问题中，对贝尔曼方程求一阶导数条件 (F.O.C.)：

$$\frac{\partial}{\partial \Delta x_n} E[\ldots] = 0$$

由于方程中只有 $S_n$ 的一次项和 $\Delta x_n$ 的一次项（求导后二次项变为一次项），解出的最优控制量 $\Delta x_n^*$ **必然与 $S_n$ 成正比**：

$$\Delta x_n^* = \text{coefficient}_n \cdot S_n$$

这就是 $\Delta x_n = \beta_n(v - p_{n-1})\Delta t_n$ 的数学来源。$\beta_n \Delta t_n$ 就是通过求导解出的**最优比例系数**。

> **核心定理**：线性转移 + 二次收益 + 正态噪声 → 最优策略是线性的 + 价值函数是二次的。这是 LQ 控制论中最优美的结论之一，也是 Kyle 模型可以获得解析解的根本原因。

---

## 五、动态规划中的"现时 vs 未来"权衡

内幕交易者是**前瞻性的 (Forward-looking)**：

- 如果本期交易太多（$\beta_n$ 很大）：
  - 本期利润**高**
  - 但下一期的状态 $S_{n+1}$ 迅速缩小（价格 $p$ 快速向 $v$ 靠拢）
  - $W_{n+1}$ 是关于 $S_{n+1}^2$ 的函数 → 未来获利能力**受损**

- 如果本期交易太少（$\beta_n$ 很小）：
  - 本期利润**低**
  - 但保留了大量的未来获利空间

最优的 $\beta_n$ 是在**"现时利益"与"未来获利潜力"**之间取得数学上的均衡点（一阶条件为零的那个点）。

> **直觉**：内幕交易者就像一个拥有有限燃料的火箭——每喷一点燃料（交易一点），就推进一点（赚一点利润），但总燃料（信息优势 $S_n$）是有限的。最优策略是**匀速消耗**，而不是一开始就把燃料烧光。

---

## 六、递推闭环：三方互锁的差分方程系统

序列拍卖中的动态规划形成了一个**三方递推闭环**：

```
┌─────────────────────────────────────────────┐
│  内幕交易者看"未来"：根据 α_n 决定 β_n       │
│        ↓                                     │
│  做市商看"当前"：根据 β_n 决定 λ_n            │
│        ↓                                     │
│  市场结果：根据 β_n, λ_n 决定新的 Σ_n         │
│        ↓                                     │
│  循环：进入第 n-1 期的计算（后向归纳）          │
└─────────────────────────────────────────────┘
```

$$\beta_n \Delta t_n = \frac{1 - 2\alpha_n \lambda_n}{2\lambda_n(1 - \alpha_n \lambda_n)}$$

$$\lambda_n = \frac{\beta_n \Sigma_{n-1}}{\beta_n^2 \Sigma_{n-1} \Delta t_n + \sigma_u^2}$$

$$\Sigma_n = (1 - \beta_n \lambda_n \Delta t_n)\Sigma_{n-1}$$

当 $N \to \infty$（$\Delta t \to 0$）时，这组差分方程变成一组**联立的常微分方程**，最终解出 Kyle 连续时间均衡的解析解。
