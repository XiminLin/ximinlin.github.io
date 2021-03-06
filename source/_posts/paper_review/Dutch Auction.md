---
title: 荷兰式竞拍
tags:
  - DeFi
comment: true
mathjax: true
---



[[https://youtu.be/1UK0KQV03Wc]]

用于很多 IPO 项目，希望散户也来参与的拍卖

总结起来就是：

开始拍卖 10 个 coins, 公司定一个最高价，之后逐渐递减。

竞拍者可以在一个价格说出自己想要买的数量，比如说：
1. A 要 6块 买 6 个，commit 36 元
2. B 要 5 块买 2 个，commit 10 元
3. C 要 4 块买 2 个，commit 8 元


有两种方式结束竞拍：
1. 价格触底但是 coins 还没卖完 （触底价会提前定好）
2. 东西卖完了

规则是，最后卖完的 价格 就是每个人要付的价格，每个人想要买的数量不变，所以 如果最后 4 元结束，A 会得到 36/4=9 coins. 

之后结束的时候就是 first-come-first-serve 的逻辑

# 为什么？

很多 IPO 使用 dutch auction, 目的是减少公司和投行的损失.

很多公司上市的时候，会由 投行 根据 roadshow 来定价，最后在上市的那天以这个价格卖出。但是出现很多在第一天开始交易的时候，价格就暴涨。这样 公司和投行就损失钱了，因为卖便宜了。

于是出现了用 dutch auction 的办法，这样就能卖出一个比较合理的价格。


# 思考：

这是因为发行价固定而导致的后期股票价格上涨，其实从上往下或者更直接一些，从下往上报价然后（有炒作嫌疑），然后没人想买的时候（价格达到预定最高价的时候），剔除最低价的，然后从最高价往下排列分 shares。

有说法是能更 民主，希望散户也能进来投资。这个也有一定道理。
