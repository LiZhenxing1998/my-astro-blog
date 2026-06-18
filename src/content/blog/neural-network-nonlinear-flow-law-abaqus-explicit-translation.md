---
title: '使用神经网络在 Abaqus Explicit FEM 代码中高效实现非线性流动定律'
description: '一篇论文翻译整理，介绍如何用人工神经网络在 Abaqus Explicit 中高效实现非线性流动定律。'
publishDate: '2026-06-18'
tags:
  - Abaqus
  - 神经网络
  - 本构模型
  - 论文翻译
language: '中文'
draft: false
comment: true
---

# 使用神经网络在 Abaqus Explicit FEM 代码中高效实现非线性流动定律

Olivier Pantale, Pierre Tize Mha, Amevi Tongne  
生产工程实验室，INP/ENIT，图卢兹大学，47 Av d'Azereix, Tarbes, France 65016

> 译者说明：本译文依据本地 PDF 文本抽取结果翻译整理。公式、变量名、模型名、软件名和参考文献编号尽量保留原文形式；个别 PDF 抽取导致的断行、连字和页眉页码已在译文中合并处理。参考文献题录保留英文原题名，便于检索。

## 摘要

机器学习技术正越来越多地用于科学应用中的材料行为预测，并且相较于传统数值方法具有显著优势。在本文中，作者将人工神经网络（Artificial Neural Network, ANN）模型用于有限元公式中，用以定义金属材料的流动定律，使其成为塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 的函数。首先，文章介绍神经网络的一般结构及其工作方式，并重点讨论网络在未进行先验学习的情况下，推导流动定律相对于模型输入变量的导数的能力。为了验证所提出模型的鲁棒性和准确性，作者以 42CrMo4 钢的 Johnson-Cook 行为定律解析形式为参照，对几种网络架构的性能进行了比较和分析。随后，在选择了一个具有 2 个隐藏层的 ANN 架构之后，作者以 `VUHARD` 子程序的形式，将该模型实现到 Abaqus Explicit 计算代码中。最后，通过两个测试算例，即圆棒颈缩和 Taylor 冲击试验的数值模拟，展示了所提出模型的预测能力。结果表明，ANN 在有限元代码中替代 Johnson-Cook 行为定律解析形式方面具有很强的能力，同时在数值模拟时间方面相对于经典方法仍保持竞争力。

关键词：人工神经网络；本构行为；有限元方法；数值实现；Johnson-Cook 流动定律

## 1. 引言

成形过程、机加工过程，或者结构在动态载荷和冲击作用下行为的数值模拟，都需要使用特定的材料行为定律。这些行为定律中的参数通常通过基于 Taylor 冲击、Hopkinson 杆或 Gleeble 热机械模拟器的试验来识别。行为定律的选择通常取决于有限元代码中是否已有相应模型；若没有，则取决于是否可以通过用户子程序实现。

本文的研究基于有限元代码 Abaqus Explicit。该软件允许通过 FORTRAN 子程序 `VUMAT` 或 `VUHARD` 来定义用户材料行为定律，类似于 Duc-Toan 等 [3] 以及较近的 Ming 等 [4] 所提出的工作。通常的流程是在文献中已有的行为定律形式中选定一种数学形式，例如 Johnson-Cook、Zerilli-Armstrong 等，然后根据实验试验结果，通过回归方法识别所选定律的参数。

在多数情况下，材料在高温和高应变率下的行为高度非线性，许多因素对流动应力的影响同样是非线性的。这会降低常用回归方法的预测精度，并限制其应用范围。此外，这类本构方程的选择、开发和数值实现都非常耗时。人工智能技术为行为定律研究提供了新的推进途径，可以更好地识别这类定律。例如，Versino 等 [5] 使用基于符号回归的机器学习技术开发数据驱动本构模型。获得一个流动方程以及它的解析导数，可以使用包含雅可比矩阵的迭代求解器，例如 Newton-Raphson 格式，从而得到更高阶的收敛。此后，符号回归技术也被其他作者采用，例如 Bomarito 等 [6]，Park 等 [7] 使用约束符号回归技术，Nassr 等 [8] 使用演化多项式回归。

在这种背景下，寻找一种能够消除实验试验与数值模拟之间若干中间步骤的方法，以简化计算链条，是很自然的思路。从这一角度看，深度学习的最新进展构成了一条值得研究的途径。其基本思想是：用人工神经网络（ANN）取代解析公式，来计算材料流动应力 $\sigma$，其中 $\sigma$ 是塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 的函数。该神经网络仅根据试验得到的实验数据来训练，以再现所研究材料的行为，而不对假定流动定律的解析形式作任何预设。因此，为了在 FEM 代码中实现行为定律，就不再需要预先假设其解析形式。

人工神经网络和深度学习在当今社会中变得越来越重要，其应用领域也越来越广。神经网络在 20 世纪 90 年代初曾经历过一次热潮，随后在 20 世纪末关注度下降；而如今，它们又以深度学习之名重新受到广泛关注，并引发了巨大的媒体热度。神经网络在科学和物理中的使用现在已经十分普遍，尤其是因为目前有高效工具可用于编写 ANN，例如 TensorFlow [9] 等广泛可用的库。深度学习最受关注的应用主要涉及医学诊断、机器人、图像识别和语言识别，但其应用远不止于此。现在，各门科学都可以使用这些技术，热机械数值模拟也包括在内。

人工神经网络能够解决传统计算方法难以概念化的问题。不同于基于回归方法的经典方法，ANN 不需要知道其试图再现的模型的数学形式。ANN 从训练数据中学习，并且只需要知道一系列输入和输出值，就能在不预设这些量的性质和相互关系的情况下，再现某个模型的行为。Hornik 等 [10] 在 1989 年严格证明了前馈神经网络是一类通用逼近器，这扩展了 Minsky 和 Papert [11] 二十年前提出的工作；Minsky 和 Papert 曾证明，简单的两层感知机无法有效表示或逼近某些很窄、很特殊类别之外的函数。ANN 具有调节、记忆和预测能力，并且相比于实现一个本构方程的方法表现更好。因此，ANN 现在能够为材料行为建模提供新的途径，并且近年来已经成功应用于一些金属和合金本构关系的预测。

ANN 用于路径相关塑性建模的适用性已经得到探索。相关文献综述可见 Gorji 等 [12] 关于循环神经网络的研究、Jamli 等 [13] 关于金属成形过程有限元分析中 ANN 应用的研究，以及 Jiao 等 [14] 关于 ANN 对超材料及其表征适用性的研究。需要区分基于 ANN 的硬化模型和基于 ANN 的本构模型。过去三十年中，许多研究者都研究过这两类方法。Ghaboussi 等 [15] 发表了一篇开创性论文，提出了一种基于 ANN 的本构模型，用于单调双轴加载和平面混凝土循环单轴加载，并成功预测了双轴加载条件下的多种加载路径。他们随后在文献 [16, 17] 中通过引入自适应神经网络和自推进神经网络改进了网络架构，使网络结构在训练阶段演化，从而利用整体载荷-挠度响应更好地学习材料复杂的应力-应变行为。本文采用的是一种基于 ANN 的硬化模型，其中 ANN 计算的材料流动应力评估与 Radial-Return 类型的积分格式相结合。

Lin 等 [18] 开发了一个神经网络，用于预测 42CrMo4 钢在 Gleeble 热机械装置热压缩试验中的流动应力，并显示出实验结果与预测结果之间很好的相关性。若能进一步扩展到该方法的数值实现，将会更有价值。Javadi 等 [19] 使用神经网络捕捉复杂材料行为，并将反向传播神经网络嵌入有限元模型中。Lu 等 [20] 对 Al-Cu-Mg-Ag 合金行为的建模进行了比较研究，比较对象包括基于 Zener-Hollomon 参数的本构方程和神经网络；结果也表明，基于 ANN 的模型比本构方程具有更好的预测能力。Ashtiani 等 [21] 将 ANN 的预测能力与基于 Johnson-Cook、Arrhenius 和应变补偿 Arrhenius 等行为定律的传统方法进行比较，并得出结论：神经网络在预测 Al-Cu-Mg-Pb 合金热变形行为方面具有更好的效率和精度。Ashtiani 等 [21] 还表明，训练良好的 ANN 能够有效克服 Johnson-Cook 或 Arrhenius 等解析本构行为中物理信息不足的问题。

