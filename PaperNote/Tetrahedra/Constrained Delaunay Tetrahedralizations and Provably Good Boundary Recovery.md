---
tags:
  - tetrahedralization
  - constrained_Delaunay_tetrahedralization
  - boundary_conformity
  - volumetric_mesh_generation
---
> @article{shewchuk2002constrained,
  title={Constrained Delaunay Tetrahedralizations and Provably Good Boundary Recovery.},
  author={Shewchuk, Jonathan Richard},
  journal={IMR},
  volume={193},
  pages={204},
  year={2002},
  publisher={Citeseer}
}
# 0 Abstract
2D中，CDT尊重一组线段，其约束三角剖分的边，同时仍然保持普通Delaunay三角化的大部分良好性质（如最大化最小角）
CDT解决了强制boundary conformity的问题，确保三角剖分边覆盖被建模区域的边界（包括内部和外部）
本文讨论了在三维模拟、CDT，及其在网格生成中的优势
CDT保持了普通Delaunay四面体化的大部分有利性质，但它们更难以处理，因为一些约束线段和面根本没有CDT
然而，boundary conformity总是可以通过明智地插入额外的顶点来强制，并结合CDT
与其它boundary recovery方法相比，本文方法有三个优势：
	它通常需要比较少的额外添加点
	它在边长度上产生可证明的良好界限（即边不会被不必要地缩短）
	并且它与用于四面体网格生成的可证明的良好Delaunay refinement方法很好地相互作用
# 1Introduction
Delaunay四面体化的优势…

然而，Delaunay四面体化是凸的，用有限元方法建模的域通常不是这样
域有着必须遵守边界网格
	包括绑定网格的外部边界和用于分离具有不同属性区域的内部边界（比如热传导问题中两种金属之间的表面）
	以表示解中已知的不连续点
	或者以建立可以有效应用边界条件的顶点、边、面

对于边界被小二面分隔的域，这个问题最困难，见下图Figure 1
为了恢复一个边界而插入的顶点——这样它就表示Delaunay四面体化的一个三角面并集——很可能会撞到相邻的面
![[Pasted image 20240615101047.png]]
解决这个问题有三个既定方法

	第一个是conforming Delaunay方法，其额外的顶点被插入到mesh中，同时保持四面体的Delaunay性质，直到它符合边界，这意味着每个边界被表示为mesh中三角面的一个并集
	在哪里插入额外的顶点以获得boundary conformity是很困难的
	许多方法在Figure 1的例子中也能工作
	然而，它们的四面体化可能具有非常短的边，从而产生非常小的四面体，并且额外顶点数可能会很大
	这些算法甚至不能生成网格以输入域的多项式复杂度
	没有人知道在一个纯粹的Delaunay网格中如何平衡对高质量element的需求和对boundary conformity的需求
	
	第二个是工程文献中最常见的方法，可能被称为almost Delaunay方法，通过在边界与四面体的面或边相交的相交的地方插入额外的顶点来恢复missing域边界
	然而，在插入顶点时，Delaunay属性并没有被完全维护，所以顶点插入不会将恢复的边界从网格中剔除
	四面体细分方式是引入新的顶点，而不消除表示域边界的现有面和边
	一旦所有边界被恢复，网格生成器可能会尝试使用拓扑翻转来恢复网格的Delaunay属性，但由于翻转不允许干扰域边界，因此网格通常不完全是Delaunay的
	偏差发生在边界上，在那里获得高质量网格元素是很困难的
	一个结果是，好的Delaunay refinement不能保证很好的工作，而almost Delaunay网格生成在许多情况下都能很好地工作，但它们不太可能控制Figure 1中元素的质量
	此外，除非它们是精心设计的，否则也可能产生非常短的边，或者需要插入比需要的更多的顶点
	
	第三种方法使用CDTs
	CDTs完全由CDT组成，它并不总是Delaunay四面体，但仍然保留了Delaunay的两个有利性质：有助于控制插值误差、有助于证明Delaunay refinement算法可靠地生成良好的网格
	CDTs具有比conforming Delaunay四面体所需的更少的顶点的优势，正如本文展示的，直接形成CDTs不会产生不必要的短边

构造CDT需要注意
本文详细介绍了以下方法
	首先，对域的顶点进行Delaunay四面体化，这种四面体化覆盖了域的整个凸包，并且可能不尊重所有域的边界
	其次，识别边界，并确定其中哪些不会出现在四面体化中（Section 4）
	第三，在四面体化中插入额外的顶点（同时保持Delaunay属性），目的是恢复missing域边，放置顶点的算法见Section 5

