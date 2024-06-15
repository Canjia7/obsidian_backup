---
tags:
---
![[Pasted image 20240615150359.png]]
> @article{10.1145/3197517.3201353,
author = {Hu, Yixin and Zhou, Qingnan and Gao, Xifeng and Jacobson, Alec and Zorin, Denis and Panozzo, Daniele},
title = {Tetrahedral meshing in the wild},
year = {2018},
issue_date = {August 2018},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {37},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/3197517.3201353},
doi = {10.1145/3197517.3201353},
abstract = {We propose a novel tetrahedral meshing technique that is unconditionally robust, requires no user interaction, and can directly convert a triangle soup into an analysis-ready volumetric mesh. The approach is based on several core principles: (1) initial mesh construction based on a fully robust, yet efficient, filtered exact computation (2) explicit (automatic or user-defined) tolerancing of the mesh relative to the surface input (3) iterative mesh improvement with guarantees, at every step, of the output validity. The quality of the resulting mesh is a direct function of the target mesh size and allowed tolerance: increasing allowed deviation from the initial mesh and decreasing the target edge length both lead to higher mesh quality.Our approach enables "black-box" analysis, i.e. it allows to automatically solve partial differential equations on geometrical models available in the wild, offering a robustness and reliability comparable to, e.g., image processing algorithms, opening the door to automatic, large scale processing of real-world geometric data.},
journal = {ACM Trans. Graph.},
month = {jul},
articleno = {60},
numpages = {14},
keywords = {mesh generation, robust geometry processing, tetrahedral meshing}
}
# 0 Abstract
我们提出了一种新的tetrahedral meshing技术，它具有unconditionally robust，不需要用户交互，可以直接将三角形汤转换为可analysis-ready volumetric mesh
该方法基于几个核心原则：
1. 初始网格的构建基于一个完全robust的，高效的，filtered exact computation
2. mesh的显式公差（自动或用户指定的）关联与input surface
3. iterative mesh improvement保证output的有效性
resulting mesh的质量是target mesh size和allowed tolerance的一个direct function：增加与initial mesh偏差 和 减少target edge length 都会导致更高的网格质量

我们的方法实现了black-box分析，即它允许在wild中可用的几何模型上自动求解偏微分方程，提供robustness和raliability comparable
	例如：图像处理算法，打开了自动、大规模处理几何数据的大门
# 1 Introduction
对一个shape的内部进行triangulating是2D和3D几何计算的基本子程序

对于需要对一个domain进行meshing的二维问题，用于constrained Delaunay triangulation问题的robust和高效的软件对于robust和高效的pipeline发展有着巨大的作用
特别是那些需要求解PDE的问题，在给定polygon boundary内robust 2D triangulation对于fast point location、path traversal，distance queries也是必不可少的空间划分

在3D中，对给定triangle surface mesh的内部进行robust triangulation，也一样有着动机
虽然在这个问题的各个方面取得了巨大的进展，但现有的方法还远远不能解决这个问题
虽然涉及3D tetrahedralization of smooth implicit surfaces非常成熟，但使用mesh作为input的pipeline要么仅限于简单的形状，要么通常需要人工干预
	由于未说明pre-conditions，用户可能不得不fix input surface mesh来诱导网格成功
	或者由于无法满足基本的post-conditions（如manifold），用户必须对output tetrahedral进行修复
现有的方法通常无法支持automatic pipelines，例如机器学习应用程序的大量数据处理或形状优化
在许多情况下，虽然网格划分可能成功，但输出网格的大小可能是很昂贵的（对于许多应用程序来说），因为一种方法缺乏对input surface的quality of approximation和size of the output mesh的控制
	即使存在这样的控制，input mesh的难以检测的特征也可能无法保留