Ali 等 [22] 提出一种 ANN 模型，并将其与率相关晶体塑性有限元方法耦合，用于模拟 AA6063-T6 材料在简单剪切和拉伸条件下的应力-应变行为及其微观组织演化。Stoffel 等 [23, 24] 将 ANN 应用于冲击波加载板的复杂结构变形问题，其中涉及几何非线性和物理非线性。Li 等 [25] 为 Abaqus 实现了一个 `VUMAT` 子程序，其中参数通过解析公式和反向传播算法相结合的方式识别。最近，Huang 等 [26] 开发了一个神经网络模型，用于预测 Ti-6Al-4V 合金的流动应力和微观组织演化；该研究同样显示出该方法相对于 Arrhenius 行为模型的优越性，尤其是 ANN 能够在整个变形范围内预测流动应力。Abueidda 等 [27] 还应用时间卷积网络来预测一类胞状材料的历史相关响应。Masi 等 [28] 提出了基于热力学的 ANN，以减少神经网络预测中的物理不一致性。他们展示了 TANN 在弹塑性材料建模中的广泛适用性，包括超塑性和亚塑性模型。如 Knight 等 [29] 所指出，神经网络架构的演化并不是进步的唯一方向；用于实现 ANN 的硬件架构也在不断演进，未来同样需要考虑这一点。

第 2 节介绍深度学习的主要基础，详细描述神经网络结构及其工作方程。正如第 4 节关于流动定律数值实现部分将会看到的，在有限元代码 Abaqus Explicit 中编写神经网络需要确定流动应力 $\sigma$ 对塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 的 3 个导数。这些导数的确定将是第 2 节第二部分的主题。第 3 节详细介绍 ANN 对 42CrMo4 钢 Johnson-Cook 型行为定律的学习，并说明网络结构对应力和导数评估精度的影响。第 4 节介绍神经网络在 Abaqus 有限元代码中的数值实现，即以 FORTRAN `VUHARD` 子程序的形式实现，并给出验证所提出方法的数值测试算例。最后给出结论和展望。

## 2. 人工神经网络设置

本节简要介绍与本文工作相关的反向传播和前向传播人工神经网络（ANN）的基本概念。本文采用的总体架构是多层前馈网络，根据 Hornik 等 [10] 的观点，它可以看作一种通用逼近器。所提出的神经网络用于逼近非线性函数。神经网络的概念是：通过定义一组神经元来模拟人脑中的信息流动，这些神经元排列在不同层中，并根据输入定义产生输出结果的函数。这些神经元从一层到另一层相互连接，并随着网络训练和学习新概念而持续提高预测能力 [30]。图 1 给出了具有多个隐藏层的 ANN 总体架构，其中神经元被设置在不同层中，从第一个隐藏层到输出层，具有以下特点：

**图 1：多层人工神经网络架构**

- 第一层称为输入层。在本文关于本构定律的应用中，它由 3 个输入组成。该层并不由后文所定义的神经元构成，而是收集输入信息：分别为塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$。
- ANN 至少包含一个隐藏层。本文只考虑具有一个隐藏层和两个隐藏层的 ANN，其中隐藏层包含可变数量的神经元。
- 最后一层称为输出层。本文中它只包含一个神经元，提供 von Mises 流动应力 $\sigma$ 的值。
- 神经元不与同一层中的其他神经元相连，只与前一层的输出和后一层的输入相连。

在图 1 中，作者将隐藏层中所有神经元的求和部分 $\mathbf{y}^{(k)}$ 与激活部分 $\hat{\mathbf{y}}^{(k)}$ 分开表示。关于符号，需要注意，后续方程中许多量采用向量形式表示，即在符号上加箭头，但即便如此，它们并不一定是严格数学意义上的向量。实际上，这些量是用于存储数据的一维竖向数组，在视觉上对应向量的分量。在所提出的架构中，输出层神经元没有激活函数，这也是回归问题中通常采用的做法，而本文正属于回归问题。数据逐层流动，直到得到 ANN 的输出。除第一层之外，ANN 中的每一层无论包含多少个神经元 $n$，都具有由可变数量条目 $m$ 组成的输入，以及由固定数量条目 $n$ 组成的输出。在所谓前向传播算法中，某一层的输出成为下一层的输入。

### 2.1 神经网络控制方程

#### 2.1.1 输入层

根据图 1，输入层定义为：

$$
\mathbf{x} = [x_1, x_2, x_3]^T
$$

在本文应用中，$\mathbf{x}$ 有 3 个分量 $x_i$，分别与塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 相关。如第 2.1.4 节所述，输入变量 $x_i$ 会被归一化，以避免优化过程中的病态问题，主要原因是它们代表不同物理量，数值幅值差异很大。

#### 2.1.2 隐藏层

任意隐藏层 $k$ 包含 $n$ 个神经元，它对紧邻前一层 $(k-1)$ 的输出 $\mathbf{x}$ 进行加权求和。若前一层包含 $m$ 个神经元，则有：

$$
y_i^{(k)}=\sum_{j=1}^{m}w_{ij}^{(k)}x_j+b_i^{(k)}
\tag{1}
$$

其中，$y_i^{(k)}$ 是第 $k$ 层第 $i$ 个神经元的节点值，$w_{ij}^{(k)}$ 是第 $k$ 层第 $i$ 个神经元与第 $(k-1)$ 层第 $j$ 个神经元之间的权重参数，$b_i^{(k)}$ 是第 $k$ 层第 $i$ 个神经元对应的偏置。这些权重和偏置就是 ANN 的训练参数，需要在第 3 节所述训练过程中调整。使用矩阵记号，式 (1) 可重写为：

$$
\mathbf{y}^{(k)}=\mathbf{w}^{(k)}\cdot\mathbf{x}+\mathbf{b}^{(k)}
\tag{2}
$$

其中，$\mathbf{y}^{(k)}=[y_1^{(k)},y_2^{(k)},\ldots,y_n^{(k)}]^T$ 包含第 $k$ 层求和操作得到的节点值，$\mathbf{b}^{(k)}=[b_1^{(k)},b_2^{(k)},\ldots,b_n^{(k)}]^T$ 是第 $k$ 层的节点偏置，$\mathbf{w}^{(k)}$ 是第 $k$ 层的 $[n\times m]$ 权重矩阵：

$$
\mathbf{w}^{(k)}=
\begin{bmatrix}
w_{11}^{(k)} & w_{12}^{(k)} & \cdots & w_{1m}^{(k)}\\
w_{21}^{(k)} & w_{22}^{(k)} & \cdots & w_{2m}^{(k)}\\
\vdots & \vdots & \ddots & \vdots\\
w_{n1}^{(k)} & w_{n2}^{(k)} & \cdots & w_{nm}^{(k)}
\end{bmatrix}
\tag{3}
$$

任意隐藏层 $k$ 的训练参数总数 $N$ 是权重参数数目与偏置参数数目的和，因此 $N=n(m+1)$。在式 (2) 定义的求和操作之后，隐藏层 $k$ 中的每个神经元都通过激活函数 $f^{(k)}$ 给出输出值 $\hat{\mathbf{y}}^{(k)}$：

$$
\hat{y}_i^{(k)}=f^{(k)}\left(y_i^{(k)}\right),
\qquad
\hat{\mathbf{y}}^{(k)}=f^{(k)}\left(\mathbf{y}^{(k)}\right)
\tag{4}
$$

文献中有许多激活函数可供选择，其选择主要取决于 ANN 的具体应用。在本文中，这些激活函数必须可导，这一点非常重要，因为需要计算 von Mises 应力 $\sigma$ 对塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 的导数。针对本文应用，作者选择测试两个常用函数：

- Sigmoid 激活函数 $\operatorname{sig}(x)$：

$$
\operatorname{sig}(x)=\frac{1}{1+\exp(-x)},
\qquad
\operatorname{sig}'(x)=\frac{\exp(x)}{(1+\exp(x))^2}
\tag{5}
$$

- 双曲正切激活函数 $\tanh(x)$：

$$
\tanh(x)=\frac{\exp(x)-\exp(-x)}{\exp(x)+\exp(-x)},
\qquad
\tanh'(x)=1-\tanh^2(x)
\tag{6}
$$

