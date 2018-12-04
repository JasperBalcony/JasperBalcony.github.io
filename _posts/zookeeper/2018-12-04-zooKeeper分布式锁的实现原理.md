---
layout: post
title: "zooKeeper分布式锁的实现原理"
description: zooKeeper分布式锁的实现原理
category: zookeeper
---

# ZooKeeper分布式锁机制

接下来我们一起来看看，多客户端获取及释放zk分布式锁的整个流程及背后的原理。

首先大家看看下面的图，如果现在有两个客户端一起要争抢zk上的一把分布式锁，会是个什么场景？

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-1.jpg)

如果大家对zk还不太了解的话，建议先自行百度一下，简单了解点基本概念，比如zk有哪些节点类型等等。

参见上图。zk里有一把锁，这个锁就是zk上的一个节点。然后呢，两个客户端都要来获取这个锁，具体是怎么来获取呢？

咱们就假设客户端A抢先一步，对zk发起了加分布式锁的请求，这个加锁请求是用到了zk中的一个特殊的概念，叫做“**临时顺序节点**”。

简单来说，就是直接在"my_lock"这个锁节点下，创建一个顺序节点，这个顺序节点有zk内部自行维护的一个节点序号。

比如说，第一个客户端来搞一个顺序节点，zk内部会给起个名字叫做：xxx-000001。然后第二个客户端来搞一个顺序节点，zk可能会起个名字叫做：xxx-000002。大家注意一下，**最后一个数字都是依次递增的**，从1开始逐次递增。zk会维护这个顺序。

所以这个时候，假如说客户端A先发起请求，就会搞出来一个顺序节点，大家看下面的图，Curator框架大概会弄成如下的样子：

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-2.jpg)

大家看，客户端A发起一个加锁请求，先会在你要加锁的node下搞一个临时顺序节点，这一大坨长长的名字都是Curator框架自己生成出来的。

然后，那个最后一个数字是"1"。大家注意一下，因为客户端A是第一个发起请求的，所以给他搞出来的顺序节点的序号是"1"。

接着客户端A创建完一个顺序节点。还没完，他会查一下"my_lock"这个锁节点下的所有子节点，并且这些子节点是按照序号排序的，这个时候他大概会拿到这么一个集合：

```
[zk: localhost:2181(CONNECTED) 32] ls /zktest/lock
[ _c_ddccd2b0-a124-474c-9dfb-31e4da4e4b5b-lock-0000000001]
```
接着客户端A会走一个关键性的判断，就是说：唉！兄弟，这个集合里，我创建的那个顺序节点，是不是排在第一个啊？

如果是的话，那我就可以加锁了啊！因为明明我就是第一个来创建顺序节点的人，所以我就是第一个尝试加分布式锁的人啊！

bingo！加锁成功！大家看下面的图，再来直观的感受一下整个过程。

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-3.jpg)

接着假如说，客户端A都加完锁了，客户端B过来想要加锁了，这个时候他会干一样的事儿：先是在"my_lock"这个锁节点下创建一个临时顺序节点，此时名字会变成类似于：

```
[ _c_84ce1f91-0159-47f1-9ec2-49d2c9395530-lock-0000000002]
```
大家看看下面的图：

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-4.jpg)

客户端B因为是第二个来创建顺序节点的，所以zk内部会维护序号为"2"。

接着客户端B会走加锁判断逻辑，查询"my_lock"锁节点下的所有子节点，按序号顺序排列，此时他看到的类似于：

```
[zk: localhost:2181(CONNECTED) 32] ls /zktest/lock
[ _c_ddccd2b0-a124-474c-9dfb-31e4da4e4b5b-lock-0000000001, _c_84ce1f91-0159-47f1-9ec2-49d2c9395530-lock-0000000002]
```


同时检查自己创建的顺序节点，是不是集合中的第一个？

明显不是啊，此时第一个是客户端A创建的那个顺序节点，序号为"01"的那个。所以加锁失败！

加锁失败了以后，客户端B就会通过ZK的API对他的顺序节点的上一个顺序节点加一个监听器。zk天然就可以实现对某个节点的监听。

如果大家还不知道zk的基本用法，可以百度查阅，非常的简单。客户端B的顺序节点是：

```
[_c_84ce1f91-0159-47f1-9ec2-49d2c9395530-lock-0000000002]
```
他的上一个顺序节点，不就是下面这个吗？

```
[ _c_ddccd2b0-a124-474c-9dfb-31e4da4e4b5b-lock-0000000001]
```
即客户端A创建的那个顺序节点！

所以，客户端B会对：
```
[ _c_ddccd2b0-a124-474c-9dfb-31e4da4e4b5b-lock-0000000001]
```
这个节点加一个监听器，监听这个节点是否被删除等变化！大家看下面的图。

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-5.jpg)

接着，客户端A加锁之后，可能处理了一些代码逻辑，然后就会释放锁。那么，释放锁是个什么过程呢？

其实很简单，就是把自己在zk里创建的那个顺序节点，也就是：
```
[ _c_ddccd2b0-a124-474c-9dfb-31e4da4e4b5b-lock-0000000001]
```
这个节点给删除。

删除了那个节点之后，zk会负责通知监听这个节点的监听器，也就是客户端B之前加的那个监听器，说：兄弟，你监听的那个节点被删除了，有人释放了锁。

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-6.jpg)

此时客户端B的监听器感知到了上一个顺序节点被删除，也就是排在他之前的某个客户端释放了锁。

此时，就会通知客户端B重新尝试去获取锁，也就是获取"my_lock"节点下的子节点集合，此时为：
```
[_c_84ce1f91-0159-47f1-9ec2-49d2c9395530-lock-0000000002]
```

集合里此时只有客户端B创建的唯一的一个顺序节点了！

