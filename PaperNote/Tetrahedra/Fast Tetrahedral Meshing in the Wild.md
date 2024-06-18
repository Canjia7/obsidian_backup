---
tags:
---
![[Pasted image 20240616160039.png]]
> @article{10.1145/3386569.3392385,
author = {Hu, Yixin and Schneider, Teseo and Wang, Bolun and Zorin, Denis and Panozzo, Daniele},
title = {Fast tetrahedral meshing in the wild},
year = {2020},
issue_date = {August 2020},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {39},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/3386569.3392385},
doi = {10.1145/3386569.3392385},
abstract = {We propose a new tetrahedral meshing method, fTetWild, to convert triangle soups into high-quality tetrahedral meshes. Our method builds on the TetWild algorithm, replacing the rational triangle insertion with a new incremental approach to construct and optimize the output mesh, interleaving triangle insertion and mesh optimization. Our approach makes it possible to maintain a valid floating-point tetrahedral mesh at all algorithmic stages, eliminating the need for costly constructions with rational numbers used by TetWild, while maintaining full robustness and similar output quality. This allows us to improve on TetWild in two ways. First, our algorithm is significantly faster, with running time comparable to less robust Delaunay-based tetrahedralization algorithms. Second, our algorithm is guaranteed to produce a valid tetrahedral mesh with floating-point vertex coordinates, while TetWild produces a valid mesh with rational coordinates which is not guaranteed to be valid after floating-point conversion. As a trade-off, our algorithm no longer guarantees that all input triangles are present in the output mesh, but in practice, as confirmed by our tests on the Thingi10k dataset, the algorithm always succeeds in inserting all input triangles.},
journal = {ACM Trans. Graph.},
month = {aug},
articleno = {117},
numpages = {18},
keywords = {mesh generation, robust geometry processing, tetrahedral meshing}
}
# 0 Abstract
我们提出了一种新的tetrahedral meshing方法，fTetWild，将triangle soups转换成高质量的tetrahedral meshes
我们的方法建立在TetWild（[[Tetrahedral Meshing in the Wild]]）算法的基础上，用一种新的incremental方法来构建和优化output mesh，将triangle insertion和mesh optimization交织在一起
我们的方法使得在所有的算法阶段保持valid floating-point tetrahedral mesh成为可能，消除了TetWild使用的昂贵的有理数构造的需要，同时保持完整的robust和类似的output quality

这让我们可以从两个方面改进TetWild：
1. 我们的算法明显更快，关于运行时间，相比于不太robust的Delaunay-based tetrahedralization算法
2. 我们的算法保证生成有着floating-point顶点坐标的valid tetrahedral mesh，而TetWild算法生成valid的有理坐标tetrahedral mesh，但不保证floating point转换后的valid

作为一种权衡，我们的算法不再保证所有input triangle都存在于output mesh中
	但在实践中，正如我们在Thingi10K数据集上测试所验证的那样，算法总是成功地inserting all input triangles
# 1 Introduction
Tetrahedral meshes通常用于图形和工程应用
Tetrahedral meshing算法通常以3D surface triangle mesh作为input，output一个tetrahedral mesh填充着以input mesh为边界的volume
传统的tetrahedral meshing算法对input有着很强的假设，要求其是一个closed manifold、不存在self-intersections、不存在numerical unstably close elements等等
然而，这些假设通常不适用于不完美的3D geometry data in the wild

最近提出的TetWild算法使得对triangle soup的reliably tetrahedralize成为可能，通过结合精确有理计算和一个geometric tolerance，来自动解决self-intersections、gaps、其它input中的imperfections
该算法没有对input mesh施加正式的假设，并且非常鲁棒

然而，TetWild有两个缺点，一个是理论上的，一个是实践上的
理论上的缺点是：它不能保证生成floating point tetrahedral mesh，该算法内部使用有理数，然后再mesh optimization过程中转换为floating point
	虽然不太可能，但mesh optimization阶段可能无法将输出mesh的所有坐标四舍五入为floating point
实践上，与基于Delaunay-based tetrahedralization算法相比，实际的缺点是运行时间长