选择这两个函数的主要依据是它们的导数表达式简单，能够得到相对紧凑的表达。第 3 节将比较这两类激活函数的性能。如果当前层之后还存在另一个隐藏层 $(k+1)$，则第 $k$ 层输出 $\hat{\mathbf{y}}$ 会作为第 $(k+1)$ 层的输入 $\mathbf{x}$。

#### 2.1.3 输出层

神经网络的输出 $s$ 由最后一个隐藏层 $l$ 的输出值计算得到。若最后一个隐藏层包含 $m$ 个神经元，则有：

$$
s=\sum_{j=1}^{m}w_j\hat{y}_j^{(l)}+b
\tag{7}
$$

其中，$b$ 是输出神经元的偏置，$w_i$ 是最后一个隐藏层与输出神经元之间的 $m$ 个权重参数。使用矩阵形式可写为：

$$
s=\mathbf{w}^T\cdot\hat{\mathbf{y}}^{(l)}+b
\tag{8}
$$

其中 $\mathbf{w}=[w_1,w_2,\ldots,w_m]^T$。如前所述，输出神经元没有激活函数，因此 $s$ 直接就是神经网络的输出。输出层训练参数总数为 $m+1$。对于一个具有两个隐藏层的神经网络，若第一隐藏层有 $m$ 个神经元，第二隐藏层有 $n$ 个神经元，则训练参数总数为：

$$
N=4m+n(m+2)+1
$$

#### 2.1.4 数据预处理与后处理

由于 ANN 被设置为处理幅值有限的数值，因此需要将塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 预处理到 $[0,1]$ 范围内，这与其他作者 [18, 20] 的做法一致。因此，ANN 的输入 $\mathbf{x}$ 根据本构流动定律按以下方式计算：

- 由于本构方程中的塑性变形率通常体现为塑性应变率的对数，作者先对塑性应变率进行预处理，即计算塑性应变率 $\dot{\varepsilon}_p$ 与参考应变率 $\dot{\varepsilon}_0$ 之比的自然对数 $\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)$。
- 然后，将变量 $x_i$ 归一化到 $[0,1]$ 范围，以避免病态系统，这与文献 [18, 20] 中许多作者的做法一致。

因此，输入 $\mathbf{x}$ 的 3 个分量由塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$ 和温度 $T$ 根据以下表达式得到：

$$
\begin{cases}
x_1=\dfrac{\varepsilon_p-[\varepsilon_p]_{\min}}
{[\varepsilon_p]_{\max}-[\varepsilon_p]_{\min}}\\[1.0em]
x_2=\dfrac{
\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)
-[\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)]_{\min}}
{[\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)]_{\max}
-[\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)]_{\min}}\\[1.0em]
x_3=\dfrac{T-[T]_{\min}}{[T]_{\max}-[T]_{\min}}
\end{cases}
\tag{9}
$$

其中，$[\,]_{\min}$ 和 $[\,]_{\max}$ 是相应场变量范围的上下边界。在 ANN 训练过程中，von Mises 应力 $\sigma$ 也会缩放到 $[0,1]$ 范围：

$$
s=\frac{\sigma-[\sigma]_{\min}}{[\sigma]_{\max}-[\sigma]_{\min}}
\tag{10}
$$

最终，von Mises 应力 $\sigma$ 可由 ANN 输出得到：

$$
\sigma=\left([\sigma]_{\max}-[\sigma]_{\min}\right)s+[\sigma]_{\min}
\tag{11}
$$

训练阶段使用的塑性应变、塑性应变率、温度、应力的 $[\,]_{\min}$ 和 $[\,]_{\max}$ 值，以及参考应变率 $\dot{\varepsilon}_0$，都应记录下来，以便后续在 Abaqus Explicit 代码中实现 ANN 时使用。这些值与隐藏层的权重 $\mathbf{w}^{(k)}$、偏置 $\mathbf{b}^{(k)}$，以及输出层的权重 $\mathbf{w}$ 和偏置 $b$ 一起构成神经网络的最终训练参数。学习阶段结束后，知道这些量便可以将完整神经网络从 Python 语言公式中提取出来，形成一个紧凑版本，不再包含反向传播学习机制，并用于 Abaqus Explicit FEM 代码中的 FORTRAN 实现。

#### 2.1.5 损失函数

在优化算法中，用于评价解质量的函数称为目标函数。在神经网络应用中，目标是最小化网络预测解时产生的误差。该误差通过测量神经网络计算得到的预测值 $\sigma_i$ 与参考值 $\sigma_i^y$ 之间的差异来评价。参考值通常来自实验；在本文中则来自解析方程。对于某个特定数据集，可以用多种方式定义误差，其中最著名的可能是平均均方根误差 $E_{\mathrm{RMS}}$：

$$
E_{\mathrm{RMS}}=
\sqrt{\frac{1}{N}\sum_{i=1}^{N}\left(\sigma_i-\sigma_i^y\right)^2}
\tag{12}
$$

其中，$N$ 是训练批次中的总数据数量。

### 2.2 导数计算

将神经网络通过 `VUMAT` 子程序或直接通过 Abaqus Explicit 的 `VUHARD` 子程序实现到 Ming 等 [4] 提出的 Radial-Return 算法中时，ANN 不仅需要返回式 (11) 给出的 von Mises 等效应力 $\sigma$，还必须在没有经过导数训练的情况下，返回 3 个导数：$\partial\sigma/\partial\varepsilon_p$、$\partial\sigma/\partial\dot{\varepsilon}_p$ 和 $\partial\sigma/\partial T$。因此，ANN 并不是有 4 个输出，而只有 1 个输出；计算这些导数的能力必须来自网络自身。因此，需要仅利用所提出的 ANN 架构来计算这 3 个导数。

一个直接但不推荐的方法，是用数值差分计算 $\sigma$ 对 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的导数：

$$
\frac{\partial\sigma(x)}{\partial x}
=\frac{\sigma(x+\delta x)-\sigma(x)}{\delta x}
\tag{13}
$$

其中，$\delta x$ 是施加在 3 个变量 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 中某一个变量上的小增量，例如 $\delta x=10^{-6}$。为了计算流动应力并近似 3 个导数，需要对 ANN 计算 4 次，这会相当耗时。

另一种方法是计算 $\mathbf{s}'=[s'_1,s'_2,s'_3]^T$，其中包含式 (8) 定义的 ANN 输出相对于输入 $\mathbf{x}$ 的 3 个导数。该解析计算取决于隐藏层数量，以及所使用的激活函数类型。下文给出了 1 个和 2 个隐藏层的结果，并考虑 $\tanh$ 和 $\operatorname{sig}$ 两类函数。$\mathbf{s}'=\partial s/\partial\mathbf{x}$ 包含 $s$ 分别对 $x_1$、$x_2$ 和 $x_3$ 的导数。根据链式法则推导，当 ANN 有 1 个隐藏层且使用 $\tanh$ 激活函数时，$\mathbf{s}'$ 形式为：

$$
\mathbf{s}'=
\left(\mathbf{w}^{(1)}\right)^T\cdot
\left[\mathbf{w}-\mathbf{w}\circ\tanh^2\left(\mathbf{y}^{(1)}\right)\right]
\tag{14}
$$

其中，$\mathbf{y}^{(1)}$ 由式 (2) 在 $k=1$ 时给出，$\circ$ 表示逐元素乘积，即 Hadamard 积。Hadamard 积是一种二元运算，对尺寸相同的矩阵 $A$ 和 $B$ 生成同尺寸矩阵 $C$，其中每个元素满足 $C_i=A_iB_i$。作用于 $\mathbf{y}^{(1)}$ 的 $\tanh$ 运算只是对 $\mathbf{y}^{(1)}$ 的每个分量逐一计算，因为它实际上不是一个实数，而是一个竖向堆叠的一维实数数组。若 1 个隐藏层使用 $\operatorname{sig}$ 激活函数，则得到：

$$
\mathbf{s}'=
\left(\mathbf{w}^{(1)}\right)^T\cdot
\left[
\mathbf{w}\circ
\frac{\exp\left(-\mathbf{y}^{(1)}\right)}
{\left(1+\exp\left(-\mathbf{y}^{(1)}\right)\right)^2}
\right]
\tag{15}
$$

