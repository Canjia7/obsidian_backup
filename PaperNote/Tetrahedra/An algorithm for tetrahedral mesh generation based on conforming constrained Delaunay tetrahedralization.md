> @article{YANG2005606,
title = {An algorithm for tetrahedral mesh generation based on conforming constrained Delaunay tetrahedralization},
journal = {Computers & Graphics},
volume = {29},
number = {4},
pages = {606-615},
year = {2005},
issn = {0097-8493},
doi = {https://doi.org/10.1016/j.cag.2005.05.011},
url = {https://www.sciencedirect.com/science/article/pii/S0097849305000932},
author = {Yi-Jun Yang and Jun-Hai Yong and Jia-Guang Sun},
keywords = {CCDT, Finite element methods, Octree},
abstract = {An unstructured tetrahedral mesh generation algorithm for 3D model with constraints is presented. To automatically generate a tetrahedral mesh for model with constraints, an advancing front algorithm is presented based on conforming constrained Delaunay tetrahedralization (CCDT). To reduce the number of visibility tests between vertices with respect to model faces as well as the computation of constrained Delaunay tetrahedra, a sufficient condition for DT (constrained Delaunay tetrahedralization whose simplexes are all Delaunay) existence is presented and utilized coupled to uniform grid and advancing front techniques in our algorithm. The mesh generator is robust and exhibits a linear time complexity for mechanical models with uniform density distribution.}
}
# 0 Abstract
提出了一种为带限制的3D model的unstructed tetrahedral mesh generation算法
为了自动生成一个带限制model的tetrahedral mesh，基于conforming constrained Delaunay tetrahedralization（CCDT）的一个advancing front算法提出
为了减少顶点之间遵从于model faces的visibility tests的数量，也即constrained Delaunay tetrahedra的计算
一个DT（constrained Delaunay tetrahedralization中simplexes都是Delaunay的）的充分性条件被提出，并在我们算法中耦合到一个uniform grid和advancing front技术
mesh generator在有着均匀密度分布的机械模型是robust且是线性时间复杂度
# 1 Introduction
方法：
1. 一个surface meshing算法用于来mesh化model，为了保证机械模型的CDT存在性，missing grazeable edges的重点被反复地添加到mesh中，直到所有grazeable edges都被表示为DT中continuous edges的集合
2. interior points被插入到一个face-centered-cubic（FCC）晶格排列中，并结合八叉树空间划分，在error bound上，它比其它Bravais晶格排列给出了更好的tetrahedra
3. 最后，为了加速tetrahedral mesh generation，一个DT存在性的充分性条件被提出，并利用该条件和uniform grid实现了一种advancing front算法
对于均匀密度分布的模型，mesh generator具有robust和线性时间复杂度
# 2 Grazeable edge protection
Shewchuk在[[Constrained Delaunay Tetrahedralizations and Provably Good Boundary Recovery]]中给出了CDT存在的充分条件：
	如果一个model是grazeable edge protected的，则能保证CDT的存在性

FEM中，surface mesh可以做局部修改，只要不违反boundary和constraints
基于这一特性，在添加Steiner point之前，先去除surface triangulation的所有non-constraint interior edge
为了得到grazeable edge protection，missing grazeable edges的重点被反复插入到mesh中，直到所有grazeable edges被表示为DT中continuous edges的集合

model中5个或更多的顶点cospherical可能会导致CDT算法失败
因此，引入symbolic perturbation来模拟一个common sphere上没有五个顶点的情况
# 3 Interior points insertion via FCC lattice
tetrahedra的size是由density函数控制的，其可能在body范围内变化
为了生成规定size的tetrahedra，在一个耦合八叉树空间细分的FCC晶格排序中插入interior poins，局部lattice参数由规定的desity function确定

...

一旦构建八叉树，所有的boundary和interior leaves都被指定为FCC cells（Figure 4a），cell的corners和face的centers在body内的都被插入到mesh中
![[Pasted image 20240628155147.png]]
Radovitzky和Ortiz【10】比较了FCC和其它13种Bravais lattice，得出结论：
	对于unit cell，FCC晶格排序的CDT tetrahedra质量（Figure 4b）优于其它的，根据它们定义的aspect ratio

根据FCC晶格排序，boundary leaves的corners和faces的centers可能位于body外部
为了解决这一问题，实现了一种传统的判断点在model内外的算法：
	从point发射一个radial，计算其与boundary faces的相交次数

最后，在model triangular faces的minimal circumspheres内的interior point都被消除，以避免slivers

插入interior points不影响CDT的存在性
# 4 Construction of uniform grid
# 5 Tetrahedralization
我们假设带限制的model存在perturbation，且是grazeable edge protected的
当model有一个CDT其中的tetrahedra都是Delaunay的，则称model有一个==DT==
visibility determination是最耗时间的，为了减少在model的顶点间遵从于model faces的visibility tests的数量，我们推进我们的advancing front算法中对DT存在性的充分必要条件

为了利用这一条件，所有model faces被分类为：Delaunay faces（也是strongly Delaunay的）和undelaunay faces
称一个model是==face protected==的，即如果其所有faces都是strongly Delaunay的
测试一个model是face protected的，也edge protection类似

如果一个有限制的model $X$，是face protected的且所有grazeable edge protected$，则$$X$有一个DT（也是唯一的DT）
如果一个model $X$有一个DT，DT可以通过一个labeling proceduce从model顶点获得
	否则，在我们的advancing front算法中首先处理undelaunay faces
boundary和constraint triangles被指定为initial front
随着undelaunay faces被处理，front也在前进和更新
如果当前front在mesh过程中是face protected的，被当前front包裹的Delaunay tetrahedra通过一个labeling procedure追加到tetrahedral list
在机械模型被meshed后，mesh的质量通过edge-face swapping和smoothing操作提高
## 5.1 Tetrahedron generation
假设没有五个或更多顶点在一个common sphere上（用符号扰动来避免）
$F=\{f_0,f_1,...,f_{m-1}\}$是model的面集，$f_i=(p_{i_1},p_{i_2},p_{i_3})$
$P=\{p_0,p_1,...,p_{n-1}\}$是model的顶点集，$p_i=(p_{i,x},p_{i,y},p_{i,z})$
$\Omega$是由boundary faces包裹的body，除了interior constraints
$f(p_{i_1},p_{i_2},p_{i_3})$为active front triangle
$\tilde{S}(p_{i_1},p_{i_2},p_{i_3},p_{i_4})$记为tetrahedron $t(p_{i_1},p_{i_2},p_{i_3},p_{i_4})$的外接球$S(p_{i_1},p_{i_2},p_{i_3},p_{i_4})$的严格内部区域

对face $f(p_{i_1},p_{i_2},p_{i_3})$，tetrahedron $t(p_{i_1},p_{i_2},p_{i_3},p_{i_4})$是constrained Delaunay的 $\Leftrightarrow$：
1.  $p_{i_4}\in V_{i_1 i_2 i_3}$，$V_{i_1 i_2 i_3}=\{p\in P | p_{i_1}p_{i_2}p\subset \Omega, p_{i_2}p_{i_3}p\subset \Omega, p_{i_3}p_{i_1}p\subset\Omega\}$ 且
2. $vS(p_{i_1},p_{i_2},p_{i_3},p_{i_4})\cap V_{i_1 i_2 i_3}=\emptyset$

令$H_{i_1 i_2 i_3}=\{p|(p-p_{i_1})\cdot d_i>0\}$为由$f(p_{i_1},p_{i_2},p_{i_3})$定义的half-space，且front advancing方向为$d_i$
则有$V_{i_1 i_2 i_3}=V_{i_1 i_2}\cap V_{i_2 i_3}\cap V_{i_1 i_3}\cap H_{i_1 i_2 i_3}$，其中$V_{i_1 i_2}$是从$p_{i_1},p_{i_2}$可见的顶点$p\in P$集合

如果一个vertex满足（1）被找到，则顶点满足（1）和（2）可以通过重复替换$p_{i_4}$和顶点$p\in\tilde{S}(p_{i_1},p_{i_2},p_{i_3},p_{i_4})\cap V_{i_1 i_2 i_3}$找到
## 5.2 A sufficient and necessary condition for DT existence
为了减少model顶点之间的visibility tests数量，Radovitzky给出了一个DT存在的充要条件：
	所有boundary triangles都有空的最小外接球
但这个条件太严格，我们给出一个DT存在性的充要条件

给定一个model有着surface triangulation
triangular face $f(p_{i_1},p_{i_2},p_{i_3})$是==strongly Delaunay==的充要条件是： 存在一个顶点$p_{i_4}$，使得：
	1. $p_{i_4}\in V_{i_1 i_2 i_3}$，$V_{i_1 i_2 i_3}=\{p\in P | p_{i_1}p_{i_2}p\subset \Omega, p_{i_2}p_{i_3}p\subset \Omega, p_{i_3}p_{i_1}p\subset\Omega\}$ 且
	2. $(S(p_{i_1},p_{i_2},p_{i_3},p_{i_4})\cup\tilde{S}(p_{i_1},p_{i_2},p_{i_3},p_{i_4}))\cap P=\{p_{i_1},p_{i_2},p_{i_3},p_{i_4}\}$
	（<font color=red>即存在包含该三角形的四面体在boundary内部，且该四面体是strongly Delaunay的</font>）
如果$F$的triangular faces都是strongly Delaunay的，我们称model是==face protected==的
### Theorem 1
一个model $X$是grazeable edge protected的且face protected的 $\Leftrightarrow$ $X$有一个DT
proof.
	$\Rightarrow$：
	grazeable edge protection确保dangling edges（不包含在任何face中）不侵犯Delaunay性质
	如果所有faces都是strongly Delaunay的，它们必定存在于model vertices的DT中
	因此，被boundary faces包裹的Delaunay tetrahedra构成了model的一个DT
	$\Leftarrow$：
	假设没有五个顶点共圆
	Delaunay edges都是strongly Delaunay的
	如果model不是grazeable edge protected，则存在一条边没有一个空circumsphere，包含这条边的CDT的tetrahedron也没有一个空circumsphere，因此model没有一个CDT
	如果model不是face protected的，则存在一个face没有一个空circumsphere，产生自该face的tetrahedron也不是Delaunay的，因此model没有一个CDT

测试face protection也grazeable edge protection类似

...

该技术可以很快地加速CDT，特别是model有着集合regular boundary faces，这也是FEM中tetrahedral generation的先决条件
2D的例子见Figure 5
	2D中，如果每条边都是strongly Delaunay的，则PSLG有一个CDT，其中simplexes都是Delaunay的
	因此，Theorem 2可以应用到2D CDT
![[Pasted image 20240628200603.png]]
## 5.3 Front update
当一个新tetrahedron被构建后，生成了三个faces和front推进
active triangle总是被从front删除，且undelaunay face数量减1
undelaunay face数量更新如下：对每个生成的face，如果face是：
1. 一个新的undelaunay face，数量增加1
2. 一个新Delaunay face，数量不变
3. 一个undelaunay face已经在front存在，数量减少1
4. 一个Delaunay face已经在front存在，数量不变
front更新如下：如果一个顶点在Section 5.1中被找到是：
1. 不是邻接于active triangle的front triangles的一个顶点，三个triangles被添加到front，Figure 6a
2. 是邻接于active triangle的front triangles的一个顶点，两个faces被添加到front，且adjacent triangles被从front中删除，Figure 6b
3. 是邻接于active triangle的两个front triangles的一个顶点，一个face被添加到front，adjacent两个triangles被从front删除，Figure 6c
4. 是邻接于active triangle的三个front triangles的一个顶点，没有faces被添加，三个adjacent triangles被删除，Figure 6d
![[Pasted image 20240628201400.png]]
（<font color=red>灰色的face为front中相关的面</font>）
# 6 Examples