目前为止，该方法与构造conforming Delaunay四面体化相同，也与一些almost Delaunay方法相同
当所有的域边缘都被恢复时，这些方法完全不同，接下来该恢复missing域面，Figure 2展示了不同
	标准方法插入更多的顶点来恢复missing面
	这里阐述的方法只是用CDT来替换Delaunay四面体，从而恢复missing面，而不是插入额外顶点
	构造CDT的算法在Section 6和7中讨论
![[Pasted image 20240615101336.png]]

这种技术可以产生大量的，甚至asymptotic，节省需要保持域conforming的顶点数
3D CDTs也使得Delaunay refinement算法可以网格化任何PLC，甚至是小角度的域，并在它们产生的四面体边缘和形状有可证明的界限

# 2 Definition: PLCs and CDTs
首先用2D例子说明，conforming和constrained的Delaunay三角剖分都定义在：假设输入是一个planar straight line graph（PSLG）$X$，它是由一组顶点和线段（限制边）组成，见Figure 3上左
三角化是寻找包含$X$的顶点，并遵守$X$的线段
![[Pasted image 20240615101511.png]]
在一个==conforming Delaunay==三角剖分（Figure 3下左），每个单纯形（三角形、边、顶点）是Delaunay的
	一个simplex是Delaunay即存在simplex的一个外接圆——一个圆穿过其所有顶点——不包含任何顶点（尽管圆上允许有任意数量的顶点）
$X$的顶点由额外的顶点（有时称为Steiner点）进行增广，这些额外顶点精心选择，使得增广顶点集的Delaunay三角化遵从所有的线段
	因此，每个线段都是三角剖分的连续线性序列的并集
	【10】展示了任意PSLG都可以通过添加$O(m^2n)$增广点来可三角化，$m$为中线段的数量，$n$为顶点数
	很少有PSLG需要这么多的增广顶点，但是PSLG被认为需要$O(mn)$增广顶点
	缩小$O(m^2n)$和$O(mn)$之间的间隔差距仍然是一个悬而未决的问题

$X$的一个CDT没有不在X中的顶点，且$X$的每一个线段都是CDT的一条边（Figure 3下右）
然而，CDT，尽管这么称呼，并不是Delaunay三角剖分，因为simplex不需要是Delaunay
相反，每个simplex要么是X中指定的线段，或者是constrained Delaunay
	一个simplex称为是constrained Delaunay的，即其有一个外接圆，不包含X中的从simplex的相对内部的任意点visible的顶点
	此外，simplex的相对内部不与任何线段相交
visibility仅被X的线段遮挡

Figure 4演示了例子，一个constrained Delaunay边$e$和一个constrained Delaunay三角形$t$
	$X$中的线段为粗线段
	尽管没有空的圆包围$e$，描绘的$e$的外接圆不包含顶点是从$e$的相对内部visible的
	有两个顶点在圆内，当它们都是hidden在线段后
	因此，$e$是constrained Delaunay的
	类似的，$t$的外接圆包含两个顶点，但它们都是通过线段hidden在$t$的内部
	因此，$t$是constrained Delaunay的
![[Pasted image 20240615102338.png]]
CDT相对于conforming Delaunay的优势在于，其除了X中的顶点没有其它顶点
缺点在于，三角形不是Delaunay的
然而，CDT保留了Delaunay三角剖分的许多理想性质
	例如，一个2D的CDT在三角剖分中，最大化最小角，相比于其它X的constrained三角化

==CDT推广到三维甚至更高维，虽然每个PSLG都有CDT，但是并不是所有多面体都有==
三维boundary recovery的困难在于，如果没有额外的顶点，多面体根本无法四面体化
	【18】提供了Figure 5右边所示的三维例子
	想象这个多面体最简单的方法是从一个三角prism开始
	想象，抓住prism，使得它的两个三角面之一不能移动，而对面的三角形被稍微绕着其中心旋转，而不离开其平面
	结果，三个方形面中的每一个都沿着对角reflex edge（多面体局部非凸的边）被打破成两个三角面
	这样后，每个以前的正方形面的左上角和右下角通过以一个reflex边被分开，并且在多面体中不再彼此visible
	多面体的任意四个顶点包括被reflex边隔开的两个顶点，因此，任何顶点是多面体顶点的四面体都不会完全位于多面体内
因此，如果没有额外的顶点，该多面体不能被四面体化（中间多一个顶点就可以了）
![[Pasted image 20240615102431.png]]
现实情况的三维区域通常复杂很多，因此考虑一种更一般的输入，称为piecewise linear complex（PLC）
一个PLC $X$是一组顶点、边、面，见Figure 6
	每个面都是一个多边形，其中可能有孔洞、缝隙、孤立顶点
	如图所示，一个面可以有任意数量的边，并且可以是非凸的
