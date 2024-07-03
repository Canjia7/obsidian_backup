![[Pasted image 20240703230623.png]]
> @incollection{jones2003non,
  title={Non-iterative, feature-preserving mesh smoothing},
  author={Jones, Thouis R and Durand, Fr{\'e}do and Desbrun, Mathieu},
  booktitle={ACM SIGGRAPH 2003 Papers},
  pages={943--949},
  year={2003}
}
# 0 Abstract
随着越来越多地使用几何扫描仪来创建3D模型，越来越需要快速和鲁棒的mesh smoothing来消除measurements中不可避免的noise
随着大多数先前的工作倾向于diffusion-based迭代技术来保持feature-preserving smoothing，但我们提出了一种完全不同的方法，基于robust statistic和surface的local first-order predictors
我们的局部估计的鲁棒性使我们能够推导出一种适用于任意triangle soups的non-iterative feature-preserving filtering技术
我们证明了其实现的简单性和效率，使其成为对大型、nosiy、non-manifold meshes的smoothing的优秀解决方案