当隐藏层数量增加时，导数复杂度也会增加。对于两个隐藏层且两层都使用 $\tanh$ 激活函数的情况，可得：

$$
\mathbf{s}'=
\left(\mathbf{w}^{(1)}\right)^T\cdot
\left[
\left(\mathbf{w}^{(2)}\right)^T\cdot
\left(\mathbf{w}-\mathbf{w}\circ\tanh^2\left(\mathbf{y}^{(2)}\right)\right)
\circ
\left(1-\tanh^2\left(\mathbf{y}^{(1)}\right)\right)
\right]
\tag{16}
$$

最后，对于两个隐藏层且两层都使用 $\operatorname{sig}$ 激活函数的情况，可得：

$$
\mathbf{s}'=
\left(\mathbf{w}^{(1)}\right)^T\cdot
\left[
\left(\mathbf{w}^{(2)}\right)^T\cdot
\left(
\mathbf{w}\circ
\frac{\exp\left(-\mathbf{y}^{(2)}\right)}
{\left(1+\exp\left(-\mathbf{y}^{(2)}\right)\right)^2}
\right)
\circ
\frac{\exp\left(-\mathbf{y}^{(1)}\right)}
{\left(1+\exp\left(-\mathbf{y}^{(1)}\right)\right)^2}
\right]
\tag{17}
$$

根据隐藏层数量和激活函数类型，由式 (14) 到式 (17)，并结合第 2.1.4 节中式 (9) 和式 (10) 对 $\sigma$、$\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的预处理与后处理，最终可以得到流动应力 $\sigma$ 分别对 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的导数：

$$
\begin{cases}
\dfrac{\partial\sigma}{\partial\varepsilon_p}
=s'_1
\dfrac{[\sigma]_{\max}-[\sigma]_{\min}}
{[\varepsilon_p]_{\max}-[\varepsilon_p]_{\min}}\\[1.1em]
\dfrac{\partial\sigma}{\partial\dot{\varepsilon}_p}
=\dfrac{s'_2}{\dot{\varepsilon}_p}
\dfrac{[\sigma]_{\max}-[\sigma]_{\min}}
{[\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)]_{\max}
-[\ln(\dot{\varepsilon}_p/\dot{\varepsilon}_0)]_{\min}}\\[1.1em]
\dfrac{\partial\sigma}{\partial T}
=s'_3
\dfrac{[\sigma]_{\max}-[\sigma]_{\min}}
{[T]_{\max}-[T]_{\min}}
\end{cases}
\tag{18}
$$

需要指出，无论采用何种方法计算神经网络输出相对于输入的导数，无论是数值方法还是网络导数公式，得到的结果都是网络所代表数学函数对输入导数的一种近似。事实上，神经网络按照其构造本身就是对某个数学公式的逼近；正如 Nguyen 等 [31] 所示，神经网络也可以逼近其所再现数学函数的导数。因此，神经网络对输入的导数，与被映射数学函数对其参数的导数是相关的。

## 3. ANN 的训练与性能评估

为了评估所提出方法的性能，作者决定使用人工神经网络来再现 Johnson-Cook [32] 流动定律的行为。这是因为 Johnson-Cook 是高应变率变形过程模拟中最常用的流动定律之一，并且已经在许多有限元代码中实现，例如 Abaqus。用神经网络再现 Johnson-Cook 定律当然不是本文的最终目的；这样做只是为了在数值上验证神经网络能够考虑非线性行为，并且能够以精确方式度量神经网络的预测误差。一般形式 $\sigma_y(\varepsilon_p,\dot{\varepsilon}_p,T)$ 为：

$$
\sigma_y=
\left(A+B\varepsilon_p^n\right)
\left[1+C\ln\left(\frac{\dot{\varepsilon}_p}{\dot{\varepsilon}_0}\right)\right]
\left[
1-\left(\frac{T-T_0}{T_m-T_0}\right)^m
\right]
\tag{19}
$$

其中，$\dot{\varepsilon}_0$ 是参考应变率，$T_0$ 和 $T_m$ 分别为参考温度和材料熔化温度，$A$、$B$、$C$、$n$ 和 $m$ 是 5 个本构流动定律参数。通常需要通过反向识别流程确定这些参数，例如 Dalverny 等 [33] 基于 Taylor 冲击试验提出的方法。

Johnson-Cook 流动应力 $\sigma_y$ 对 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的 3 个解析导数可写为：

$$
\begin{cases}
\dfrac{\partial\sigma_y}{\partial\varepsilon_p}
=nB\varepsilon_p^{n-1}
\left[1+C\ln\left(\dfrac{\dot{\varepsilon}_p}{\dot{\varepsilon}_0}\right)\right]
\left[
1-\left(\dfrac{T-T_0}{T_m-T_0}\right)^m
\right]\\[1.2em]
\dfrac{\partial\sigma_y}{\partial\dot{\varepsilon}_p}
=\dfrac{C}{\dot{\varepsilon}_p}
\left(A+B\varepsilon_p^n\right)
\left[
1-\left(\dfrac{T-T_0}{T_m-T_0}\right)^m
\right]\\[1.2em]
\dfrac{\partial\sigma_y}{\partial T}
=-\dfrac{m\left(A+B\varepsilon_p^n\right)}{T_m-T_0}
\left[1+C\ln\left(\dfrac{\dot{\varepsilon}_p}{\dot{\varepsilon}_0}\right)\right]
\left(\dfrac{T-T_0}{T_m-T_0}\right)^{m-1}
\end{cases}
\tag{20}
$$

按照 Ming 等 [4] 提出的通常做法，在 Abaqus FEM 代码中实现一个新的本构定律，需要在 `VUHARD` FORTRAN 子程序中编写式 (19) 定义的流动应力评估，以及式 (20) 给出的流动应力导数。

### 3.1 训练数据与测试数据的生成

本文选取 42CrMo4 钢作为研究材料，其材料参数采用 Sattouf 等 [34] 提出的数值，并列于表 1。为了训练并验证 ANN，作者利用 Python 程序根据式 (19) 和式 (20) 生成两个不同的数据集。

**表 1：42CrMo4 钢材料性能 [35]**

| 参数 | 数值 |
|---|---:|
| $E$ | 206.9 GPa |
| $\nu$ | 0.29 |
| $A$ | 806 MPa |
| $B$ | 614 MPa |
| $C$ | 0.0089 |
| $n$ | 0.168 |
| $m$ | 1.1 |
| $\dot{\varepsilon}_0$ | 1 s^-1 |
| T0 | 20 deg C |
| Tm | 1540 deg C |
| $\rho$ | 7830 kg/m3 |
| $C_p$ | 460 J/kg deg C |
| $\alpha$ | 12.3 x 10^-6 / deg C |
| $\eta$ | 0.9 |

第一个数据集是训练集，包含 2520 个数据点。这些数据点由如下组合定义：$\varepsilon_p\in[0,1]$ 范围内的 70 个等距值，6 个塑性应变率 $\dot{\varepsilon}_p\in[1,10,50,500,5000,50000]$，以及 6 个温度 $T\in[20,100,200,300,400,500]$。

第二个数据集是测试集，包含 5000 个随机生成的数据点，范围为 $\varepsilon_p\in[0,1]$、$\dot{\varepsilon}_p\in[1,50000]$ 和 $T\in[20,500]$。该数据集不参与训练阶段，只在第 3.3 节的验证中使用。

两个数据集都包含塑性应变 $\varepsilon_p$、塑性应变率 $\dot{\varepsilon}_p$、温度 $T$，以及由式 (19) 计算得到的流动应力 $\sigma_y$。第二个数据集还包含由式 (20) 计算得到的 3 个导数。需要记住的是，本文提出的神经网络必须能够替代行为定律的解析形式，从而能够基于热机械试验得到的实验数据进行数值模拟。在这些试验中，可以获得温度、应变、应变率和应力；但无法获得应力相对于 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的导数。因此，在常规使用中，不可能把导数加入神经网络训练的目标函数。

### 3.2 人工神经网络的训练

