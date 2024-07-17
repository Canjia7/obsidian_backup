> @ARTICLE{9351738,
  author={Chen, Honghua and Huang, Yaoran and Xie, Qian and Liu, Yuanpeng and Zhang, Yuan and Wei, Mingqiang and Wang, Jun},
  journal={IEEE Transactions on Automation Science and Engineering}, 
  title={Multiscale Feature Line Extraction From Raw Point Clouds Based on Local Surface Variation and Anisotropic Contraction}, 
  year={2022},
  volume={19},
  number={2},
  pages={1003-1016},
  keywords={Feature extraction;Three-dimensional displays;Shape;Surface reconstruction;Image edge detection;Feature detection;TV;Anisotropic contraction;multiscale feature detection;pose estimation;primary topics: point clouds;surface variation},
  doi={10.1109/TASE.2021.3053006}}
# 0 Abstract
近期的3D扫描技术可以提供多种样的数字化3D数据。
许多扫描的数据的形式是一个无结构化的点云。这样低级的3D数据表示通常只包含几何性质（如点的位置），而缺少高级的结构线索，如特征线。
特征线可以定义为形状上的视觉显著特征，包括边、脊线、谷线在多种形状中，可以支持一系列下游应用，例如形状重建和分析。
我们提出了一个2-phase算法来从点云中提取出特征线。
为了踢出去大尺寸和潜在的特征线，我们首先定义一个统计指标来检测所有的潜在特征点（避开噪声的影响）。然后为了正确地从这些定义的粗糙特征点重建出特征线，我们介绍了一种各向异性收缩方案来强制特征点落在潜在的实际特征线上。
为了说明我们方法的可靠性，大量的实验在综合的和raw数据上。视觉上和定量上都显示我们的方法对噪声鲁棒并且可以准确地提取出多尺寸的特征线。
另外，我们的方法逐渐应用于robotic picking
### Note to Practitioners
本文动机自从raw扫描点云中提取特征线。
特征线作为最重要的结构信息之一，描述了我们生活中实际物体的形状。
从无结构点云中提取出这种尖锐特征可以促进一系列下游实际应用，例如工业设计，工业制造，robotic grasping。
现有方法检测特征要不就严重依赖于differential quantities，对噪声很敏感，或需要一个精心设计的局部表达，在识别小尺寸特征时会失败。
这些挑战激发我们来设计一个新的方法，目的是提取出多尺寸特征线，且对严重的噪声鲁棒。
我们的工作发展的技术可以提供高质量的特征点和特征线，可以服务于高等级结构信息和促进许多应用。
在6-DoF的额外应用造成评估证明我们的方法潜在地之于robotic picking
# 1 Introduction
# 2 Related Work
## A. Feature Detection on Meshes
## B. Feature Detection on Unstructured Point Clouds
# 3 Algorithm Overview
Fig. 3展示了所提出的方法的概述，由以下四个主要步骤组成：
### 1 Feature Point Candidates Localization
为了提取不同尺度的feature lines，我们首先对所有可能的feature points进行定位
观察到，feature points经常存在于surface bending区域，我们利用每个点附近的local maximal variance作为一个度量来详尽地检测所有feature candidates
更进一步，由于input的退化，比如不规则噪声（bumps和stepping noise），一些点可能被错误地认为是feature point，因此我们合并PFR来获得更鲁棒的结果
### 2 L1-Median Normal Filter
由于所有检测的feature points的normals都是简单地通过PCA计算出来的
	因此，当input噪声很大时，某些误差可能被保留
直接将检测到的feature points和不准确的normals作为input到optimization步骤，并不能驱使这些points移动到它们正确的位置
因此，我们为feature points采用anisotropic L1-median normal filter，它比基于quadric-error-minimization（QEM）的bilateral filter性能更好，而不涉及额外的artifacts
### 3 Feature Points Optimization
我们观察到检测到的feature points大致分布在真实的潜在feature lines附近
虽然从大尺度上看，它们是线性分布的，但从局部来看，它们是band-like的，覆盖在两个相交的surfaces上
因此，我们需要从中提取合适的对象来支持下面的connection操作
在这项工作中，我们将正确的feature points的提取问题表述为anisotropic contraction问题
### 4 Feature Line Connection
一旦我们得到最终的feature points，我们对它们统一采样，并执行connection操作以创建line-type几何表示
![[Pasted image 20240717143615.png]]
# 4 Method
## A. Feature Point Candidates Localization
定义一个无序、不均匀的噪声点集为$P$，点$p_i$的最近邻域点为$\mathcal{P}_i^r$，我们考虑的都是以$p_i$为圆心半径为$r$的球。
我们想要点云集具备法向，但可能只有位置信息。为了得到点云的法向，我们使用利用PCA来得到初始的法向场。

