>@article{10.1145/3197517.3201353,
author = {Hu, Yixin and Zhou, Qingnan and Gao, Xifeng and Jacobson, Alec and Zorin, Denis and Panozzo, Daniele},
title = {Tetrahedral meshing in the wild},
year = {2018},
issue_date = {August 2018},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {37},
number = {4},
issn = {0730-0301},
url = {https://doi.org/10.1145/3197517.3201353},
doi = {10.1145/3197517.3201353},
abstract = {We propose a novel tetrahedral meshing technique that is unconditionally robust, requires no user interaction, and can directly convert a triangle soup into an analysis-ready volumetric mesh. The approach is based on several core principles: (1) initial mesh construction based on a fully robust, yet efficient, filtered exact computation (2) explicit (automatic or user-defined) tolerancing of the mesh relative to the surface input (3) iterative mesh improvement with guarantees, at every step, of the output validity. The quality of the resulting mesh is a direct function of the target mesh size and allowed tolerance: increasing allowed deviation from the initial mesh and decreasing the target edge length both lead to higher mesh quality.Our approach enables "black-box" analysis, i.e. it allows to automatically solve partial differential equations on geometrical models available in the wild, offering a robustness and reliability comparable to, e.g., image processing algorithms, opening the door to automatic, large scale processing of real-world geometric data.},
journal = {ACM Trans. Graph.},
month = {jul},
articleno = {60},
numpages = {14},
keywords = {mesh generation, robust geometry processing, tetrahedral meshing}
}