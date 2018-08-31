---
title: L-GEM在learning based隐写分析中的应用(2014)
categories: 
- 论文笔记之L-GEM
tags: 
- L-GEM的应用
updated: 2018-05-08
---
 <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/title.png" width="600px" height="400px"/>

&nbsp;[原文链接](https://ac.els-cdn.com/S0020025514005805/1-s2.0-S0020025514005805-main.pdf?_tid=a93c705f-1667-4153-901e-28ca5d125a28&acdnat=1526630588_68895812c6eb9050afa5267cf8e6a80d)

# 1.简介

  &nbsp;&nbsp;&nbsp;隐写术(steganography)是信息安全中的一种技术，简单地说就是将一些私密传输信息隐藏在图像、视频、音频中进行传输。隐写分析技术(steganalysis)则是判断一个多媒体文件是否隐藏了私密传输信息。learning bsed的隐写分析技术主要分为两部分：特征提取和分类器。
  
  &nbsp;&nbsp;&nbsp;下面简单介绍下隐写和隐写分析的过程。由于JPEG类型的图片很常见，故而其是本文的讨论对象。对于一张原始图片，通常我们需要利用JPEG压缩标准的量化表(quantization table)将其压缩，然后再运用隐写术隐藏私密信息。当我们应用learning based隐写分析技术来判断一个JPEG图像是否隐层了私密信息，我们首先对图像进行隐写分析特征提取，而后将提取特征送入分类器，由分类器给出该JPEG图像是否隐藏了私密信息。具体如下图所示。

 <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/a.png" width="600px" height="400px"/>
 
 <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/b.png" width="600px" height="400px"/>
 
# 2.LG-Steganalyzer算法

 <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/c.png" width="600px" height="400px"/>
 
 &nbsp;&nbsp;&nbsp;本文这里L-GEM里面的Q值通过交叉验证确定。而考虑到RBFNN适合大数据场景以及其训练速度很快，故而本文采样RBFNN作为分类器。RBFNN训练分为结构选择以及权重微调两个阶段，具体如下：
 
  <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/d.png" width="600px" height="400px"/>

 &nbsp;&nbsp;&nbsp;对于权重微调，本文以降低训练误差和ST-SM作为优化目标，利用iRprop+方法对连接权重进行微调：
 
   <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/e.png" width="600px" height="400px"/>

# 3.实验

   <img src="{{ site.url }}/assets//blog_images//L-GEM在learning based隐写分析中的应用(2014)/f.png" width="600px" height="400px"/>

