---
title: liquidity pool
tags:
  - DeFi
comment: true
mathjax: true
---

[[https://youtu.be/cizLhxSKrAc]]

我们先从传统 股票 trading 开始，传统的股票trading依赖 **order book model**，就是 buyer 定价，seller 也定价，只要价格上有 match，就成交。

但是这种方式存在问题：

1. 如果价格一直没有match，就一直不能成交
2. 如果 buyer 或者 seller 虽然定价了，但其实没有这么多的 钱 或者 股票，不能成交

这时候是有一群 <u>market maker to facilitate trading, always willing to buy or sell certain an asset.</u> ?????? 

> they provide liquidity, so that users can always trade without another counterparty to show up



### market makers unsuitable for DeFi

Order Book Model **relies heavily on market makers**, and **order book model is unsuitable for DeFi**

market makers track the current price, and constantly change the price, 这在 DeFi 里面就**会产生很多的 blocks**，因为 ETH 的 processing power limited, 而且每次执行 ETH 的 DeFi 都要提供 gas fee, <u>所以 market makers 每次 update order 都要付 gas, cost is too high.</u>



这里还提到了 **second layer scaling.**

<img src="liquidity pool.assets/截屏2021-11-13 下午7.07.58.png" alt="截屏2021-11-13 下午7.07.58" style="zoom:33%;" />

> 这里的意思是 market makers 自己可能也有 **liquidity issues**
>
> 而且 single user 想要 trade 的话，要把东西 move in and out of the 2nd layer, **so 2 extra steps, inefficient**



### how Liquidity Pool works

比如说 uniswap 里面想要 交换 DAI 和 ETH. 



想要交换的双方提供 **tokens**，**DeFi 对于每一种 pair of tokens 都创建 new market.**

然后会有 **liquidity provider (LP)** that sets the initial price of exchange, DeFi 这里用 incentive 来给 LP 希望能 **set equal values of both tokens.**

> 如果 initial price 和 market value diverges, 就会创造 **arbitrage opportunity (套利空间)**，会导致 LP **lose capital** 

之后还可以有 新的 LP 进来把钱放进 **liquidity pool**



每个 LP 把钱放进 liquidity pool 之后，会得到 proportional number of LP tokens, 之后每次交换都会 收取 0.3% 的 fee. **这个 fee 就会均匀的分给 LP token holders based on the number of LP tokens.**

> 如果 LP 想把钱拿回来，就把 LP tokens 给销毁就能拿回 钱 和 incentives 了



AMM (automatic market maker) 是 deterministic algorithm 能自动 price adjustment. 不同的 平台类似 uniswap 和 别的就 区别在这个算法上面

#### Uniswap 的 constant product market maker 算法

<img src="liquidity pool.assets/截屏2021-11-13 下午7.28.22.png" alt="截屏2021-11-13 下午7.28.22" style="zoom:33%;" />

我的理解是 本来 DAI 价格等于 0.5 ETH, 然后 x = 1 个 DAI, y = 1 个 ETH, k = 0.5. 之后如果 有人想要拿更多的 DAI 出来换 ETH，x 上升，y 下降，导致 ETH 的价格变高

>  这种算法的好处是 can always provide liquidity, no matter how larger a trade is ????

简单来说就是, ratio of the **token in the pool** dictates the price (注意这里不是 trade 的 tokens 数量)

所以 bigger pool 导致 lesser price impact 导致 lower slippage 



### other algos

1. Balancer 可以支持 up to 8 tokens in a market
2. Curve 认为 UniSwap 对于 similar price tokens 之间交换有问题，改进后的 similar price tokens 有 lower exchange fee, and lower slippage





