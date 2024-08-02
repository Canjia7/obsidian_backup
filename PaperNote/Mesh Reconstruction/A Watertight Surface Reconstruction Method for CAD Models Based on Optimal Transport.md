---
tags:
  - point_cloud
  - feature_preserve
  - mesh_reconstruction
  - robust_geometry_processing
  - optimal_transport
  - mesh_refinement
---
![[Pasted image 20240801203626.png]]
> Yuanyan Ye, Yubo Wang, Juan Cao, Zhonggui Chen. A Watertight Surface Reconstruction Method for CAD Models Based on Optimal Transport. Computational Visual Media, 2023.
# 0 Abstract
从点云进行feature-preserving的mesh reconstruction是有挑战性的任务
implicit方法倾向于去拟合smooth surface，且不能重建sharp features
explicit方法对noise敏感，且只有在points分布在feature lines才能interpolate sharp features

我们提出了一种watertight surface reconstruction方法，基于optimal transport
	其可以忠实地重建CAD模型上经常出现的sharp features
我们将surface reconstruction问题形式化为最小化point cloud和重建surface之间的optimal transport cost
算法包括initialization和refinement步骤
	initialization步骤中，point cloud的凸包被变形，在transport plan的指导下，得到一个初始的近似surface
	接下来，使用operations（包括vertex relocation和edge collapses/flips）优化mesh surface，以得到feature-preserving结果
实验表明我们的方法可以保持sharp features，同时对noise和缺失数据鲁棒
# 1 Introduction
...

我们的工作的主要贡献如下：
1. 我们介绍了一种feature-preserving surface reconstruction算法，基于optimal transport，其将surface reconstruction问题形式化为最小化point cloud和重建mesh之间的optimal transport
2. 为了避免由于transportation误差产生的self-intersection，我们提出在initialization步骤中，将局部邻域映射的方差，加入到optimal transport的代价函数中
3. 我们定制了optimal transport代价驱动的vertex relocation、edge collapse、edge flip操作来优化mesh，并很好地重建sharp edges和corners
# 2 Related Work
## 2.1 Surface reconstruction
### Feature-preserving reconstruction
### Deformation-based reconstruction
## 2.2 Optimal transport
# 3 Overview
目标是构建一个watertight的三角网格$\hat{M}$近似于给定的point cloud $P=\{p_1,p_2,...,p_n\}$，同时保持3D模型的sharp features
我们将surface reconstruction视为最优化input point cloud和重建mesh surface的离散optimial transport cost
	$P$和$\hat{M}$之间的transport plan的表示，是通过$P$和从mesh surface$\hat{M}$采样的点$S=\{s_1,s_2,...,s_n\}$的映射

定义三角网格，通过一组vertices、faces、edges $(V,F,E)$，并将每个face视为以 有着unit capacity的sampling points 为中心的Dirac度量的加权和
	$P$中的每个点也被视为一个以$p_i$为中心的有着unit mass的Dirac度量
$P$和$S$的尺寸都是$n$
因此，减少reconstruction error等价于最小化$P$和$S$之间的离散optimal transport cost，通过优化以下式子：
	![[Pasted image 20240801203207.png]]
	其中transport plan $\pi_{ij}$为从点$p_i$到sampling point $s_j$的mass transported，值严格为1或0
	$P$和$S$之间的映射是一一对应的
	$C_{ij}$为$p_i$和$s_j$的transport cost
		cost function可能会因为application而变化，一个常见的选择是欧氏距离

给定一个point cloud，其convex hull选择为shrink-wrap实现的watertight surface（Figure 1b）
mesh首先变形为粗糙地拟合point cloud，并优化其来recover sharp features（Figure 1c-d）
![[Pasted image 20240801203626.png]]
方法的两个主要步骤：
### Initialization
初始mesh构建为remeshing了input point cloud的convex hull，如Figure 1
我们在初始mesh上采样，并在input point cloud和当前sampling点之间计算$\pi_{ij}$
初始近似mesh通过依据$\pi_{ij}$进行变形构建的，计算细节见Section 4
### Refinement
我们执行优化算子，包括vertex relocation、edge collapse、edge flips，通过调整mesh的拓扑连通性和顶点位置来减少transport cost，细节见Section 5
# 4 Initialization
## 4.1 Vertex displacement
为mesh的每个face计算transformation矩阵，基于optimal transport plan，来变形初始mesh
新的顶点位置由其1邻域faces共同决定，每个face的transformation矩阵由其sampling points的transport plan驱动

