![[Pasted image 20240729171758.png]]
> @article{10.1145/3658215,
author = {Ju, Yiwen and Du, Xingyi and Zhou, Qingnan and Carr, Nathan and Ju, Tao},
title = {Adaptive grid generation for discretizing implicit complexes},
year = {2024},
issue_date = {July 2024},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {43},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/3658215},
doi = {10.1145/3658215},
abstract = {We present a method for generating a simplicial (e.g., triangular or tetrahedral) grid to enable adaptive discretization of implicit shapes defined by a vector function. Such shapes, which we call implicit complexes, are generalizations of implicit surfaces and useful for representing non-smooth and non-manifold structures. While adaptive grid generation has been extensively studied for polygonizing implicit surfaces, few methods are designed for implicit complexes. Our method can generate adaptive grids for several implicit complexes, including arrangements of implicit surfaces, CSG shapes, material interfaces, and curve networks. Importantly, our method adapts the grid to the geometry of not only the implicit surfaces but also their lower-dimensional intersections. We demonstrate how our method enables efficient and detail-preserving discretization of non-trivial implicit shapes.},
journal = {ACM Trans. Graph.},
month = {jul},
articleno = {82},
numpages = {17},
keywords = {implicit surfaces, grid refinement, surface networks}
}
# 0 Abstract
我们提出了一个方法来生成一个simplicial（例如triangular或tetrahedral）grid来实现对一个vector function定义的implicit shape的自适应离散化
	这种形状，我们称之为implicit complexes，是implicit surfaces的推广，且对于表示non-smooth和non-manifold很有用
虽然对implicit surfaces的自适应grid generation已经被广泛研究了，对implicit complexes的方法很少
我们的方法可以为多种implicit complexes生成自适应grid，包括implicit surfaces的组合、CSG shapes、材料interfaces、curve networks
更重要的是，我们的方法将grid适应于（不仅是implicit surfaces，而且是它们的低维交点的）geometry
我们演示了我们的方法如何使non-trivial implicit surfaces进行高效和detail-preserving的离散化
# 1 Introduction
implicit representation再CG中很常用
一个3D的smooth和manifold的surface可以表示为一个scalar function的level set $f:\mathbb{R}^3\rightarrow\mathbb{R}$，也可以称为==implicit surface==
implicit surface有很多好处：定义简单、容易修改（geometry和topology）、对操作方便（如offset、boolean）

smooth manifolds以外的shapes也可以用implicit表示，通过使用一个==vector function== $f:\mathbb{R}^3\rightarrow\mathbb{R}^m$，其中$m>1$
	例如，一个CSG shape通常用于表示piecewise smooth surfaces，它由多个implicit surfaces组成，每个implicit surface都可以由$f$的一个scalar分量的level set定义
	surface的non-manifold network可以由多重implicit surfaces的排列表示，或者作为一个$f$的一个分量大于其它分量的区域的interface
	vector function也可以定义低维shapes，例如由两个implicit surfaces相交定义的spatial curves
我们把由某些vector function implicitly定义的shape称为==implicit complex==
一些例子可以见Figure 1 bottom
![[Pasted image 20240729171758.png]]

为了对下游应用有用，一个implicit surface或complex必须要被离散化
许多discretization方法，例如marching cubes、marching tetrahedra，在由cubic或simplicial cells组成的空间上操作（见Section 2.1）
grid structure对这些方法的性能有着关键的作用
虽然更精细的grid通常会导致更高的discretization精度，其会导致grid generation和discretization更慢，更大的内存占用和输出尺寸
用于discretization的一个理想的grid应该是自适应的，因为更精细的grid cells只位于需要更高精度的地方
	这些位置通常是implicit shape具有non-trival的geometry或topology

虽然自适应grid generation已经被广泛研究，用于implicit surfaces的多边形化，用在implicit complexes的工作很少
后者的一个关键挑战是调整grid structure以精确地discretize多个implicit surfaces的交叉
	这样的交叉在一个implicit complex构成低维elements，例如在一个CSG shape的sharp edges和corners，或者在non-manifold surface network的junction graph
注意到implicit surfaces的交叉可能有一个比它们本身更复杂的geometry，如Figure 2a的例子
	因此，仅适应于implicit surfaces的grids可能在其它交叉处的分辨率不足，见Figure 2b，或者在其它地方过分细化，见Figure 2c
![[Pasted image 20240729184149.png]]
现有的这种capture的方法要么局限于低次多项式，或依赖于interval分析，因此它们不适用于一般函数，因为在一般函数中，tight intervals如果是可能求得的话会很难求得

本文提出了一种生成自适应simplical grids来discretizing各种implicit complexes（如Figure 1）
该方法的一个关键特点是：将grid structure适应于implicit surfaces和它们的交叉的geometry
	这使得所有维度的implicit complex中的elements都可以精确地discretized，而无需对grid进行过多细化，见Figure 2d
与现有的intersection-aware方法不同，我们的方法不需要intervals，且可以应用于任何可以查询values和gradients的函数

我们的主要贡献是一套标准，用来决定一个simplicial cell是否需要细化
	给定implicit complex的特定定义
这些标准的核心是（近似地）新颖地检查一个$n$-simplex是否包含$m$个implicit surfaces的交叉，以及这样的交叉是否是否被线性插值很好地近似
	我们的测试可以高效鲁棒地应用在任何维度$n$和任何$m\leq n$，不需要interval分析
这些测试成为implicit complex的细化标准的building blocks，其包含额外的check，以避免 沿着一个不在complex上的implicit surface（或交叉）的一部分（如CSG的一个primitive的裁剪过的部分） 的细化

我们的第二个贡献是对一个simplicial grid进行top-down细化的新方法，给出一些细化标准
我们的方法是经典的longest edge bisection（LEB）方法的一个变体
	它平分simplex，在最长edge的中点
虽然LEB可以生成size平滑变化的高质量cells，但grid可以为了discretizing一个implicit shape的目的过分细化（见Figure 3a）
我们的变体产生的cells比LEB少，通过允许cell远离implicit shape，这些cell与discretization无关，得到更差的shapes，因此有更好的适应性（见Figure 3b）
![[Pasted image 20240729200706.png]]
# 2 Previous Work
## 2.1 Grid-based discretization
## 2.2 Grid generation for discretization
### 2.2.1 Refinement criteria
### 2.2.2 Refinement method
# 3 Preliminaries: Implicit Representations
## 3.1 Level sets and zero sets
## 3.2 Implicit complexes
# 4 Refinement Criteria for Zero Sets
## 4.1 Proxy construction
## 4.2 Zero-crossing test
## 4.3 Distance test
# 5 Refinement Criteria for Implicit Complexes
## 5.1 General criteria
## 5.2 Active components
# 6 Simplicial Refinement
# 7 Results
## 7.1 Zero-crossing and distance tests
## 7.2 Active components
## 7.3 Simplicial refinement
## 7.4 Implementation and performance
## 7.5 More examples
# 8 Discussions