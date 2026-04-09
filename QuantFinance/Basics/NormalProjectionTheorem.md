# 正态投影定理：联合正态分布下的条件期望

纯数学视角的推导。将问题还原为：**在已知两个独立的正态随机变量经过线性组合生成了第三个随机变量的情况下，求其中一个初始变量关于这个新变量的条件期望（最小均方误差估计）。**

> 应用场景：[Kyle (1985) 模型推导](../Microstructure/KyleModel_Derivation.md)（做市商的贝叶斯更新步骤）

---

## 一、前置条件与变量定义

在概率空间中，有两个**独立的**连续随机变量 $V$ 和 $U$：

$$V \sim N(\mu_V, \Sigma_V)$$

$$U \sim N(0, \sigma_U^2)$$

**独立性**：$V$ 和 $U$ 相互独立，因此：

$$\text{Cov}(V, U) = 0$$

定义一个由 $V$ 和 $U$ **线性组合**而成的新随机变量 $Y$：

$$Y = \alpha + \beta V + U$$

其中 $\alpha$ 和 $\beta$ 为已知常数。

**目标**：计算在观测到 $Y$ 的取值后，$V$ 的条件期望 $E[V \mid Y]$。

---

## 二、正态投影定理 (Normal Projection Theorem)

### 定理陈述

因为 $V$ 和 $U$ 是正态分布且相互独立，它们的线性组合 $Y$ 也服从正态分布。更重要的是，$V$ 和 $Y$ 构成了**联合正态分布 (Joint Normal Distribution)**。

对于二维联合正态分布 $(V, Y)$，概率论中有一个非常重要的定理：

> $V$ 给定 $Y$ 的条件期望，恰好等于 $V$ 在 $Y$ 上的**最佳线性投影 (Best Linear Predictor)**。

$$\boxed{E[V \mid Y] = E[V] + \frac{\text{Cov}(V, Y)}{\text{Var}(Y)} \left(Y - E[Y]\right)}$$

### 公式结构的直觉

这个公式与一元线性回归 (OLS) 和卡尔曼滤波 (Kalman Filter) 的更新步具有完全相同的结构：

| 组成部分 | 公式中的位置 | 含义 |
|---------|-------------|------|
| **先验估计** | $E[V]$ | 没有观测到 $Y$ 之前，对 $V$ 的最佳猜测 |
| **新息 (Innovation)** | $Y - E[Y]$ | 观测值与预期观测值之间的偏差——"意料之外"的部分 |
| **增益系数 (Gain)** | $\dfrac{\text{Cov}(V, Y)}{\text{Var}(Y)}$ | 决定了将多少"偏差"转化为对 $V$ 的修正量 |

> **核心直觉**：条件期望 = 先验 + 增益 × 新息。观测到的 $Y$ 偏离了预期，我们按比例（增益系数）将这部分偏离折算为对 $V$ 的估计修正。增益系数本质上就是以 $Y$ 为自变量、$V$ 为因变量的**线性回归斜率**。

### 为什么联合正态分布下条件期望恰好是线性的？

这是正态分布的一个**极其特殊的性质**（非正态分布不一定成立）：

- 一般情况下，$E[V \mid Y]$ 可以是 $Y$ 的任意复杂函数
- 但当 $(V, Y)$ 服从联合正态分布时，$E[V \mid Y]$ **一定是 $Y$ 的线性函数**
- 这意味着最佳非线性预测 = 最佳线性预测，不需要更复杂的估计器

这个性质使得在正态假设下，许多原本困难的贝叶斯推断问题变得**可解析求解**。

---

## 三、逐步推导

为了将具体变量代入投影定理，我们需要分别计算 $E[V]$、$E[Y]$、$\text{Cov}(V, Y)$ 和 $\text{Var}(Y)$。

### 3.1 计算期望

已知：$E[V] = \mu_V$

根据期望的**线性性质**：

$$E[Y] = E[\alpha + \beta V + U] = \alpha + \beta E[V] + E[U]$$

因为 $E[U] = 0$，所以：

$$E[Y] = \alpha + \beta \mu_V$$

### 3.2 计算协方差 $\text{Cov}(V, Y)$

协方差算子具有**双线性 (Bilinearity)** 性质，可以展开：

$$\text{Cov}(V, Y) = \text{Cov}(V, \;\alpha + \beta V + U)$$

$$= \text{Cov}(V, \alpha) + \text{Cov}(V, \beta V) + \text{Cov}(V, U)$$

逐项分析：

| 项 | 计算 | 理由 |
|----|------|------|
| $\text{Cov}(V, \alpha)$ | $= 0$ | 常数的方差为 0，与任何变量的协方差也为 0 |
| $\text{Cov}(V, \beta V)$ | $= \beta \cdot \text{Cov}(V, V) = \beta \cdot \text{Var}(V) = \beta \Sigma_V$ | 提取常数 $\beta$，自协方差 = 方差 |
| $\text{Cov}(V, U)$ | $= 0$ | 已知 $V$ 和 $U$ 相互独立 |

三项相加：

$$\boxed{\text{Cov}(V, Y) = \beta \Sigma_V}$$

### 3.3 计算方差 $\text{Var}(Y)$

