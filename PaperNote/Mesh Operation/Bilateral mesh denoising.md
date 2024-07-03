![[Pasted image 20240703225959.png]]
> @incollection{fleishman2003bilateral,
  title={Bilateral mesh denoising},
  author={Fleishman, Shachar and Drori, Iddo and Cohen-Or, Daniel},
  booktitle={ACM SIGGRAPH 2003 Papers},
  pages={950--953},
  year={2003}
}
# 0 Abstract
我们提出了一种anisotropic mesh denoising算法，高效、简单、快速
是通过使用局部邻域在法向上过滤mesh的顶点来实现的
受到bilateral filtering对image denoising的显著效果，我们采用其来denoise 3D meshes，解决从二维到三维流形转换所需的具体问题
结果表明，该方法在保持特征的同时成功去除了mesh中的noise
此外，该方法在概念和实现上都很简单