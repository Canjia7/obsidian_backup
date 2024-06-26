> @inproceedings{10.1145/800076.802459,
author = {Chazelle, Bernard M.},
title = {Convex decompositions of polyhedra},
year = {1981},
isbn = {9781450373920},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/800076.802459},
doi = {10.1145/800076.802459},
abstract = {An important direction of research in computational geometry has been to find methods for decomposing complex structures into simpler components. In this paper, we examine the problem of decomposing a three-dimensional polyhedron P into a minimal number of convex pieces. Letting n be the number of vertices in P and N the number of edges which exhibit a reflex angle (i.e. the notches of P), our main result is an O(nN3) time algorithm for computing a convex decomposition of P. The algorithm produces O(N2) convex parts, which is optimal in the worst case. In most situations where the problem arises (e.g. graphics, tool design, pattern recognition), the number of notches N seems greatly dominated by the number of vertices n; the algorithm is therefore viable in practice.},
booktitle = {Proceedings of the Thirteenth Annual ACM Symposium on Theory of Computing},
pages = {70–79},
numpages = {10},
location = {Milwaukee, Wisconsin, USA},
series = {STOC '81}
}
# 0 Abstract
计算几何的一个重要研究方向是寻找方法来分解complex structures为simpler components
本文中，我们研究了将一个3D polyhedron $P$分解为最小数量convex pieces的问题
令$n$为$P$的顶点数量，$N$为呈现reflex angle的边（即$P$的notches）的数量
我们的主要结果是一个$O(nN^3)$时间的计算$P$的convex decomposition的算法
算法产生$O(N^2)$块convex parts，其是最差情况下的最优
在该问题出现的大多数情况下（如graphics，tool design，pattern recognition），notches $N$的数量似乎很大程度上受顶点数量$n$的支配，因此该算法在实践中是可行的
# 1 Introduction
寻找有效的方法将non-convex对象rewriting为convex parts的并集是重要的
本文研究问题：
	给定一个3D polyhedron $P$，寻找其最小的pairwise disjoint convex polyhedra集合，使其并集恰好是$P$

polygon的minimal decomposition通常对于顶点数量是linear的，但对polyhedra的情况并非如此
我们将证明minimum number可能是quadratic的，这为问题的复杂性提供了一个平凡的下界

令$n$为==顶点的数量==，令$N$为其==notches的数量==（即边，其邻接面构成一个reflex angle）
我们证明该算法产生的convex polyhedra的总数不会超过$N^2/2$
## 2 The Convex Decomposition Algorithm
我们定义一个3D polyhedron为一个simple plane polygons的有限connected set，使得每个polygon的所有边都恰好属于另一个polygon
为了排除退化的情况，我们要求每个顶点周围的polygons形成一个simple circuit
我们允许faces和polyhedron有孔洞，我们定义==一个face的genus==为其孔洞的数量，==一个polyhedron的genus==为构成其边界的surface的genus
	即，$P$是一个polyhedron，有着$n$个顶点$v_1,...,v_n$，有$p$条边$e_1,...,e_p$，有$q$个面$f_1,...,f_q$
	$P$的一条边称为是notch的，如果其邻接面构成了一个reflex angle（相对于它们的内侧），令$g_1,...,g_N$为$P$的notches

令$g$为一个$P$的notch，令$T$为一个包含$g$的plane且resolve其reflex angle（即$T$和$g$的邻接面形成的两个angle都不是reflex的）
观察到$T$和$P$的交叉通常是一组polygons（Figure 1）
	这些polygons可能有holes，且holes本身包含其它polygons
![[Pasted image 20240626111434.png]]
令$S$为包含$g$的唯一polygon，称$S$为decomposition的一个==cut==
可以知道沿着$S$ cutting $P$会移除notch $g$

一般情况下，这个操作会将$P$分为两部分
然而，由于它也可以引入handles，即创建非零genus的polyhedra（Figure 1），因此我们必须考虑$P$有任意genus的情况

然后移除$g$可能简单地cut一个$P$的handle，并保持其connectivity（Figure 2）
所有情况下，我们观察到在每个non-convex part重复此过程最终会产生$P$的一个convex decomposition
![[Pasted image 20240626112018.png]]
注意，移除一个notch可能会将其它notches分裂为两个segments（称为==subnotches==），每一个都属于一个不同的polyhedron（Figure 3）
![[Pasted image 20240626112220.png]]
...

我们接下来稍微修改这个方法，使其保证$O(N^2)$个convex part
我们只需要确保每个notch的所有subnotches都被coplanar cuts去除
准确地说：对每个notch $g_i$定义一个plane $T_i$，使其能resolve这个reflex angle，我们像之前那样处理，额外的要求是$g_i$的每个subnotch的cuts都要coplanar
### Theorem 1
改进后的算法应用到$P$上至多产生$N^2/2+N/2+1$个convex parts
	proof：共面的cuts至多与其它notch相交于一点
