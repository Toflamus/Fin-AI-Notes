# 多元普通最小二乘法 (Multivariate OLS)

多元普通最小二乘法（Multivariate Ordinary Least Squares, 多元 OLS）是统计学和计量经济学中最核心的参数估计方法之一。它的主要目的是通过多个自变量（特征）的线性组合来拟合因变量（目标），并通过**最小化误差平方和**来寻找最佳的拟合参数。

> 相关：[正态投影定理](NormalProjectionTheorem.md)（OLS 的闭式解与正态投影的数学结构完全一致）| [t 统计量](T_Statistic.md)（回归系数的显著性检验）| [数据处理工作流中的中性化](../DataProcessing/Workflow.md)（截面回归取残差本质上就是 OLS）

---

## 1. 数学模型与矩阵表达

在多元线性回归中，假设因变量 $Y$ 受到 $k$ 个自变量 $X_1, X_2, \ldots, X_k$ 的影响。对于包含 $n$ 个观测样本的数据集，第 $i$ 个样本的线性模型表示为：

$$y_i = \beta_0 + \beta_1 x_{i1} + \beta_2 x_{i2} + \cdots + \beta_k x_{ik} + \epsilon_i$$

其中：
- $\beta_0$ 是截距项
- $\beta_1, \ldots, \beta_k$ 是偏回归系数
- $\epsilon_i$ 是随机误差项（残差）

为了进行全局优化和推导，将其转换为**矩阵形式**：

$$Y = X\beta + \epsilon$$

各矩阵的维度与定义：

| 符号 | 维度 | 定义 |
|------|------|------|
| $Y$（因变量向量） | $n \times 1$ | 列向量，包含 $n$ 个观测值 |
| $X$（设计矩阵） | $n \times (k+1)$ | 第一列全设为 1（截距项），其余列为自变量 |
| $\beta$（参数向量） | $(k+1) \times 1$ | 包含 $\beta_0, \beta_1, \ldots, \beta_k$，需要估计 |
| $\epsilon$（误差向量） | $n \times 1$ | 随机误差 |

---

## 2. 优化目标与参数推导

### 2.1 优化目标

OLS 的核心思想：寻找一组参数估计值 $\hat{\beta}$，使得模型预测值 $\hat{Y} = X\hat{\beta}$ 与真实观测值 $Y$ 之间的**残差平方和 (Sum of Squared Residuals, SSR)** 最小。

残差向量为 $\hat{\epsilon} = Y - \hat{Y} = Y - X\hat{\beta}$。

目标函数 $SSR$ 可以写为残差向量的内积：

$$SSR(\hat{\beta}) = \hat{\epsilon}^T \hat{\epsilon} = (Y - X\hat{\beta})^T (Y - X\hat{\beta})$$

### 2.2 展开二次型

$$SSR(\hat{\beta}) = Y^T Y - Y^T X \hat{\beta} - \hat{\beta}^T X^T Y + \hat{\beta}^T X^T X \hat{\beta}$$

因为 $Y^T X \hat{\beta}$ 是一个标量（$1 \times 1$ 矩阵），其转置等于自身，即 $(Y^T X \hat{\beta})^T = \hat{\beta}^T X^T Y$，所以方程可化简为：

$$SSR(\hat{\beta}) = Y^T Y - 2\hat{\beta}^T X^T Y + \hat{\beta}^T X^T X \hat{\beta}$$

### 2.3 求导令梯度为零

为了求极小值，对参数向量 $\hat{\beta}$ 求偏导，并令梯度为零。

需要用到的矩阵微分法则：
- $\frac{\partial}{\partial \beta}(a^T \beta) = a$（线性项的导数）
- $\frac{\partial}{\partial \beta}(\beta^T A \beta) = 2A\beta$（二次型的导数，当 $A$ 对称时）

应用到 SSR：

$$\frac{\partial SSR(\hat{\beta})}{\partial \hat{\beta}} = -2X^T Y + 2X^T X \hat{\beta} = 0$$

### 2.4 正规方程与闭式解

由此得到著名的**正规方程 (Normal Equation)**：

$$\boxed{X^T X \hat{\beta} = X^T Y}$$

如果矩阵 $X^T X$ 是**非奇异的**（即可逆的，意味着特征之间不存在完全多重共线性），两边同乘 $(X^T X)^{-1}$，即可得到 OLS 估计量的**闭式解（解析解）**：

$$\boxed{\hat{\beta} = (X^T X)^{-1} X^T Y}$$

> **与正态投影定理的联系**：比较 [正态投影定理](NormalProjectionTheorem.md) 中的增益系数 $\lambda = \frac{\text{Cov}(V,Y)}{\text{Var}(Y)}$，它与 OLS 的 $\hat{\beta} = (X^TX)^{-1}X^TY$ 在数学结构上完全一致——都是"协方差除以方差"的矩阵推广。在联合正态分布下，条件期望恰好等于 OLS 投影。

---

## 3. 高斯-马尔可夫假设 (Gauss-Markov Assumptions)

为了使 $\hat{\beta}$ 具有良好的统计学性质，多元 OLS 依赖于以下核心假设：

### 假设 1：线性于参数

模型 $Y = X\beta + \epsilon$ 在参数 $\beta$ 上是线性的。

> 注意：**自变量可以是非线性的**（如 $X_2 = X_1^2$），只要参数 $\beta$ 是线性出现的即可。

### 假设 2：随机抽样

样本 $(Y_i, X_i)$ 是从总体中独立同分布（i.i.d.）抽取的。

