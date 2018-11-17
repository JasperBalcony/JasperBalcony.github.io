---
layout: post
title: "ThreadLocal内存泄露"
description: ThreadLocal内存泄露
category: 并发
---

# 关于ThreadLocal内存泄露的备忘


ThreadLocal从名字上来说就很好理解，就是用于线程（Thread）私有（Local）的存储结构，这种结构能够使得线程能够使用只有自己能够访问和修改的变量，从而实现多个线程之间的资源互相隔离，达到安全并发的目的。

也因此，ThreadLocal作为线程并发中的一种资源使用方式，得到了很广泛的应用，比如Spring MVC、Hibernate等。
不过值得一提的是，通常有人会讲ThreadLocal和synchronised等放在一起，作为形成安全并发的手段之一。其实我觉得这是比较容易使人误导的，因为两者的目的性完全不一样。
ThreadLocal主要的是用于独享自己的变量，避免一些资源的争夺，从而实现了空间换时间的思想。
而synchronised则主要用于临界（冲突）资源的分配，从而能够实现线程间信息同步，公共资源共享等，所以严格来说synchronised其实是能够实现ThreadLocal所需要的达到的效果的，只不过这样会带来资源争夺导致并发性能下降，而且还有synchronised、线程切换等一些可能不必要的开销。

对于ThreadLocal而言，其实使用起来有点像基础类型的装箱类型的感觉（个人觉得其实也可以算是一种装饰器模式的使用？），具体的使用就不在啰嗦了。下面就看看这次备忘的重点，如何导致内存泄漏的。

其实网上有的文章已经讲的听清楚的，觉得有张图特别好先引用到这里，来源于ThreadLocal可能引起的内存泄露：

[ThreadLocal可能引起的内存泄露](http://www.cnblogs.com/onlywujun/p/3524675.html)

![image](https://jasperbalcony.github.io/images/concurrent/ThreadLocal.png)


所以简单的说，主要原因就是在于TreadLocal中用到的自己定义的Map(和常用的Map接口不同)中，使用的Key值是一个`WeakReference`类型的值（弱引用会在下一次GC时马上释放而不管是否被引用）。那么如果这个Key在GC时被释放了，就会导致Value永远都不会被调用到，但是如果线程不结束，又一直存在。

因为可能不熟悉这部分内容的同学（例如几周以后的我）会感觉有点迷糊为什么这个图是这样的，就具体再解释一下细节点：

- 首先当然是看一下我们的主角ThreadLocal类，只保留了几个重点的地方，特别的是内部静态类的ThreadLocalMap是ThreadLocal自己实现的一个Map，而这个Map用使用了ThreadLocal作为了一个弱引用的Key（也就是主要问题点）。

_p.s.不知道各位第一次看的时候会不会跟我一样有种我是老子的儿子的同时又是老子的老子的感觉，哈哈哈_

```java
public class ThreadLocal<T> {
    
    //  获取Thread里面的Map
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    // （敲黑板）
    // 这里是重点！！！
    static class ThreadLocalMap {
        
        // 这里是凶器！！！
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        ... 
    }
    ... 
 }
 ```
- 接着不得不说的就是我们的大佬Thread类，里面关于ThreadLocal部分的内容主要是这样滴。我们可以看到这里主要是声明了ThreadLocal里面的Map作为类变量来提供给线程使用的。也正式因为如此，才会在ThreadLocal里面的getMap方法是拉取的Thread里面的Map。

_p.s. 感觉确实有点绕_
```java
public class Thread implements Runnable {

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. 
     */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```
    
- 于是到这里我们就明白了，其实每个Thread里面都有一个Map，Map里面的Key是ThreadLocal类的一个实例，之所以会比较混淆主要还是因为这里的Map又是ThreadLocal里面的一个内部静态类。

所以到这里其实有两个问题是暂时还没想通的，也希望有各位大佬指点一二：

- TreadLocalMap 其实是可以抽取成单独的类的？这样就使得逻辑和嵌套关系没有这么绕的感觉。
- 为什么只有Key要设计成WeakReference而不是Key和Value都是，或者这里为什么要设置弱引用？如果为了保护内存空间其实两者都是弱引用更好吧，是不是有什么其它考虑？


回归到内存泄露是因为WeakReference Key的问题，当然，Java的各位大佬肯定早就想到这个问题了，可以看到人家注释里面是这么说的，大意就是如果key==null的时候，就可以认为这个值无效了，可以调用expunged进行清理：

```java
/**
  * The entries in this hash map extend WeakReference, using
  * its main ref field as the key (which is always a
  * ThreadLocal object).  Note that null keys (i.e. entry.get()
  * == null) mean that the key is no longer referenced, so the
  * entry can be expunged from table.  Such entries are referred to
  * as "stale entries" in the code that follows.
  */
```  

而这个expungeStaleEntry方法在get、set时都会有间接的调用，而且remove方法中也会显示的调用，这也就是为什么有的文章中说通过在线程调用完成之后，通过调用remove方法能有效的杜绝该泄露问题的原因。

当然简单来说理解到这里就基本明了内存泄露的原因，但是其实再深入一点来说，如果泄露的原因是Key被释放，而Value没有释放，那么是否一定会有泄露呢？
答案当然是否定的，因为如果是一般的线程场景中，除了会调用expungeStaleEntry来进行清理，最差，在线程结束之时，自然也就消除了引用从而使得Value得以GC回收。

所以，会不会有线程一直不结束的场景呢？
当然答案是肯定的，最简单来说线程只要一直在wait就不会结束了，不过这种场景下其实和泄露也没啥关系的感觉。
其实最常用的线程一直不结束的场景，自然就是线程池了。因为这种情况下，线程是一直在不断的重复运行的，从而也就造成了value可能造成累积的情况。具体的模拟可以参考： 深入理解ThreadLocal的"内存溢出"

最后来做个总结吧，可能泄露的场景仅且仅在：

- 线程run方法结束后没有显示的调用remove进行清理
- 线程在线程池的模式下，一直重复运行


[转自](https://www.jianshu.com/p/250798f9ff76)