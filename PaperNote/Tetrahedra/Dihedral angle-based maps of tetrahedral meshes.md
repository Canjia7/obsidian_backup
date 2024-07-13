> @article{10.1145/2766900,
author = {Paill\'{e}, Gilles-Philippe and Ray, Nicolas and Poulin, Pierre and Sheffer, Alla and L\'{e}vy, Bruno},
title = {Dihedral angle-based maps of tetrahedral meshes},
year = {2015},
issue_date = {August 2015},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {34},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/2766900},
doi = {10.1145/2766900},
abstract = {We present a geometric representation of a tetrahedral mesh that is solely based on dihedral angles. We first show that the shape of a tetrahedral mesh is completely defined by its dihedral angles. This proof leads to a set of angular constraints that must be satisfied for an immersion to exist in R3. This formulation lets us easily specify conditions to avoid inverted tetrahedra and multiply-covered vertices, thus leading to locally injective maps. We then present a constrained optimization method that modifies input angles when they do not satisfy constraints. Additionally, we develop a fast spectral reconstruction method to robustly recover positions from dihedral angles. We demonstrate the applicability of our representation with examples of volume parameterization, shape interpolation, mesh optimization, connectivity shapes, and mesh compression.},
journal = {ACM Trans. Graph.},
month = {jul},
articleno = {54},
numpages = {10},
keywords = {volume parameterization, optimization, interpolation, compression}
}
# 0 Abstract
我们提出了一种完全基于dihedral angle的tetrahedral mesh的几何表示

我们首先证明了tetrahedral mesh的shape完全由它的dihedral angle来定义
这个证明引出了一组angular constraints，其必须满足immersion在$\mathbb{R}^3$中的条件
这个公式让我们很容易地指定条件，来避免inverted tetrahedra和multiply-covered vertices，从而导致局部injective maps

然后我们提出了一种constrained optimization方法，当input angles不满足constraints时，修正input angles

此外，我们开放了一种快速spectral reconstruction方法，来鲁棒地从dihedral angles恢复positions

我们用volume parameterization、shape interpolation、mesh optimization、connectivity shapes、mesh compression来证明我们的表示的适用性