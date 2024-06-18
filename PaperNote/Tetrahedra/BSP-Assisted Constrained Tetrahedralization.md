---
tags:
---
> @inproceedings{joshi2003bsp,
  title={BSP-Assisted Constrained Tetrahedralization.},
  author={Joshi, Bhautik J and Ourselin, S{\'e}bastien},
  booktitle={IMR},
  pages={251--260},
  year={2003}
}
# 0 Abstract
本文通过将non-convex polyhedra分解为concex subpolyhedra，将这些convex subpolyhedra tetrahedralizing并merging到一起
我们从一个polyhedron的triangular faces生成一个==Binary Space Partition（BSP）tree==，并用它来识别polyhedron中的convex subpolyhedra
每个convex subpolyhedron都是独立地tetrahedralized
使用原始的merging过程，这些subpolyhedra之间的边界都被连接且tetrahedralized，确保在此merging过程中不会在原始polyhedron之外创建tetrahedra
# 1 Introduction
3D中的closed polyhedra可以用一个或多个non-self-intersecting的closed boundaries来描述
这些boundaries通常用三角面来构造
这些polyhedra有着一个interior和exterior，并且有着一个有限的volume
它们的tetrahedralization需要寻找一个完全填满polyhedra的tetrahedra的集合，且它们完全在boundaries上或内部

通常，这种tetrahedralization可以用Delaunay tetrahedralization算法来构建
给定polyhedron中的一组顶点，Delaunay tetrahedralization算法构建一个有着这些顶点的tetrahedra的convex集
	没有tetrahedra彼此相交，且mesh中的edges或faces之间的最小角最大

然而，Delaunay triangulation或tetrahedralization算法仅生成convex meshes
这意味着它可能无法recover polyhedron的边界，因为局部的non-convex区域（如dents和holes），往往倾向于被网格覆盖和密封
解决这个问题的一般方法包括：首先生成整个domain的Delaunay tetrahedralization，并去除位于该domain之外的tetrahedra
	然而，这并不能保证所有的boundary faces都被recovered

为了部分解决这个问题，non-convex polyhedron的tetrahedralization通常涉及在上面或内部放置extra points
	这些extra point通常被称为Steiner points，使得算法能够完全覆盖domain
	代价是generated mesh的额外复杂度
2D中，提出了几种用最小点插入保持边界的可靠方法
然而，在3D中，如何创建一个保持边界并具有可接受额外复杂度的tetrahedralization算法仍然是一个具有挑战性的问题

在过去的十年中，出现了两种主要的preserve boundaries in 3D meshing：
	conforming和constrained tetrahedralizations
下面我们描述了两种主要的技术，然后简要回顾了文献中可用的一些其它方法

==conforming Delaunay tetrahedralization==是对input polyhedron的strict Delaunay tetrahedralization
必要时，在mesh中插入Steiner point，以确保保留input mesh的所有边界，同时严格遵循Delaunay的empty sphere准则
2D中，对于$n$个输入顶点，建立用于执行conforming triangulation的Steiner points明确上限为$O(n^3)$
然而，conforming tetrahedralization可能在插入Steiner point会付出更高的代价

==constrained Delaunay tetrahedralization==不是strict Delaunay的
为了帮助boundary recovery，empty sphere定理被放松为允许locally Delaunay的elements
elements在boundary edge或face上被允许创建，即使它们违反empty sphere定理（Figure 1）
constrained tetrahedralization对input meshes的限制较少
虽然它们不是strict Delaunay tetrahedralization，但它们仍然有助于生成高质量的mesh，并且比conforming算法需要插入的点更少
![[Pasted image 20240617192757.png]]
[[Tetrahedral Mesh Generation by Delaunay Refinement]]提出了constrained Delaunay tetrahedralizations的一种扩展，利用Delaunay refinement方法实现了non-convex polyhedra的tetrahedralize
然而，它被限制在面之间构成的角度大于90°的meshes工作

Murphy等人...
但它添加到mesh的Steiner point的数量太大，不实用
它们证明了使polyhedron tetrahedralize所需的Steiner point有一个上界（有待确定）

Cohen-Steiner等人...
实践中，添加的顶点数与输入顶点数的比例在3:1和10:1之间变化
然而，该技术并没有说明这个ratio的任何bounds

另一种方法...
然而，它并没有表明是否能够处理non-convex polyhedra

通过将polyhedron划分为subpolyhedra，简化了tetrahedralizing问题，因为它允许将每个subpolyhedron单独tetrahedralizing，并将结果合并到一起
已经被证明了将一个polyhedron划分为没有Steiner point的convex subpolyhedra的任务是NP完全的
然而，有好的polygon划分算法存在，需要更多的Steiner point插入

polyhedral分解的常用方法是使用pre-defined grid（通常由正交平面组成），将polyhedron划分为有限大小的cell
然而，这些技术可能会受到polyhedron的局部geometry的限制，特别是当complex features小于cell尺寸时

为了解决这个问题并利用grid-based mesh generation的简单性，我们提出使用BSP tree将input mesh分解为convex regions
每个regions都是单独地tetrahedralized，然后与其它tetrahedra合并在一起，来重建原始polyhedron
该技术可以有效地处理non-convex polyhedra
并且与其它grid-based技术不同，它不受grid resolution的限制，因为grid conforms于原始polyhedron本身的faces

本文提出的算法作为高质量mesh generation的初始化
一旦它创建了polyhedron的有效covering tetrahedralization，generated tetrahedra可以很容易地被subdivided，而不需要edge filps，从而保证了input boundary topology的保留

该算法还可以对2D non-convex进行triangulate，然而本文的重点是它的3D应用
Section 2详细描述了我们提出的算法
Section 3给出了使用non-convex的初步结果
Section 4我们讨论了当前技术的优势和局限性，最后提出了未来研究的几个路径
# 2 Methodology
给定一个2D或3D的input polygon，可以创建它的BSP tree
利用该tree，在polyhedron内建立convex subpolyhedra
这些convex subpolyhedra的每个的点集都被identified，并使用一个incremental Delaunay算法tetrahedralizaed
使用BSP tree，使用相同的incremental Delaunay算法将这些convex sites都glued到一起
BSP tree可以递归地遍历，以有效地执行gluing过程，一次仅两个subpolyhedra
要完成tetrahedralization，还需要另一个步骤来correct彼此not coincident的tetrahedra

BSP辅助tetrahedralization算法流程见Figure 2
为了清楚，convex decomposition和convex subpolyhedra的tetrahedralization被显示为一个单独的步骤，且subpolyhedra的gluing和fixing也是如此
实际上，这四个操作都可以作为一个递归算法的一部分来执行
![[Pasted image 20240617201008.png]]
## 2.1 Generating BSP Trees
BSP tree可以被视为最通用的空间细分技术，适用于2D、3D和更高维度...

利用BSP tree来寻找polyhedra的convex subpolyhedra是一个BSP tree众所周知的性质
然而，由于BSP tree分割面和插入点，它并不是一个polygon的strict convex decomposition
...

其它空间细分技术（如kd-tree或octrees）不被使用，因为在具有高曲率边界的polyhedra中，可能需要对初始mesh进行高度细分，从而产生大量的附加点
此外，正如在介绍中提到的，与其它空间细分技术不同，这种分解不受grid resolution的限制，因为grid是conforming于原始polyhedron本身的faces
## 2.2 BSP Trees for Convex Decomposition
在BSP tree的root的polygonal将空间分两个subpolyhedra
当在前面或后面穿越BSP tree时，这些subpolyhedra再次被leaf nodes的faces划分
这些leaf nodes可以作为进一步划分subpolyhedra的sub-tree的root，这些sub-trees可以继续作为leaves来划分这些subpolyhedra，以此类推
实际上，polyhedron中的convex subpolyhedra是这些subpolyhedra的groups的交集，且可以使用BSP tree的structured traversal来识别它们

为了阐述BSP tree进行convex decomposition的过程，Figure 3给出了一个2D的实例
Figure 3a中的polygon有5个faces，每个faces都有一个orientation，有箭头的一面表示front，没有箭头的表示behind
这些faces一次一个地被添加到BSP tree中，如Figure 3b所示
在tree中，相对于一个face，右边的leaf是front halfspace，左边的leaf是behind halfspace
如果一个face所在plane与一个leaf相交，那么这个leaf face被分割了
	因此face 2将face 5分为了5f和5b
在遍历tree时，会保留一个tuples列表，表示遍历的face以及相对于root face采取的direction
	如果遇到一个空的front face，则该list被存储为一个convex subpolyhedron
convex subpolyhedron $A$由2、1、5f的front halfspace的交集表示，$B$由3、4、5b的front halfspace和2的behind构成
![[Pasted image 20240617202856.png]]
### 2.2.1 Creation of BSP points
BSP tree的convex decomposition算法所识别的的subpolyhedra的implicit顶点并不能保证在原始polyhedron都有对应的顶点
然而，在BSP tree的创建过程中，位于parent node平面上的face被分割
这个分裂过程在被tetrahedralized的boundary上引入了新的点
这些BSP点有助于确保执行完整的tetrahedralization
然而，目前尚不清楚BSP点和Steiner point之间是否存在关系
## 2.3 Tetrahedralization of subpolyhedra
一旦确定了一个convex subpolyhedron，就可以确定其内部的点的list
	本文中将恰好位于subpolyhedron边界上的点归为subpolyhedra的内点
这些点可以tetrahedralized，形成一个convex subpolyhedron的mesh
采用randomized point insertion Delaunay tetrahedralization算法
然而，不能保证这组点完全描述convex subpolyhedron的边界
	当BSP tree的node与child node的plane相交，但不与child node本身相交时，通常会发生这种情况

Figure 4（<font color=red>该Figure中A和B位置可能需要对调</font>）展示了不能完全描述的subpolyhedra的情况的2D示例
Figure 4a中的polygon和一个可能的对应BSP tree（Figure 4b），可以分割成两个convex subpolyhedra $A$和$B$
虽然child node 3的plane与parent node 1相交，其face本身并未与parent node相交
这意味着对于convex subpolyhedra，在plane 1和3的交点没有顶点，因此subpolyhedra没有在subpolyhedra的每个corner上的顶点
Figure 4c说明了subpolyhedra的triangulation不能recover整个polygon
![[Pasted image 20240617212322.png]]
如果不加以纠正，这个问题将导致在tetrahedralization中丢失tetrehedra
为了recover这些missing tetrahedra，一个glue算法被设计于用来merge subpolyhedra到一起
### 2.3.1 Pseudocode Implementation
tetrahedralization算法通过遍历BSP tree来操作
一个tuples的list表示遍历的node和所去的direction，识别convex subpolyhedra是必要的
	例如：`(node_n,[front, behind])`
一旦访问了一个node，它就会从list中被移除
Algorithm 1描述了这种伪码实现
![[Pasted image 20240617213252.png]]
## 2.4 Gluing subpolyhedra
当使用glue算法合并两个subpolyhedra时，它们被一个single splitting plane精确地分开，该plane是BSP tree当前位置的root node
因此，该算法每次递归地合并两个growing subpolyhedra，直到tree中的所有subpolyhedra都被合并
需要生成tetrahedra来填充merging subpolyhedra之间的空间

生成subpolyhedra之间的tetrahedra的一种方法是：合并front和behind subsets的points，并在这个合并后的点集执行一个Delaunay tetrahedralization
	然后可以使用简单的cross test来reject没有合并tetrahedralization的tetrahedra
	这个合并的tetrahedra集之后添加到已经生成的front和behind的subpolyhedra的tetrahedra集合中

对于一个通过cross test的tetrahedra，它必须满足三个标准
1. 它的所有顶点必须不完全位于joining plane的上方或下方
2. tetrahedra至少有一条edge必须与joining plane相交
3. tetrahedra的任何edge不能与joining plane上的现有face相交

一个2D的例子见Figure 5
在此合并中，joining plane用虚线表示，且从edge $cf$生成
使用cross test
	triangle $\triangle cde$是合法的，因为其顶点都位于joining plane的上方和下方，一条边穿过joining plane，没有edge与$cf$相交
	triangle $\triangle afb$是不合法的，因为其所有点都在joining plane上方
	triangle $\triangle abg$是不合法的，因为其有edges穿过plane $cf$
![[Pasted image 20240618143722.png]]
不幸的是，上述算法并非适用于所有情况，有时无法完全将subpolyhedra之间的区域网格化
为了解决这个问题，我们提出了一种受Bowyer-Watson随机点插入（RPI）算法启发的glue算法
它的工作原理是：连接front subset的边界的triangles，to the points of the hull below that face those triangles

...

Figure 6显示了这种类型的点插入的2D示例
外接圆包含point的triangles被删除，并与新point进行re-meshed（Figure 6a）
一旦这是完成的，hull的edges被用来创建新triangles（与点如果完全在triangulation和外接圆外是一样的）（Figure 6b）
使用gluing算法，如Figure 6c，如果没有删除tetrahedra，而是从hull的edges和exterior point创建triangles
![[Pasted image 20240618145337.png]]
## 2.5 Fixing Crossed Tetrahedra Edges
# 3 Proof of Concept
# 4 Discussion
## 4.1 Theoretical Bounds
## 4.2 BSP and Steiner Points
## 4.3 Practical Considerations
从相同的input mesh可以构造许多不同的BSP tree
为了使生成的BSP tree更加一致，决定在BSP tree构建中选择root node的选择标准是必要的
使用的度量是：如果选择了特定的root，其分裂的leaf node数
实践中，当BSP root node的选择使分裂的leaf node数量最小化时，mesh往往具有更少的附加点和tetrahedra

mesh generator不能保证任何生成的tetrahedra的质量
然而，mesh generation过程不受到local geometrical feature的尺寸限制，因此是scale independent

...
## 4.4 Limitations
该算法最薄弱的部分是gluing算法，需要进一步改进
主要问题在于，生成的subpolyhedra之间的tetrahedra是non-Delaunay的，并且这些tetrahedra的可靠性质和属性尚未确定

与此相关的问题是，在gluing过程中找到上述subpolyhedra中的面集，其面向下方subpolyhedra
...
# 5 Conclusion and Future Directions
本文提出了一种covering tetrahedralization算法，为高质量的mesh generator提供了有效的初始化
因此，这些tetrahedra可以在没有edge flips的情况下进行subdivided和refined，从而保证了input polyhedron boundary的拓扑结构

主要的创新之处在于将BSP tree和Delaunay tetrahedralization结合在一起，以及一个不受local complexity限制的tetrahedral mesh generator

根据实验，added points的数量一直很低