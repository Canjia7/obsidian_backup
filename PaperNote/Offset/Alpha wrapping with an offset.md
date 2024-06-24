![[Pasted image 20240623161306.png]]
> @article{10.1145/3528223.3530152,
author = {Portaneri, C\'{e}dric and Rouxel-Labb\'{e}, Mael and Hemmer, Michael and Cohen-Steiner, David and Alliez, Pierre},
title = {Alpha wrapping with an offset},
year = {2022},
issue_date = {July 2022},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {41},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/3528223.3530152},
doi = {10.1145/3528223.3530152},
abstract = {Given an input 3D geometry such as a triangle soup or a point set, we address the problem of generating a watertight and orientable surface triangle mesh that strictly encloses the input. The output mesh is obtained by greedily refining and carving a 3D Delaunay triangulation on an offset surface of the input, while carving with empty balls of radius alpha. The proposed algorithm is controlled via two user-defined parameters: alpha and offset. Alpha controls the size of cavities or holes that cannot be traversed during carving, while offset controls the distance between the vertices of the output mesh and the input. Our algorithm is guaranteed to terminate and to yield a valid and strictly enclosing mesh, even for defect-laden inputs. Genericity is achieved using an abstract interface probing the input, enabling any geometry to be used, provided a few basic geometric queries can be answered. We benchmark the algorithm on large public datasets such as Thingi10k, and compare it to state-of-the-art approaches in terms of robustness, approximation, output complexity, speed, and peak memory consumption. Our implementation is available through the CGAL library.},
journal = {ACM Trans. Graph.},
month = {jul},
articleno = {127},
numpages = {22},
keywords = {wrapping, watertight mesh generation, strictly enclosing, steiner points, outer approximation, offset, delaunay-based meshing, carving, alpha-balls, 3D delaunay triangulation}
}
# 0 Abstract
给定triangle soup或point set，我们的问题希望生成一个watertight且orientable的surface三角网格，该三角网格尽可能贴近输入点。
得到该输出网格的方法为对输入数据的offset surface的3D Delaunay划分进行改进或雕刻。
提出的算法控制两个参数：alpha、offset。
	alpha参数用于控制雕刻过程中不能穿过的空腔或空洞的大小。
	offset 参数用于控制output网格上的点到input的距离
该算法希望生成一个有效且严格闭合的网格，即使input充满缺陷。
# 1Introduction
常见网格生成算法的结果依赖于input mesh的有效性：watertight，combinatorially 2-manifold，orientable
输入的mesh可能有的问题：重复、退化的、有洞或缝隙、自交、非流形、定向不一致。
这些问题会复合成严重的问题如：生成模型边界表示不一致，严重阻碍后续操作。
### Repair
很难找到可靠的repair方法，因为很多问题都是ill-posed的：如可能有很多种方式来补洞或去除自交，无法具体确定哪一种最好。
主要有两种repair方式：
	Local repair：仅修改缺陷处的input
		用于已经生成高细节质量的网格上有局部的缺陷
		比global repair快
		一般只用于解决单个缺陷问题
		有本质缺陷：当这类算法尝试解决ill-posed问题时很难达到unconditional robustness，且实际问题中模型常有多种缺陷
	Global repair：重新生成一个新的没有缺陷的网格（近似于input）
		缺点：对没必要进行缺陷修复的区域会造成额外cost
由于global方法常用同一种原理，也称为approximation方法
### Approximation
实际应用中输入的mesh常有许多问题。
原始输入数据甚至不是一个mesh，也可能是一些soup（三角形、点、线的分划）
但是不需要考虑这些，只需要最终生成一个近似于原始input数据的approximation即可
此处的approximation为一种方法，能够修复缺陷、多余细节、内部拓扑结构。从而生成一个合理的，watertight的mesh
本文重点为这样的approximation方法，且有可靠性和鲁棒性
### Enclosing
我们还希望生成的output能尽可能enclose input
考虑一些性质，需要丢弃input中的unreachable cavities和内部结构
生成一个outer approximation
# 2 Contributions
输入input（一个3d几何soup），生成一个watertight，orientable的surface三角网格，该网格enclose于input
该算法提供以下性质：
	能够终止，且严格enclose input
	对缺陷无条件鲁棒
	通过两个参数调整
	通过探测input，避免处理多余的内部结构
	与先进方法相比，更可靠、鲁棒，近似
	通过abstract interface探测input来实现 Implementation agnostic