我们引入fTetWilid，TetWild算法的一个变体，解决了这两个限制，同时保持了TetWild的重要属性：
* 对不完美input的robustness
* 批量处理大量模型而无需调整参数
* 同时生成高质量的tetrahedral meshes
与TetWild一次性插入所有triangles生成polyhedral有理mesh不同，我们从floating point tetrahedral mesh开始，每次插入一个input triangle，并局部地re-tetrahedralize，拒绝参数inverted或degenerate elements的操作
然后我们迭代地改进mesh的质量，并尝试将被rejected triangles插入到更高质量的mesh中，这样就不太可能失败

我们的算法总是保证生成一个valid tetrahedral mesh，带着floating point顶点位置，独立于stopping criteria或mesh的质量
==它可能无法插入一些input triangles，从而导致不太准确的boundary preservation，但我们从未在实验中观察到这种行为==
新算法可以使用floating point结果实现，避免了与有理数相关的开销
floating point数的使用也简化了parallelization，我们在mesh optimization过程中使用它来进一步改善大型模型的运行时间
因此我们的新算法比TetWild快得多，运行时间和Delaunay-based的算法相当（Figure 1），同时提供更强的保证，总是同时产生valid floating point output
![[Pasted image 20240616160039.png]]
这些改进使得fTetWild不仅在volumetric meshing问题上比TetWild更实用，而且在mesh repair和approximate mesh arrangements也更实用
...

我们通过在Thingi10K数据集上计算tetrahedral meshes和计算approximate Booleans来证明我们的算法的robustness和practical
我们使用生成的tetrahedral meshes来求解复杂几何域上的弹性、流体流动和热扩散方程
# 2 Related Work
## 2.1 Tetrahedral Meshing
### Delaunay Meshing
目前研究最多，应用最广泛的tetrahedral meshes是基于Delaunay condition的【】
这些方法是有效的，在商业软件中得到了广泛的应用
它们既可以应用于point cloud input，也可以应用于没有degenerate faces的non self-intersecting meshes的内部tessellate，但它们不是为处理不完美input而设计的
	因此，这些技术在data in the wlid上都失败了
### Grid Methods
### Front-Advancing Methods
### Envelope Meshing
### Mesh Improvement
## 2.2 Applications
### Mesh Repair
### Booleans and Mesh Arrangements
# 3 Method
fTetWild将一个3D triangle soup视为input（即一组任意连接的，可能相交的triangles，顶点可能重复），其顶点以floating-point坐标表示，表示一个object的surface
算法有两个用户定义的参数：target edge length $l$，和envelop size $\epsilon$
	$\epsilon$-envelope表示用户期望接受的的从input surface的最大偏差，例如在增材制造中，可以表示加工精度

输出一个包含input的轴对齐bounding box的volumetric tetrahedral mesh，具有floating-point顶点坐标，其elements是：
1. non-verted（即positive volume）
2. 在用户定义的$\epsilon$-envelope内有一些faces近似于input soup

fTetWild对input triangle soup没有任何假设，并且在处理self-intersection或small gaps时robust
这种robustness通过允许对应于input surface的tetrahedral mesh的面移动到一个$\epsilon$-envelope的内部来实现的：
	当与适当的mesh improvement操作相结合时，self-intersections、degenerate、near-degenerate faces和gaps包含在envelope内都会被自动removed（Figure 2）
![[Pasted image 20240616214043.png]]

output tetrahedral mesh可以选择性地post-processed来去除input surface外部的tetrahedra（Section 3.6）
我们注意到这个可选阶段依赖于input triangles（的geometry或orientation）来得到一个valid volume
这种启发式的filtering可能会失败
	例如，如果input是远离于一个closed surface（eg. 半球体），fTedWild将生成一个valid tetrahedral mesh，其face符合input，但filtering阶段可能会丢弃所有tetrahedra，因为“outside” region不是well defined的
### Similarities and Defferences to Existing Face Insertion Algorithms
现有的tetraheral meshing算法面临的主要挑战是：对input faces的保持（精确或近似）

最著名的算法是[[‘Ultimate’ robustness in meshing an arbitrary polyhedron]]，该算法通过与input faces相交来细分background mesh
	然而，这个过程可能由于floating-point舍入而引入inverted elements
	这是一项困难的任务，目前还没有robust的算法
