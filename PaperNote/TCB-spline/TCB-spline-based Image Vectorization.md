> @article{10.1145/3513132,
author = {Zhu, Haikuan and Cao, Juan and Xiao, Yanyang and Chen, Zhonggui and Zhong, Zichun and Zhang, Yongjie Jessica},
title = {TCB-spline-based Image Vectorization},
year = {2022},
issue_date = {June 2022},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {41},
number = {3},
issn = {0730-0301},
url = {https://doi.org/10.1145/3513132},
doi = {10.1145/3513132},
abstract = {Vector image representation methods that can faithfully reconstruct objects and color variations in a raster image are desired in many practical applications. This article presents triangular configuration B-spline (referred to as TCB-spline)-based vector graphics for raster image vectorization. Based on this new representation, an automatic raster image vectorization paradigm is proposed. The proposed framework first detects sharp curvilinear features in the image and constructs knot meshes based on the detected feature lines. It iteratively optimizes color and position of control points and updates the knot meshes. By using collinear knots at feature lines, both smooth and discontinuous color variations can be efficiently modeled by the same set of quadratic TCB-splines. A variational knot mesh generation method is designed to adaptively introduce knots and update their connectivity to satisfy the local reconstruction quality. Experiments and comparisons show that our framework outperforms the existing state-of-the-art methods in providing more faithful reconstruction results. In particular, our method is able to model undetected features and subtle or complicated color variations in-between features, which the previous methods cannot handle efficiently. Our vectorization representation also facilitates a variety of editing operations performed directly over vector images.},
journal = {ACM Trans. Graph.},
month = {jun},
articleno = {34},
numpages = {17},
keywords = {mesh optimization, knot placement, simplex splines, Vector images}
}
# 0 Abstract
# 1 Introduction
# 2 Related Work
# 3 TCB-SPLINES
## 3.1 Simplex Spline
## 3.2 T-configs and TCB-splines
# 4 Algorithm overview
# 5 Mesh parametrization
## 5.1 Feature Detection
## 5.2 Feature Fairing
## 5.3 Color Correction
## 5.4 Image Mesh Parametrization
# 6 Knot Mesh Generation
## 6.1 Initial Knot Placement
## 6.2 Knot Mesh Optimization
# 7 Image Approximation
## 7.1 TCB-splines with DIscontinuous Functions
## 7.2 Geometry Approximation
## 7.3 Color Approximation
# 8 Apaptive Refinement
## 8.1 New Knot Insertion
## 8.2 Knot Mesh Local Optimization
# 9 Experimental Results
## 9.1 Comparison to Patch-based Methods
## 9.2 Comparison to Diffusion Curve-based Methods
## 9.3 Comparison to Subdivision Surface-based Methods
## 9.4 Statistics
## 9.5 Editing
## 9.6 Authoring
## 10 Conclusions