# 3 Related Work
在现实世界中，现在许多应用中的mesh都有很高频率出现缺陷，如没有保证流形，包含自交或退化。
因此对于mesh的repair和approximation是现在的热门课题。
## 3.1 Alpha Shape
点集的Alpha shape是对convex hull概念的一般化。
可以理解为：在空间中不断去除一个半径为alpha的球，且该球不与点集中的点接触到。最终得到点集的轮廓。
选择合适的alpha值，alpha shape的轮廓近似于input的形状。
Alpha shape也不能完全符合我们的需求：
	需要一个完整的输入，会造成很大的负担
	output强烈依赖于input的采样质量，采样稀疏的地方会造成孔洞
重建实体中的体积洞也可能被雕刻出来（是不需要的），从而增加额外的负担
## 3.2 Sculpting
sculpting技术和alpha shape一样都是在周围的空间进行雕刻，不同的是sculpting只在内部雕刻（洪水填充方式）。
从input点集的凸包的Delaunay三角化开始，以特定的顺序移除边界上的cell，直到终止条件。
介绍了一些sculpting方法

alpha solid概念：混合了alpha shape和sculpting的修复CAD模型的方法
## 3.3 Constrained Polygonalizations
input中预知点之间的连接关系，在处理triangle soup中是很重要的信息，但是先前许多方法都没利用上。
许多方法都是基于Delaunay三角化构建mesh的，则相当于input的面是限制住的。此时输入的input三角面要不是就是三角化的一部分，要不是一些三角形的union。
三角化过程中的四面体可以被标记为inside或outside，以从边界上拓展出一个surface。
介绍一些方法解决这类问题。
这类方法虽然无条件鲁棒，但是不能满足enclose需求，且临时数据可能过大（包含了无关的内部结构信息）
## 3.4 Implicit Surfaces
level-set重建方法，隐函数重建方法。
介绍了一些隐函数重建方法。
这类方法生成的mesh都能保证结果的有效性，基于理论保障。
但是内部结构仍然被重建出来，且output mesh不能满足我们的enclose需求。
大多数这类方法都需要构建底层结构，复杂度与input相关。
## 3.5 Shrink Wrapping
shrink wrapping先构建一个围绕在input周围的初始的watertight mesh，然后不断将其缩小。
初始mesh称为wrapper surface：
	通过用自适应八叉树标记cell为inside或outside，然后从边界上提取出mesh
	介绍了一些label的方法
介绍一些处理问题的方法。
许多shrink方法对最终网格的有效性或enclose性质没有保证
## 3.6 Bounding Volumes
前述方法只有很少能保证算法的近似误差，更少能保证output严格enclose于input。
介绍了一些构建enclosing volume的方法。
一种和我们很像的方法：从一个由规则网格雕刻出的外壳开始，将顶点投影到保证enclosure的offset上
# 4 Method
首先介绍一个2D的情况，以及方法的优势
## 4.1 Overview
方法的核心思想：通过雕刻从input的几何结构中分理出结构。
而不是从input的点的三角划分中雕刻出来（类似于sculpting方法）
我们从一个完全新的粗糙的enclosing mesh开始，不断交错雕刻、细化来构建最终的approximation。

与先前方法类似的是，我们的雕刻操作也是基于3D Delaunay三角化中被标记在inside或outside的四面体cell。
结果得到的也是用于分割inside和outside的Delaunay cell的Delaunay facet
carving雕刻操作：向内修整mesh，通过标记之前被标记为inside的Delaunay cell为outside
refinement细化步骤：当carving操作expose input的内部结构时进行。（当实际上与input相交的四面体cell即将被标记为outside时）
	此时carving操作不进行，而是一个Steiner point插入Delaunay三角化以改进四面体cell。
	Steiner point不是添加在input中，而是直接添加在offset surface上。
	offset surface为一个定义在input上的unsigned distance field的level-set。
	更具体地说，Steiner point计算为：在offset surface的dual Voronoi edge相交，或在delaunay cell的外接圆心投影在offset surface上。