本文我们提出了一个新的方法来处理由任意mesh表示的mesh domains（通常是ambiguously），不需要对mesh manifoldness，watertightness，absence of self-intersections进行假设
而不是将mesh repair视为单独的preprocessing问题，我们认识到，clean meshes在许多情况下更多的是例外而不是规则
> ==Rather than viewing mesh repair as a separate preprocessing problem, we recognize the fact, that “clean” meshes are more of an exception than a rule in many settings.==

基于对实际网格问题的仔细分析，以及现有的最先进解决方案的缺点，我们的方法的主要特点是：
* *我们认为输入从根本上来说是不精确的，允许在用户定义的大小$\epsilon$的envelop内的输入出现偏差*
* *我们没有对input mesh structure作任何假设，并相应地重新制定了meshing problem*
* *我们遵循robust第一的原则（即算法应该未最大范围的输入产生有效且尽可能有用的输出），并在鲁棒性约束允许的范围内进行质量改进*
* *虽然允许input的偏差，这对质量和性能都是至关重要的，但我们的目标是使我们的算法conservative，使用input surface mesh作为3D mesh construction的起点，而不是丢失其连通性并仅使用surface sampling*

我们的方法被明确设计为输出float point坐标，但同时在有理数下严格封闭，允许它整齐地适用于robust，exact rational computational几何pipelines

我们通过经验比较了最先进的方法和我们的新方法，在Thingi10K的10000个模型的大型数据库上的性能和鲁棒性
为了促进结果的replicability，我们发布了我们的算法的完整参考实现、论文中显示的所有数据、以及复制我们结果的脚本

我们的方法虽然较慢，但当应用于在wild发现的meshes时，在许多质量度量上，我们的方法在robust和结果质量上有了显著的提高
# 2 Related Work
### Background Grids
### Delaunay
###  Restricted Delaunay tetrahedralization
###  Variational meshing
###  Surface Envelope
# 3 Method
我们首先更精确地定义我们的问题
作为输入，我们假设一个triangle soup，一个用户指定的tolerance $\epsilon$，和一个期望的target edge length $l$
目标是构建一个approximately constrained tetrahedralization，即一个tetrahedral mesh：
1. 包含input的set of triangles的approximation，在用户定义的$\epsilon$内
2. 没有inverted elements
3. edge length低于用户指定的界限$l$
在满足这些约束条件的同时，优化网格质量
如果一个mesh满足前两个条件，则称为==valid==的

resulting tetrahedralization可以用于各种目的
最重要的是，我们可以使用一组triangles内部的任意definition来提取一个包含在input triangles soup“内部”的tetrahedralized volume

本文中，我们使用术语surface来指代surface的集合，不一定是manifold、或connected、或self-intersection free
我们的算法分为两个不同的阶段来处理这个问题：
1. the generation of a valid mesh，不考虑其geometric quality，用arbitrary-precision有理数表示其坐标
2. 改进element的geometric quality，并将顶点坐标四舍五入为floating point数，同时保持mesh的validity
解耦这两个子问题是我们算法的关键，它与大多数竞争方法形成对比，这些方法试图直接生成高质量的网格

第一阶段只依赖于在有理数下closed的操作
	即，如果顶点坐标是有理数，则可以精确地执行整个计算，从而避免所有robustness问题（但也增加了计算成本）
第二阶段使用hybrid geometric kernel，允许我们在可能的情况下切换到浮点运算，以保持运行时间合理（Section 3.4）
因此我们的算法保证产生valid mesh（阶段1），但我们不能对其质量提供正式的约束（阶段2）
在实践中，我们的prototype在数据集中10000个wild模型获得的质量很高（Section 4）
### Overview
该算法创建了一个volumetric Binary Space Partitioning（BSP）tree，每个input triangles包含一个plane，并将其坐标存储为精确的有理数

