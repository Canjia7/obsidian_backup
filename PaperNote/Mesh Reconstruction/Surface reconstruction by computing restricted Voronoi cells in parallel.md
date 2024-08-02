---
tags:
  - parallelization
  - point_cloud
  - mesh_reconstruction
  - restricted_Voronoi_diagram
---
![[Pasted image 20240802112555.png]]
> @article{BOLTCHEVA2017123,
title = {Surface reconstruction by computing restricted Voronoi cells in parallel},
journal = {Computer-Aided Design},
volume = {90},
pages = {123-134},
year = {2017},
note = {SI:SPM2017},
issn = {0010-4485},
doi = {https://doi.org/10.1016/j.cad.2017.05.011},
url = {https://www.sciencedirect.com/science/article/pii/S0010448517300829},
author = {Dobrina Boltcheva and Bruno Lévy},
keywords = {Surface reconstruction, Point cloud, Restricted Voronoi diagram},
abstract = {We present a method for reconstructing a 3D surface triangulation from an input point set. The main component of the method is an algorithm that computes the restricted Voronoi diagram. In our specific case, it corresponds to the intersection between the 3D Voronoi diagram of the input points and a set of disks centered at the points and orthogonal to the estimated normal directions. The method does not require coherent normal orientations (just directions). Our algorithm is based on a property of the restricted Voronoi cells that leads to an embarrassingly parallel implementation. We experimented our algorithm with scanned point sets with up to 100 million vertices that were processed within few minutes on a standard computer. The complete implementation is provided.}
}
# 0 Abstract
我们提出一个方法来从一个input point set重建3D surface triangulation
方法的主要部分是一个计算，其计算restricted Voronoi diagram
	在我们的特定情况中，它对应于 input points的3D Voronoi diagram 和 一组以points为中心正交于其估计的法向方向的disk 的交叉
方法不需要相关的normal定向（只需要方向）
我们的方法基于restricted Vornoi cells的性质，这导致了并行实现
我们在标准计算机上几分钟处理了多达1亿个顶点的扫描点集，并对我们的算法进行了实验
提供了完整的实现
# 1 Introduction
本文专注于解决reconstruction问题：给定从surface上采样的点集，重建出采样的surface。
通常reconstruction问题分为几步：将从不同view扫描的结果对齐到同一个坐标系下，除去噪声和离群点，估计点的法向，应用reconstruction算法得到surface。
本文贡献：
	本文将已经过滤过的点集作为输入，通过计算它们的restricted Voronoi diagram来连接三角形。
	不需要计算任何三维的结构（grid或空间的三角剖分），与常规的方法的区别在于该方法只需要计算点集的3D Voronoi diagram与以点为中心的disks（与计算的点法向垂直）的交叉。
	可以并行计算。
	不需要一致法向。
	此外还提出了一种提取流形的方法，和一组启发式的后处理方法，以得到干净的网格。
主要的局限性：
	虽然重建算法是被设计于应用在已经过滤过的点集上，但是也可以在真实数据上应用。在这样的数据上操作的话预处理步骤就非常必要，但是这个步骤是很困难的。
这里用一到二次标准平滑迭代近似（投影到一个领域的平均plane来平滑）。
# 2 Related work
有两种重建的方法，区别是重建的结果是近似于input点还是插值input点。
## Approximation methods
目标是对input点集拟合一个smooth surface。
这类方法对data中的噪声、缺陷等能很好应对。
介绍了一些先进的方法，如6、7。
许多这类算法需要计算一个distance field，通过估计每一个点的切平面，并计算到切平面的最短距离来实现，如8、9。
VRIP方法，如10。
通过计算一个指标函数，来局部拟合并least-squares移动的方法，如11、3、12、13、4。
Poisson surface方法，通过解决一个关于inferred solid（梯度和输入法向最匹配）的approximate indicator函数，如11。其它的一些Poisson介绍。
## Interpolation methods
这类方法基于Voronoi-Delaunay思想，生成一个插值为三角化的surface，利用的是部分的input点集。如20。
这类方法常从Delaunay三角化中提取出三角化。
介绍了一些Interpolation重建方法。
# 3 Computing the restricted Voronoi diagram in parallel
Figure 1中展示了我们算法的工作流。
	输入input是一个3D点的集合，不一定含有法向。需要注意的是，法向不需要一致，只需要有个方向就可以了。
	输出output是一个近似于surface的三角mesh，连接着input中的点。
	输出的三角化是一个compact 2-pseudo-manifold，是可定向的，且包含几部分连接的部件和边界，见Section 3.5。
Section 3.1介绍点集的smooth方法。
Section 3.2介绍法向的估计方法。
Section 3.3介绍经典的方法。
Section 3.4介绍的是我们的主要贡献
	关于candidate候选三角形的步骤，通过计算input点的Voronoi cell与以input点为中心的disk的交点。
	与37、1、2、44方法类似的思想。
	由于计算的不是Delaunay三角化而是restricted Voronoi cell（即计算input点的Voronoi cell与以input点为中心的disk的交点），然后根据这些信息计算restricted的Delaunay三角化。
	这使得我们的算法简单，而且容易并行。
Section 3.5介绍了一种新的流形提取方法。
	从一个候选三角形的子集开始，这个子集构建了一个带边界的clean的且可定向的2-manifold流形。
	通过一个一个以任意顺序添加剩下的三角形，且不破坏初始mesh的拓扑性质。
	最后可以得到一个最大的非空manifold mesh。
![[Pasted image 20240802105836.png]]
## 3.1 Kd-tree construction
该算法最基本的操作是：寻找一个point的最邻近。
为了优化这个操作，将input point放在kd-tree中，使用的是ANN库。
## 3.2 Smoothing
由于我们的算法是将三角化插入input point中，所以需要一个clean和准确的点集。
如果输入的是一个充满噪声的点集，如扫描得到的点集。
注意这里可以使用任意的点云平滑技术。

这里使用的是将点投影到其k临近域的最佳plane上。
	点p，其k临近域$\{p_i\}_{i=1}^k$，可以找到p的一个fitting plane，最小化误差项。然后将点p投影到该plane上。
	关于fitting plane的表示，可以表示为该plane的单位法向n，和常数项c。
	设该plane上有一点o，则该plane可以表示为$n^T (x−o)=0$，整理为$n^T x=n^T o=c$。
	如果某点p_i 在该平面上，则应有$n^T p_i−c=0$。
	则k临近域plane的误差项为$e(n,c)=\sum_{i=1}^k(n^T p_i−c)^2$。
计算best fitting plane的过程是并行的，计算完成后更新kd-tree。

smooth过程使用两个参数：
	nb_neighbor为最邻近域内点的数量。
	smooth_iter为smooth的次数。
## 3.3 Normal estimation
如果输入点集没有法向，则在smooth操作中通过计算局部best fitting plane进行估计。
注意该算法不需要对法向进行定向，只需要直到每个点的法向方向即可，后续算法步骤中需要用来创建一个disk。
## 3.4 Candidate Triangles
给定点集$\{p_i \}_{i=1}^n$，以及对应法向$\{n_i \}_{i=1}^n$，半径$r$。
我们的算法计算点的Voronoi cell $V(p_i)$，以及以点$p$为中心$r$为半径的与法向正交的disk $D_i^r$ 。
首先对每一个点构建一个disk $D_i^r$，算法中用一个多边形近似该disk（10个顶点）。
然后对每个点$p_i$在disk $D_i^r$的限制内计算Voronoi cell $V(p_i)$，即$V(p_i )∩D_i^r$ 。
Figure 2中展示了在disk的限制中计算Voronoi diagram的情形。
![[Pasted image 20240802110514.png]]
Figure 4中展示了二维情况中的限制情况。
![[Pasted image 20240802110532.png]]
这样我们就得到了一系列restricted Voronoi点。
候选三角形集合T被定义为restricted Voronoi点的对偶。

disk的半径选择需要考虑采样率，并考虑准确率和计算速度。
小的半径会导致最终mesh出现孔洞（Figure 2中A和B），大的半径会增加计算时间。

该步骤中最基础的操作是对每个点由Voronoi cell修剪disk。
这里用的方法类似于Localized Cocone algorithm（1），只需要使用k邻域就能构造局部的候选三角形。
### Computing Voronoi cells through iterative clipping
Voronoi cell是halfspace的交集，令bisector $Π^+ (i,j)$为分割$(p_i,p_j)$的halfspace，包含了$p_i$的一边。则$V(p_i )=\cap_{j\ne i}Π^+ (i,j)$ 。
实际中，只有一部分点的halfspace与Voronoi edge有关。
因此将分割出来的空间$Π^+ (i,j)$分为两种：有贡献的，没有贡献的。
	由一个没有贡献的bisector裁剪Voronoi cell是不会有改变的。
	反之，Voronoi cell是由有贡献的超平面交叉成的。
将点根据到点$p_i$的距离增序排列为$p_j1,p_j2,…,p_{jn−1}$ 。
令$V_k$为前$k$个半空间的交集，即$V_k(p_i )=\cap_{l=1}^k Π^+ (i,j)$。
令$R_k$为$V_k$空间中以点$p_i$为中心的最大半径，即$R_k=max⁡\{d(p_i,x)|x∈V_k(p_i)\}$

Figure 3中展示了，当$d(p_i,p_j )>2R_k$时，bisection $Π^+ (i,j)$是没有贡献的。即$V_k (p_i )⊂Π^+ (i,j)$。
	证明：
	如果有一个点$p∈V_k (p_i )$，且有一个点$p_j$ 使得$d(p_i,p_j )>2R_k$ 。
	根据$R_k$的定义，$R_k$ 是$V_k (p_i )$中的最大半径，则$V_k (p_i )$中的点到$p_i$ 的距离都要小于$R_k$，即有$d(p,p_i )<R_k$ 。
	由三角不等式$d(p_i,p_j )\leq d(p_i,p)+d(p,p_j)$。
	由于$d(p_i,p_j )>2R_k$，即$d(p_i,p)+d(p,p_j )>2R_k$，且$d(p,p_i )<R_k$，则$d(p,p_j )>R_k$。
	这与$p_j∈V_k (p_i )$矛盾。
得到结论，如果距离p_i 第k+1个远的点$p_(j_k+1)$，到点p_i 的距离大于$2R_k$，则这个点对于$V_k$ 没有贡献，故此时$V_k$就是点$p_i$的Voronoi cell，即$V_k=V(p_i)$。
![[Pasted image 20240802110930.png]]
称第一个满足该条件的$R_k$值为radius of security。
以上的说明对于常规的无限制Voronoi diagram没有什么用，因为有些cell是无界的，R_k 是无限大的，对于这样的cell，需要计算所有的bisector，会造成很大的计算时间。
对于受限制的Voronoi cell是很有用的，计算受限制的Voronoi cell $RV(p_i)=V(p_i)∩D_i^r$，都是有限的cell。

使用Algorithm 1来对每个点构建受限制的Voronoi cell，即disk和Voronoi cell的交点。
	Algorithm 1
	输入：点$x_i$，以该点为圆心的圆盘$D_r$，考虑该点的$n$邻域，点集$P$。
	输出：点$p_i$ 受限制的Voronoi facet $RV(p_i)$。
	首先让RV初始化为圆盘$RV=D_i^r$，且$R_k$ 为当前RV的最大半径，$R_k=max⁡\{d(p,p_i)|p∈RV\}$。
	然后不断遍历该点周围的点，按照距离点$p_i$ 的距离增序。
	每次遍历到一个点$p_(j_k )$ 时，更新$RV=RV∩Π^+ (i,j_k )$，再更新$R_k$ 。
	直到遍历到$d(x_i,x_(j_k ) )\geq2R_k$ 或者$k\geq n$时停止。
![[Pasted image 20240802111411.png]]

注意到每个受限制的Voronoi顶点q都是两个bisector（由$p_{j_1},p_{j_2}$ 生成的）的交叉点在disk内的。
如Figure 4 。
则得到Delaunay三角形，由$p_i,p_{j_1},p_{j_2}$ 构建，记为$T$，即候选三角形。
![[Pasted image 20240802110532.png]]
### Parameters
该步使用一个参数：disk的半径$r$，表示为包围盒对角线的百分之几。
当$r$大于Voronoi cell的宽度时，disk会接触到Voronoi cell的所有边。接着增加r的值也不会对结果产生影响。
因此，从经验上，常设定$r$为包围盒对角线的5%。对于大规模的数据，因为Voronoi cell比较小，可以降低r为包围盒对角线的0.5%，让计算更快。
### Structure of the set of candidate triangles
许多的应用中需要得到一个干净的流形，且有一致法向。
所以我们要用manifold extraction方法来从候选三角面集T中构建一个有效的mesh。
候选三角形集合T可以理解为restricted Delaunay三角形集合$T_3$再加上一些三角形。
	$T_3$的定义为：一个三元数组$(i,j,k)$，表示构成三角形的三个顶点的索引。且满足$p_i$ 的disk和$p_j,p_k$ 的Voronoi cell有交集，$p_j$ 的disk和$p_i$,$p_k$ 的Voronoi cell有交集，$p_k$ 的disk和$p_j,p_i$ 的Voronoi cell有交集。即$T_3$中三个顶点的restricted Voronoi cell相互之间都相交。
	这类三角形容易找到，通过生成所有的索引triple对应于restricted Voronoi点，再字典排序。保持在排序后的列表中，每个triple出现3次（因为每个点会产生一个triple）。
	结果如Figure 5A。
在排序后的列表中，有的triple只出现了1次或2次，将它们定义为$T_{12}$ 。
见Figure 5B，比$T_3$ 中的三角形不那么可靠，会容易产生非流形的结构。但是也有一些信息，可以方便后续填补$T_3$ 中的孔洞。
![[Pasted image 20240802111906.png]]
整体思路为从$T_3$ 三角形开始，再谨慎地添加$T_{12}$中的三角形，每次只添加一个三角形，已确保得到的mesh是流形。如果不是流形，则拒绝该三角形。具体见下一section。
## 3.5 Manifold mesh extraction
首先根据$T_3$ 构建一个可定向的2-流形mesh，然后用余下的三角形迭代填补孔洞，不造成额外的拓扑错误。
初始化output mesh为所有的$T_3$ 三角形，然后用excess操作去除所有关联于非流形的边和非流形的点。
	一个非流形的边会连接多余2个三角形。
	by-excess的非流形点会有一个三角形构成的闭环在其邻域。
	定向初始mesh的时候检查是否有Moebius环，有的话移除loop的最后一个三角形。
	此时初始的mesh可以由一个点查询邻接的三角形直到查询完全部三角形。
然后填补初始mesh的孔洞，用剩下的三角形$T_12$ 。根据以下的规则进行测试，从最简单的到最耗时的，只有通过所有规则的三角形才会被加入到最终的surface中。
	Geometric test：确保其邻接三角形的法向正确，不能形成sharp angle，取为max_N_angle，常为pi/3。
	Combinatorial test I：确保三角形正确连接在当前mesh中，每个新三角形需要有两条边邻接当前的三角形，或者一条边且有一个孤立顶点。
	Combinatorial test II：检查非流形边，检查候选三角形的三条边是否是流形。
	Combinatorial test III：检查插入新三角形后不会产生by-excess非流形点。
	Combinatorial test IV：检查插入的新三角形邻接的两个三角形是否有不一样的法向方向，以及插入后是否会生成Moebius环。

输出的结果是以最大可能性连接input的三角网格，是可定向的2-manifold，没有非流形的边，但可能有非流形的点，如pinched torus。可能有几个连接的部分和边界。
Parameters
这部分中使用的参数是两个三角形进行连接的最大偏差角度max_N_angle，通常设为pi/3。
## 3.6 Post-processing
在提取网格操作后，得到的mesh可能含有一定数量的孤立孔洞，可能是由于在尖锐特征采样不足导致的。因此增加一个后处理步骤来修补这些洞。
算法中需要用户指定一个最大修补孔洞尺寸参数maximum hole size，表示为边的数量。
然后寻找simply connected hole并用loop-split算法修补（48），该算法将孔洞迭代分裂，直到获得由3条边构成的孔洞，再将其用三角形填补。

为了确保孔洞都是simple loop的，首先寻找并移除包含bridge的孔洞，见Figure 7。
bridge三角形是：沿着孔洞的边界走，会遍历不止一次的三角形。
	该定义不会捕获宽于两个三角形的bridge。
![[Pasted image 20240802112133.png]]
在后处理步骤中，同时会根据尺寸（facets的数量）移除一些小组件。
### Parameters
该步骤中选择两个参数：
	需要填补的孔洞的尺寸上限，以边的数量为单位，默认为500。
最小组件的尺寸下限，以facet为单位，默认为10。
# 4 Experimental results and Discussion
对比四种先进的重建方法：ScaleSpace reconstruction（51），Screened Poisson（3），SuperCocone（1），Wavelet reconstruction（12）。
实验中使用的数据是类似于人工场景，点集是从一个已知surface上采样得到的，数据点集大小从几千到百万不等。
## 4.1 Comparison - Hausdorff distance to reference surface
首先测试在不同采样条件下，是否能够重建得到与previous work相同的surface。
通过测量reference surface和重建出的output的Hasudorff距离。
采样见Figure 9。
同时测试有无后处理步骤的情况，结果见Table 1。
我们的方法的误差和先进的方法类似。
## 4.2 Real-scale data
测试用数据集来自（52，53，54，55，49，50），也是其它先进方法测试用的数据。
Figure 8中列出了各种算法所使用的时间。
Figure 10中列出了不同规模数据集用时变化。
### Non-uniform sampling
Figure 11展示了horse模型在不同区域以不同采样密度采样100K个点的重建结果。
### Small details
Figure 12展示了一个有许多小的凹凸不平特征的clean的点集。
### Noise
实际的数据集常由几个range map组合而成，常见方法常会在该区域生成两层点而产生错误，见Figure 15。
可以作1-2次smooth来应对大部分情况。
Figure 13中展示了对noisy点集的重建表现。
Figure 14中展示了都做1次smooth后各种重建方法的结果。
我们的方法需要一个非常温和的smooth，以不洗掉任何小的细节，如Figure 16中，不丢失细节
### Impact of pre and post-processing steps
Figure 17，Figure 18展示了预处理步骤和后处理步骤对结果的影响。
没有预处理步骤，重建结果会有很多噪声，和边界边。
没有后处理步骤，重建结果会有很多孔洞。
### Large datasets
对于大数据集，基于Delaunay的方法，如ScaleSpace和SuperCocone会遇到内存限制。
ScaleSpace方法会运行崩溃，SuperCocone会给出不正确的结果，具体见Figure 19和Figure 20。
Poisson方法会因为一些不正确的法向，而产生bubbles，Wavelets方法也有类似的问题。这类问题不难修复，但是需要一致法向。
Figure 22和Figure 21展示了另一个大数据模型重建，以及介绍了一些大规模点集运行的时间和存储消耗。
# 5 Conclusion and Discussion
本文介绍了一种从3D的点集重建出surface的算法，基于简单的restricted Voronoi cell思想。
简单的实现，可以做到并行，以及对内存资源消耗经济。
因此可以在大数据集上很好地运行，计算时间在5min内。

对比当下先进的算法：
	在填补孔洞方面不如Poisson和Wavelets方法，这两个方法构建一个smooth的近似mesh。
对比于interpolation方法，我们的方法表现上类似于Cocone和ScaleSpace方法。

我们的算法提取出一个input点的Delaunay三角化，限制在disk的并集中。
实验中，我们的算法类似于Cocone算法。

在运行时间上，我们的算法是最快的。
我们的算法不用计算3D的Delaunay三角化，而是直接计算Restricted 2D的Delaunay三角化。

不像Poisson类方法，我们的算法不需要一致的法向，只需要法向的方向即可。
当数据出现错误时，我们的算法需要做一个平滑。

算法耗时上，manifold提取和后处理步骤占大部分时间。
后续可以考虑优化补洞操作，这个操作还未做到并行。