将carving和refinement结合的优势：
	mesh只有在carving操作暴露input的情况下才会进行refinement操作，因此可以避免多余的操作（如考虑复杂的内部结构）
	通过在offset surface上放置点，算法可以保证输出的mesh有watertight和orientable，对于退化的情况也能有保证
	尽管构造一个大offset的approximation不需要太多的mesh元素，用户可以自由选择tightness来控制最终mesh的复杂度。

与其他人相似的方法作比较。其它方法的缺点：...

![[Pasted image 20240623161723.png]]
最左上角为原始的线段。这些线段有很多缺陷：如相交、缺口、非流形特征。
该算法初始化时，将一个松散的bounding box插入到Delaunay三角化中，亮蓝色线为Voronoi edge。三角化后的facet都被标记为inside，即图中灰色的部分。
绿色的线为考虑进行carving的边，称为gate。
绿色的点标记了用于插入三角化的候选点，大的点是下一个要考虑的点。
Steiner point会被插入到offset curve中，Steiner point有两种情况：
	或者为Voronoi edge与offset的交点（蓝色）
	或者为Voronoi vertices在offset上的投影（绿色）。
一旦一个Steiner point被插入，则所有产生的facets都被标记为inside。每个新的Steiner point被插入，都会新增carving的自由度。
Carving的发生条件：
	当inside facet邻接当前与input没有相交的gate
	或者当空的Delaunay圆盘的pencil限制gate的最小从尺寸大于alpha
注意refinement操作是如何忽略内部结构的。
最右下角是最终的output，将inside和outside的Delaunay facet分割开，且是一个1-manifold，enclosing于input
## 4.2 Algorithm
接下来进一步介绍该算法，伪码见Algorithm 1。
![[Pasted image 20240623161816.png]]
### Parameters
alpha α：控制最小的carving距离，所以alpha尺寸下的孔洞和峡谷都没有办法在carving过程中穿过。
offset δ：offset surface的距离场的水平集的值，控制最终生成的mesh的点到input的距离，可以理解为包裹的松紧程度。
这些参数都需要是正的。
### Initialization
算法从一个有效且严格enclose于input的粗糙mesh开始。
有效、严格enclose这两个性质在carve和refine操作中一直保持着。
简单的符合上述要求的粗糙mesh即：松散的与坐标轴对齐的包围盒，包围盒上的角落顶点被加入到一个空的3D Delaunay三角化中。
为了只处理四面体，三角化的边界（即相当于凸包）邻接与一个infinite cell，这个cell四个顶点中有一个是infinite vertex。
这样，三角化中的每个三角形facet都只邻接两个四面体cell。
在将包围盒的角落顶点插入后，所有的infinite cell标记为outside，所有的finite cell标记为inside。
此时初始网格的所有三角facet都被分为inside和outside。
### Alpha wrapping
初始化后该算法由外向内内遍历三角化的cell，尽可能carving inside的cell，否则进行refine。
gate：一个facet同时邻接outside和inside的cell。
alpha-traversable：这个facet的最小Delaunay ball半径大于alpha。

gate的遍历顺序：
	将满足alpha-traversable的gate都放在优先队列里，根据gate中facet的最小Delaunay ball半径大小降序排列。
	优先队列的初始化，用凸包的松散的包围盒的alpha-traversable gate。
	这种排序方式类似于：大的元素基于高优先级。
若从outside的cell贯穿到inside的cell时通过了alpha-traversable的gate，这样会暴露input。需要用那两个rule来阻止这种情况。
Rule R1
检查facet的对偶Voronoi edge之间的交点（facet邻接的两个cell的外心连线与offset surface的交点）
如果有一个以上的交点，在从outside指向inside的对偶Voronoi edge上第一个交点，作为Steiner point加入三角划分中。
Rule R2
如果Rule R1为不能应用上，当其相邻cell与input有交叉（相邻的cell未被carve，否则得到的mesh不是enclose于input的）
将该cell的外接圆心与其在input上的投影连接，交offset surface于一点（多个交点的情况下，采用接近于该cell的交点），将该点作为Steiner point加入三角划分中。

在根据这两条rule加入Steiner point后，所有新生成的Delaunay cell都被标记为inside，并检测新的alpha-traversalble gate加入优先队列。
如果两条rule都不能满足，则gate贯穿进去，并令其相邻cell为outside。

