---
title: PARE- Part Attention Regressor for 3D Human Body Estimation
categories: 
  - paper review
tags: 
  - computer vision
  - 3d human pose and shape
  - occlusion
comment: true
mathjax: true
sticky: 0
---



主要解决 single image 3D human pose and shape estimation 里面的 **occlusion 问题**



# 1. 问题

SOTA single image method 经常是 ResNet 提取 features 之后直接 regress SMPL params. 

文章认为 这种提取出来的 global feature, **对于 pixel changes 非常的 sensitive**，所以对于 occlusion performance bad



### 1.1 Occlusion

存在 self-occlusion, close-range interaction with other people occlusion, and occlusion due to objects



# 2. Intro

主要的方法是：

分两个branch：

1. 3D body params branch，产生 $H \times W \times C$ dimensions 的能直接 regress SMPL params 的 features, $C$ 就是 embed dimensions
2. 2D joints segmentation mask, 产生类似 attention 的 $H \times W \times J$ dimensions 的，$J$ 是 number of joints, 对于每个 pixel 都产生 probability that it belongs to one joint.

这里的 intuition 是，to be robust to occlusion (pixel changes), model should leverage **pixel-aligned image features of visible parts** to **reason about the occluded parts**. 

> 这里 ==pixel-aligned image features 我的理解是 pixel-wise features.== 
>
> 但是不够，==这里也希望说 occluded parts 不能影响到 visible parts, 然后还希望 visible parts **help** occluded parts.==
>
> 所以需要 attention 来生成 joint features dependent **only visible parts pixels (soft-attention mask)**

这里的训练方法是 2D joints segmentation mask 先分开 supervised training，之后再和 3D body params branch 一起训练

> 这里也是实验得出的 效果最好的训练方式
>
> 后面会提到，对比了三种训练方式:
>
> 1. 不单独训练 2D，直接一起训练 (效果最差)
> 2. 单独训练 2D 之后，weight freeze
> 3. 单独训练 2D 之后，合在一起训练 (Best)

这里 第三种的意思 是，希望 attention 能学习到 用 visible 来 预测 occluded，**(希望不只是 joints segmentation mask 了)**

之后 combine 在一起之后直接 regress SMPL parameters

> 看 Methods 里面的 图片



# 3. related works



### 3.1 implicit occlusion handling (data augmentation)

1. 之前的办法就是 data augmentation, cropping frame, overlaying patches as occluding body parts

2. Cheng et al. apply augmentations to heatmaps that contain richer semantic information and hence occlusions can be simulated in a more intelligent way

   > 就是说在 heatmaps 上面做 augmentation，比起直接挡掉**更能模拟真实的 occlusion**

作者认为虽然有帮助，但是 **不够** 平常的 occlusions 那样 **complex**，而且**作者认为模型架构上需要改变**

<u>总之对于 implicit occlusion 就是看不起</u>



### 3.2 explicit occlusion handling

Cheng et al. [9] avoid including occluded joints when computing losses during training. Such visibility information is obtained by approximating the human body as a set of cylinders, which is not realistic and only handles self occlusion.

> 把人想成 set of cylinders, 然后把 occluded joints 部分的 loss 在 training 的时候删除
>
> 作者认为 这种 approximation 不好，而且只能处理 self occlusion

Wang et al. [56] learn to predict occlusion labels to zero out occluded keypoints before applying temporal convolution over a sequence of 2D keypoints.

> 这里是想 预测 occlusion label for keypoints，我理解是想法和上面类似，把 occluded 的部分的 loss 删除

Zhang et al. [61] leverage saliency masks as visibility information to gain robustness to scene/object occlusions. Human meshes are parameterized by UV maps where each pixel stores the 3D location of a vertex, and occlusions are cast as an image-inpainting problem.

> 用 saliency map, 把人体弄成 UV maps，然后 occlusion 被看成是 image inpainting problem，我理解是 直接涂掉是 一样的
>
> 作者认为 in-the-wild data 可能**没有很好的 saliency maps**, **<u>UV-coordinates 能导致 mesh artifacts.</u>**



# 4. occlusion sensitivity analysis

**就是检测 算法对于 occlusion 的 sensitivity 程度的**



To extract features from the input image *I*, current direct regression approaches [24, 29] use a ResNet-50 [17] backbone and take the features after global average pooling (GAP), followed by an MLP that regresses and refifines the parameters iteratively.

> 当前的方法就是 resnet50 提取 image feature, 之后 global average pooling,  最后 MLP 生成 SMPL params



Zeiler et al. [60] who systematically cover different portions of the image with a gray square to analyze how feature maps and classififier output changes. In contrast, we slide a gray occlusion patch over the image and regress body poses using SPIN [29]. Instead of computing a classifification score as in [60], we measure the per joint Euclidean distance between ground truth and predicted joints.

> 这里的 difference 感觉比较 trivial，我的理解是这里的 sliding 更有更全一些的 output，就之前是 different portios, 这里是 every pixel

We create an error heatmap, in which each pixel indicates how much error the model creates for joint *j* when the occluder is centered on this pixel.

