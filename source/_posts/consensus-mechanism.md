---
title: 区块链共识机制
date: 2018-01-31 23:23:30
tags: blockchain
---

在区块链世界中，最为常见的一个词就是“去中心化”。

“去中心化”是一个摆脱垄断和集权的美好愿景，但是在现实生活中，很多做出决策的场景都是存在中心节点的（回忆一下领导这个角色在生活中存在的广泛性），存在即合理，中心化决策所具有权威性是很明显的。

但是在“去中心化”的世界里，如何在没有中心节点的情况下完成决策呢？最简单的方式自然就是投票了，但是投票也存在着信道不可信、通讯故障等等问题，区块链作为“去中心化”的典型应用，其具有很宝贵价值的一块核心就是共识机制(consensus mechanism)了。

目前比较常见的共识机制有如下几种

1. PoW: Proof of Work, 工作量证明
2. PoS: Proof of Stake, 权益证明
3. DPos: Delegate Proof of Stake, 股权授权证明
4. PBFT: Practical Byzantine Fault Tolerance, 实用拜占庭容错
5. dBFT: delegated Byzantine Fault Tolerance, 授权拜占庭容错

<!-- more -->

1. PoW
   工作量证明，从字面来看就已经可以知道，PoW 通过节点的工作量作为共识机制的衡量标准。
2. 