当优先队列空时算法停止（可以确保完成），可以通过提取Delaunay三角化中outside和inside的分割facets得到的output三角surface mesh，该mesh满足所有需求，见后续。
## 4.3 Guarantees
### Termination
由于该算法交错执行carving和refinement操作，终止条件并非立即确定，但是肯定会发生。
有两方面：
	Lemma 1说明了Rule规定的插入的Steiner point严格保证在cell的Delaunay sphere中，这样整体最小的Delaunay ball半径会在每次加入点后不断减少。
	Lemma 2中说明了三角化中任意两个顶点的距离是有下界的。offset参数直接出现在下界的表达式中。
	Theorem 1 中说明了offset surface在一个有限区域内可以保证终止。
### Watertight
由定义保证，output mesh是Delaunay三角化中邻接outside和inside的cell的facet。
考虑到对无限cell的讨论，可以认为我们构建的三角化是在Euclidean 3D space中。
如果最终得到的mesh不是closed的，那会存在一条从某个outside到某个inside的道路，该道路不会穿过mesh的facet，这是不可能的。
### Manifoldness
每次carving操作都会移除一整个四面体，因此output mesh的每条edge都会邻接偶数个face，从而确保可定向。
### Enclosure
初始的mesh为松散的input包围盒，严格包含了input。
且craving操作不允许标记一个与inside交叉的cell为outside，每次有这种情况都会触发Rule 2然后cell会被refine。
### Quality Guarantees
Rule 1构建Steiner point类似于常见的严格Delaunay三角化refinement中，surface构建过程中对偶facet的交点。
Steiner point与facet上的顶点是等距的，这样会产生well-shaped的facet。
见Section 5。
## 4.4 Implementation
### Genericity
在算法中用户可以输入任意几何形式的input。
在算法中需要用数据结构做到：
	需要指导一个Delaunay cell四面体是否与input相交，以判断是否能够进行craving操作。
	知道线段与offset surface的交点，以在Rule R1中计算Steiner point。
	将一列点投影到input上，以在Rule R2中计算Steiner point。
算法中，这种抽象数据结构称为oracle。
基础的实现中，oracles可以为点集、线段、三角形soup、三角网格，或混合输入，见Section 7.1。
### Complexity Analysis
算法的每次迭代中：
	a，需要计算一小部分Delaunay三角cell的外接圆圆心和半径。
	b，检查一个四面体与input的交叉情况。
	c，有可能需要计算一个refinement点。
	d，将该点插入三角化中。
a是一个很小的运算量。
d是将一个Steiner point插入3D Delaunay三角化的cell中，这个车cell称为conflict zone，即会被插入的点打破已知的cell。
因此我们不需要去定位Steiner point在三角化中的位置，这会造成对数复杂度。
其余部分中，计算线段与offset surface的交点是消耗最大的。
	没有闭合的表达式，只能二分法逼近。
### Implementation Correctness
为了提高计算速度，实际应用中采用浮点数表示，这会导致表示的不精确，但能保证不会出现严重的错误无效的结果。
# 5 Experiment
实验中两个参数alpha和offset经常表示为模型包围盒的内对角线的比例。
### Validation
成功实现生成mesh有watertight和orientable，且严格包含input。
轮胎模型：
	有许多缺口、洞、高出的平面部分、自交。
	该方法仍然能给出一个有效的结果。
兔子模型：
将模型的每个facet沿重心旋转一个随机的角度。仍然能给出合理的结果。
### Influence of Parameters
alpha和offset控制着输出output的细节、复杂度、质量。
### Level of Detail
alpha控制output中细节的大小，可以理解为三角facet的最大尺寸。
offset控制input到其在output mesh上对应点的距离，也可以理解为包裹的松紧程度。
offset在特征处理上类似于alpha，越小的offset使output mesh越贴紧input，细节保留得越好。
### Complexity
alpha参数用于控制output mesh中facet的大小。
边界上连接outside和inside cell的facet如果最小Delaunay外接圆半径大于alpha，则根据inside的cell是否和input相交来决定craving或refinement。
反之，如果所有边界上的facet的最小Delaunay外接圆半径都大于alpha，则算法会终止。
offset参数还决定output mesh的疏密程度。
但是alpha有时并不能确定output mesh的疏密程度，如在高curvature部分，会进行许多次的refinement操作，见Appendix A。
### Quality
两种插入Steiner point的Rule对生成mesh的质量的影响不同。
由Rule R1生成的Steiner点与被该点refine的facet上的顶点是等距的
当Rule R1负责大部分插入的时候，可以获得比较好的网格质量。
这两条规则对所有的alpha和offset参数所起的作用不同：
	isosurface离input越远，越多新的点由Rule R1插入，同时能获得更好的网格质量。
