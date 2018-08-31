---
title: 基于sensitivity measure的下采样方法(2015)
categories: 
- 论文笔记之L-GEM
tags: 
- L-GEM与不平衡学习
updated: 2018-05-07
---
 <img src="{{ site.url }}/assets//blog_images//基于sensitivity measure的下采样方法(2015)/title.png" width="600px" height="400px"/>

&nbsp;[原文链接](https://ieeexplore.ieee.org/document/6971101/)

# 1.简介

处理不平衡分类问题，常用的采样方法有上采样和下采样两种。上采样方法生成少数类数据，容易造成过拟合问题，而下采样方法虽然训练数据都是真实数据，但是传统的方法往往忽略数据分布，使得分类器的范化性能下降。本文提出一种基于SM (sensitivity measure) 的下采样的训练方法 (DSUS)，其基本思想是，**每次迭代的时候，先通过聚类方法选出多数类的代表性样本(从而不破坏原本的数据分布)，然后在多数类代表性样本以及少数类样本中分别选出前k个SM值最高的样本，组成训练集，对分类器进行训练。**

# 2.算法

 <img src="{{ site.url }}/assets//blog_images//基于sensitivity measure的下采样方法(2015)/a.png" width="600px" height="400px"/>
 
 
  其中，t是训练的迭代次数，t的值最大等于k (少数类总样本数开根号)。从上图我们可以看出，每一轮迭代的时候，都有2k个类别平衡的样本补充进训练集，对网络进行训练。而这2k个样本的挑选，则根据当前网络计算出的样本的SM值挑选而来 (挑选SM值大的)。k个样本来自于经过聚类的多数类的代表性样本 (这里多数类聚类的簇的个数等于当前迭代中少数类样本的个数;代表性样本则是距离每个簇中心最近的那个样本)，另外k个样本则来自于当前迭代中的少数类样本中SM值最大的前k个。
 
 <img src="{{ site.url }}/assets//blog_images//基于sensitivity measure的下采样方法(2015)/b.png" width="600px" height="400px"/>
 
  本文这里每一轮训练RBFNN的方法为传统的二阶段方法：第一阶段，基于训练数据的分布信息，计算出隐层神经元的中心及宽度;第二阶段，则利用训练数据，采用伪逆法，计算出连接权重值。
    
  最早提出L-GEM的那篇文章[1]介绍的SM的计算是考虑整个训练样本的，并没有介绍单个训练样本的SM值如何计算。下面介绍单个训练样本Xb的SM值的计算[2]：

 <img src="{{ site.url }}/assets//blog_images//基于sensitivity measure的下采样方法(2015)/c.png" width="600px" height="400px"/>

 <img src="{{ site.url }}/assets//blog_images//基于sensitivity measure的下采样方法(2015)/d.png" width="600px" height="400px"/>
 
  <img src="{{ site.url }}/assets//blog_images//基于sensitivity measure的下采样方法(2015)/e.png" width="600px" height="400px"/>

# 3.实验

&nbsp;&nbsp;&nbsp;&nbsp;实验部分的细节这里不做赘述，具体看原论文。这里主要介绍下原论文都做了哪些实验，以期给自己写论文做实验时能有一些启发。  
&nbsp;&nbsp;&nbsp;&nbsp;论文首先在人造数据上依次做了**采样可视化实验**(比较不同的方法采样的样本能否服从原始样本分布)，**少数类多数类样本重叠实验**(重叠率20%/40%/60%/80%/100%)，**有无清晰决策边界以及可靠聚类中心实验**(SM依赖于决策边界，如果决策边界不清晰，那么计算出的SM将可能会有一定的误导性;DSUS算法依赖于聚类中心找到代表性样本，以期维持数据原始分布，如果找的的聚类中心不靠谱，那么DSUS的性能可能也会收到影响)。最后在**真实数据UCI数据上比较了不同算法的优劣**(对比的算法有随机采样算法，集成算法，在决策边界采样的算法;评价指标有F值，AUC以及G-mean)。  
&nbsp;&nbsp;&nbsp;&nbsp;从实验结果上看，DSUS采样出的数据基本服从数据的原始分布，而当少数类多数类样本重合率低于80%时，DSUS效果比其他算法要好，同时，如若数据本身无清晰决策边界或者其分布导致聚类算法没法找出可靠的聚类中心来代表样本分布，那么DSUS算法的性能也将会打折扣。而在UCI数据上，DUSUS在60.71%的实验中，效果达到最优。除此之外，由于DSUS是通过SM挑选样本，这意味着DSUS将很可能挑选出少数类的异常样本，对分类器学习不利。
  
# 4.参考资料
[1] W.W.Y. Ng, D.S. Yeung, D. Wang, E.C.C. Tsang and X-Z Wang, “Localized Generalization Error and Its Application to RBFNN Training”, Proc. of Intl. Conf. on Machine Learning and Cybernetics, pp. 4667- 4673, 2005 

[2] B. Sun, W. W. Y. Ng, D. S. Yeung, and P. P. K. Chan, “Hyper-parameter selection for sparse LS-SVM via minimization of its localized general-ization error,” Int. J. Wavelets Multiresolut. Inf. Process., vol. 11, no. 3, 2013, Art. ID 1350030.