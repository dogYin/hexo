---
title: 分布式之注册中心
date: 2020-10-13 09:37:33
tags:
    - 分布式
    - 注册中心
categories:
    - 分布式
top_img: https://cdn.jsdelivr.net/gh/dogYin/imgOSS/img/top.PNG
cover: https://cdn.jsdelivr.net/gh/dogYin/imgOSS/img/top.PNG
---
&nbsp;&nbsp;&nbsp;&nbsp;Dubbo结合分布式各治理组件是现在国内比较常用的微服务脚手架





## 名词解释
脑裂：当网络发生异常时，分布式系统中的部分节点之间的通信会出现延时，当延时不断增大最后导致部分节点不可能用我们将这种情况称之为脑裂。

CAP:分别为Consistency一致性，Availability可用性，Partition tolerance分区容错性；一个分布式系统中最多只能同时满足其中两项

BASE:分别为基本可用（Basically Available）、软状态（Soft state）、最终一致性（Eventually consistent）；即使无法达到强一致性，但是每个应用应该根据自身的
特点达到最终一致性  

分布式事务：事务ACID的特性在单机系统下可利用[数据库隔离级别](https://dogyin.wang/2020/04/25/20200421-mysql07/)实现;但是分布式项目是n节点部署，每个节点都只能知道自身事务是否成功，并不能知道其他节点是否执行
成功，因此我们需要一个中间“协调者”来统计各个节点的事务状态然后决定是否提交，所以就会有接下来介绍的2PC，3PC，Paxos算法，ZAB协议等。

至于下面的两阶段提交三阶段提交和拜占庭将军算法，都是分布式一致性的解决方案，均为了追求分布式节点下事务的强一致性

2PC:两阶段提交

3PC:三阶段提交                                                                                                                                         

Paxos算法：实现代表有Chubby

ZAB协议：实现代表zookeeper