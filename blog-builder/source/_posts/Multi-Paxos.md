---
title: Multi-decree Paxos
description: 理解分布式一致性:Paxos协议之Multi-Paxos
author: Starry
sticky: 6
tags:
  - Distributed
categories:
  - Distributed
date: 2023-02-16 13:33:00
index_img: https://photos.prnewswire.com/prnfull/20160912/406726LOGO?max=400
banner_img: https://th.bing.com/th/id/OIP.AWAytmCscBVVkyqoU3x33gHaE9?pid=ImgDet&rs=1
---

> 转载自[理解分布式一致性:Paxos协议之Multi-Paxos](https://juejin.cn/post/6891459638546399245)

在前面一篇文章我们讲到了Basic Paxos，本篇文章我会讲解更加通用和普遍的Multi-Paxos协议。

在Basic Paxos协议中，每一次执行过程都需要经历Prepare->Promise->Accept->Accepted 这四个步骤，这样就会导致消息太多，从而影响分布式系统的性能。 如果Leader足够稳定的话，Phase 1 里面的Prepare->Promise 完全可以省略掉，从而使用同一个Leader去发送Accept消息。 当然我们还要对请求消息做一些改造，这里我们在请求里面加入了轮数`I`，每过一轮，`I+1` 。 下面我们用序列图的形式尽可能的去展示Multi-Paxos的魅力。

# Multi-Paxos without failures

下图我们展示一个基本的Multi-Paxos一次执行交互流程，系统有1个Client，1个Proposer， 3个Acceptor， 1个Learner。

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a67f5d240e094e01947f94d242677afe~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

# Multi-Paxos when phase 1 can be skipped

上面我们讲到在Multi-Paxos中，如果Leader足够稳定的话，在接下来的执行中，phase 1 的请求其实是可以被省略的，那么接下来我们看一下被省略的整个流程。 这里round number需要+1,表示已经进入下一轮了。 ![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35340a553ea046c989cec0c56f01a6de~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

# Multi-Paxos when roles are collapsed

在Basic-Paxos中我们区分了很多角色，有Clients，Proposers, Acceptors and Learners。实际上Proposers, Acceptors and Learners可以合并成一个，我们把它统称为Server。下面是合并之后的序列图。 ![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3180b6914d5e4b3e992e5df1d55e6c75~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

大家看看，是不是实现起来简单很多？

# Multi-Paxos when roles are collapsed and the leader is steady

同样的，当Leader很稳定的时候，我们可以在接下来的执行中忽略Phase 1. 如下图所示： ![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7292a7b04cc54393b23d9f901cc259a7~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