### 假设 3：不存在完全多重共线性

设计矩阵 $X$ 必须是**满列秩**的，即：

$$\text{Rank}(X) = k + 1$$

如果特征之间存在完美的线性关系（如 $X_3 = 2X_1 + 3X_2$），$X^TX$ 将**不可逆**，导致无法求解 $\hat{\beta}$。

> 这正是 [ZCA 正交化](../DataProcessing/FeatureOptimization.md) 和 [PCA 降维](../DataProcessing/Covariance_PCA_ZCA_Math.md) 要解决的问题——消除多重共线性使得回归可行。

### 假设 4：零条件均值（外生性假设）

给定解释变量 $X$，误差项的条件期望为零：

$$E(\epsilon \mid X) = 0$$

这意味着**特征 $X$ 包含的信息不能预测误差**。如果违反（如遗漏了重要变量），OLS 估计将产生偏差。

### 假设 5：同方差且无自相关（球形扰动假设）

- **同方差性**：$\text{Var}(\epsilon_i \mid X) = \sigma^2$（所有样本误差的方差恒定）
- **无自相关**：$\text{Cov}(\epsilon_i, \epsilon_j \mid X) = 0, \; \forall i \neq j$

用矩阵表示：

$$\text{Var}(\epsilon \mid X) = \sigma^2 I$$

其中 $I$ 是 $n \times n$ 的单位矩阵。

> 如果违反同方差假设（如金融收益率数据常见的异方差），需要使用**加权最小二乘法 (WLS)** 或 **White 稳健标准误**进行修正。

---

## 4. 估计量的统计性质

如果上述假设成立，根据**高斯-马尔可夫定理 (Gauss-Markov Theorem)**，多元 OLS 估计量 $\hat{\beta}$ 是 **BLUE (Best Linear Unbiased Estimator)**，即"最佳线性无偏估计量"。

### 4.1 无偏性 (Unbiasedness)

$$E[\hat{\beta} \mid X] = E[(X^TX)^{-1}X^T(X\beta + \epsilon) \mid X]$$

$$= (X^TX)^{-1}X^TX\beta + (X^TX)^{-1}X^T E[\epsilon \mid X]$$

$$= \beta + (X^TX)^{-1}X^T \cdot 0$$

$$\boxed{E[\hat{\beta} \mid X] = \beta}$$

> **无偏**意味着：如果你重复抽样无数次，$\hat{\beta}$ 的平均值恰好等于真实的 $\beta$。单次估计可能有偏差，但不存在系统性的偏移方向。

### 4.2 最佳性 (Best / Minimum Variance)

在所有**线性**和**无偏**的估计量中，OLS 估计量具有**最小的方差**。

其协方差矩阵为：

$$\boxed{\text{Var}(\hat{\beta} \mid X) = \sigma^2 (X^TX)^{-1}}$$

> **直觉**：$(X^TX)^{-1}$ 越"大"（即特征之间的信息越少或共线性越强），$\hat{\beta}$ 的估计越不稳定。这就是为什么多重共线性是回归分析的大敌——它不影响无偏性，但严重膨胀方差。

---

## 5. OLS 的几何解释：正交投影

从线性代数和几何的角度来看，多元 OLS 本质上是一个**正交投影 (Orthogonal Projection)** 问题。

### 几何图景

- 向量 $Y$ 存在于一个 $n$ 维空间中
- 矩阵 $X$ 的列向量张成了一个 $k+1$ 维的**子空间（Column Space of $X$）**
- 因为误差 $\epsilon$ 的存在，$Y$ 通常**并不在**这个子空间内
- OLS 的目标是寻找一个在这个子空间内的向量 $\hat{Y} = X\hat{\beta}$，使得它到 $Y$ 的**欧氏距离最短**

### 正交条件

几何上，最短距离的点 $\hat{Y}$ 就是 $Y$ 在 $X$ 的列空间上的**正交投影**。

这意味着残差向量 $\hat{\epsilon} = Y - \hat{Y}$ 必须**垂直于** $X$ 的列空间中的任意向量：

$$X^T \hat{\epsilon} = 0$$

展开后：

$$X^T(Y - X\hat{\beta}) = 0$$

$$X^TX\hat{\beta} = X^TY$$

这就是正规方程！

> **OLS 的几何本质**：在所有可能的线性组合 $X\beta$ 中，找到那个离 $Y$ 最近（欧氏距离最短）的点。这个最近点就是 $Y$ 在 $X$ 列空间上的正交投影。残差向量垂直于拟合平面——这就是"最小二乘"的几何含义。

### 投影矩阵 (Hat Matrix)

定义投影矩阵 $H$（也称帽子矩阵，因为它把 $Y$ "戴上了帽子" 变成 $\hat{Y}$）：

$$H = X(X^TX)^{-1}X^T$$

$$\hat{Y} = HY$$

$H$ 的性质：
- **幂等性**：$H^2 = H$（投影两次等于投影一次）
- **对称性**：$H^T = H$
- **迹**：$\text{tr}(H) = k + 1$（等于参数个数）

残差向量可以写为：

$$\hat{\epsilon} = (I - H)Y$$

其中 $M = I - H$ 也是一个投影矩阵，将 $Y$ 投影到 $X$ 列空间的**正交补空间**上。

> **与因子中性化的联系**：[数据处理工作流](../DataProcessing/Workflow.md) 中的"中性化"步骤——对因子值关于市值和行业哑变量做截面回归取残差——数学上就是用 $M = I - H$ 将因子投影到市值/行业子空间的正交补上，提取纯 Alpha。