就像一个线段在三角剖分中施加一维约束，一个面对一个$X$的CDT的四面体剖分$T$施加二维约束，$X$的每个面都必须是$T$中三角形面的并集
![[Pasted image 20240615102519.png]]
==PLCs==有限制，就像其它类型的complex
如果$X$包含面$f$，则$X$必须包含每个f的线段和顶点
一个线段和一个面只能在一个共享顶点相交，除非线段在面的边界上
PLC的任意两个面只能在共享的线段或顶点相交，或者是共享的线段和顶点的并集
	因为面是非凸的，两个面可能在几个地方相交

三角剖分域是用户希望四面体化的空间区域
我们可以默认选择凸包，但有时需要更具体，因为有些PLC存在三角域的CDT，但其凸包的CDT不存在
	例如，可以很容易将夹在Schonhardt多面体和其凸包之间的区域四面体化，即使多面体的内部不是四面体化的

三角剖分域要求是facet-bounded的，也就是说X的面完全覆盖边界，该边界将了三角剖分域从其补集中分离，即exterior域
exterior域包括由三角剖分域包围的任意空腔，还有外部空间
一些面，称为interior面（内切面），可能两边都有三角剖分域
	这些面允许PLC表示非流形和多组件域，例如，作为两种不同材料之间的接口

为了定义三维空间中的CDT是什么，需要更多的定义
==visibility==
	如果说两点$p$,$q$是occluded的，即由一个$X$中的constraining面$f$，使得$p$,$q$在包含f的平面的不同边，且线段$pq$与$f$相交
	如果$p$,$q$都在包含f的平面上，那么$f$不会occlude它们的visibility，见Figure 7
	$X$中的线段不occlude visibility
如果$X$中没有occluding面，$p$,$q$相互visible（等价于，可以看到彼此）
![[Pasted image 20240615102720.png]]
令$s$为任意simplex（多面体、三角形、边、顶点），其顶点在$X$，但其本身不需要在$X$中
令$S$为一个球体，$S$是$s$的外接球，如果$S$穿过$s$的所有顶点
	如果$s$是一个四面体，则$s$有唯一的外接球
	否则，$s$有着无穷多的外接球
$s$是==Delaunay==的，即存在$s$的外接球$S$，不包含$X$的顶点（尽管球体本身允许有任意数量的顶点）
$s$是==strongly Delaunay==的，即存在$s$的外接球$S$，使得没有X的顶点在$S$上（及其内部），除了$s$的顶点
每个顶点都是strongly Delaunay的

Delaunay和strongly Delaunay simplex的区别是：
	如果一个顶点集有着5个甚至更多的顶点在同一个common空球上，顶点集具有多于一个（无约束）Delaunay四面体
	每一个Delaunay simplex至少出现在其中一个四面体中，但一个strongly Delaunay simplex出现在每一个Delaunay四面体中

广义地说，simplex $s$遵从$X$，即没有线段被$s$切成两半，且$s$不从面的一边穿透到另一边
形式上，$s$遵从$X$，如果$s$在三角剖分域，且$s$和$X$中任意线段、面的交叉是$s$中面的并集
	这个并集可能是空集、$s$本身、或者$s$的几个面的融合，可能是混合维度的
例如，一个非凸面可能与三角形的两条或者三条边相交，而不包括三角形的内部

simplex $s$是==constrained Delaunay==的，如果：
	$s$遵从$X$
	存在$s$的外接球$S$，使得S中没有X中的顶点是visible，从s的相对内部的任意点
Figure 8描述了一个constrained Delaunay四面体t
	$t$和面$f$的交叉是$t$的一个面，所以$t$遵从$X$
$t$的圆周包含一个顶点$v$，但$v$从$t$的相对内部的任意点不可见（即使$v$在$t$的边界上的某些点上是可见的）
![[Pasted image 20240615103026.png]]
一个四面体化$T$是X的constrained四面体化，即$T$和$X$有着相同的顶点（不多不少），$T$中的四面体遵从$X$，且$T$中的四面体完全覆盖了三角剖分域
这个定义意味着X中的每一个面都是$T$中三角形面的并集

一个$X$的constrained Delaunay四面体化是一个$X$的constrained四面体化，其中每个四面体是constrained Delaunay的

合并线段是现在CDTs存在的主要困难
Figure 9提供了一个例子
	一个PLC没有CDT，有一个线段贯穿了PLC的凸包的内部
	该PLC只有一个constrained四面体化——由三个与中心部分接壤的四面体组成——并且它的四面体不是constrained Delaunay的，因为它们每个都有一个顶点在它的外接球内
	如果中心部分被移除，PLC有一个由两个四面体组成的CDT
