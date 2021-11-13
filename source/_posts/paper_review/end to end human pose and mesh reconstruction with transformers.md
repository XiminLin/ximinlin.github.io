---
title: end to end human pose and mesh reconstruction with transformers
categories: 
  - paper review
tags: 
  - computer vision
  - 3d human pose and shape
  - transformers
comment: true
mathjax: true
sticky: 0
---



# 1. Intro



### 1.1 parametric model:

利用 ==提前定义好的 parametric model== 像是 **SMPL**，最新的是 **STAR** 来直接 regress params

1. 有 **strong prior about human body**, 所以对于 **pixels environment 比较 robust**

2. 受到 **定义的 parametric model 的限制**，**因为是从 limited examples 里面 learned 出来的，所以作者认为不能很好 generalize**

### 1.2 直接 regress vertex:

利用 ==**3D mesh, volumetric space, 或者 occupancy field** 来表示3D人体==

第二种常见的方法就是 **GCNN to model neighbor vertex-vertex interactions**，或者 **1D heatmap to regress vertex coordinates**. 但是作者认为这种方法**不能 capture non-local vertex interaction**

而且现对于 SMPL 来说就不是 很 robust 了



### 1.3 inverse kinematic technology

In computer graphics and robotics, inverse kinematics techniques[2] have been developed to estimate the internal joint positions of an articulated figure given the position of an end effector such as a hand tip.

> inverse kinematic techniques 我的理解是 机器手，给定机器手的顶端的位置，就能直接 estimate 每个 joint 的 位置，
>
> 就说明各个 joints 之间存在 strong interaction



# 2. related works

### 2.1 loss 方面

Since it is challenging to regress the pose and shape coefficients directly from an input image, recent works further propose to leverage various human body priors such as human skeletons [21, 34] or segmentation maps [29], and explore different optimization strategies[19, 17, 46, 12] and temporal information [18] to improve reconstruction.

> 就是说 loss 里面结合 temporal，skeleton, segmentation maps 之类的方法来 improve reconstruction. 



### 2.2 non-prior representation

researchers have also proposed approaches todirectly regress 3D human body shape from an input image. For example, researchers have explored to represent human body using a 3D mesh [20, 7], a volumetric space [47], or an occupancy field [41, 42]. 

>不用 prior models，直接 regress 的话 用来表示 body shape 的有：
>
>1. 3D mesh; 
>2. volumetric space; 
>3. occupancy filed



# 3. methods



先是有 **CNN backbone** 得到 2048-dim vector representation. (<u>CNN 是 pretrained on ImageNet classification task</u>), 就生成了下面的 左边的 **粉红色的框**

Transformer **是需要queries的(类似于 posititonal encoding)**, 对于每一个要预测的 3d点 (joint or vertex), 把 一个3d query 和 上面的 feature 合并，然后得到 num_queries x 2051 的 matrix 给 Transformer 做 input.

Transformer 因为 output size 不变，所以这里用 cascade of transformers, 两个之间使用 **dimension reduction 的方法**, 来最终得到 **3D outputs*

> 我的理解是 这个reduction可以是 MLP来做，或者更简单就直接是 pooling

input to transformer 是 joint 和 mesh vertex 的 query，这里用了一个 template human mesh model 来 preserve positional information of each query. 

> 我的理解是这种 mesh template 可变参数更多，对比 SMPL 更 general

每个点都是  3D coordinate 作为 positional encoding, 这里直接 和 image feature concat 在一起，所以每个 query 都是 (img_feat) 2048 + 3 = 2051 features，3 就是 positional encoding of query

就得到了 joint queries $Q_J = \{q_1^J, q_2^J, ... q_n^J\}$, vertices queries $Q_V = \{q_1^V, q_2^V, ... q_m^V\}$, 

> 这里 n 就是 number of joints, m 就是 number of vertices



<img src="end to end human pose and mesh reconstruction with transformers.assets/截屏2021-11-13 上午11.33.53.png" alt="截屏2021-11-13 上午11.33.53" style="zoom:50%;" />



### 3.1 MVM (Mask Vertex Modeling)

Masked Language Modeling (MLM) to learn the linguistic properties of a training corpus. However, MLM aims to recover the inputs, which cannot be directly applied to our regression task.

> 在 NLP 里面用的是 MLM 的方法来做训练的，是去预测别的 query 下的词，作者认为这里不能直接用于 regression 3D coordinates

所以提出了 MVM

We mask some percentages of the input queries at random. Different from recovering the masked inputs like MLM [8], we instead ask the transformer to regress all the joints and vertices.

> mask some input queries at random, 但是直接要 transformer regress all joints and vertices
>
> 这里的 mask 方法没讲清楚是怎么mask 的，我的理解是不给 CNN features, 但还是给 positional encoding?????



