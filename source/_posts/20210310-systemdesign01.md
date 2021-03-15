---
title: 分布式ID和编号该怎么做？
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
　　从上面的分析我们知道一个问题是机器重启数大于workId时，产生的ID会有问题，去看了github也有人提出来，但是好像现在没有人维护了，所以我们自己动手。先设想一个场景，假如我们订单服务部署了1000台服务器（或者虚拟服务器），第一次项目上线，分配一千个workId给每个服务器，第二次上线在第一次的基础上往上加1000，如此往复上线4200次左右workId就用完了，再往后就有问题了，其实这个问题也比较好解决，我们只需要复用之前的workId就可以了，就算有机器突然宕机，重启之后继续往后加就是了；好，至此为止我们基本解决的workId耗尽的问题，但是，在设想一个场景，如果我们线上机器突然有随机的100台机器宕机了而且这个时候刚好到workId复用临界点了，我们继续复用原先的workId会有问题吗？当然会的，因为很有可能你复用的workId原先的宿主还活着，那完了，P0级事故；那我们再优化下，我们把这些活着的机器放在一个共享列表中，去拿workId时只需要校验是否在链表中即可。  
　　上述workId的分配都是基于mysql的，mysql是个重量级存储工具而且我们原先的用过的号会一直在数据库中存储，单机存储量也有限，而且多业务线需要统一发号时需要依赖多数据源甚至共享数据库；duck不必，试想如果我们将其迁移到Redis中，是不是会爽点，取号时只需要原子的加一就行了，活跃列表也不要了，我们对于每个workId我们定时过期就好了，达到最大值归零就好了这里会不会重复？如果服务到这里workId为1的早就过期了。  
　　对于分布式Id生成就写到这里，其实当有专门的的人力去开发时，单独一个ID生成挺浪费人力的，这个jar包还可以丰富各种业务编号生成，以及复杂的动态参数给业务方提供方便的页面配置等，让这个项目做的有意义些；后续有空我会搞一份单独的项目上传到github上。
