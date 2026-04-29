# 因子中性化与移动最小二乘（MLS）

> **核心命题**：截面中性化的本质是**投影**——把原始因子投影到风险空间的补空间中。
>
> - **OLS** = 全局投影（一个 $\beta$ 通吃全市场）
> - **WLS** = 加权全局投影（按可信度加权）
> - **MLS** = 局部投影的滑动合成（逐点拟合 + 核函数平滑）
>
> 本文从函数逼近论的角度系统推导 MLS 中性化算子，并与 OLS / WLS 做严格对比。
>
> **阅读路径**：§0–§1 是概念基础（"为什么这些叫风险"、"为什么主动放弃 Beta"）；§2–§7 是从 OLS 到 MLS 的完整数学；§8–§9 是工程实现与高频因子应用。

---

## 0. 概念基础：为什么市值 / 行业 / 波动率被称为"风险"

把"市值"、"行业"或"波动率"这些因子称为 **风险（Risk Factors）**，而非简单的"自变量"，背后蕴含着现代投资组合理论（MPT）和多因子定价模型（如 Barra 模型）的核心逻辑。可以从三个维度来理解。

### 0.1 概念层：风险 = "同涨同跌"的共同来源

在金融学中，**风险（Risk）** 的本质是**不确定性**，而这种不确定性通常分为两类：
- **特异性风险（Idiosyncratic Risk / Alpha）**：单只股票由于自身基本面导致的波动。
- **系统性风险（Systematic Risk / Factor Risk）**：一群股票由于具有某种共同特征而产生的共同波动。

**为什么对数市值是风险？**
如果你构建了一个因子，但这个因子在小盘股上表现极好，在大盘股上表现极差。那么当你使用这个因子建仓时，你的收益将高度依赖于"小盘股整体是否走强"。
- 如果市场风格切换到大盘股，你的组合会集体溃败。
- 这种因为"某种共同特征"导致的集体回撤压力，就是**风险暴露（Risk Exposure）**。

### 0.2 数学层：解释方差的"主成分"

从物理学或统计学的角度看，中性化其实是在做**信号分解**。

假设股票收益率矩阵为 $R$，可以将其分解为：
$$R = \underbrace{\sum_{k} \beta_k F_k}_{\text{风险贡献}} + \underbrace{\alpha}_{\text{超额收益}} + \epsilon$$

- **风险因子 ($F_k$)**：指那些能够解释市场上大部分股票**共性收益**的变量。对数市值（Size）被证明是解释股票横截面方差最强的变量之一。
- **数学上的"意义"**：如果不剔除市值的影响，Alpha 信号其实是混杂了市值因子的"杂质"。在回归方程中，这些共性变量吸走了大部分的解释力（$R^2$），剩下的残差 $\epsilon$ 才是真正属于策略的、与已知公共因子无关的**纯净信号**。

### 0.3 工程层：为了获得"风险中性"的 Alpha

在量化交易（尤其是高频 LOB / OFI 领域）中，目标是寻找 **Alpha（绝对收益）**，而不是 **Beta（随波逐流的收益）**。

| 特征 | 为什么被视为"风险" | 中性化的目的 |
| :--- | :--- | :--- |
| **对数市值** | 规模小的股票通常流动性差、风险溢价高。 | 确保信号在不同体量的公司间是通用的。 |
| **行业因子** | 同一行业的股票会受宏观政策、商品价格共同影响。 | 防止策略变成"赌某个行业会涨"。 |
| **波动率** | 高波动股票在市场恐慌时具有趋同的下行压力。 | 消除由于风险偏好变化带来的虚假信号。 |

**为什么叫"中性化"？**
"中性化"这个词本身就代表了 **"免疫"**。
通过 MLS 或 OLS 处理后，你的因子值与市值之间的相关性变为 0。这意味着无论市场是大盘股涨还是小盘股涨，你的因子理论上都不会受到冲击。你实现了一种**对风险的免疫**，因此这些被剔除的变量就被统一称为"风险因子"。

### 0.4 类比：实验室控制变量

为方便理解，可以想象在测试一种**新型碳纤维材料的导电性能**：
- **实验目标**：找出 PANI 改性工艺（Alpha）对导电性的提升。
- **"风险因子"**：环境湿度、环境温度、测试压力。
- **为什么要"中性化"？**
  如果实验数据发现改性后导电性变好了，但实验时的环境温度也刚好升高了。你无法确定性能提升是因为"工艺好"（Alpha），还是因为"温度高"（系统性风险）。
- **MLS 的角色**：由于温度对性能的影响可能不是线性的（比如在某个温度区间影响剧烈），需要用 MLS 这种局部回归方法，把温度这个"风险因素"的影响精确地从实验数据中扣除，剩下的才是改性工艺的纯贡献。

### 0.5 小结

