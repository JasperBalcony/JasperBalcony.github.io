---
layout: post
title: "Java中的hashcode和equals"
description: Java中的hashcode和equals
category: Java
---

# equals作用

equals() 的作用是 用来判断两个对象是否相等。

equals() 定义在JDK的Object.java中。通过判断两个对象的地址是否相等(即，是否是同一个对象)来区分它们是否相等。源码如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

既然Object.java中定义了equals()方法，这就意味着所有的Java类都实现了equals()方法，
所有的类都可以通过equals()去比较两个对象是否相等。 
但是，我们已经说过，使用默认的“equals()”方法，等价于“==”方法。
因此，我们通常会重写equals()方法：若两个对象的内容相等，则equals()方法返回true；否则，返回fasle。  

下面根据“类是否覆盖equals()方法”，将它分为2类。
- 若某个类没有覆盖equals()方法，当它的通过equals()比较两个对象时，
实际上是比较两个对象是不是同一个对象。这时，等价于通过“==”去比较这两个对象。
- 我们可以覆盖类的equals()方法，来让equals()通过其它方式比较两个对象是否相等。
通常的做法是：若两个对象的内容相等，则equals()方法返回true；否则，返回fasle。


顺便说一下java对equals()的要求。有以下几点：

- 对称性：如果x.equals(y)返回是"true"，那么y.equals(x)也应该返回是"true"。
- 反射性：x.equals(x)必须返回是"true"。
- 类推性：如果x.equals(y)返回是"true"，而且y.equals(z)返回是"true"，那么z.equals(x)也应该返回是"true"。
- 一致性：如果x.equals(y)返回是"true"，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是"true"。
- 非空性，x.equals(null)，永远返回是"false"；x.equals(和x不同类型的对象)永远返回是"false"。

# equals() 与 == 的区别是什么？

它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不试同一个对象。

# hashCode() 的作用

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。
这个哈希码的作用是确定该对象在哈希表中的索引位置。

hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

虽然，每个Java类都包含hashCode() 函数。但是，仅仅当创建并某个“类的散列表”(关于“散列表”见下面说明)时，该类的hashCode() 才有用(作用是：确定该类的每一个对象在散列表中的位置；其它情况下(例如，创建类的单个对象，或者创建类的对象数组等等)，类的hashCode() 没有作用。
上面的散列表，指的是：Java集合中本质是散列表的类，如HashMap，Hashtable，HashSet。

也就是说：hashCode() 在散列表中才有用，在其它情况下没用。在散列表中hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。


为了能理解后面的内容，这里简单的介绍一下散列码的作用。

**我们都知道，散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！
散列表的本质是通过数组实现的。当我们要获取散列表中的某个“值”时，实际上是要获取数组中的某个位置的元素。而数组的位置，就是通过“键”来获取的；更进一步说，数组的位置，是通过“键”对应的散列码计算得到的。
下面，我们以HashSet为例，来深入说明hashCode()的作用。**

假设，HashSet中已经有1000个元素。当插入第1001个元素时，需要怎么处理？因为HashSet是Set集合，它允许有重复元素。
“将第1001个元素逐个的和前面1000个元素进行比较”？显然，这个效率是相等低下的。散列表很好的解决了这个问题，它根据元素的散列码计算出元素在散列表中的位置，然后将元素插入该位置即可。对于相同的元素，自然是只保存了一个。
由此可知，若两个元素相等，它们的散列码一定相等；但反过来确不一定。在散列表中，

- 如果两个对象相等，那么它们的hashCode()值一定要相同；
- 如果两个对象hashCode()相等，它们并不一定相等。

注意：这是在散列表中的情况。在非散列表中一定如此！

# hashCode() 和 equals() 的关系

接下面，我们讨论另外一个话题。网上很多文章将 hashCode() 和 equals 关联起来，有的讲的不透彻，
有误导读者的嫌疑。在这里，我自己梳理了一下 “hashCode() 和 equals()的关系”。

我们以“类的用途”来将“hashCode() 和 equals()的关系”分2种情况来说明。

## 第一种不会创建“类对应的散列表”

这里所说的“不会创建类对应的散列表”是说：我们不会在HashSet, Hashtable, HashMap等等这些本质是
散列表的数据结构中，用到该类。例如，不会创建该类的HashSet集合。

**在这种情况下，该类的“hashCode() 和 equals() ”没有半毛钱关系的！**

这种情况下，equals() 用来比较该类的两个对象是否相等。而hashCode() 则根本没有任何作用，
所以，不用理会hashCode()。

## 第二种 会创建“类对应的散列表”
   
这里所说的“会创建类对应的散列表”是说：我们会在HashSet, Hashtable, HashMap等等这些本质是
散列表的数据结构中，用到该类。例如，会创建该类的HashSet集合。

**在这种情况下，该类的“hashCode() 和 equals() ”是有关系的**：
- 如果两个对象相等，那么它们的hashCode()值一定相同。这里的相等是指，通过equals()比较两个对象时返回true。
- 如果两个对象hashCode()相等，它们并不一定相等。因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等。补充说一句：“两个不同的键值对，哈希值相等”，这就是哈希冲突。

此外，在这种情况下。若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。
例如，创建Person类的HashSet集合，必须同时覆盖Person类的equals() 和 hashCode()方法。
如果单单只是覆盖equals()方法。我们会发现，equals()方法没有达到我们想要的效果。


一般来说涉及到对象之间的比较大小就需要重写equals方法，**但是为什么第一点说重写了equals就需要重写hashCode呢**？

**实际上这只是一条规范**，如果不这样做程序也可以执行，只不过会隐藏bug。一般一个类的对象如果会存储在HashTable，HashSet,HashMap
等散列存储结构中，那么重写equals后最好也重写hashCode，否则会导致存储数据的不唯一性（存储了两个equals相等的数据）。
而如果确定不会存储在这些散列结构中，则可以不重写hashCode。但是个人觉得还是重写比较好一点，
谁能保证后期不会存储在这些结构中呢，况且重写了hashCode也不会降低性能，因为在线性结构（如ArrayList）中是不会调用hashCode，
所以重写了也不要紧，也为后期的修改打了补丁。

# 总结
- hashCode是为了提高在散列结构存储中查找的效率，在线性表中没有作用。
- equals和hashCode需要同时覆盖。
- 若两个对象equals返回true，则hashCode有必要也返回相同的int数。
- 若两个对象equals返回false，则hashCode不一定返回不同的int数,但为不相等的对象生成不同hashCode值可以提高哈希表的性能。
- 若两个对象hashCode返回相同int数，则equals不一定返回true。
- 若两个对象hashCode返回不同int数，则equals一定返回false。
- 同一对象在执行期间若已经存储在集合中，则不能修改影响hashCode值的相关信息，否则会导致内存泄露问题。

转自

[Java hashCode() 和 equals()的若干问题解答](https://www.cnblogs.com/skywang12345/p/3324958.html)

[彻底搞懂hashCode与equals的作用与区别及应当注意的细节](https://blog.csdn.net/haobaworenle/article/details/53819838)