训练 ANN，就是寻找第 2.1 节中定义的所有训练参数 $\mathbf{w}^{(k)}$、$\mathbf{b}^{(k)}$、$\mathbf{w}$ 和 $b$ 的最佳取值，以减少 ANN 计算流动应力时产生的误差。该训练过程使用训练集，ANN 使用 TensorFlow Python 库 [36] 实现。训练过程基于自适应矩估计优化器 ADAM [37]。作者使用了前文介绍的两种激活函数，即 $\operatorname{sig}$ 和 $\tanh$，并使用一个或两个隐藏层，隐藏层中的神经元数量可变。

所有模型均根据其组成方式命名。例如，`3-x-1-tanh` 表示一个具有 1 个隐藏层的 ANN，使用 $\tanh$ 激活函数，隐藏层包含 $x$ 个神经元；`3-x-y-1-sig` 表示一个具有 2 个隐藏层的 ANN，两层都使用 $\operatorname{sig}$ 激活函数，第 1 层包含 $x$ 个神经元，第 2 层包含 $y$ 个神经元。所有模型都训练了相同的迭代次数，即 50000 次。所有模型的训练时间大致相同，在一台 Dell XPS 13 笔记本电脑上约为 1 小时。

### 3.3 人工神经网络性能分析

为了说明 ANN 的效率，图 2 和表 2 给出了训练 6 个不同模型后得到的一些结果，其中包括 2 个单隐藏层模型和 4 个双隐藏层模型。本文研究中获得了更多结果，但这里只展示这 6 个模型，以说明总体趋势。

**图 2：不同 ANN 精度比较**

图 2 显示了式 (12) 定义的 $E_{\mathrm{RMS}}$ 在训练过程最后 5% 迭代中的平均值柱状图，从而给出 ADAM 算法整体收敛情况的印象。表 2 中，$N$ 表示 ANN 内部参数数量。$\Delta$ 值表示平均绝对相对误差，即 $E_{\mathrm{AAR}}$，定义为：

$$
\Delta_{\square}=
\frac{1}{N}\sum_{i=1}^{N}
\left|
\frac{\square_i^e-\square_i^p}{\square_i^e}
\right|
\tag{21}
$$

其中，$\Box_i^e$ 是由式 (19) 和式 (20) 得到的解析精确值，$\Box_i^p$ 是神经网络根据式 (11) 和式 (18) 计算得到的同一物理量的 ANN 预测值。

**表 2：训练阶段 ANN 的整体性能分析**

| 模型 | $N$ | $E_{\mathrm{RMS}}\times 10^{-7}$ | $\Delta\sigma$ | $\Delta(\partial\sigma/\partial\varepsilon_p)$ | $\Delta(\partial\sigma/\partial\dot{\varepsilon}_p)$ | $\Delta(\partial\sigma/\partial T)$ |
|---|---:|---:|---:|---:|---:|---:|
| 3-7-4-1-tanh | 65 | 6.32 | 0.038% | 1.977% | 0.792% | 0.556% |
| 3-15-1-tanh | 78 | 3.46 | 0.039% | 1.506% | 0.269% | 0.371% |
| 3-15-7-1-tanh | 180 | 1.71 | 0.030% | 0.519% | 0.380% | 0.408% |
| 3-15-1-sig | 78 | 2.00 | 0.030% | 0.686% | 0.521% | 0.675% |
| 3-7-4-1-sig | 65 | 1.27 | 0.024% | 0.670% | 0.415% | 0.499% |
| 3-15-7-1-sig | 180 | 0.68 | 0.011% | 0.247% | 0.199% | 0.256% |

由表 2 可见，所提出神经网络的整体性能非常好。对于性能最好的网络，流动应力评估误差约为 0.01%；对于所有测试网络，误差均不超过 0.04%。导数评估效果也很好：性能最好的网络导数误差约为 0.2%；而对于 $\partial\sigma/\partial\varepsilon_p$ 项，误差仍低于 2.0%；对于 $\partial\sigma/\partial\dot{\varepsilon}_p$ 项，误差低于 0.8%；对于 $\partial\sigma/\partial T$ 项，误差低于 0.6%。

不出意料，从整体上看，流动应力评估误差比导数评估误差约低一个数量级。这很容易解释，因为网络训练时只最小化流动应力误差。尽管如此，即使从未针对导数计算进行优化，得到的导数结果依然很好。

图 3 显示了 `3-15-7-1-sig` ANN 模型在流动应力和 3 个导数评估方面随迭代次数变化的收敛情况。从图中可以看到，ANN 对 $\sigma$ 的评估收敛速度快于对导数的评估收敛速度。前文提到的应力与导数之间约 10 倍的误差因子，在图中清晰可见。如果约束应力计算误差在学习算法迭代 10000 次后不再变化，导数评估误差则需要多得多的迭代才能收敛。因此，在流动应力收敛准则已经满足时，仍有必要继续训练这类模型，以使导数收敛。此时可能会遇到过学习问题，因此有必要根据具体应用尽可能准确地确定神经网络规模。

**图 3：`3-15-7-1-sig` ANN 预测值收敛情况**

## 4. 神经网络在 Abaqus Explicit 中的实现

本节介绍所提出 ANN 本构流动定律的数值实现。当第 2 节提出的 ANN 网络按照第 3 节完成训练后，就可以用它计算流动应力及其 3 个导数。该实现通过为 Abaqus Explicit 有限元代码编写 `VUHARD` 子程序完成，其思路类似于 Jansen van Rensburg 等 [2] 提出的方法。该 `VUHARD` 子程序用于 Radial-Return 算法内部，如图 4 所示，用于计算流动应力 $\sigma_y$ 及其导数 $d\sigma_y/d\Gamma$，该导数用于两个量 $\gamma(\Gamma)$ 和 $\gamma'(\Gamma)$ 的表达式中。根据文献 [4]，有：

$$
\begin{aligned}
\frac{d\sigma_y}{d\Gamma}
&=
\frac{\partial\sigma_y}{\partial\varepsilon_p}
\frac{d\varepsilon_p}{d\Gamma}
+
\frac{\partial\sigma_y}{\partial\dot{\varepsilon}_p}
\frac{d\dot{\varepsilon}_p}{d\Gamma}
+
\frac{\partial\sigma_y}{\partial T}
\frac{dT}{d\Gamma}\\
&=
\sqrt{\frac{2}{3}}
\left(
\frac{\partial\sigma_y}{\partial\varepsilon_p}
+
\frac{1}{\Delta t}
\frac{\partial\sigma_y}{\partial\dot{\varepsilon}_p}
+
\frac{\eta\sigma_y}{\rho C_p}
\frac{\partial\sigma_y}{\partial T}
\right)
\end{aligned}
\tag{22}
$$

其中，$\Gamma$ 是 Simo 等 [38] 定义的 Radial-Return 算法中的一致性参数，$\Delta t$ 是时间增量，$\eta$ 是 Taylor-Quinney 系数，用于定义转化为热能的塑性功比例，$C_p$ 是比热系数，$\rho$ 是材料密度。因此，ANN 用于计算式 (22) 中涉及的流动应力 $\sigma_y$ 以及流动应力对 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的 3 个导数。图 4 中流程图中央的黄色块即为使用 ANN 的位置。由于该 ANN 位于一个 CPU 密集型循环的中心，并且该循环会涉及 FEM 模型的所有积分点，因此必须对其进行优化以减少计算时间，后文将对此进行介绍。关于 Radial-Return 算法的更多细节可见 Simo 等 [38]；关于该算法在 Abaqus Explicit 中实现的更多细节可见 Ming 等 [4]。Ming 等采用了同样的方法，不同之处在于他们在该文中用解析方式计算流动应力及其导数。

**图 4：用于计算最终应力的 Radial-Return 算法流程图**

所提出实现的验证通过若干基准测试完成：将 ANN 得到的结果与 Abaqus 原生 Johnson-Cook 定律结果进行比较，后者在下文称为 Built-in；同时也与 Ming 等 [4] 先前提出的解析 `VUHARD` 实现进行比较，后者在下文称为 Analytical。

### 4.1 神经网络的数值实现

神经网络的数值实现通过 Abaqus Explicit 代码中的 `VUHARD` 子程序完成。这是在该 FEM 代码中实现新的本构流动定律的一种直接方法：只需实现一个 FORTRAN 子程序，根据式 (11) 计算材料流动应力 $\sigma_y(\varepsilon_p,\dot{\varepsilon}_p,T)$，并根据式 (18) 计算其对 $\varepsilon_p$、$\dot{\varepsilon}_p$ 和 $T$ 的导数。在这种方法中，内置本构定律的主要部分用于给定时间增量内的应力时间积分，而用户提供的子程序被调用来计算硬化流动定律。

