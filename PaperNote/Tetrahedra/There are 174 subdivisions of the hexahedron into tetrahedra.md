> @article{10.1145/3272127.3275037,
author = {Pellerin, Jeanne and Verhetsel, Kilian and Remacle, Jean-Fran\c{C}ois},
title = {There are 174 subdivisions of the hexahedron into tetrahedra},
year = {2018},
issue_date = {December 2018},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {37},
number = {6},
issn = {0730-0301},
url = {https://doi.org/10.1145/3272127.3275037},
doi = {10.1145/3272127.3275037},
abstract = {This article answers an important theoretical question: How many different subdivisions of the hexahedron into tetrahedra are there? It is well known that the cube has five subdivisions into 6 tetrahedra and one subdivision into 5 tetrahedra. However, all hexahedra are not cubes and moving the vertex positions increases the number of subdivisions. Recent hexahedral dominant meshing methods try to take these configurations into account for combining tetrahedra into hexahedra, but fail to enumerate them all: they use only a set of 10 subdivisions among the 174 we found in this article.The enumeration of these 174 subdivisions of the hexahedron into tetrahedra is our combinatorial result. Each of the 174 subdivisions has between 5 and 15 tetrahedra and is actually a class of 2 to 48 equivalent instances which are identical up to vertex relabeling. We further show that exactly 171 of these subdivisions have a geometrical realization, i.e. there exist coordinates of the eight hexahedron vertices in a three-dimensional space such that the geometrical tetrahedral mesh is valid. We exhibit the tetrahedral meshes for these configurations and show in particular subdivisions of hexahedra with 15 tetrahedra that have a strictly positive Jacobian.},
journal = {ACM Trans. Graph.},
month = {dec},
articleno = {266},
numpages = {9},
keywords = {triangulation enumeration, tetrahedrization, oriented matroids, meshing, isomorphism, combination}
}
# 0 Abstract
本文回答了一个重要的理论问题：hexahedron分为tetrahedra有多少种不同的subdivisions？
众所周知，cube有5个subdivisions为6个tetrahedra，以及1个subdivison为5个tetrahedra
然而，并非所有hexahedra都是cube，移动顶点的未知会增加subdivisions的数量
最近的hexahedral dominant meshing方法尝试考虑将 tetrahedra合并为hexahedra 的这些configurations，当未能全部列举出来：它们只使用了我们在本文中发现的174个subdivisions中的10个

这174个将hexahedron分为tetrahedra的subdivisions的枚举就是我们的组合结果
174个subdivisions中的每一个都有5-15个tetrahedra，实际上是一个由2到48的equivalent实例组成的class，这些实例在顶点relabeling之前都是相同的

我们进一步证明，这些subdivisions中有171个具有geometrical realization
	即在三维空间中存在8个hexahedron的顶点的坐标，使得geometrical tetrahedral mesh是有效的
我们展示了这些configurations的tetrahedral meshes，并特别展示了hexahedra的subdivisions为15个tetrahedra，其具有严格正的Jacobian矩阵的
# 1 Introduction
于tetrahedral mesh相比，hexahedral meshes有着重要的数值性质：更快的assembly、solid machanics的高精度、或者不可压缩材料
然而，目前还没有一种鲁棒的hexahedral meshing技术能够处理一般的3D domains，以及生成unstructured hexahedral finite element meshes，仍然是mesh generation中最大的挑战

一种有前途的方法利用现有的鲁棒的高效的tetrahedral meshing算法
	如【Si 2015】通过将tetrahedra组合成hexahedra来生成hexahedral meshes
最近【Pellerin 2018】提出，大多数这些方法使用的hexahedron的10个subdivisons为tetrahedra的集合是不完整的

本文中，一个hexahedron是一个组合的3D cube，其square facets已经被三角化，且其geometry被定义为一个reference cube通过一个trilinear mapping $\varphi$ 的image，如Figure 1
我们的hexahedra并不是严格意义上的cube，并且它们只有在从reference cube的mapping是injective时是valid的
	validity很难去evaluate的，因为Jacobian行列式在hexahedron上不是恒定的
![[Pasted image 20240713170521.png]]

在这项工作中，我们考虑了将hexahedron为tetrahedra的elementary subdivisions
在本文的其余部分，我们使用triangulation的通用术语来自带d维simplicial subdivisons
	即tetrahedral或triangular meshes
hexahedron的faces被细分为两个triangles，正如我们在Section 2中看到的，没有hexahedron细分为tetrahedra的完整枚举是有效的，除了cube的特殊情况

我们的第一个贡献是解决了hexahedron triangulations的纯组合问题
我们的结果是，有174个hexahedron有着5-15个tetrahedra的triangulations同构，Table 1
![[Pasted image 20240713172045.png]]
根据对称性的不同，1174个triangulations实际上是一个2，4，8，12，16，24，48 equivalent实例组成的class，这项实例直到vertex relabeling都是相同的
我们通过回顾所有具有8个顶点的3-ball的组合triangulations来证明这个结果

我们的第二个贡献是证明了171个hexahedron triangulations在$\mathbb{R}^3$中具有geometrical realization
为了解决这个问题，我们实现了【Firsching 2017】中提出的方法
特别的，我们展示了具有14，15个tetrahedra的hexahedra的meshes，这与认为可能只有多达13个tetrahedra的subdivisions的信念相矛盾
这是将hexahedral meshes与tetrahedral meshes联系起来的重要理论结果
# 2 Related Work
# 3 Combinatorial Triangulations
hexahedron的triangulation的几何问题是困难的，我们首先处理这个问题的纯组合方面

一组tetrahedra称为是hexahedron $\{12345678\}$（Figure 6）的一个==valid==的组合的triangulation，需要满足：
	它是具有8个顶点的valid组合3-manifold
	并且它的边界包含8个顶点，12条边$\{12\}$，$\{14\}$，$\{15\}$，$\{23\}$,...,$\{78\}$，12个triangular面能配对成6个四边形$\{1234\}$,$\{1265\}$,...
![[Pasted image 20240713201003.png]]
### Theorem 3.1
hexahedron $\{12345678\}$有着174个不包含任何boundary tetrahedra的组合的triangulation，直到同构

Table 2列举了174种triangulations，在补充材料中提供了它们的完整描述
![[Pasted image 20240713202053.png]]
为了证明Theorem 3.1，我们首先使用有着9个顶点的3-sphere的triangulations的结果，来计算所有有着8个顶点的3-ball的triangulations
下一步，我们选择边界为triangulated hexahedron的边界的triangulations（Section 3.2）
最后将valid的组合划分为同构类（Section 3.3）
## 3.1 Triangulations of the 3-Ball
==2-sphere==，是一个三维ball的二维表面边界
==3-sphere==推广了这一点，并定义为一个四维ball的三维边界
2-sphere的triangulation和2-ball的triangulation有着直接的联系
	3-sphere的triangulation和3-ball的triangulation有着直接的联系
实际上，2-sphere的triangulation可以由2-ball的triangulation构建而成，如Figure 7
	cone是通过在polygon上添加point $v$，并且将其连接到5个boundary edges来构建的
	inverse transformation，即去掉sphere triangulation的一个点$v$，以及与其相关的三角形，就可以得到一个ball的triangulation（<font color=red>这里应该指的是将2-sphere（是三维ball的边界）的triangulation逆变换为2-ball（二维的ball，即平面圆）的triangulation</font>）（<font color=red>那以此类推，3-sphere（是四维ball的边界）的triangulation可以逆变换到3-ball（是三维ball，即球体）的triangulation</font>）
![[Pasted image 20240713203709.png]]

本文中，我们列举了所有hexahedron的triangulations，即有着8个顶点的三维ball
它们可以由具有9个顶点的3-sphere的triangulations得到
【...】列举了3-sphere的1296个triangulations
9个有着8个顶点的3-ball triangulations可以从每个1296 triangulations中通过移除一个顶点$v_i,i=1,...,9$，以及其邻接（所有邻接于$v_i$的tetrahedra）
## 3.2 Triangulations of the Hexahedron
下一步是测试所获得的3-ball的triangulations是否都是valid的hexahedron triangulations
	即它们的boundary是否匹配于hexahedron的triangulated boundary
hexahedron的边界boundary的triangulation有着8个顶点和18条边，其中边中有12条是固定的，剩下的6条四边形的对角线有两种可能的位置
	我们有$2^6=64$种可能的triangulations
	这些triangulations可以被分为7个equivalence classes，即有7个hexahedron boundary的triangulation同构，见Figure 8
![[Pasted image 20240714130733.png]]

## 3.3 Hexahedron Triangulation Comparison
最后一步是将得到的hexahedron的triangulation进行比较，以确定那些相同的同构
我们使用【Mashkat and Talmor 2000】中提出的graph formalism来表示和比较hexahedron triangulation

一个hexahedron的triangulation的decomposition graph（Figure 9）是一个edge-colored graph
	其中每个tetrahedron是一个顶点
	顶点之间有一个黑色的（plain）边，即对应的tetrahedra是邻接的
	顶点之间有一个灰色的（dashed）边，即对应的tetrahedra有着在同一个hexahedron facet的triangle faces
通过构造，每个顶点的degree是4，plain edges的simple chordless cycles（<font color=red>环里面的点与除了环连着的点，与其它点之间没有边</font>）对应于triangulation的interior edges
![[Pasted image 20240714132225.png]]

在没有boundary tetrahedra的triangulations和decomposition graphs之间有着一一对应（Figure 10）
假设固定triangulation的vertex labels使得$\{12345678\}$是一个hexahedron，则12条边是固定的，然后有一个选择将6个facet细分为两个三角形
	一旦选择了6条边界对角线，对hexahedron进行triangulation的剩余自由度由内部facets控制
decomposition graph的plain edge表示内部facets，6条dashed edges 表示对角边
![[Pasted image 20240714140618.png]]

==两个组合triangulations是同构的，当且仅当它们的decomposition graphs是同构的==
	即关于它们的顶点的relabeling是相同的
更正式地说，即graphs $G$和$H$的同构是$G$和$H$的顶点的bijection $f:V(G)\rightarrow V(H)$，使得$G$的任意两个顶点$u,v$是邻接的当且仅当$f(u),f(v)$在$H$中是邻接的

因此，在前一节中得到的hexahedron的triangulations的decomposition graphs被分类为同构类，依次获得我们的主要组合结果
# 4 Geometrical Triangulations
我们现在处理$\mathbb{R}^3$中hexahedron的triangulations问题的几何方面
确实不能保证174个组合triangulations中的每一个都存在valid geometric triangulation

更真实地说，我们正在解决一个几何实现问题，为每个组合triangulation $T$，即我们正在$\mathbb{R}^3$中寻找==8个顶点的位置==$\mathcal{A}$，使得由$T$定义的几何triangulation是valid的
一个triangulation是==valid==的，即满足：
1. tetrahedra的并集是points的凸包
2. tetrahedra之间没有不必要的相交
### Theorem 4.1
174个hexahedron的组合triangulations中有171个在$\mathbb{R}^3$有一个几何实现

Table 2列举了171个triangulations，并且在补充材料中提供了实现
没有实现的3个triangulations有15个tetrahedra，如Figure 14所示
![[Pasted image 20240714152059.png]]

我们的证明包含：
1. 求解可以实现一个给定的triangulation $T$的point configurations的潜在的组合问题
2. 在$\mathbb{R}^3$中找到对应于一个configuration的点坐标
主要步骤描述在Algorithm 1
![[Pasted image 20240714152440.png]]
## 4.1 Definitions
为了对实现给定组合triangulation的point set的组合属性进行编码，我们使用了oriented matroids
	这是离散数学的一个重要课题，这里我们将定义限制到所研究的问题
## 4.2 Combinatorial Formulation
我们首先回顾了实现一个组合triangulation $T$的point sets的oriented matroids必须遵守的5个约束条件
### Constraints
point set在一般位置，因此对应的oriented matroid应该：
1. 是均匀的，$\chi\ne 0$（没有4共面或5共面的points）
2. 是非循环的，这是所有point sets的oriented matroids所共有的一种性质
3. 是valid的，所有$r-2$子集应该满足Section 4.1定义的condition 2
几何的triangulation的tetrahedra是valid的，因此oriented matroid应该：
4. 定义valid的tetrahedra，$\forall \sigma \in T$,$\chi(\sigma_1,...,\sigma_{d+1})=+1$，我们假设它们是一致定向的，且在$T$中是正的
## 4.3 Geometric Realizations
一旦oriented matroids满足所有的constraints（与一个给定的组合triangulation相关）被计算，最后一步是找到8个顶点的实数坐标来实现这些oriented matroids中的一个
我们遵循【Firsching 2017】的方法，其提出了解决形式为$(n,d+1)$多项式不等式的系统：
![[Pasted image 20240714165236.png]]
	其中$\chi(1,2,...,d+1)$的可能的值都在Section 4.2中被计算
### Solving
### Result
所有171个admit一个oriented matroid的triangulations都有一个实现，所有的实现都在补充材料中随我们的实现一起提供，这就完成了Theorem 4.1的论证
Figure 12给出了具有8个tetrahedra的configurations的实现
![[Pasted image 20240714165717.png]]
## 4.4 Convex Geometric Realization
进一步研究了有着限制条件（8个point位置确保它们在convex位置）的实现问题
	即所有的顶点都在它们的凸包上（没有一个在其内部）
这是hexahedra在有限元计算中让hexahedra成为valid的一个合理条件
convexity条件可以直接地考虑，在通过排除所有（其中一个顶点位于其它四个顶点定义的tetrahedra的内部的）configurations来测试可接受的手性
### Result
在hexahedron的171个可实现的triangulations，13个不具有使其顶点处于convex位置的实现
# 5 Discussion
我们证明了有174种hexahedron的组合triangulation同构，以及它们中有171个有几何实现
## 5.1 Pattern occurrences in real meshes
hexahedron的triangulation的理论问题是更好地结合tetrahedral mesh元素构建hexahedron的重要一步
事实上，在实践中，当组合tetrahedral mesh的元素时，我们发现的许多附加patterns都会出现

为了识别input tetrahedral mesh中tetrahedra到hexahedra的组合，我们使用【Pellerin 2018】提出的方法
	该算法可证明地计算出连接性与hexahedron相匹配的八个顶点的可能组合
	其进一步保证了所识别的hexahedron在有限元计算中都是有效的
我们使用【Pellerin 2018】的可用实现来计算那篇文章给出的input tetrahedral meshes的所有valid hexahedra（Table 3），以及随机point set的Delaunay triangulations（Table 4）
我们将所有确定的hexahedra的decomposition graphs与171种可能的triangulations进行分类并比较
![[Pasted image 20240714193054.png]]
![[Pasted image 20240714193102.png]]
## 5.2 Future Work
使用我们的结果，我们将能够决定是否存在一个由8个顶点组成的triangulation，其边界与一个和hexahedron相匹配
我们也可以生成所有存在的triangulations
由于我们的论证原理是一般的，它可以被扩展到枚举任何具有四边形和三角形faces的polyhedron的triangulations，并且可以用来表征non-triangulable polyhedra，这是许多几何和组合问题的主要困难，在理论上所知甚少