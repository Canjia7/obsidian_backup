![[Pasted image 20240717105349.png]]
> @article{10.1145/3550454.3555443,
author = {Xu, Rui and Wang, Zixiong and Dou, Zhiyang and Zong, Chen and Xin, Shiqing and Jiang, Mingyan and Ju, Tao and Tu, Changhe},
title = {RFEPS: Reconstructing Feature-Line Equipped Polygonal Surface},
year = {2022},
issue_date = {December 2022},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {41},
number = {6},
issn = {0730-0301},
url = {https://doi.org/10.1145/3550454.3555443},
doi = {10.1145/3550454.3555443},
abstract = {Feature lines are important geometric cues in characterizing the structure of a CAD model. Despite great progress in both explicit reconstruction and implicit reconstruction, it remains a challenging task to reconstruct a polygonal surface equipped with feature lines, especially when the input point cloud is noisy and lacks faithful normal vectors. In this paper, we develop a multistage algorithm, named RFEPS, to address this challenge. The key steps include (1) denoising the point cloud based on the assumption of local planarity, (2) identifying the feature-line zone by optimization of discrete optimal transport, (3) augmenting the point set so that sufficiently many additional points are generated on potential geometry edges, and (4) generating a polygonal surface that interpolates the augmented point set based on restricted power diagram. We demonstrate through extensive experiments that RFEPS, benefiting from the edge-point augmentation and the feature preserving explicit reconstruction, outperforms state of the art methods in terms of the reconstruction quality, especially in terms of the ability to reconstruct missing feature lines.},
journal = {ACM Trans. Graph.},
month = {nov},
articleno = {228},
numpages = {15},
keywords = {computer-aided design, feature line, point cloud, restricted power diagram, surface reconstruction}
}

# 0 Abstract
feature line是描述CAD模型结构特征的重要几何线索。
在显式重建和隐式重建中，重建有feature line的多边形surface是很有挑战的，特别是输入的点云是有噪声且缺乏可靠的法向量时。
本文提出了一种多阶段的算法，称为RFEPS，来应对这一挑战。
关键步骤包括：
	1、对点云根据局部平面性加深进行去噪。
	2、通过离散最优运输来识别feature-line区域。
	3、扩张点集使得足够多的额外点生成在潜在的几何边上。
	4、基于restricted geometry diagram生成一个多边形网格，插值扩张的点集。
我们通过大量的实验，得益于edge-point的扩张和保特征的显式重建方法，重建质量优于最先进的方法，特别是在重建缺失feature line的情况下。
# 1 Introduction
# 2 Related Work
### Feature Preserving Surface Reconstruction
在许多几何处理任务中，尖锐特征需要非常小心地进行处理。
一个典型的情形是目标surface可能由光滑的patch被feature line分开，特别是在CAD邻域中。
由于分段尖锐表示（Du et al.2021），提出一个Boundary-Sampled Halfspace BSH的表示，因此，在重建CAD模型中有必要去保持line-type的特征。
Oztireli et al.2009 提议可以自动检测尖锐特征，通过度量法向量之间的不同。然而，最终的surface仍然是可微的，甚至是光滑的。
Wang et al.2013 给出了一个kernel-based尺寸估计算子用于预测最佳切平面并移除离群部分。
Salman et al.2010 建议使用基于Voronoi cell的协方差矩阵特征检测过程，来提取尖锐特征集，能够促进特征在mesh生成的过程中的保持。
Dey et al.2012 提出基于Gaussian权重图Laplacian和Reeb图来识别和重建尖锐特征曲线，他们算法的重建步骤类似于Cocone reconstruction，不同点在于算法使用了带权重的Delaunay三角化技术，能够允许用ball来保护特征采样。
Digne et al.2014 主张通过最优传输来迭代简化初始3D Delaunay三角化，其中通过使用点集和三角mesh的feature-sensitive metric，尖锐特征和边界能被很好地保持。
Xiong et al. 2014 提出一个框架能够同时优化mesh生成和连通性，从无定向点云，基于dictionary learning。
经验证据展示了保特征重建的工作是很有挑战性的，当点在edge zone是稀疏的时候。
# 3 Method
给定一个CAD模型的有噪声点云，目标是重建一个干净的多边形surface，且有整洁的特征线。
最重要的任务是预测额外点来编码几何上的边。因此有必要先正则化点的位置和法向量，则激励我们提出一个多阶段的算法，将Figure 3。
	Step 1、初始化法向量，过滤掉输入点云的噪声，这两步是同步进行的，见Figure 3b。
	Step 2、确定edge zone的点，通过离散最优传输，见Figure 3c。
	Step 3、改进法向量，基于目标model是局部平面的，见Figure 3d。
	Step 4、微调点的位置，使其适应于正则化正态信息，见Figure 3e。
	Step 5、预测额外点，使其在几何上的潜在边上，见Figure 3f。
	Step 6、在由screened Poisson reconstruction SPR所产生的基本surface上计算restricted power diagram RPD。
	Step 7、提取RPD的对偶，给出恢复丢失的feature line的重建多边形surface，见Figure 3h。
1、2用于寻找edge zone，点位置和法向量不用高精度。
3、4、5用于在几何的边上生成额外点。一个潜在的观察是，一个edge point的足够小的区域可以近似于两个half-plane（是一个feature-line point），或更多的half-plane（是一个feature-tip point）。
6、7聚焦于将扩张后的点集扩张为三角网格，关键是要保证预测edge point的连接关系能够表达出feature line。
![[Pasted image 20240717110139.png]]
## 3.1 Point Cloud Denoising and Normal Initialization
给定的噪声点云P={p_i }_(i=1)^n 进行降噪是一个保证最终重建结果包含主要的simple patch的必要步骤。
点的位置{p_i }_(i=1)^n 和法向量{n_i }_(i=1)^n 是相互影响的，我们共同优化它们。
通过允许一个点p_i 沿着法向n_i 移动，使用p_i^′=p_i+ϵ_i n_i 来表示新的位置。
公式（1）计算M_(3×3)^i 。
其中p_i 的邻域由一个以r为半径的球，默认设置r=2δ，平均gap δ的计算见脚注。
M_(3×3)^i 是一个在点p_i 的协方差矩阵，其特征向量编码了局部surface的方向。
如果n_i 能够反映点p_i 的实际法向信息，则M_(3×3)^i n_i 会接近于null向量。
同时，我们需要控制降噪的degree，启发自PCA（Pearson 1901），必要去优化一个关于{ϵ_i }{n_i}的联合变量.
公式（2）中ξ为一个参数，以平衡噪声的去除和保真度，默认设置为0.1，Figure 3b展示了降噪的结果。
### Optimization details
为了保证n_i 是一个单位向量，参数为为(sin⁡(u)  cos⁡(v),sin⁡(u)  sin⁡(v),cos⁡(u))。
初始时，ϵ_i=0,i=1,2,…,n，通过应用在PCL中的PCA初始化{n_i}。
注意到其将法向的一致性进行了考虑，在Section 5中通过翻转一定量的法向方向来观察算法对法向一致性的敏感性。
### Remark
我们观察到降噪的步骤和检测edge zone的步骤是相互影响的，换言之，不可能在检测edge zone前完全消除数据的噪声。
因此等式（2）让点的分布更好地遵守CAD模型的特征，因此优化了后续步骤的鲁棒性，不单单用于降噪点和法向量。
## 3.2 Edge Zone Identification
### Observation
通常一个CAD模型的surface包含许多光滑且简单的片段。
假设点p_i 位于一个接近平的区域，点p_i 附近的法向量的可能性密度函数（probability density funcion PDF）可以被近似为一个single Dirac delta function乘以一个向量。
如果点p_i 非常接近于一个条边，其PDF可以表示为连两个分开的Dirac delta function的组合。更进一步，这两个子分布有相同的quantities。
如果点p_i 接近于一个角落点，PDF可以被分解成三个或更多的Dirac delta function。
假设点p_i 落在几何边上，见Figure 4，Step 1所得到的法向量会如Figure 4a，它们定义了source的分布。target的分布由两个分开的法向量给出，见Figure 4b。
比起用k-means来显式聚类成两类，我们提出了一种更加精确的方法来描述source分布和target分布的区别。
![[Pasted image 20240717110233.png]]
### Optimal Mass Transport
Optimal Mass Transport OMT目的是寻找一个最低的传输误差，在移动source分布μ_s 到target分布μ_t 中。
其中source domain和target domain由单位球S^2 上的surface给出，见公式（3）。其中f是一个S^2 到S^2 的一一映射，c(x,f(x))描述了从x到f(x)传输一个单位mass的cost。本文中c(⋅,⋅)简单地度量为两个单位法向量的平方误差，见公式（4）。
我们的场景中，由Sec3.1生成的法向量定义source分布μ_s，由两个独立的Dirac delta function定义target分布。
### Formulation
我们考虑点p_i 落在几何边上的情况。
可以观察到在p_i 的邻域，法向量可以被组织成两个相同quantity的聚类，其代表的法向量分别为n_i,n_2 。因此平方传输误差非常接近于0，见公式（4），优化λ_j 的值，其中λ_j∈[0,1]，用于定义n_j 传输到n ̂_1 的比例。但公式（4）的目标函数最小时，λ_j 取值有两种情况，分别为0和1，见公式（5）。
![[Pasted image 20240717110321.png]]
当两个法向量类有接近的quantity时，有公式（6），其中k时点p_i 邻域内点的数量。
这启发我们描述edge zone为公式（7）。
该形式描述了当前法向量分布到perfect几何边（其上的点的法向量为n ̂_1 或n ̂_2，且二者数量相同）的偏差度。为了更好地讲解，见Figure 5，以观察传输cost公式（7）在穿过几何边的时候是如何变化的。
	一方面，可以看到传输cost在平的区域或在边上时非常接近于0。
	另一方面，当点到达边上时邻域内点的n ̂_1 与n ̂_2 之间的角度达到最大，如Figure 5中的点c。
我们提出一个几何上有意义的规则来确定edge zone。
![[Pasted image 20240717110354.png]]
### Three situation
公式（7）被优化后，n ̂_1 与n ̂_2 之间的角度记为Angle(n ̂_1,n ̂_2)，可以用于从edge zone中区分平面区域。
但我们观察到如果p_i 落在薄的tube上，Angle(n ̂_1,n ̂_2)和Cost(p_i)都会很大，因此我们考虑三种情况。
	Situation 1：Angle(n ̂_1,n ̂_2 )≤π/6，将base point p_i 设置为off-edge point。
	Situation 2：Angle(n ̂_1,n ̂_2 )>π/6 且Cost(p_i )≤0.25，将base point p_i 标记为edge-zone。
	Situation 3：Angle(n ̂_1,n ̂_2 )>π/6 且Cost(p_i )>0.25，将p_i 视为off-edge point。
### Implementation Details
值得注意的是，确定edge zone的步骤假定了问题中的法向定向在先前的步骤中被处理为一致的。
初始的n ̂_1,n ̂_2 由k-means的k法向量给出。
参数λ_j 初始化为0.5。
介绍一种公式（7）的自适应加权方法来处理不均匀的点分布，见公式（8），其中ϵ=10^(−4) 用于处理一个消失的分母。
## 3.3 Edge Point Prediction
该步骤用于通过检测edge zone来调整法向量和点的位置，并且生成额外的edge point。
### Regularizing Normal Vectors
在edge point被检测到后，它们的法向量需要被重新计算为在边上有着突然变化的法向量。
让F为edge-point set，对于每个F中的点p_i，由两种典型的情况：
	Case 1：p_i 接近于一个feature-tip point，其邻域法向量至少有三个聚类。
	Case 2：p_i 接近于一个feature-line point但远离一个feature-tip point，其邻域法向量有两个聚类。
为了调整点p_i 处的法向量，沿用公式（7），但放松quantity均衡的要求，正则化法向量以达到的最优化。
显然公式（9）取决于法向定向一致性，假设在Step 1已经解决了一致性问题。
在优化的最后，我们计算∑_(j=1)^k▒λ_j^((1)) ,∑_(j=1)^k▒λ_j^((2)) ,∑_(j=1)^k▒λ_j^((3))  来看一下那个最大。然后使用n ̂_1,n ̂_2,n ̂_3 中最显著的法向量来更新点p_i 的法向量。
整个正则化步骤就是对每个点重复完成该操作，见Figure 3d。
值得注意的是，公式（8）公式（9）通过优化操作，而不是k-means，在Section 4.6中介绍了区别。
### Point Location Refinement
接下来共同优化点的位置以和正则化后的法向量一致。类似于公式（2），提出公式（10）的最优化。其中Neigh′ (p_i )⊂Neigh(p_i)，包含了那些法向量接近的，即<n_j,n_i>≤π/6 。
Figure 3e展示了refinement结果。
### Edge Point Generation
令p_i 为一个被标记为edge zone的点，每个点p_i∈Neigh(p_i)定义了一个切平面Π≜(p_j,n_j)，且Π经过临近的几何变，类似于（Chen 2021），我们将点p_i 投影到feature line上最近的点，通过公式（11）介绍的最优化。
公式（11）中μ帮助将投影点z_i 到feature line上的最近点，注意到如果p_i 靠近一个corner点，最小化的过程会将p_i 拉向corner。默认设置μ=0.01。
Figure 3f展示了生成edge point。
## 3.4 Feature Preserving Interpolation-Based Reconstruction
最后阶段涉及surface重建。
由于隐式的方法很难处理feature line，我们产生一个多边形surface来插值扩张后的点集（包含edge point）。
RVD方法常用于mesh生成，如果能够找到一个base surface足够接近target surface，则RVD可以生成潜在的surface的一个高质量的近似。
然而Voronoi Diagram对待每一个位置都是相同的重要度，因此如果边上的点不够密集会导致结果的三角化不会明确地对齐潜在的feature line，如Figure 6ab。
我们观察到power diagram是一个更好地工具来应对这个问题，边上的点会造成更大的影响，见Figure 6 cd

在我们的方案中，首先在扩张后的点集运行screened Poisson reconstruction solver来生成base surface。
然后将每个点投影到重建的surface并且使用RPD来推断点之间的连接关系。
最后将连接关系复制到原先的点上，完成重建的任务，见Figure 7。
令δ为点之间的平均间隔，我们的应用中，edge point被指定为一个8δ^2 的权重，其它点指定0权重。
在Figure 8中比较了RVD和RPD，可以看到RVD没有对edge point作额外的处理，结果的三角化没有保留feature-line。相反，RPD能够保持点在不同的几何边上不会被连接。
# 4 Experimental Results
## 4.1 Experimental Setting
### Experimental Platform and Parameters
所有的点云模型都被规范化到[−0.5,0.5]^3，每个模型提取50K个surface采样点，用white noise sampling方法进行采样。
所有的实验都用先相同的参数。
使用LBFGS function在ALGLIB solver中来求解Section 3中的限制最优化问题，并且设置function的限制为“minbleicsetlc”。
所有最优化问题的终止条件设置为梯度法向norm不超过10^(−4)，公式（11）设置为10^(−6) 来获得更高的准确度。
此外，我们延申RVD的应用来计算RPD。
### Datasets
测试都是在人造的和raw-scan数据，其中人造的数据采样自ABC dataset。
由于一些模型有着缺陷（如自交），我们选取了dataset中的100个模型。对于每个模型，采样50K个点，用white noise sampling方法来获得noise-free的数据。
另外，我们添加不同等级的Gaussian noise来生成noisy数据。
### Evaluation Metrics
本文的主题关系到点的合并和surface重建，我们使用OCD和OECD来测量一个合并后的点云到ground-truth surface有多近，其中OECD是由在feature line附近距离小于0.005的这些点的平均偏差计算的。
为了评价重建网格的正确性，我们使用了三个indicator，包括CD（Chamfer Distance）、F1（F-score）、NC（Normal Consistency），我们还使用NMC提出的ECD（Edge Chamfer Distance）、EF1（Edge F-score）来评价重建mesh的sharpness。
## 4.2 Point Cloud Consolidation
我们对比我们的方法和最先进的点云合并方法，包括RIMLS，EAR，EC-Net，Dis-PU，MFLE。
同时， 用0.25%，0.5%， 1%的高斯噪声来观察降噪的稳定性。
Figure 9 给出了例子，关于可视化点云合并方法的差异。
OCD，OECD统计在Table 1。
同时给出了一个降噪能力、边缘意识、法向修正质量的质量评估，为了得到公正的比较，在调和RIMLS，EAR，MFLE的参数时我们使用预先训练好的EC-Net和Dis-PU模型，以达到最好的分数。
从Figure9 中看出我们的方法得到的点更好地反映出几何边。
合并一个CAD型点云准确性的挑战在于潜在的特征边，但对于RIMLS很难去再生出几何上的尖锐特征，因为它是保持C2连续的。
类似的，尽管EAR、EC-Net、MFLE被设计来保持尖锐特征，但它们很难足够地重生出几何边其上有突然变化的法向。
Dis-PU更多地聚焦于上采样、目标分布均匀性、接近surface上，没有很明确地保持线特征。
Figure 10 中可视化了EAR和我们的方法的差别。EAR倾向于增加边区域上扩张点的密度，但额外增加的点并没有准确地对齐潜在的特征线，特别是在噪声的表现下。
总的来说，CAD-type点云在结构上不同于其它模型，数据降噪操作，边区域识别，点合并依赖。
我们算法的关键想法基于一个弱前提，即点云是局部平面的，因此我们框架中的Step1，Step3，Step4聚焦于正则化点的位置和法向。当点位置和法向量正确后，edge point生成步骤可以生成出一个扩张的点集对齐几何边。
## 4.3 Surface Reconstruction Quality
本文的主题是恢复潜在的几何，以及网格生成，从一个CAD-type点云。它包含点合并和mesh生成的步骤。
因此有必要来找到关于点合并方法和surface重建方法的最佳组合，Table2 给出了评价不同“点合并加上surface重建”的合并在不同噪声等级的点云数据，其中w.o.表示没有任何点合并。

surface重建算子用于比较包含GD（Greedy Delaunay），RIMLS（robust implicit moving least-square），SPR（Screened Poisson Reconstruction），P2S（Points2Surf），DSE（DSE-meshing）。
其中P2S、SPR，RIMLS是隐式的方法，GD和DSE是基于插值的。
注意到本文提出的RPD的重建策略需要一个base surface作为支撑，我们使用SPR的输出来提供base surface，但一个不同的方法同样有效。
另外，我们的点合并策略中所有新增加的点都有一个edge-point标签，这使得我们能够设置一个更大的权重在计算RPD上。
RIMLS和EAR的点云合并缺乏edge-point标签，因此我们使用RVD。
我们的重建框架包含一个正则化法向量的步骤，因此不需要输入点云有可靠的法向量。
值得注意的是给定点云可能缺乏法向信息，这些重建方法可能需要法向信息，我们用PCA来初始化法向量。
Table 2的统计能让我们作一些观察。
	首先，我们的重建框架，包含了edge-point扩张策略，和基于RPD的重建策略是很好的结合。它的ECD和EF1分数显著好于其它的组合，CD和F1和NC分数好于其它组合。
	第二，对于隐式方法很难在输出mesh中保持特征线（见Table2中的ECD和EF1分数），例如RIMLS没法生成尖锐特征线，Figure11 可视化了所有“点组合+surface重建”的重建结果。但我们也指出现有的基于插值的方法，如GD和DSE，没法严格地保证一个水密流形output，特别是给定点云包含薄的结构。DSE甚至没法保证face定向一致，所以我们不得不使用额外的预处理。相比之下，我们的RPD重建继承RVD很好的特征，并且有保证流形的优势。Figure12 展示了一些典型的重建结果在有噪声的点云上，或在原始点云上，或在合成的结果上。更多的CAD重建结果展示在Figure 13，我们提供一些更多的可视化细节在附录A。
## 4.4 Runtime Performance
Table3 中统计了运行时间，#V点的数量从10K到100K。
我们的算法由多个阶段，记录每个步骤T1（点云降噪），T2（edge zone识别），T3（法向量正则），T4（点位置修正），T5（edge point生成），SPR（使用SPR来生成base surface），RPD（使用RPD来生成最终三角mesh）。
考虑到一些步骤可以并行，使用24核来提高计算速度。
从统计中可以看到每个stage的运行时间线性提高。
我们的实验中，50K是默认的输入点云尺寸，需要大概10s。
不同方法的细节统计比较运行时间表现见Figure 14，可以看到我们的方法有一个有竞争力的运行时间表现，特别是相比于深度学习技术。
## 4.5 Influence of Parameters
Figure 15可视化了参数ξ,r,μ的影响。
参数ξ在公式（2）中用于控制降噪程度，一个比较大的值会倾向于抑制一个点的运动，然而一个小的值倾向于鼓励平滑点的位置和法向。
	实验中设置为1.0。
	我们的算法流程对ξ是迟钝的，因为流程中还包含更进一步的法向量正则化（Step 3）和点位置改进（Step 4）。
patch半径r用于控制edge zone的宽度，如果r太小，很有可能过少点参与优化（公式7、11），导致点的误分类。如果r过大，我们的算法会在薄板模型上失败。
	Figure 15中展示了不同的半径设置r=3δ,2δ,1.5δ，其中δ是点之间的平均距离。
	实验中默认设置r=2δ。
在edge point生成过程中，我们edge zone的投影一个点到临近的几何边上，优化公式（11）包含一个项μ|(|z_I−p_i |)|^2 来避免点沿着潜在的feature line漂移。
默认设置为μ=0.01，Table 4给出了不同的参数设置对重建质量的影响统计。
## 4.6 Optimal Mass Transport v.s. K-means
本文中我们规划了edeg zone识别的任务为最优传输，见公式8，更进一步使用公式9来正则化法向量。它们都基于优化。
尽管k-means看起来也行，Figure 16中的对比展示了我们的方法可以生成更高保真度的feature line，得益于我们的优化能力驱动公式更准确地表征几何边缘的几何特征。
值得注意的是k=2用来替换公式8的k-means，k=3来替换公式9的k-means。
## 4.7 Robustness to Point Density
我们的认知中，许多现有的重建方法对点的密度是敏感的。
为了测试对密度的鲁棒性，我们准备了一个有着不同点密度综合点云，见Figure 17a，可以看到我们的重建算法框架可以处理不均匀的采样数据，因为我们选择的参数可以适应于点密度的变化。
## 4.8 Result on Real Scans
我们扫描了真实世界的物体，见Figure 18。
## 4.9 Potential Applications
复现一个有特征边的模型最大的好处在于支持多样的模型编辑工作，例如局部调整模型的大小。
在被重建的surface上，可以简单地分解整个surface为一系列surface patch集。如Figure 21中，CAD surface的每个facet是平面或圆柱面。根据surface类型的先验知识，每个facet的隐式等式可以被拟合，因此能够快速地估计驱动参数以定义每个surface原始。因此用户允许指定不同参数来局部改变形状。
# 5 Limitation
### Dihedral angle
REFPS在现在的形式下，支持二面角的范围是[π/6,5π/6]，如果二面角太接近π（即近似平面），则由于数值问题，很难由公式（11）推断点在edge zone的精确投影位置。见Figure 22顶上一行。
如果二面角太接近于0（即一个尖锐的转角）或者模型包含一个非常薄的部分，则RPD可能会错误地连接不同side的两个点，见Figure 22底下一行。
### Normal inconsistency and noise
注意到我们多部操作框架中有两个步骤来refine法向量，有必要去测试我们的算法是否高度依赖法向的一致性和正确性。
在Figure 23中，顶部一行展示了扩张后的点集，当随机反转5%，10%，20%的法向量，底部一行展示了不同噪声的结果。
可以看出，RFEPS对法向一致性比较迟钝，仅仅在20%法向反转时失败。
### Planarity assumption
平面性假设的主要目的是生成edge point并帮助恢复feature line。
我们使用Figure 24的例子来测试我们的算法可以恢复圆柱面的一个光滑surface。
尽管我们的算法包含一个对点的位置去噪的步骤，结果点的位置并不有序地落在母线上，见Figure 24a-c.
surface光滑度取决于大量的点采样密度，见Figure 24d-f.
因此，从Figure 24中可以看到我们的算法得到的扩张后的点并不会帮助提高光滑度，不像其它上采样点的算法。
# 6 Conclusion
本文中，我们提出了转变CAD模型有噪声点集数据为一个具备特征边的多边形surface。
我们的算法包含多个阶段，其中两个是edge-point合并和特征线保持重建。
对于edge-point合并，我们提出一个离散最优传输形式来识别edge zone并生成足够多的额外点来对齐几何边。
对于保特征线重建，我们使用了RPD来插值扩张后的点集，通过给予edge point高优先级连接。
实验结果展示了两个结合的技术可以利用CAD模型的先验知识，即目标surface包含多个光滑patch缝纫在一起，通过刚性特征线。
在综合的和真实扫描的数据证实了给出算法的效率和有效性。
