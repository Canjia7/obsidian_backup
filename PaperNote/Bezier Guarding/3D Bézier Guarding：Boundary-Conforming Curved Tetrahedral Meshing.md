![[Pasted image 20240806221035.png]]
> @article{10.1145/3618332,
author = {Khanteimouri, Payam and Campen, Marcel},
title = {3D B\'{e}zier Guarding: Boundary-Conforming Curved Tetrahedral Meshing},
year = {2023},
issue_date = {December 2023},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {42},
number = {6},
issn = {0730-0301},
url = {https://doi.org/10.1145/3618332},
doi = {10.1145/3618332},
abstract = {We present a method for the generation of higher-order tetrahedral meshes. In contrast to previous methods, the curved tetrahedral elements are guaranteed to be free of degeneracies and inversions while conforming exactly to prescribed piecewise polynomial surfaces, such as domain boundaries or material interfaces. Arbitrary polynomial order is supported. Algorithmically, the polynomial input surfaces are first covered by a single layer of carefully constructed curved elements using a recursive refinement procedure that provably avoids degeneracies and inversions. These tetrahedral elements are designed such that the remaining space is bounded piecewise linearly. In this way, our method effectively reduces the curved meshing problem to the classical problem of linear mesh generation (for the remaining space).},
journal = {ACM Trans. Graph.},
month = {dec},
articleno = {182},
numpages = {19},
keywords = {curved mesh, high-order mesh, isogeometric analysis, tetrahedral mesh}
}
# Abstract
我们提出了一种方法来生成high-order tetrahedral meshes
与先前的方法不同，curved tetrahedral elements被保证为无degeneracies和inversions的，同时完全conforming于规定的piecewise polynomial surfaces（例如domain boundaries或meterial interfaces）
任意polynomial order都是支持的
在算法上，polynomial input surfaces首先被一层精心构造的curved elements覆盖，使用递归refinement过程，可以证明是避免degeneracies和inversion的
这些tetrahedral elements被设计使得remaining space是piecewise linearly的
这样，我们的方法有效地将curved meshing问题简化为经典的linear mesh generation问题（对于remaining space）
# 1 Introduction
...

现有的volumetric higher-order mesh generation方法遵循一个posteriori curving：
	首先生成一个linear tetrahedral mesh，然后正式提升element的order，最后对它们精心curve化
	例如，通过优化一个几何deformation，以思想对boundary的minimal geometric approximation error
在这种情况下，有两种选择：
	约束optimization，以保持regularity（但因此可能会使其无法达到完全conformance）
	或者不约束optimization，以确保达到conformance（但因此可能会失去regularity）

而我们在下面提出的方法是一种priori的方法
tetrahedra立即在curved state下生成，保证了通过construction的regularity和conformance
construction不仅限于quadratic或cubic elements，对任意polynomial order都是支持的
在一个high-level，其遵循了用于2D mesh generation的最近的Bezier Guarding方法[[Bézier guarding：precise higher-order meshing of curved 2D domains]]
	覆盖input boundary的partially-curved elements的第一层被精心构造，有效地shielding off其潜在的复杂curved nature，这样remaining space可以使用linear meshing技术以一种兼容的方式被meshed
	我们在Section 3中详细介绍这一点

让我们指出，mesh quality（相对于mesh validity）不是这项工作的重点
	由于存在validity preserving mesh optimization和remeshing技术来提高quality，这可以被视为一个分离的单独方面

我们还注意到，除了给定domain boundary的精确interpolation之外，一个相关但不同的问题设置是（bounded）approximation of the domain
# 2 Related Work
## 2.1 2D Curved Meshing
## 2.2 3D Curved Meshing
# 3 Method Overview
我们的方法的输入是一个对$\mathbb{R}^3$的surface的description，使得生成的tetrahedral mesh应该conform于它
	这可以是要meshed的domain的boundary，但也可以包括material interfaces或任何其它需要respected的feature surfaces
	一般来说，我们假设这些surfaces被表示为任何curved polynomial triangles、Bezier triangles的structured meshes
注意到各种smooth surface表示，例如polynomial tensor product patches或（trimmed）B-spline surfaces可以被精确地划分为sufficient polynomial order的triangles
关于input假设的细节见Section 3.2

我们的目标是通过算法生成一个higher-order tetrahedral mesh，它完全conforms于这些input surfaces（即curved triangles组）
主要的概念思想如下：
1. 在每个input triangles，创建一个tetrahedron，通过构造使得它是regular的，并conform于triangle
2. 在两个input triangles的每个input edge，创建一个tetrahedra，通过构造使得它们是regular的，并conform于curved edge，并一起填补step 1构造的adjacent tetrahedra之间的gaps
重要的是，我们创建所有这些tetrahedra，使得它们不仅conforming和regular，而且所有exposed facets都是planar的，且所有exposed edge都是直的
	==exposed facets==是：不conforming于一个input triangle或一个neighboring tetrahedron
remaining space因此是polyhedral的，以piecewise linear方式有界的
3. 采用标准的linear tetrahedral conforming meshing算法来对remaining space进行mesh
4. 结合steps 1-3中所有类型的tetrahedra，以生成output mesh