---
title: Beyond Static Features for Temporally Consistent 3D Human Pose and Shape from a Video
categories: 
  - paper review
tags: 
  - computer vision
  - 3d human pose and shape
  - video
comment: true
sticky: 0
mathjax: true
---

# Beyond Static Features for Temporally Consistent 3D Human Pose and Shape from a Video

这里开头说的 **temporally consistent and smooth 3D human motion from video** 应该意思就是能够生成 consistent pose and shape



### 1. intro

**用 video 学习的方法**

1. encode each frame into img_feat
2. send img_feat of all frames to temporal encoder to get temporal_feat
3. send temporal_feat to SMPL param regressor to regress values

但是这上面的方法上面**还是有很多的 temporal inconsistency**

> 作者认为类似这种的方法对于 current frame 的 dependency 太强，导致了 temporal learning 没学到

> 作者特别 criticize VIBE 里面的 **residual connections** 把 **current frame's img_feat 直接 拼到 the final temporal_feat** 上面给 SMPL regressor 是导致只学习到 current frame 的问题所在

> 中心思想 就是 **强制 模型去学习 temporal features**



作者的 解决思路是 提出了 **PoseForecast 模组**

中心思想就是 训练模型，能从 past frames 和 future frames 里面 **infer current frame**, 这样就类似于强制模型学习 **temporal information**

**之后把这个模组生成的 feature 给 integrate 起来最后做 SMPL parameter regression**



### 2. existing works

- **HMR**: SMPL + **adversarial loss** 来使 mesh anatomically plausible (???????)
- **HMMR**: **hallucinator**(???): quote, "It hallucinates the past and future 3D poses from a current frame and is self-supervised by the output of the 1D fully convolutional temporal encoder". 我这里的理解是不是 类似 **siamese network**??

- 2D joint heatmaps and silhouettes to predict SMPL params

- **self-improving system** consisting of SMPL regressor and **iterative fitting network**

  > 应该就类似 RNN，把生成的 output 重新作为 input feed 进去，多次 fitting

- 类似 SMPL 的，用 template human mesh 

- lixel-based 1D heatmap 来 localize mesh vertices

- **optical flow + 2D poses** 来使 模型 **==generalize to unseen video==**

- enforce network to **re-order shuffled frames** 来 enforce learning temporal info

- **==motion discriminator 来保证 plausible human motion==**

- **MEVA**: **coarse to find estimation**: estimate coarse 3D human motion then predict residual motion to refine
- predict future 2D poses, 再从 2D poses 上面 predict 3D poses



#### 2.1 consistency metric

衡量 temporal consistency 使用的是 **==3D pose acceleration error==**

分析之前的 methods 类似 HMMR 和 VIBE

发现 **似乎有 consistency 和 per-frame accuracy 之间的 trade-off**





### 3. TCMR

#### 3.1 temporal encoding

每个 frame 过 resnet 生成 features，2048-dim

利用 3 个 GRU units:

1. $\mathcal{G}_{all}$: bi-directional GRU, 中间点是 current frames $\lfloor \frac{T}{2} \rfloor$, 总共长度是 T frames，生成 $\mathcal{g}‘_{all}$
2. $\mathcal{G}_{past}$: uni-directional GRU, 起始是 frame $0$, 终止是 frame $ \lfloor \frac{T}{2} \rfloor -1$, 生成 $\mathcal{g}’_{past}$

3. $\mathcal{G}_{future}$: uni-directional GRU, 起始是 frame $T-1$, 终止是 frame $\lfloor \frac{T}{2} \rfloor +1$, 生成 $\mathcal{g}’_{future}$

> past 和 future 都是要 predict current frame 的

> **==一个问题是: 既然 temporal 已经被 past 和 future 包含了，为什么还需要 bi-directional GRU?==** 
>
> 文章这里的解释是 **不用 residual connection** 来使模型 strongly depend on current frame

> 注意，这里 GRU predict 出来的都是 temporal features 了，<u>之后就给 SMPL regressor 了</u>

<img src="Beyond Static Features for Temporally Consistent 3D Human Pose and Shape from a Video.assets/截屏2021-11-10 下午2.17.14.png" alt="截屏2021-11-10 下午2.17.14" style="zoom: 33%;" />



#### 3.2 integration

<img src="Beyond Static Features for Temporally Consistent 3D Human Pose and Shape from a Video.assets/截屏2021-11-10 下午2.17.58.png" alt="截屏2021-11-10 下午2.17.58" style="zoom: 50%;" />

利用了类似 **attention module** 来 生成 attention weight $a_{past}, a_{all}, a_{future}$, 最后产生 输入 SMPL regressor 里面的 $g'_{int}$

> 训练的时候，分别训练 $\mathcal{g}'_{past}$ 给 regressor 和 当前 frame 对比，对于$\mathcal{g}'_{future}$ 和 $g'_{int}$  同理

> 测试的时候直接 $g'_{int}$



#### 3.3 loss

quote, "L2 loss between predicted and groundtruth SMPL parameters and 2D/3D joint coordinates are used"





### 4. implementation details

human are cropped from image, and then cropped region is resized to 224x224

> ==这里的担忧是 resize 不就产生 deformation 了吗？更好的办法不应该是 **缩小然后 padding 吗？**==
>
> todo: **查一下代码是怎么操作的!!!**





### 5. experiments

发现如果移除了 residual connection, **acceleration error 小了很多**，而且离谱的是，在没有增加 PoseForecast 的时候，**PA-MPJPE 反而下降了**

> ?????? 减少了 current frame residual connection, 我们应该看到 single frame accuracy 下降才对???

<img src="Beyond Static Features for Temporally Consistent 3D Human Pose and Shape from a Video.assets/截屏2021-11-10 下午2.36.56.png" alt="截屏2021-11-10 下午2.36.56" style="zoom: 33%;" />

> 这是这里的解释。。。



> 这里对比的 SOTA 模型全部：

<img src="Beyond Static Features for Temporally Consistent 3D Human Pose and Shape from a Video.assets/截屏2021-11-10 下午2.38.41.png" alt="截屏2021-11-10 下午2.38.41" style="zoom:50%;" />



### 6. Other

这里还发现了 attention 里面，**past and future weights are larger than current weight**

最后提到了一个 **temporal smoothing 的 technique**，应该是能接着降低 acceleration 的

发现 FPS 从 15 到 30，acceleration error 也下降了一半