根据方差的性质，常数平移不改变方差。对于独立变量之和，和的方差等于方差之和：

$$\text{Var}(Y) = \text{Var}(\alpha + \beta V + U) = \text{Var}(\beta V + U)$$

$$= \text{Var}(\beta V) + \text{Var}(U) + 2\text{Cov}(\beta V, U)$$

逐项分析：

| 项 | 计算 | 理由 |
|----|------|------|
| $\text{Var}(\beta V)$ | $= \beta^2 \text{Var}(V) = \beta^2 \Sigma_V$ | 提取常数需**平方** |
| $\text{Var}(U)$ | $= \sigma_U^2$ | 直接由定义 |
| $2\text{Cov}(\beta V, U)$ | $= 2\beta \cdot \text{Cov}(V, U) = 0$ | $V$ 和 $U$ 独立 |

三项相加：

$$\boxed{\text{Var}(Y) = \beta^2 \Sigma_V + \sigma_U^2}$$

---

## 四、最终结果

将计算得到的统计量代回投影定理公式：

$$E[V \mid Y] = \mu_V + \frac{\beta \Sigma_V}{\beta^2 \Sigma_V + \sigma_U^2} \left(Y - (\alpha + \beta \mu_V)\right)$$

定义增益系数 $\lambda$：

$$\boxed{\lambda = \frac{\text{Cov}(V, Y)}{\text{Var}(Y)} = \frac{\beta \Sigma_V}{\beta^2 \Sigma_V + \sigma_U^2}}$$

则条件期望可以简写为一条**以 $Y$ 为自变量的直线方程**：

$$\boxed{E[V \mid Y] = \mu_V + \lambda \left(Y - E[Y]\right)}$$

---

## 五、增益系数 $\lambda$ 的纯数学分析

$$\lambda = \frac{\beta \Sigma_V}{\beta^2 \Sigma_V + \sigma_U^2}$$

这本质上是一个**信噪比 (Signal-to-Noise Ratio)** 的数学表达：

| 组成部分 | 含义 |
|---------|------|
| **分子** $\beta \Sigma_V$ | 目标变量 $V$ 在观测变量 $Y$ 中留下的"痕迹"强度 |
| **分母** $\beta^2 \Sigma_V + \sigma_U^2$ | 观测变量 $Y$ 的总波动程度（总方差），由 $V$ 贡献的有用波动 + $U$ 贡献的纯噪声 |

### 极端情况分析

**情况 1：噪声方差极大**（$\sigma_U^2 \to \infty$）

$$\lambda \to 0$$

观测到的 $Y$ 几乎全是噪声，$Y$ 的变化无法提供关于 $V$ 的有效信息。条件期望退化为先验期望：

$$E[V \mid Y] \to \mu_V$$

> 直觉：信号被噪音彻底淹没，观测等于没观测。

**情况 2：无噪声**（$\sigma_U^2 \to 0$）

$$\lambda \to \frac{\beta \Sigma_V}{\beta^2 \Sigma_V} = \frac{1}{\beta}$$

此时 $Y = \alpha + \beta V$（确定性线性关系），可以完美逆向解出：

$$V = \frac{Y - \alpha}{\beta}$$

条件期望等同于代数精确解。

> 直觉：没有噪音干扰时，一次观测就能完美推断 $V$。

**情况 3：目标变量无波动**（$\Sigma_V \to 0$）

$$\lambda \to 0$$

$V$ 退化为常数 $\mu_V$，没有什么可估计的，条件期望就是 $\mu_V$ 本身。

---

## 六、使用到的概率论性质汇总

本推导涉及的**核心概率论性质**一览：

| 性质 | 数学表达 | 适用条件 |
|------|---------|---------|
| 期望的线性性 | $E[aX + bY + c] = aE[X] + bE[Y] + c$ | 对**任意**随机变量成立 |
| 协方差与常数 | $\text{Cov}(X, c) = 0$ | $c$ 为常数 |
| 协方差的双线性 | $\text{Cov}(X, aY + bZ) = a\text{Cov}(X,Y) + b\text{Cov}(X,Z)$ | 对任意随机变量成立 |
| 自协方差 = 方差 | $\text{Cov}(X, X) = \text{Var}(X)$ | 定义 |
| 方差提取常数 | $\text{Var}(aX) = a^2 \text{Var}(X)$ | 注意平方 |
| 独立变量的方差加法 | $\text{Var}(X + Y) = \text{Var}(X) + \text{Var}(Y)$ | 仅当 $X, Y$ **独立**时成立 |
| 独立性 → 零协方差 | $X \perp Y \implies \text{Cov}(X, Y) = 0$ | 反之不一定成立（零协方差 $\not\Rightarrow$ 独立） |
| 联合正态下条件期望线性 | $E[V \mid Y]$ 是 $Y$ 的线性函数 | **仅当 $(V,Y)$ 联合正态时**成立 |

> **最后一条是最核心的**：正态分布的特殊性使得条件期望恰好等于线性投影。对于非正态分布，$E[V \mid Y]$ 可能是 $Y$ 的高度非线性函数，投影定理就不再适用，需要用更复杂的贝叶斯方法。
