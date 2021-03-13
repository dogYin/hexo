---
title: 发号器？发号服务？
date: 2021-03-10 21:50:27
tags:
    - 系统设计
categories:
    - 系统设计
top_img:
cover:
---
　　分布式ID的生成也算是业界老生常谈的话题，Twitter的雪花ID生成算法作为标杆还是很受欢迎的，但是随着国内互联网快速的发展服务部署方式从机器集群到k8s集群部署以及雪花id没有解决时钟回拨问题，雪花ID不是能很好的适配；为此，百度基于雪花id改造开源了适用于k8s集群部署的uid-generator。  
uid-generator的构成是
|描述|  标记位   |  相对时间序列   |  workid   |  每秒并发序号   |
|---| --- | --- | --- | --- |
|位数|  1   |    28 |    22 |   13  |

其实我们主要考虑两个问题：1.时钟回拨怎么办？ 2.workId怎么配发？  
我们先看看百度是怎么处理的，通过以下代码我们可以看出来，如果发生时钟回拨的情况会抛出异常给使用者，当然这种生成方式跟普通的雪花id没什么区别只是校验了可能发生的坏味道！
```java
 // Clock moved backwards, refuse to generate uid
 //DefaultUidGenerator
  if (currentSecond < lastSecond) {
      long refusedSeconds = lastSecond - currentSecond;
      throw new UidGenerateException("Clock moved backwards. Refusing for %d seconds", refusedSeconds);
  }
```
　　那还有CachedUidGenerator是干嘛的，它是就是借用未来时间将生成的号存在一个大小为序列号最大值的环形数组中，其实这里还有一个相对应大小的数组用来标记存号数组中的号是否可以用，使用这种方式可以避免时间的判断直接使用未来时间发号。
```java
  /**
     * Padding buffer fill the slots until to catch the cursor
     */
    public void paddingBuffer() {
        LOGGER.info("Ready to padding buffer lastSecond:{}. {}", lastSecond.get(), ringBuffer);
        // is still running
        if (!running.compareAndSet(false, true)) {
            LOGGER.info("Padding buffer is still running. {}", ringBuffer);
            return;
        }
        // 如果数组中还有位置，或者相应标记位的号不能用了那么久用下一个时间点填充
        boolean isFullRingBuffer = false;
        while (!isFullRingBuffer) {
            List<Long> uidList = uidProvider.provide(lastSecond.incrementAndGet());
            for (Long uid : uidList) {
                isFullRingBuffer = !ringBuffer.put(uid);
                if (isFullRingBuffer) {
                    break;
                }
            }
        }
        // not running now
        running.compareAndSet(true, false);
        LOGGER.info("End to padding buffer lastSecond:{}. {}", lastSecond.get(), ringBuffer);
    }
```
　　通过上述了解我们基本已知道时钟回拨百度的uig是怎么解决，以及百度缓存未来号的设计；为了更高的性能uid在取号时使用了缓存填充来解决伪共享的问题，据说QPS可以达到600万；但是通常情况下我们不追求极致性能以及业务使用场景的并发量没那么大时大可不必这样设计。  
好，接下来的问题就是workid分配，其实这块百度的做法是比较简单的，每次应用重启时给数据库中插入一条记录（IP+PORT），然后返回主键id作为workid
```java
  @Transactional
    public long assignWorkerId() {
        WorkerNodeEntity workerNodeEntity = buildWorkerNode();
        workerNodeDAO.addWorkerNode(workerNodeEntity);
        LOGGER.info("Add worker node:" + workerNodeEntity);
        return workerNodeEntity.getId();
    }
    /**
     * Build worker node entity by IP and PORT
     */
    private WorkerNodeEntity buildWorkerNode() {
        WorkerNodeEntity workerNodeEntity = new WorkerNodeEntity();
        if (DockerUtils.isDocker()) {
            workerNodeEntity.setType(WorkerNodeType.CONTAINER.value());
            workerNodeEntity.setHostName(DockerUtils.getDockerHost());
            workerNodeEntity.setPort(DockerUtils.getDockerPort());

        } else {
            workerNodeEntity.setType(WorkerNodeType.ACTUAL.value());
            workerNodeEntity.setHostName(NetUtils.getLocalAddress());
            workerNodeEntity.setPort(System.currentTimeMillis() + "-" + RandomUtils.nextInt(100000));
        }
        return workerNodeEntity;
    }

```
　　显然可以看到，uid中给workid的份额有22位即2^22-1，但是数据库自增id最大值可以达到2^48-1；如果机器重启次数超过2^22-1，我们是不是应该考虑一下又一个坏味道，接下来就说说我们对此是怎么做的！

