> @article{CAO2019112598,
title = {A finite element framework based on bivariate simplex splines on triangle configurations},
journal = {Computer Methods in Applied Mechanics and Engineering},
volume = {357},
pages = {112598},
year = {2019},
issn = {0045-7825},
doi = {https://doi.org/10.1016/j.cma.2019.112598},
url = {https://www.sciencedirect.com/science/article/pii/S0045782519304748},
author = {Juan Cao and Zhonggui Chen and Xiaodong Wei and Yongjie Jessica Zhang},
keywords = {Simplex spline, TCB-splines, Isogeometric analysis, Triangulation},
abstract = {Recently, triangle configuration based bivariate simplex splines (referred to as TCB-spline) have been introduced to the geometric computing community. TCB-splines retain many attractive theoretic properties of classical B-splines, such as partition of unity, local support, polynomial reproduction and automatic inbuilt high-order smoothness. In this paper, we propose a computational framework for isogeometric analysis using TCB-splines. The centroidal Voronoi tessellation method is used to generate a set of knots that are distributed evenly over the domain. Then, knot subsets are carefully selected by a so-called link triangulation procedure (LTP), on which shape functions are defined in a recursive manner. To achieve high-precision numerical integration, triangle faces served as background integration cells are obtained by triangulating the entire domain restricted to all knot lines, i.e., line segments defined by any two knots in a knot subset. Various numerical examples are carried out to demonstrate the efficiency, flexibility and optimal convergence rates of the proposed method.}
}
# 0 Abstract
# 1 Introduction
# 2 TCB-Splines
TCB-splines是simplex splines的线性组合
这些simplex splines是在所谓triangle configurations上定义的
本节首先介绍simplex splines和triangle configuration，然后介绍在此基础上构造TCB-splines的基函数
关于TCB-splines的详细介绍可以在[[Quadratic and cubic b-splines by generalizing higher-order voronoi diagrams]]发现
## 2.1 Simplex splines
simplex splines是由$\mathbb{R}^2$中的一组点（称为==knots==）定义的piecewise polynomials
给定一组三个non-collinear knots$V=\{t_0,t_1,t_2\}$
由$V$定义的0次simplex spline是由$\{t_0,t_1,t_2\}$构成的triangle的normalized characteristic function
![[Pasted image 20240621161330.png]]
	其中$[...)$定义了一组点的half-open convex hull，它是$\mathbb{R}$中half-open domain的推广

利用Micchelli递归关系，$k$次simplex spline可以递归地表示为$k-1$次simplex splines
给定一组$\mathbb{R}^2$中的knots $V=\{t_0,...,t_{k+2}\}$，以及一个任意选择的knot set $X=\{t_{i_0},t_{i_1},t_{i_2}\}$由$V$中的三个non-collinear knots组成
定义在$V$上的$k$次simplex splines $M(u|V)$的Micchelli递归关系有：
![[Pasted image 20240621162225.png]]
	其中$\lambda_j(u|X)$是$u$关于$X$的重心坐标，满足$\sum_{j=0}^2\lambda_j(u|X)=1$且$\lambda_{j=0}^2\lambda_j(u|X)t_{ij}=u$

simplex splines与B-splines具有许多相同的性质
	例如：在knot set $V$上定义的simplex spline是非负的，并且在$V$中的knots的凸包上有局部支撑

在knot set $V=\{t_0,...,t_{k+2}\}$上定义的的simplex spline是一个k次的piecewise polynomial除以由$V$中的knots引起的partition

knots称为在==general position==，如果没有coalescent knots，并且没有三个knots collinear
如果$V$是在general position，这个simplex spline在所谓的==knot lines==上是$C^{k-1}$-continuous
	knot lines指的是$[t_i,t_j],i,j\in\{0,...,k+2\},i\ne j$这些线段
另一方面，当$V$中出现coalescent knots或collinear knots时，simplex function的continuity order降低
	类似于单变量B-spline情况下的coalescent knots
如果$V$中$s$个knot collinear，则simplex在这个line是$C^{k+1-s}$连续
	特别是，由$k+3$个knots定义的$k$次simplex spline有着$k+2$个collinear knots，当限制在这条line时，退化为由$k+2$个knots定义的单变量B-spline
## 2.2 Link triangulation procedure (LTP)
TCB-spline的构造有两个主要部分：simplex splines和triangle configurations
==triangle configurations==是从给定的用于定义simplex splines的knot set中选择的knot subsets

link triangulation procedure（LTP）是一个递归算法，用于计算triangle configuration的family，使得定义在它们上定义的simplex splines张成的空间保留了单变量B-splines的基本性质

为了方便解释LTP，我们首先介绍几个必要的术语

我们用$\#X$来表示$X$的cardinality

给定一个在general position上的knots set $K\subset\mathbb{R}^2$
一个$k$次的==triangle-configuration（t-config）==是一对knot subsets $(T,I)$，其中$T,I\subset K$， $\#T=3$且$\#I=k$
	其中knot subset $T,I$分别被称为t-config $(T,I)$的first knot subset和second knot subset
我们用$\Gamma_k$来表示一个$k$次==t-config family==
注意，给定knot set $K$的任意triangulation对应于一个0次t-configs family $\Gamma_0$，在每个t-config，first knot subset是由triangle的三个顶点构成的，second knot subset是空的，见Figure 1a
LTP从任意triangulation $\Gamma_0$开始，迭代地计算高阶的t-config families
对于一个k阶的t-config $(T,I)$，first knot subset $T$对应于一个counter-clockwise定向的triangle
我们进一步定义如下术语：
	==v-config==是由一个knot $v\in T$和knot subset $I$的并集所构成的尺寸为$k+1$的一个knot subset，换言之为$\{v\}\cup I$
	（<font color=red>也即，给定一个t-config后，一个knot可以诱导一个v-config</font>）
	因此，一个$k$次的t-config可以生成三个次数为$k+1$的v-config
	例如，一个0次的t-config $T=(\{t_0,t_1,t_2\},\emptyset)$生成三个1次的v-config $\{t_0\}$，$\{t_1\}$，$\{t_2\}$
	==e-config==是一个与knot $v\in T$相对的 有定向的edge（在由$T$构成的定向triangle中），记为$vT$
	（<font color=red>也即，给定一个t-config后，一个knot可以诱导一个e-config</font>）
	一个k次的t-config可以生成三个e-config
	例如，一个1次的t-config $T=(\{t_0,t_1,t_2\},\{t_0\})$生成3个e-config $t_1V=\vec{t_2 t_3}$，$t_2V=\vec{t_3 t_1}$，$t_3V=\vec{t_1 t_2}$，见Figure 1c
	一对相反的e-configs是两个e-configs有着相同的端点，但相反的方向
	==e-v-config pair==是一个由$v\in T$生成的 e-config 和 v-config 的pair，即$(vT, \{v\}\cup I)$
	（<font color=red>也即，给定一个t-config后，一个knot可以诱导一个e-v-config pair</font>）
	一个k次的t-config可以生成三个不同的e-v-pairs
	例如，一个0次的t-config $T=(\{t_0,t_1,t_2\},\emptyset)$生成e-v-pair $(\vec{t_0 t_1},\{t_2\})$，$(\vec{t_1 t_2},\{t_0\})$，$(\vec{t_2 t_0},\{t_1\})$

不同的k次t-config可能有着共享一个共同v-config $J$的e-v-config
	例如，有五个e-v-config pairs $(\vec{t_1 t_2},\{t_0\})$，$(\vec{t_2 t_3},\{t_0\})$，$(\vec{t_3 t_4},\{t_0\})$，$(\vec{t_4 t_5},\{t_0\})$，$(\vec{t_5 t_6},\{t_0\})$，分别五个t-config $(\{t_0,t_1,t_2\},\emptyset)$，$(\{t_0,t_2,t_3\},\emptyset)$，$(\{t_0,t_3,t_4\},\emptyset)$，$(\{t_0,t_4,t_5\},\emptyset)$，$(\{t_0,t_5,t_6\},\emptyset)$诱导
	（<font color=red>也即，对每个t-config，选择顶点v0，该顶点是所有t-config共享的，则诱导出的所有e-v-config pair都共享该顶点</font>）
	这五个e-v-config pairs共享一个共同的v-config $\{t_0\}$，见Figure 1b，其中e-config标记为红色虚线

从共享一个共同的v-config $J$的e-v-config pairs中的e-configs集合称为v-config $J$的==link==，记为：
![[Pasted image 20240621210010.png]]
对于一个link $lk(J,\Gamma_k)$，我们移除其相对的e-config pair（如果其存在的话）
已经被证明了：剩余的e-configs构成一个==simple polygon==（记为$\sum lk(J,\Gamma_k)$），通过连接有定向的edges在coincident source和destination endpoints
特别的，一个v-config $J$的link（该v-config由单个顶点组成）是与这个顶点相对的有定向边的集合，见Figure 1b（<font color=red>红色的带箭头虚线</font>），其中$J=\{t_0\}$，且$\sum lk(\{t_0\},\Gamma_0)$通过counter clockwise连接$t_0$的one-ring邻域构成

我们记一个k次t-config family $\Gamma_k$的所有v-config的集合为$V(\Gamma_k)$

LTP取一个k次t-config family $\Gamma_k$作为input，生成一个k+1次的t-config family $\Gamma_{k+1}$作为output

对每个v-config $J\in V(\Gamma_k)$，我们首先得到由$J$的link构成的polygon $P$
	即$P=\sum lk(J,\Gamma_k)$，见Figure 1b和f分别为$\sum lk(\{t_0\},\Gamma_k)$和$\sum lk(\{t_0,t_5\},\Gamma_k)$的例子
如果$P\ne\emptyset$，则我们triangulate或partition $P$为triangle set $C$，见Figure 1c和f分别为$\sum lk(\{t_0\},\Gamma_k)$和$\sum lk(\{t_0,t_5\},\Gamma_k)$进行triangulation（<font color=red>蓝色虚线</font>）
然后，每个三角形$T\in C$和$J$构成一个k+1次的t-config（见Figure 1c和f），其包含进k+1 t-config family
	即 $\Gamma_{k+1}\leftarrow\Gamma_{k+1}\cup\{(T,J)|T\in C\}$

伪码见Algorithm 1
![[Pasted image 20240621214752.png]]
我们从任意triangulation $\Gamma_0$开始LTP，并重复应用LTP k次以获得k次t-config family $\Gamma_k$
Figure 1展示了LTP达到2次的例子
	第一行展示了从0次t-config生成1次t-config
	第二行展示了从1次t-config生成2次t-config
![[Pasted image 20240621201126.png]]
1. （a）初始的三角化，它对应于一个0阶==t-config== family $(\{t_0,t_1,t_2\},\emptyset)$， $(\{t_0,t_2,t_3\},\emptyset)$， $(\{t_0,t_3,t_4\},\emptyset)$，...
2. （b）1阶的v-config $\{v_0\}\in V(\Gamma_0)$标记为黄色，以及e-v-config pairs $(\vec{t_1 t_2},\{t_0\})$，$(\vec{t_2 t_3},\{t_0\})$，$(\vec{t_3 t_4},\{t_0\})$，$(\vec{t_4 t_5},\{t_0\})$，$(\vec{t_5 t_6},\{t_0\})$，$(\vec{t_6 t_1},\{t_0\})$，分别由$(\{t_0,t_1,t_2\},\emptyset)$， $(\{t_0,t_2,t_3\},\emptyset)$， $(\{t_0,t_3,t_4\},\emptyset)$， $(\{t_0,t_4,t_5\},\emptyset)$， $(\{t_0,t_5,t_6\},\emptyset)$， $(\{t_0,t_6,t_1\},\emptyset)$生成（<font color=red>一个t-config由一个knot可以生成出一个e-config，与v-config合并为一个e-v-pair</font>），其中e-config标记为红色虚线
3. （c）polygon $\sum lk(\{t_0\},\Gamma_0)=[t_1,t_2,t_3,t_4,t_5,t_6]$（填充为灰色），由links $lk(\{t_0\},\Gamma_0)=\{\vec{t_1 t_2},\vec{t_2 t_3},\vec{t_3 t_4},\vec{t_4 t_5},\vec{t_5 t_6},\vec{t_6 t_1}\}$和其triangulation（蓝色虚线）构成，其产生了四个1次的==t-config== $(\{t_1,t_2,t_3\},\{t_0\})$，$(\{t_1,t_3,t_6\},\{t_0\})$，$(\{t_6,t_3,t_4\},\{t_0\})$，$(\{t_6,t_4,t_5\},\{t_0\})$
4. （d）e-v-config pair $(\vec{t_6 t_4},\{t_0,t_5\})$由1阶的t-config $(\{t_6,t_4,t_5\},\{t_0\})$生成（<font color=red>选的knot v=t5，则生成的v-config为{t0，t5}，生成的e-config为vec(t6 t4)</font>），其中e-config标记为红色虚线
5. （e）e-v-config pairs $(\vec{t_8 t_6},\{t_0,t_5\})$，$(\vec{t_7 t_8},\{t_0,t_5\})$，$(\vec{t_4 t_7},\{t_0,t_5\})$共享一个v-config $\{t_0,t_5\}$，由1阶==t-configs== $(\{t_6,t_0,t_8\},\{t_5\})$，$(\{t_8,t_0,t_7\},\{t_5\})$，$(\{t_7,t_0,t_4\},\{t_5\})$分别生成（<font color=red>选择knot v=v0</font>），其中e-configs标记为红色虚线。这三个e-v-config pairs共享一个共同的v-config $\{t_0,t_5\}$，其中e-v-config pair $(\{t_6,t_4,t_5\},\{t_0\})$展示在图d
6. （f）polygon $[t_8,t_6,t_4,t_7]$（填充为灰色）由link $lk(\{t_0,t_5\},\Gamma_1)$和其triangulation（蓝色虚线）构建，生成两个2次的==t-configs== $(\{t_8,t_6,t_7\},\{t_0,t_5\})$，$(\{t_7,t_6,t_4\},\{t_0,t_5\})$
### Remark 2.1
值得指出的是，如果使用不同的triangulation方法来获得初始triangulation $\Gamma_0$，或者划分LTP中出现的polygon，LTP可以为给定的一组knots产生不同的t-config families
作为TCB-splines的一种特殊情况，当Delaunay triangulation用于初始triangulation和polygon划分时，LTP产生称为Delaunay configurations
	结果t-config family由此称为==Delaunay configuration family==，相关的simplex splines称为DCB-splines

LTP的一个适当的triangulation的criteria可能会因为不同的应用而不同
从几何角度出发，针对基于TCB-splines的surface fitting，提出了一种feature sensitive的triangulation方法

在Section 3.2中，我们会提出一种triangulation方法来生成TCB-splines basis function，它们至少是$C^1$-连续的，这样它们就可以作为解高阶PDE方程的基础
## 2.3 Definition of TCB-splines
由k次t-configs family $\Gamma_k$，我们可以为每个k次t-config $(T,I)$ 定义simplex splines $M(u|T\cup I)$，则TCB-spline basis functions为这些simplex splines的线性组合

我们记 $\Gamma_k$ 中所有second knot subsets的集合为 $\mathcal{J}$
给定$I\in\mathcal{J}$，$X_I=\{(T,I^*)|I^*=I,(T,I^*\in\Gamma_k)\}$为共享共有second knot subset $I$的t-configs集合
与一个knot subset $I\in\mathcal{J}$相关联的k次TCB-splines basis function定义为：
![[Pasted image 20240622171358.png]]
$B_I(u)$的Greville site $\xi_I$定义为second knot subset $I$的平均，即$\xi_I=(\sum_{v\in I}v)/k$，这是单变量B-spline情况的推广
Figure 2给出了一阶到三阶的TCB-spline basis function的例子
已经被证明了：TCB-spline basis function构成一个非负的unity划分，具有automatic smoothness，具有local support和reproduce polynomials
![[Pasted image 20240622171856.png]]
### Remark 2.2
TCB-splines可能是线性相关的
虽然在实践中有一种方法可以消除TCB-splines的线性依赖性，但目前还没有保证TCB-splines线性无关的规则，导致分析中可能出现stiffness matrix奇异的情况
处理这个问题有两种方法：
1. 当coefficient matrix具有spurious zero eigenvalues时，我们可以使用conjugate gradients（CG），CG只需要stiffness matrix与向量的乘积，CG首先收敛到较大的特征值，然后按顺序扫过较低的modes，CG不能收敛到0特征值的mode
2. 可以使用LDU分解...
# 3 Computation of TCB-splines
本节中，我们首先介绍如何在2D bounded domain（即PDE的solution domain）上放置knots，为了TCB-splines构造
然后，我们描述LTP来保证在convex region上$C^{k-1}$-continuity和在concave region上$C^{1}$-continuity
其次，我们详细介绍了TCB-splines的积分，这是在IGA中assembling stiffness matrix的关键步骤
最后，我们提出了一种为TCB-splines的error-guided knot放置方法，以实现自适应refinement
## 3.1 Evenly-distributed knot placement
为了定义TCB-spline space，我们首先需要放置一组knots $K$
这里，我们使用centroidal Voronoi tessellations（CVT）来在给定domain内均匀分布的general position生成knots
...
CVT的generators，其在general positions，被用于knots来定义TCB-splines space

CVT方法在interior domain生成均匀间隔的knots，关联的$k(\geq 2)$次TCB-splines是$C^{k-1}$-continuity，这些TCB-splines及其derivatives在domain boundary上消失
为了保证TCB-splines在domain boundary $\partial \Omega$ 的划分的untiy性质，与单变量情况类似，我们在boundary上引入coalescent和collinear knots，并计算TCB-splines如下：
1. 均匀分布的boundary knots和interior knots通过调整CVT结果来生成...
2. 对于k次TCB-splines，在corner的knot被重复$k+1$，即有$k+1$个knots共享在corner的相同位置，这样的knots是==coalescent==
3. 每个corner上的coalescent knots被扰动，使得它们在general positions，这一步提供了一个拓扑结构（即初始triangulation $\Gamma_0$的connectivity）来计算t-config family，下一节中将详细讨论
4. 我们计算t-config family，并构造了相应的simplex splines，并让扰动size消失，即我们只是在计算t-config family中单独对待coalescent knots，在相关的simplex splines的计算中使用无扰动的knots

上述knot放置方法产生了沿boundaries执行等效的单变量B-splines的TCB-splines集合
Figure 3展示了一个3次TCB-spline basis function的例子，在boundaries上有5个collinear knots，其对边界的约束是经典的单变量3次B-spline basis function
![[Pasted image 20240622202255.png]]
Figure 4上左展示了一个定义在coalescent knots（multiplicity为3）的2阶TCB-spline basis function例子，其对边界的约束时multiplicity为3的knots上的单变量2次B-spline basis function
![[Pasted image 20240622202518.png]]
### Remark 3.1
最直接获得均匀间隔的knots的方法是使用一个knots的regular grid
然而，以这种grid构建的knots不适合TCB-splines，因为其包含collinear knots，导致TCB-splines的continuity降低
为了在interior domain中避免collinear knots，我们使用CVT方法来均匀且随机地生成knots
其它方法，例如possion-disk sampling方法，可能也可以在general position生成knots
### Remark 3.2
实际应用中，collinear knots和coalescent knots在parametric domain的内部是不希望出现的，它们会降低simplex splines的continuity order
在我们的框架中，使用CVT来生成给定预先指定密度的均匀间隔的knots，虽然不能保证CVT方法可以避免collinear knots，但在数值上我们的实验从未在parametric domain内部发现这些knots
如果在任何情况下collinear knots出现，我们可以扰动这些knots，或者在局部执行Lloyd的迭代来数值地消除它们
## 3.2 C1-continuity around corners
![[Pasted image 20240625204055.png]]