在量化金融里，**"风险"不代表"坏事"，而代表"公认的、已知的波动来源"**。

把市值称为风险，是因为我们不想赚"因为股票市值小而产生的风险溢价"，要赚的是"基于微观结构（OFI）捕捉到的定价偏差"。

---

## 1. 战略意图：为什么主动放弃赚 Beta 的钱

中性化确实意味着主动放弃了赚取"大盘涨跌"或"风格切换"那部分钱的机会。在量化金融中，这被称为 **Alpha（阿尔法）与 Beta（贝塔）的分离**。下面剖析为什么要"自废武功"。

### 1.1 核心矛盾：辛苦钱 vs 顺风钱

量化投资将收益分解为：
$$R_i = \underbrace{\beta \cdot R_{market}}_{\text{Beta (顺风钱)}} + \underbrace{\alpha}_{\text{Alpha (辛苦钱)}} + \epsilon$$

- **Beta（风险溢价）**：这是大盘给的"低保"。如果行情好，只要买股票就能赚钱。
- **Alpha（超额收益）**：这是通过深入研究 LOB（限价单簿）、OFI（订单流不平衡）等微观数据，发现别人没看到的定价错误而赚到的钱。

**如果不进行中性化**：
当一个 OFI 信号买入股票时，如果此时大盘暴涨，赚了很多钱。但你无法分辨：这钱是因为 OFI 信号准，还是仅仅因为大盘在涨？
> **后果**：一旦大盘掉头向下，因子即便再准，账户也会亏损。这对于追求稳健收益的对冲基金来说是不可接受的。

### 1.2 为什么要"主动放弃"大盘的钱

既然大盘能带我飞，为什么非要去掉它？主要有三个原因。

#### A. 策略的可解释性与"纯度"
作为研究者，你需要知道因子是否真的有**预测力**。
- 如果不去风险，其实是在做一个"预测大盘"的策略。
- 如果去了风险，因子依然能赚钱，说明真正掌握了市场的微观规律。这就像在实验室里排除所有干扰项，只观察单一变量的反应。

#### B. 风险控制与"回撤管理"
量化策略往往带有高杠杆。
- **不中性化**：组合与大盘高度相关（相关系数可能 0.8 以上）。大盘跌 10%，你可能亏 12%。
- **中性化后**：组合与大盘相关性接近 0。大盘跌 10%，策略可能还能涨 1%，或只亏 0.1%。这种**独立于市场环境**的赚钱能力（绝对收益），在金融市场上非常昂贵。

#### C. 容量与杠杆的置换
因为中性化策略的波动率极低（去掉了剧烈的大盘波动），可以通过**加杠杆**来放大收益。
- 赚 2% 的 Beta 收益，可能要承担 20% 的市场波动风险。
- 赚 0.5% 的纯 Alpha 收益，通过 4 倍杠杆变成 2%，而此时承担的风险远小于市场整体风险。

### 1.3 中性化前后的策略对比

| 维度 | **非中性化 (Long-only)** | **中性化 (Market Neutral)** |
| :--- | :--- | :--- |
| **收益来源** | 大盘涨幅 + 因子选股能力 | 纯粹的选股能力 (Alpha) |
| **市场表现** | 牛市大赚，熊市必亏 | 牛市熊市均可获利 |
| **核心风险** | 市场系统性风险 (Beta) | 因子失效风险 |
| **适用人群** | 公募基金、长期持有者 | 对冲基金、高频交易者 |

### 1.4 微观结构因子的特殊性

回到 LOB 和 OFI 研究。这些微观因子的预测周期通常很短（几秒到几分钟）。在这么短的时间内，大盘的剧烈波动（比如突然的政策新闻）对因子来说就是巨大的**随机噪声**。

通过 MLS 中性化得到的因子残差 $\epsilon$ 会呈现出更好的**统计特性**：
1. **IC（信息系数）更稳定**：不会因为今天大盘是涨是跌而大幅波动。
2. **胜率更真实**：反映的是对微观买卖盘力量对比的判断准确度。
3. **夏普比率（Sharpe Ratio）更高**：这是量化评价体系的核心——用单位风险换取的收益。

### 1.5 小结：放弃为了掌控

**去掉风险确实意味着赚不到大盘的钱，但这正是量化中性策略的初衷。** 如果既想赚大盘的钱，又想赚 Alpha 的钱，通常的做法是：**构建一个中性化因子池，但在执行层不完全对冲市场头寸。** 但在研究阶段，如果不做中性化，你永远无法确认你的 OFI 因子到底是真金白银，还是只是某种风格因子的影子。

这就像研究碳纤维 PANI 改性时，如果不控制环境温度（风险），永远不知道导电性能的提升是源于工艺，还是仅仅因为实验室今天暖气开得足。这种"放弃"是为了获得更高维度的掌控感。

