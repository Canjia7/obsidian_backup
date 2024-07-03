![[Pasted image 20240703225540.png]]
> @inproceedings{10.1145/1629255.1629258,
author = {Sheung, Hoi and Wang, Charlie C. L.},
title = {Robust mesh reconstruction from unoriented noisy points},
year = {2009},
isbn = {9781605587110},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/1629255.1629258},
doi = {10.1145/1629255.1629258},
abstract = {We present a robust method to generate mesh surfaces from unoriented noisy points in this paper. The whole procedure consists of three steps. Firstly, the normal vectors at points are evaluated by a highly robust estimator which can fit surface corresponding to less than half of the data points and fit data with multi-structures. This benefits us with the ability to well reconstruct the normal vectors around sharp edges and corners. Meanwhile, clean point cloud equipped with piecewise normal is obtained by projecting points according to the robust fitting. Secondly, an error-minimized subsampling is applied to generate a well-sampled point cloud. Thirdly, a combinatorial approach is employed to reconstruct a triangular mesh connecting the down-sampled points, and a polygonal mesh which preserves sharp features is constructed by the dual-graph of triangular mesh. Parallelization method of the algorithm on a consumer PC using the architecture of GPU is also given.},
booktitle = {2009 SIAM/ACM Joint Conference on Geometric and Physical Modeling},
pages = {13–24},
numpages = {12},
keywords = {GPU, noisy points, parallel computing, robust approach, surface reconstruction},
location = {San Francisco, California},
series = {SPM '09}
}
# 0 Abstract
我们提出了一种鲁棒的算法，来从无序的noisy points中生成mesh surfaces
整个过程包括三个步骤：
1. 点的法向通过一个高度鲁棒的estimator来估计，其能根据不到一半的数据点来拟合surface，并且可以拟合multi-structures，这使得我们能够很好地重建sharp edges和corners。同时，根据鲁棒拟合方法对点进行投影，得到具有piecewise normal的干净点云
2. 采用最小化误差的subsampling方法生成well-sampled点云
3. 采用一个组合的方法来重建一个triangular mesh，连接down-sampled points，并利用triangular mesh的dual-graph构造出保持sharp features的polygonal mesh
给出了在消费型PC上使用GPU的并行化算法