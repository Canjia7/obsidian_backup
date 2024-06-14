![[./Pic/Pasted image 20240614192705.png]]
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

