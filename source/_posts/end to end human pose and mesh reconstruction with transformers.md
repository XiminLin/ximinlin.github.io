---
title: end to end human pose and mesh reconstruction with transformers
categories: [paper review]
tags: [computer vision, 3d human pose, 3d human shape, transformers]
comment: true
sticky: 0
---



# 1. 当前方向

## todo:
1. 了解 3d 人体的各种表示方法
2. 了解 SMPL 等方法
3. 了解 直接 regress 的方法
4. 了解 masked vertex modeling



## 1.1 parametric model:
利用 ==提前定义好的 parametric model== 像是 SMPL 来 predict shape 和 pose coefficients

而且提出可以 ==利用其他的检测信息==，类似 skeleton , segmentation map, 其他的 optimization strategies 和 利用 temporal info 来 提升效果

1. 优点：
	1. 有 strong prior about human body, 所以对于 environment 比较 robust
2. 缺点：
	1. 受到 定义的 parametric model 的限制，不能很好的 scale 到其他的 task 上，类似 hand-reconstruction 这种

## 1.2 直接 regress vertex:

考虑利用 ==3D mesh, volumetric space, 或者 occupancy field 来表示3D人体==

用 Graph CNN 来估计 vertex-vertex interaction, 或者直接估计 vertex coordinates

1. 优点：
	1. 能很好的 scale 到 别的 reconstruction task 上，只要提供足够的数据
2. 缺点：
	1. 不是很robust，对于遮挡问题，而且只能找到 local vertex 的关系，但是 non-local vertex 的 关系找不到

---

# 2. 方法

## 2.1 workflow

1. image -> CNN 得到 2048-dim vector representation. 然后 和 template 的 joint queries 和 template vertex queries 合并起来。==(CNN 是 pretrained on ImageNet classification task)==

2. 对于每一个要预测的3d点(joint + vertex), 把 一个3d query 和 上面的 feature 合并，然后得到 num_queries x 2051 的 matrix 给 Transformer 做 input.

3. 期间加上 Masked Vertex Modeling (MVM), 类似 dropout. 然后 让 Transformer 预测 全部的 joint 和 vertex.

4. transformer 因为 output size 不变，所以这里用 cascade of transformers, 两个之间使用 dimension reduction 的方法 ==(用 encoder 来把 dimension 降下来)==

## 2.2 loss

1. $L_v$: 3d vertex 之间的 L1 loss
2. $L_j$: 3d joint 之间的 L1 loss
3. $L_j^{reg}$: 存在 regression matrix G, 能从 vertex 里面计算出 joint 坐标，这里就是L1 loss of GT joint 和 用 G 算出来的 joint
4. $L_J^{proj}$: L1 loss of GT 2d joint point 和 投影出来的 2d joint point ==(这里在 transformer output 上加了一层 linear layer 能自动学习 camera params)==

最后是 $L = \alpha(L_v + L_j + L_j^{reg}) + \beta L_J^{proj}$

==alpha/beta 是 binary flag==, 看看当前的 dataset 有没有 ==3d 或者 2d ground truth.==


## 2.3 additional

使用 mixed training 2d + 3d (????), todo...

---


# 3. experiments

## 3.1 datasets:

**Human3.6M**: 2d + 3d annotations, 但是 3d mesh 有 license 不能用，转而使用 pseudo 3D meshes provided in papers: 
1. pose2mesh, ECCV, 2020	
2. I2Lmashnet, ECCV, 2020

**3DPW**: outdoor 2d + 3d annotations, training images 22K, testing 35K.

**UP-3D**: outdoor image data, 3d annotations 是 model fitting 创造的, 7K training images.

**MuCo-3DHP**: synthesized data from MPI-INF-3DHP dataset, 有很多 real-world background images, 总共 200K training images.

**COCO**: 3d data 是 papers 生成的:
1. learning to reconstruct human body pose and shape shape via model-fitting in the loop, ICCV, 2019
	1. 这里使用了 Simplify-X 来 fit 的: expressive body capture: 3d hands, face, and body from a single image, CVPR, 2019

**MPII**: outdoor image with 2d poses, 14K training images

**FreiHAND**: 3d hand dataset, 130K training, 4K testing.

---

## 3.2 Metrics:

**MPJPE**: mean-per-joint-positioning-error ==(用于3d pose)==: euclidean distance between GT joint 和 predicted joints

**PA-MPJPE**: ==(一般用于 reconstruction error)==, 用 ==Procrustes Analysis(PA)== 来做 3d alignment，然后再算 MPJPE, 因为和 scale 和 rigid pose 没关系

**MPVE**: mean-per-vertex-error, euclidean distance between GT vertex 和 predicted vertex

---

# Results:

和众多 SOTA 比，主要使用 Human3.6M (?????) 来对比，都比 SOTA 好

