![[Pasted image 20240802201750.png]]
> @article{10.1145/3658159,
author = {Xu, Rui and Liu, Longdu and Wang, Ningna and Chen, Shuangmin and Xin, Shiqing and Guo, Xiaohu and Zhong, Zichun and Komura, Taku and Wang, Wenping and Tu, Changhe},
title = {CWF: Consolidating Weak Features in High-quality Mesh Simplification},
year = {2024},
issue_date = {July 2024},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {43},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/3658159},
doi = {10.1145/3658159},
abstract = {In mesh simplification, common requirements like accuracy, triangle quality, and feature alignment are often considered as a trade-off. Existing algorithms concentrate on just one or a few specific aspects of these requirements. For example, the well-known Quadric Error Metrics (QEM) approach [Garland and Heckbert 1997] prioritizes accuracy and can preserve strong feature lines/points as well, but falls short in ensuring high triangle quality and may degrade weak features that are not as distinctive as strong ones. In this paper, we propose a smooth functional that simultaneously considers all of these requirements. The functional comprises a normal anisotropy term and a Centroidal Voronoi Tessellation (CVT) [Du et al. 1999] energy term, with the variables being a set of movable points lying on the surface. The former inherits the spirit of QEM but operates in a continuous setting, while the latter encourages even point distribution, allowing various surface metrics. We further introduce a decaying weight to automatically balance the two terms. We selected 100 CAD models from the ABC dataset [Koch et al. 2019], along with 21 organic models, to compare the existing mesh simplification algorithms with ours. Experimental results reveal an important observation: the introduction of a decaying weight effectively reduces the conflict between the two terms and enables the alignment of weak features. This distinctive feature sets our approach apart from most existing mesh simplification methods and demonstrates significant potential in shape understanding. Please refer to the teaser figure for illustration.},
journal = {ACM Trans. Graph.},
month = {jul},
articleno = {80},
numpages = {14},
keywords = {mesh simplification, weak feature, centroidal Voronoi tessellation, feature consolidation}
}
# 0 Abstract
网格简化中，常见的需求是：精度、三角形质量、feature alignment，通常被视为一种trade-off
现有的算法仅关注这些需求的一个或几个特定方面
如quadric error metric（QEM）方法，优先考虑准确性，也可以保留strong feature lines/points，但在确保高三角形质量有不足，且可能会degrade weak features（其不像strong feature那么独特）

本文，我们提出一个同时考虑这些要求的smooth functional
该functional包含一个normal anisotropy项和一个Centroidal Voronoi Tessellation（CVT）能量项，变量是位于surface上一组可移动的点
	前者继承了QEM的精神，但在一个continuous setting中运行
	后者鼓励均匀的点分布，允许各种surface metrics
我们进一步引入了decaying weight来自动平衡这两个项

我们选择了ABC dataset中的100个CAD models，与21个organic models一起，将现有的网格简化算法与我们的算法进行比较
实验结果揭示了一个重要的观察结果：decaying weight的引入有效地减少了两个项之间的冲突，使得weak features能够对齐

这种独特的特征使得我们的方法与大多数现有的网格简化算法不同，并在shape understanding方面显示出巨大的潜力