假设face $f$的sampling point $B=\{b_1,b_2,...,b_k\}$，$B$在point cloud中的对应点$Q=\{q_1,q_2,...,q_k\}$
$B$和$Q$之间的旋转矩阵$R$，旋转向量$t$，缩放比例$s$，可以通过优化以下等式：
	![[Pasted image 20240801204804.png]]
由此，vertex $v$在face $f$的新位置可以计算为$\hat{v}=sRv+t$
vertex $v$的最终位置是其邻接面推到出的新位置的平均，见Figure 2
![[Pasted image 20240801205020.png]]
## 4.2 Calculation of optimal transport
当我们使用convex hull来构建初始mesh时，其shape与input point cloud有很大的不同
如果将cost function简单地设置为欧氏距离的平方，那么在upper surface接近于lower surface的thin region可能会发生不正确的传输
Figure 3 left展示了optimal transport plan使用欧氏距离的平方作为cost function
	目标点的upper和lower surfaces在右侧是很接近的，且在区域内的一些source points应该被传输到lower surface的传输到了upper surface，这将导致初始mesh出现concave和自交
![[Pasted image 20240801205725.png]]

为了防止将point cloud中的邻接点映射到scattered点，我们采用variance-minimizing transport【Mandad et al.】来计算input point cloud和sampling points之间的optimal transport plan
对于点$p_i$和其在$P$中的邻居$p_k$，我们希望它们的相对位置关系在映射后仍然保持
	因此，我们为$p_k$设置不同的权重值，基于其到点$p_i$的距离，并使用标准化Gauss权重函数如下，来决定哪些点是点$p_i$的邻居
	![[Pasted image 20240801211756.png]]
	其中
	![[Pasted image 20240801211812.png]]
	其中$\sigma$设置为input point cloud的平均点间距$d_{avg}$
	如果$w_{p_k}$大于一个阈值$\epsilon$，则$p_k$认为是$p_i$的一个邻域点，$\epsilon=exp(-d^2_{avg}/2\sigma^2)$
我们记$p_i$的邻域点为$N_{p_i}$，且其邻域点的mass指定为其Gaussian权重

mapped points的邻域可以用variance来表示
假设每个点的mass在$X=\{x_1,x_2,...\}$是$m_i$，$X$和其中心$\eta$的variance可以被计算为：
	![[Pasted image 20240801212245.png]]
	其中$d$为欧式距离
假设$N_{p_i}$的mapped points在$S$是$\pi_P(N_{p_i})$，邻域到邻域的映射可以通过优化$\pi_P(N_{p_i})$随其中心$\eta_{p_i}$的variance来实现
	为此，我们在optimal transport（1）的计算中采用以下cost function：
	![[Pasted image 20240801212914.png]]

我们交替优化公式（1）：
	首先固定transport plan，并对每个点优化中心$\eta_{p_i}$和$\eta_{s_j}$
	其次，根据中心优化transport plan
优化后，input point cloud的邻域会被映射到sampling points的邻域，如Figure 3 right
随着transport plan的更新，存储着从$p_i$到$s_j$的传输成本的成本矩阵需要更新，Algorithm 1描述了initialization步骤中构造初始近似mesh的过程
![[Pasted image 20240801213404.png]]
# 5 Refinement
初始近似mesh只是粗略拟合point cloud
我们将在refinement步骤中使用vertex relocation、edge collapse、edge flip操作来微调mesh，使得重建结果能够恢复sharp features，进一步减少近似误差

refinement算子也是由transport cost驱动的，但cost function不同于Section 4
计算variance minimizing triansport plan需要维持一个$n\times n$大小的cost矩阵，这消耗了大量的内存和运行时间
为了提高速度，我们使用欧式距离的平方来作为cost function
则不会导致不近视的transport plan，因为input point cloud和mesh此时已经很接近了
...

