## DeepMD基本流程：

![[Pasted image 20260520150832.png]]

## 1. 构建局域环境

在介绍DP具体方法之前, 我们首先定义一个 <img src="https://latex.codecogs.com/svg.image?N"> 原子系统的坐标矩阵 <img src="https://latex.codecogs.com/svg.image?\mathcal{R}\in\mathbb{R}^{N\times3}">，

<img src="https://latex.codecogs.com/svg.image?\mathcal{R}=\left\{{r}_{1}^{T},\cdots,{r}_{i}^{T},\cdots,{r}_{N}^{T}\right\}^{T},{r}_{i}=\left(x_{i},y_{i},z_{i}\right),(1)">

<img src="https://latex.codecogs.com/svg.image?{r}_{i}"> 表示原子 <img src="https://latex.codecogs.com/svg.image?i"> 的三维笛卡尔坐标。此外，我们将坐标矩阵 <img src="https://latex.codecogs.com/svg.image?\mathcal{R}"> 转换成局域坐标矩阵 <img src="https://latex.codecogs.com/svg.image?\left\{{\mathcal{R}}^{i}\right\}_{i=1}^{N}">,

<img src="https://latex.codecogs.com/svg.image?{\mathcal{R}}^{i}=\left\{{r}_{1i}^{T},\cdots,{r}_{ji}^{T},\cdots,{r}_{N_{i},i}^{T}\right\}^{T},{r}_{ji}=\left(x_{ji},y_{ji},z_{ji}\right),(2)">

其中 <img src="https://latex.codecogs.com/svg.image?j"> 和 <img src="https://latex.codecogs.com/svg.image?N_{i}"> 是原子 <img src="https://latex.codecogs.com/svg.image?i"> 在截断半径 <img src="https://latex.codecogs.com/svg.image?r_{c}"> 内近邻原子的编号，<img src="https://latex.codecogs.com/svg.image?j(1\leq j\leq N_{i})"> 表示原子 <img src="https://latex.codecogs.com/svg.image?i"> 的近邻原子编号, <img src="https://latex.codecogs.com/svg.image?{r}_{ji}\equiv{r}_{j}-{r}_{i}"> 表示的是原子 <img src="https://latex.codecogs.com/svg.image?j"> 和原子 <img src="https://latex.codecogs.com/svg.image?i"> 之间的相对距离。

<img src="https://latex.codecogs.com/svg.image?r_{c}"> 是**用户预设的超参数**，取决于要模拟的物理体系

## 2. 广义坐标映射

在DP方法中, 一个系统的总能量 <img src="https://latex.codecogs.com/svg.image?E"> 等于各个原子的局域能量的总和

<img src="https://latex.codecogs.com/svg.image?E=\sum_{i}E_{i},(3)">

其中 <img src="https://latex.codecogs.com/svg.image?E_{i}"> 是原子 <img src="https://latex.codecogs.com/svg.image?i"> 的局域能量. 此外，<img src="https://latex.codecogs.com/svg.image?E_{i}"> 取决于原子 <img src="https://latex.codecogs.com/svg.image?i"> 的局域环境:

<img src="https://latex.codecogs.com/svg.image?E=\sum_{i}E_{i}=\sum_{i}E\left(\mathcal{R}^{i}\right),(4)">

通过将 <img src="https://latex.codecogs.com/svg.image?{\mathcal{R}}^{i}"> 要映射到特征矩阵，或者说描述子 <img src="https://latex.codecogs.com/svg.image?{\mathcal{D}}^{i}">，这里的 <img src="https://latex.codecogs.com/svg.image?{\mathcal{D}}^{i}"> 保留了体系的平移、旋转和置换不变性。具体来说，<img src="https://latex.codecogs.com/svg.image?{\mathcal{R}}^{i}\in\mathbb{R}^{N_{i}\times3}"> 首先被映射到一个扩展矩阵 <img src="https://latex.codecogs.com/svg.image?\tilde{{\mathcal{R}}}^{i}\in\mathbb{R}^{N_{i}\times4}">，

<img src="https://latex.codecogs.com/svg.image?\left\{x_{ji},y_{ji},z_{ji}\right\}\mapsto\left\{s\left(r_{ji}\right),\hat{x}_{ji},\hat{y}_{ji},\hat{z}_{ji}\right\},(5)">

