---
tags:
  - feature_point
  - point_cloud
  - Gauss_map
  - feature_detection
---
> @INPROCEEDINGS{5521460,
  author={Weber, Christopher and Hahmann, Stefanie and Hagen, Hans},
  booktitle={2010 Shape Modeling International Conference}, 
  title={Sharp feature detection in point clouds}, 
  year={2010},
  volume={},
  number={},
  pages={175-186},
  keywords={Computer vision;Clustering algorithms;Gaussian processes;Shape;Surface reconstruction;Feature extraction;Solid modeling;Three-dimensional displays;Sampling methods;unstructured point sets;feature detection;sharp features;Gauss map clustering},
  doi={10.1109/SMI.2010.32}}
# 0 Abstract
本文提出了一个新的技术在point-sampled几何上来检测sharp features
不同性质和possessing角度（从钝角到锐角不等）的尖锐特征都可以被识别，无需用户交互
算法直接在point cloud上工作，没有surface reconstrction的需要
给定一个unstructured point cloud，我们的方法首先在局部邻域计算一个Gauss map，以丢弃所有不可能属于sharp feature的点
通常，一个全局sensitivity的参数在这个阶段被使用
第二阶段，剩余的feature candidates经历一个更加精确的迭代选择过程
我们的方法的核心是自动计算自适应sensitivity参数，大大提高了可靠性，并使识别钝角和锐角的存在时更加鲁棒
该算法基于局部邻域graph计算，速度快，不依赖于sampling resolution
# 1 Introduction
# 2 Related Works
# 3 Feature Extraction
本节介绍了我们基本的feature extraction方法
包含三个步骤：
1. 首先对我们的点集构建数据结构
2. 构建点集的局部邻域
3. 我们分析这些邻域来识别sharp features
定义point cloud和sharp feature
	==point cloud==是一个3D point坐标的simple set $P=\{p_1,p_2,...,p_N\},p_i\in\mathbb{R}^3$，没有法向和任何连接信息，数据点是unstructured，但属于2-manifold surface，$N=|P|$是point set的cardinal
	我们想在point cloud中检测到的==sharp feature==可以是不同属性的，可以是在两个surface之间的edge或line，或多于三个surface的一个corner
## 3.1 Data structure
为了检测point cloud中的sharp features，我们首先要先检测cloud中的每个点，是否属于sharp feature的一部分
我们分析每个single point的邻域，来决定其是否属于一个sharp feature
	邻域使用k近邻

结果可以存储在一个Riemannian graph
	图中，连接该点的最小生成树扩展为将每个点与其k最近邻域连接起来
## 3.2 Neighborhood analysis
有着点$p\in P$的k最近邻点构建的局部邻域$N_p$，我们现在想要分析其来决定sample point是否属于一个sharp feature
为此，我们应用一个Gauss map clustering
## 3.3 Discrete Gauss map
令$N_p$为点$p$的邻域，包含其k最近邻，$I_p$为$N_p$的index set
令$T$为$p$用其邻域点的所有可能的$k(k-1)$三角化的集合：
![[Pasted image 20240726145558.png]]
$p$的邻域的discrete Gauss map现在可以定义为$T$到 以$p$为中心的unit sphere $S^2$ 的映射：
![[Pasted image 20240726145743.png]]

feature detection现在分两步执行：
1. 通过简单的flatness test来丢弃所有属于planar region的点
2. 剩下的feature candidate point在第二步总进行更精确的选择过程，成为Gauss map clustering
	Gauss map clustering也可以用于其它应用，例如将point sampled geometry分割成连接区域，将具有相同local curvature的点分组在一起
### Flatness Test
这些normal向量现在允许得出点云$p$处local surface behavior的一些结论
如果一个lines经过$p$，且张成的$n_{ij},i<j$几乎平行，则潜在的surface在$p$是接近flat的
	注意到$n_{ij}$和$n_{ji}$张成同一条line，为了比较它们，我们计算这些lines的角度

flatness test包括计算这些角度的差异
	即检测角度到中值方向的距离
如果它低于给定的阈值（使用15%，约为13°），其相对应的surface在这个邻域被假定是flat的或接近flat的，没有sharp feature
但反过来就不成立，$p$的高偏差并不意味着一个sharp feature，它也可以出现在没有sharp feature的高曲率区域
### Gauss Map Clustering
这就是为什么我们也通过计算集合$T$的Gauss map $G_p$来分析$p$

