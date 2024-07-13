> PG_2024 - paper1327
# 0 Abstract
point cloud fitting是计算几何中的经典问题
本文提出了一种深度学习方法来拟合point cloud的隐式函数
由于fitting error与point cloud segmentation的精度高度相关，因此建立segmenting point cloud为primitive patches的精确的模型至关重要
该方法引入了基于PCBNET，实现了最先进的==SegMiou==（Segmentation Mean Intersection over Union）的分割结果
该网络引入了proximal point的标签，这项点邻接于有着不同primitive labels的点
此外，利用Gaussian和mean curvatures来提高预测proximal point的label的准确率
随后，利用预测的proximal points的encoded features来指导和提高模型的分割性能
对于从PCBNET的输出，采用DBSCAN来进一步提高segmentation精度
最后，利用segmentation结果和一种新的深度隐式函数学习方法（该方法最初是为B-Rep solid拟合而设计的）来拟合point cloud
# 1 Introduction
point cloud fitting涉及用一个连续surface或几何shape来approximating或modeling一组空间中的离散点

本文的重点是直接拟合point cloud，由于在raw point cloud内缺乏structural信息，有着很大的挑战
为了解决，我们首先需要将point cloud分割为patches，然后单独拟合这些patches

将point cloud分割成不同primitive patches（planes，cylinders，cones，spheres，etc.）涉及给point cloud中的每个点分配一个特定的primitive label
传统方法往往是基于局部几何分析，和寻找一些最优解
越来越多的趋势是利用先进的计算机技术，特别是深度学习，来辅助分割过程
	深度学习，这些模型通常基于convolutional neural networks（CNNs）或其它为3D数据量身定制的架构，可以从annotated examples中学习，以区分不同的几何primitive
	在point cloud分割中使用深度学习不仅加快了逆向工程过程，而且提高了结果的一致性和可靠性
==自动分割可以处理复杂的和有噪声的数据，这通常是人工方法的局限性==

以下，我们提出了一种深度学习方法来实现高精度的分割，有着很高的SegMiou表现，命名为PCBNET
该方法包含三个有效模块
	首先，我们识别proximal points，其是point cloud中的key point
		这些proximal point被用于在深度学习训练过程中作为true label
	然后，我们计算point cloud中的每个point的curvature，来用作深度训练网络的input，帮助预测proximal point
	然后，我们应用DBSCAN来增强分割表现，通过基于密度进一步clustreing points

第一部分用于为proximal points（其很邻接于有着不同primitive labels的points）获得labels
这些points很可能被network不正确分割，因此他们对指导network分割很重要
它们预测特征，通过一个MLP层编码，被集成到network分割模块的primitive feature来增强表现

第二个模块，我们有效地计算了每个点的mean和Gaussian curvatures
	curvature被用于预测proximal points的network的input
我们局部拟合一个一般形式的二次surface到point cloud中每个点的k最近邻（实验中k=20）来获得curvatures
求解这些二次surfaces的系数是一个非线性最小二乘问题，对于大数据集直接求解很耗时
为了解决这个问题，我们采用了【YWLY12】的二阶段方法
	该方法用于实现triangular mesh patches的快速近似拟合，并将其修改为适合point cloud拟合的离散形式

第三个模块，我们通过引入DBSCAN来进一步cluster primitive labels以提高分割的准确性
	该网络以point cloud的位置、法向信息、curvatures为输入，输出proximal points
为了保存model parameters，同样的network之后以point cloud的位置、法向信息为输入，输出primitive features（用于在mean-shift来决定每个点的primitive labels）和primitive types
	primitive features结合了从proximal points编码的features，来提高network的分割表现
然而，在network-predicted primitive features仅使用mean-shift clustering来获得primitive labels可能会错误地将多种primitive patches的points分组为同样type到一个单独的patch
这些primitive patches有着显著的欧氏距离，且【EKS\*96】提出的DBSCAN可以基于point密度高效地聚类
利用这一性质，我们使用DBSCAN来以显著spatial gaps来分离不同primitive patches到离散labels
实验表明，该后处理步骤显著提高了分割性能

最后，为了拟合point cloud，我们引入一个深度隐式函数学习网络，但修改了适合不包含B-Rep信息的raw point clouds的Booalean tree构建方法
...