TetWild（[[Tetrahedral Meshing in the Wild]]）提出了一个robust的解决方案，该解决方案最初使用有理数精确插入faces来避免数值问题，但随后强制允许它们移动，来将有理坐标舍入到floating point
	尽管该解决方案robust且conservative，但它依赖于昂贵的有理结构，并且不能保证在舍入阶段成功

我们的方法遵循TetWild类似的方法，允许从input surface参数small且controlled的偏差，但避免了构建rational mesh的必要，同时继承了TetWild的robust
算法上，有三个主要的区别：
1. fTetWild通过在现有的background tetrahedral mesh中每次插入一个input triangle来preserve the input faces。为了方便插入，它通过一个snapping tolerance（相对于floating point machine precision，仅与$\epsilon$-envelope有关）来放松插入
2. fTedWild总是通过查找pre-computed table来tetrahedralizes受到新插入face影响的区域，并始终保持一个valid inversion-free tetrahedral mesh（使用exact predicates）
3. fTedWild仅使用floating point来表示顶点，从而减少了运行时间和内存消耗

我们注意到，由于floating-point表示的限制，插入triangle可能会失败
	例如，插入的face可以任意靠近现有的一个顶点，插入将引入一个体积为零的tetrahedron
在这种情况下，我们rollback有问题的操作，将有问题的faces标记为un-inserted，迭代地对整个mesh执行mesh improvement操作，当mesh质量增加时，尝试再次插入该face
==这个过程显示了fTetWild唯一可能的失败：添加一些input faces的不可能==
虽然这确实是可能的，但它从未在我们的实验中得到证实
注意，即使某些face不能插入，fTetWild仍然输出一个符合所有其它faces的valid mesh
## 3.1 Algorithm Overview
我们的算法由四个phase组成（Figure 3）：
1. 简化了input triangle soup，同时确保它保持在input的$\epsilon$-envelope中（Section 3.3）
2. 生成一个background mesh，迭代地插入triangles，如果插入没有引入inverted elements（Section 3.4）
3. 使用局部操作来改进mesh（Secion 3.5），并在每三次改进迭代结束时，我们re-attempt在phase 2中无法插入的三角形
4. 可选地filter mesh elements以去除surface外的element，或执行boolean操作（Section 3.6）
![[Pasted image 20240617152158.png]]
在整个过程中，我们确保tetrahedral mesh保持valid，也即我们确保：
1. 每个element都有positive volume（使用exact predicates）进行检查
2. 所有成功插入的triangles，从现在起称为tracked surface，留在input的$\epsilon$-envelope内

在整个算法中，如果两点之间的距离低于一个数值tolerance $\epsilon_{zero}$，我们将其视为0
类似的，我们用$\epsilon_{zero}^2$和$\epsilon_{zero}^3$于面积和体积
我们发现，算法的性能不会受到这个tolerance的影响，只要$\epsilon_{zero}>10^{-20}$
	实验中，设置$\epsilon_{zero}>10^{-8}$
## 3.2 Envelope
我们使用envelope定义和[[Tetrahedral Meshing in the Wild]]中介绍的算法来构建envelope，并检查其内部是否包含triangles
特别是，检测一个triangle是否包含在envelope内，是通过对input triangle进行采样并检查samples是否都在较小的envelope（with the sampling error conservatively compensated）内进行
## 3.3 Preprocessing
我们使用与[[Tetrahedral Meshing in the Wild]]相同的preprocessing过程来简化input
我们merge顶点近于$\epsilon_{zero}$并collapse一条边（通过merge一个端点到另一个），如果：
1. 它是一个manifold edge（不超过两个incident triangles），并且vertex-adjacent edges也是manifold的
2. collapsing操作不会移动triangles到一个smaller envelope（尺寸$\epsilon_{prep}<\epsilon$）的外部
这个阶段选择$\epsilon_{prep}=0.8\epsilon$，这个值为triangle insertion（Section 3.4）的snapping提供了空间，并防止顶点过于靠近envelope的boundary，从而为surface vertives在mesh improvement（Section 3.5）中移动留下了更多的自由
数据集中，我们观察到，在0.7-0.999的范围内，改变这个参数对运行时间的影响很小，对output的影响可以忽略不记
注意，它不可以选择为1，因为它会防止snapping（Section 3.4）

