![[Pasted image 20240703205210.png]]
> @article{cao2023parallel,
  title={A Parallel Feature-preserving Mesh Variable Offsetting Method with Dynamic Programming},
  author={Cao, Hongyi and Xu, Gang and Gu, Renshu and Xu, Jinlan and Zhang, Xiaoyu and Rabczuk, Timon},
  journal={arXiv preprint arXiv:2310.08997},
  year={2023}
}
# 0 Abstract
mesh offset在离散几何处理中起着重要的作用。
本文中我们提出一个并行的保特征的可变距离mesh offset框架。
不同于传统基于距离和法向的方法，一个新的计算offset位置的方法被提出，通过使用动态规划和二次规划，且sharp feature可以在offset后保持。
区别于隐式距离场，提出了基于一个空间覆盖区域（表示为多面体）来计算偏移量的方法。
我们的方法可以生成一个小尺寸的mesh的offset，并且可以达到高质量，没有间隙、孔洞和直角。更进一步，集中加速技术被提出来进行更高效的mesh offset，如grid并行计算，AABB tree，ray计算。
为了证明该方法的有效性和鲁棒性，我们测试了我们的方法在quadmesh数据集上。
# 1 Introduction
可变距离的offset，在许多CAD模型中有着广泛的应用。
更进一步，该技术可以将三角网格变形为空心物体。
然而，如何从多边形表示边界鲁棒地生成一个offset模型当前还是一个很有挑战性的问题。
Fig 1中展示了直接直接offset会造成的自交。
基于距离场的方法可以准确地定义offset后生成的surface，但由于要在隐式表示上采样，常会造成问题，见Fig 2和Fig 3。
此外，对于feature-preserving、可变offset的高效鲁棒的方法研究还很少。

本文主要聚焦于可变距离的mesh offset 问题。
我们提出一个并行的feature-preserving mesh可变距离框架，来生成一个offset mesh，没有间隙、孔洞和自交。
input为三角网格记为Mesh0 和其上每个面期望的offset距离。
output时一个offset三角网格记为Mesh1 。
不同于传统基于distance和normal vector，提出一个新的计算offset position，通过动态规划和二次规划，而不是隐式距离场，提出了一个用多面体表示的空间覆盖区域来计算偏移量。
我们的方法具有以下几个现有方法不具备的优点：
	Variable offsetting：生成offset mesh有着不同的距离，在形状优化中很重要。
	Feature-preserving：我们的offset结果可以保留原始模型的所有尖锐特征。
	Efficient：提出了几种加速技术。
	Light-weight：可以生成小mesh尺寸的offset surface，近似于原mesh。
	Simple：容易实现，可以视为直接offset方法和基于距离场的方法的结合。
# 2 Related Work
# 3 Method
从两个基本方法开始我们的探索：
	第一个涉及直接offset方法，mesh上每个顶点根据其顶点法向进行offset，这种方法产生的结果具有弧状结构，如Fig 5b。
	然后考虑基于距离场的方法。也会出现弧状和球状的特征，如Fig 6a。尽管对于保持尖锐特征存在挑战，但它在三维空间中独特地描绘了一个区域
这两种方法都不能保持尖锐特征，直接offset方法提供了改进的机会，调整偏移量的大小和方向可以潜在地帮助保存尖锐的特征。
对三角网格Mesh0 的每个面face_i，offset的结果在一个多面体中，这个多面体表示了face_i 的空间覆盖，记为F_i 。
## 3.1 Dynamic Programming and Quadratic Programming
这是我们的方法中关键的一步
它结合了dynamic programming和quadratic programming来决定$Mesh_0$中每个顶点的最佳offset位置
最初，我们的offset过程从单个顶点开始，考虑每个顶点的singular offset位置

通过考虑局部网格，可以独立地解决特征保留问题

$Mesh_0$的顶点$V$邻接1邻域的几个面，构成一个==local mesh==
local mesh中给定k个面，我们记每个面为$O_i$，$0\leq i \leq k-1$，$O_i$期望的offset距离为$L_i$，由$O_i$生成的plane表示为$A_i$