![[Pasted image 20240615103206.png]]
按照惯例，称$T$是$X$的CDT，即称$T$没有除$X$外的顶点
然而，Delaunay网格生成器插入额外的顶点来提高element的质量
此外，尽管一个给定的PLC X可能没有CDT，但是可以将额外的顶点插入到X中来产生一个新的PLC $Y$，其有一个CDT $T$（Section 5解释如何选择额外的点）
$T$不是$X$的CDT，因为其有着$X$缺少的顶点，但$T$是称为$X$的conforming constrained Delaunay四面体化（CCDT），因为其四面体（和面、边）都是constrained Delaunay

CCDT相对于$X$的conforming Delaunay四面体化的一个优点是，插入的额外顶点通常很少
本文的其余部分描述如何构造$X$的CCDT
# 3 Edge Protection
定理使得3D CDTs是有用的，有一个简单的条件可以保证CDT的存在
一个PLC $X$是edge-protected的，即$X$的所有线段都是strongly Delaunay的

==Theorem 1==【20】如果$X$是edge-protected的，即$X$有一个CDT


每个线段都是Delaunay是不够的，如果Schonhardt多面体被指定为其六个顶点都在同一个common球上，则其所有边（和面）都是Delaunay的，但它仍然没有四面体化
不可能在Schonhardt上放置顶点，使得它的所有三个reflex边都是strongly Delaunay的（尽管任意两个可能是）

虽然一个面的边界线段必须是strongly Delaunay的才能保证成立，但这种限制不适用于通过meshing引入的面的边
例如，寻找一个立方体的四面体化
	每个方形面必须用对角线分割成三角形，对角线不是strongly Delaunay的
	如果它们是PLC中的线段，一个constrained四面体化可能不存在（取决于对角线的选择）
如果对角线不是PLC的元素，则由Theorem 1保证CDT的存在性，且一个CDT构造算法可以选择一组兼容的对角线

接下来，一个更强更有用的Theorem 1的版本
一个PLC线段通常作为几个面的边界，这些面可以按照在线段周围的旋转顺序排序
一个线段称为grazeable
	即按旋转顺序的两个连续面被180°或更大的内角分开（见Figure 10）
	或者线段包含在少于两个面中（一个内角正对着三角剖分域的内部，180°或更大的外角不妨碍CDT）
只有grazeble线段需要为strongly Delaunay来保证CDT
如果一个三维PLC $X$是weakly edge-protected，即$X$中的每个grazeable线段都是strongly Delaunay的
如果$X$是weakly edge-protected，则$X$有一个CDT
![[Pasted image 20240615103425.png]]
实践中经常发生线段是不可grazeable的
例如，在一个cubical cube的regular complex中，没有一个线段是grazeable的
较强的结果排除了non-grazeable线段对strongly Delaunay的需求

但如果$X$不是weakly edge-protected的，并且没有一个CDT？
任何PLC都可以通过插入额外的顶点来变成edge-protected的（weakly或fully的），这些顶点将线段分为更小的段，见Section 5
# 4 Testing Edge Protection
检测一个PLC $X$是edge-protected的很直接
用任意标准算法【1，3，28】构建$X$的顶点的Delaunay四面体$T$，$T$覆盖了$X$的整个凸包
如果一个线段$s$从$T$中missing，则该线段$s$不是strongly Delaunay的

如果$s$是$T$的边，则$s$是Delaunay的，但可能不是strongly Delaunay的
推荐的解决方法在构建Delaunay四面体时使用Section 9中的symbolic perturbation技术
perturbation确保了每条边（以及三角形和四面体）是Delaunay的且是strongly Delaunay的
那么如果$s$是$T$的边，则$s$是strongly Delaunay的

如果不使用perturbation技术，则有一种测试，仅通过检测$T$中有边$s$的四面体，就可以判断一条Delauany边$s$是strongly Delaunay的
因为perturbation技术有许多优点，因此省略了细节
# 5 Provably Good Edge Protection
给定三维空间中的一组3D的顶点和线段，应该在哪里插入额外的顶点（将线段分为更小的片段）来确保所有的线段是strongly Delaunay的，以确保所有的线段保证出现在Delaunay四面体化中

这个问题很难，取决于你在新顶点上设置了什么约束
如果你要求添加顶点的数量为输入顶点和线段数的多项式，则该问题无法解决
Edelsbrunner和Tan的$O(m^2 n)$顶点并没有推广到三维，即使它也很复杂

然而，网格生成有不同的需求
PLC要求最多额外顶点，具有线段空间非常紧密或以小角度相交
	然而，这种PLC的高质量网格可能需要一个超多项式数量的顶点

一个更有意义的目标，对于可证明的良好边界recovery确保两个顶点之间的间隔不会太近，相比于形成高质量四面体网格所需的距离还近，例如通过一个小的常数因子
可证明的良好Delaunay refinement算法构建网格，其每条边的长度和该边的local feature size的大小成正比
一个边界recovery算法也应该这样
如果为了精度需要更小的四面体，以后可以很容易对它们进行细化，但是如果边界recovery产生太小的四面体，则可能无法修复它们