想法基于这样的事实，即在一个光滑的piecewise$C^0$ surface，无论点是否是flat、curved、tangent plane不连续的，surface的邻域的Gauss map是不同的
	接近flat point的邻域点的Gauss map表示为sphere上的一个点cluster
	在curved point的情况，其在sphere上的点会分散到一个更大的区域
	在tangent plane不连续的情况，sphere上的点会成为两个不同的cluster
见Figure 2一些2D的例子
![[Pasted image 20240726172243.png]]

在point-sampled surface，困难在于我们对附近的surface一无所知
	我们没有一个局部三角化，也没有一个与邻域点相关的normal vector
因此我们将Gauss map定义为所有可能的局部三角化，见Section 3.3
由此产生的sharp feature point的Gauss map将包含一些额外的noisy point，其对应于不属于潜在surface的triangles
	这些noisy points并不影响一般的clustering表现
	实际上，它们在计算clusters时将会消失

举例说明两个相交plane的常见情况，$p$位于sharp edge上
一半的k邻域点在每个plane上，见Figure 3 left
	计算Gauss map会产生两个相对的cluster（由$O(k^2/4)$个相同的点所对应的triangles完全在一个plane，两个cluster在另外的plane）
	注意到$n_{ij}$和$n_{ji}$属于不同的clusters
sphere上的其它点（noisy point）对应的triangle，其$p_i$属于一个plane，$p_j$属于另一个plane
	这些点稀疏地分布在sphere上
Figure 3给出了一个例子，左侧展示了planes和16个邻域点，右侧展示了Gauss map，其中粗大的点表示clusters
![[Pasted image 20240726184458.png]]
对sharp features的非详尽列表，可以观察到类似的clustering表现，见Figure 4
![[Pasted image 20240726195416.png]]
因此，我们使用sphere上的投影点的clustering表现（称为Gauss map clustering），来确定采样点$p$是否属于sharp feature

关于clusters，我们需要记住，我们不能确定上述提到的normals的方向
	这意味着我们不需要额外的信息，即normals指向surface内部或外部
但对于识别sharp feature来说，这一点都不重要
#### Distance measure
clustering算法的一个重要的步骤是选择距离度量，它决定两个elements的距离有多近
目前的point set有两个特点：
1. 它是一个球面点集
2. 它由对称点组成，即位于球面上相对位置的点
适当的距离度量必须考虑到这一点

为了保证点集的对称性，我们选择穿过对称点的lines之间的夹角作为距离度量
可以证明，我们的Gauss map的两个normals之间的角度，等于球面上两点之间测地线的距离
由normals张成的两个lines的最小角度是满足先前提到的恰当的距离度量的特殊要求
![[Pasted image 20240726200955.png]]
#### Clustering
我们的目标不是对整个点集进行聚类，而是将一些clusters从稀疏分布的Gauss map中区分开来
...
#### Analysis
所有只包含少量点的cluster都被丢弃，因为它们对应于有噪声的点
剩下的聚类分析如下：
	sphere上相对的cluster被认为是一个cluster
	如果最后仍保留有1个单独的cluster，我们认为当前point不属于feature
	如果保留有2、3、4个clusters，我们认为该point属于一个sharp feature
	如果保留多余4个clusters，我们认为当前point不属于一个feature
		大多数数据集不会有四个sharp features相交于一点，见Figure 5例子
		如果有必要，可以在特定数据集上进行调整
		![[Pasted image 20240802182348.png]]
# 4 Local-adaptive Method
决定是否声明一个sample point为sharp point，取决于局部领域的大小（$k$）以及sensitivity参数（$\sigma$）
## 4.1 Size of neighborhood 
## 4.2 Sensitivity parameter for Gauss map clustering
...

显然，当point cloud包含钝角和锐角features时，一个全局的sensitivity参数是不够的
因此，全局值总是需要在找到所有features（requirement 1）和找到features的准确位置（requirement 2）之间进行权衡，这一观察促使开发下一节介绍的算法的扩展
## 4.3 Adaptive and local sensitivity parameter
# 5 Results
### Robustness w/r to varying angles
### Comparison between global and local-adaptive method
#### Global method
#### Local-adaptive method
### Robustness to noise
### Complex point-sampled surfaces
### Comparison to PCA methods
# 6 Conclusion and Future Work
...

所有的line-type、corner-type sharp features都被检测
cone peaks此处并没有处理，为了识别这种情况下的特定clustering行为，需要对方法进行调整