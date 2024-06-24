> @ARTICLE{5570949,
  author={Liu, Shengjun and Wang, Charlie C. L.},
  journal={IEEE Transactions on Automation Science and Engineering}, 
  title={Fast Intersection-Free Offset Surface Generation From Freeform Models With Triangular Meshes}, 
  year={2011},
  volume={8},
  number={2},
  pages={347-360},
  keywords={Surface reconstruction;Solid modeling;Computational modeling;Narrowband;Solids;Three dimensional displays;Mathematical model;Filtering;freeform surface;intersection-free;offset surface generation;signed distance field},
  doi={10.1109/TASE.2010.2066563}}
# 0 Abstract
介绍了一种快速的offset surface生成方法，构建的是无交集的offset surface，且能保持尖锐特征。
基本思想是对input模型采样一个narrow-band带符号的距离场，然后使用一个contour算法根据距离场构建出offset mesh surface。
构造了四个filter来便捷生成narrow-band。
contour中用一种混合方法来避免使用昂贵的二分算法。且基于凹凸分析，更加robust和高效。
### Note to Practitioners
首先offset surface被采样到一个隐式表示中，即narrow-band带符号的距离场。
然后contour为一个无交集的mesh surface，以近似于offset surface。
# 1 Introduction
offset surface定义：一个固体H的offset surface是一些与H和H的边界有相同offset距离r的点。
考虑距离的符号，向外扩展的offset surface为H_r^+，其上的点都在H外部；向内收缩的offset surface为H_r^−，其上的点都在H内部。
==则本文要解决的问题==：给定固体H，其表面表示为三角网格，需要计算其表面的offset surface。

offset操作在数学上是良定的（1），当对固体进行offset是困难的。
对于多边形mesh，可以从点、边、面来看待，一个构造offset surface的方法也基于这些元素。但是在计算上很难。
（4，5）中用体积的方法，（6）中用基于点的方法。这些方法首先构造一个体积网格，然后采样点来近似offset model，然后用距离场或碰撞检测技术来计算隐式曲面或采样点。
但是这种方法会丢失尖锐特征（如4，6），或计算时间长（如4，5）。

我们的方法可以归类为体积方法.
由于offset surface有固有特性，即每个offset surface上的点距离原mesh的最短距离都是r。可以利用这个特性在三角面上直接定义距离函数。
则以固体H生成的距离为r的offset surface上的点p，可以定义为：
	f(p)=dis(p,∂H)−r=0
	其中dis为带符号的。
带符号的距离场在narrow-band方法中在均匀网格上采样的，即只需要计算offset surface上网格点的距离值。
关于构造带符号的距离场，可以使用后续介绍的四个filter可以显著减轻计算冗余。

然后使用dual contouring方法（7）提取网格，可以保留尖锐特征，该方法需要精确计算网格边和offset surface的交点。
此处利用一种混合方法来避免使用二分法计算交点。
# 2 Related Work
offset操作可以看作是一种特殊情况的Minkowski sum（1），有几种方法。

offset操作还可以应用在曲线、表面或者整个3D模型中。

有许多用于offset surface的生成方法。
# 3 Overview
给定一个被两个定向的2-manifold mesh为界限的固体model，目标是获得一个没有交集的offset surface，且保留尖锐特征。
算法包含三个步骤：
	1、给定模型H被嵌入在一个空间Φ中，这个空间包含了该模型和其offset surface。
	该空间Φ被均匀网格采样到一个narrow-band带符号的距离场中，其中narrow-band在offset surface周围，但不在H的surface周围。
	用四个filter来高效生成距离场。（见Section 4）
	2、计算距离场网格的边和offset surface的交点。
	通过分析可能的交点情况，可以用一个高效的反法来计算。（见Section 5）
	3、用一个无交点dual contouring算法来提取offset surface的mesh。
	我们的算法基于凹凸分析。（见Section 6）
Figure 1给出了2D情况的算法讲解。
![[Pasted image 20240623172201.png]]
# 4 Filters for distance-field  construction
构建一个给定模型H的带符号距离场，在一个n×n×n的均匀网格中。
最直接的方法是计算这n^3 个点到原surface的最短距离。
H和∂H是用三角网格表示的。