相反，越小的offset会让更多的四面体与input有交叉，因此会采用Rule R2插入。
### Approximation Error
将mesh的顶点放置在offset surface上，根据定义，这些顶点到input的距离是offset。
这里寻output mesh上facet中任意点到input的距离上下界。
Figure 7中展示了output和input之间L2-范数距离的分布，通过在output网格上均匀采样10000个点。
### Sharp Features
input中所有的凸角在offset后都会变圆。
当offset比较小时，算法倾向于保留凸的尖锐特征。
见Figure 8。
### Two-sided Wrap
对于零体积的input（一层薄面），会生成2\*offset厚的wrap包裹该零体积物体。
此外还能应对open mesh。
见Figure 9。
## 5.3 Performance
基于算法使用的底层结构，在运行时间上没有遇到太大的问题。
目前该算法的瓶颈主要在距离场的evaluation，使用了一些优化技术来提速，具体见Appendix E。
在Figure 10中，模型内部没有办法被alpha半径的球接触的部分都会在算法中被忽略（在底层结构中都不会出现）。
Figure 11和Figure 12展示了计算的速度等。
## 5.4 Comparisons
找了一些能够生成watertight和严格enclose的mesh的方法，已经一些有不同目的需求的方法。
# 6 Applications
## 6.1 Alpha Wrapping for Flow Simulations
工业设计中需要预测和优化object周围的流量flow。
## 6.2 Alpha Wrapping for Efficient Swept Volumes
在运功规划或碰撞检测中，需要描绘出一个object沿着指定轨道运动的空间。
# 7 Extensions
## 7.1 Mixed Inputs
## 7.2 Inside-out Alpha Wraps
我们的方法设计为向内四面体carving，实际使用中也会期待能carving内部的结构以得到一个内部体积。
在知道一个在inside的seed point和至少offset离开input，可以改进该算法以得到向外的alpha wrapping。
见Figure 20。
# 8 Limitations and Future work
### Repair
我们的方法不期望成为repair方法，而是一种能够得到保守的enclose近似网格的包裹-remeshing方法。
我们的方法本意不是用plane去填补大的孔洞，因为则需要选择一个大的alpha值，而这会造成意想不到的特征丢失。
### Immersion versus embedding
该算法生成的是一个immersion的组合流形，而不一定是embedded流形。
### Spatially-varying parameters
alpha和offset是全局且均匀的参数。
在寻找粗网格时期望能有自适应的alpha参数，但是增大alpha参数会与进入小的空腔产生矛盾。
所以自适应计算的alpha只能在大区域的平面上，且没有空腔的地方使用。这个问题是不适定的，特别是在充满缺陷的input中。
### Concave sharp features
输出的mesh在凹特征出比较少贴紧。
这是一个Delaunay mesh中常出现的问题，有许多的解决方案，介绍了一些。
对于我们的方法，会更有难度，因为要：
	检测说所有的凹尖锐特征和转角。
	对折痕进行offset。
	重新对折痕进行alpha密度的采样。
	将采样点插入初始的3D Delaunay三角化中。
对于解决方法还有一些问题：
	检测凹尖锐特征和拐角的问题是不适定的。
	边向内很小的角度，需要用常规的三角化方法。
offset surface上的尖锐特征可能并未在input的几何结构中体现出来。
### MLS
MLS方法对于从一个raw 3D结构中生成一个隐式surface很常用。
可以在我们的方法中加入MLS。
# A Interplay between alpha and offset
alpha和offset的相互影响需要考虑input的局部curvature。
对于凸的情况，element的最大尺寸决定于offset和局部curvature。
Figure 21中，将offset设置很小的值，为包围盒对角线的1/1000。此时即使alpha设置得很大，在圆角的高curvature区域也会触发多余的refinement操作。
Figure 22中展示了2D下的表现。