由于envelope包含检测，preprocessing在计算上是昂贵的，因此我们提出一种基本的parallelization策略，使用8核时可以加速preprocessing 4倍
我们的parallel edge collapsing过程使用一个在所有input mesh上连续的2-coloring pass
	初始阶段将所有input triangles标记为白色
	然后迭代地将所有edge标记为parallel-independent，如果所有其vertex-adjacent triangles都是白色的。然后将这些triangles标记为黑色（Figure 4）
此时，我们可以安全地平行地collapse所有标记的parallel-independent edge
我们迭代这个过程，直到我们无法删除超过0.01%的原始input顶点
![[Pasted image 20240617154914.png]]
## 3.4 Incremental Triangle Insertion
### 3.4.1 Background Mesh Generation
triangle insertion阶段需要一个background mesh（不一定conform于input triangles），我们在preprocessing阶段的顶点上使用Delaunay tetrahedralization（Geogram）来创建background mesh

因为我们允许surface在一个$\epsilon$-envelope内移动，所以我们为一个比input顶点的bounding box大$2\epsilon$的bounding box生成background mesh
与TetWild类似，在Delaunay tetrahedralization之前，在box内均匀地添加additional points，并且至少远离input faces $\epsilon$，以获得更均匀形状的初始elements
### 3.4.2 Single Triangle Insertion
算法的关键部分是three-stage过程
	将一个triangle $T$插入到一个valid tetrahedral mesh $M$
	添加新顶点和tetrahedra
	调整mesh connectivity
以尽可能减少插入的失败 和 插入产生的badly shaped tetrahedra

注意，我们不插入degenerate triangles
我们的算法使用marching tetrahedra【】和其它tetrahedralization methods的思想，以及marching cubes
==它包含以下步骤：==
1. 寻找triangle $T$ cut的tetrahedra $M$ 组成的集合$\mathcal{T}_I$，定义见下
2. 计算由$T$ span成的plane 和 $\mathcal{T}_I$的tetrahedra的edges的交点，记为$P$
3. 使用pre-computing的tet-subdivision table中的connectivity pattern细分所有的cut tetrahedra，这些pattern保证了valid的tetrahedral mesh connectivity是manifold的
#### Finding Cut Tetrahedra
我们首先定义object $A$ ==cuts though== object $B$，如果它们的intersection同时包含$A$和$B$的interior points
我们称triangle $T$ ==cuts== tetrahedron $\mathcal{T}$，如果：
1. 它完全包含在$\mathcal{T}$中，或
2. 它至少穿过$\mathcal{T}$的一个face（Figure 6）
![[Pasted image 20240617172633.png]]
我们初始化$\mathcal{T}_I$为$T$ cuts的$M$中的tetrahedra的集合
注意，这个集合会被算法迭代地expanded

我们用exact predicates来检查三角形是否包含在tetrahedron中
为了检测一个triangle是否穿过另一个，我们将exact predicates和算法结合起来【】
predicates的使用确保了使用floating-point坐标时拓扑的正确性
#### Plane-Tetrahedra Intersection
将定义plane $P$ 的triangle $T$插入到$\mathcal{T}_I$（Figure 5c、d）
我们需要确保插入后：
1. 对于每一个点$p\in T$，在集合$\mathcal{T}_I$中存在一个面$F$，使得$min_{q\in F}\Vert p-q\Vert<\delta$
2. 面$F$在$\mathcal{T}_I$中的投影，在从$T$到plane $P$距离$\delta$的$T$的covers内
我们称具有这些性质的集合为$T$的covers
我们允许triangles的cover偏离$P$，这对于使用floating point计算robustly插入triangles是至关重要的
	如果没有，我们观察到由于更多的插入失败，运行时间明显更长，这会导致额外的mesh optimization迭代
