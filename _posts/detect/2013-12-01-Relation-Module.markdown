---
title: Relation-Module（Microsoft, CVPR, 2018）
date: 2018-08-27 21:00:00
categories: fDetect
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

论文： Hu H, Gu J, Zhang Z, et al. Relation Networks for Object Detection[J]. 2017.

Github: [https://github.com/msracver/Relation-Networks-for-Object-Detection](https://github.com/msracver/Relation-Networks-for-Object-Detection)

### 论文算法概述

    目前优秀的检测算法仍依赖于单独对目标的识别，在训练时并没有利用多个目标之间关系。这篇论文主要提出了一个目标关系模块（object relation module），通过目标的外观和几何特征的相互作用来同时处理一组目标。该模块可看成是一个优化的注意力模块，将原始的注意力权重扩展成了两部分；原始权重和一个新的几何权重。后者对目标之间的空间关系进行建模，仅考虑之间的相互几何关系。使多个目标在识别时同时处理相互影响，而不再是分开单独处理。同时还提出了一种可代替NMS的去重模块，避免了NMS的手动参数设置。
	      
### Object Relation Module

   首先回顾一个基础的注意力模块--Scaled Dot-Product Attention，其输入包含queries、维度为d_k的keys和维度为d_v的value。点积是应用在query和所有keys上以得到他们之间的相似度，并用一个softmax函数去获取values上的权重。输入为一个query、所有keys（已打包成一个矩阵K）和values（已打包成一个矩阵V），而输出值是输入值的加权平均值，<img src="{{ site.baseurl }}/images/pdDetect/rm1.png">。
   
   现在介绍如何计算目标关系，令一个目标包含它的几何特征f_G和外观特征f_A。在这里f_G为一个简单的四维度的边框，而f_A是重点在后面介绍。给定一系列目标<img src="{{ site.baseurl }}/images/pdDetect/rm2.png">，而关系特征f_R(n)是第n个目标在整个目标集合的关系特征，其计算方式为<img src="{{ site.baseurl }}/images/pdDetect/rm3.png">，输出为来自其他目标的外观特征经W_V变换后的加权和。
   
   关系权重<img src="{{ site.baseurl }}/images/pdDetect/rm4.png">，里面的分母是对分子的归一化，第m个目标对第n个目标产生影响的权重是由外观和几何共同决定的。

   外观权重<img src="{{ site.baseurl }}/images/pdDetect/rm5.png">，表示分别将下标为m和下标为n的目标映射到低维空间，用点积来得到它们之间的相似度。其中W_K和W_Q都是一个矩阵，起的作用跟K和Q一样，将原始特征f_mA和f_nA投影到子空间以度量其匹配程度，投影到的维度为d_k。
   
   几何权重的计算为：<img src="{{ site.baseurl }}/images/pdDetect/rm6.png">。总共分两步，首先将两个目标的几何特征嵌入到高维特征向量中，表示为E_G(f_mG, f_nG)。为令其具有平移和缩放不变性，使用了一个四维的相对几何特征<img src="{{ site.baseurl }}/images/pdDetect/rm7.png">（因为是相对的并且归一化的），并将这四维特征映射到一个高维特征向量中，维度为d_g。然后与W_G点积调整维度，后通过relu。
   
   通常目标关系模块会整合多个关系特征，用concat来合并：<img src="{{ site.baseurl }}/images/pdDetect/rm8.png">。整体也如下图所示：
   
   <center><img src="{{ site.baseurl }}/images/pdDetect/rm9.png"></center>
   
### Relation for Instance Recognition

   上面提到的目标关系模块可以直接应用到如Faster RCNN等目标检测算法的第二阶段上，原始如下图左，应用后如下图右，而不会改变特征维度。其中的r1和r2表示关系模块重复运行的次数。整体有也如下图3(a)所示。
   
   <center><img src="{{ site.baseurl }}/images/pdDetect/rm10.png"></center>
   
### Relation for Duplicate Removal

   这是论文的另一个贡献，可替代NMS。NMS为一种贪婪算法，且需要手动设置参数，因此很容易会陷入局部最优。该论文的替代方案是将其看成二分类问题，对每个GT目标，只有一个检测结果与之匹配，认识正确。其他与之匹配的都认为是重复的。该模块的输入时instance recognition模块的输出，含一系列目标，每个目标带有1024维特征向量、分类评分s0和边界框，而模块的输出是一个0到1的二分类概率s1。该网络如下图3(b)所示，分以下几步：

1. 首先将所有目标属该类别的概率score进行排序，则对于某一目标n，取出其在排序下的序号（即rank值）来代替score。

2. 将rank值映射到128维，同时也将该目标的1024维特征映射到128维，将两个128维特征相加后一起作为新的外观特征；

3. 结合该目标的边框信息，输入到一个关系模块中进行转换；

4. 将转换结果通过一个线性分类器和sigmoid来输出概率值s1。

5. 最终的输出评分为s0 x s1。

   也相当于将NMS整合进了网络中，使整个检测器变成一个高效地能够端到端利用多个信息（边界框、原始外观特征和分类评分）进行训练的检测器。
   
<center><img src="{{ site.baseurl }}/images/pdDetect/rm11.png"></center>

### Experiments

<center><img src="{{ site.baseurl }}/images/pdDetect/rm12.png"></center>

<center><img src="{{ site.baseurl }}/images/pdDetect/rm13.png"></center>

<center><img src="{{ site.baseurl }}/images/pdDetect/rm14.png"></center>