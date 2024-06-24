> @article{CHEN201131,
title = {Uniform offsetting of polygonal model based on Layered Depth-Normal Images},
journal = {Computer-Aided Design},
volume = {43},
number = {1},
pages = {31-46},
year = {2011},
issn = {0010-4485},
doi = {https://doi.org/10.1016/j.cad.2010.09.002},
url = {https://www.sciencedirect.com/science/article/pii/S0010448510001697},
author = {Yong Chen and Charlie C.L. Wang},
keywords = {Offset surfaces, Geometric modeling, Trimming self-intersections, Layered Depth-Normal Images, Point-sampled geometry},
abstract = {Uniform offsetting is an important geometric operation for computer-aided design and manufacturing (CAD/CAM) applications such as rapid prototyping, NC machining, coordinate measuring machines, robot collision avoidance, and Hausdorff error calculation. We present a novel method for offsetting (grown and shrunk) a solid model by an arbitrary distance r. First, offset polygons are directly computed for each face, edge, and vertex of an input solid model. The computed polygonal meshes form a continuous boundary; however, such a boundary is invalid since there exist meshes that are closer to the original model than the given distance r as well as self-intersections. Based on the problematic polygonal meshes, we construct a well-structured point-based model, Layered Depth-Normal Image (LDNI), in three orthogonal directions. The accuracy of the generated point-based model can be controlled by setting the tessellation and sampling rates during the construction process. We then process all the sampling points in the model by using a set of point filters to delete all the invalid points. Based on the remaining points, we construct a two-manifold polygonal contour as the resulting offset boundary. Our method is general, simple and efficient. We report experimental results on a variety of CAD models and discuss various applications of the developed uniform offsetting method.}
}
# 0 Abstract
uniform offset是一个在计算机辅助设计更重要的几何操作。
本文提出了一种新的方法，offset一个solid模型以任意的距离r。
首先，offset多边形直接在input solid模型的面、边、点上计算。
计算得到的多边形mesh有一个连续的边界。然而这个边界是非法的，因为存在原model距离小于r的自交部分。
对于有问题的多边形mesh，我们构造一个well-structured point-based模型，Layerd Depth-Normal Image（LDNI），在三个正交方向上。
生成的point-based模型的准确性被控制，通过设定棋盘和采样率在构建过程中。
然后加工model上所有采样点，通过一系列point filters来决定哪些是无效点。
根据剩余的点，构建一个2-manifold多边形轮廓，作为最终结果的offset边缘。
我们的方法简单且高效。
# 1 Introduction
对一个solid S作距离为r的offset，点集定义在Euclidean空间E^2 或E^3 上。
在Figure 1中，半径为r的球定义为b_r，则可以定义两个操作：
	S以r作grow为S↑_r=S⊕b_r
	S以r作shrunk为S↓_r=S⊗b_r
	A⊕B为A和B的一种Minkowski sum的特殊形式，定义为：C=A⊕B={a+b|a∈A,b∈B}。
	A⊗B为一种Minkowski difference的特殊形式，定义为：A⊗B=(A ̅+B) ̅。
![[Pasted image 20240623172745.png]]
本文的目标对一个多边形model是计算一个任意offset距离的均匀offsetting model，并且没有瑕疵，如间隙、洞、自交等。

尽管offset操作在数学上是良定的，但是计算一个offset model是困难的。
由offset距离导致的位置变化经常会导致自交和拓扑改变。因此整理无效的offset surface是必要的，但是计算很复杂而且不稳定。实践中在点、边、面之间有许多退化情况需要谨慎地考虑。
为了避免这些困难，前期工作基于体积方法（4-7）和采样点方法（6）。这些方法先生成体积网格和采样点来近似offset model，然后计算距离场以用来计算点到原boundary的最短距离，依次来判断位于inside或outside。最后一个offset model从体积表示的重建出来。
## 1.1 Our approach
我们的工作中，用一个体积方法来计算给定solid的offset boundary。
然而，不同于之前基于距离场计算的体积方法，我们的方法是直接计算offset boundary，将boundary转变为结构化采样点，并进行filter，用于重建出最后的轮廓。
Figure 2展示了方法的说明，首先从input的顶点、边、三角面计算一个offset surface。该offset surface有一个连续的边界。（见Figure 2a）
然而，生成的offset surface可能有自交，是surface上距离到原surface小于r的。

为了整理无效的offset mesh，构建一个well-structured point表示，称为Layer Depth-Normal Image（LDNI）（8，9），来采样offset mesh。（见Figure 2b）

生成LDNI模型的准确率可以被重建过程控制。然后用点filter处理所有的offset LDNI采样点。
因此所有的无效点都被offset LDNI识别和去除。
在filter操作后剩余点展示如Figure 2c。

最终重建一个offset轮廓，从加工好的LDNI model，使用适应性采样（8）和保持流形的contouring方法（10）。
计算出的均匀offset model见Figure 2d。
![[Pasted image 20240623172758.png]]
## 1.2 Key contributions
### General
可以处理任意的offset距离。
关于检测奇异点和处理自交的情况，通过point-based表示进行处理。因此重建的offset mode在拓扑上可能与input完全不同。
### Accurate
结果的多边形model是一个实际offset boundary的近似。
其近似误差is bounded by四面体密度和用于构建LDNI model的采样率。
基于mesh tiling技术，LDNI技术可以设置地足够小，就能满足大部分应用需求了。
### Efficient
我们的算法直接计算offset surface，以及一个关联的LDNI模型。
对于每一个采样点，判断是inside或outside，直接基于一个ray casting test，而不是通过计算到原网格的最短距离。
因此，我们算法在实现上很少联系到offset距离。其它算法（4，7）因为需要大量计算offset距离而显著变慢。
另外，几个构建LDNI model的关键步骤可以并行。
### Simple
我们的方法主要基于LDNI，相对简单和易于执行。

Figure 3和Figure 4展示了我们的算法生成的两个offset例子。
![[Pasted image 20240623172832.png]]
# 2 Related work
先前的方法（11-13）首先计算一个offset surface的superset，通过将点offset成sphere，边offset成cylinder，面offset成parallel face。然后整理superset，删除离原solid距离太近的元素。整理过程的计算复杂度和数值难度让这个方法很难鲁棒应用。
为了避免整理surface的计算难度，一些方法提出。（14）

另一种近似方法是将3D model的offset转变为2D轮廓的offset。如（31，15，16）。
然而，这些方法主要用在一些特殊情况。没有构建offset surface model。

ray-rep表的方法。（17）

基于距离体积和快速marching方法被提出用于CSG model。该方法计算一个在估计的surface附近的narrow-band到CSG model的一些点的距离。如（19.28，30）。
主要的步骤是在一个均匀网格上计算节点值，用快速marching方法重建等值面。

最近，（4）提出了point-based的offset方法，首先计算原model的均匀采样点。然后对于每个点，构建一offset line集以标记所有的相交voxel网格和offset点是否有效。
最终offset surface通过标记voxel和offset point进行重建。
具体算法如（5，6，7）。
# 3 Principle of the LDNI-baesd offsetting method
我们的方法基于offset的数学性质。
假设∂S是一个S的拓扑边界。
（20）中定义了正规化集S通过一个正距离r的正规化offset：S↑∗r={p:d(p,S)≤r}，其中d(p,S)=inf_(q∈S)⁡〖||p−q||〗 。
根据该定义，∂(S↑∗r)⊂{p:d(p,S)=r}。
对于p∉S,d(p,S)=d(p,∂S)。
负距离的offset是定义为对S的补集的正距离offset的补集。可以得到类似的结果：∂(S↓∗r)⊂{p:d(p,c∗S)=r}，其中c∗定义为正则化补集。

我们方法的原理是首先生成一个superset{p:d(p,∂S)=r}，然后从中计算offset surface ∂(S↑∗r)。
对于一个多边形model正则化的集合S，其边界表示为：
	∂S=V(S)∪E(S)∪F(S)
	其中V(S),E(S),F(S)分别依赖于S的点、边、面。
相应地，对于点q∈∂S，可以计算集合{p:d(p,q)=r}作为〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 。
	其中〖F||〗_r^+,〖E||〗_r^+,〖V||〗_r^+ 分别为V(S),E(S),F(S)的正法向offset。


Faces F(S)
	假设f是S的一个面，且q∈f。
	可以通过沿单位法向n移动每个点q以距离r，构造〖F||〗_r^+ 。即令p=q+rn。
Edges E(S)
	假设e是S的一个面，且q∈e，其相邻的两个面为f_1,f_2 。
	可以通过构造一个中心在e半径为r的圆柱，构造〖E||〗_r^+ 。
	该圆柱可以be bounded by 〖f_1 ||〗_r^+,〖f_2 ||〗_r^+，因为点在bounded外的部分离f_1,f_2 很近。
Vertices V(S)
	假设v是S的一个点，且q=v。点v的相邻边为e_1,e_2,…,e_i 。
	可以通过构建一个中心在v半径为r的球体，构造〖V||〗_r^+ 。
	 该球体可以be bounded by 〖e_1 ||〗_r^+,〖e_2 ||〗_r^+,…,〖e_i ||〗_r^+，因为点在bounded外的部分离相邻面很近。
因此，∂(S↑∗r)⊆〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 。
注意到所有〖F||〗_r^+,〖E||〗_r^+,〖V||〗_r^+ 中的点到∂S的三某些点的距离为r，记这些点为q_i 。然而有一些点到∂S上一些点的距离会小一点，记为q_k 。
当p为一个无效点时，〖q_i ||〗_r^+ 和〖q_k ||〗_r^+ 会互相相交。
这样的自交是一个核心挑战。
定义inner point为：
	采样自〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+，且到∂S的最短距离小于offfset距离r
定义boundary point为：
	所有在〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 上的点同时也在∂(S↑∗r)。
因此我们算法的本质是移除〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 中的inner point，使得boundary point可以近似于S↑∗r的边界。

本文讨论的均匀offset操作是一种常规的Minkowski操作。
（21）中提出了一个框架，将Minkowski操作转变为卷积操作。
我们的方法在计算offset边界，用多边形tracing tour和判断其ray casting value基于winding number。
该原则的2D情况见Figure 5。
假设定义在2D的一个边界上的部分如Figure 5a，包含三条边E_1,E_2,E_3，两个点V_1,V_2 。
offset得到∂(S)是向内收缩r的，可以计算每一条边E(S)中的〖e||〗_r^−，每一个点V(S)中的〖v||〗_r^− 。
注意到〖V||〗_r^−∪〖E||〗_r^+ 构成了一个连续的边界（见Figure 5a右），本身有许多自交。
我们使用一个LDNI来采样边界〖V||〗_r^−∪〖E||〗_r^+，Figure 5b展示了一个LDNI的2D的示意图。圆圈指示的点记录在x-LDNI，方块指示的点记录在y-LDNI。
因此每个LDNI的像素包含一系列Hermite data，说明了交点处到viewing plane的深度，以及采样surface在交点上的单位法向。
另外，所有像素的深度都被排序为升序。

一系列point filter被构建出来以处理所有的LDNI点。
Figure 5c展示了对于像素X_i,Y_i 的ray casting test value。
当一个点的两个邻域没有在(0,1)或(1,0)的ray casting value，该点会被删除。
Figure 5c的B视图中，展示了如果两个LDNI点的距离Δd<ε=10^(−5) 则会被删除。
在filtering操作后，剩余的LDNI点如Figure 5c左图。
注意到这些点的准确face法向也是已知的。
因此基于在这些点的Hermite date，一个流形轮廓可以被计算出来，通过x-LDNI，y-LDNI像素ray的交点定义。
该轮廓就是一个∂(S↓∗r)的近似边界。

接下来说明我们的方法通过3D模型的2D轮廓切飘，如Figure 6。
offset boundary 〖V||〗_r^−∪〖E||〗_r^− 的内部loop见Figure 6a。（蓝色的为原模型，原模型内部有两个loop，外部有一个loop）
通过AB两个放大的视图，可以看到复杂的自交存在于offset边界中。
通过之前的讨论，我们通过转变边界为LDNI模型并用point fliter来生成采样点，以处理该自交。
处理后的LDNI点见Figure 6c上图，offset边界的内部loop2（即一个圆圈向内offset）以及关联的LDNI点见Figure 6c中间。
注意到如果只有ray casting filter被使用，错误的判断可能会出现在分类点上。例如结果点基于这样一个fliter被展示在Figure 6c右图，对于这样的内部点，另一种fliter——offset property filter，可以正确地移除它们。
最终重建轮廓以近似∂(S↑∗r)见Figure 6d。

向内offset的轮廓∂(S↓∗r)也能被重建，如Figure 7a。
Figure 7b为一个更复杂的，有更多loop的结果。

我们的方法是基于近似方法的，因此与其它体积方法有类似的缺点。
	feature的尺寸如果小于LDNI的resolution，可能在最终生成的网格中不会被捕捉到。
然而对比其它仅基于cell或voxel的体积方法，我们的方法更准确：
	可以准确计算offset boundary 〖V||〗_r^−∪〖E||〗_r^− 。
	可以准确计算〖V||〗_r^−∪〖E||〗_r^− 沿着ray的ray casting point。因此，边界点在关联到LDNI像素的ray是准确的，包括位置和法向。
	尽管在实际应用中，计算四面体形式而不是准确的〖V||〗_r^−∪〖E||〗_r^−，计算的准确率可以通过控制四面体的密度来控制。
因此计算得到的offset轮廓是一个没有大于LDNI resolution尺寸特征的准确的offset边界的近似。
（8）一个volume tiling技术可以用来保证一个足够小的LDNI resolution。

该文后续介绍：
	Section 4介绍计算候选offset mesh的方法。
	Section 5介绍决定无效offset point的方法。
	Section 6介绍从处理过的LDNI model构建offset轮廓的方法。
	Section 7介绍有关我们方法的应用技术。
	Section 8介绍实验结果和多样的测试case，以及一些我们方法的应用
Section 9介绍了后续工作。


4 Compute continuous offset boundary
（22，29）已经讨论过将face、edge、vertex offset成face、cylinder、sphere。
我们的方法中，需要确保〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 能够生成一个连续边界且有统一定向。
Theorem 4.1
	让S为一个正则化点集，∂S为其边界定义为一个多边形model。
	其offset mesh 〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 如Section 3中定义会构建一个连续的边界且有统一定向。
Proof
	根据positive normal offset。
	1、〖F||〗_r^+ 是一个变换的face集，根据surface的法向方向平移得到。（见Figure 8b）
	2、对一个有效的input model ∂S，每条边e有两个半边he_1,he_2，根据其邻接的面是f_1,f_2 。
	半边生成的h〖e_1 ||〗_r^+,h〖e_2 ||〗_r^+ 会在同一个圆柱上，中心是e半径为r。
	进一步的，he_1,he_2 在∂S上的方向是相反的，我们必须构建一个圆柱形部分能够bounded by h〖e_1 ||〗_r^+ 的反向边和h〖e_2 ||〗_r^+，以填补〖F_1 ||〗_r^+,〖F_2 ||〗_r^+ 的缝隙。因此〖E||〗_r^+ 能够构建一个〖F||〗_r^+ 的连续边界。（见Figure 8c）
	3、对一个有效的input model ∂S，每个点有一系列相邻边e_1,e_2,…,e_n 。在〖E||〗_r^+ 中关于e_i 在点v构建的圆柱体，一定有一条边在以v为中心半径为r的球体的great circle上。
	所有边都有一个统一的定向，所以我们可以利用来构建一个闭合loop。
	因此我们能够构建一个圆球区域bounded by反向边以密封e_i 之间的空隙。
	因此〖V||〗_r^+ 会构建一个〖E||〗_r^+ 的连续边界。（见Figure 8d）
注意offset mesh可能是退化的，如Figure 8a中平面上的边e_5 其相邻的两个面的法向是相同的，因此生成的圆柱区域就退化成一条线。
最后生成的〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 如Figure 8d，通过移除自交和重叠生成的轮廓surface如Figure 8e。

构建〖F||〗_r^+ 和〖E||〗_r^+ 是直接的，因为可以直接变化面F和截断E生成的圆柱体，而构造〖V||〗_r^+ 需要更多考量。
点v的邻接边e_1,e_2,…,e_n 可能是凹或凸的，这样构建〖V||〗_r^+ 是一个困难。
（23）提出了一个多边形tracing在多边形之间的卷积中，建立了三种类型的多边形tracing，分别在面、边、点区域。它说明了复杂的自交可能出现在点生成的球面。
在我们的方法中，我们标记这样复杂的点并见它们的offset mesh转变为有结构的采样点。两个filter被用于处理采样点。
Figure 9a中展示了一个复杂的点例子。

接下来介绍计算offset mesh中任意给定点的方法。
假设一个点v有邻边e_i，邻接两个面f_1,f_2 。
基于f_1,f_2 的法向定义的一个plane，可以构造一个great circle。在great circle的两条弧之间，loop edge Φ_ei 是小的那条，因为：
	如果e_i 是一个凸边或平边，f_1,f_2 之间的角度α满足0<α≤180。f_1,f_2 的正法向offset会组成一个角度∠n_1 vn_2=180−α，因此Φ_ei 小于一半的great circle。
	如果e_i 是一个凹边，有180<α<360，正法向offset得到的角度∠n_2 vn_1=α−180，因此Φ_ei 小于一半的great circle。
（23）中也给出了解释。另外，所有计算弧Φ_ei 会构造一个闭合的圆球loop Φ。（如Figure 9c）
圆球loop Φ可能会有许多自交，我们的关于构建这样的loop的方法如下：
	1、假设C_ei 是边Φ_ei 的中心，首先计算Φ的点O′基于其长度，即O^′=(∑_(i=1)^n▒〖|(|Φ_ei |)| C_ei 〗)/(∑_(i=1)^n▒〖||Φ_ei ||〗) 。
	然后投影O′到球体以获得点O。
	因此一个plane P相切于球体于点O被生成。
	然后映射Φ的所有的点V_i 到P，记为V_i′，通过确保所有点v,O,V_i,V_i′在同一个平面上，且OV_i′的长度和弧OV_i 相同。
	因此基于这样一个映射，圆球loop Φ被转化为一个2D平面loop Φ′。
	2、用一个类似于XOR多边形filling算法的三角化方法，来计算Φ′的三角化。
	先分别对每条边Φ_ei 构建一个三角化，如Figure
	 9c中的边界重建为Figure 9d。
	为了保证offset mesh的连续性，一些三角形O−ϕ_ei
	可能需要flipped，取决于其Φ_ei 到O的方向。因此对点v构建的offset mesh可能有重叠的surface。
	3、平面的三角化是准确的，并mapped back为圆球。
	三角化的refinement需要被控制，以保证和O−Φ_ei
,〖V||〗_r^+ 和〖E||〗_r^+  的loop edge有相同的边界三角化。（如Figure 9e）
	4、Section 5所述，offset mesh被转化为采样点，并且与重叠面关联的点因此会被一个segment filter过滤。
	基于XOR多边形filling算法的性质。即采样点p是inside或outside根据ray casting value。两个重叠面如果有flipped法向，p的ray casting value不会改变。
	5、另外，需要确定所有采样点P_i 是被哪个复杂点v构建的（复杂点：即v既有凸边也有凹边），并将它们分类为有效或无效点，根据其关联面法向与vP_1 是相同或相反。
	这样的一个分类会在移除内部shell用到，即包含无效点和复杂点。
基于处理过的点，一个流形offset轮廓可以被构建出来（如Figure 9f），结果轮廓可以进一步下降面数如Figure 9g。
下一Section讨论如何移除offset mesh 〖F||〗_r^+∪〖E||〗_r^+∪〖V||〗_r^+ 的自交和重叠surface。
