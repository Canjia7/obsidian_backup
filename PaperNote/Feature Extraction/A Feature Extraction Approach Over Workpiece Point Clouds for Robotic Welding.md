---
tags:
  - point_cloud
  - feature_extraction
  - feature_point
---
> @ARTICLE{10379030,
  author={Zhang, Yuankai and Geng, Yusen and Tian, Xincheng and Zheng, Fuquan and Jiang, Yong and Lai, Min},
  journal={IEEE Transactions on Automation Science and Engineering}, 
  title={A Feature Extraction Approach Over Workpiece Point Clouds for Robotic Welding}, 
  year={2023},
  volume={},
  number={},
  pages={1-10},
  keywords={Feature extraction;Welding;Point cloud compression;Robots;Three-dimensional displays;Programming;Education;Weld seam extraction;point cloud feature extraction;LOBB feature descriptor;K-means},
  doi={10.1109/TASE.2023.3345868}}
# 0 Abstract
welding轨迹的规划阻碍了welding机器人的发展
随着机器视觉的发展，基于3D视觉的welding path planning可以省去机器人的教学和编程工作
然而，目前的weld extraction研究依赖于workpiece的几何信息，导致现有的方法的通用性和鲁棒性较低
point cloud的edge和corner features是提取welds的基础
因此，针对上述问题，本文提出了一种应用于3D视觉welding的workpiece的edge和corner extraction方法
首先，提出LOBB feature descriptor，以构建workpiece point cloud的feature space
然后，增强feature clustering的鲁棒性，对feature description进行非线性激活
最后，设计了一种基于k-means的hierarchical clustering方法，实现了提取crease points、boundary points、corner points
实验表明，该方法不仅可以完整地、低噪声地提取各种workpieces的edges，而且可以有效地识别出corners，这在总体上优于现有的方法
# 1 Introduction
# 2 Related Work
# 3 Algorithm
## A. Local Oriented Bounding Box
对workpiece point cloud的任意点求解以R为半径的==FDN（Fixed Distance Neighbor）==，见Figure 1，并得到一组邻域点$P=\{p_1,p_2,p_3,...,p_k|p_i\in R^3\}$
![[Pasted image 20240717214852.png]]
从图中可以看出，OBB紧密包围着离散点，其是正长方体，可以数学地描述
因此，该研究认为query point的邻域OBB的geometric features可以描述point的neighbor features，其后续会用作feature extraction的基础

本文建立在某一点的FDN上的OBB，成为==LOBB（Local Oriented Bounding Box）==，求解步骤如下：...
![[Pasted image 20240717215654.png]]
## B. LOBB Feature Descriptor
### Edge Feature Description
edge即object的surface的属性突然发生变化的部分
	The edge refers to the part where the properties of the surface of the object are abruptly changed. 
根据属性变化不同，可以将edge point分为crease point和boundary point
	crease point是反映物体形状变化的点
	boundary point是point cloud在某一方向上不再连续的最远点

Figure 3建立LOBB的点云中不同类型点的示意图
	face point和boundary point的LOBB比crease point的LOBB更平坦，这个特征可以分离出crease point
	可以看到，boundary point位于edge的neighbor的边缘
![[Pasted image 20240717220018.png]]
为了量化上述特征，我们提出了flatness $f$和center offset $d$
![[Pasted image 20240717220350.png]]
![[Pasted image 20240717220315.png]]
### Corner Feature Description
corner points是edge line相交的点
在提取完edge line point cloud，对所有点分别建立LOBB
Figure 5展示了黄色为一个line point，红色为一个corner point
	我们发现line point呈现带状分布，因此LOBB的长度远大于高度和宽度，但LOBB在corner point的长度、宽度、高度相对均匀
![[Pasted image 20240717220644.png]]
## C. Feature Extraction Based on K-Means
workpiece point cloud根据center offset被分为non-boundary point、boundary point，根据flatness被分为face points、crease points
edge point cloud根据corner entropy被分为corner point、line point
以上分析表明，本文的所有的feature clustering问题都是1维数据的二分类问题
这样的问题，k-means具有较高的聚类效率

首先，为了优化clustering效果，对feature进行非线性激活
然后基于k-means提出一个分层聚类算法，一实现point cloud segmentation
### Non-Linear Activation
由于点云质量和workpiece surface curvature不均匀，在feature计算过程中会有许多中间值位于分类边界处
	这会影响k-means聚类效果
为了优化聚类效果，我们引入非线性激活函数来激活边界特征
经过实验，选择tanh函数
	具有双边性质，输入feature能够被很好地极化
![[Pasted image 20240717222021.png]]
对feature的激活过程：一个单位化的feature vector $T$，将其线性映射到$[-2,2]$得到$T_1$，带入上式得到$T_2$，再单位化为$T'$

例子见Figure 6，其为一个point cloud
	我们对workpiece point cloud计算LOBB feature descriptor，并对得到的feature vector单位化
![[Pasted image 20240717222549.png]]
Figure 7，上面的曲线为原始feature分布，结果见Figure 8a
	可以看到许多face points被误识别为crease points
下面的曲线为经过非线性激活后的feature分布
![[Pasted image 20240717222315.png]]
![[Pasted image 20240717222849.png]]
### Hierarchical K-Means Clustering
# 4 Experiment
## A. Experimental System
## B. Algorithm Robustness to Parameters
## C. Evaluation of Feature Extraction
### Edge Extraction
### Corner Extraction
## D. Advantage Analysis
## E. Efficiency Analysis
# 5 Conclusion and The Future Work