其中 <img src="https://latex.codecogs.com/svg.image?\hat{x}_{ji}=\dfrac{s\left(r_{ji}\right)x_{ji}}{r_{ji}}">, <img src="https://latex.codecogs.com/svg.image?\hat{y}_{ji}=\dfrac{s\left(r_{ji}\right)y_{ji}}{r_{ji}}">, <img src="https://latex.codecogs.com/svg.image?\hat{z}_{ji}=\dfrac{s\left(r_{ji}\right)z_{ji}}{r_{ji}}">. <img src="https://latex.codecogs.com/svg.image?s\left(r_{ji}\right)"> 是一个权重函数，用来减少离原子 <img src="https://latex.codecogs.com/svg.image?i"> 比较远的原子的权重, 定义如下:

<img src="https://latex.codecogs.com/svg.image?s\left(r_{ji}\right)=\begin{cases}\dfrac{1}{r_{ji}}, %26 r_{ji}%3C r_{cs}\\[8pt]\dfrac{1}{r_{ji}}\left\{\left(\dfrac{r_{ji}-r_{cs}}{r_c-r_{cs}}\right)^3\left(-6\left(\dfrac{r_{ji}-r_{cs}}{r_c-r_{cs}}\right)^2+15\dfrac{r_{ji}-r_{cs}}{r_c-r_{cs}}-10\right)+1\right\}, %26 r_{cs}%3C r_{ji}%3C r_{c}\\[8pt]0, %26 r_{ji}%3E r_{c}\end{cases},(6)">

其中 <img src="https://latex.codecogs.com/svg.image?r_{ji}"> 是原子 <img src="https://latex.codecogs.com/svg.image?i"> 和原子 <img src="https://latex.codecogs.com/svg.image?j"> 之间的欧式距离, <img src="https://latex.codecogs.com/svg.image?r_{cs}"> 是"平滑截断半径"。

## 3. 嵌入网络

引入 <img src="https://latex.codecogs.com/svg.image?s\left(r_{ji}\right)"> 之后，<img src="https://latex.codecogs.com/svg.image?\tilde{{\mathcal{R}}}^{i}"> 里的各个参数会从 <img src="https://latex.codecogs.com/svg.image?r_{cs}"> 到 <img src="https://latex.codecogs.com/svg.image?r_{c}"> 平滑地趋于零。

定义局域嵌入网络 <img src="https://latex.codecogs.com/svg.image?\mathcal{N}^{e}_{\alpha_j,\alpha_i}(s(r_{ji}))"> ，它是一个神经网络：
**输入**：标量 <img src="https://latex.codecogs.com/svg.image?s\left(r_{ji}\right)">（仅径向距离信息）
**输出**：<img src="https://latex.codecogs.com/svg.image?M_1"> 维向量
网络参数取决于中心原子 i 和邻居原子 j 的化学物种组合

将输出整理为嵌入矩阵：<img src="https://latex.codecogs.com/svg.image?(\mathcal{G}^{i})_{jk}=(\mathcal{G}(s(r_{ji})))_{k}">

接着 <img src="https://latex.codecogs.com/svg.image?\left\{s\left(r_{ji}\right)\right\}_{j=1}^{N_i}">, 也就是 <img src="https://latex.codecogs.com/svg.image?\tilde{{\mathcal{R}}}^{i}"> 的第一列通过一个嵌入神经网络得到一个嵌入矩阵 <img src="https://latex.codecogs.com/svg.image?\mathcal{G}^{i1}\in\mathbb{R}^{N_{i}\times%20M_{1}}">. 选取 <img src="https://latex.codecogs.com/svg.image?{\mathcal{G}}^{i1}\in\mathbb{R}^{N_{i}\times M_{1}}"> 的前 <img src="https://latex.codecogs.com/svg.image?M_{2}(%3CM_{1})"> 列，我们就得到了另外一个嵌入矩阵 <img src="https://latex.codecogs.com/svg.image?\mathcal{G}^{i2}\in\mathbb{R}^{N_{i}\times%20M_{2}}">.

## 4. 对称性特征矩阵

我们可以得到原子 <img src="https://latex.codecogs.com/svg.image?i"> 的描述子 <img src="https://latex.codecogs.com/svg.image?{\mathcal{D}}^{i}">：

<img src="https://latex.codecogs.com/svg.image?\mathcal{D}^{i}=\left(\mathcal{G}^{i1}\right)^{T}\tilde{\mathcal{R}}^{i}\left(\tilde{\mathcal{R}}^{i}\right)^{T}\mathcal{G}^{i2},(7)">

在描述子中, 平移和旋转不变性是由矩阵乘积 <img src="https://latex.codecogs.com/svg.image?\tilde{\mathcal{R}}^{i}\left(\tilde{\mathcal{R}}^{i}\right)^{T}"> 来保证的, 置换不变性是由矩阵乘积 <img src="https://latex.codecogs.com/svg.image?\left(\mathcal{G}^{i}\right)^{T}\tilde{\mathcal{R}}^{i}"> 来保证的。