为了在mesh上执行variable offset时保留sharp features，由于面之间的offset distance不同，距离不能作为特征保持的度量
相反，使用角度作为度量是一个可行的解决方案
广泛的研究表明，从点$V$ offset的点$V′$，为了保持local mesh的角度属性，从$V′$到$A_i$的距离应该等于$L_i$ 

为了不引起数值错误，设置容差$tole_l$，$tole_r$，确保offset距离落在$tole_l L_i$~$tole_r L_i$
这两个值可由用户选择，默认设置为$tole_l=1.0, tole_r=1.3$

有无数个点可以作为$V′$满足前述距离限制的点
	例如，当$V$的所有1-ring邻域的所有面都落在同一个plane上时
我们的实验表明，在有无限解的空间中选择最接近$V$的点作为$V'$可以获得最佳feature fidelity
因此优化的目标是尽量减少$V$和$V′$的距离

考虑到点对点的距离的二次性质，和点到平面距离定义的一组线性约束，我们的问题符合二次规划的框架

二次规划方程的表述如下：...
平面$A_i$上的随机点为$(x_i,y_i,z_i)$，平面$A_i$的法向缩写为$n_i$

在$Mesh_0$的每个顶点都经过上述动态规划处理过程后，结果会像Fig 8中那样
此外，在上述方法中，进一步优化可以将当前状态再细分成更小的子状态
![[Pasted image 20240703224507.png]]
## 3.2 Reassign by Octree
在3.1节中，每个顶点的生成首先要考虑local mesh孤立的特征，不考虑顶点间的关系。因此在某些情况下可能会出现位置紧密的点的扩散。
为了改善网格质量并降低后续计算的复杂度，我们选择在我们的方法中将紧密的点合并成一个点。

尽管如此，这个过程可能会产生连结效应，由于传递关系。
会让两个很远的点合并，会导致local mesh引起局部扭曲。为了解决这个挑战，我们设计了一种方法，它需要一个初始的合并阶段，然后是随后的分裂阶段。
这需要用八叉树结构将每个合并的顶点集合重新分配为多个离散点（更多细节见Algorithm 2）。
此步骤对后续步骤的影响见Figure 9。
## 3.3 Construct the Spatial Coverage of Each Face in $Mesh_0$
一旦三角面的三个顶点后经历了上述计算，我们可以描绘处对应的空间覆盖范围。这个过程需要用三角形的三个顶点及其对应的偏移顶点进行Delauany四面体化。
然后计算Delaunay四面体的外表面，这个外表明包围的体量表示相应面的空间覆盖范围，如Fig 10所示，F_i 构成多面体。

我们的方法的面数高出几个数量级，在计算交叉时带来挑战。
## 3.4 Build Grid and Parallel Computation of Intersections
为了提高效率和改进并行计算，我们采用空间网格将计算域划分为离散的单元，几何方式呈现为立方体。这种结构化的空间分解通过确保每个党员可以独立处理来促进并行计算。
## 3.5 Extraction of $Mesh_0$ from Cumulative Spatial Coverage
本节中将从离散空间覆盖提出出Mesh1，见Fig 15。对每个单元独立。
### Stage One: Ray-Based detection
在每个单元的空间域中，可以找到许多空间覆盖的实例。
### Stage Two: check each S_ij  internal/external in the Mesh_0
假设采取了向内offset的方法。
### Post processing
完成上述阶段后，将生成一个三角形soup，见Fig 16，Fig 17。
如果生成的质量不满意，建议执行后续的remeshing程序，如…
# 4 Experimental Results
结果数据如Table 2中所示，该数据是执行一个可变offset操作，其中offset距离在D_min 和D_max 之间，每个面的offset距离决定于其几何中心的x轴坐标
# 5 Conclusion and Future Work
未来的研究方向有几个。
对于input mesh质量很差且需要向内offset时，我们的方法仍然能够生成offset surface，但在某些区域，特征重构还有待改进。未来可以在增加局部网格密度来解决。
此外我们希望直接从空间覆盖中生成偏移网格，而不依赖tetwild。