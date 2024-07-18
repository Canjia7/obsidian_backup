> @ARTICLE{10458949,
  author={Chen, Mengqi and Zhou, Laishui and Chen, Honghua and Wang, Jun},
  journal={IEEE Transactions on Instrumentation and Measurement}, 
  title={Thin-Walled Aircraft Panel Edge Extraction From 3-D Measurement Surfaces via Feature-Aware Displacement Learning}, 
  year={2024},
  volume={73},
  number={},
  pages={1-11},
  keywords={Image edge detection;Feature extraction;Aircraft;Point cloud compression;Three-dimensional displays;Vectors;Learning systems;3-D measurement;edge extraction;feature-aware displacement learning;point cloud processing;thin-walled aircraft panel},
  doi={10.1109/TIM.2024.3373087}}
# 0 Abstract
为了能够为沿着thin-walled飞机面板进行精细加工生成可靠的tool paths，我们引入了一个feature-aware displacement学习框架，以精确提取飞机面板的edges
具体来说，我们设计了一个dual-task神经网络，命名为APEE-Net（aircraft panel edge extraction network）
该网络具有识别落在飞机面板的edges附近的点，以及预测displacement vectors指向局部edge features的dual目的
识别出的edge points后续用这些displacement vectors重定位，从而提取出精确的edge points
我们提出的方法在训练阶段加入了feature-aware displacement optimization loss，显著提高了该方法在处理有噪声的尖锐几何features时的鲁棒性和准确性
大量的实验表明，我们的方法在准确性方面优于现有的提取方法
通过实际的机器应用，进一步验证了该方法的可行性和现实世界的适用性
# 1 Introduction
# 2 Related Work
## A. Traditional Methods for Edge Detection
## B. Learning-Based Methods for Edge Detection
# 3 Method
## A. Overview
## B. Training Data Preparation
## C. Point Classification Module
## D. Feature-Aware Displacement Regression Module
## E. Joint Loss Function
## F. Implementation Details
# 4 Experiments and Discussion
## A. Ablation Studies
## B. Robustness Studies
## C. Comparison With Competitors
## D. Generalization Analysis
## E. Limitations
# 5 Application
# 6 Conclusion