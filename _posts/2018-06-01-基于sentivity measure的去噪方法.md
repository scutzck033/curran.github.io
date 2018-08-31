---
title: 基于sensitivity measure的去噪方法
categories: 
- 论文笔记之L-GEM
tags: 
- L-GEM与不平衡学习
updated: 2018-06-01
---
 <img src="{{ site.url }}/assets//blog_images//TRPS去噪/title.PNG" width="600px" height="400px"/>

&nbsp;[原文链接](http://nbviewer.jupyter.org/github/scutzck033/scutzck033.github.io/blob/master/docs/Zhang%20等。%20-%202015%20-%20Two-Phase%20Resampling%20for%20Noisy%20Imbalanced%20Multi-cl.pdf)

# 简介
针对不平衡分类问题，之前介绍过一个[基于ST-SM的下采样算法](https://scutzck033.github.io/论文笔记之l-gem/2018/05/07/基于sensitivity-measure的下采样方法(2015)/#)[1]，但是该算法无法应对当少数类样本存在噪声的情况。本文介绍一个算法框架TRPS，同时考虑了样本存在噪声以及类别不平衡两个问题。
# TPRS算法框架
论文提出的TPRS算法框架，分为两个阶段：第一个阶段为样本去噪，第二个阶段为样本重采样。经过这两个阶段处理后得到的数据，便为相对干净不含噪声且类别间样本数量较为平衡的数据。该论文的原创部分为第一阶段的去噪部分，第二阶段的重采样算法论文用的是经典的smote算法。这里具体介绍一下第一阶段的样本去噪是怎么做的。  
去噪的过程依赖于每个样本的ST-SM值。论文认为，当某个样本的ST-SM值大于某个阈值时，其便被认为是噪声样本。它的思想可以简单地如下解释：  
`首先我们假设了training samples和其perturbed samples分布应是类似的，即边缘分布和条件概率分布类似，那么其都应属于相同的类别。如果一个training sample的ST-SM值较大(如大于0.5)，那么说明分类器的对其perturbed samples的预测标签大多(超过一半)不同于training sample的真实标签，那么这种情况下，分类器对training sample的预测标签很大可能也是不同于其真实标签，即分类器把该training sample分错了。现在考虑label-noise的情形，如若该分类器是一个好的分类器，那么其分错的训练样本则大概率是噪声样本。`  
**这里值得注意的是，要计算样本正确的ST-SM值的前提是，我们首先要有一个靠谱的分类器。**（这篇文章与[1]的出发点不同，前者(本文)假设的是分类器是一个好的分类器，我们需要通过它来过滤样本；后者则假设样本是好的样本，分类器需要不断学习这些样本。）  

可能这里有人会想：既然是把分类器大概率分错的样本当成噪声样本去除，那么直接将分类器分错的样本去除不就好了，为什么还要通过计算ST-SM值再根据它来去除噪声样本？—— 根据笔者个人的理解，我们可以考虑如下图情形：
<img src="{{ site.url }}/assets//blog_images//TRPS去噪/example.PNG" width="600px" height="400px"/>

从上图我们可以看到，样本a非常靠近蓝色类的样本，虽然分类器把它分对了，但分类器明显对它过拟合了。这样在测试的时候，分类器很则可能由于样本a的缘故，把一部分蓝色类样本错分为黄色类，所以这里很明显，样本a是一个噪声样本，应该去除。然而，通过分错样本的方法去噪没法去除样本a，但是通过计算ST-SM值的方法，则有可能去除样本a(样本a的ST-SM值显然较大)，从而达到去噪的目的。

那么，如何得到一个靠谱的分类器呢？论文这里采用的是Balanced Bagging的方法。即对数据进行有放回采样，以此来训练分类器。但是，我们知道，有放回采样容易造成过拟合问题，这样的话，在噪声比例较高的情况下，噪声样本极有可能被当成正确样本加入训练，即使噪声比例不高，一部分正确样本也有可能由于没加入训练而使得分类器过分关注另一部分正确样本从而被当成了噪声样本。由此可见，只对原始不平衡数据重采样一次训练得到的分类器也不太靠谱，那怎么办？论文里采用的是集成的方法，即对原始数据进行多次重采样训练得到多个分类器，然后再平均通过它们计算的每个样本的ST-SM值。
 <img src="{{ site.url }}/assets//blog_images//TRPS去噪/a.PNG" width="600px" height="400px"/>

此外，为了进一步降低分类器的计算方差，论文并没有对整个数据集采用有放回采样（因为多类分类的话有可能使得学到的决策平面过于复杂，从而降低去噪准确率），而是采用one versus one的方法。整个去噪过程的伪代码如下：
 <img src="{{ site.url }}/assets//blog_images//TRPS去噪/b.PNG" width="600px" height="400px"/>
 
 其中，每个样本的最终ST-SM值由多个基分类器计算的ST-SM值平均而来：
<img src="{{ site.url }}/assets//blog_images//TRPS去噪/c.PNG" width="250px" height="120px"/>  
每个基分类器通过如下公式计算每个样本的ST-SM值：
<img src="{{ site.url }}/assets//blog_images//TRPS去噪/e.PNG" width="250px" height="120px"/>  
其中，x(i)为被计算样本Q-neighbor里通过均匀采样所得的样本，预测函数h的取值为0或1。
# 实验
 
 论文实验部分的评价指标为一些非参数检验方法及G-mean：<img src="{{ site.url }}/assets//blog_images//TRPS去噪/f.PNG" width="200px" height="100px"/>
 
 论文在9个UCI数据上加入不同程度的lable-noise做了比较实验，同时也在人体动作识别数据上做了比较实验：
  <img src="{{ site.url }}/assets//blog_images//TRPS去噪/d.PNG" width="600px" height="400px"/>
 
 笔者也对该论文进行了复现，github地址为：[https://github.com/scutzck033/TRPS](https://github.com/scutzck033/TRPS)  
 笔者在new_throyid这份UCI数据上进行了测试，发现论文提出的去噪方法可以准确去除90%以上的噪声样本，同时笔者也对处理前和处理后的数据进行了可视化，其中，rebalanced算法为smote算法，k为3：
  <img src="{{ site.url }}/assets//blog_images//TRPS去噪/raw_data.PNG" width="600px" height="400px"/>
  <img src="{{ site.url }}/assets//blog_images//TRPS去噪/raw_rebalanced_data.PNG" width="600px" height="400px"/>
  <img src="{{ site.url }}/assets//blog_images//TRPS去噪/filtered_data.PNG" width="600px" height="400px"/>
  <img src="{{ site.url }}/assets//blog_images//TRPS去噪/filtered_rebalanced_data.PNG" width="600px" height="400px"/>
# 参考文献
[1] Ng W W Y, Hu J, Yeung D S, et al. Diversified sensitivity-based undersampling for imbalance classification problems[J]. IEEE transactions on cybernetics, 2015, 45(11): 2402-2412.