对于一个fixed PLC $X$，任意点p在空间中的局部特征大小$lfs(p)$为：以$p$为半径的最小球的半径，该球与X中的不相交的两个线段或顶点相交、
Delaunay refinement算法通常根据这个或类似的$lfs$定义来约束其element的大小
$lfs$是一个连续函数，它在任何地方都是正的，并给出了高质量element可以有多大的粗略上界
它可以在域中变化很大，反映了域几何对element尺寸的影响
函数$lfs$是根据输入PLC定义的，它不会随着新顶点的插入而改变

注意，这个local feature size的定义没有考虑到面
这很好，这意味着面对最终三角剖分中边的长度没有影响（尽管包围面的线段有关）
也许以后，面和顶点之间的小距离可能会迫使Delaunay refinement算法构建更小的边（以消除差质量的element），但这不是边界recovery算法的错

edge protection算法分两步进行，见Figure 11
![[Pasted image 20240615103755.png]]
第一步使用以输入顶点为中心的protecting球来选择插入新顶点的位置
	令$V$为$X$的顶点集合，其中的顶点至少两个线段相交的夹角小于90°
	假设$V$中的每个顶点$v_i$ 是球$S_i$ 的中心，以$r_i $为合适的半径，每个顶点的值可能不同（暂时不考虑如何选取半径）
	对于每个线段s与其它线段相交一个角度小于90°，令$v_i $为相交的顶点
	算法插入一个新顶点在$s∩S_i$，如图所示
	将以这种方式生成的所有新顶点插入到Section 4中构造的Delaunay四面体剖分
		使用【1，28】算法，使得mesh保持Delaunay
	protecting球切断了一些线段的ends
	这些ends保证是strongly Delaunay，因为除了端点外，没有顶点位于直径球体的内部（或在其上）
		直径球体为包含end的最小球体
	因此，所有的ends视为为边来更新Delaunay四面体化
	唯一能刺穿ends的直径球体的是其它ends
	这样在第二步中不将顶点插入任何ends，因此ends仍然保持strongly Delaunay

第二步recover不是ends的线段，通过递归二分
	任何不是strongly Delaunay的线段在其中点插入新的顶点，分为两段
	顶点的Delaunay四面体化始终保持，因此不是strongly Delaunay的子线段可以按照Section 4中的描述进行诊断
	当每个子线段都是strongly Delaunay时，PLC时edge-protected的，算法终止

这个程序可以用于让一个PLC weakly或fully edge-protected
为了获得full edge protection，该程序考虑所有的线段，除了凸包的边缘，它总是strongly Delaunay的
	这意味着必须考虑凸包的true边——位于凸包的面内的边，但不在面的边界
	凸包边可以通过搜索初始四面体化的边界上大于180°的二面角来发现
为了获得weak edge protection，该程序可以只考虑grazeable段，而忽略其它部分，就好像它们不存在一样
weak edge protection的优势是：插入的新顶点更少，但Section 7的增量面插入算法只能构建fully edge-protected PLC的CDT；weak edge protection足以满足Section 6讨论的CDT构造算法

半径$r_i$如何选择，半径应该在满足两个目标的同时，尽可能大
	首先，如果一个线段的both ends都被切除，中间的子线段不能太短
		这个目标是通过选择$r_i$ 不大于$l_i/3$，其中$l_i$ 为被$S_i$切割的最短线段的长度
	其次，ends的直径球体不能包围任何顶点，也不能包围任何非ends的子线段
		这个目标是通过选取$r_i$ 不大于$lfs(v_i)$，不大于$2d_i/3$，其中$d_i$ 是与$v_i$ 邻接的其它end被切割的最短线段的长度
		（注意$d_i$ 有时候比$l_i$ 小，因为一个线段的远end被切割，近end未被切割——因为只有一end邻接另一线段夹角小于90°，计算$d_i$ 而不是$l_i$）
		因此，设置$r_i=min⁡{lfs(v_i,l_i/3,(2d_i)/3)}$


该算法必须显式计算出某些顶点的local feature size
local feature size $lfs(v_i )$简单的就是从$v_i$ 到最近顶点或不与$v_i$ 相交的线段的距离
Delaunay四面体化将每个顶点何其最近邻连接起来（通过一条边），因此初始四面体化有助于快速找到最近的顶点
通过检查PLC的每个线段，可以找到最近线段

==Theorem 2== 为一个PLC进行edge-protecting的算法，创建的线段中没有小于该线段中任意点的local feature size的四分之一
Proof

同样的结果适用于CDT的所有边，除了两个线段相交的角度小于60°
不幸的是，小角度有时会不可避免地产生短边，但边的长度仍然受到角度的限制

