---
tags:
  - point_cloud
  - feature_point
  - feature_extraction
  - multiscale
---
> @article{LIU2023103592,
title = {Robust and Accurate Feature Detection on Point Clouds},
journal = {Computer-Aided Design},
volume = {164},
pages = {103592},
year = {2023},
issn = {0010-4485},
doi = {https://doi.org/10.1016/j.cad.2023.103592},
url = {https://www.sciencedirect.com/science/article/pii/S0010448523001240},
author = {Zheng Liu and Xiaopeng Xin and Zheng Xu and Weijie Zhou and Chunxue Wang and Renjie Chen and Ying He},
keywords = {Feature detection, Point clouds, Tensor voting, Segmentation, Surface reconstruction, Feature line extraction},
abstract = {Geometric feature detection on surfaces is a crucial task for the characterization and understanding of geometry shapes. In this paper, we present a robust and reliable approach for accurately capturing local surface variations at different feature sizes within point clouds. To this end, we define a bilateral weighted centroid projection-based metric to quantify surface deviations. Based on the metric, we propose a structure-to-detail feature perception algorithm to accurately locate geometric features of varying sizes. Additionally, we use tensor analysis to extract boundary features. We evaluate our method on various object- and scene-level point clouds, demonstrating its superiority and versatility over existing techniques. Computational results show that our method is capable of identifying a wide range of geometric characteristics within point clouds, including complicated structures, rich textures, fine details, shallow curves, and geometric boundaries. We also validate the effectiveness of our approach on several downstream applications, including segmentation, surface reconstruction, and feature line extraction.}
}
# 0 Abstract
surface上的geometric feature detection是表征和理解几何形状的关键任务
本文，我们提出了一个鲁棒可靠的方法来准确捕获point clouds中的不同feature尺寸的局部surface变化
为此，我们定义了一个基于bilateral weighted centroid projection-based度量，来量化surface偏差
基于这个度量，我们提出了一个structure-to-detail的feature感知算法，来准确定位不同尺寸的geometric features
此外，我们使用tensor分析来提取boundary features
我们在各种物体和场景级别的point clouds上评估了我们的方法，证明了它比现有计算的优越性和通用性
计算结果表明，该方法能够识别point cloud中的复杂结构、丰富纹理、精细细节、shallow曲线、几何边界
我们还验证了我们的方法在几个下游应用中的有效性，包括segmentation、surface reconstruction、feature line extraction
# 1 Introduction
# 2 Related Work
### Geometry-based methods
### Statistic-based methods
### Learning-based methods
# 3 Method
我们定义feature为surface特征，包括complex结构、rich textures、fine details、shallow curves、geometric boundaries
采样surfaces通常具有不同的尺度，因此在检测大小不等的features时需要改变邻域大小
给定一个point cloud，我们可以直接使用初始邻域来提取其大尺寸结构，如Figure 1b
然而，使用大尺度邻域，我们不能准确地识别中尺度和小尺度features
为了解决这个局限，我们提出将初始邻域划分为许多个sub-邻域，并选择有着最高局部surface变化的sub-邻域，如Figure 1c
为了进一步检测更精细的细节，我们继续将sub-邻域分割为更精细的，然后寻找局部变化最大的邻域，如Figure 1d
如Figure 1，从结构features到细节features，邻域大小对feature检测有着巨大的影响
因此，为了尽可能准确地检测structure-to-detail的features，我们迭代地进行邻域分割和选择
![[Pasted image 20240718161931.png]]
## 3.1 Structure-to-detail feature perception
令$P=\{p_i\}_{i=1}^M$是从一个$\mathbb{R}^3$的2-manifold上采样的非结构性point cloud，其中$M$是采样点数量
令$F$为point cloud的feature point set
且$V$为潜在的feature point set，初始化为$V=P$
给定点$p_i\in P$，我们定义其multiscale邻域为$\Omega^z(p_i)$
	当$z=0$，我们获得$p_i$的原始邻域$\Omega^0(p_i)=\{p_j|\Vert p_j-p_i\Vert < r\}$，其中$r$是邻域半径
	当$z$增加时，$\Omega^z(p_i)$表示$p_i$的refined的sub-邻域，这样可以更准确地定位局部形状变化
为了识别point cloud上structure-to-detail features，我们迭代地执行以下步骤：
1. 对每个点$p_i\in V$，我们使用当前邻域$\Omega^z(p_i)$计算其变化度量值$\delta_i$，由以下公式（1），然后将这些值放入集合$\varphi$，并降序排序
2. 我们将具有最大变化值$\beta$的点移动到当前的feature point set $P_F$，并移动最小变化值$\gamma$的点到当前non-feature point set$P_{nonF}$，然后识别剩余点作为更新后的潜在feature point set $V$，其中$\beta,\gamma$是百分比系数
3. 对每个点$p_i\in V$，我们使用邻域splitting操作（后面叙述）来分裂其当前邻域$\Omega^z(p_i)$为multiple sub-邻域，然后我们选择变化最大的sub-邻域作为$p_i$的refined邻域$\Omega^{z+1}(p_i)$，见Figure 2为邻域splitting和选择策略的2D示意图
![[Pasted image 20240718164350.png]]
Algorithm 1概述了上述过程
	算法终止并返回feature point set $F$
	大多数情况下，3次邻域更新就足以产生令人满意的feature检测结果
Figure 1给出了一个示例
![[Pasted image 20240718164719.png]]
### Neighborhood splitting operation
给定当前的$\Omega^z(p_i)$，我们希望细化它来匹配弯曲区域的形状
因此我们设计了一个splitting操作，将当前邻域重复划分为一系列sub-邻域

给定一个点$p_i$，我们首先计算其在当前邻域的centroid $c_i$
然后对于每个邻域点$p_j$，我们可以构建一个plane $\Lambda(p_i,p_j,c_i)$，其将$\Omega^z(p_i)$分为两个sub-邻域
通过在当前邻域$\Omega^z(p_i)$重复splitting操作，我们可以获得$2K$个sub-邻域，$K=|\Omega^z(p_i)|$
然后，对于下一轮splitting，我们可以选择sub-邻域中有着最大$\delta$的作为$\Omega^{z+1}(p_i)$
见Figure 3
![[Pasted image 20240718165327.png]]
## 3.2 Weighted Centroid Projection Metric
为了鲁棒灵活地量化给定点$p_i$在不同邻域尺寸下的geometric saliency，我们定义了weighted centroid projection distance
![[Pasted image 20240718190352.png]]
	其中$k_i$为bilateral weight，度量了$p_i$附近的一个大邻域的整体的flatness
	$D_i$量化了$p_i$的给定邻域的几何变化
### Centroid projection term
为了度量$p_i$在当前邻域$\Omega^z(p_i)$的局部弯曲变化，提出了centroid projection项如下：
![[Pasted image 20240718190558.png]]
	其中$n_i$为$p_i$处用PCA估计的normal，$c_i$是$\Omega^z(p_i)$的centroid
	该项描述了$p_ic_i$在法线$n_i$的投影，见Figure 4
![[Pasted image 20240718190724.png]]
然而，当采样surface被noise破坏时，如果单独使用公式（2）来检测特征，可能会将许多noise point错误识别为features，见Figure 5b
	因为$p_ic_i$的投影由于噪声干扰明显变长，如Figure 4c
![[Pasted image 20240718191913.png]]
该计算过程与三角网格上的Laplace-Beltrami算子类似...
### Bilateral weights
仅仅依靠mean curvature作为局部metric会导致noisy point被错误识别为features
为了抵消noise对feature检测的影响，我们提出了一种包含centroid projection term的公式（2），以便对feature points和noisy points进行对比，从而更容易增强识别feature points
该权重由两个因素决定：
![[Pasted image 20240718192544.png]]
	其中$k_i^n$时单调递增的指数函数，其关于邻域$\Omega^0(p_i)$类的整体法向偏差
	$k_i^p$时相同的函数，其关于整体距离差异
![[Pasted image 20240718192826.png]]
![[Pasted image 20240718192852.png]]
通过这两项相乘，我们可以减少错误估计的法向对bilateral weight公式（3）的影响，使计算更加稳定，Figure 6给出一个例子
![[Pasted image 20240718193012.png]]

bilateral weight公式（3）允许我们度量$p_i$附近更大的邻域$(\Omega^0(p_i))$的flatness
## 3.3 Detection of boundary features
这里使用一个简单高效的tensor分析来提取boundary features
给定一个点$p_i$，其voting tensor被构造为其邻域点的坐标协方差矩阵的加权和
![[Pasted image 20240718193943.png]]
	由于该tensor是对称半正定的，可以用特征值分解对角化
	![[Pasted image 20240718194035.png]]
...
# 4 Experiments and discussions
## 4.1 Parameter setting
## 4.2 Qualitative comparison
### Feature detection on CAD surfaces
### Feature detection on non-CAD surface
### Feature detection on scene data
## 4.3 Quantitative evaluation
## 4.4 Comparison with SGLBP and PCEDNet
## 4.5 Robustness tests
### Large noise & irregular sampling
### Complex scenes
### Generalization
## 4.6 Impact of normal estimation
## 4.7 Limitations
# 5 Applications
### Segmentation
### Surface reconstruction
### Feature line extraction
# 6 Conclusion