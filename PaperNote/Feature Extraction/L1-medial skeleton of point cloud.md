---
tags:
  - feature_extraction
  - point_cloud
  - skeleton
  - multiscale
---
![[Pasted image 20240719191251.png]]
> @article{huang2013l1,
  title={L1-medial skeleton of point cloud.},
  author={Huang, Hui and Wu, Shihao and Cohen-Or, Daniel and Gong, Minglun and Zhang, Hao and Li, Guiqing and Chen, Baoquan},
  journal={ACM Trans. Graph.},
  volume={32},
  number={4},
  pages={65--1},
  year={2013}
}
# 0 Abstract
我们引入了L1-median skeleton作为3D point cloud数据的一个curve skeleton表示
L1-median被广泛认为是一个任意点集的鲁棒的全局中心
我们作了一个关键的观察，即局部地适应L1-median到一个代表3D shape的点集，从而产生一维结构，该结构可以被视为shape的局部中心
我们的方法的主要优势是，其对于input point cloud的质量没有很强的要求，也没有对捕获形状的几何或拓扑有很强的要求
我们开发了一种L1-median skeleton构建算法，该算法可以直接应用于具有显著noise、outliers、大面积缺失数据上
我们展示了从各种形状的原始扫描中提取L1-median骨骼，包括那些建模高亏格的3D物体、plant-like结构、curve network
# 1 Introduction
给定一个无组织定向点集$Q=\{q_j\}_{j\in J}\subset \mathbb{R}^3$，我们研究以下定义的L1-medial skeletons，其将产生一个最优投影点集$X=\{x_i\}_{i\in I}$
![[Pasted image 20240723102924.png]]
	其中第一项是$Q$的局部化L1-median
	第二项$R(X)$正则化$X$的局部点分布
	$\theta(r)=e^{-r^2/(h/2)^2}$是一个快速衰减光滑函数，有着支撑半径$h$定义了构造L1-medial skeleton的支撑局部邻域的尺寸

本文方法的主要贡献和优势如下
1. 从一般的由点集表示的3D shapes（没有预先假设的几何或拓扑）中提取curve skeleton
2. 直接对原始扫描数据进行操作，即不进行预处理，包括denoising、outlier去除、正态估计、空间离散、数据不全、网格重建或参数化
3. 一个简单的公式（1），可以基于默认的参数快速鲁棒地提取skeleton
# 2 Related work
# 3 Overview
我们算法的输入是一个无组织点集$Q$，通常是无定向、不均匀分布的，且包含噪声和outliers
	输出是一个curve skeleton，其表示了一个input $Q$潜在的shape的1维局部中心
主要算法（Section 4）
	一个点集从给定raw scan上随机采样（Figure 2c中红色）
	每个点在其局部邻域内迭代地被投影和重分布到input points的中心，邻域的尺寸在处理不同等级的细节是被逐步增加，产生一组干净的且well-connected的skeletal points（如Figure 2d-g）
![[Pasted image 20240723104907.png]]
如果input point高度不均匀，通过上述算法得到的skeletal points可能也会不均匀，见Figure 7
对于高度不完整的point cloud，生成的branches可能会off-centered，见Figure 8
为了解决这两个问题，Section 4.3和Section 4.4提出了两个改进：基于密度的weighting和re-centering
最终改进的L1-medial skeleton见Figure 2h
![[Pasted image 20240723105350.png]]
![[Pasted image 20240723105400.png]]
# 4 L1-medial skeleton and construction
考虑问题，寻找点$x$的位置使其到一组点有着最小的总共欧氏距离，即：
![[Pasted image 20240723105600.png]]
	该优化问题的解被认为是空间中位数，或L1-median
	当$\{q_j\}_{j\in J}$不共线时，该L1-median $x$是唯一的
![[Pasted image 20240723102924.png]]
	第一项可以看作是L1-median的一个局部版本，定位与有限支撑半径$h$和权函数$\theta$有关

使用L1-median的一个关键优势是，L1-median对数据中的noise和outliers不敏感，不像通常的平均值，见Figure 3
![[Pasted image 20240723110007.png]]
## 4.1 Conditional regularization
应用L1-median倾向于产生一个离散分布，其中局部中心可能会累计成一组point clusters，见Figure 4b
![[Pasted image 20240723110205.png]]为了避免这样的clustering并保持适当的medial几何表示，当points已经收缩到它们的局部中心位置后，我们需要防止进一步积累
这由公式（1）中$R(X)$实现，当skeleton branch局部地构建时，其添加了一个斥力
	这样的一个惩罚项被称为conditional regularization

