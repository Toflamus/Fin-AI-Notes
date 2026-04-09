# 正态分布与 t 分布

两种最核心的连续概率分布。形式上都呈钟形对称结构，但在应用场景和数学定义上有严格区分。

> 相关：[t 统计量](T_Statistic.md) | [t 与 IR 的数学推导](T_IR_Derivation.md)

---

## 一、正态分布 (Normal Distribution)

又称**高斯分布 (Gaussian Distribution)**。当一个随机变量的变异由大量微小、独立的随机因素叠加而成时，该变量通常服从正态分布（**中心极限定理**）。

记号：

$$X \sim \mathcal{N}(\mu, \sigma^2)$$

### 1. 概率密度函数 (PDF)

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

- $\mu$（均值）：分布的中心位置
- $\sigma$（标准差）：分布的宽度
  - $\sigma$ 越大 → 曲线越扁平
  - $\sigma$ 越小 → 曲线越陡峭

当 $\mu = 0, \sigma = 1$ 时为**标准正态分布**：$Z \sim \mathcal{N}(0, 1)$

### 2. 累计分布函数 (CDF)

$$F(x) = \frac{1}{\sigma\sqrt{2\pi}} \int_{-\infty}^{x} \exp\left(-\frac{(t-\mu)^2}{2\sigma^2}\right) dt$$

由于这个积分**没有闭式解**（无法用初等函数表达），通常用**误差函数 (Error Function, $\text{erf}$)** 表示：

$$F(x) = \frac{1}{2} \left[ 1 + \text{erf}\left( \frac{x-\mu}{\sigma\sqrt{2}} \right) \right]$$

实际应用中通过查表或数值算法（如 `scipy.stats.norm.cdf`）获得。

---

## 二、t 分布 (Student's t-Distribution)

由 William Sealy Gosset 以笔名 **"Student"** 发表。主要用于：

- **总体方差未知**
- **样本量较小**（通常 $n < 30$）

用样本标准差去估计总体标准差时对均值的推断。

### 1. 概率密度函数 (PDF)

形态完全由一个参数决定：**自由度 (Degrees of Freedom, $\nu$ 或 $df$)**。单样本检验中 $\nu = n - 1$。

$$f(t) = \frac{\Gamma\left(\frac{\nu+1}{2}\right)}{\sqrt{\nu\pi} \, \Gamma\left(\frac{\nu}{2}\right)} \left(1 + \frac{t^2}{\nu}\right)^{-\frac{\nu+1}{2}}$$

其中 $\Gamma(\cdot)$ 是 Gamma 函数。

### 2. 与正态分布的核心区别：厚尾效应 (Heavy Tails)

由于使用样本方差（本身具有随机性）替代未知的总体方差，引入了额外的不确定性。反映在图像上：

| 特征 | t 分布 | 标准正态 |
|------|--------|----------|
| 中间峰值 | **稍低** | 较高 |
| 两端尾部 | **较厚** | 较薄 |
| 极端值出现概率 | **更高** | 较低 |

**极限性质**：当 $\nu \to \infty$（样本量趋于无穷大），t 分布的 PDF 严格收敛于标准正态分布的 PDF。

> **直觉**：自由度小 = 样本少 = 对真实方差的估计不靠谱 = 必须为"可能算错"留出更宽的尾部容差。样本越大，估计越准，t 分布越接近正态。

### 3. 累计分布函数 (CDF)

$$F(t) = \int_{-\infty}^{t} f(u) \, du$$

同样没有简单的初等函数表达。可用**正则化不完全 Beta 函数 (Regularized Incomplete Beta Function, $I_x(a,b)$)** 精确表达：

$$F(t) = \begin{cases} \dfrac{1}{2} I_{x(t)}\left(\dfrac{\nu}{2}, \dfrac{1}{2}\right) & \text{if } t < 0 \\[8pt] 1 - \dfrac{1}{2} I_{x(t)}\left(\dfrac{\nu}{2}, \dfrac{1}{2}\right) & \text{if } t \ge 0 \end{cases}$$

其中 $x(t) = \dfrac{\nu}{t^2 + \nu}$。统计软件（如 `scipy.stats.t.cdf`）会直接处理。

---

## 三、对比速查

| 特性 | 正态分布 $\mathcal{N}(\mu, \sigma^2)$ | t 分布 $t(\nu)$ |
|------|---------------------------------------|------------------|
| 参数 | 均值 $\mu$、标准差 $\sigma$ | 自由度 $\nu$ |
| 适用 | 总体方差**已知**，或大样本 | 总体方差**未知**，小样本 |
| 形态 | 钟形，由 $\sigma$ 控制宽度 | 钟形，但**尾部更厚** |
| 极端值概率 | 较低 | **较高** |
| 极限 | — | $\nu \to \infty$ 时收敛于 $\mathcal{N}(0,1)$ |
| CDF 闭式解 | 无（用 erf 表示） | 无（用正则化不完全 Beta 函数表示） |

### 量化金融的启示

- 金融数据往往呈现"肥尾"，与 t 分布的形态吻合度比正态分布更高
- 用正态假设做风险管理（如标准 VaR）会**严重低估**极端损失的概率
- 这也是为什么 [尾部风险](PortfolioOptimization.md) 与 CVaR 在金融实务中比 VaR 更受重视