### 3.2 loss

$L_v$: 3d vertex 之间的 L1 loss

$L_j$: 3d joint 之间的 L1 loss

$L_j^{reg}$: 存在 regression matrix G, 能从 vertex 里面计算出 joint 坐标，这里就是L1 loss of GT joint 和 用 G 算出来的 joint

> 能从 mesh vertex 中提取 joint, 就是希望 mesh 能和 joint 对应上
>
> 这里的思考是 为什么这里 **==不是和 predict 出来的 joint 对比，而是和 ground truth 的 joint 对比????????==**

$L_J^{proj}$: L1 loss of GT 2d joint point 和 投影出来的 2d joint point <u>(这里在 transformer output 上加了一层 linear layer 能自动学习 camera params)</u>

最后是 $L = \alpha(L_v + L_j + L_j^{reg}) + \beta L_J^{proj}$



# 4. implementation details

### 4.1 coarse mesh learning

our transformer processes a coarse mesh: (1) We use a coarse template mesh (431 vertices) for positional encoding, and transformer outputs a coarse mesh; (2) We use learnable Multi-Layer Perceptrons (MLPs) to upsample the coarse mesh to the original mesh (6890 vertices for SMPL human mesh topology); (3) The transformer and MLPs are trained end-to-end; Please note that the coarse mesh is obtained by sub-sampling twice to 431 vertices with a sampling algorithm [37]. As discussed in the literature [20], the implementation of learning a coarse mesh followed by upsampling is helpful to reduce computation. It also helps avoid redundancy in original mesh (due to spatial locality of vertices), which makes training more efficient.

> mesh template 只有 431 vertices, 最后生成的 3D mesh 有 6890 vertices, 这里是 subsample 下来之后，先预测 coarse vertices，之后用 MLP 来upsample到 6890 vertices. 这里用了 [37] 的**sampling methods twice** 才得到 431 vertices 的 coarse mesh. 
>
> 而且 [20] 还证明了 learning coarse mesh 再 upsampling 能 reduce computation, 而且 **avoid redundancy in original mesh????** 
>
> (我的理解是，很多点之间位置其实存在强关系，类似于中点之类的，所以不需要全部包括进去)



# 5. experiments

### 5.1 datasets:

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



### 5.2 Metrics:

**MPJPE**: mean-per-joint-positioning-error ==(用于3d pose)==: euclidean distance between GT joint 和 predicted joints

**PA-MPJPE**: ==(一般用于 reconstruction error)==, 用 ==Procrustes Analysis(PA)== 来做 3d alignment，然后再算 MPJPE, **因为和 scale 和 rigid pose (rigid transformation)没关系**

**MPVE**: mean-per-vertex-error, euclidean distance between GT vertex 和 predicted vertex



# 6. Result analysis:

<img src="end to end human pose and mesh reconstruction with transformers.assets/截屏2021-11-13 上午11.58.04.png" alt="截屏2021-11-13 上午11.58.04" style="zoom:33%;" />

> 这里提取 attention weight 来 analyze vertices interaction, 颜色越亮代表 weight 越大

At the first row, the subject is severely occluded and the right body parts are invisible. As we predict the **location of right wrist**, METRO **attends to relevant non-local vertices, especially those on the head and left hand**. At the bottom row, the subject is heavily bended. For the head position prediction, METRO attends to the feet and hands (6th column at the bottom row).

<img src="end to end human pose and mesh reconstruction with transformers.assets/截屏2021-11-13 下午12.02.24.png" alt="截屏2021-11-13 下午12.02.24" style="zoom: 33%;" />

estimate an overall self-attention map. It is the average attention weight of all attention heads at the last transformer layer.

> 就是估计 self-attention map

We visualize the interactions among 14 body joints and 431 mesh vertices in Figure 4. Each pixel shows the intensity of self-attention, where darker color indicates stronger attention. Note that the first 14 columns are the body joints, and the rest of them represent the mesh vertices. We observe that METRO pays strong attentions to the vertices on the lower arms and the lower legs. This is consistent with the inverse kinematics literature [2] where the interior joints of a linked figure can be estimated from the position of an end effector.

> 这个 visualization 不错，前14个是 joint, 后面都是 vertices，颜色越深代表weight越大。
>
> 发现 lower 的四肢 的 weights 一般都很高，**这个和 之前机械手 的 那个结论吻合**



### 6.1 transfer to other tasks

这个 transformer 的 structure 可以很容易 transfer 到其他 tasks 上面去，像是 3D hand in-the-wild reconstruction,

其实只要替换一下 template mesh, 然后重新训练就 ok

> 能 easily transfer to other tasks. 而且 在 3D hand reconstruction 上面也是 SOTA



