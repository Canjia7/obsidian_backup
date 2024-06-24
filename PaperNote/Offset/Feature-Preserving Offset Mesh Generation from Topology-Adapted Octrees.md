![[Pasted image 20240623171556.png]]
> @article{https://doi.org/10.1111/cgf.14906,
author = {Zint, D. and Maruani, N. and Rouxel-Labb, M. and Alliez, P.},
title = {Feature-Preserving Offset Mesh Generation from Topology-Adapted Octrees},
journal = {Computer Graphics Forum},
volume = {42},
number = {5},
pages = {e14906},
keywords = {CCS Concepts, • Computing methodologies → Shape modeling},
doi = {https://doi.org/10.1111/cgf.14906},
url = {https://onlinelibrary.wiley.com/doi/abs/10.1111/cgf.14906},
eprint = {https://onlinelibrary.wiley.com/doi/pdf/10.1111/cgf.14906},
abstract = {Abstract We introduce a reliable method to generate offset meshes from input triangle meshes or triangle soups. Our method proceeds in two steps. The first step performs a Dual Contouring method on the offset surface, operating on an adaptive octree that is refined in areas where the offset topology is complex. Our approach substantially reduces memory consumption and runtime compared to isosurfacing methods operating on uniform grids. The second step improves the output Dual Contouring mesh with an offset-aware remeshing algorithm to reduce the normal deviation between the mesh facets and the exact offset. This remeshing process reconstructs concave sharp features and approximates smooth shapes in convex areas up to a user-defined precision. We show the effectiveness and versatility of our method by applying it to a wide range of input meshes. We also benchmark our method on the Thingi10k dataset: watertight and topologically 2-manifold offset meshes are obtained for 100\% of the cases.},
year = {2023}
}
# 0 Abstract
介绍了一种可靠的方法，用于生成一个input的三角mesh或三角soup的offset mesh。
我们的方法分为两步：
	第一步是在offset surface上执行一个dual contouring方法，在offset某些拓扑复杂的地方时，放在一个自适应八叉树上执行。相比isosurface方法我们的方法降低了内存消耗和计算时间。
	第二步用一个offset-aware算法改进输出的dual contouring mesh，减少mesh facets和实际offset的法向偏差，这一remeshing处理构建凹的尖锐特征和凸的光滑形状近似，达到用户指定的精度。
测试了一系列模型，offset mesh达到100%的watertight和拓扑上2-manifold。
# 1 Introduction
滚动一个固定半径的球在一个surface上，这样定义了一个体积的边界被称为offset surface。更正式的的，这样的操作被认为是surface和球体的Minkowski和。
offset surface也可以被定义为input的无符号距离场的水平集。
构建offset surface是一个基本工具，在CAD的很多应用中，如碰撞检测，路径规划，边界图层mesh生成，建筑设计，探测设计。

尽管是一个简单的，良定的数学操作，可靠的生成离散offset surface仍然是一个众所周知的困难的科学挑战，即使是简单的输入也会导致复杂的offset surface。这样的复杂造成offset surface在离散输入和输出的背景中被广泛学习。
给定离散性质的现实世界数据，使用离散输入和输出虽然足够可靠。
另外，一个离散的offset近似在一个用户定义的误差上限下，这样就足够了。
## 1.1 Focus and Problem Statement
本文关注的问题是，给定一个surface mesh作为input，生成一个离散的offset以三角网格的形式。
有两个主要的用户定义的参数：offset距离δ和最大法向偏差角度σ_max 。最小边长度和最大八叉树深度提供由用户额外控制。
最大法向偏移σ_max 提供一个手段来控制近似误差。
最小边长度和最大八叉树深度限制了mesh的复杂度。
我们的方法关注于生成一个闭合的，组合2-流形的输出surface三角网格，通过一个对输入缺陷（自交、洞）鲁棒的过程。我们同样关注于生成offset mesh有着well-shaped三角形，而不是在接近尖锐特征处有小的角度。
# 2 Method Overview
给定一个input三角形surface meshT和offset半径δ，offset surface S(T,δ)定义为等值面d(x,T)=δ，其中d(x,T)是T的距离场。
如果input是一个闭合的mesh，距离场T可以带符号，且offset由多部份闭合surface组成，我们定义V(T,δ)为S(T,δ)包围的体积。
我们的方法包含两个不同的但互补的步骤，用Figure 1呈现：
### Octree construction and Dual Contouring
该步骤设计来生成一个初始的offset mesh，有着我们想要的拓扑：闭合的，组合2-manifold，大约等同于offset surface（在用户指定的误差下）。
八叉树用几种拓扑准则来构建，以推动改进。
生成mesh的几何依然很粗糙，如没有特别注意尖锐特征，见Section 4。
### Remeshing
该步骤应用一个新奇的remeshing算法，根据offset surface来提高几何精度且同时保持拓扑不变，见Section 5。
# 3 Positioning and Contributions
我们的方法建立在几个关于offset mesh生成、尖锐特征恢复的现有的方法之上。
### Adapted Octrees
初看，PK08的方法和我们的很接近：都是基于相似的2-phase方法。
	第一个phase也是基于八叉树，但八叉树在offset surface周围被refine到最大深度。这有一系列缺点，首先在简单的区域会造成显著的计算冗余，见Section 6.2.1，同时也意味着当八叉树非常大时会被最大深度限制住。此外，用户需要选择一个最大八叉树深度。虽然与近似误差的关系是清楚的，和拓扑误差的关系不是直观的。最终，这样一个dense refinement转化为一个复杂的中间mesh，这减慢了进一步的处理步骤，如remeshing。
Varadhan等人利用拓扑自适应八叉树来计算Minkowski sum。offset需要通过离散球体扫过input mesh，offset的质量取决于离散球体，且计算pairwise-convex Minkowski sum的计算代价昂贵。

我们的第一个主要贡献是八叉树生成，可以在任何时候适当地估量offset，因此可以保证永远不会错过精确的偏移量，到达浮点数点精度。
此外，我们保守的cell-to-triangle距离估计计算比其它方法更轻便，并且不需要input是闭合的或流形，见Figure 3。
我们的拓扑自适应八叉树极大地减少了计算量和输出复杂度，通过呈现更少的区域划分。
在我们的框架中，最大深度参数不像其他方法里面作为一个限制因子，因为最大深度可以设置到一个很大的值，在不达到最大深度时不妨碍运行。

中间的offset mesh由Dual Contouring生成，通过设计生成闭合的mesh。
通过后处理output，我们同样可以保证最终的offset mesh不只是闭合的，各处还是拓扑2-manifold的。
### Remeshing
通过Dual Contouring生成的mesh可以包含低质量或纠结的元素。
我们的第二个主要贡献是降低这个限制并提高现有remeshing算法。更具体地说，我们提出一种新的remeshing算法，为offset surface量身定制，可以生成整体质量好的元素，同时确保偏移网格仍然很好地近似偏移。
remeshing算法和之前的方法在两个地方不同：
	是由法向偏差驱动的，而不是常见的边长度或近似误差。
	顶点移动操作利用二次误差度量QEM，最初设计用于网格提取。
结果的remeshing算法是特征敏感的，且能忠实的恢复离散或凹的折痕和角落。它还可以对具有挑战的几何特征（如薄的隧道）具有高质量重建。
最后，它独立于八叉树，可以作为其它offset mesh生成方法的一个后处理步骤。

基于QEM的remeshing方法先前已经被探索过，例如VCP08。这也是一个根本上不同的remeshing方法：surface被采样过且采样点被重新分组为cluster。从这些clustrer开始，Centroidal Voronoi Diagram通过最小能量项被生成，二次误差被用于重新定位Voronoi cell的中心，有着强烈的作用使得点到特征边或角落。尽管高效，这个方法无法应用在隐式形式的offset surface。如果我们要近似一个mesh的offset，我们无法保证这个mesh不包含任何无法被remesh解决的混乱。另外，结果mesh可能包含非流形边和点。
最终，在Valette等人的工作下，生成用户指定数量顶点数的方法，尽管这是一个surface抽取的重要特征，但生成offset mesh的ouput的复杂度是无法预先知道的。
# 4 Octree Construction and Dual Contouring
八叉树的构建以一个单个cell作为初始化，该cell足够大以包含input mesh和它的offset。
我们利用offset surface S(t_i,δ) 和它包裹的体积 V(t_i,δ) ，对于每个三角面T={t_i}，每个八叉树cell被递归划分直到遇到以下准则：
	1、没有三角形offset V(t_i,δ) 交叉cell。
	2、cell在 V(t_i,δ) offset surface内完全封闭。
	3、offset surface S(T,δ) 拓扑等价于一个cell内的disk。
	4、达到用户指定的最大八叉树深度。
前两个准则设计于停止对与offset没有相交的cell的refinement，它们被重新分组为Intersection criterion，在Section 4.1中。
第三个准则目的是停止与offset相交的cell的refinement，但cell的分割没有增加任何有关的拓扑信息，称为Disk criterion，在Section 4.2。
最后我们可能需要分割cell来包围已知的问题，即Dual Contouring可能生成非流形点和边，该步骤在Section 4.3 中详细介绍。
所有的准则都谨慎地评估，意味着我们可能会过度refine，而不是过早地结束。

对所有cell都检测这些准则可能代价很高昂。我们存储每个八叉树cell的所有三角面使得V(t_i,δ)与cell的限制球相交，类似于PK08。我们的初始cell因此包含所有的input三角形，每次新分割cell只需要找到父cell中与之相关的三角形。
## 4.1 Intersection Criterion
八叉树cell只有当offset surface与它们相交时才关注。然而，检测一个cell与所有三角形offset的相交是消耗巨大的。
我们加速这个检测，通过近似cell为它的闭合球，来检测三角形t_i 的offset是否与球体相交，我们找到三角形上距离cell中心点p_c 的最近点(p_i ) ̃，然后计算该点的距离d_i=||(p_i ) ̃−p_c ||。半径为r的球体与三角面的offset相交在d_i−r<|δ|<d_i+r。与球体没有相交的三角形从cell的三角形list中去除。

当cell被分割，一些cell会完全被三角形的offset包围在内，d_i+r<|δ|，这种情况下，整个三角形list被丢弃，cell从分割的队列中移除。

到此为止，intersection criterion只考虑无符号offset半径，当input mesh是闭合的，我们使用带符号的offset，通过检测cell对于任何基元是否包含正确的offset符号，d_(s,i)−r<δ<d_(s,i)+r。我们利用带符号的距离d_(s,i)，符号表示中心点p_c 在闭合mesh的bounded side。符号由CGAL中Side_of_triangel_mesh得来。一个负的offset半径对应于向内偏移，见Figure 4。一个cell如果没有三角形t_i 满足上式不再继续分割，它不包含与offset的相交。
## 4.2 Disk Criterion
大多数区域offset的拓扑结构很简单，但在一些多个offset片接触的区域会很复杂。捕捉拓扑然后需要一个高质量的八叉树refinement。
我们希望保持实际offset surface的拓扑，我们应用一个准则来捕获拓扑特征：一个cell中的偏移面如果拓扑等价于一个disk，则不再分割。
为了评价这个预想，我们对每个投影点(p_i ) ̃构建一个半径为δ的球体，如果球集有一个非空的交叉，为三角形offset surface⋃_(t_i∈T)▒〖V(t_i,δ)〗 构成的star域，三角形的offset经常是凸的。所有的star域都是简单连接的，因此genus 0。由此得出任何闭合流形的subset都有disk的拓扑。

如果有一个点到所有的基元的距离小于offset，则存在一个非空的交叉，通过构造包含所有投影点的最小球体来找到这样的一个点。
如果球体的半径小于offset，则交叉存在。
我们同时提出一个更便宜的检测，通过检测是否所有三角形共享一个顶点，可以保证非空交叉的存在。

我们需要确保以上genus 0的物体可以被Dual Contouring构造。因此，如果所有cell角落都落在offset的同一侧，cell一定要被进一步分割。

虽然我们可以保证三角形offset surface的并集构成一个star域，表明在一个闭合的流形中cell和star域的交集，更加复杂。需要证明所有三角形发offset surface和cell的交集都是非空的。
在折痕朝向角度很小的情况下，由于第二部分的disk criterion不再满足，Dual Contouring倾向于区构建错误的拓扑。我们解决这个问题，通过划分一个cell，如果cell中的offset法向变化大于120度。
然而，当offset包含非常尖锐的折痕时，我们不能保证我们的offset mesh拓扑正确性，它可能包含小的洞和不相连的岛屿。

我们的disk criterion与VKSM04中的subdivision criteria相关，但使用方式和评价有很大的不同。
## 4.3 Subdivision for Manifoldness
一个已知的Dual Contouring的陷阱是可能创建非流形的边和顶点。如果检测到此类情况发生，我们对cell应用一个划分步骤。
如果这样一个划分步骤是禁止的（因为达到用户指定的最大八叉树深度），非流形通过复制顶点解决后仍然是非流形或者邻接的边是非流形的，或者更换Dual Contouring为SJW07中提出的Manifold Dual Contouring也是可行的。
# 5 Offset-Aware Remeshing
我们使用常见的用于surface remeshing的mesh操作：边分裂、边坍缩、边翻转、顶点重定位。
不像其它remeshing处理中根据边长度驱动，我们利用offset mesh三角形和准确的offset的法向偏差。
### Definition 5.1
一个三角形t（offset mesh的三角化上的？）的法向偏差σ(t)是三角形法向n_t 和该三角形对应的offset area中的offset normal n_o 的最大角度：σ(t)=max∠(n_t,n_o (x))，其中x表示三角形的offset area中的任何位置。

法向偏差可以理解为offset曲率的一种度量，但优点是法向偏差在凹折痕处仍然是良定的，而offset曲率是不连续的。因此，可以用于各向同性refine光滑的区域，隐式地适应曲率元素，并检测凹折痕。

每个操作完成remeshing中的一个不同的任务。
	边分裂增加法向偏差过大区域的顶点数量，更多的顶点数更灵活地导向适应mesh为我们想要的几何。
	边坍缩作相反的事情，在低法向偏移区域降低mesh复杂度，如平面区域。
	边翻转提高元素质量，并在凹折痕处将边折断。
顶点重定位使用QEM最小化法向偏差。
## 5.1 Computing Normal Deviation
计算实际法向偏差是很耗时的，因为它需要使用搜索局部映射的最优化。距离场只是C0连续的，最优化问题是有挑战性的。
反而，我们均匀采样offset mesh的三角面，并将采样点投影到输入mesh上。
对每一个采样点，offset normal n_t 之后被构造为从投影点(p_s ) ̃指向采样点p_s 的单位向量。
一个三角形的最大法向偏差计算为，取三角形法向和所有三角形内的采样点的offset normal，的最大角度。
我们使用CGAL中的AABB tree数据结构来投影采样点到input mesh。
## 5.2 Edge Split
我们执行一系列边分裂操作。
要进行分裂的候选边list，通过搜索三角形其σ(t)>σ_max，其中σ_max 是一个用户指定的最大法向偏差，这样的三角形的最长边被考虑为候选边。
边分裂操作中插入的新的顶点的位置，位于顶点重定位步骤中，见Section 5.5。
注意到我们只呈现了单个批次的边分裂操作，没有考虑新生成的边作为候选边递归分裂。边分裂操作并不保证法向偏移的减少，但在mesh上增加更多的点，增加更多自由度用于后续顶点重定位。
我们的方法包含一个可选参数，所以用户可以指定一个最小长度，如果一个边已经比最小边还要小，这条边不会被refine。这个选项避免了处理用户不感兴趣的小特征。
## 5.3 Edge Collapse
在包含太多顶点的区域，我们提出一系列半边坍缩操作来降低mesh复杂度。
当一个顶点的所有的邻接面的法向偏差都小于用户指定最大偏差时，该顶点被坍缩为另一个。
我们的remeshing操作设计在网格是各向同性的假设下。因此，我们必须在元素尺寸下避开快速的变化。如果我们坍缩所有的边仅通过法向偏差，平面区域会被完全坍缩，且我们的各向同性假设也不再保持。我们避免这样，通过仅坍缩边在凸offset区域上小于2倍最大边长度的，l_max=2δsin⁡(δ_max)
## 5.4 Edge Flip
边翻转操作用于帮助well-shaped（各向同性）三角形。
因为该步骤不应该干扰法向偏差的优化，我们介绍以下对一个三角形的度量：
	m_q=(2√3 A)/(l_1^2+l_2^2+l_3^2 )，常见为mean-ratio metric，常用于mesh优化
	m_∠=1−(∠(n_t,n_o))/(90°)，表示了三角形法向和三角形中心的offset surface法向的正规化角度，范围为[−1,1]。
	m_flip=m_q m_∠^3，结合了两个metric，有更的表现。
	其中A为表示三角形区域的符号，l_i 为三角形邻接的每条边，n_t 为三角形法向，n_o 为三角形中心的offset法向。
最终，一个边当其邻接的两个三角形的最小m_flip 质量提高时，进行边翻转。
在有着σ(t)<σ_max 的三角形进行翻转，会造成法向偏差禁止超过用户定义的最大值。
## 5.5 Vertex Relocation
顶点通过使用一个结合了Laplace光滑和最小二次误差的重定位方法。该方法类似于VCP08。直觉上，我们的顶点重定位操作会在光滑区域提高三角形的形状，并将顶点固定到折痕或角落上。
给定一个顶点v在位置x，首先计算其邻域点的质心：
	x^′=1/(|N(v)|) ∑_(x_n∈N(v))▒x_n 
	N(v)表示顶点位置的邻域顶点的位置，|N(v)|表示邻域数量，该步骤指的是Laplace光滑（Fie88）。
然后计算所有采样点投影到offset s ̂的切平面的权重二次误差。我们使用计算法向偏差时的相同采样，见Section 5.1。在邻接v的每个三角形，切平面根据它所属的三角形区域赋予权重。对每个点(s_i ) ̂，需要offset法向(n_i ) ̂和三角形区域a_i 。
权重二次误差和通过求解以下线性系统来最小化：
	Ax^′′=b
	A=∑_(i=1)^(n_samples)▒〖a_i⋅(n_i ) ̂(n_i ) ̂^T 〗
	b=∑_(i=1)^(n_sample)▒〖a_i⋅(n_i ) ̂^T (s_i ) ̂ 〗
	求解经常是不可能的，因为A不是满秩的，如在平面区域上，所有法向(n_i ) ̂都很相似。我们计算奇异值分解的pseudo-inverse，如Lin00中所提出的：
	A=UΣV^T
	A^+=VΣ^+ U^T
	x^′′=x^′+A^+ (b−Ax′)
	使用pseudo-inverse需要一个初始位置x′，选择之前计算的质心。
这样，顶点会通过二次误差有明显的最小时被重定位。如果二次型最小化时生成一条线或一个平面的最佳点时，我们选择最接近x′的一个点。
# 6 Result
我们评价我们的方法用了广泛的输入mesh和广泛的offset半径。Section 6.1示范了我们的方法在Thingi10k数据集的鲁棒性。Section 6.2提供了一个更深的分析，讨论了参数对质量和运行时间的影响。
我们的方法应用在C++，依赖于库Eigen和CGAL。为了测试时间，该代码在集群上串行执行。
## 6.1 Validation
我们从Thingi10K数据集中的所有模型生成offset，除非另有说明，本文呈现的所有结果都使用以下默认参数。最大二叉树深度10，在解决非流形问题时增加2个额外的等级。remeshing操作进行10步，使用最大法向偏差σ_max=7°，Figure 6中展示了生成的结果。

我们的方法希望input为non-degenerate三角形，但其基本原理使得可以在点云或有dangling边的mesh上操作。最后运行了9952个mesh，所有生成的offset mesh都是闭合的流形。
除了上述提及的默认参数，我们设置一个相对offset半径为包围盒的最大边的10%和5%。
Figure 7中，为了评估，我们密集地采样offset mesh并计算offset的距离和法向偏移，使用Libigl几何处理库。

我们用ε_m 表示离散误差，度量一个mesh到offset surface的平均距离与给定offset半径的关系。用σ_m 表示平均法向偏差。
对于几乎所有的offset mesh，ε_m 小于0.5%，这意味着对一个mesh进行0.05距离的偏移，实际offset的误差平均在0.00025。所有offset mesh的平均最大偏移误差和offset半径的比例为2.12%和2.49%，分别对应10%和5%偏移。

Dual Contouring不保证对给定隐式surface重建拓扑正确，这是一个对于基于结构体素离散化方法的根本性的问题，没有完全解决。我们的情况中，这意味着拓扑在角度太小无法被八叉树解决的凹折痕处可能会不正确。
由于remeshing期望给的offset mesh有正确的拓扑，这可能会造成在这样的区域自交。
离散误差很小，甚至在自交的区域，因为自交区域会被推向凹折痕处。
自交的出现可以有效地通过CGAL函数remove_self_intersections来减少，且不会影响到σ_m 和ε_m.
例子见Figure 8。

我们还测量了Thingi10k的运行时间，见Figure 9。
对于10%偏移，我们观察到输入复杂度和运行时间的关系对于拓扑自适应Dual Contouring。
相反，remeshing不展示任何关联，甚至对于输出复杂度。5%偏移也有类似的表现。
后续会展示运行时间对于小偏移倾向于增加，大距离偏移更像密封feature。
## 6.4 Limitationis
提出的方法保证输出mesh是闭合的和组合2-流形，但不能保证不出现由offset mesh和实际offset拓扑不同导致的自交。这是一个所有基于Dual Contouring或Marching Cubes方法都会出现的问题。
大多数情况下，自交可以通过一个简单的后处理进行修复。可以作为下流应用在未来解决。
用户参数提供一个控制输出mesh复杂度的方法，但Hausdorff误差的严格三界仍然是一个未解决的问题。
remeshing步骤在重建尖锐特征是很有效的，但它的运行时间很难取预测，因为offset surface的特征数量和尺寸很难预先知道。
# 7 Conclusion and Future Work
本文介绍了一种方法来从input三角mesh生成各向同性的offset mesh。
第一步应用Dual Contouring方法在一个自适应八叉树上，仅refine当offset拓扑复杂时。
第二步操作一个新奇的remeshing方法，依据offset来重建出尖锐特征，并在优化元素时降低输出offset mesh和offset mesh的法向偏差。
结果时一个可靠的算法，甚至可以操作有缺陷的input，保证生成闭合和组合2-流形的输出mesh，并保证offset mesh近似误差低和well-shaped三角形。
该remeshing方法同样可以成功应用在其它方法生成的offset mesh。

后续工作，我们计划探索一个不同的生成各向异性mesh，改进complexity-distortion tradeoffs。
我们同样希望探索对齐最表的八叉树数据结构的替代选择，例如非结构的四面体mesh和二分空间划分。
