![[Pasted image 20240717223722.png]]
> @article{XIN2024103728,
title = {Accurate and complete line segment extraction for large-scale point clouds},
journal = {International Journal of Applied Earth Observation and Geoinformation},
volume = {128},
pages = {103728},
year = {2024},
issn = {1569-8432},
doi = {https://doi.org/10.1016/j.jag.2024.103728},
url = {https://www.sciencedirect.com/science/article/pii/S1569843224000827},
author = {Xiaopeng Xin and Wei Huang and Saishang Zhong and Ming Zhang and Zheng Liu and Zhong Xie},
keywords = {Line segment extraction, Large-scale point clouds, Feature point detection},
abstract = {Line segment extraction from point clouds is critical for analyzing and understanding large-scale scenes. The main challenge is to generate line segments accurately as well as completely. However, state-of-the-art approaches continue to struggle with this issue. In this paper, we propose a novel method for effectively generating line segments from large-scale point clouds. To this end, we design a weighted centroid displacement scheme for identifying comprehensive feature points. Then, we employ an L1-median optimization to refine the identified feature points to perceive geometric edges on the underlying surface accurately. Following that, we generate complete and concise line segments from the refined feature points by designing three geometric operators: clustering, exclusion, and assimilation. The clustering operator generates the initial line segments based on optimized feature points, and the exclusion operator and the assimilation operator ensure the completeness and continuity of these line segments. We evaluate our approach on various scene point clouds, such as TLS, MLS, and ALS data. Extensive experimental results show that our method can outperform the competing approaches in terms of both accuracy and efficiency.}
}
# 0 Abstract
从point cloud中提取line segment对于分析和理解大规模场景至关重要
主要挑战是准确而完整地生成segment
然而，最先进的方法仍在努力解决这个问题

本文提出了一种从大规模点云中有效生成segment的新方法
为此，我们设计了一种weighted centroid displacement方案来识别comprehensive feature points
然后，我们使用L1-median optimization来refine识别的feature points，以准确地感知underlying surface的geometric edges
然后，通过设计三个geometric operators：clustering、exclusing、assimilation，从refine的feature points生成完整且简洁的line segments
	clustering operator基于优化后的feature points来生成初始的line segments
	exclusion operator和assimilation operator确保这些line segments的完整性和连续性

我们在各种场景point clouds上评估了我们的方法
大量实验结果表明，我们的方法在精度和效率方面都优于竞争对手
# 1 Introduction
# 2 Related work
## 2.1 Feature point detection
## 2.2 Line segment generation
# 3 Methodology
## 3.1 Detecting feature points
### 3.1.1 Coarse feature points detection by a weighted centroid displacement scheme
### 3.1.2 Coarse feature point optimization
## 3.2 Line segment extraction
### Clustering operator
### Exclusion operator
### Assimilation operator
# 4 Experiments
## 4.1 Parameter setting
## 4.2 Performance of feature point detection
### Qualitative comparison
### Evaluation of feature detection
## 4.3 Performance of line segment generation
### Qualitative comparison
### Quantitative evaluation
## 4.4 Comparison with deep-learning methods
## 4.5 Generalization tests
# 5 Conclusion