改进后的方法不依赖于每个surface patch的类型信息，允许直接拟合每个patch，显著地提高了拟合准确率
现有的方法经常难以准确预测一些patches的primitive type，使用不准确的primitive types来拟合不仅会增加拟合误差，且会导致model的shape的不正确表示
因此，该方法在准确性和可靠性方面有很大的优势
# 2 Related Work
primitive patches的分割和拟合已经进行了大量的研究，大致可以分为传统方法和基于学习的方法
### Traditional methods
### Deep learning-based methods
### Surface reconstruction
# 3 Method
## 3.1 Problem Description and Solution Framework
给定点云$P$，其中的每个点$p_i$包含位置和法向信息
我们的目标是用一个隐式函数拟合point cloud model
	我们首先将point cloud $P$分割为几个primitive patch，每个标签为以下primitive types：plane、cylinder、cone、sphere、B-spline
		这些类型允许我们使用传统的参数形式进行最小二乘拟合来获得surface patches
	然而，patches进行交叉不是一个简单的工作，使用不准确预测出的primitive type来拟合会增加拟合误差，并歪曲model的shape
		因此，我们使用NH-Rep来表示我们的拟合后的CAD模型，其中隐式函数通过一个神经网络直接拟合到primitive patch，而不需要每个patch的primitive type信息
过程的拟合误差直接关系到point cloud分割的准确率，因此提出具有准确分割能力的模型至关重要

我们提出了一个精确分割模型PCBNET，我们的解决方案涉及一个2阶段方法，来快速计算拟合points的一个一般二次曲面的系数
	其后续用于在每个点计算Gaussian和mean curvature信息
point cloud $P$的curvature数据之后作为input输入到一个网络模型中，来预测出proximal points
	其有用于指导分割

为了减少模型参数，我们使用相同的网络模型来处理“仅point cloud数据$P$”，来预测primitive features F和types T

proximal points features之后被传递到hidden layer来获得feature $P_{encoded}$
	其之后被添加到primitive features F来进一步指导分割

网格结构见Figure 1
![[Pasted image 20240711144115.png]]
为了获得primitive label，我们首先在primitive features F使用mean-shift clustering来生成初始的labels
然后通过应用DBSCAN来进一步cluster point cloud中共享相同primitive label的points，以增强分割表现
## 3.2 Proximal Points Acquisition
point cloud对象中，经常有着多个primitive patches，每个都与不同的primitive labels相关联
邻接于其它有着不同labels的points被称为==proximal points==

正确识别并利用它们的信息在分割过程中有着重要作用
本研究中，使用网络来预测这些points并encoding它们的信息到primitive feature F，为分割提供了有价值的指导
## 3.3 Curvature Acquisition
在edge上的proximal points，其curvature通常经历显著变化
包含curvature数据在网络中输入，对于训练模型来更准确识别这些proximal points有着重要意义
求解一个一般二次surface拟合于point cloud是一个最小二乘问题，然而求解这个问题通常非常耗时
为了解决这个问题，我们采用[[Variational mesh segmentation via quadric surface fitting]]描述的2阶段方法，并修改为适合我们应用的离散形式
	这种方法可以更有效地计算curvature，Algorithm 1给出了具体的方法
![[Pasted image 20240711151711.png]]
## 3.4 Application of DBSCAN for Post-Processing
实验表明，虽然深度学习神经网络具有显著的分割能力，但它们偶尔会将相同类型的不同primitive patch错误地分类为一个单独的patch
	这些错误分类的patch在空间上是分开的，但在它们自己的边界上表现出很强的continuity
DBSCAN是一个density-based clustering算法，通过识别density-connected point的每个最大set将point set聚类
	它可以将附近的点分组到同一个cluster中，并将远处的点分组到不同clusters
	即使在有噪声的数据上也能表现良好

在网络预测primitive features上使用了mean-shift clustering，使用DBSCAN可以有效地解决网络将错误分类patches为同一个的问题
## 3.5 NH-Rep for segmented point clouds
通过上述过程，我们得到了分割point cloud中每个点的primitive label
我们首先使用这个label信息来构建一个Boolean tree，其连接了每个patch的隐式函数来表示CAD模型
然后使用【GLPG22】的深度学习方法来对每个patch直接拟合函数$f_1,f_2,...,f_K$（$K$是预测的primitive patches的数量）
## 3.6 Loss Function
# 4 Experiments
## 4.1 Experimental Setup
## 4.2 Comparison
比较了[[Parsenet：A parametric surface fitting network for 3d point clouds]]、[[HPNet：Deep Primitive Segmentation Using Hybrid Representations]]、[[Surface and Edge Detection for Primitive Fitting of Point Clouds]]
## 4.3 Ablation Study
## 4.4 Results Visualization
# 5 Conclusion and Future work
经过实验，我们观察到primitive features的收敛速度相比于primitive types慢
这种差异导致当primitive features的训练误差接近最优时，primitive types已经表的过拟合了
这种不匹配导致TypeMiou的表现略有下降，未来的研究将集中在同步primitive features和types的收敛过程
此外，我们的目标是利用primitive features的信息来进一步指导primitive types的预测