==Theorem 3== 令T为一个PLC X由上述edge-protecting算法产生的CCDT，然后构造增广PLC的CDT，令e为T的任意边，令u为e上任意一点，如果e的端点在X的两个不同线段上，相交的角度Φ<60°，则|e|≥lfs(u)sin⁡(Φ/2)，对任意其它边，|e|≥lfs(u)/4
Proof
# 6 Constructing CDTs
构建一个PLC的CDT的第一步是构造PLC的所有面的二维CDTs（三角化）
	这个可以使用Chew的$O(n log⁡n)$算法【2】（其中n为一个面的顶点数），通过使用下面所述的2D版本的gift-wrapping算法
	或者通过构造一个面顶点的Delaunay三角剖分，然后一个接一个插入线段
将每个面CDT的每个三角形称为constraining三角形，因为这些三角形被约束为四面体的面

有两种已知的算法可以构造任意有CDT的PLC $X$的CDT，包括任何weakly edge-protected PLC——如果X中五个顶点都不是共球的（如果后一个条件不满足，见Section 9）
	一个是naive的gift wrapping，和一个sweep算法【23】
	naive的gift wrapping算法更容易实现，但更复杂的sweep算法更快
	gift wrapping算法的运行时间为$O(n_v n_f n_s)$，分别为顶点数、constraining三角形数、CDT中的四面体数
		对于大的PLC，这很慢
	sweep算法的运行在$O(n_v n_s)$，但实践中时间更接近于$O(n_v^2+n_s  log⁡ n_v )$，除了病态的情况外，其它情况更好
		因此sweep算法在大多数情况下都足够快

然而，gift-wrapping算法通常足够快，可以用语增量面插入（在下一节描述），因此这里对其进行描述
它是用语构造凸包和普通Delaunay三角化的gift-wrapping算法的一种变种

gift-wrapping通过选择一个constraining三角形开始，它作为一个seed，在这个seed上constrained Delaunay四面体一个接一个结晶
这个三角形的两边，加上每一个结晶四面体的每个面，被作为一个base，从这个base上搜索邻接四面体的apex

gift-wrapping是基于一个直接的程序，从一个三角形成长为一个四面体，见Figure 12
设$f$为一个constraining三角形或一个constrained Delaunay四面体的一个面
在不失一般性的前提下，假设$f$是水平方向的，并在$f$的正上方寻找constrained Delaunay四面体
假设至少有一个$X$中的顶点在$f$上方
令$S$为一个可以收缩或膨胀的球体，但总是外接于$f$
假设$S$的中心O最初是无限低于$f$的，因此$S$的内部是$f$下面的开半空间
然后$O$向上移动，直到$S$高于$f$的部分接触到第一个从$f$内部任意点（任意fixed点都可以）visible的顶点$v$
![[Pasted image 20240615104442.png]]
令$t$为四面体$conv(f∪v)$
如果$X$的一个CDT存在，则$t$是constrained Delaunay
如果找不到合适的顶点，或者$t$不是constrained Delaunay，则$X$没有CDT

gift-wrapping算法如下
	令$f$为seed三角形，称$f$的两边都是unfinished的
	此后，称growing四面体的一个三角面unfinished的，如果算法还未确定共享该面的第二个constrained Delaunay四面体
finish一个面就是构造第二个四面体，或者确定该面与外域相邻并且不存在第二个四面体

gift-wrapping算法维护一个unfinished面的dictionary（即hash表），初始包含f的两边
重复以下步骤：
	从dictionary中移除一个任意的unfinished面$f$，并搜索一个顶点$v$来finish $f$
	如果没有顶点在$f$上，则$f$在凸包的边界上
	如果$f$不在凸包边界上，并且没有bear a notation表示它与外域邻接，寻找$v$通过growth程序，并添加$t=conv(f∪v)$到growing四面体剖分中
	根据dictionary检查$t$的每个面，除了$f$
		如果一个面已经在dictionary中，那么这个面现在是finished，将它从dictionary中移除
		否则，该面是新的，并且有一个unfinished面，将其插入dictionary

完成一个单面需要$O(n_v n_f)$时间，因为它需要测试面的每个顶点的visibility，这是通过测试$X$中每个constraining三角形的每个顶点
	很明显，visibility测试是gift wrapping CDT很慢的原因
因此，gift wrapping需要$O(n_v n_f n_s)$时间
这留下了很大的改进空间，可以使用sweep算法实现
从实际目的来看，通过gift wrapping的CDT构建可以通过配合增量面插入来加速
# 7 Incremental Facet Insertion
由于naive的gift-wrapping算法很慢，且更快的sweep算法需要很大的努力来实现，并且很难做到鲁棒性，我建议第三种CDT构造算法
该算法首先从一个fully edge-protected的PLC的顶点的Delaunay四面体化开始（回想我们在Section 5中构建了一个edge-protected的PLC和它的四面体化）
然后一个接一个增量恢复缺失的面

