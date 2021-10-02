---
title: SimPoE Simulated Character Control for 3d human pose estimation
categories: [paper review]
tags: [computer vision, 3d human pose, 3d human shape, Reinforcement Learning]
comment: true
mathjax: true
---

todo:

1. 了解为什么没有 flaws，然后具体是怎么解决的
2. 了解一下 root torque 是怎么分成 15 steps 来 apply 的，看看之前的 paper
3. action generation unit 不是 RL 吗，但是这里好像是 MLP 直接 infer action？？？
4. 增加 metric 的 formula 啥的



# 1. intro

### 1.1 problem:

kinematics 已经很多研究了，但是 dynamics 还没怎么研究.
- kinematic: 主要集中在 geometric relationships of 3D poses and 2D images. **但是只考虑 positions 的**，文章指出因为**没考虑 physical forces (dynamics),** 所以经常出现物理条件下不可能的情况：
	- 穿模
	- 在视频中预测的 3d 模型抖动
	- 脚在地面上滑行

- dynamics: 也关注了 physical forces 等类似 friction，joint torque 啥的

### 1.2 加入了 dynamics 的 related works:

大部分都是 先 kinematic motion estimation, 之后再用 physics-based trajectory optimization (**two stages**)

但是存在问题：
1. trajectory optimization highly complex，而且不是 train 出来的，只是 post-processing 的一个 step，**所以 test-time 有很长 latency**
2. 用 simple，differentiable physics model 来 optimization，但是更准确的 model 都是 un-differentiable 的，所以这种办法可能有 **high optimization error.**
3. 因为是 post-processing, **不能 end2end learning**

### 1.3 用 RL 来模拟 character control 的

1. 有 manually driven awards 的
2. 有 **GAIL based method RL** 不用 reward engineering 的

来达到 long-term behavior，有的 work 用了 hierarchical RL 来 control characters

最近的 work 来学习 user-controllable policies from motion capture data. **但是，这种学习智能 reproduce training motions, 对于没见过的就不行了**
> ????? 了解一下具体是怎么有 flaws，然后这个是怎么解决的

# 2. method

![image-20211001201237465](image-20211001201237465.png)

### 2.1 在 simulation 里面创建一个人的 model

用 SMPL model 的算法 类似 VIBE 来产生 skinned mesh human model，这些模型提供了：

1. skeleton of $B$ bones
2. mesh of $V$ vertices
3. skinning weight matrix $W \in \mathbb{R}^{V \cross B}$, 每个 i-j element 都代表了 influence of $j$ th-bone's transformation on $i$ th-vertex's position
   1. 接着把每个 vertex 分配到 bone，从 $W$ 里面看哪个 bone 对 这个 vertex 的影响最大就分配给 哪个bone， 计算得到 $A \in \mathbb{R}^V$ 
   2. 用 convex hull 来计算 bone 的 geometry，接着假设 constant density of bone，mass 就由 geometry 决定

### 2.2 simulate character control

####  2.2.1 用 PPO 算法（2017）来做 RL

#### 2.2.2 定义 states：$s_t = (q_t, \dot{q_t}, \tilde{q}_{t+1}, \check{x}_{t+1}, c_{t+1})$:

1. $q_t$ 是 current pose
2. $\dot{q_t}$ 是 joint velocities
3. $\tilde{q}_{t+1}$ 是 estimated kinetic pose
4. $\check{x}_{t+1}$ 是 2D keypoints
5. $c_{t+1}$ 是 keypoint confidence

#### 2.2.3 定义 actions:

1. 一般的 action 设计是计算 每个 **non-root joint** 的 从一帧到下一帧的 **torque**. 

2. **一个 sampling difference 问题**：这里存在 video 是 30Hz 的，但是 physics simulator 是 450Hz 的，所以我们每帧之间的 action 需要分配成 15 个 simulation steps. 这里我们使用 **proportional-derivative(PD) controllers** 来分解这 15 个 steps，生成 15 个action来得到最终效果，PD controller 类似于 damped spring. 

   1. PD 的 定义：$\tau_t = k_p \circ (u_t - q_t^{nr}) - k_d \circ \dot{q}_t^{nr}, t \in [0,15)$ 

      1. $\tau_t$ 就是每一个 step 施加的 torque
      2. $u_t$ 是 **target joint angles** of PD controllers
      3. $q_t^{nr}, \dot{q}_t^{nr}$ 分别代表 在这个 step 开始的时候的 joint angles 和 joint angle velocity
      4. <u>$k_p, k_d$ 分别代表 stiffness 和 dampness of spring, 是 hyperparam，需要 manual 调整</u>

   2. 我们之后 介绍 meta-PD controller, 为了**弥补上面需要 手动调整 $k_p, k_d$ 的问题，我们引入 $\lambda_p, \lambda_d$**

   3. prior works 介绍了如果**给 root joint 加 torque 提升模型 robustness，**所以 action 再 **predict residual forces and torque $\eta_t$ 给 root**

      > ？？？？？？这里文章没说 root torque 是怎么分成 15 steps 的，如果没有用 meta-PD 的话，就可能 apply same torque 15 times 来训练也行，可能隐藏在 prior works 里面了，参考 "residual force control for agile human behavior immitation and extended motion synthesis. 2020"

#### 2.2.4 定义 rewards：

