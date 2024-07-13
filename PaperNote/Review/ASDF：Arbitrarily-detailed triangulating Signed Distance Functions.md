![[Pasted image 20240712113930.png]]
> PG_2024 - paper1528 
# 0 Abstract
signed distance functions（SDF）的重要性最近显著上升，因为它们构成了基于scans或photogrammetry的deep learning reconstruction methods的output
Marching Cubes（MC）是一种经典且快速的算法，经过深度学习变体的改进，但速度较慢且仍然会生成artifacts
最近的Reach for the Spheres method设法在小的grid size上重建更高的细节，但需要一个topologically correct prior来修复genus，并且会更慢
我们提出了一种基于Boissonnat and Oudot的meshes的方法，可以从任意surface构建triangulated meshes，其提供了intersection oracle
我们的方法还受益于它们对重建meshes对特定采样条件的homeomorphy和error bounds的理论保证
我们的SDF重建算法不仅比最先进的方法快得多，而且可以通过在cell内而不是在edge上sub-sampling来提供SDF的任意细节
这导致更高的精度和更少的内存使用，同时仍然留给用户控制一个参数来决定速度和内存之间的权衡
# 1 Introduction
为了更好地捕获这些精细的feature，我们将在比grid cell level更细致的intervals来evaluate SDFs，并展示了我们因此能够超过从均匀grid SDFs（对神经网络中的三维形状表示至关重要）最先进的triangulated surface reconstruction

本文的贡献：
1. 从SDFs的surface reconstruction是robust、manifold、重建任意genus，以及附加理论保证
2. 更准确的reconstruction在medium和higher的grid sizes
3. 更快的reconstruction，使用更少的内存，具有有竞争力的质量
4. 一个参数允许在运行时间和精度之间进行权衡
# 2 Related Work
### Reconstructing a triangle mesh from an SDF
### Meshing Surfaces with Triangles