计算点p到∂H的最短距离，可以改为搜寻∂H的所有三角面。
当找到∂H上一个最近点c_q，则带符号的距离可以由c_q 的angle weighted pseudonormal决定（22）。

尽管计算距离是很高效的，但是计算一个点集还是很耗时的，特别是当n很大的时候。因此设定四个filter来提高速度。
### A Swept Sphere Volume Hierarchy (SSVH) Filter
SSVH用在∂H的三角面集上，用于提高点的距离的计算速度。
思路对三角面集为建立一个体积包围体系（volume bounding hierarchy，BVH），这样在计算时可以把离点集较远的三角面抛弃掉。
SSVH在（23）中介绍了，改进后可以计算一个点到一个多边形集合的最短距离。

我们使用（24）中一个快速的算法计算一个点到一个三角形的距离。
三角形可以定义为：t(u,v)=b+ue_0+ve_1，参数域为D={(u,v):u,v∈[0,1],u+v≤1}。
则最短距离的计算即寻找一个(u^′,v^′ )∈D，使得其为到查询点q的距离最近。
称：三角形holding c_q 。
参数u',v'以及带符号的距离，都会在后续使用上。
### B Bounding Box Filter
尽管使用了SSVH，但是对于n^3 个点还是耗时太多了。
所以应该只计算H的offset surface周围的narrow-band区域的距离场。
一般而言，offset surface在网格cell的narrow-band区域中。
#### Definition 1
对于网格中的任何结点，如果点p的位置满足|dis(p,∂H)−r|≤l，其中l为网格box的宽度，则该网格点p是offset surface的有效网格节点。
如果网格节点不满足，则称为无效网格节点。
可见Figure 1。

然而直接计算一个网格节点是否有效还是要涉及计算点到三角网格的距离，为了避免这种情况，接下来介绍bounding box filter。

网格cell被分组为大体积的立方体bounding box，每个bounding box包含m×m×m个网格box，即包含(m+1)^3 个网格节点。则原先的均匀网格被分为(n−1)/m×(n−1)/m×(n−1)/m 块。
#### Remark 1
对于一个半径δ的球体，如果其中心s_c 到H的边界surface上的最短距离满足：|dis(s_c,∂H)−r|>l+δ，则球内的所有网格节点都是无效的。
bounding box的外接球半径是(√3 ml)/2，因此我们可以Remark 1来设定δ=(√3 ml)/2 来检测哪些网格节点是无效的。（此时的bounding box由m×m×m个原网格cell组成）

检测Remark 1，需要计算所有bounding box的带符号距离，为了加速计算，我们计算无符号距离undis(s_c,∂H)，只要|undis(s_c,∂H)−d|>l+δ，d为r的绝对值，可以达到和Remark 1同样的效果。
如果满足Remark 1的话称为无效bounding box，否则称为有效bounding box。
#### Remark 2
所有在无效bounding box内的网格节点都是无效的，但在有效bounding box内的网格节点不一定是有效的。

Figure 2中展示了该Filter的结果，对于本就在offset surface的点应用此Filter后结果不变。
![[Pasted image 20240623172311.png]]
### C Signed Distance Filter
为了保留只有在offset surface周围的bounding box，使用以下Filter检查在有效bounding box中的带符号距离dis(s_c,∂H)。
#### Remark 3a
对于扩张出去的offset surface，即r>0，如果在bounding box中的网格节点满足：dis(s_c,∂H)∈[−d−l−δ,min⁡(−d+l+δ,d−l−δ)]的点是无效的。
#### Remark 3b
对于收缩进去的offset surface，即r<0，如果在bounding box中的网格节点满足：dis(s_c,∂H)∈[max⁡(d−l−δ,−d+l+δ),d+l+δ]的点是无效的。

当offset的距离很小时，两个Remark 3的区域重叠在一起。
Figure 3展示了应用的图示。
![[Pasted image 20240623172344.png]]
### D Octree Filter
bounding box filter和signed distance filter方法可以快速辨别出无效的网格节点。
然而，当选择了一个大尺寸的bounding box后，许多无效网格节点也会被包含在有效bounding box中。（如Figure 4a）
为了更进一步排除无效网格节点，介绍一个octree filter。

