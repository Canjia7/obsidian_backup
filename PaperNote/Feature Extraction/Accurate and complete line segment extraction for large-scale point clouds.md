---
tags:
  - point_cloud
  - feature_point
  - feature_extraction
  - large_scale
---
![[Pasted image 20240717223722.png]]
> @article{XIN2024103728,
title = {Accurate and complete line segment extraction for large-scale point clouds},
journal = {International Journal of Applied Earth Observation and Geoinformation},
volume = {128},
pages = {103728},
year = {2024},
issn = {1569-8432},
doi = {https://doi.org/10.1016/j.jag.2024.103728},
url = {https://www.sciencedirect.com/science/article/pii/S1569843224000827},
author = {Xiaopeng Xin and Wei Huang and Saishang Zhong and Ming Zhang and Zheng Liu and Zhong Xie},
keywords = {Line segment extraction, Large-scale point clouds, Feature point detection},
abstract = {Line segment extraction from point clouds is critical for analyzing and understanding large-scale scenes. The main challenge is to generate line segments accurately as well as completely. However, state-of-the-art approaches continue to struggle with this issue. In this paper, we propose a novel method for effectively generating line segments from large-scale point clouds. To this end, we design a weighted centroid displacement scheme for identifying comprehensive feature points. Then, we employ an L1-median optimization to refine the identified feature points to perceive geometric edges on the underlying surface accurately. Following that, we generate complete and concise line segments from the refined feature points by designing three geometric operators: clustering, exclusion, and assimilation. The clustering operator generates the initial line segments based on optimized feature points, and the exclusion operator and the assimilation operator ensure the completeness and continuity of these line segments. We evaluate our approach on various scene point clouds, such as TLS, MLS, and ALS data. Extensive experimental results show that our method can outperform the competing approaches in terms of both accuracy and efficiency.}
}
# 0 Abstract
从point cloud中提取line segment对于分析和理解大规模场景至关重要
主要挑战是准确而完整地生成segment
然而，最先进的方法仍在努力解决这个问题

本文提出了一种从大规模点云中有效生成segment的新方法
为此，我们设计了一种weighted centroid displacement方案来识别comprehensive feature points
然后，我们使用L1-median optimization来refine识别的feature points，以准确地感知underlying surface的geometric edges
然后，通过设计三个geometric operators：clustering、exclusing、assimilation，从refine的feature points生成完整且简洁的line segments
	clustering operator基于优化后的feature points来生成初始的line segments
	exclusion operator和assimilation operator确保这些line segments的完整性和连续性

我们在各种场景point clouds上评估了我们的方法
大量实验结果表明，我们的方法在精度和效率方面都优于竞争对手
# 1 Introduction
# 2 Related work
## 2.1 Feature point detection
## 2.2 Line segment generation
# 3 Methodology
我们提出了一个2阶段方法来从大规模point clouds提取高质量line segments
一阶段，我们准确检测feature points，其精确落在underlying surface的geometric edges
二阶段，使用3哥geometric operators，从检查到的feature points重建完整且简洁的line segments
## 3.1 Detecting feature points
首先引入一个weighted centroid displacement方案来检测coarse feature points
然后用一个L1-median optimization来refine检测到的feature points，使得这些refined points可以精确地匹配underlying geometric edges
### 3.1.1 Coarse feature points detection by a weighted centroid displacement scheme
记input cloud为$P$
给定一个点$p_i\in P$，其传统centroid displacement vector $L_i$可以计算为$L_i=c_i-p_i$，其中$c_i$是通过对$p_i$的k邻域计算平均位置得到的centroid
粗略地，$\Vert L_i\Vert$可以作为识别coarse feature points的一个度量
	大多数情况下，$\Vert L_i\Vert$较大可以识别$p_i$越接近为一个feature point
然而传统的centroid displacement不能有效地感知surface的细微变化
	可能导致surface变化较小的区域误认为是non-feature point，见Figure 3b
	原因是，$L_i$本质上是一个均匀的Laplacian vector，其权值对surface变化不敏感
	从Figure 3a，3b可以看出flat region和shallow feature的传统displacement vectors的长度较小，无法清晰区分shallow feature points和non-feature points
![[Pasted image 20240718141303.png]]
为了解决这个问题，我们引入一个新的weighted centroid displacement方案
关键是为每个point估计一个新的centroid，以扩大feature points和non-feature points的区别，特别是在shallow feature区域
如Figure 2
	如果$p_i$在flat区域或shallow区域，其相关的传统centroid $c_i$很接近于underlying surface，见Figure 2a、2b
	因此，使用传统centroid displacement vector $L_i$很难从shallow features中识别出feature points
因此对每个点$p_i$，我们估计一个新的centroid
	为此，对于每个邻域点$p_j$，我们首先生成一个temp point $m_j$，即Figure 2c中的绿色点，通过构建向量$p_im_j=p_ip_j+p_ic_i$
	构建所有vector后，我们可以估计一个新的centroid $d_i$来计算weighted centroid displacement vector $\delta_i=d_i-p_i$
	![[Pasted image 20240718142325.png]]
可以看出，weighted centroid displacement方案放大了在shallow features和falt区域的points的差异
![[Pasted image 20240718141712.png]]
Figure 4展示了使用$\Vert L_i\Vert$和$\Vert \delta_i\Vert$的不同
![[Pasted image 20240718142510.png]]
基于此，我们定义一个新的度量来从point cloud中检测feature points
![[Pasted image 20240718142707.png]]
	其中$\mathcal{K}_i$是一个weighted项，旨在进一步增加feature points和non-feature points之间的差异，特别是在有着finer edges的区域
	我们使用flatness来估计权值$\mathcal{K}_i$，基于weighted centroid displacement vector $\delta_i$
	flatness估计为
	![[Pasted image 20240718142936.png]]
	通过公式（3），我们可以捕获不同轴方向的independent surface变化，从而精确度量local region的flatness
	因此，feature points的$\mathcal{K}_i$倾向于大于non-feature points
现在可以从input point clouds中获得feature points $F$通过：
![[Pasted image 20240718144443.png]]
### 3.1.2 Coarse feature point optimization
由于point cloud的采样率是不规则的（特别是大规模point cloud场景），检测到的feature points可能是冗余的，如Figure 5b
	因此，有必要对其进行进一步细化
![[Pasted image 20240718144706.png]]
通常，检测到的feature points位于underlying surface的geometric edges
因此，我们的目标是重新定位这些点，使它们的分布变thin，同时不改变input point clouds的形状
为此，我们使用L1-median optimization，其设计为整个point cloud optimization
![[Pasted image 20240718145548.png]]
结果上
	第一项可以驱动优化后的feature points $X$来近似粗略检测到的feature points $F$的geometry
	第二项可以保持优化后的feature points $X$的分布公平

我们用梯度下降法最小化公式（6）
应用在feature points上，最优化结果见Figure 6
	可以看到，忽略第二项，会导致不连续
![[Pasted image 20240718145809.png]]
## 3.2 Line segment extraction
利用检测到的feature points，line segment generation用于准确提取完整line segments
为此，我们提出以恶搞coarse-to-fine generation算法，其首先使用feature points来生成coarse line segments，并从完整性和简单性两个方面来refine它们
完整算法见Algorithm 1，关键是一系列geometric operators，包括clustering、exclusion、assimilation
	clustering operator见feature points分为很多组，每组用于生成线段
	exclusion operator从生成的line segment set移除离群line segments，通过识别line segments之间的包含关系
	assimilation operator对line segments进行组合，并保证它们的完整性和简洁性
![[Pasted image 20240718150048.png]]
### Clustering operator
目的是根据检测到的feature points在underlying surface的几何性质，将其分为不同的聚类
使用region-growing策略
	将feature points分组到不同聚类，每个聚类中的points往往位于同一条edge
算法细节：
1. 对每个feature point $x_i\in X$，计算最大特征值的特征向量$e_i$
2. 使用广度优先搜索策略来获得所有属于与$x_i$同一个cluster的points，记该聚类为$v_i$
	通过主成分方向与聚类中方向平均的角度差异...
3. 聚类后，得到最后结果$R=\{v_1,v_2,...,v_n\}$，见Figure 7b
进一步，可以最小二乘拟合每个组，获得coarse line segments $LS=\{l_1,l_2,...,l_n\}$，见Figure 7c
![[Pasted image 20240718150655.png]]
### Exclusion operator
处理具有丰富细节的大规模点云时，不可避免地会产生离群segments
需要将它们从$LS$中移除，关键是如何识别

此处，我们提出一种利用line segment间邻域关系的识别策略
我们首先计算两个中点的距离来度量相应line segments的距离
然后通过下式来识别离群line segments
![[Pasted image 20240718151315.png]]
Figure 8展示了一个line segmet set
	如果line segment是一个离群的，line segment和其k邻域line segments之间的邻域关系不是dual的（<font color=red>离群line segment的k邻域中的line segments通常离它很远，因此其k邻域中的line segments中的某个的k邻域一般不包含该离群line segment</font>）
![[Pasted image 20240718151455.png]]
### Assimilation operator
过滤了离群line segments后，剩余的line segments仍然存在完整性和简洁性的问题，如Figure 7c
为此，assimilation operator尽可能合并具有相似几何属性的line segments
考虑location、length、direction，我们定义一个指标来衡量两个邻接line segments之间的相似性
![[Pasted image 20240718152048.png]]
![[Pasted image 20240718152246.png]]
通常，如果两个线段$l_i,l_j$很相似，三个度量都有着很小的值

基于这个度量，我们提出一个2阶段assimilation operator，基于一个扩展方案来合并相似的line segments
	第一个阶段，创建一个neighbor line segment set $S_i$，通过搜索在一个指定region的line segments
	然后，可以通过检测$S(l_i,l_j)<\epsilon_l$来获得一个assimilation set $S_i$
	第二个阶段，为集合$S_i$生成一个merged line segment $l_a$
![[Pasted image 20240718152301.png]]
# 4 Experiments
## 4.1 Parameter setting
## 4.2 Performance of feature point detection
### Qualitative comparison
### Evaluation of feature detection
## 4.3 Performance of line segment generation
### Qualitative comparison
### Quantitative evaluation
## 4.4 Comparison with deep-learning methods
## 4.5 Generalization tests
# 5 Conclusion