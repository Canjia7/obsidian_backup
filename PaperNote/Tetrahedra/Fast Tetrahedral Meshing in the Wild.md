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
## 3.1 Algorithm Overview
## 3.2 Envelope
## 3.3 Preprocessing
## 3.4 Incremental Triangle Insertion
### 3.4.1 Background Mesh Generation
### 3.4.2 Single Triangle Insertion
#### Finding Cut Tetrahedra
#### Plane-Tetrahedra Intersection
#### Table-based Tetrahedron Subdivision
### 3.4.3 Open-boundary Edge Preservation
## 3.5 Mesh Improvement
## 3.6 Filtering
# 4 Results
## 4.1 Success Rate
## 4.2 Running Time
## 4.3 Mesh Quality
## 4.4 Mesh Density
# 5 Applications
## 5.1 Mesh Repair
## 5.2 Mesh Arrangements
## 5.3 Simulation
# 6 Concluding Remarks