---
title: 瑞幸一年半复盘
date: 2020-12-06 10:38:54
tags:
     - 工作总结
categories:
     - 工作总结
top_img:
cover:
---
　　说说这半年做了什么，以及这半年我认识到了什么；
　　年初公司动荡，年中人员动荡；不过我这边还是比较稳的，或许有人听了会觉得我装B；稳是因为有准备或者有准备的计划；先不解释心态，说说我在这期间干了什么？
1. 因为人员迭代，有些开发中的需求一时无人接手，所以我被捞去sapi负责独立接手一个开发到一半的系统（前后端分离），负责该项目的开发人员刚离职，离职前还是留了些接口文档；然后就是梳理业务和接口文档的过程，花了几天时间整理完之后，就接上开发了；虽然是个陌生系统，但整理不算复杂，主要问题点在于和前端的磨合，因为前端跟我一样也是半路接手。
2. 咖啡师系统压单需求独立开发，该需求需要统计门店由于机器还是人力等原因导致订单积压，然后推送给相应岗位人员；该需求需要对接多方系统以及比较复杂的计算过程；整个开发过程比较顺利，测试过程场景比较复杂，需要开发配合，但总体还算顺利
3. 后期由于公司电商业务的发展，我便被调去营销系统了，因为刚介入该系统，所以前期主要是熟悉系统，做一些简单的活动配置；后期主要做一些优惠券相关的开发，因为当前还在此阶段，且咖啡营销和电商营销系统耦合，以及历史营销数据都在DB导致工作开展比较困难，有各方面的限制，包括但不限于人员沟通、历史问题导致的性能问题等；但后续应该会将电商营销系统迁出，由我们组单独负责
4. 参与了公司组织的各架构运维治理，了解了公司redis和mysql集群模式，以及部分高危线上故障案例剖析；收获很大，感慨很多。。。

以上就是这半年在公司做的事情，较之刚来公司之初，后续大部分需求会独立负责，也算是组织上的信任以及自身的一种成长；同时发现了自己在某些方面的缺点
1. 写完博客，不会去再次审批，导致格式不规整，部分语句不通顺
2. 老是打破生物钟，导致精神状态不佳

---
　　什么样的文章才是有意义的呢？
　　关于这个问题的思考从未终止过，每次写完一篇都会回过头来思考，这篇文章我从立题到最终完善我用了多久？文章耗时跟知识吸收程度有关系吗？有时候一个简单的知识点可能要一周或者更多时间去完善，写完后回头看看字数，也没那么多，那大部分时间去哪里了；查阅资料，阅读源码；就说tcp那两篇文章虽然知识点常见字数也不多，但是为此我下了linux源码去寻找TCP的踪迹；怎么说呢，很多所谓的“八股文”，从自己拿源码调试的，对比社区的文章以及有前辈总结好的文章来说，知识点就是那些点，但是阅读源码你却能把知识点“质化”，虽然过程比较坎坷，但是在平时编码过程中碰到问题，不会首先去打开浏览器，而是Ctrl+鼠标点进去看看源码。
　　最后希望自己以后的文章质量能更高点
