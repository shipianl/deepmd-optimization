# DeepMD 基本原理

![DeepMD 流程图](images/v2-5f49363ccfbe5bea68286aece684e6b2_r.png)

## 1. 构建局域环境

定义一个 $N$ 原子系统的坐标矩阵 $\mathcal{R} \in \mathbb{R}^{N \times 3}$：

$$\mathcal{R}=\{r_{1}^{T}, \cdots, r_{i}^{T}, \cdots, r_{N}^{T}\}^{T}, \quad r_{i}=\left(x_{i}, y_{i}, z_{i}\right) \quad (1)$$

$r_{i}$ 表示原子 $i$ 的三维笛卡尔坐标。将坐标矩阵 $\mathcal{R}$ 转换成局域坐标矩阵 $\{\mathcal{R}^{i}\}_{i=1}^{N}$：

$$\mathcal{R}^{i}=\{r_{1i}^{T}, \cdots, r_{ji}^{T}, \cdots, r_{N_{i},i}^{T}\}^{T}, \quad r_{ji}=\left(x_{ji}, y_{ji}, z_{ji}\right) \quad (2)$$

其中 $j$ 和 $N_{i}$ 是原子 $i$ 在截断半径 $r_{c}$ 内近邻原子的编号，$j (1 \leq j \leq N_{i})$ 表示原子 $i$ 的近邻原子编号，$r_{ji} \equiv r_{j}-r_{i}$ 表示原子 $j$ 相对于原子 $i$ 的位置向量。

$r_{c}$ 是**用户预设的超参数**，取决于要模拟的物理体系。

## 2. 广义坐标映射

在 DP 方法中，一个系统的总能量 $E$ 等于各个原子的局域能量的总和：

$$E=\sum_{i} E_{i} \quad (3)$$

其中 $E_{i}$ 是原子 $i$ 的局域能量，$E_{i}$ 取决于原子 $i$ 的局域环境：

$$E=\sum_{i} E_{i}=\sum_{i} E\left(\mathcal{R}^{i}\right) \quad (4)$$

$\mathcal{R}^{i}$ 被映射到特征矩阵（描述子）$\mathcal{D}^{i}$，保留了体系的平移、旋转和置换不变性。具体来说，$\mathcal{R}^{i} \in \mathbb{R}^{N_{i} \times 3}$ 被变换为 $\tilde{\mathcal{R}}^{i} \in \mathbb{R}^{N_{i} \times 4}$，每一行代表：

$$\{x_{ji}, y_{ji}, z_{ji}\} \mapsto\{s(r_{ji}), \hat{x}_{ji}, \hat{y}_{ji}, \hat{z}_{ji}\} \quad (5)$$

其中：

$$\hat{x}_{ji}=\frac{s(r_{ji}) x_{ji}}{r_{ji}}, \quad \hat{y}_{ji}=\frac{s(r_{ji}) y_{ji}}{r_{ji}}, \quad \hat{z}_{ji}=\frac{s(r_{ji}) z_{ji}}{r_{ji}}$$

$s(r_{ji})$ 是一个权函数，其定义为：

$$s(r_{ji})=\begin{cases}
\dfrac{1}{r_{ji}}, & r_{ji}<r_{cs} \\[12pt]
\dfrac{1}{r_{ji}} \left\{ \left(\dfrac{r_{ji} - r_{cs}}{r_c - r_{cs}}\right)^3 \left[-6\left(\dfrac{r_{ji} - r_{cs}}{r_c - r_{cs}}\right)^2 +15\dfrac{r_{ji} - r_{cs}}{r_c - r_{cs}} -10\right] +1 \right\}, & r_{cs} \leq r_{ji} < r_c \\[12pt]
0, & r_{ji}>r_{c}
\end{cases} \quad (6)$$

其中 $r_{ji}$ 是原子 $i$ 和原子 $j$ 之间的欧氏距离，$r_{cs}$ 是"平滑截断半径"。

## 3. 嵌入网络

引入 $s(r_{ji})$ 之后，$\tilde{\mathcal{R}}^{i}$ 里的各个参数会从 $r_{cs}$ 到 $r_{c}$ 平滑地趋于零。

定义局域嵌入网络 $\mathcal{N}^{e}_{\alpha_j,\alpha_i}(s(r_{ji}))$，它是一个神经网络：

- **输入**：标量 $s(r_{ji})$（仅径向距离信息）
- **输出**：$M_1$ 维向量
- 网络参数取决于中心原子 $i$ 和邻居原子 $j$ 的化学物种组合

将输出整理为嵌入矩阵，其中矩阵元素为：