此外，在最终output中仍有更多未插入的faces
对于第一次triangle插入pass（即，在进行任何mesh improvement之前），我们使用$\delta=max(\epsilon_{zero},10^{-3}\epsilon)$，而对于所有后续passes，我们减少到$\delta=\epsilon_{zero}$
![[Pasted image 20240618191926.png]]
我们从infinite-precision算术的理想情况开始
在这种情况下，我们可以很容易地通过将$\mathcal{T}_I$中tetrahedra的所有faces与plane $P$相交来实现对$T$的covering
这将在$P$上生成一个covering $T$的planar polygonal mesh $\mathcal{F}$，且从$T$到$\mathcal{F}$的最大距离是0（Figure 7a）
	$\mathcal{F}$的顶点是$P$和$\mathcal{T}_I$的交点
![[Pasted image 20240618193448.png]]
当使用floating point表示来表示顶点坐标时，舍入误差可能导致degenerate或inverted tetrahedra
我们的方法拒绝是拒绝插入这些情况
然而，为了最小化rejected的triangle数量，我们要么将tetrahedral mesh $M$的顶点snap到$P$（Figure 7b），要么将交叉点snap到$M$的顶点（Figure 7c，d）...
#### Table-based Tetrahedron Subdivision
与$\mathcal{T}_I$中tetrahedra（被$P$切割的）共享边的所有的tetrahedra，都被细分为sub-tetrahedra，根据tet-subdivision table
注意，这组tetrahedra通常包括一些来自$\mathcal{T}_I$的tetrahedra（Figure 5e中红色的element），和一些邻近的$\mathcal{T}_I$的tetrahedra（Figure 5e中黄色的element）

由于一条edge最多只能有一个与$P$相交的点，subdivided tetrahedron的分解在很大程度上（但不完全）取决于哪些edge被切割
	如果一条edge与$P$相交，并且两个端点都没有被snapped，则该edge被切割
我们在一个subdivision table中记录了一个tetrahedra的所有可能的分解，以primary cut indices和secondary cut indices为索引
primary index (I)（Table 1）是一个二进制字符串，表示哪些边被切割
secondary index (II) （Table 1）是特定分解的编号表
	如果一个face上的两条edges被切割（一个face的三条edge不可能被同时切割），则该face有两种可能的triangulations，也有多个tetrahedron分解
primary index配对着secondary index表示了一个tetrahedron的唯一分解
![[Pasted image 20240618201737.png]]
对于一个oriented tetrahedron $\mathcal{T}$，有edge上有$2^6=64$种可能的交点组合，但并非所有64种组合都能实际实现
直接枚举表明，下列edge-cut配置是不可能的：
1. $\mathcal{T}$有6条cut edge
2. $\mathcal{T}$有5条cut edge
3. $\mathcal{T}$有4条cut edge，且其中的3条在同一个面上
4. $\mathcal{T}$有3条cut edge在同一个面上
总共，有23种不可能的edge-cut配置

剩下的41种可实现的edge-cut配置覆盖了所有的细分情况，我们可以将它们分成7个对称类（Figure 9）
其中的5种已经被讨论了【】，且用于一个plane切割一个tetrahedron
我们需要2个额外的配置（Figure 9(4)(6)）来细分相邻的$\mathcal{T}_I$的tetrahedra，只有$P$切割的某些edge（Figure 5e的黄色element）
![[Pasted image 20240618202616.png]]
我们检索对应于primary index的对$\mathcal{T}$的分解的list，我们现在需要选择一个secondary index，对应于一个分解，该分解保持了mesh $M$在分解后的validity，即我们想要$M$有着valid拓扑且没有inverted tetrahedra
为了确保一个valid拓扑，两个邻接的subdivided tetrahedra必须在共享的face上有着相同的triangulation
我们设置了一个规则来选择这样的triangulation，只使用$M$的顶点的global ordering
	对于tetrahedra $\mathcal{T}$的一个face $[v_0,v_1,v_2]$，其上有着两个交点$p_1,p_2$（见插图）
	如果顶点$v_1$的唯一整数label大于$v_2$的，则选择包含$[p_2,v_1]$的triangulation
	否则选取包含$[p_1,v_2]$的triangulation
这个简单的规则完全确定了secondary index，并保留了mesh的拓扑...
![[Pasted image 20240618211809.png]]
然后我们检查是否所有的sub-tetrahedra的体积都大于$\epsilon_{zero}^3$，如果不是这种情况，则拒绝插入
	因为我们观察到，体积为正但极小的element可能会在局部区域延迟插入
