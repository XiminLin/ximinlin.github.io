---
title: Analyzing and Preventing Sandwich Attacks in Ethereum
categories:
  - paper review
tags:
  - DeFi
  - front-running
mathjax: true
comment: true
---

# 1. Abstract



### 1.1 Front-running

在传统 trading 里面，如果对一件东西的需求提高，这件东西就会升值，如果这时候 有人得到**内部消息(insider information)**, 知道说有人准备买入大量的 x，我就可以先买一些 x，然后坐等手上的 x 升值，之后再卖出去。

在 DeFi 里面，**DEX (decentralized exchange)** 是用来交换不同种的 crypto 的，同样遵循价格随需求上升而上升的规律，这个时候如果有 node 接收到了 有大额买入的 transaction, 他可以在 这个 transaction 发生之前也买入 一些，要保证他的这个操作发生在 transaction 之前，有几种做法:

1. node 就是 miner，那么可以直接把自己的 买入 放进去，然后把 transaction 放到 block 的后面
2. node 希望 别的 miner 能提前发布自己的 买入，则把 incentive (gas price) 调高一些，那么miners就会更先 mine 这个 node 的 买入



# 2. Intro

Flashbots [5] which allows users to submit suggestions for block compositions directly to miners.

> 软件 Flashbots 能提供合适的 combination or ordering of transactions in a block 来获得收益



**MEV (miner extractable value)**: a measure of the profit a miner (or validator, sequencer, etc.) can make through their ability to **arbitrarily include, exclude, or re-order transactions within the blocks they produce.**



当前的解决办法是 private mempools, 然后直接付钱给miners, 但是这个就**不是 decentrialized, 而且 cost 很高**

> 这里的思考是，给 incentives 也是没用的，**因为 users 在其中是有损失的**，如果 frontrunning 只是 赚钱的话那提出 incentives 啥的还行，但是这里有损失的话可能就不太行了。**(单纯的坏人)**

> **这个问题比较严重，因为这里 frontrunner 不算是 dishonest users, 因为不算 break rules**





# 3. background



### 3.1 AMM (automated market maker)

#### 3.1.1 liquidity pool

[liquidity pool.md](liquidity pool.md) 

liquidity pool, 用来方便交易

liquidity provider 类似于 order book model 里面的 market maker 愿意提供资金来买交易中的东西



#### 3.1.2 constant product AMM (UniSwap)

Users are able to swap tokens in the liquidity pool if the product of the two reserves, $r_1$ and $r_2$, is the same before and after the swap. Most AMMs also charge a fee for executing trades (currently 0.3% of the input amount on Uniswap). This fee stays in the asset pool and is eventually distributed to the liquidity providers as incentive to provide liquidity.

> 如果 pool 里面的 **product of the two reserves are the same**, 那就能够 swap tokens.
>
> Most AMMs 都会收 **0.3% fee for executing trades**. 最后变成 **激励 liquidity providers to provide liquidity**.



If a user swaps an **amount $a$ of asset $t_1$** on Uniswap, the **output amount of asset $t_2$** is hence calculated using the formula:

<img src="Analyzing and Preventing Sandwich Attacks in Ethereum.assets/截屏2021-11-14 下午12.53.12.png" alt="截屏2021-11-14 下午12.53.12" style="zoom:33%;" />

> r1, r2 都是 liquidity pool 里面的 资金数量，a 就是 trade 里面的 token 数量
>
> 这里就是说 如果 **想交易 a 数量的 token**，这个就是**==最后实际能买到的 token 的数量，因为升值了==**

因为每笔交易都为引起 价格变化，在发起交易的 user 那边，可以设置 **maximum relative price increase,** 如果在买之前已经升值超过了这个值，则交易失败

**==slippage rate==**: 本来是<u>预估价和实际成交价之间的差值</u>，现在变成 **衡量 上面的 maximum relative price 的，但是实际上会被用到的值**，**==代表了 最少 被交换到的 数量==**，如果上面公式计算出来的**最后收到的 数量比 slippage rate 要低，则交易失败**

> 但是 gas fee 还是要交的，**这里的 0.997 就是因为 0.3% 交了 gas**



### 3.2 sandwich attacks

其实就是 front-running + back-running, <u>有时候直接整个叫做 front-running.</u>

back-running 就类似于知道 X 升值了，马上的卖出 X 的操作，因为希望马上卖出 X，所以可能就会用 更高的 gas price 之类的 incentive 操作来使得 back-running 的操作先执行。

> 本来这种操作没有问题，还会帮助使 price across markets stable. 但是这种操作会对网络产生 congestion, 会使得很多有用的 transaction 发布不出去，这个问题也叫做 **transaction spam**

There is a current discussion between experts whether the existence of MEV is a risk for consensus layer security [3], or fundamental for keeping the Ethereum network decentralized[6].

> 但是 researchers 内部还是对于 front-running 这个东西是好是坏有争议的



#### 3.2.1 profit of attackers

我们这里把 正常的大额买入叫做 **victim transaction.**

这种 **sandwich attack 是被 slippage rate 给限制的**，比如说**有可能有很多 front-runner**, 他们都同时买入了 X，导致 X 在 victim transaction happen 之前已经产生最后 **output amount lower than the slippage rate**, 那么 victim transaction 就不执行了，attackers 就没收益了.

用公式来说明上面条件就是:

<img src="Analyzing and Preventing Sandwich Attacks in Ethereum.assets/截屏2021-11-14 下午2.34.46.png" alt="截屏2021-11-14 下午2.34.46" style="zoom: 50%;" />