在许多实际应用中，增量面插入算法似乎在易于实现和速度中取得了很好的平衡
该算法的另一个优势是不需要预先计算面的二维CDT，这些都是自动产生的
因此，这可能是最容易实现的选项

增量面插入算法的缺点是不适用于weakly edge-protected的PLC，它们必须是fully edge-protected
一个面只有在包围它的所有线段都已经在四面体化中显示为边（包括不grazeable的线段）时，才能被插入
这意味着，与gift-wrapping和sweep算法相比，构建一个CCDT必须插入更多的顶点
	然而，（gift-wrapping和sweep算法得到的网格）由于许多额外的顶点可能会在稍后插入以获得高质量element，因此在最终网格中的差异会很小

思考增量CDT操作（如插入或删除面和顶点）的最好方法，时想象底层的PLC和CDT是同时增量修改的
如果底层PLC在每次操作后总是保留edge-protected，则一个CDT在每步后也存在，且增量操作是可行的

想象一个PLC $X$，它的CDT $T$已经构造好了，令$f$是一个我们想要插入PLC的面（并在四面体化中recover）
包围$f$的线段必须已经存在于X中，并且它们必须作为边出现在$T$中
令$X^f=X∪{f}$
	如果$X$是weakly edge-protected的，则$X^f$也是，因为$X^f$ 和$X$有着完全相同的线段和顶点（面对一个线段是否是strongly Delaunay的没有影响）
	因此，$X^f$ 也有一个CDT $T^f$
面插入算法的目标是见T转化为$T^f$，稍后介绍这个算法
## Incremental facet insertion
增量面插入利用这样一个事实来从一个平凡的Delaunay四面体化中构造一个CDT
为了寻找一个edge-protected PLC $Y$的CDT，令$X$为一个PLC包含了$Y$的所有顶点和线段，但不包含面
	注意算法不需要实际存储$X$，$X$只是我们理解的一个数学结构
令$T$为$X$中顶点的Delaunay四面体化
因为$Y$是fully edge-protected的，所有$X$中的线段都是strongly Delaunay的，因此$T$是$X$的CDT

接下来，插入$Y$的面到$X$中，一个接一个
每次面插入，更新$T$使其仍然为$X$的CDT
关键的观察：因为$X$始终保持edge-protected，更新总是可能的
当所有的面都被插入，$X=Y$，且$T$是$Y$的CDT，或者如果$T$没有覆盖三角化域的整个凸包
最后一步是从$T$中移除任何不在三角化域的四面体

让我们考虑一种recover一个面f的一个算法
$T$怎么变化为$T^f$
	首先，寻找$T$中所有与f的相对内部相交的四面体
	可能$f$已经被表示为三角面的并集，这种情况下不用再做什么了
	否则，下一步是从$T$中删除与$f$相对内部相交的四面体，如Figure 13所示（仅与f在边界上相交的四面体仍保留）
	很容易验证每个其它四面体仍然是constrained Delaunay，因此必须出现在新CDT中
![[Pasted image 20240615104949.png]]
然后，我们使用naïve gift-wrapping算法来重新三角化由$f$的每一边创建的多面体腔
需要警告的是，在$f$的每一边可能有多个多面体腔，因为在$f$插入之前，一些四面体剖分的三角面可能已经conform于$f$

对于大多数域，多面体腔将被少数三角面包围，因此naïve gift wrapping的时间复杂度不太可能成为负担
当然，也有可能设计出空腔有许多面的例子，因此还增量插入算法可能比从头开始gift wrapping整个PLC还要慢
这样的例子在实践中可能是例外

为了重新三角化一个空腔，令$Z$为一个PLC，它只包含包围空腔的三角面，加上它们的边和顶点
令人高兴的是，naïve gift-wrapping算法可以正确地工作，即使域的凸包的一些或全部面都被为指明
因此，当一个多边形腔被三角化后，f可以从空腔的描述中省略
因此，在插入它前不需要预先计算f的2D CDT
当空腔被四面体化，f的2D CDT自动出现在空腔四面体表面上
# 8 Vertex Insertion and Deletion
一旦一个域被constrained Delaunay elements四面体化（共同构成输入PLC的一个CCDT），其中一些element的质量很差，需要改进
Delaunay meshing算法插入额外的顶点来用更好的element替换差的element，同时在整个插入过程中保持Delaunay（或almost Delaunay）属性
幸运的是，从一个constrained Delaunay四面体化中插入或删除一个顶点，几乎和在平凡Delaunay四面体化中一样简单

首先考虑顶点删除
假设我们有一个PLC $X^v$的CDT $T^v$，令X为通过从$X^v$ 中删除一个顶点$v$得到的PLC（假设v不是任何线段的端点，因为端点不能被删除）
我们希望转换$T^v$ 为X的CDT T