---

## 2. 数学目标：中性化作为投影算子

在多因子选股中，原始 Alpha 因子 $\mathbf{y}$ 往往与已知的风格/行业风险 $\mathbf{x}$（市值、Beta、行业哑变量等）相关。如果不剥离这些风险暴露：
- 回测里的"超额收益"实际上来自**风格 Beta**而非 Alpha。
- 组合在风格切换（如小盘→大盘）时承受不可控波动。
- 多因子合成时，风险之间的协方差掩盖了真实信号。

中性化的目标：构造算子 $\mathcal{N}$，使
$$\mathbf{y}_{\text{neutral}} = \mathcal{N}\mathbf{y}, \qquad \mathbf{y}_{\text{neutral}} \perp \mathbf{x}$$

正交性可以是**全局正交**（$\langle \mathbf{y}_{\text{neutral}}, \mathbf{x} \rangle = 0$，OLS 满足），也可以是**局部正交**（在 $x$ 的每个邻域内都正交，MLS 满足）。

---

## 2.5 几何直觉：投影 + 减法

> **一句话本质**：中性化 = 把因子投影到已知风险因子张成的子空间，然后把投影部分**减掉**，剩下的残差就是纯净因子。

### 2.5.1 把因子当作向量

在某个交易日，截面上有 $N$ 只股票。把因子值看作一个 $N$ 维向量 $\mathbf{f} \in \mathbb{R}^N$；同样把 size、momentum、行业哑变量等已知风险因子也各看作一个 $N$ 维向量，组成矩阵 $\mathbf{X} \in \mathbb{R}^{N \times k}$。

中性化要解决的问题是：**$\mathbf{f}$ 中有多少是已知风险因子的影子，多少是真正的 alpha？**

### 2.5.2 投影：解释力的最大重叠

把 $\mathbf{f}$ 沿 $\mathbf{X}$ 列空间投影：
$$\text{proj}_{\mathbf{X}}(\mathbf{f}) = \mathbf{X}\,\hat{\boldsymbol{\beta}}, \qquad \hat{\boldsymbol{\beta}} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{f}$$

这就是 OLS。$\hat{\boldsymbol{\beta}}$ 是让 $\mathbf{X}\boldsymbol{\beta}$ 尽量接近 $\mathbf{f}$ 的**最佳线性组合**——即"用风险因子能解释 $\mathbf{f}$ 多少"。