## 5. 拟合网络

每一个描述子 <img src="https://latex.codecogs.com/svg.image?{\mathcal{D}}^{i}"> 展平为向量，通过一个拟合神经网络被映射到一个局域能量 <img src="https://latex.codecogs.com/svg.image?E_{i}"> 上面。

嵌入神经网络 <img src="https://latex.codecogs.com/svg.image?\mathcal{N}^{e}"> 和拟合神经网络 <img src="https://latex.codecogs.com/svg.image?\mathcal{N}^{f}"> 都是包含很多隐藏层的前馈神经网络。前一层的输入数据 <img src="https://latex.codecogs.com/svg.image?d_{l}^{\mathrm{in}}"> 通过一个线性运算和一个非线性的激活函数得到下一层的输入数据 <img src="https://latex.codecogs.com/svg.image?d_{k}^{\mathrm{out}}">。

<img src="https://latex.codecogs.com/svg.image?d_{k}^{\mathrm{out}}=\varphi\left(\sum_{kl}w_{kl}d_{l}^{\mathrm{in}}+b_{k}\right),(8)">

输出描述子 <img src="https://latex.codecogs.com/svg.image?{\mathcal{D}}^{i}"> 对应的能量 <img src="https://latex.codecogs.com/svg.image?E_{i}">

## 6. 计算物理量

总能量：<img src="https://latex.codecogs.com/svg.image?\displaystyle%20E=\sum_{i}E_{i}">

原子力：<img src="https://latex.codecogs.com/svg.image?F=-\nabla_{\mathcal{R}}E">（能量对坐标的负梯度）

维里张量：<img src="https://latex.codecogs.com/svg.image?\Xi=\text{tr}[\mathcal{R}\otimes%20F]">

维里张量（virial tensor）是分子动力学/材料模拟中描述**应力状态**的量，本质上是系统对外部压力的响应

## 7. 训练过程

在公式（8）中, <img src="https://latex.codecogs.com/svg.image?{w}_{kl}"> 是权重参数, <img src="https://latex.codecogs.com/svg.image?{b}_{k}"> 是偏置参数，<img src="https://latex.codecogs.com/svg.image?\varphi"> 是一个非线性的激活函数。需要注意的是，在最后一层的输出节点是没有非线性激活函数的。在嵌入网络和拟合网络中的参数由最小化代价函数 <img src="https://latex.codecogs.com/svg.image?L"> 得到:

<img src="https://latex.codecogs.com/svg.image?L\left(p_{\epsilon},p_{f},p_{\xi}\right)=\frac{p_{\epsilon}}{N}\Delta\epsilon^{2}+\frac{p_{f}}{3N}\sum_{i}\left|\Delta{F}_{i}\right|^{2}+\frac{p_{\xi}}{9N}\|\Delta\xi\|^{2},(9)">

其中 <img src="https://latex.codecogs.com/svg.image?\Delta\epsilon">, <img src="https://latex.codecogs.com/svg.image?\Delta{F}_{i}">, 和 <img src="https://latex.codecogs.com/svg.image?\Delta\xi"> 分别表示能量、力和维里的方均根误差 (RMSE) . 在训练的过程中, 前置因子 <img src="https://latex.codecogs.com/svg.image?p_{\epsilon}">, <img src="https://latex.codecogs.com/svg.image?p_{f}">, 和 <img src="https://latex.codecogs.com/svg.image?p_{\xi}"> 由公式

<img src="https://latex.codecogs.com/svg.image?p(t)=p^{\text{limit}}\left[1-\frac{r_{l}(t)}{r_{l}^{0}}\right]+p^{\text{start}}\left[\frac{r_{l}(t)}{r_{l}^{0}}\right],(10)">

决定，其中 <img src="https://latex.codecogs.com/svg.image?r_{l}(t)"> 和 <img src="https://latex.codecogs.com/svg.image?r_{l}^{0}"> 分别表示在训练步数为 <img src="https://latex.codecogs.com/svg.image?t"> 和训练步数为0 时的学习率。<img src="https://latex.codecogs.com/svg.image?r_{l}(t)"> 的定义为

<img src="https://latex.codecogs.com/svg.image?r_{l}(t)=r_{l}^{0}\times%20d_{r}^{t/d_{s}},(11)">

其中 <img src="https://latex.codecogs.com/svg.image?d_{r}"> 和 <img src="https://latex.codecogs.com/svg.image?d_{s}"> 分别表示学习衰减率以及衰减步数。学习衰减率 <img src="https://latex.codecogs.com/svg.image?d_{r}"> 要严格小于1。