Algorithm 2给出了整个refinement过程，我们将在下面的部分中详细描述每个refinement操作
![[Pasted image 20240801213906.png]]
## 5.1 Vertex relocation
为了减少重建误差，我们进一步减少点到mesh surface的距离，通过移动顶点
假设face $f(v,v_1,v_2)$的sampling point $s_j$有重心坐标$(\alpha_j,\beta_j,\gamma_j)$对应于$(v,v_1,v_2)$
	$s_j$的对应点是$p_i$，且点$p_i'$是$p_i$到一条穿过$s_j$且方向为face normal的line的投影
顶点$v$的新位置可以有下式得出：
	![[Pasted image 20240801214314.png]]
由此，与$v$相邻的每个三角形$f$都推导出$v$的新位置，表示为$v_f^*$
$v$的最终位置$v^*$计算为：
	![[Pasted image 20240801214433.png]]
	其中$\pi_f$是到surface $f$的总mass transported

vertex relocation分为两步执行：
	首先我们固定transport plan $\pi_{ij}$并为每个顶点根据$\pi_{ij}$以及Eq. (7)计算新位置
	将vertex移动到其新位置，如果不造成face flipping的话
	然后，重新采样当前mesh，并更新transport plan $\pi_{ij}$

Figure 4展示了vertex relocation后的mesh变化，部分feature edges已经恢复
![[Pasted image 20240801214751.png]]
## 5.2 Edge collapse
vertex relocation操作仅更新顶点位置，并不改变mesh的连通性
mesh可以通过edge collapse进一步refined
经典的mesh简化方法仅基于平方误差度量，但其仅考虑了原始mesh的几何误差
相反，我们的edge collapse的误差度量设置为optimal transport cost，且sharp features在edge collapse操作后更好地恢复
我们的方法中，如果使用halfedge collapse来mesh简化，可能产生slim三角形，从而导致网格质量低，并且在后续处理中出现face flipping
	因此，我们不使用halfedge collapse
	我们首先collapse edge到其中点，并为该点寻找optimal位置，通过vertex relocation

为了决定哪些edge需要被collapse，一个edge collapse模拟被执行，来计算操作前后的transport cost改变
有着最大减少的transport cost的edge被选择collapse
如果每个edge collapse都计算全局transport cost，很耗时
	仅计算被影响面的代价变化

Figure 5展示了edge collapse的结果，collapse后的新顶点的位置可以更好地recover feature edge
![[Pasted image 20240801215822.png]]
## 5.3 Edge flip
edge flip是提高网格质量并减少重建误差的简单高效的操作
edge flip操作在plane regions中邻接faces角度大于阈值$\theta_2$执行
	我们flip所有edges满足条件（与edge相对的两个内角之和大于其它两个角之和），如Figure 6
	它将移除narrow三角形，并提高mesh质量
![[Pasted image 20240801220108.png]]

然后，我们在non-planar上邻接faces的角度小于阈值$\theta_3$的区域执行edge flip
	只有在能降低transport cost的情况下才执行edge flip，如Figure 7
我们将满足上述条件的edge加入按transport cost降序排列的优先队列，flip一条边后更新受影响的edge
![[Pasted image 20240801220312.png]]
# 6 Experiments
## 6.1 Implementation details
假设sampling points和input point cloud规模相同，需要确定每个face有多少采样点
在initialization step中，我们假设point cloud和sampling points是均匀分布的
每个face的sampling points的数量与face面积成正比
我们还确保每个face至少包含一个sampling point
实践中的points通常是不均匀分布的
在refinement步骤中，我们不保持上述假设
	我们临时将point cloud中的每个点分配给离它最近的三角形
	face上的sampling points的规模就是离它最近的点的数量
	然后，每个face使用CVT进行重采样
## 6.2 Feature-preserving
![[Pasted image 20240801220901.png]]
![[Pasted image 20240801220936.png]]
## 6.3 Robustness analysis
## 6.4 Reconstruction of smooth surfaces
# 7 Conclusion
...

尽管提出的算法达到了与其的效果，但仍存在一定局限性
对内存和时间要求非常高