代回得到投影矩阵：
$$\hat{\mathbf{f}} = \underbrace{\mathbf{X}(\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T}_{\mathbf{P}_{\mathbf{X}}} \mathbf{f}$$

$\mathbf{P}_{\mathbf{X}}$（即 §3.2 的 Hat 矩阵 $\mathbf{H}$）是**正交投影算子**，把任何向量压扁到 $\mathbf{X}$ 列空间上。

### 2.5.3 减法：剩下的就是干净 alpha

$$\boxed{\;\boldsymbol{\varepsilon} = \mathbf{f} - \hat{\mathbf{f}} = (\mathbf{I} - \mathbf{P}_{\mathbf{X}})\mathbf{f}\;}$$

$(\mathbf{I} - \mathbf{P}_{\mathbf{X}})$ 叫**正交补投影**——把 $\mathbf{f}$ 沿垂直于 $\mathbf{X}$ 列空间的方向截下来。这就是中性化后的因子。

**直观图**：

```
                    f (原因子)
                   /|
                  / |
                 /  | ε = (I - P_X)f
                /   |   ← 中性化后, 与风险子空间正交
               /    |
              /     |
             /______|________  X 张成的风险子空间
                                (size, momentum, 行业 ...)
             proj_X(f) = Xβ̂
              "已被风险解释的部分"
```

### 2.5.4 为什么叫"正交"——可验证的 sanity check

由 $(\mathbf{I} - \mathbf{P}_{\mathbf{X}})$ 的构造可证：
$$\mathbf{X}^T \boldsymbol{\varepsilon} = \mathbf{X}^T(\mathbf{I} - \mathbf{P}_{\mathbf{X}})\mathbf{f} = \mathbf{0}$$

也就是说，残差 $\boldsymbol{\varepsilon}$ 与 $\mathbf{X}$ 的**每一列**都正交——和 size 截面相关 = 0，和 momentum 截面相关 = 0，和**每个**行业哑变量截面相关 = 0。

**实操中的 sanity check**：跑完中性化后

```python
np.corrcoef(neutral, panel_with_risk['SIZE'])[0, 1]      # 应 ≈ 0
np.corrcoef(neutral, panel_with_risk['MOMENTUM'])[0, 1]  # 应 ≈ 0
```

如果不为 0，说明回归奇异（截面样本数 < 自由度、或 $\mathbf{X}$ 内部高度共线），需要查回归条件。

### 2.5.5 numpy 落地：一行字背后的几何

```python
# X = [const, SIZE, MOMENTUM, BETA, LIQUIDTY, RESVOL, 行业_1, ..., 行业_30]
coef, *_ = np.linalg.lstsq(X, y, rcond=None)
resids = y - X @ coef        # ← 这一行就是 (I - P_X) f
```

`y - X @ coef` 就是**正交补投影**的具体形式。`lstsq` 没有显式构造 $\mathbf{P}_{\mathbf{X}}$（$N \times N$ 太大），而是用 QR 分解高效求出 $\hat{\boldsymbol{\beta}}$，这等价于"先投影、再相减"。

### 2.5.6 与 Barra 模型互为镜像

同一套数学，目的相反：

| 视角 | 目标 | 用 $\hat{\boldsymbol{\beta}}$ 干什么 | 用 $\boldsymbol{\varepsilon}$ 干什么 |
|---|---|---|---|
| **Barra** | 解释收益的最大部分 | 系数即风险因子的**定价** | 残差是 specific return，配 srisk |
| **中性化** | 剥离已知部分留下未知 alpha | 系数丢弃 | 残差就是纯净因子 |

Barra 关心 $\hat{\boldsymbol{\beta}}$，中性化关心 $\boldsymbol{\varepsilon}$；两者是同一个回归 OLS 的"两面"。这也解释了为什么用 Barra 的风格暴露 (`SIZE`, `MOMENTUM`, ...) 当 $\mathbf{X}$ 是天然的——它们本身就是被设计来吸收系统性风险的最佳线性投影方向。

### 2.5.7 OLS 几何 → MLS 几何（铺垫 §5）

OLS 投影是**一刀切**的：用同一组 $\hat{\boldsymbol{\beta}}$ 解释所有股票。

但实际上 size 在大小盘股上对收益的影响曲线**形状不同**。MLS 把"全局投影"换成"在每个 $x$ 的局部邻域内单独做一次投影"——几何上相当于沿着 $\mathbf{X}$ 子空间走一条**弯曲的曲面**而不是平直的超平面。详细推导见 §5。

---

## 3. OLS 中性化（回顾）

### 3.1 模型与正规方程
设截面有 $N$ 只股票，$\mathbf{y} \in \mathbb{R}^N$ 为因子向量，$\mathbf{X} \in \mathbb{R}^{N \times k}$ 为风险设计矩阵（首列为 1，其余列为风险因子）。

模型：
$$\mathbf{y} = \mathbf{X}\boldsymbol{\beta} + \boldsymbol{\epsilon}$$

最小化 $J = \|\mathbf{y} - \mathbf{X}\boldsymbol{\beta}\|^2$，对 $\boldsymbol{\beta}$ 求导：
$$\frac{\partial J}{\partial \boldsymbol{\beta}} = -2\mathbf{X}^T(\mathbf{y} - \mathbf{X}\boldsymbol{\beta}) = 0$$

得到正规方程：
$$\mathbf{X}^T\mathbf{X}\,\hat{\boldsymbol{\beta}} = \mathbf{X}^T\mathbf{y} \quad\Longrightarrow\quad \hat{\boldsymbol{\beta}} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{y}$$

### 3.2 中性化算子（投影矩阵）
拟合值：
$$\hat{\mathbf{y}} = \mathbf{X}\hat{\boldsymbol{\beta}} = \underbrace{\mathbf{X}(\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T}_{\mathbf{H}}\mathbf{y}$$

$\mathbf{H}$ 是著名的 **Hat 矩阵**（投影矩阵），满足 $\mathbf{H}^2 = \mathbf{H}$、$\mathbf{H}^T = \mathbf{H}$。

中性化残差：
$$\mathbf{y}_{\text{neutral}} = (\mathbf{I} - \mathbf{H})\mathbf{y}$$

### 3.3 OLS 的两个致命假设
1. **关系是线性的**：风险贡献 $f(x) = \beta_0 + \beta_1 x$ 是直线。
2. **系数是空间不变的**：所有股票共享同一个 $\boldsymbol{\beta}$。

但现实中：
- **市值非线性**：超大盘股（$\log\text{MV} > 28$）和中小盘股（$\log\text{MV} \in [22, 24]$）对因子值的影响曲率完全不同。
- **行业异质性**：消费股的 Beta-收益关系与科技股不同。
- **流动性分层**：高流动性区间因子有效，低流动性区间因子失真。

→ OLS 留下的残差仍然包含 $x$ 的**非线性结构信息**，中性化不彻底。

---

## 4. WLS：加权最小二乘（OLS → MLS 的桥梁）

WLS 给每只股票一个**信任度权重** $w_i$（如流通市值、$1/\sigma_i^2$）：
$$J = \sum_{i=1}^{N} w_i (y_i - \mathbf{x}_i^T \boldsymbol{\beta})^2$$

写成矩阵形式 $\mathbf{W} = \text{diag}(w_1, \dots, w_N)$：
$$J = (\mathbf{y} - \mathbf{X}\boldsymbol{\beta})^T \mathbf{W} (\mathbf{y} - \mathbf{X}\boldsymbol{\beta})$$

求导：
$$\frac{\partial J}{\partial \boldsymbol{\beta}} = -2\mathbf{X}^T \mathbf{W} (\mathbf{y} - \mathbf{X}\boldsymbol{\beta}) = 0$$

WLS 闭式解：
$$\hat{\boldsymbol{\beta}}_{\text{WLS}} = (\mathbf{X}^T \mathbf{W} \mathbf{X})^{-1} \mathbf{X}^T \mathbf{W} \mathbf{y}$$

**WLS 与 OLS 的关键区别**：
- 权重 $w_i$ **不依赖于查询点 $x$**，它对每只股票是固定的。
- WLS 仍然是**全局**拟合，只产生一组 $\hat{\boldsymbol{\beta}}$。

**WLS 与 MLS 的关键区别**：
- MLS 的权重 $w(x - x_i)$ 依赖于"我们正在评估的位置 $x$"；
- 每移动一步，权重整体重算，得到一组新的 $\mathbf{a}(x)$；
- 因此 MLS 的系数 $\mathbf{a}(x)$ 是 $x$ 的**函数**，不是常数向量。

→ MLS = "对每个查询点 $x$ 做一次 WLS"。

---

## 5. MLS 数学建模：从全局到局部

### 5.1 局部基函数展开
假设在 $x$ 的邻域内，风险与因子值的关系可以用一组基函数 $\mathbf{p}(x)$ 的线性组合来近似：
$$f(x) \approx \hat{f}(x) = \mathbf{p}^T(x) \mathbf{a}(x)$$

其中：
- $\mathbf{p}(x) = [1, x, x^2, \dots, x^m]^T \in \mathbb{R}^{m+1}$ 是 $m$ 阶多项式基（也可以是径向基 / 三角基）。
- $\mathbf{a}(x) \in \mathbb{R}^{m+1}$ 是**待求的系数向量**。

> **关键观察**：在 OLS 中 $\boldsymbol{\beta}$ 是常数；在 WLS 中 $\boldsymbol{\beta}$ 仍是常数；
> 而在 MLS 中，$\mathbf{a}(x)$ 是 $x$ 的函数——这是 MLS 与传统最小二乘的本质分水岭。

### 5.2 局部加权损失函数
为了求出在特定点 $x$ 处的系数 $\mathbf{a}(x)$，定义局部加权损失：
$$J(\mathbf{a}; x) = \sum_{i=1}^{N} w(x - x_i) \left[ \mathbf{p}^T(x_i) \mathbf{a}(x) - y_i \right]^2$$

权重函数 $w(x - x_i)$ 通常选择**紧支集核函数**：
- **高斯核**：$w(d) = \exp(-d^2 / 2h^2)$，$h$ 为带宽。
- **三次样条核**（Wendland 核）：
  $$w(d) =
  \begin{cases}
  \frac{2}{3} - 4r^2 + 4r^3, & r \le 1/2 \\
  \frac{4}{3} - 4r + 4r^2 - \frac{4}{3}r^3, & 1/2 < r \le 1 \\
  0, & r > 1
  \end{cases}, \quad r = |d|/h$$
- **Epanechnikov 核**：$w(d) = \max(0, 1 - (d/h)^2)$，方差最小、ISE 最优。

含义：**离当前计算点 $x$ 越近的样本，对该点性质的决定权越大**；超出带宽 $h$ 的样本权重为 0（紧支集）或迅速衰减（高斯）。

### 5.3 求解正规方程（完整推导）
为了最小化 $J$，对 $\mathbf{a}(x)$ 求梯度：
$$\frac{\partial J}{\partial \mathbf{a}} = 2\sum_{i=1}^{N} w(x - x_i) \mathbf{p}(x_i) \left[ \mathbf{p}^T(x_i) \mathbf{a}(x) - y_i \right] = 0$$

整理得：
$$\sum_{i=1}^{N} w(x - x_i) \mathbf{p}(x_i) \mathbf{p}^T(x_i) \mathbf{a}(x) = \sum_{i=1}^{N} w(x - x_i) \mathbf{p}(x_i) y_i$$

写成矩阵形式：
$$\mathbf{A}(x) \mathbf{a}(x) = \mathbf{B}(x) \mathbf{y}$$

其中：
- **形态矩阵（Moment Matrix）**：
$$\mathbf{A}(x) = \sum_{i=1}^{N} w(x - x_i)\, \mathbf{p}(x_i) \mathbf{p}^T(x_i) \in \mathbb{R}^{(m+1)\times(m+1)}$$

- **载荷矩阵（Loading Matrix）**：
$$\mathbf{B}(x) = \big[w(x-x_1)\mathbf{p}(x_1),\ w(x-x_2)\mathbf{p}(x_2),\ \dots,\ w(x-x_N)\mathbf{p}(x_N)\big] \in \mathbb{R}^{(m+1)\times N}$$

解得系数向量：
$$\boxed{\mathbf{a}(x) = \mathbf{A}^{-1}(x) \mathbf{B}(x) \mathbf{y}}$$

> **与 WLS 形式对比**：
> $$\hat{\boldsymbol{\beta}}_{\text{WLS}} = (\mathbf{X}^T \mathbf{W} \mathbf{X})^{-1} \mathbf{X}^T \mathbf{W} \mathbf{y}$$
> 与 MLS 的
> $$\mathbf{a}(x) = \mathbf{A}^{-1}(x) \mathbf{B}(x) \mathbf{y}$$
> 形式上完全一致。差别只在于：MLS 的 $\mathbf{W}$ 矩阵随 $x$ 变化、$\mathbf{X}$ 的每行用 $\mathbf{p}(x_i)$ 而非 $\mathbf{x}_i$ 表达。

### 5.4 矩阵 $\mathbf{A}(x)$ 可逆性的讨论
$\mathbf{A}(x)$ 可逆 $\Leftrightarrow$ 在 $x$ 的核支集内，至少有 $m+1$ 个**线性无关**的样本点 $\{x_i\}$。

**实务退化情况**：
1. **数据稀疏**：低流动性股票分布的尾部（如 $\log\text{MV} < 22$）样本太少 → $\mathbf{A}(x)$ 病态。
2. **共线性**：基函数维度 $m$ 过高、邻域内样本聚集 → 数值秩亏。
3. **带宽过小**：核内有效样本数 $< m+1$ → 直接奇异。

**工程缓解**：
- **Tikhonov 正则化**：$\mathbf{A}(x) \to \mathbf{A}(x) + \lambda \mathbf{I}$（即 Ridge 风格），等价于添加先验"系数不要太大"。
- **自适应带宽**：$h(x) = h_0 \cdot \rho(x)^{-1/d}$，$\rho(x)$ 是局部样本密度。
- **降阶**：在稀疏区域强制 $m=1$（局部线性），仅在密集区域使用 $m=2$。

---

## 6. 中性化算子的最终形式

### 6.1 拟合值与 Smoothing Matrix
对截面上任一点 $x_i$，MLS 预测的风险贡献值为：
$$\hat{y}_i = \hat{f}(x_i) = \mathbf{p}^T(x_i) \mathbf{a}(x_i) = \mathbf{p}^T(x_i) \mathbf{A}^{-1}(x_i) \mathbf{B}(x_i) \mathbf{y}$$

把该过程抽象为 **Smoothing Matrix** $\mathbf{S} \in \mathbb{R}^{N \times N}$，使得 $\hat{\mathbf{y}} = \mathbf{S} \mathbf{y}$：
$$\boxed{\,S_{ij} = \mathbf{p}^T(x_i) \mathbf{A}^{-1}(x_i)\, w(x_i - x_j)\, \mathbf{p}(x_j)\,}$$

**最终中性化因子**：
$$\boxed{\mathbf{y}_{\text{neutral}} = (\mathbf{I} - \mathbf{S}) \mathbf{y}}$$

### 6.2 Smoothing Matrix 的性质
不同于 OLS 的 Hat 矩阵 $\mathbf{H}$：
| 性质 | OLS $\mathbf{H}$ | MLS $\mathbf{S}$ |
|---|---|---|
| 幂等 $\mathbf{S}^2 = \mathbf{S}$ | ✅ 是 | ❌ 不是（除非核退化为 $\delta$） |
| 对称 $\mathbf{S}^T = \mathbf{S}$ | ✅ 是 | ❌ 一般不对称 |
| 秩 | $k$（风险维度） | 介于 $m+1$ 与 $N$ 之间 |
| 谱含义 | 严格投影 | 低通滤波器 |

**自由度（Effective Degrees of Freedom）**：
$$\text{df}_{\text{MLS}} = \text{tr}(\mathbf{S})$$
带宽越小，$\text{tr}(\mathbf{S})$ 越大（拟合越激进）；带宽越大，$\text{tr}(\mathbf{S}) \to k$（退化为全局多项式回归）。

### 6.3 物理意义
1. **局部正交性**：$\mathbf{y}_{\text{neutral}}$ 在每个 $x$ 的邻域内与 $\mathbf{p}(x)$ 张成的子空间近似正交。
2. **自适应去噪**：$\mathbf{S}$ 是低通滤波器，提取因子中与风险相关的"低频"分量；$(\mathbf{I} - \mathbf{S})$ 是高通滤波器，留下"高频"Alpha 信号。
3. **局部保真**：在低流动性区间和高流动性区间分别做最优纠偏，而不是用密集区域的统计特性粗暴覆盖稀疏区域。
4. **算子连续性**：如果核函数 $w$ 是 $C^k$ 的，那么 $\mathbf{y}_{\text{neutral}}(x)$ 也是 $C^k$ 的——避免分箱中性化的阶跃突变。

---

## 7. OLS / WLS / MLS 系统对比

| 维度 | OLS | WLS | MLS |
|---|---|---|---|
| **损失函数** | $\sum (y_i - \mathbf{x}_i^T\boldsymbol{\beta})^2$ | $\sum w_i (y_i - \mathbf{x}_i^T\boldsymbol{\beta})^2$ | $\sum w(x-x_i)(y_i - \mathbf{p}^T(x_i)\mathbf{a}(x))^2$ |
| **系数** | 常数 $\hat{\boldsymbol{\beta}}$ | 常数 $\hat{\boldsymbol{\beta}}_w$ | 函数 $\mathbf{a}(x)$ |
| **权重来源** | 无 | 静态信任度（市值/$1/\sigma^2$） | 动态核 $w(x-x_i)$，依赖查询点 |
| **拟合形态** | 全局直线/超平面 | 全局直线/超平面（加权） | 任意光滑曲线 |
| **拟合矩阵** | $\mathbf{H}=\mathbf{X}(\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T$ | $\mathbf{H}_w=\mathbf{X}(\mathbf{X}^T\mathbf{W}\mathbf{X})^{-1}\mathbf{X}^T\mathbf{W}$ | $\mathbf{S}$（每行重新计算） |
| **复杂度** | $O(Nk^2 + k^3)$ 一次解 | $O(Nk^2 + k^3)$ | $O(N \cdot (n_h k^2 + k^3))$ |
| **超参数** | 无 | $w_i$（先验给定） | 核类型、带宽 $h$、阶数 $m$ |
| **偏差** | 模型偏差大（线性强假设） | 同 OLS（线性仍是常数 $\boldsymbol{\beta}$） | 偏差小（局部多项式逼近） |
| **方差** | 方差小（自由度低） | 方差小 | 方差大（自由度高，需带宽控制） |
| **正交性** | 全局严格正交 | 全局加权正交 | 局部近似正交 |
| **典型场景** | 标准风格中性化 | 大盘股加权的市值中性化 | 非线性因子（OFI、订单簿熵）中性化 |

> 其中 $n_h$ 是核支集内的有效样本数（$N$ 远大于 $n_h$ 时，MLS 的 $O(N \cdot n_h k^2)$ 项是主项）。

### 7.1 何时该升级到 MLS？
满足以下任一条件：
- 风险因子与 Alpha 之间存在**已知的非线性**（市值、流动性的对数-对数曲线、波动率-收益的微笑曲线）。
- 因子在**截面分位数极端**位置表现异常（小市值尾部、超低换手尾部）。
- 信号是**高频微观结构因子**（OFI、Order Imbalance、Tick-by-Tick Entropy），其与流动性的关系本身就是曲面。

否则——尤其是数据稀疏、$N$ 较小、风险维度高的情况——OLS / Ridge 通常更稳。

---

## 8. 实现要点与超参数选择

### 8.1 带宽 $h$ 的选择
- **GCV（Generalized Cross-Validation）**：
  $$h^* = \arg\min_h \frac{\frac{1}{N}\|\mathbf{y} - \mathbf{S}_h\mathbf{y}\|^2}{(1 - \text{tr}(\mathbf{S}_h)/N)^2}$$
- **AICc**：$\text{AICc}(h) = N\log\hat{\sigma}^2 + \frac{2(\text{tr}(\mathbf{S}_h)+1)}{N-\text{tr}(\mathbf{S}_h)-2}$
- **经验法则（金融）**：$h \approx (\max(x) - \min(x))/20$ 起步，在 OOS IC 上扫描。

### 8.2 多项式阶数 $m$
- $m = 0$：核回归（Nadaraya–Watson），最简单，存在边界偏差。
- $m = 1$：**局部线性回归（LOESS）**，金融截面默认推荐。边界处偏差远小于 $m=0$，对带宽不敏感。
- $m = 2$：局部二次，能捕捉曲率，但样本量要求高、对噪声敏感。

> 经验：因子中性化用 $m = 1$ 已经足够；只有当你确实在拟合**碗状/驼峰状**的关系（如波动率微笑）时才考虑 $m = 2$。

### 8.3 多维风险（市值 + 行业 + Beta）的处理
当 $\mathbf{x}_i \in \mathbb{R}^d$（$d > 1$）时：
- **核函数张量化**：$w(\mathbf{x} - \mathbf{x}_i) = \prod_{j=1}^{d} K\!\left(\frac{x_j - x_{i,j}}{h_j}\right)$
- **维度灾难**：$d > 3$ 时核内有效样本数指数衰减，MLS 失效。
- **实务做法**：
  1. **行业先分箱**：按行业分组后在组内做 MLS（仅市值/Beta 这 1-2 维连续变量上局部）。
  2. **加性结构**：$f(x_1, x_2) = f_1(x_1) + f_2(x_2)$，逐维做 MLS（GAM 思想）。
  3. **降维后 MLS**：先用 PCA 把风险压到 2 维，再在 PCA 空间上 MLS。

### 8.4 计算复杂度优化
- **KD-Tree 索引**：把样本点 $\{x_i\}$ 建树，查询每个 $x$ 的核内邻居只需 $O(\log N)$。
- **批量求逆**：$N$ 个 $\mathbf{A}(x_i)$ 都是 $(m+1) \times (m+1)$ 小矩阵，$O((m+1)^3)$ 解析求逆即可。
- **GPU 并行**：每个查询点独立，天然并行。

---

## 9. 与高频 / 微观结构因子的连接

### 9.1 为什么 OFI / 订单簿因子需要 MLS
OFI（Order Flow Imbalance）、深度不平衡、Micro-price 偏离等微观结构因子有共同特征：
- 与流动性的关系是**强非线性**（Square-Root Law、Kyle's Lambda 都是凹函数）。
- 在不同 Tick Size 分类下行为不同（参考 [TickSize.md](TickSize.md) 的 Small/Medium/Large 分类）。
- 与对数市值之间存在**幂律**关系而非线性。

→ 如果用 OLS 中性化，残差仍然带有显著的市值斜率；MLS 能干净剥离。

### 9.2 体积时钟下的中性化
当因子在 **Volume Clock** 上构造（如 VPIN，参考微观结构模块）时：
- 横轴不再是物理时间，而是均匀的成交量桶。
- 但截面上不同股票成交速度不同，单位时间内累积的桶数不同 → 风险暴露随成交活跃度变化。
- 此时的中性化 $x$ 应取**单位物理时间桶数 $\rho_i$** 而非市值，MLS 的 $\mathbf{p}(x) = [1, \log\rho, (\log\rho)^2]$ 能很好捕捉曲率。

### 9.3 与本笔记其他主题的交叉
- **OLS 基础**：参考 [OLS_MultipleRegression.md](OLS_MultipleRegression.md)。
- **Ridge / LASSO**：MLS 的 $\mathbf{A}(x) + \lambda\mathbf{I}$ 退化即 Local Ridge，参考 [Ridge_LASSO.md](Ridge_LASSO.md)。
- **去极值**：MLS 对离群点敏感（核内一个极端样本就能扭曲局部拟合），先做 [MAD 去极值](OutlierDetection_MAD.md) 是标准前置步骤。
- **截面工作流**：完整顺序参考 [DataProcessing/Workflow.md](../DataProcessing/Workflow.md)。

---

## 10. 总结

MLS 中性化在数学上是把**全局线性回归**升级为**空间自适应的局部算子**：

$$\underbrace{\mathbf{y}_{\text{neutral}} = (\mathbf{I} - \mathbf{H})\mathbf{y}}_{\text{OLS：全局投影}} \quad\Longrightarrow\quad \underbrace{\mathbf{y}_{\text{neutral}} = (\mathbf{I} - \mathbf{S})\mathbf{y}}_{\text{MLS：局部投影合成}}$$

关键升级：
1. **解决非线性暴露**：把直线换成任意光滑曲线。
2. **核函数平滑性**：保证因子在不同风险特征值之间过渡自然连续。
3. **局部保真**：稀疏区域和密集区域分别拟合，不发生信息互染。

代价：
1. **超参数调优**：核类型、带宽、阶数都要扫描。
2. **维度灾难**：$d > 2$ 后必须配合 GAM / PCA / 行业分箱。
3. **计算成本**：$O(N \cdot n_h k^2)$，比 OLS 的 $O(Nk^2 + k^3)$ 高一个 $N$ 因子（虽常数小）。

→ 在低频选股场景下 OLS 仍然是默认工具；在高频微观结构因子上，MLS 是结构性必需品。

→ 在战略层面，中性化的本质是**用 Beta 收益换取 Alpha 的可观测性与可控性**——只有当因子残差不再依赖大盘风格时，IC、夏普、回撤这些指标才度量的是"你"，而不是"市场"。

#中性化 #MLS #OLS对比 #Alpha与Beta #风险因子 #因子工程 #高频因子