观察到，特征点经常存在于surface区域中变化最大的区域。因此，对于每个点，我们定义一个简单且直观的度量来度量 ==local surface variance LSV== 局部 surface 变化$D_i$ 。
![[Pasted image 20240717150410.png]]
显然$D_i$是点$p_i$邻域点里到点$p_i$由$n_i$定义的切平面的最大距离。
见Fig 4，我们发现给定一个小邻域，如果一个点属于候选特征点，它的LSV值会很大，而非特征点会提供小的LSV。
![[Pasted image 20240717150512.png]]
不幸的是，当我们在raw数据中计算这个度量时，一些在光滑surface上非特征点被考虑为潜在的特征点，见Figure 5b
通过缩放局部区域，我们观察到局部surface片看起来有波动，由不规则的噪声造成的。
![[Pasted image 20240717150630.png]]
考虑到上述提到的问题，我们建议包含PFR来建立一个对heavy噪声的阻力。
对每个点$p_i$我们决定一个最佳fitting平面$θ_i$在一个相对较大的范围，通过下述对象函数。
	$argmax_{θ_i} E_i (θ_i)$
	其中$E_i (θ_i )=1/(|\mathcal{P}_i^r2 |) ∑_(p_j\in\mathcal{P}_i^{r_2}) W_{σ_j} (p_j,θ_i)$
	$\mathcal{P}_i^{r_2}$ 是$p_i$在一个大范围中的邻域点集，默认设置$r_2=5r_1$
	$W_{σ_j}(p_j,θ_i)=e^{−r_{j.θ_i}^2/σ_f^2}$是高斯权重函数，$r_{j,θ_i}$定义了点$p_j$到平面$θ_i$的距离，$σ_f$是残差bandwidth。
然后我们制定联合度量PFR-LSV为：
![[Pasted image 20240717151444.png]]
	其中η是一个增大参数
由于在噪声plane上的任意点的PFR值都会接近于一个比较大的值，因此$L_i$的值会受到限制，这使得错误识别的潜在feature points成为nonfeature的
同时，structures附近的点的PFR值会保持比较小，它鼓励这些点仍然属于候选feature points
最后，如果当前点的PFR-LSV大于或等于一个阈值$l_{th}$，则其作为一个潜在feature points

值得注意的是，对于非平面的局部区域，其PFR值会相对较小，从而使$L_i$变大
因此，有许多nonfeature points（input圆柱的光滑surface）被视为feature points，如Figure 5e
因此，处理此类数据时，我们调整一个比较小的$\eta$以使得PFR度量对最终结果的贡献较小

【10】【11】也使用了类似的想法来识别feature points...
然而，这两种方法都可能无法识别小尺度特征，如Figure 6b、6d
由于PFR-LSV中的LSV部分是为了直接检测surface波动而精心设计的，因此我们的方法能够覆盖multiscale特征
![[Pasted image 20240717152303.png]]
## B. L1-Median Normal Filter
我们将识别出的band-like feature point候选视为一个局部点集$Q={q_i}_{i=1}^I\subset\mathbb{R}^3$
为检测出的feature points $Q$计算可靠的normal field，我们首先构建一个新的点集$Q^{new}$，其包含$Q$中每个点$q_i$和原始点云$P$的$q_i$的邻域点
	邻域选择在一个固定的半径$5d$，$d$是input point cloud $P$中任意最近邻的平均距离

我们使用L1-median normal filter，其对异常值和噪声不太敏感
### L1-Median
L1 median对离群点更加鲁棒
### L1-Median Normal Filter
![[Pasted image 20240717153743.png]]
对每个feature point $q_i$，最小化（变量是$n_i$）附近的normal differences，可以得到最后过滤后的normals

给定具有两条shallow feature lines的带噪声的band-like input，如Figure 7，通过L1-median normal filter可以很好地区分两条feature lines上的normals
注意我们仅计算$Q^{new}$的点的法向，而不是整个input，因此计算速度相对较快
![[Pasted image 20240717153450.png]]
## C. Feature Point Optimization
我们观察到潜在feature point set $Q$被许多接近真实feature lines的nonfeature points污染
为了恢复正确的feature lines，我们需要从这些候选点中生成一个更加精确的feature point set $X$

【14】、[[L1-medial skeleton of point cloud]]局部地使用L1-median来创建skeleton-like几何表示作为feature line
它们考虑正确的features大致落在feature候选点bands的中间
Figure 8a展示了L1-median优化后feature结果
	我们可以明显地发现，优化后的feature缩小了，并没有准确地定位在真实的feature line上，从视觉上讲，它们更靠近中轴线
![[Pasted image 20240717154637.png]]
为了解决这个问题，我们使用anisotropic contraction方法来恢复feature line的底层真实几何【49】【50】
我们的优化函数如下：
![[Pasted image 20240717155050.png]]
	意味着，到 由每个邻域点$q_j$及其法向$n_j$ 定义的切平面 的距离总和
