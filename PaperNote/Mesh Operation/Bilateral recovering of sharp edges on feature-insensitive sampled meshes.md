![[Pasted image 20240703231047.png]]
> @ARTICLE{1634326,
  author={Wang, C.C.L.},
  journal={IEEE Transactions on Visualization and Computer Graphics}, 
  title={Bilateral recovering of sharp edges on feature-insensitive sampled meshes}, 
  year={2006},
  volume={12},
  number={4},
  pages={629-639},
  keywords={Robustness;Filtering;Computer graphics;Application software;Shape;Sampling methods;Computer errors;Filters;Smoothing methods;Noise reduction;Boundary representations;geometric algorithms;languages;systems.},
  doi={10.1109/TVCG.2006.60}}
# 0 Abstract
各种计算机图形学应用程序在regular grid中采样3D shapes的surface，而不使sampling rate适应于surface curvature或sharp features
插值或近似这些samples的triangular meshes通常在insensitive的sampled sharp features表现出相对较大的误差
本文提出了一种鲁棒的通用方法，通过bilateral filters来恢复这种insensitive sampled triangular meshes上的sharp edges
受到为mesh smoothing和denoising的bilateral filtering令人印象深刻的结果，我们采用它来控制triangular meshes的锐化
在识别了嵌入了sharp features的区域后，我们通过bilateral filtering来恢复sharpness几何，然后迭代修改给定mesh的连通性，形成可以通过其二面角轻松检测到的singlewide sharp edges
结果表明，该方法可以在feature-insensitive sampled meshes上鲁棒地重建sharp edges