我们采用经典的带权PCA来检测skeleton branches的构成
在每个点$x_i$（一个行向量），我们计算$3\times3$的带权协方差矩阵的特征值和特征向量
![[Pasted image 20240723110807.png]]
	其中$\lambda_i^0\leq\lambda_i^1\leq\lambda_i^2$是实值，且对应的特征向量构成一个正交框架，即point set的主成分
然后我们在一个局部邻域定义$x_i$的directionality degree
![[Pasted image 20240723111000.png]]
	$\sigma_i$越接近1，则更多$x_i$附近的点沿着一个branch排列

为了有条件地施加斥力，我们定义我们的regularization function为：
![[Pasted image 20240723111316.png]]
	其中$\{\gamma_i\}_{i\in I}$是$X$的balancing constants

当公式（1）中的能量梯度等于0时，以下关系在有着固定系数的每个点位置满足
![[Pasted image 20240723114231.png]]
设置
![[Pasted image 20240723114250.png]]
有
![[Pasted image 20240723114259.png]]
应用fixed point迭代，有
![[Pasted image 20240723114334.png]]
由于自适应的$\sigma_i^k\in(0,1]$可以自动计算来调整局部沿着points对其方向的斥力，我们只选择参数$\mu\in(0,1/2]$来控制应用在累积点上的全局惩罚等级，见Figure 4
	根据经验设置$\mu=0.35$
# 4.2 Curve skeleton via iterative contraction
给定一个邻域尺寸$h$，前述的迭代投影策略（4）产生了一组点$X=\{x_i\}_{i\in I}$，其表示了局部邻域的L1-median
简单的情况下，这些points直接构成了潜在的shape的一个skeleton，见Figure 4d
然而在更复杂的情况，只有其中一些点表示skeleton的branches，其它的需要进一步收缩，见Figure 5c和6e
![[Pasted image 20240723150914.png]]
![[Pasted image 20240723150926.png]]
接下来我们描述如何识别和标记这些属于skeleton branches的点
	称为branch points，并在图中标记为绿色
然后我们解释如何从剩余的non-branch points（红色）中选择bridge points（蓝色），并且使用它们来保持两个group的connectivity
最后，我们展示branches如何通过逐渐增大邻域尺寸来迭代地grow和merge

从初始的sample points开始，其全部视为non-branch points
sample points基于一个默认初始邻域半径$h_0=2d_{bb}/\sqrt[3]{|J|}$被收缩，其中$d_bb$为输入$Q$的包围盒的对角线长度，$|J|$为$Q$中点的数量
为了标记一个点在收缩后是否是branch point，我们使用与Section 4.1相同的directionality degree度量$\sigma$（2）
	具体来说，我们首先计算所有non-branch points $x_i$的$\sigma_i$，然后在分别的knn（默认k=5）中光滑它们来移除孤立的outliers，即$\sigma_i=\sum_{j\in Knn(i)}\sigma_i/K$
	如果在光滑后，$\sigma_i>0.9$，那么$x_i$被认为是一个branch point的==candidate==，因为点在$x_i$的邻域被很好地排列

为了从这些candidiate中识别branch points，我们首先定位一个seed point，称为$x_0$，其有着最大的$\sigma$值，并以其trace邻近的candidate满足下式，沿着主要的PCA方向：
![[Pasted image 20240723164238.png]]
当上述局部邻域内没有满足上述条件的candidate时，tracing停止
如果至少有5个点在trace中被找到，这些点被标记为branch points
	否则，它们从candidate中被移除
这个过程从一个新的seed point重复，其为剩余的candidate中有着最大的$\sigma$，直到所有candidate都被处理过

如Figure 5c和6e，上述过程剩余的sample points被标记为non-branch points，它们需要在更大的邻域尺寸$h$更进一步收缩来构建新的branches
然而，依靠一个固定的$h$，无论大小，在整个过程中保持不变，这是行不通的
Figure 6提供了一个例子，使用了一个大的$h$，表示小的subparts的points会over-contracted，这样就不会维持median结构
我们的解决方案时逐渐增加$h$，同时排除已经识别为branch points，来做进一步收缩
	即$h_i=h_{i-1}+\Delta h$，其中默认$\Delta h=h_0/2$，直到所有点都成为branch points