> 这里就会产生在 遮挡 square 的中心点在 某个 pixel 上时，预测出来的 joint 和 GT joint 的 error 是多少，然后这个 error 就导致了heatmap 上 这个 pixel 的颜色。

<img src="PARE- Part Attention Regressor for 3D Human Body Estimation.assets/截屏2021-11-12 下午11.11.26.png" alt="截屏2021-11-12 下午11.11.26" style="zoom:50%;" />

> 这里是跑 model SPIN 的结果，**作者从中得出三个结论**：
>
> 1. error集中在人的pixel上面，说明 SPIN working
>
> 2. 原来能看见的 joints 现在被挡住了产生的 error 更大
>
> 3. ==本来就挡住的 joints，能从其他的 joints 中 infer 出来 $\leftarrow$ <u>*可以从对比 left/right ankles(occluded) 的 error 当把 thigh region occluded 的时候，发现 error 较大，说明有利用到 thigh 方面的  features*</u>==



# 5. Methods



### 5.1 SMPL 简单介绍

SMPL [33] represents the body pose and shape by $\theta$, which consists of the pose $\theta \in \mathbb{R}^{72}$ and shape $\beta \in \mathbb{R}^{10}$ parameters. Here we use the gender-neutral shape model as in previous work [24, 29]. Given these parameters, the SMPL model is a differentiable function that outputs a posed 3D mesh $M(\theta, \beta) \in \mathbb{R}^{6890 \times 3}$. The 3D joint locations $\mathcal{J}_{3D} = W M \in \mathcal{R}^{J \times 3}, J = 24$ are computed with a pretrained linear regressor $W$.

> 就是说 3D poses 里面包含了 24 个 joints (24 x 3 = 72), 然后 shape params 是 10-dims 的，最后生成的 mesh 有 6890 个 keypoints on surface. 
>
> 这里的 W 就是把 SMPL 的 M 最后提取出 24 个 joints 的，用来和 GT 3D joint 对比来做 loss



### 5.2 Methods



<img src="PARE- Part Attention Regressor for 3D Human Body Estimation.assets/截屏2021-11-12 下午9.21.13.png" alt="截屏2021-11-12 下午9.21.13" style="zoom: 50%;" />

先是 通过 CNN backbone (ResNet50) 来提取出来 pixel-aligned volumetric feature

之后通过 two branches:

1. 2D part branch 来得到 *“类似”* joint segmentation mask 的 features, $P$, dimension = $H \times W \times J$
2. 3D body branch 来直接生成给后面 SMPL regressor 用的 features, $F$, dimension = $H \times W \times C$

之后我们整合起来得到 $F_{j,c}' = \sum_{h,w} \sigma(P_j) \odot F_c$

> $\sigma(P)$ 理解成 soft attention mask, 中间的乘积叫做 hadamard product, 其实就是 elementwise product, 所以是 attention
>
> 然后 $F'$ 就是 num_joints $\times$ embed_dims 的结构，类似于 joint features



This attention operation suggests that if a particular pixel has a higher attention weight, its corresponding feature contributes more to the final representation $F'$

> 就类似于去 pixel-aligned 的 features 里面找能相关的 pixels，**开始的时候 2D joint map 就是 segmentation mask, 后面就变得灵活了**

In the case of occlusion, however, if we predict part segmentation as $P_j$ , the feature for the joint $j$ can aggregate per-pixel features only belonging to that particular body part.

> 这就是需要后面 $P$ 变得灵活的原因，不然的话 occluded joint feature 就都是 0 了，这里灵活之后就希望能够学习用 visible joints 来 infer missing joints



### 5.3 losses

<img src="PARE- Part Attention Regressor for 3D Human Body Estimation.assets/截屏2021-11-12 下午11.58.02.png" alt="截屏2021-11-12 下午11.58.02" style="zoom: 33%;" />

> $\mathcal{L}_P$ 就是 2D segmentation mask 的 loss，作者的意思是 前期 $\lambda_p$ nonzero, 后期给 调成 0 来使学习 attention



# 6. implementation details

To increase robustness to occlusion, we use common occlusion augmentation techniques; i.e. synthetic occlusion(SynthOcc) [45] and random crop (RandCrop)

> 还是用了 occlusion data augmentation 的



<img src="PARE- Part Attention Regressor for 3D Human Body Estimation.assets/截屏2021-11-13 上午12.05.52.png" alt="截屏2021-11-13 上午12.05.52" style="zoom:50%;" />

> 这就是 ablation study 的结果，重点关注，parts, unsup, and parts/unsup，就代表了之前说的 三种 $P$ 的方法



<img src="PARE- Part Attention Regressor for 3D Human Body Estimation.assets/截屏2021-11-13 上午12.08.13.png" alt="截屏2021-11-13 上午12.08.13" style="zoom:50%;" />

> 这是 occlusion data augmentation, 其实就是 inpaint objects over the person



### Challenging Cases

<img src="PARE- Part Attention Regressor for 3D Human Body Estimation.assets/截屏2021-11-13 上午12.08.13-6780242.png" alt="截屏2021-11-13 上午12.08.13" style="zoom:50%;" />