通过优化，候选feature points在两个surface patches相交的区域累计
然而，在我们的测试中，我们发现新生成的点集$X$是扭曲的，并且没有准确地累计到特征上，见Figure 8b
为了得到更稳定的解，我们重新表述我们的问题为：
![[Pasted image 20240717155536.png]]
	添加了新的项，意味着到由点$x_i$和$n_i$定义的切平面的距离之和
最终目标函数可以鼓励优化的feature points沿着所有可能的方向移动
见Figure 8c，我们发现，通过添加了新的项后，我们的optimization模型生成了一个更准确的结果

另外，为了构造出feature lines，$X$中的点应该不能离彼此太近，它们应该以dominant direction方向排列
受到[[L1-medial skeleton of point cloud]]启发，我们还加入了一个正则化项
![[Pasted image 20240717160149.png]]
正则项的分母为向量$x_i-x_i'$投影到由$x_i'$和$n_i'$定义的切平面的长度
其中$\sigma_i$定义为驱动$x_i$附近点沿着principal方向排列的
![[Pasted image 20240717165745.png]]
其中$\lambda_i^0\geq\lambda_i^1\geq\lambda_i^2$是在$x_i$邻域使用PCA时的特征值
几何意义是当$\sigma_i$越靠近1，则$\lambda_i^0$比$\lambda_i^1,\lambda_i^2$要大，即$x_i$附近更多点对其到相同的方向

我们使用梯度下降法以迭代的方式来求解这个最终的优化问题
初始点集$X$从$Q$中随机采样
梯度下降的learning rate设置为一个比较小的值（默认0.001），以避免优化结果偏离实际直线
## D. Feature Line Connection
目前采取的步骤使我们能够成功地获得准确的特征点
注意，我们的目标是寻求feature的line-type表示
因此，我们最后需要连接所有精确的feature points

像【10】【12】一样，我们直接将连接问题转换为最小生成树构造问题
具体来说
	我们首先从精确feature points中统一地downsample一组点
	然后构建一个graph，用sampled feature points作为graph的nodes，将sampled feature point与k（k=5）邻域点的欧氏距离作为edge weights
	最后，采用经典的Prim算法来生成feature line

此外，由于refined feature points可能在拓扑上很复杂，我们经常将它们分割成不同的geometry-aware parts来处理connection问题
# 5 Results and Analysis
## A. Parameters Selections and Tuning
涉及几个关键参数来控制最终结果
	邻域半径$r_1$
		根据feature尺寸来设置，首先计算一个比 检测到特征的最小尺寸 大的粗糙的尺寸
		然后设置为比粗糙的尺寸更大的尺寸
	计算PFR-LSV的augmentation系数$\eta$
		用于控制PFR-LSV度量的PFR项的作用
		如果input包含low-level噪声或者包含很多平面区域时，我们设置为比较大的值
	检测feature候选的feature point阈值$l_{th}$
		小的值帮助定位相对小尺寸的features
		然而，如果noise level很高，小的值会导致noisy points被误识别为feature points
		手动选择
	normal filtering的bandwidths $\sigma_s,\sigma_n$
		$\sigma_n$表示normal差异角度，实验中在$[10^\circ,30^\circ]$
	balance参数$\gamma$
		经验上固定为0.1
		较大的值会减弱feature point contraction的效果，并倾向于产生更多的分散特征（Figure 8d）
	contraction的迭代数量$iter$
		在10-20之间
具体的列在Table 1和Table 2中
![[Pasted image 20240717172002.png]]
![[Pasted image 20240717172011.png]]
## B. Synthetic Point Clouds
如Figure 9，我们的方法在有噪声时比基于differential quantity的方法要好得多
![[Pasted image 20240717172131.png]]
## C. Scanned Point Clouds
## D. Quantitative Comparison
此外，由于很难直接评估一个curve feature polyline和其ground-truth版本之间的误差
因此，我们通过计算feature points和相应的ground-truth feature points的误差
我们均匀地在这些polylines上采样密集的点来作为ground-truth feature points
误差通过三个指标度量
	$D_{mean}$为ground-truth feature points和最终检测feature points之间的平均距离
	$D_{max}$为最大距离
	$\sigma$为距离standard deviation
结果见Table 3
![[Pasted image 20240717172758.png]]
## E. Robustness to Noise
## F. Robustness to the Uniformity of Point Sampling
## G. Effectiveness to Shallow Features
## H. Complex Scene
## I. Timing
![[Pasted image 20240717172953.png]]
## J. Application: Pose Estimation
# 6 Conclusion
### Limitations
我们的方法效率相对较慢，可以通过并行feature detection、feature optimization来加速
当input被噪声严重破坏时，优化函数可能不会收敛到真实的feature line上
	这是因为normal field计算不准确
我们的方法的最佳feature extraction依赖于人工参数选择