> 注意这里 highlight 部分，就是 attackers 用 x tokens 买了的 **output amount**
>
> $v$ is the amount of asset $t_1$ to be swapped in **victim transaction**, $r_1, r_2$ are the reserves for assets $t_1, t_2$
>
> $m$ is the **slippage rate** of victim transaction
>
> ==这个公式就是 把 r_2 减去 attacker 换走的数量，再把 r_1 加上 attacker 放进来的数量，之后再通过公式计算即可==



上面不等式取等号就能算出 attackers **能投的最大的数量** (超过的话，victim transaction 就执行不了)

<img src="Analyzing and Preventing Sandwich Attacks in Ethereum.assets/截屏2021-11-14 下午2.40.49.png" alt="截屏2021-11-14 下午2.40.49" style="zoom: 33%;" />



# 4. Analysis method

观察 ETH 的历史 transactions，**identify sandwich attacks**, 用了以下的 heuristic 来判断是不是 **sandwich attacks**:

<img src="Analyzing and Preventing Sandwich Attacks in Ethereum.assets/截屏2021-11-14 下午2.43.41.png" alt="截屏2021-11-14 下午2.43.41" style="zoom:50%;" />

> 总结就是找一对 transaction, 数量相同，方向相反，同一个 block 内，且不同对 transactions 之间不能重叠
>
> 作者认为 **Rule 1** 保证了能找到 sandwich attacks 的数量的 **lower bound**， 
>
> **Rule 2 也是必须的**因为存在 一个 transaction 里面同时 swap 两种 assets 的，但是因为没用 victim transaction 在中间，所以不可能是 sandwich attack
>
> Equal amount 其实也不一定，可以买多一些然后卖少一些都行，但是这里先找lower bound，全部卖掉也是当前attacker的最佳策略



### 4.1 differ from prior works

这里不需要在两次 attacks 之间寻找 victim transaction, **因为 victim transaction 可能会 fail**, 导致 sandwich attacks 失败

之前的 research 要求 liquidity for the two swaps must be provided by the same address. 

还有的要求 两个相反的 transaction 必须是 signed by same account, 而且 sent to same smart contract

> 作者认为有 more sophisticated attacks 了现在所以 上面不一定 hold

还有 认为 gas price of victim transaction must be between front-running transaction and back-running transaction

> 但是作者认为 miner 现在也可能是 sandwich attacker，对于自己mine出来的 transaction，就不需要 gas fee 了





# 5. empirical result

发现 attackers 也并不是稳定获利的，attackers 的 transaction 可能会被 miner 给利用，来 reorder transaction in unfavorably order.

比如说 attacker 和 miner 都看见了 victim transaction 买入 asset X

attacker 发送了 high gas price 买入 X，和 卖出 X 的 操作，miner 如果把 attacker 的操作顺序倒过来 (在 attackers 手上有 X 的时候)，那么 买入 X 的操作 miner 反而还能以更低的价格来买入

attacker 在这种情况下就会承受损失了

Daian [6] even argues that this process is fundamental to keep the Ethereum network secure in the future.

> 所以也有 researcher 认为 miner 的这种操作其实也保证了 chain 的稳定
>
> 我这里的理解是，因为 miner is incentivized to reorder transactions, **所以只要出现能在 自己买之前把 asset X 的价格降下来的transaction 都放到前面去，然后自己买入，坐等升值**。首先在一定程度上，**attackers 也冒着风险了**
>
> 但是不管 miner 和 attackers 之间的博弈怎么样，**victim transactions 大概率都是损失了**
>
> **victim transactions 有可能获利的情况是, total attackers selling outrun victim transactions buying, 导致价格还没 victim transaction 预估的高，几率还没计算，todo!!!!**



==**思考**： 把 X 的价格先降下去，再通过 victim transaction 提升的价格是会更赚钱吗？==

是的，因为 价格变化 是和 reserve 大小有关的，reserve 变少了，同样数量的买入，提升价格的能力就提升了，所以会更赚，所以我们发现**其实上面 计算的 maxInput 还可以更大**，因为把 别人的卖出操作放在 miner 的 front-running 前面还有好处，**能够降低 低于 slippage rate 的风险**



# 6. order splitting as solution

Main Idea of front-running attacks 就是:

==**A sandwich attack is only possible if the difference in market price before and after the swap is large enough to compensate for transaction and exchange fees.**==

作者认为只要把 这种 price change 影响降低到够低，就避免了 front-running attacks

The number of split orders must **be kept as low as possible** - not only because of the **constant transaction fee** which needs to be paid for every transaction, **but also because the exchange fee stays in the pool after every swap and changes the market price for the worse**

> 作者希望使 number of splitted transactions 降到最低，因为 transaction fee 是 constant for each transaction，而且 transaction fee 会被放进 reserve 里面最终仍然会提高价格

<img src="Analyzing and Preventing Sandwich Attacks in Ethereum.assets/截屏2021-11-14 下午3.51.56.png" alt="截屏2021-11-14 下午3.51.56" style="zoom:50%;" />

> 这里 $output_{A_2}$ 计算了 attacker 把 assets 再卖出所得到的 token 数量
>
> v 是 victim transaction 要买的 token 数量
>
> 我们把 v 分成 $v_1, v_2, ...$ 之后再回来算，就能知道 lower bound to prevent front-running, 也能算出 trade-offs



















