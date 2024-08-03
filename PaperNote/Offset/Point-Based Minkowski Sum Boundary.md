---
tags:
  - parallelization
  - Minkowski_sum
  - point_based
---
![[Pasted image 20240803115830.png]]
> @INPROCEEDINGS{4392736,
  author={Lien, Jyh-Ming},
  booktitle={15th Pacific Conference on Computer Graphics and Applications (PG'07)}, 
  title={Point-Based Minkowski Sum Boundary}, 
  year={2007},
  volume={},
  number={},
  pages={261-270},
  keywords={Filters;Application software;Robots;Solid modeling;Virtual prototyping;Robustness;Computer graphics;Computational complexity;Yarn;Surface morphology},
  doi={10.1109/PG.2007.49}}
# 0 Abstract
Minkowski sum是一个许多几何应用中的基本操作
	包括robotics、penetration深度估计、solid modeling、virtual prototyping
然而，由于其高计算复杂性和几个重要的实现问题，计算任意两个polyhedra的精确边界通常是一项困难的任务
这项工作中，我们提出表示Minkowski sum的边界近似地仅使用points
结果表明，这种point-based的表示可以有效地生成
我们的方法的一个重要特点是它的简单实现和并行化
我们还证明了point-based表示的Minkowski sum边界表示确实可以提供与mesh-based表示类似的功能
我们展示了几个应用在motion planning、pentration depth approximation和modeling
# 1 Introduction
$\mathbb{R}^3$中的两个集合$P,Q$的Minkowski sum定义为：
![[Pasted image 20240803133142.png]]
## 1.1 Our Approach
我们的策略是，表示Minkowski sum的边界仅使用points，而不把它们连接成meshes
	由于这种point-based表示，我们比mesh-based表示更有效，更容易实现
	特别是我们的方法不需要convex decomposition，因此不需要merge子问题的result

我们的方法是简单的
给定两个polygons或polyhedra $P,Q$，我们的目标是生成一个点集$S$，使得Minkowski sum $P\oplus Q$的external boundary $\partial(P\oplus Q)$ 被well covered（在Section 3中定义）

为了生成点集$S$，我们从$P,Q$的边界均匀抽取两个点集$S_P,S_Q$
然后将$S_Q$中的每个点替换为$S_P$，记最后得到的points为$S_{P\oplus Q}$
最后，我们过滤掉不再boundary上的（inner）points
	本文后续介绍三个filters，collision detection filter（Section 4）、normal filter（Section 5）、octree filter（Section 6）
Figure 2给出了提出方法的overview
![[Pasted image 20240803130515.png]]

由于提出的方法不依赖于从子问题中获得解，因此我们的方法可以很容易地并行化以处理大型几何问题

在Section 8，我们展示Minkowski sum边界的point-based表示可以提供与mesh-based表示提供类似的功能
## 1.2 Key Contributions
提出了一种计算Minkowski sum边界的算法，结果表示是point based的
我们的方法放弃了精确和连续的表示，但获得了现有方法所没有的优点，包括：
1. 有效率的
2. 鲁棒的（即使是有open boundary的non-manifold models）
3. 容易实现
4. 容易并行
5. multiresolution表示
6. 与mesh-based表示类似的功能
# 2 Related Work
# 3 Point-Based Minkowski Sum Boundary
我们的目标是生成一组点来d-cover两个给定polyhedra的Minkowski sum的边界
	其中的$d$控制了边界的采样密度，一个更小的$d$会产生一个更密集的边界近似
### Definition 3.1
我们称一组点$S$是一个surface $M$的==d-covering==，即对$M$的所有点$m$，在$S$中存在一个点，其到$m$的距离小于$d$

我们的方法由三个主要步骤组成：
1. 首先，我们在input $P,Q$上采样两个点集
2. 其次，我们用Eq.(1)生成point sets的Minkowski sum
3. 然后，我们将boundary points（包括hole和external boundaires）从internal points进行分离
Algorithm 3概述了这一策略
![[Pasted image 20240803133407.png]]
## Sample points
### Theorem 3.2
令$S_P,S_Q$为从polyhedral surface $\partial P,\partial Q$采样的两个d-covering集合，令$S_{P\oplus Q}=S_P\oplus S_Q$，以及$S=S_{P\oplus Q}\cap\partial(P\oplus Q)$，则$S$是$\partial(P\oplus Q)$的一个d-covering
## Compute Minkowski sum
## Extract boundary points
最后，我们将点分成两组：boundary points和inner points
	boundary points将作为最终答案返回，而inner points将被抛弃

我们执行三个filters
1. normal filter
	见Section 5，决定一对sampled points是inner point，通过检测它们的origins（Section 5.1中定义）和orientations
2. octree filter
	见Section 6，构造一个octree，其允许我们只探索boundary附近的点，避免确定inner points
	以上两个filters是有效的，但它们不能单独过滤掉所有的inner points
3. CD filter
	见Section 4，使用collision detecion来从inner points中分离boundary points
	该filter是计算昂贵的，但它提供了一个明确的决定
这三个filters可以组合为Algorithm 3.1中的FILTER
# 4 Collision Detection (CD) Filter
Minkowski sum边界和Robotics中的contact space概念密切相关
	contact space中的每个点表示一个configuration，其将robot与obstacles接触
给定一个平移后的robot $P$和障碍物$Q$
	$P$和$Q$的contact space可以表示为$\partial((-P)\oplus Q)$
	其中$-P=\{-p|p\in P\}$

如果一个点$x$在两个polyhedra $P,Q$的Minkowski sum的边界上，则以下条件为true：
![[Pasted image 20240803135308.png]]
	其中$Q^\circ$是$Q$的open set，$P+x$表示移动$P$到$x$
利用这一观察，CD filter简单地将$-P$放在$S_{P\oplus Q}$的一个点，并检测$-P$是否与$Q$发生碰撞
	如果$-P$没有碰撞，该点被认为是Minkowski sum boundary上的一点
# 5 Normal Filter
# 6 Octree Filter
## 6.1 Extract external boundary
## 6.2 Extract hole boundaries
# 7 Speedup via Parallelization
# 8 Experimental Results and Applications
## 8.1 Experimental Results
## 8.2 Applications
# 9 Conclusion and Discussion
## Limitation and Future work
有两个主要缺点：
1. 尽管证明了我们的方法生成了一个d-covering，但我们可能会生成比需要的更多的点
2. 我们目前的实现中没有任何机制来创建更多的点来增强Minkowski sum和边界上的新sharp features
3. point-based表示可能在以下情况不能使用，比如CAD，其通常需要连续的边界表示