投影剩余的non-branch points，同时固定branch points，可能会分成两个groups，导致不l连接的skeleton branches
为了解决这个问题，我们沿着一个识别的skeleton branch的每个端点选择一个bridge point
记branch的端点为$e$，然后其相对应的bridge point $b$是一个non-branch point，满足：
1. 沿着branch的方向，即$eb$和branch方向的角度小于90°
2. 比其它满足1的non-branch更接近$e$
3. 距离$\Vert b-e\Vert$要小

这些bridge points提供branch points和non-branch points之间的恰当连接
	每个bridge point连接着其相对应的branch的端点，且因此保持着与现有branch的连接
另外，作为一个non-branch point，其参与进一步收缩，因此是新的branch的一部分
结果上，从不同邻域尺寸构建的branches被连接，构成完整的skeleton

在收缩过程中应用两个额外的规则：
1. 当一个局部邻域包含两个或以上bridge points时，它们收缩到一个branch point，其连接着原始bridge points所属的所有branch
2. non-branch points被移除，当它们靠近一个现有的branch但不沿着其对其方向，或者它们的邻域内没有其它点
第一个规则可以更好地创建joints，第二个规则帮助清理孤立点

一旦所有点被连接到skeleton，down-sampling和smoothing独立地被应用到每个branch来减少冗余
Figure 5h和6i分别展示了从3D和2D点云生成的最终的L1-medial skeleton
## 4.3 Density-based
L1-median对outliers鲁棒的，展示与Figure 3
如果给定point cloud高度non-uniform，局部中心倾向于偏向点密度高的区域，见Figure 7a
为了缓解这个问题，我们提出在迭代方案（4）中加入局部自适应density weights
![[Pasted image 20240723173641.png]]
在原始输入$Q$中对每个点$q_j$定义weighted local density：
![[Pasted image 20240723173948.png]]
因此，集合$Q$中密集点区域的影响通过weighted local densities在第一项被减轻了
	相比于Figure 7a和7b的结果skeletal point clouds
注意到点密度的局部性被函数$\theta(\Vert p_j-p_j'\Vert)$的支撑邻域参数$h_d$控制
	我们默认设置$h_d=h_0/2$
## 4.4 Re-centering
通过设计，生成的L1-medial skeleton遍历input point cloud附近点的median位置
然而，如果大部分的point cloud是缺失的（Figure 8左侧），一个额外的re-centering步骤可以应用来fine-tune curve skeleton的位置
为了保持高效且避免变形，我们分别在每个branch经过down-sampling后运行re-centering，然后curved branch被smoothed（Figure 8右侧）
![[Pasted image 20240723174900.png]]
取一个branch作为例子
对于branch上的每个点$x$，我们定义plane $P_x$穿过$x$，并且有着normal对齐着branch在点$x$的方向
然后，input point cloud中靠近$P_x$的点被投影到上面
对于大多数自然的形状，我们期望投影点近似一个椭圆
因此，调整$x$的位置到椭圆的中心$c_x$会允许skeleton curve更好地表示input point，见Figure 8
	此处$c_x$的为位置通过求解一个非线性最小二乘问题
# 5 Results and discussions
### Results
### Comparison to ROSA
### Relation to mean shift
### Curve skeleton properties
### Limitations
与surface reconstruction类似，从point clouds中skeletonization通常是一个ill-posed的问题（特别是有缺失数据时）
如果noise或missing数据总量太大，我们的算法可能会缺失某些fine-scale结构，并产生错误的输出，如Figure 15
![[Pasted image 20240723180542.png]]
部分缓解这个问题的一种可能的方法是依赖于比我们现在使用的随机抽样更复杂的抽样方法

对于包含close-by surface sheets，仅使用位置信息很难区分不同表面上的点
因此，我们的算法可能会生成拓扑不正确的skeleton，如Figure 14d
如果有更多可用的信息，如从scanner处获得的法向或向外方向，我们的算法可以得到增强，以产生更准确的结果
# 6 Conclusion and future work