作者开发了一个 Python 程序，用于提取训练后神经网络的内部参数，包括网络架构、所有层的权重和偏置值 $\mathbf{w}$ 与 $b$ 等，并自动写出 FORTRAN 子程序。下文详细说明具有 2 个隐藏层且使用 $\operatorname{sig}$ 激活函数的神经网络实现，因为这是本文中较复杂的模型之一。

如第 2 节所述，神经网络包含两个主要部分，分别对应第 2.1 节和第 2.2 节：第一部分计算 von Mises 等效应力 $\sigma$，第二部分计算 3 个导数 $\partial\sigma/\partial\varepsilon_p$、$\partial\sigma/\partial\dot{\varepsilon}_p$ 和 $\partial\sigma/\partial T$。如果要实现一个具有 2 个隐藏层、使用 $\operatorname{sig}$ 激活函数、第一层含 $m$ 个神经元、第二层含 $n$ 个神经元的 ANN，则应使用式 (5) 计算应力，并使用式 (17) 计算导数。为了略微优化数值实现，并且由于流动应力和导数计算共享一些公共项，这些公共项可以在计算过程中存储并重复使用，因此作者将式 (5) 和式 (17) 拆分为若干子项 $z_a$ 到 $z_f$。

从式 (9) 定义输入 $\mathbf{x}$ 的值出发，可写为：

$$
\begin{cases}
z_{a,i}=
\exp\left(
-\sum_j w_{ij}^{(1)}x_j-b_i^{(1)}
\right),
\quad i\in[1,m],\ j\in[1,3]\\[0.8em]
z_{b,i}=1+z_{a,i},
\quad i\in[1,m]\\[0.8em]
z_{c,i}=
\exp\left(
-\sum_j \dfrac{w_{ij}^{(2)}}{z_{b,j}}-b_i^{(2)}
\right),
\quad i\in[1,n],\ j\in[1,m]
\end{cases}
\tag{23}
$$

其中，$z_{a,i}$、$z_{b,i}$ 和 $z_{c,i}$ 是式 (17) 中出现的三个项，分别对应 $\exp(-\mathbf{y}^{(1)})$、$1+\exp(-\mathbf{y}^{(1)})$ 和 $\exp(-\mathbf{y}^{(2)})$。随后，为了计算导数，将 $z_{a,i}$、$z_{b,i}$ 和 $z_{c,i}$ 组合起来计算完整的式 (17)：

$$
\begin{cases}
z_{d,i}=\dfrac{w_i z_{c,i}}{(1+z_{c,i})^2},
\quad i\in[1,n]\\[1.0em]
z_{e,i}=\dfrac{z_{a,i}}{z_{b,i}^2},
\quad i\in[1,m]\\[1.0em]
z_{f,i}=
\sum_j\left(w_{ji}^{(2)}z_{d,j}\right)z_{e,i},
\quad i\in[1,m],\ j\in[1,n]
\end{cases}
\tag{24}
$$

根据这些定义，神经网络输出可写为：

$$
s=\sum_i\left(\frac{w_i}{1+z_{c,i}}\right)+b,
\quad i\in[1,n]
\tag{25}
$$

3 个导数 $s'_i$ 可由下式得到：

$$
s'_i=\sum_j\left(w_{ji}^{(1)}z_{f,j}\right),
\quad i\in[1,3],\ j\in[1,m]
\tag{26}
$$

最后，使用式 (11) 和式 (18)，由式 (25) 和式 (26) 计算得到的 $s$ 和 $\mathbf{s}'$ 求得神经网络的 von Mises 等效应力 $\sigma$ 以及 3 个导数 $\partial\sigma/\partial\varepsilon_p$、$\partial\sigma/\partial\dot{\varepsilon}_p$ 和 $\partial\sigma/\partial T$。由于从式 (23) 到式 (26) 的实现是直接的，Python 接口使用循环在 FORTRAN 子程序中显式写出所有矩阵乘积，如图 5 所示。图 5 只给出了完整 FORTRAN 代码的一小部分，以说明其实现方式。对实现细节感兴趣的读者可参考 Software Heritage archive 网站 [39]，其中提供了本文工作的源代码。

```fortran
subroutine vuhard (
 + nblock, nElement, nIntPt, nLayer, nSecPt, lAnneal, stepTime,
 + totalTime, dt, cmname, nstatev, nfieldv, nprops, props,
 + tempOld, tempNew, fieldOld, fieldNew, stateOld, eqps, eqpsRate,
 + yield, dyieldDtemp, dyieldDeqps, stateNew)
include 'vaba_param.inc'
dimension nElement(nblock), props(nprops), tempOld(nblock),
 + fieldOld(nblock,nfieldv), stateOld(nblock,nstatev),
 + tempNew(nblock), fieldNew(nblock,nfieldv), eqps(nblock),
 + eqpsRate(nblock), yield(nblock), dyieldDtemp(nblock),
 + dyieldDeqps(nblock,2), stateNew(nblock,nstatev)
character*80 cmname
do k = 1, nblock
  xepsp = eqps(k)
  xdepsp = log(eqpsRate(k)) / 10.819778
  xtemp = (tempNew(k) - 20.0) / 480.0
  za0 = exp(0.171193*xepsp + 0.498235*xdepsp -1.572309*xtemp + 0.549710)
  za1 = exp(0.182183*xepsp + 0.279642*xdepsp -1.381547*xtemp + 1.007667)
  ...
  Yield(k) = 977.555715*y + 579.184642
  dyieldDeqps(k,1) = 977.555715*yd0
  dyieldDeqps(k,2) = 90.348959*yd1 / eqpsRate(k)
  dyieldDtemp(k) = 2.036574*yd2
end do
return
end
```

**图 5：用于 `3-15-7-1-sig` 模型的 VUHARD FORTRAN 子程序片段，该子程序使用 ANN 计算流动应力及其导数**

`VUHARD` 子程序使用 GNU gfortran 9.3.0 编译，并链接到主 Abaqus Explicit 可执行程序。所有基准测试均使用 Abaqus Explicit 2021 求解，运行环境为 Dell XPS 13 笔记本电脑，Ubuntu 20.04 64 位系统，16 GiB 内存，4 核 i7-10510U Intel 处理器。所有计算均使用 Abaqus 双精度选项，并以两个核心并行线程执行。为了减少后文展示模型数量，只选取表 2 中最后两个模型用于后续基准模拟。

### 4.2 圆棒颈缩基准测试

Ponthot 等 [40] 已经介绍过圆棒颈缩试验，该试验可用于评价非线性本构定律的性能。本文采用试样的轴对称四分之一模型，该模型也已由 Ming 等 [4] 给出，试样尺寸见图 6。作者在试样左端沿 $z$ 轴施加总位移 7 mm，同时假定该边界的径向位移为零。在相对的一侧，轴向位移受到约束，而径向位移自由。

网格由 400 个 `CAX4RT` 单元组成，即 4 节点双线性位移-温度、减缩积分、带沙漏控制单元。其中右侧总长度 1/3 区域设置了包含 200 个单元的加密区。FEM 模型为温度-位移耦合显式模型，即热传导解和力学解通过显式耦合同步获得，总模拟时间设为 $t=0.01\ \mathrm{s}$。

**图 6：圆棒颈缩数值模型**

图 7 显示了两个不同模型下变形圆棒的 von Mises 应力 $\sigma$ 云图：上半部分为 Built-In 模型，下半部分为 ANN `3-15-7-1-sig` 模型。从应力空间分布和最大值来看，两者差异很小。最大应力位于圆棒中心，因此作者在图 8 和图 9 中绘制了试样中心单元（图 6 右下角红色单元）的等效塑性应变 $\varepsilon_p$ 和 von Mises 应力 $\sigma$ 演化。图 8 和图 9 表明，Built-In 模型、Analytical 模型以及两种 ANN 模型给出的结果几乎相同；只有当伸长量大于 6 mm 时，ANN 模型与 Built-In、Analytical 模型之间的 von Mises 应力开始出现差异，这一点后文将进一步说明。