从一个被保留的bounding box开始，迭代不断将bounding box分为八个sub-box。
对于sub-box记为B，计算其中心到∂H的带符号距离dis_B
。如果该距离满足Remark 1，则该sub-box的分裂停止，并将其内的所有网格节点标记为无效的。
# 5 Surface Intersections on Grid Edges
通过应用以上四个filter，可以高效地构建一个offset surface的narrow-band带符号的距离场。
带符号的距离只在有效网格节点上计算。
通过局部的带符号距离计算，可以检测一个网格cell是否与offset surface相交。
### Definition 2
对于距离场中一个网格边e，其有两个网格节点p_1,p_2，若满足f(p_1 )f(p_2 )<0，则该网格边被称intersected edge。
其中f(p)=r为offset surface的隐式表示。

使用改进的dual contouring算法来提取网格surface，且能够自动重建出尖锐特征和进行并行计算。
dual contouring算法在均匀网格上需要知道intersected edge和隐式surface的交点，以及该交点在surface上的法向量。
交点可以通过计算f((1−α) p_1+αp_2 )=0。
由于带符号的距离dis在f(p)中不是线性的，所以常见的求解方法时二分法，但是我们不是这么做的。
### A. Feasibility of Analytical Solution
#### Definition 3
对于一个查询点q，用cp(q)记为在∂H上最靠近点q的点。

当一条网格边e的所有点p在∂H上最靠近的点cp(q)，都在∂H的同一个三角形上，则有一种分析的解决方法。
这种情况经常出现在网格边足够短的情况下。
#### Remark 4
当一个intersected edge e的两个端点p_1,p_2，满足cp((1−α) p_1+αp_2 )∈T,∀α∈[0,1]，则cp(p_1 ),cp(p_2)在同一个三角形T中。
反之不一定成立。

如果一条intersected edge不满足Remark 4，则使用二分法搜索其与offset surface的交点。
同时还需要检测cp(p_1 ),cp(p_2)是否在三角面的边界上，因为可能有几种情况（三角形t_1,t_2 邻接，公共边e）：
	cp(p_1)在三角形t_1 内，cp(p_2)在边e上。
	cp(p_1)在三角形t_1 内，cp(p_2)在t_1,t_2 的共同顶点v上。
	cp(p_1)在t_1 的边e上，cp(p_2)在t_2 的顶点v上，有一个共同的三角形邻接e,v。
	cp(p_1 ),cp(p_2)在不同三角形的顶点上，有一个三角形邻接这两个顶点。

当Remark 4满足时，寻找intersected edge e与offset surface的交点，即f((1−α) p_1+αp_2 )=0的解α_0 。
找到解后，作一步检验步骤，检查是否成立|dis((1−α_0 ) p_1+α_0 p_2 ),∂H)−r|≤ε，即计算出的交点到offset surface的距离是否在误差范围内，取ε=10^(−5) 。
如果不成立，就用二分法重新计算交点。（二分法同样在ε误差中计算）

在找到点p到offset surface上的交点cp(p)后，点p在offset surface的法向可以容易得到：(p−cp(p))/(||p−cp(p)||) 。
如果offset surface是收缩的情况，则取负号。
### B. Analytical Solution
对于一个intersected edge e，两个端点为p_1,p_2 。
如果e上所有点到∂H的最近点都在同一个∂H上的三角形内，则我们会在边e上找一个点p，满足f(p)=0。
分析后，一个查询点q在三角形T周围的不同区域，它的在T上的最近点cp(q)可能会在三角形的面上、边上、点上。
一般地，将三角形T周围的区域分为七个区域（见Figure 5a）。

#### Definition 4
将一个三角面T周围的区域划分为：如果一个点
不在T的边界上，称为T的face region。
在T唯一的边上，称为T的边e_i 的edge region。
在T唯一的点上，称为T的点v_i 的vertex region。

#### Remark 5
详见Figure 5b。

对于线段上所有点都落在edge region的情况，解不明显，如Figure 5c。
但也可以推到出计算公式。
#### Remark 6
	对一个线段p_1 p_2，如果线段上的所有点都落到三角形T的edge region时，可以通过解一个方程算出α的有效值。