### 3.4.3 Open-boundary Edge Preservation
## 3.5 Mesh Improvement
我们使用TetWild中提出的improvement框架，其优化conformal AMIPS 3D energy来提高mesh quality，但通过将框架专门优化于floating point计算，可以避免hybrid kernel带来的开销
注意，我们使用AMIPS energy，因为它是可微的和scale-invariant，我们对原始的优化做了三个改变：
1. 我们尝试每三次mesh improvement迭代就插入未插入的input face，直到所有input face都被插入，或mesh improvement终止
2. 我们使用简单的graph coloring策略并行化vertex smooth步骤
3. 我们发现AMIPS energy计算在floating point的评估存在不稳定性，这有时会导致TetWild中的overrefinement，我们提出了一种使用有理数计算中间结果的混合评估方法
（1）是新算法需要的改变，因为不是所有的face都可以在floating point中进行计算时插入
（2）是一个较小但有效的修改，它略微提高了性能
（3）是一个微妙的问题，我们现在更详细地解释
	
## 3.6 Filtering
# 4 Results
## 4.1 Success Rate
## 4.2 Running Time
## 4.3 Mesh Quality
我们的算法生成的mesh的geometric quality与TetWild生成的mesh相似（Figure 18），因为实现了类似的mesh optimization策略
![[Pasted image 20240618162912.png]]
我们使用五种不同的方法定量地评估和比较
1. AMIPS energy
2. Minimal dihedral angle
3. Volume-to-edge ratio
4. Aspect ratio
5. Radius-to-edge ratio
## 4.4 Mesh Density
与TetWild相比，我们的方法生成的mesh density相似（Figure 20）
![[Pasted image 20240618163213.png]]
TetWild和我们的方法都旨在生成尽可能coarse的mesh，同时preserving input surface
	这种选择在下游应用中非常有用，可以减少它们的计算成本
	如果需要，算法还支持用户指定sizing field来增加density

与我们的方法相反，TetGen精确地保留了input surface的geometry
因此，如果input mesh是dense的，则在surface周围生成dense tetrahedral mesh，如Figure 1所示
CGAL通过implicit function逼近surface，但有时会对sharp features和tiny artifacts过度refines，如Figure 1，其中黑点是over-refined的区域
![[Pasted image 20240616160039.png]]
# 5 Applications
## 5.1 Mesh Repair
类似于TetWild，我们的算法可以通过将volume进行tetrahedralizing并提取生成的tetrahedral mesh的surface来repair不完美的triangle meshes
## 5.2 Mesh Arrangements
## 5.3 Simulation
# 6 Concluding Remarks
我们提出了一种新的robust的tetrahedral meshing算法fTedWild，它结合了TetWild的robust和基于Delaunay方法相当的运行时间
改进后的算法不仅适用于需要volumetric discretization的应用，也适用于surface mesh repair和boolean operations

我们目前naive parallelization方法表面，我们的算法受益于共享内存并行化
坍缩更先进的并行化技术，并将其扩展到高性能计算集群的分布式计算是未来工作的重要方向
我们的迭代三角形插入算法可用于动态重划分任务，可能允许重用现有网格并仅在高变形区域插入新faces

我们的算法可以使用winding number或flood fill filters来提取由input surface包围的内部volume
虽然这些启发式的方法对于表示具有一致法线方向的closed input models的不完美input很有效，但如果input surface包含未包围体积的open shells或具有错误的法线方向的nested components，则它们可能会失败（Figure 30）
这些情况下，volume不是well defined的，我们的filtering会任意丢弃或保持这些区域周围的tetrahedra
如果input包含open shell，我们建议不要依赖这些启发式的方法，而是使用特别的算法进行过滤
在nested components的情况下，我们建议纠正方向，以确保正确定义in-out
![[Pasted image 20240618164943.png]]
fTetWild使用conformal AMIPS能量来测量和优化tetrahedra的质量
[[Harmonic triangulations]]在我们的工作中同时引入一个有趣的替代方案：他们建议直接优化tetrahedralization的Dirichlet能量，并表明该措施在去除slivers是有效的，同时计算效率高
对这两种方法的对比研究是有趣的，并且使用Dirichlet能量可以进一步减少我们方法的运行时间