表 3 也证实了这一点。表 3 比较了四个模型在两个位移值下的结果，即 3.5 mm 和 7 mm，分别称为 mid 和 end。由表可见，在 mid 位移下，两种 ANN 模型得到的等效塑性应变 $\varepsilon_p$、von Mises 应力 $\sigma$ 和温度 $T$ 与 Built-In 和 Analytical 模型非常接近；而在 end 位移下则略有差异。这验证了所提出方法的有效性。

一个有意思的结果与表 3 中 end 位移下的等效塑性应变 $\varepsilon_p$ 和温度 $T$ 有关。可以看到，在 end 位移下，表 3 报告的塑性应变约为 2.1，而模型训练所使用的塑性应变范围为 $[0,1]$。此外，最高温度约为 587 deg C，而训练范围为 $[20,500]$。因此，所提出模型能够以较好精度对超出训练范围的数据进行外推。由表 3 以及图 8、图 9 可明显看出，当参数处于训练范围内时，所有模型给出的结果非常接近；当参数远离训练范围时，ANN 模型之间会出现小差异。在这里，伸长量大于 6 mm 时 $\varepsilon_p>1.7$。这表明，在一定程度上，并且在通常需要谨慎对待的前提下，本文 ANN 模型能够对训练范围之外的行为进行泛化；但神经网络通常在插值方面较可靠，在外推方面则不如插值可靠。如果希望减小训练范围之外预测值与真实值之间的差距，就需要在有数据可用的情况下扩大输入变量的训练范围。

表 3 还报告了同一模型重复运行 10 次后得到的计算时间和总增量数。可以看到，`3-7-4-1-sig` 模型的计算时间与 Built-in 模型相当，并且低于 Analytical 模型；而最复杂的 ANN 模型计算时间有所增加。在本文方法中，ANN 用于在 Radial-Return 算法中计算材料流动应力。它替代了基于某些解析表达式的流动应力及其导数评估。`VUHARD` 实现通过 FORTRAN 子程序调用、数据传输等过程打断了用于计算应力的内置自然优化算法，从而导致 CPU 时间增加。Built-in 与 Analytical 的 CPU 时间比较也说明了这一点。这就是为什么相对于 Built-in 模型无法获得计算时间降低的原因。因此，应将 ANN 模型的 CPU 性能与 Analytical 模型进行比较。由于 Johnson-Cook 解析行为定律相当简单，最复杂的 ANN 模型比解析模型更慢。然而，对于更复杂的行为定律，例如 Zhou 等 [41] 提出的某些修正 Johnson-Cook 定律，这一趋势将会反转，因为 Radial-Return 算法所需的解析导数评估会变得非常耗费 CPU。根据表 3 结果还可得出结论：`3-7-4-1-sig` 模型已经足以获得有价值的结果，并且相对于内置本构定律具有可比的 CPU 时间。

**图 7：圆棒颈缩在伸长 7 mm 时的 von Mises 应力 $\sigma$ 云图；上半部分为内置流动定律，下半部分为 ANN `3-15-7-1-sig` 流动定律**

**表 3：圆棒颈缩基准测试在位移 3.5 mm（mid）和 7 mm（end）下的结果比较**

| 模型 | Incr. | Time (s) | $\varepsilon_p$ mid | $\sigma_{mid}$ (MPa) | $T_{mid}$ (deg C) | $\varepsilon_p$ end | $\sigma_{end}$ (MPa) | $T_{end}$ (deg C) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 3-7-4-1-sig | 191768 | 29.98 | 0.51 | 1293.81 | 182.01 | 2.16 | 1064.43 | 587.78 |
| 3-15-7-1-sig | 194432 | 38.54 | 0.51 | 1293.97 | 181.52 | 2.03 | 1060.50 | 587.29 |
| Analytical | 200145 | 35.50 | 0.51 | 1293.59 | 182.47 | 2.16 | 1045.75 | 585.85 |
| Built-In | 199474 | 28.71 | 0.51 | 1293.76 | 180.36 | 2.14 | 1043.19 | 587.66 |

**图 8：圆棒颈缩中等效塑性应变 $\varepsilon_p$ 随位移变化**

**图 9：圆棒颈缩中 von Mises 应力 $\sigma$ 随位移变化**

### 4.3 Taylor 冲击基准测试

接下来，通过 Taylor 冲击试验 [42] 的模拟，在高变形率条件下验证所提出 ANN 子程序的性能。在该试验中，圆柱试样以给定初速度 $V_c=287\ \mathrm{m/s}$ 撞击刚性靶。圆柱高度为 32.4 mm，半径为 3.2 mm，如图 10 所示。试样左侧轴向位移受到约束，而径向位移自由，以表示弹体与靶之间无摩擦的理想接触。Taylor 圆柱试样采用 3D 四分之一模型，并用 4455 个 `C3D8T` 单元划分网格，即 8 节点三线性位移-温度单元。Taylor 冲击试验总模拟时间为 $t=80\ \mu\mathrm{s}$。

图 11 显示了两个模型下变形杆的等效塑性应变云图：Built-In 模型位于试样上半部分，ANN `3-15-7-1-sig` 模型位于试样下半部分。两个模型的等效塑性应变分布几乎相同。最大等效塑性应变 $\varepsilon_p$ 位于模型中心单元，即图 10 中的红色单元。表 4 显示，几个模型在 $\varepsilon_p$、$T$ 以及试样最终尺寸 $L_f$（最终长度）和 $D_f$（冲击面最终直径）方面给出了几乎相同的值。至于表 4 报告的模拟时间，在该测试算例中也可观察到与第 4.2 节相同的趋势。再次，数值结果的比较验证了所提出方法，并显示出结果之间非常好的相关性。

**图 10：Taylor 冲击试验数值模型**

**图 11：Taylor 冲击试验的等效塑性应变 $\varepsilon_p$ 云图；上半部分为 Built-in 模型，下半部分为 ANN `3-15-7-1-sig` 模型**

**表 4：3D Taylor 冲击试验结果比较**

| 模型 | Incr. | Time (s) | $L_f$ (mm) | $D_f$ (mm) | $T$ (deg C) | $\varepsilon_p$ |
|---|---:|---:|---:|---:|---:|---:|
| 3-7-4-1-sig | 6344 | 63.08 | 26.52 | 11.18 | 584.28 | 1.83 |
| 3-15-7-1-sig | 6239 | 90.32 | 26.52 | 11.17 | 584.47 | 1.83 |
| Analytical | 6419 | 71.71 | 26.53 | 11.19 | 585.64 | 1.84 |
| Built-In | 6570 | 52.82 | 26.54 | 11.21 | 588.66 | 1.84 |

## 5. 总结与结论

本文提出了一个基于人工神经网络的框架，用于建模非线性流动定律 $\sigma_y(\varepsilon_p,\dot{\varepsilon}_p,T)$，并将其应用于 42CrMo4 钢和 Johnson-Cook 型本构行为。文章介绍了多层感知机神经网络的一般架构，重点讨论了为了在 Abaqus Explicit 有限元代码中实现 `VUHARD` 用户子程序而必须计算的流动应力导数，即流动应力相对于塑性应变、塑性应变率和温度的导数；这些量并没有按照经典监督训练方案由网络直接学习。

文章详细给出了 1 个或 2 个隐藏层、2 种激活函数条件下这些导数的计算方法，并将其精度与基于 Johnson-Cook 流动定律的参考解进行了比较。结果表明，神经网络对应流动应力具有出色的评估能力，对导数也具有很好的评估能力。将神经网络数值实现到 Abaqus 代码之后，所使用的测试算例表明，在圆棒颈缩和 Taylor 冲击试验数值模拟中，所提出方法表现良好。

## 参考文献

> 以下参考文献保留英文原题名和原始题录形式，便于检索。

[1] C. Y. Gao, FE realization of a thermo-visco-plastic constitutive model using VUMAT in ABAQUS/Explicit program, in: Computational Mechanics, Springer Berlin Heidelberg, Berlin, Heidelberg, 2007, pp. 301-301.

[2] G. Jansen van Rensburg, S. Kok, Tutorial on state variable based plasticity: An Abaqus UHARD subroutine, in: Eighth South African Conference on Computational and Applied Mechanics - SACAM2012, Johannesburg, 2012.