通过构造，resulting convex（但不一定是严格的convex）cell分解符合input triangle soup，并且可以通过将每个cell独立地四面体化来轻松创建tetrahedral mesh
volumetric mesh不仅在模型内部创建，而且在模型周围创建，填充一个比输入略大的bounding box
这使我们robust地处理包含gaps和self-intersection的不完美几何形状，并将空间的inside/outside segmentation推迟到pipeline的后期阶段

然后通过一组和局部操作来对mesh element进行refine，coarsen，swap，smooth（Section 3.2）
这些操作只有在它们不破坏一组能够确保validity of the mesh的invariants，才会被执行

然后使用winding-number filtering来提取final mesh，其对不完美的real-world input具有robust
## 3.1 Generation of a Valid Tetrahedral Mesh
robust地生成一个保持原始triangle soup的faces的valid tetrahedral mesh是有挑战性的，即使是忽略任何的quality考虑
real-world meshes进程受到各种defects的困扰，包括degenerate elements、holes、self-intersection、topological noise
即使是手工建模的CAD几何也不能导出clean boundary format，因为最常见的建模操作在spline representation下closed，不可避免地出现小cracks和self-intersections

清洗polygonal meshes或CAD models是一个长期存在的问题，bullet-proof solutions仍然难以捉摸

因此，我们建议按照原样的input geometry，并依靠robust几何构建来将整个volume填充为tetrahedra，而不需要在这个阶段承诺boundary的几何和拓扑，并将此挑战推迟到pipeline的后期阶段，在所有的degeneracies被消除后
### BSP-Tree Approach
我们建立了一个精确的BSP细分，使用infinite-precision有理坐标，并且只依赖于在此表示下的closed操作
pipeline的2D示意图见Figure 2：
![[Pasted image 20240615193933.png]]
与surface-conforming Delaunay tetrahedralization相反，其对于设计一个robust的实现是很有挑战的，不受约束的版本可以用精确有理数robust地实现
因此，我们创建了一个初始的、不一致的tetrahedral mesh $M$，其顶点和输入的triangle soup相同，使用CGAL中的精确有理内核

生成的tetrahedral mesh不保持input surface，使其无法应用于大多数下游应用
为了加强conformity，我们使用的方法受到了[[BSP-Assisted Constrained Tetrahedralization]]的启发，但设计于保证valid output
我们将input triangle soup中的每个triangle视为一个plane，并将其与$M$中包含的所有tetrahedra相交
换言之，我们将每个tetrahedron作为一个BSP cell的root，并使用input几何中所有的与之相交的triangles来切割cell
这个计算可以完全使用有理坐标来完成，因为planes之间的交点在有理数下是closed的，即使对于degenerate input也能确保robustness和correctness

利用cell的convexity将polyhedral mesh转换为tetrahedral mesh：
	我们将它的faces进行triangulate，在重心处添加一个顶点，并将其连接到在boundary的所有triangular faces
由于唯一需要的操作是顶点位置的平均值，因此可以用有理数精确地计算重心
只要至少有四个输入顶点是线性无关的，那么所有的convex cell都是non-degenerate
	也就是说，连接到重心产生的resulting tetrahedra也将是non-degenerate（尽管可能质量很差）

output mesh是valid的，且完全conforming于input triangle soup

input中的self-intersections可以自然地处理：
	它们显式地meshed，相应地splitting为相应的三角形（Figure 3）
![[Pasted image 20240615200049.png]]
## 3.2 Mesh Improvement
给定一个用有理数表示的valid tetrahedral mesh，我们提出了一个算法来提高其quality，并将其顶点四舍五入到floating point位置，同时保持其有效性
我们遵循基于局部网格改进操作的常见greedy optimization pipeline，但有四个重要的区别：
1. 我们明确地使用exact predicates来防止inversions（Validity Invariant 1）
2. 我们在操作过程中追踪surface mesh，并且我们只允许自input triangle mesh保持一个$\epsilon$距离的操作
3. 我们使用一个conformal energy来直接惩罚所有shapes的bad elements
4. 我们使用hybird geometric kernel来减少计算时间，同时确保correctness和termination，尽可能使用floating point，只有在严格必要的地方以来exact坐标
### Invariant 1: Inversions
我们不允许引入方向为负的inverted tetrahedra，使用exact predicates来实现有理和floating point坐标
这确保了output没有inversions，从算法在我们的BSP-tree算法开始构建inversion-free tetrahedral mesh（Section 3.1）
### Invariant 2: Input Surface Tracking and Envelope
通过构造，Section 3.1中生成的tetrahedral mesh包含所有input triangle的精确表示，以tetrahedral的face的collection的形式
	也就是说，tetrahedral mesh包含一个（或多个）tetrahedra，whose faces精确地与input triangle匹配