当线段上所有点落在vertex region上时，见Figure 5d。
#### Remark 7
对一个线段p_1 p_2，如果线段上的所有点都落到三角形T的vertex region时，可以通过解|(|q−(1−α) p_1−αp_2 |)|^2=r^2 算出α的有效值。

以上方法用于快速计算intersected edge e的精确的交叉点，但是实际上edge经常会穿过多个区域。、
为了能够应用以上方法，需要将边分为好几段，使得每一段都在一个唯一的单面、边、点上。
具体的，分段后端点为s_p1,s_p2，如果f(s_p1 )f(s_p2 )>0则忽略它；如果如果f(s_p1 )f(s_p2 )<0，则可以用之前的算法计算交点。

实际应用中，为了更快计算分割后线段。
在已知网格边e的第一个端点落在那个区域后，可以将三角面T的bounding plane按顺时针或逆时针顺序排序，再计算。
# 6 Contouring：An Intersection-free method
该section目的为研究一个高效的方法来生成无交叉的等值surface，从均匀采样的网格中。使用的是dual contouring（DC）方法。
DC方法能够生成无裂缝的mesh surface，且如果是Hermite data（交点带法向）的DC方法能够保留尖锐特征。
然而，DC方法生成的surface能难做到没有自交。
说明了在（8）中提到的DC方法的问题。

在使用DC算法时在均匀网格中，等值面的生成只需要连接由boundary网格cell生成的点。
boundary网格cell是指网格cell的八个网格顶点有不一致的配置。
在每个boundary网格cell c中，创建输出网格上的点v_c，落在定义在采样自网格c的edge的Hermite data的最小二次误差函数（Quadratic Error Function，QEF）位置。
每个cell的intersection edge都包含一个符号变化，一个端点在offset surface的inside，另一个在outside。
通过连接围绕cell的四个顶点来构造两个三角面。
啊但是这种方法很难做到无自交。

在（8）中介绍了一种混合方法，混合了原始和dual的方法，见Figure 6。
# 7 Experimental Result and Discussion
### A. Experimental Tests
测试了大量模型，生成扩展或收缩surface，以不同的offset参数（以包围盒对角线的倍率）。
Table 1中展示了用时统计。
Table 2中展示了对于同一个模型用不同尺寸的网格尺寸的计算时间。
通过统计，我们的方法对于网格的尺寸和三角面的数量增长有很好的适应性。计算时间与网格尺寸近乎呈线性增长。
Table 3中展示了不同offset距离对计算时间的影响。
总的来说，对于不同的三角面，不同的offset参数，不同的网格尺寸都有好的表现。

Table 4中展示了使用了Filter后能够节省97%~99.9%的无效点计算。
Table 5中展示了intersected edge的计算效率。
Figure 20中给出了移除自交三角形的例子。

我们的方法是体积的，且避免自交，且offset surface是带距离函数定义的。
所以能够保证墙的厚度均匀，在生成空心模型时很重要，见Figure 21。
### B. Discussion
与（5）中算法的对比，为了对比控制为单核，再添加上并行。

Section 6中给出的自交检测算法与（8）中方法作比较。

Table 7展示了对结果model的精度分析，包括平均距离误差和最大距离误差。
我们的算法在平均距离误差上很小，但是最大距离误差有时候很大。
这是因为我们本身contouring算法的局限性。

另一个局限性是当使用一个固定的网格尺寸来采样问题所在区域时，小于采样距离的特征会在重建过程中丢失。这样会导致不拓扑同胚。

另一个局限是一些比采样距离小的尖锐特征会在无自交重建中被损坏。
然而在原始DC算法中能够保存。
# 8 Conclusion
本文介绍了一个快速的offset surface生成算法，生成的网格无自交，能够保持尖锐特征。
基本思想是：
	对input model采样一个带符号的距离场，以均匀网格的形式，只需要采样一个narrow-band就可以了。
	然后应用一个contouring算法从距离场中生成最后的结果网格。
用四个Fliter来以一个快速的方法生成距离场。
我们的方法中，用一个改进过的dual contouring算法，基于精确计算网格edge与等值面的交点。
用一个复合方法来避免使用昂贵的二分计算交点。
改进的无自交dual contouring方法基于凹凸分析，更加鲁班和高效。
许多模型测试和与先进方法对比。