[3] N. Duc-Toan, B. Tien-Long, J. Dong-Won, Y. Seung-Han, K. Young-Suk, A Modified Johnson-Cook Model to Predict Stress-strain Curves of Boron Steel Sheets at Elevated and Cooling Temperatures, High Temperature Materials and Processes 31 (2012) 37-45.

[4] L. Ming, O. Pantale, An efficient and robust VUMAT implementation of elastoplastic constitutive laws in Abaqus/Explicit finite element code, Mechanics & Industry 19 (3) (2018) 308.

[5] D. Versino, A. Tonda, C. A. Bronkhorst, Data driven modeling of plastic deformation, Computer Methods in Applied Mechanics and Engineering 318 (2017) 981-1004.

[6] G. Bomarito, T. Townsend, K. Stewart, K. Esham, J. Emery, J. Hochhalter, Development of interpretable, data-driven plasticity models with symbolic regression, Computers & Structures 252 (Aug. 2021).

[7] H. Park, M. Cho, Multiscale constitutive model using data-driven yield function, Composites Part B: Engineering 216 (Jul. 2021).

[8] A. Nassr, A. Javadi, A. Faramarzi, Developing constitutive models from EPR-based self-learning finite element analysis, International Journal for Numerical and Analytical Methods in Geomechanics 42 (3) (2018) 401-417.

[9] C. Mattmann, Machine Learning with Tensorflow, O'REILLY MEDIA, S.l., 2020.

[10] K. Hornik, M. Stinchcombe, H. White, Multilayer feedforward networks are universal approximators, Neural Networks 2 (5) (1989) 359-366.

[11] M. L. Minsky, S. Papert, Perceptrons; an Introduction to Computational Geometry, MIT Press, 1969.

[12] M. B. Gorji, M. Mozaffar, J. N. Heidenreich, J. Cao, D. Mohr, On the potential of recurrent neural networks for modeling path dependent plasticity, Journal of the Mechanics and Physics of Solids 143 (Oct. 2020).

[13] M. Jamli, N. Farid, The sustainability of neural network applications within finite element analysis in sheet metal forming: A review, Measurement 138 (2019) 446-460.

[14] P. Jiao, A. H. Alavi, Artificial intelligence-enabled smart mechanical metamaterials: Advent and future trends, International Materials Reviews (2020) 1-29.

[15] J. Ghaboussi, J. H. Garrett, X. Wu, Knowledge-Based Modeling of Material Behavior with Neural Networks, Journal of Engineering Mechanics 117 (1) (1991) 132-153.

[16] J. Ghaboussi, D. A. Pecknold, M. Zhang, R. M. Haj-Ali, Autoprogressive training of neural network constitutive models (1998) 22.

[17] J. Ghaboussi, D. Sidarta, New nested adaptive neural networks (NANN) for constitutive modeling, Computers and Geotechnics 22 (1) (1998) 29-52.

[18] Y. Lin, J. Zhang, J. Zhong, Application of neural networks to predict the elevated temperature flow behavior of a low alloy steel, Computational Materials Science 43 (4) (2008) 752-758.

[19] A. Javadi, M. Rezania, Intelligent finite element method: An evolutionary approach to constitutive modeling, Advanced Engineering Informatics 23 (2009) 442-541.

[20] Z. Lu, Q. Pan, X. Liu, Y. Qin, Y. He, S. Cao, Artificial neural network prediction to the hot compressive deformation behavior of Al-Cu-Mg-Ag heat-resistant aluminum alloy, Mechanics Research Communications 38 (3) (2011) 192-197.

[21] H. R. Ashtiani, P. Shahsavari, A comparative study on the phenomenological and artificial neural network models to predict hot deformation behavior of AlCuMgPb alloy, Journal of Alloys and Compounds 687 (2016) 263-273.

[22] U. Ali, W. Muhammad, A. Brahme, O. Skiba, K. Inal, Application of artificial neural networks in micromechanics for polycrystalline metals, International Journal of Plasticity 120 (2019) 205-219.

[23] M. Stoffel, F. Bamer, B. Markert, Artificial neural networks and intelligent finite elements in non-linear structural mechanics, Thin-Walled Structures 131 (2018) 102-106.

[24] M. Stoffel, F. Bamer, B. Markert, Neural network based constitutive modeling of nonlinear viscoplastic structural response, Mechanics Research Communications 95 (2019) 85-88.

[25] X. Li, C. C. Roth, D. Mohr, Machine-learning based temperature- and rate-dependent plasticity model: Application to analysis of fracture experiments on DP steel, International Journal of Plasticity 118 (2019) 320-344.

[26] X. Huang, Y. Zang, B. Guan, Constitutive models and microstructure evolution of Ti-6Al-4V alloy during the hot compressive process, Materials Research Express 8 (1) (Jan. 2021).

[27] D. W. Abueidda, S. Koric, N. A. Sobh, H. Sehitoglu, Deep learning for plasticity and thermo-viscoplasticity, International Journal of Plasticity 136 (Jan. 2021).

[28] F. Masi, I. Stefanou, P. Vannucci, V. Maffi-Berthier, Thermodynamics-based Artificial Neural Networks for constitutive modeling, Journal of the Mechanics and Physics of Solids 147 (Feb. 2021).

[29] J. C. Knight, T. Nowotny, Larger GPU-accelerated brain simulations with procedural connectivity, Nature Computational Science 1 (2) (2021) 136-142.

[30] D. Specht, A general regression neural network, IEEE Transactions on Neural Networks 2 (6) (Nov. 1991) 568-576.

[31] T. Nguyen-Thien, T. Tran-Cong, Approximation of functions and their derivatives: A neural network implementation with applications, Applied Mathematical Modelling 23 (9) (1999) 687-704.

[32] G. R. Johnson, W. H. Cook, A Constitutive Model and Data for Metals Subjected to Large Strains, High Strain Rates, and High Temperatures, in: Proceedings 7th International Symposium on Ballistics, The Hague, 1983, pp. 541-547.

[33] O. Dalverny, S. Caperaa, O. Pantale, C. Sattouf, Identification de lois constitutives et de lois de frottement adaptees aux grandes vitesses de sollicitation, Journal de Physique IV - Proceedings 12 (11) (2002) 275-282.

[34] C. Sattouf, O. Pantale, S. Caperaa, A methodology for the identification of constitutive and contact laws of metallic materials under High Strain Rates, in: Advances in Mechanical Behaviour, Plasticity and Damage, Elsevier, Tours, France, 2000, pp. 621-626.

[35] C. Sattouf, Caracterisation en dynamique rapide du comportement de materiaux utilises en aeronautique, PhD Thesis, Toulouse, INPT (Jan. 2003).

[36] M. Abadi, P. Barham, J. Chen, Z. Chen, A. Davis, J. Dean, M. Devin, S. Ghemawat, G. Irving, M. Isard, M. Kudlur, J. Levenberg, R. Monga, S. Moore, D. G. Murray, B. Steiner, P. Tucker, V. Vasudevan, P. Warden, M. Wicke, Y. Yu, X. Zheng, TensorFlow: A system for large-scale machine learning, in: Proceedings of the 12th USENIX Conference on Operating Systems Design and Implementation, OSDI'16, USENIX Association, USA, 2016, pp. 265-283.

[37] D. P. Kingma, J. Lei, Adam: A Method for Stochastic Optimization (2015) 15.

[38] J. C. Simo, T. J. R. Hughes, Computational Inelasticity, Interdisciplinary Applied Mathematics, Springer, New York, 1998.

[39] O. Pantale, ANN VUHARD repository, Software Heritage archive. ANN-FEAD-2021 (2021).

[40] J. P. Ponthot, Unified stress update algorithms for the numerical simulation of large deformation elasto-plastic and elasto-viscoplastic processes, International Journal of Plasticity (2002) 36.

[41] Q. Zhou, C. Ji, M.-Y. Zhu, Research on several constitutive models to predict the flow behaviour of GCr15 continuous casting bloom with heavy reduction, Materials Research Express 6 (12) (Jan. 2020).

[42] G. I. Taylor, The testing of Materials at High Strain Rates of Loading, Journal of the Institution of Civil Engineers 26 (8) (1946) 486-519.
