---
tags:
  - mesh_generation
  - boundary_conformity
  - constrained_Delaunay_tetrahedralization
  - tetrahedralization
  - numeric_robustness
---
![[Pasted image 20240615111146.png]]
> @article{10.1145/3618352,
author = {Diazzi, Lorenzo and Panozzo, Daniele and Vaxman, Amir and Attene, Marco},
title = {Constrained Delaunay Tetrahedrization: A Robust and Practical Approach},
year = {2023},
issue_date = {December 2023},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {42},
number = {6},
issn = {0730-0301},
url = {https://doi.org/10.1145/3618352},
doi = {10.1145/3618352},
abstract = {We present a numerically robust algorithm for computing the constrained Delaunay tetrahedrization (CDT) of a piecewise-linear complex, which has a 100\% success rate on the 4408 valid models in the Thingi10k dataset.We build on the underlying theory of the well-known tetgen software, but use a floating-point implementation based on indirect geometric predicates to implicitly represent Steiner points: this new approach dramatically simplifies the implementation, removing the need for ad-hoc tolerances in geometric operations. Our approach leads to a robust and parameter-free implementation, with an empirically manageable number of added Steiner points. Furthermore, our algorithm addresses a major gap in tetgen's theory which may lead to algorithmic failure on valid models, even when assuming perfect precision in the calculations.Our output tetrahedrization conforms with the input geometry without approximations. We can further round our output to floating-point coordinates for downstream applications, which almost always results in valid floating-point meshes unless the input triangulation is very close to being degenerate.},
journal = {ACM Trans. Graph.},
month = {dec},
articleno = {181},
numpages = {15},
keywords = {numeric robustness, representability, volume meshing}
}