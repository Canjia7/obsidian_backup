![[Pasted image 20240803154941.png]]
> @inproceedings{lien2009simple,
  title={A simple method for computing Minkowski sum boundary in 3D using collision detection},
  author={Lien, Jyh-Ming},
  booktitle={Algorithmic Foundation of Robotics VIII: Selected Contributions of the Eight International Workshop on the Algorithmic Foundations of Robotics},
  pages={401--415},
  year={2009},
  organization={Springer}
}
# 0 Abstract
计算两个polyhedra的Minkowski sum已经被证明是困难的
尽管它在robotics的许多集合问题中起着关键的作用，但据我们所知，目前还没有针对一般polyhedra的3-d Minkowski sum软件可控公众使用
其中一个主要原因是现有方法难以实现
计算Minkowski sum有两种主要方法：divide-and-conquer和convolution
	第一种方法将输入的polyhedra分解成convex pieces，计算一堆convex pairs之间的Minkowski sum，并将所有pairwise Minkowski sums统一起来
	尽管概念上很简单，但这种方法的主要问题包括：
		1. decomposition和pairwise Minkowski sums的大小可能非常大
		2. 鲁棒地计算大量components的并集可能非常棘手
	另一方面，convolving两个polyhedra可以更有效地完成
	所得到的convolution是一个Minkowski sum边界的superset
		对于non-convex输入，需要filtering或trimming，这通常涉及计算：
		1. convolution的排列和其substructures
		2. winding numbers用于安排subdivisions
		这两种计算都难以在3-d环境中实现

本文我们提出了一种新的方法，很简单实现，且可以高效地生成Minkowski sum边界
我们的方法是convolution based，但它避免了3-d排列和winding number
我们的方法的前提是将trimming问题简化为计算2-d排列和collision detection，这在文献中得到了很好的理解
为了保持简单性，我们故意牺牲了精确性
虽然我们的方法在大多数情况下产生精确解，但它不会产生低维边界，例如包含zero volume的边界
我们将我们的方法分类为nearly exact，以区分于exact和approximate方法
# 1 Introduction
# 2 Related Work
# 3 Our Method