$$(\mathcal{G}^{i})_{jk} = (\mathcal{G}(s(r_{ji})))_{k}$$

$\{s(r_{ji})\}_{j=1}^{N_i}$（即 $\tilde{\mathcal{R}}^{i}$ 的第一列）通过一个嵌入神经网络得到一个嵌入矩阵 $\mathcal{G}^{i1} \in \mathbb{R}^{N_{i} \times M_{1}}$。

类似地，$\{\hat{x}_{ji}, \hat{y}_{ji}, \hat{z}_{ji}\}_{j=1}^{N_i}$（即 $\tilde{\mathcal{R}}^{i}$ 的后三列）通过另一个嵌入神经网络得到一个嵌入矩阵 $\mathcal{G}^{i2} \in \mathbb{R}^{N_{i} \times M_{1}}$。

## 4. 对称性特征矩阵

原子 $i$ 的描述子 $\mathcal{D}^{i}$：

$$\mathcal{D}^{i}=\left(\mathcal{G}^{i1}\right)^{T} \tilde{\mathcal{R}}^{i}\left(\tilde{\mathcal{R}}^{i}\right)^{T} \mathcal{G}^{i2} \quad (7)$$

在描述子中，平移和旋转不变性由矩阵乘积 $\tilde{\mathcal{R}}^{i}(\tilde{\mathcal{R}}^{i})^{T}$ 保证，置换不变性由矩阵乘积 $(\mathcal{G}^{i})^{T} \tilde{\mathcal{R}}^{i}$ 保证。

## 5. 拟合网络

每一个描述子 $\mathcal{D}^{i}$ 展平为向量，通过一个拟合神经网络被映射到一个局域能量 $E_{i}$。

嵌入神经网络 $\mathcal{N}^{e}$ 和拟合神经网络 $\mathcal{N}^{f}$ 都是包含很多隐藏层的前馈神经网络。前一层的输入数据 $d_{l}^{\mathrm{in}}$ 通过一个线性运算和非线性激活函数得到后一层的输出数据 $d_{k}^{\mathrm{out}}$：

$$d_{k}^{\mathrm{out}}=\varphi\left(\sum_{kl} w_{kl} d_{l}^{\mathrm{in}}+ b_{k}\right) \quad (8)$$

输出描述子 $\mathcal{D}^{i}$ 对应的能量 $E_{i}$。

## 6. 计算物理量

- **总能量**：$\displaystyle E=\sum_{i} E_{i}$
- **原子力**：$\displaystyle F = -\nabla_{\mathcal{R}}E$（能量对坐标的负梯度）
- **维里张量**：$\displaystyle \Xi = \mathrm{tr}[\mathcal{R} \otimes F]$

维里张量（virial tensor）是分子动力学/材料模拟中描述**应力状态**的量，本质上是系统对外部压力的响应。

## 7. 训练过程

在公式 (8) 中，$w_{kl}$ 是权重参数，$b_{k}$ 是偏置参数，$\varphi$ 是一个非线性的激活函数。在最后一层的输出节点是没有非线性激活函数的。嵌入网络和拟合网络的权重通过最小化以下损失函数来训练：

$$L(p_{\epsilon}, p_{f}, p_{\xi})=\frac{p_{\epsilon}}{N} \Delta \epsilon^{2}+\frac{p_{f}}{3N} \sum_{i}\left|\Delta F_{i}\right|^{2}+\frac{p_{\xi}}{9N}\|\Delta \xi\|^{2} \quad (9)$$

其中 $\Delta \epsilon$、$\Delta F_{i}$ 和 $\Delta \xi$ 分别表示能量、力和维里的均方根误差（RMSE）。在训练过程中，前置因子 $p_{\epsilon}$、$p_{f}$ 和 $p_{\xi}$ 会随训练步数 $t$ 逐渐变化：

$$p(t)=p^{\mathrm{limit}}\left[1-\frac{r_{l}(t)}{r_{l}^{0}}\right]+p^{\mathrm{start}}\left[\frac{r_{l}(t)}{r_{l}^{0}}\right] \quad (10)$$

其中 $r_{l}(t)$ 和 $r_{l}^{0}$ 分别表示在训练步数为 $t$ 和训练步数为 $0$ 时的学习率。$r_{l}(t)$ 的定义为：

$$r_{l}(t)=r_{l}^{0} \times d_{r}^{t / d_{s}} \quad (11)$$

其中 $d_{r}$ 和 $d_{s}$ 分别表示学习衰减率以及衰减步数。学习衰减率 $d_{r}$ 要严格小于 $1$。
