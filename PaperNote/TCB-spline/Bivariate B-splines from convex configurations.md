> @article{SCHMITT202142,
title = {Bivariate B-splines from convex configurations},
journal = {Journal of Computer and System Sciences},
volume = {120},
pages = {42-61},
year = {2021},
issn = {0022-0000},
doi = {https://doi.org/10.1016/j.jcss.2021.03.002},
url = {https://www.sciencedirect.com/science/article/pii/S002200002100026X},
author = {Dominique Schmitt},
keywords = {B-spline, Simplex spline, Convex pseudo-circles},
abstract = {An order-k univariate spline is a function defined over a set S of at least k+2 real parameters, called knots. Such a spline can be obtained as a linear combination of B-splines, each of them being defined over a subset of k+2 consecutive knots of S, called a configuration of S. In the bivariate setting, knots are pairs of reals and B-splines are defined over configurations of k+3 knots. Using convex pseudo-circles, we define a family of configurations that gives rise to bivariate B-splines that retain the fundamental properties of univariate B-splines. We also give an algorithm to construct such configurations.}
}
# 0 Abstract
一个k阶**univariate spline**是一个定义在集合$S$上的至少$k+2$个参数（称为==knots==）的函数
这样的一个spline可以由B-splines的线性组合得到，每一个B-splines都定义在$S$中一个有$k+2$ consecutive knots的subset上，称为$S$的一个==configuration==
在**bivariate** setting中，knots是实数对，B-splines是定义在$k+3$ knots的configurations上
利用convex pseudo-circles，我们定义一个configuration family，其产生的bivarite B-splines保留了univariate B-splines的基本性质
我们还给出了构造这种configuration的方法
# 1 Introduction
spline是定义在一组knots上的piecewise polynomial function

在==univariate==情况，knots是实数
给定一个n个实数的集合$S$，和一个整数$0\leq k\leq n-2$，一个$S$上的k阶spline可以写为k阶base splines的线性组合，称为B-splines
每个k阶B-spline是一个定义在$S$的$k+2$个consecutive knots的subset上的piecewise k次polynomial（见Figure 1）
![[Pasted image 20240622211208.png]]
在==d-variate==情况，knots是$\mathbb{R}^d$的points
为了定义在这样的knots的集合$S$上的multivariate splines，特别是如果想要表示d-variate surfaces，需要拓展B-spline的定义和 $S$*的consecutive knots*的子集 的概念，也成为$S$的==configurations==

人们提出了不同的推广方法，但Neamtu在2001年观察到，没有一种方法能保留univariate splines的所有基本性质
其中一个性质，称为polynomial reproduction property，说明由k阶B-splines张成的univariate spline space包含所有的k阶多项式，这个性质是spline space具有最有逼近性质所必须的
因此，Neamtu提出了满足这一基本性质的univariate splines的新扩展：
	首先，利用de Boor引入的simplex splines推广B-splines，其中k阶simplex spline定义在一个k+d+1个knots的subset上
	其次，选择的$S$中k+d+1个knots的configuration，是那些存在一个sphere穿过d+1个knots，其它k个knots在sphere内部，且剩余的$S$中的knots在sphere的外部
	这样的configuration称为==Delaunay configurations==

本文，我们提出了一个更一般的方法，来在$d=2$的情况下选择configurations，即用convex pseudo-circles的极大family来替代circles
回想一下，一个convex pseudo-circles是一组最多pairwise intersect两次的convex Jordan curves
我们证明了，如果我们取这样定义的任意configuration family，且是最大的包含，则在这些configurations定义的k阶simplex splines张成一个bivariate spline space，其妈妈组polynomial reproduction性质
我们也证明了这样的一个configuration family与Delaunay configuration family包含相同数量的elements
	即相应的spline space由带下相等的spanning set生成

在2007年，Liu和Snoeyink已经指出Neamtu的推广虽然优雅，但在可以生成的spline类型上是有限制的
他们提出了一种算法方法，来在$d=2$生成更一般的configurations
他们证明了他们的算法有效地构造了$k=3$以内的valid configurations
即使实验结果表明该算法始终有效，但这从而在一般情况下得到证明
然而，在有其算法构造的configurations上定义的simplex splines在应用中显得很有趣，特别是它们可以表示具有sharp features的represent surfaces，并作为isogeometric analysis的基础

本文证明了由Liu和Snoeyink的算法构造的configurations正是这里定义的convex pseudo-circles的最大family，这证明了算法总是有效的