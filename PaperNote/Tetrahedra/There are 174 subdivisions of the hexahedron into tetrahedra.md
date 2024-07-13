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