然后呢，客户端B判断自己居然是集合中的第一个顺序节点，bingo！可以加锁了！直接完成加锁，运行后续的业务代码即可，运行完了之后再次释放锁。

![image](https://jasperbalcony.github.io/images/zookeeper/zk-lock-7.jpg)

# 总结

其实如果有客户端C、客户端D等N个客户端争抢一个zk分布式锁，原理都是类似的。

- 大家都是上来直接创建一个锁节点下的一个接一个的临时顺序节点
- 如果自己不是第一个节点，就对自己上一个节点加监听器
- 只要上一个节点释放锁，自己就排到前面去了，相当于是一个排队机制。

而且用临时顺序节点的另外一个用意就是，如果某个客户端创建临时顺序节点之后，不小心自己宕机了也没关系，zk感知到那个客户端宕机，会自动删除对应的临时顺序节点，相当于自动释放锁，或者是自动取消自己的排队。

最后，咱们来看下用Curator框架进行加锁和释放锁的一个过程：

```java
package com.tiffany.client;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.RetryNTimes;
import org.junit.Test;
import org.slf4j.LoggerFactory;

import java.util.concurrent.TimeUnit;

/**
 * 分布式编程时，比如最容易碰到的情况就是应用程序在线上多机部署，于是当多个应用同时访问某一资源时，
 * 就需要某种机制去协调它们。例如，现在一台应用正在rebuild缓存内容，
 * 要临时锁住某个区域暂时不让访问；又比如调度程序每次只想一个任务被一台应用执行等等。
 * <p>
 * 下面的程序会启动两个线程t1和t2去争夺锁，拿到锁的线程会占用5秒。运行多次可以观察到，
 * 有时是t1先拿到锁而t2等待，有时又会反过来。Curator会用我们提供的lock路径的结点作为全局锁，
 * 这个结点的数据类似这种格式：[_c_64e0811f-9475-44ca-aa36-c1db65ae5350-lock-0000000005]，
 * 每次获得锁时会生成这种串，释放锁时清空数据。
 *
 * @author wangxinguo
 * @date 2016-9-3
 */
public class CuratorDistrLockTest extends BaseTest {

    private final static org.slf4j.Logger logger = LoggerFactory.getLogger(CuratorDistrLockTest.class);

    private static final String ZK_LOCK_PATH = "/zktest/lock";

    @Test
    public void lockTest() throws Exception {
        // 1.Connect to zk
        final CuratorFramework client = CuratorFrameworkFactory.newClient(ZK_ADDRESS, new RetryNTimes(10, 5000));
        client.start();
        logger.info("zk client start successfully!");

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                doWithLock(client);
            }
        }, "t1");

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                doWithLock(client);
            }
        }, "t2");

        Thread t3 = new Thread(new Runnable() {
            @Override
            public void run() {
                doWithLock(client);
            }
        }, "t3");

        t1.start();
        t2.start();
        t3.start();

        Thread.sleep(Integer.MAX_VALUE);
    }


    private static void doWithLock(CuratorFramework client) {
        InterProcessMutex lock = new InterProcessMutex(client, ZK_LOCK_PATH);
        try {
            if (lock.acquire(10 * 1000, TimeUnit.SECONDS)) {
                logger.info(Thread.currentThread().getName() + " hold lock");
                Thread.sleep(5000L);
                logger.info(Thread.currentThread().getName() + " release lock");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                lock.release();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }

}

```

运行结果
```
2018-12-04 10:22:27 [t3] INFO com.tiffany.client.CuratorDistrLockTest  - t3 hold lock
2018-12-04 10:22:32 [t3] INFO com.tiffany.client.CuratorDistrLockTest  - t3 release lock
2018-12-04 10:22:32 [t2] INFO com.tiffany.client.CuratorDistrLockTest  - t2 hold lock
2018-12-04 10:22:37 [t2] INFO com.tiffany.client.CuratorDistrLockTest  - t2 release lock
2018-12-04 10:22:37 [t1] INFO com.tiffany.client.CuratorDistrLockTest  - t1 hold lock
2018-12-04 10:22:42 [t1] INFO com.tiffany.client.CuratorDistrLockTest  - t1 release lock
```

zookeeper锁节点
```
[zk: localhost:2181(CONNECTED) 32] ls /zktest/lock
[_c_7bf8662c-862d-449c-b9c5-3376ee572135-lock-0000000002, _c_ddccd2b0-a124-474c-9dfb-31e4da4e4b5b-lock-0000000000, _c_84ce1f91-0159-47f1-9ec2-49d2c9395530-lock-0000000001]
[zk: localhost:2181(CONNECTED) 35] ls /zktest/lock
[_c_7bf8662c-862d-449c-b9c5-3376ee572135-lock-0000000002, _c_84ce1f91-0159-47f1-9ec2-49d2c9395530-lock-0000000001]
[zk: localhost:2181(CONNECTED) 38] ls /zktest/lock
[_c_7bf8662c-862d-449c-b9c5-3376ee572135-lock-0000000002]
[zk: localhost:2181(CONNECTED) 39] ls /zktest/lock
[]
```

转自 

[七张图彻底讲清楚ZooKeeper分布式锁的实现原理](https://www.toutiao.com/i6629671372463276552/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1543884311&app=news_article_lite&utm_source=weixin&iid=52867771468&utm_medium=toutiao_ios&group_id=6629671372463276552)

参考 

[一文看懂的 Zookeeper 分布式锁](https://www.toutiao.com/a6630942959032336909/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1543890723&app=news_article_lite&utm_source=weixin&iid=52867771468&utm_medium=toutiao_ios&group_id=6630942959032336909)

[跟着实例学习ZooKeeper的用法： 分布式锁](http://ifeve.com/zookeeper-lock/)
