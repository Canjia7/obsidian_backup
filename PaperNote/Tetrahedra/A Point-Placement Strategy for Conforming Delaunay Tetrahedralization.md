> @article{doi:10.1142/S0218195901000699,
author = {MURPHY, MICHAEL and MOUNT, DAVID M. and GABLE, CARL W.},
title = {A POINT-PLACEMENT STRATEGY FOR CONFORMING DELAUNAY TETRAHEDRALIZATION},
journal = {International Journal of Computational Geometry \& Applications},
volume = {11},
number = {06},
pages = {669-682},
year = {2001},
doi = {10.1142/S0218195901000699},
URL = {https://doi.org/10.1142/S0218195901000699},
eprint = {https://doi.org/10.1142/S0218195901000699},
abstract = { A strategy is presented to find a set of points that yields a Conforming Delaunay tetrahedralization of a three-dimensional Piecewise-Linear complex (PLC). This algorithm is novel because it imposes no angle restrictions on the input PLC. In the process, an algorithm is described that computes a planar conforming Delaunay triangulation of a Planar Straight-Line Graph (PSLG) such that each triangle has a bounded circumradius, which may be of independent interest. }
}
# 0 Abstract
提出了一种策略来寻找一组点，使其能产生一个3D Piecewise-Linear Complex（PLC）的Conforming Delaunay tetrahedralization
该算法的新颖之处在于：其对输入PLC没有角度限制
在过程中，描述了一种计算Planar Straight-Line Graph（PSLG）的planar conforming Delaunay triangulation的算法，使得每个triangle有一个有界的circumradius，其可能与interset独立
# 1 Introduction
## 1.1 Piecewise-linear representations
## 1.2 Conforming Delaunay triangulations
==However, it should be noted that Constrained and Conforming-constrained Delaunay triangulations and tetrahedralizations are not as helpful with some numerical schemes because the quality requirements imposed on internal boundaries (material interfaces) imply that triangles incident upon these edges are “locally Delaunay.”==
### Lemma 1.1
有着点集$V$的PSLG $P$中的一条边$e$，其是相对于点集$V$的Delaunay triangulation $\Leftrightarrow$ 存在一个circle穿过$e$的顶点并且其内部不包含$V$的任何点

Steiner points可以放置在$e$上的tangency，如Figure 1b
Steiner points放置在disk和$e$的交点上，如Figure 1a
![[Pasted image 20240628095404.png]]
可以看作是conforming Delaunay triangulation的存在性证明，并不实用
## 1.3 A strategy for comforming Delaunay tetrahedralization
我们的目标：给出一个算法寻找一个没有角度限制的PLC的conforming Delaunay triangulation
三维情况更复杂
### Lemma 1.2
有着点集$V$的PLC $P$的一个triangular face $f$（或是有着四个或以上cocircular顶点的face）是$V$的Delaunay tetrahedralization的一个face $\Leftrightarrow$ 存在一个sphere穿过$f$的顶点且内部不包含$V$的点

我们的算法受到以下观察的启发：
	假设给定$\mathbb{R}^3$中有一组不相交的faces，我们希望对这些faces进行细化，使增广点集conform于这些faces
	一个充分但不必要的条件是（$\Rightarrow$但不$\Leftarrow$）：对每个face寻找一个planar Delaunay triangulation，其性质是对每个triangle $t$，半径等于 $t$的planar circumcircle 的circumscribing sphere不与其它face相交
	注意到这个sphere是外接$t$的最小半径球之一
	一个每个face的planar conforming Delaunay triangulation，其每个triangle的radius有一个被到PLC最近面的距离为界限，将满足此条件
# 2 Delaunay triangulations with bounded circumradii
我们希望寻找PSLG的conforming Delaunay triangulation，使得每个triangle的circumradius由一个预先指定的常数限定
Chew的方法生成一个constrained Delaunay triangulation，使得triangulation的角度在30°到120°之间，同时生成circumradii有界限的triangles
然而，用他的算法到我们的任务上，需要解决两个问题：
1. 算法的前提条件是输入角度不能小于30°
2. 算法不能保证生成conforming Delaunay triangulation
## 2.1 Review of Chew's algorithm
## 2.2 Treating the small angles
![[Pasted image 20240628103341.png]]
## 2.3 Protecting the input edges
### Theorem 2.1
一个PSLG可以被refined，使得得到一个conforming Delaunaytriangulation，其有着性质：每个Delaunay triangle的circumradii被一个预先指定的常数限定，使用有限数量的Steiner points
# 3 Extending to three dimensions
## 3.1 Overview
## 3.2 Creating the buffer-zones
### Theorem 3.1
一个PLC $P$可以被refined，使得得到一个其增广点集的Delaunay tetrahedralization，conform于$P$，用有限数量的Steiner points
# 4 Conclusion
我们的算法放置了太多的点而不实用，应该更多地被视为存在性证明