1. $r_t = r_t^p * r_t^V * r_t^j * r_t^k$

   1. $r_t^p$ 是 pose reward, 计算 difference of local joint orientations $\sigma_t^j$ and ground truth $\hat{\sigma}_t^j$

   <img src="image-20211001172725330.png" alt="image-20211001172725330" style="zoom:33%;" />

   > $J$ is number of joints, $\ominus$ is relative rotation between two rotations

   2. $r_t^V$ is velocity reward<img src="image-20211001172642341.png" alt="image-20211001172642341" style="zoom:33%;" />
   3. $r_t^j$ is 3d world joint position reward

   <img src="image-20211001172820774.png" alt="image-20211001172820774" style="zoom:33%;" />

   4. $r_t^k$ is 2d projection of joints to match 2d joints

   <img src="image-20211001172913683.png" alt="image-20211001172913683" style="zoom:33%;" />

2. 这里选择 **multiplication of sub-awards** 由 prior works “A scalable approach to control diverse behaviors for physically simulated characters” 显示，保证说 每个reward都不会被 overlooked.

### 2.3 kinematic aware policy

> 这里 RL 的 explore 是定义成 normal distribution 来选择 action 的，这里定义一个 mean 和 variance，mean 在最后应该是 optimal action

<img src="image-20211001184256526.png" alt="image-20211001184256526" style="zoom: 50%;" />

这里 $R_\theta$ 是 kinematic refinement unit, 这里的 $\tilde{q}_{t+1}^{(n)}$ 就是 kinematic pose after n iterations of refinement. 

$\mathcal{G}_\theta$ 是 control generation unit, 通过当前 pose 和 velocity 和 下一帧的 pose，能得到这些值

**<u>这里没有直接 regress 到 $\overline{u}_t$，因为 author 说这样 learning easier</u>**



#### 2.3.1 kinematic refinement unit $\mathcal{R}_\theta$

<img src="image-20211001185712007.png" alt="image-20211001185712007" style="zoom:33%;" />

这里定义 MLP $\mathcal{U}_\theta$， $z$ 是 gradient of reprojection loss $\mathcal{L}$, 

> inspired by prior work "Human body model fitted by learned gradient descent", 这里不是想 minimize loss，只是想用这个 as informative kinematic feature to learn a pose update 是最后的 pose stable 且 accurate

**reprojection loss $\mathcal{L}$ 定义：**

<img src="image-20211001190340020.png" alt="image-20211001190340020" style="zoom:33%;" />

大写 X 是 3d 的，小写是 2d 的，这里就是做了一个 projection，并且**乘上了 2d joints uncertainty**, 来 account for **keypoint uncertainty.**

>  z converted to character's root coordinate to be invariant of character's orientation



> **文章这里说 因为 dynamic 那边用了 kinematic 的预测，然后在 RL 里面 train 的时候。就类似于 jointly training 了**



#### 2.3.2 feature extraction layer

这里在 直接把 current pose, pose velocity, 和 next pose 传给 action generation unit 之前，先传给 feature extraction layer, 再经过 normalization layer 之后，再传给 最后的一个 MLP 来生成 action

> ?????? 这里不应该是 RL 吗，但是这里看到的好像是 MLP 直接 infer？？？todo



#### 2.3.3 Meta-PD control

主要关注 $k_p$ 和 $k_d$ 的 ratio，large ratios lead to unstable and jittery motions, small ratios 可能就太 smooth 然后 lag behind ground truth.

我们使用两个 initial params $k_p'$ 和 $k_d'$，然后对于每个 15 steps，我们生成参数 $\lambda_{tj}^p$ and $\lambda_{tj}^d$ 来得到每一个 step 的 新的 $k_p$ 和 $k_d$

<img src="image-20211001192613941.png" alt="image-20211001192613941" style="zoom:33%;" />



# 3. Experiments



### 3.1 Metrics

**检验 shape accuracy 的**

#### 3.1.1 MPJPE (mean per joint position error)

> todo: .... add formula

#### 3.1.2 PA-MPJPE (procrustes-aligned mean per joint position error)

> todo; ... add formula



**检验物理稳定性的**

#### 3.1.3 Accel (difference in accleration between the predicted 3d joint and GT)

> Todo: .... add formula



#### 3.1.4 FS (foot sliding)

对比 body mesh vertices that contact the ground on 连续 2 帧，compute average displacement within frames.

> todo:.... add formula



#### 3.1.5 GP (ground penetration)

compute average distance to the ground for mesh vertices below the ground.

> Todo: .... add formula



### 3.2 datasets

这里用了 human3.6M (S1, S5, S6, S7, S8) 做 training，(S9, S11) 做 testing

还有 in-house human motion dataset，其中包括了 detailed finger motion.



### 3.3 具体的测试

#### 3.3.1 simulation character create

using SMPL model for Human3.6M

For in-house dataset, use **non-rigid ICP** and **linear blend skinning** 来 reconstruct skinned human mesh model.

> Todo: 1. non-rigit ICP: optimal step nonrigit icp algo for surface registration; 2. linear blend skinning: skinning with dual quaternions



#### 3.3.2 initialization

for Human3.6M, VIBE 直接 predict pose 

For in-house dataset, 自己创建了一个 kinematic pose estimator



#### 3.3.3 Other details

experiments 中，**之前的 physics-based method 产生很大的 Accel 因为 character 经常 falling when losing balance. (但是其实人没有倒...)** 



# 4. limitations

这里因为用了 3d scene simulation 来 enforce contact constraint during motion estimation. **所以对于 in-the-wild dataset 就不能用。**



