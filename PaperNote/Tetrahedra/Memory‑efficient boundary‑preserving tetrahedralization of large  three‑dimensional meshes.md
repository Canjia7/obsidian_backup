---
tags:
  - tetrahedralization
  - parallelization
  - mesh_generation
  - boundary_conformity
---
![[Pasted image 20240614192705.png]]
> @article{erkocc2023memory,
  title={Memory-efficient boundary-preserving tetrahedralization of large three-dimensional meshes},
  author={Erko{\c{c}}, Ziya and G{\"u}d{\"u}kbay, U{\u{g}}ur and Si, Hang},
  journal={Engineering with Computers},
  pages={1--17},
  year={2023},
  publisher={Springer}
  }
# 0 Abstract
提出了一种分治算法，对3D mesh进行boundary-preserving方式的四面体化
包含三个阶段：1、input partitioning，2、surface closure，3、merge

首先将输入分为几个部分，以减少problem size
然后应用2D三角剖分来闭合open boundaries，以创建水密的新piece
然后每个部分用tetgen（一个基于Delaunay四面体网格生成器，是我们实现的基础）
最后合并每个四面体网格来计算最终的解

此外，我们使用post-processing来移除在input partitioning阶段引入的顶点，以保持输入三角形

我们的方法的好处是，它可以减少峰值内存使用或提高处理速度，它甚至可以对tetgen无法四面体化的网格进行四面体化
# 1 Introduction
tetrahedral mesh的应用...

constrained tetrahedralization算法以3D surface或三角网格和顶点为输入，并对其input mesh的内部进行四面体化
构成输入网格的边界的表面三角形是 描述生成的四面体化边界的约束面
constrainedness性质让这些算法在许多应用中具有价值
...

用于ray tracing和FEM的3D object或surface会有着数百万顶点，constrained tetrahedralization算法必须处理大量的对象和场景
然而，当前的constrained tetrahedralization算法会fall，在内存需求超过可用内存时
此外，当前的方法可能需要大量的时间来执行

...

我们的算法思想了parallel processing和memory requirement reduction
第一个mode使用多线程减少了执行时间
第二个mode通过使用文件存储部分结果来减少单个线程的内存需求

实验表明，根据选择的模式，我们的方法要么比tetgen快，或者比它快
当tetgen无法处理大型对象时，我们的算法可以作为一种替代的四面体网格生成工具
虽然我们可能会生成一些non-Delaunay四面体，但我们可以生成比tetgen质量更好或相似的网格
我们的分治算法提高了tetgen的能力，我们最初的input division过程帮助我们实现了大量物体的四面体化，这是sequential算法无法做到的
然而，我们的memory-efficient process可能稍微比tetgen差

算法流程：
![[Pasted image 20240614194909.png]]

# 2 Related Work
## 2.1 Sequential CDT
## 2.2 Parallel Delaunay triangulation
## 2.3 Input partitioning

# 3 The proposed algorithm
算法分为三个部分：1、Input Partitioning，2、Surface Closure，3、Merge
## 3.1 Input partitioning
该算法首先将输入网格分为几部分
我们的目标是将input surface mesh划分为尽可能多的均匀大小的piece
为此需要找到将mesh划分为大小相等的部分的parallel planes

我们用主成分分析（PCA）找到这样的平面
我们对输入网格的顶点应用PCA来计算第一主成分$PC_1$，$PC_1$允许我们对input mesh划分为最大数量的piece，$PC_1$ vector是这些parallel planes的normal vector
然后我们在每个这些planes找到一个不同的point，来定义它们的equations
为了做到这一点，我们将mesh vertices投影到$PC_1$，由此我们得到了一个1D的projection array，然后我们对其排序
这个阶段，我们需要一个输入参数$k$，表示parts的数量
...
我们知道 如何寻找projected elements，并back-project它们到有着索引的世界坐标中
沿着$PC_1$的resulting point，用于构建planes
每个部分将包含大约相同数量的顶点，因此具有相同数量的faces

算法伪码：
![[Pasted image 20240614201239.png]]

当已知的planes和mesh相交时，我们需要在mesh中插入新points
point插入是必要的，因为我们希望所有的点都是planar，以简化surface closure阶段
intersection和point插入算法首先遍历mesh的所有edge
对于每条edge，我们运行一个plane-segment交叉测试，相交的结果可以是一个segment、point或nothing
当交叉是一个point，且plane不是很靠近edge的端点时，我们分裂edge
如果新插入的point靠近其中一条edge，我们就不插入该point
交叉检测使用CGAL，我们没有遇到交叉检测产生的问题
当插入新point会出现floating point错误
如果太多相交的cut plane，我们的方法可能会失败
当有如此多的parallel planes时，有些planes可能会由于floating point误差而相交，尽管在我们的实验中从未发生过

分割edge将会在edge的两个端点中引入一个新的point，我们设置新point的位置为交点的位置
如果交叉为一个line segment，我们不分裂它，因为line segment在plane上，消除了插入点的必要性
如果平面几乎与它的一个端点相交，我们也跳过这一条边，省略这一步将引入可能被认为是自交的duplicate point和tiny triangle，没有足够的floating-point精度

![[Pasted image 20240614203150.png]]
## 3.2 Surface closure
我们在open boundaries上应用hole-filling操作
虽然最先进的技术提供了各种方法来close surface并很好地遵循surface的曲率，但它们并不能满足我们的需求
我们的目标是close holes，以保证它是平面
filling boundaries减轻为2D CDT问题

前一阶段产生的每个part都有open boundaries，我们需要填补这些洞，因为tetgen仅能在closed mesh上运行
我们用2D Constrained Delaunay Triangulation来闭合边界
直接的方法可能会失败，当物体有着大于0的genus或concavity

在我们获得闭合open boundaries的三角化后，我们应用进一步的refinement来提高三角形的质量
refinement阶段用一个density control factor parameter来细分三角形，提高这个parameter会使得三角形更均匀，如下图：
![[Pasted image 20240614204859.png]]

然而，这个过程也会产生许多新的点和三角形，使要四面体化的对象变得更加复杂
然后我们将这个三角剖分插入到dividing plane的两侧网格
这样，我们就得到了两个closed、watertight、intersection-free的网格，正如tetgen所要求的，如下图：
![[Pasted image 20240614205334.png]]

surface closure process应对不同拓扑的情况...
## 3.3 Merge
在merge阶段，我们merge几个四面体网格（tetmesh），并创建一个最终的tetgen
merge步骤的一个困难在于，在cut region周围寻找四面体之间的对应关系
因为每个part是独立的四面体，相邻的四面体不会意识到彼此，所以我们在surface closure阶段存储neighborhood information
由于tetgen会保留原始的三角形，所以四面体网格创建后，cut region周围三角形最终会完美匹配

我们调整tetgen的参数，使得其保留所有的三角形，我们不允许它将任何新的点插入到boundary上，否则在merge过程中，相应的piece将不匹配
很少情况下，tetgen拒绝移除插入到boundary的Steiner points
我们发现这种情况，并立即终止进程，会导致不成功的操作
这种情况不会造成重大的问题，因为它很少发生，它在即将发布的tetgen中得到改进，在[[On tetrahedralisations of generalised Chazelle polyhedra with interior Steiner points]]中描述了Chazelle多面体（一类非凸多面体）如何在不修改其外部边界的情况下以保持边界的方式四面体化，仅在其内部插入Steiner点

在merge步骤中，我们应用post-processing来移除在划分input时引入的顶点
我们跟踪这些顶点并将它们从四面体网格中移除，这会在四面体网格中创建一个cavity，然后我们用tetgen将cavity四面体化
由于这个操作，我们确保了输入网格的原始面保持完整
![[Pasted image 20240614212731.png]]
# 4 Models of the algorithm
我们的算法有两个模式：parallel processing和memory requirement reduction
parallel processing模式使用并行化来加快处理速度
memory requirement reduction模式减少模式使用单线程变成和中间文件来减少内存使用
## 4.1 Parallel processing
## 4.2 Memory requirement reduction
# 5 Experiments
## 5.1 Mesh quality
我们使用slim energy作为质量指标，如[[Tetrahedral meshing in the wild]]中

尽管我们的算法可能会创建一些非Delaunay三角形，但与减少内存占用和计算时间所获得的增益相比，质量差异显得微不足道
## 5.2 Parallel processing
## 5.3 Memory requirements
# 6 Conclusion
我们提出了一种分治算法，可以用于减少内存使用或加快constrained tetrahedral meshing process
虽然我们的算法可能会引入一些non-Delaunay三角形，但它可以提高四面体网格的质量
我们甚至可以成功地四面体化tetgen中由于缺乏内存而无法做到的

我们可以用各种方式扩展我们的工作
首先，我们在input partitioning中使用PCA和parallel planes来减少开销
为了在速度和更平衡的分解之间权衡，我们可以尝试其它方法，例如convex decomposition和recursive PCA
convex decomposition相对较慢，并且用non-parallel planes需要复杂的合并步骤，因此整体过程可能较慢，但分解可能更平衡
其次，我们只将框架用于tetgen，然而它可以和其它网格工具（如tetwild [[Tetrahedral meshing in the wild]]），由于tetwild也可能出现out-of-memory，因此它会受益于本文方法