我们把这个face的collection称为==embedded surface==，且所有在tetrahedral mesh上执行的操作会保持追踪它

为了限制mesh improvement过程中引入的geometric approximation error，我们只接受保持embedded surface的faces在一个小于用户指定$\epsilon$的距离
直观的说，这可以被描述为在input triangles soup周围建立一个厚度为$\epsilon$的envelope
通过不允许任何破坏这个invariant的操作，我们确保embedded surface始终包含在envelope中（Figure 4）
![[Pasted image 20240615202354.png]]
### Quality Measure
作为优化的quality的度量，我们使用了3D conformal energy，这与许多常见的quality度量标准密切相关，表示为：$\varepsilon=\sum_{t\in T} \frac{tr(J_t^T)J_t}{det(J_t)^{\frac{2}{3}}}$ ，其中$J_t$是将tetrahedron $t$转换为regular tetrahedron的独特3D deformation的Jacobian
这个energy不受到任何isotropic scaling的影响，但自然会惩罚needle-like element、flat和fat elements、slivers，并防止inversion
	因为当element接近零体积时，它会发散到无穷大
它是可微的，可以用Newton或Quasi-Newton iteration有效地最小化
### Local Operations
我们使用四种局部操作来改进mesh：
	edge splitting、edge collapsing、face swapping、vertex smoothing（Figure 5）
![[Pasted image 20240615203819.png]]
这些操作只影响mesh的局部区域，因此可以高效地执行

我们提出了一种非对称的优化方案：
	coarsening和optimization算子只有在改善mesh quality时才会应用
	而refinement算子则会应用到达到预定义的edge length（用户控制），或者缺乏足够的自由度
这一策略背后的基本原理是，我们希望避免在没有必要提高质量的区域过度调整，因此我们只添加额外的顶点来匹配用户提供的密度，或者如果需要提高质量，则在局部添加额外的顶点
这种策略允许我们生成高质量的网格，即使input surface质量较低（Figure 6）
![[Pasted image 20240615204446.png]]
我们使用四个passes来优化mesh：
1. splitting（refining）
2. collapsing（coarsening）
3. swapping
4. smoothing
我们在tetrahedral mesh的顶点处存储一个target edge length值，用用户指定的期望edge length $l$来初始化
...
## 3.3 Interior volume extraction
注意，到目前为止，我们的算法还没有尝试定义一个closed surface来bounding a volume：
	之前阶段构建了一个approximately constrained tetrahedralization，可能有一个nonmanifold、disconnected、open embedded surface
我们使用[[Robust Inside-outside Segmentation Using Generalized Winding Numbers]]中提出的方法来解决embedded surface可能存在的缺陷
	方法是定义一个inside-outside函数，该函数可以用于提取与mesh相关的interior volume
...
## 3.4 Technical Detail
### Hybrid Kernel
### Voxel Stuffing
### Input Simplification
### Open Boundaries
### Envelope
# 4 Results
### Robustness and Performance
### Parameters
### Spatially Varying Sizing Field
### Surface Repair
### Finite Element Method Validation
### Structured Meshing
### Noise Stress-Test
### Meshing for Mulimaterial Solids
# 5 Limitations and Concluding Remarks