第一步是删除有$v$为一个顶点的所有四面体
假设$P$是这些四面体的集合
如Figure 14展示，$P$是一个星形多面体，多面体的点都从$v$ visible，没有其它多面体被删除
![[Pasted image 20240615105141.png]]
多面体通过一个简单的sweep算法重新三角化，以构建星形多面体的CDT，描述于【23】
算法基于一个类似的算法【5】，运行$O(n_s  log⁡ n_v)$，$n_s$ 是$P$的新四面体化的四面体数，$n_v$ 是$P$的顶点数

但这也有危险，如果P没有CDT？如果P是Schonhardt多面体，或其它困难多面体
有时候顶点不能被删除，在删除一个顶点之前，应用程序有责任确保它是安全的

实现这一目标的最佳方法是在每个顶点插入或删除后，确保底层PLC始终保持wealy edge-protected
一个简单的准则是，如果$X^v$ 是weakly edge-protected，则只要$v$不在线段上，$X$始终保持weakly edge-protected，因此具有CDT
然而，如果$v$从线段内部删除，则将删除的两个线段合成一个
只有当新的已知的大线段是strongly Delaunay的，删除是安全的
例如，令$X$为Figure 9中的PLC，令v为中间线段的中间顶点，令$X^v=X∪{v}$，$X^v$ 有一个CDT $T^v$，其中$v$不能被安全删除

顶点插入比顶点删除容易实现，因为不需要sweep算法
插入算法只是一个Bowyer-Watson算法中Delaunay三角化的顶点插入的小修改【1，28】

顶点插入是顶点删除的逆
给定一个PLC $X$的一个CDT T和一个顶点$v$，目标是构建$X^v=X∪{v}$的CDT $T^v$，如果其存在
首要任务是确定$P$，即需要重新三角化的多面体
不幸的是，从$T^v$ 中决定$P$是容易的，从$T$中决定$P$是更困难的

首先，寻找包含$v$的四面体（通常只有一个，但如果$v$在$T$的一个面或边上则有更多）
这些四面体不再constrained Delaunay，因此它们必须被删除
$P$是连通的，因此$P$的其它四面体可以通过深度优先搜索从包含v的四面体中搜索到
	搜索不会遍历constraining三角形，只要遇到一个四面体的外接球不包含v，就往回走
	搜索找到所有不再constrained Delaunay的simplex

在哪个图上执行搜索？
令$G$为一个图，每个四面体有一个节点，一条边连接两个节点，根据四面体共享的三角面不是constraining三角形

$P$的所有四面体都被识别并删除，$P$被重新四面体化以产生$T^v$
新四面体化很容易生成：
	对于$P$边界的每个三角面s，构建一个新四面体$conv(s∪v)$，如Figure 14
总运行时间为$O(n_s)$，其中$n_s$为删除的老四面体的数量（新四面体创建数量不大于$2n_s+2$，尽管可以渐进变小）
	这个运行时间不包括找到包含v的四面体所需的时间

就像顶点删除，顶点插入也不总是安全
一个新顶点可能会造成线段不再是strongly Delaunay的，并将其从四面体化中敲出来
Delaunay mesh生成程序应该检测这种可能性，在顶点不能成功插入时退出

我在其它地方描述一个3D Delaunay refinement算法【21】处理这个困难，通过考虑每个线段的直径球（包含线段的最小球）为protected域
任何插入顶点在球内或球上的努力都会挫败，相反，线段在其中心分裂，并且产生的两个结果子线段有着更小的直径球
因此，每个子线段在任何时候都保持strongly Delaunay，并且CDT的数学完整性从不会被提出

不幸的是，有一种情况这种策略不起作用
一个顶点插入到一个grazeable线段可能会将第二grazeable线段从mesh挤出，第二个线段的顶点可能会将第一个挤出
	这是一个先有鸡还是先有蛋的问题，必须同时插入两个顶点，以保持网格的weakly edge-protected
同时顶点插入可以被执行，通过识别和移除所有可能由单顶点插入而移除的所有四面体，然后用gift-wrapping算法填补空腔
# 9 Degeneracies and Robustness
不幸的是，CDT构造算法比其它大多数几何算法对数值误差更敏感
此外，有着五个或更多顶点共球的PLC在gift wrapping中可能失败，如果退化不能小心处理的话

因此，我建议使用精确算法来实现数值predicate（即orientation和insphere测试）
且所有算法使用一个改进的insphere测试，该测试使用一个简单的symbolic perturbation技术【9，23】来模拟没有五个顶点共球的情况
symbolic perturbation确保gift-wrapping和sweep算法正确地产生任何weakly edge protected PLC的CDT，并简化了strongly Delaunay的线段测试（见Section 4）