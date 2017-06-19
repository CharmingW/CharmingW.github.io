# 引言

> 俗话说：程序 = 数据结构 + 算法，搞定数据结构就相当于搞定了一半。数据结构是以某种形式将数据组织在一起的集合，它不仅存储数据，还支持访问和处理数据的操作。Java 提供了几个能有效地组织和操作数据的数据结构，这些数据结构通常称为 Java 集合框架。在平常的学习开发中，灵活熟练地使用这些集合框架，可以很明显地提高我们的开发效率，当然仅仅会用还是不够的，理解其中的设计思想与原理才能更好地提高我们的开发水平。

![Java集合框架图](/images/20160918105654_491.jpg)

![Java集合框架简化图](/images/1337080070_5795.jpg)

![Collection接口方法](/images/2017-05-09_153916.png)

在 Java 2 之前，Java 是没有完整的集合框架的。它只有一些简单的可以自扩展的容器类，比如 `Vector` `，Stack` ， `Hashtable` 等。这些容器类在使用的过程中由于效率问题饱受诟病，因此在Java 2中，Java设计者们进行了大刀阔斧的整改，重新设计，于是就有了现在的集合框架。需要注意的是，之前的那些容器类库并没有被弃用而是进行了保留，主要是为了向下兼容的目的，但我们在平时使用中还是应该尽量少用。

Java 集合框架经典问题的第一篇，就来讲讲 `HashMap` 和 `Hashtable` 的异同之处

# HashMap & Hashtable

1. 定义
2. 原理
3. 相同点
4. 区别
  - 线程安全
  - 存储形式
  - 迭代器
  - 使用性能
5. 面试题
6. 总结

# 定义

```Java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable

public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable
```

`Hashtable` 继承于陈旧的 `Dictionary`类，HashMap继承于 `AbstractMap`

# 原理

![put方法流程图](/images/Image.png)

![HashMap存储](/images/HashMap存储.png)

HashMap基于hashing原理，我们通过 `put`() 和 `get`() 方法储存和获取对象。当我们将键值对传递给put()方法时，它调用键对象的 `hashCode`() 方法来计算 `hashcode` ，让后找到 `bucket` 位置来储存值对象。当获取对象时，通过键对象的 `equals`() 方法找到正确的键值对，然后返回值对象。 `HashMap` 使用链表来解决碰撞问题，当发生碰撞了，对象将会储存在链表的下一个节点中。  `HashMap` 在每个链表节点中储存键值对对象。

当两个不同的键对象的 `hashcode` 相同时会发生什么？ 它们会储存在同一个 `bucket` 位置的链表中。key 对象的 `equals`() 方法用来找到键值对，key的存储是唯一的，当 `put()` 方法执行时 key 的 `equals()` 返回true，则不再存储这个 (key, value)。

关键字：hashcode、equals

# 相同点

`HashMap` 和 `Hashtable` 采用相同的存储机制，二者实现基本一致。

# 区别

## 1.线程安全

```java
//Hashtable
public synchronized V put(K key, V value)

//HashMap
public V put(K key, V value)
```

`Hashtable` 的方法基本都有 `synchronized` 关键字修饰，是线程安全的，`HashMap` 是不是线程安全的，但可以通过以下这种方式来同步

```java
Map m = Collections.synchronizeMap(hashMap)
```
如果你不需要同步，则使用`HashMap`即可

## 2.存储形式

```java
//Hashtable
public synchronized V put(K key, V value) {
       // Make sure the value is not null
       if (value == null) {
           throw new NullPointerException();
       }

       // Makes sure the key is not already in the hashtable.
       Entry<?,?> tab[] = table;
       int hash = key.hashCode();
       int index = (hash & 0x7FFFFFFF) % tab.length;
       @SuppressWarnings("unchecked")
       Entry<K,V> entry = (Entry<K,V>)tab[index];
       for(; entry != null ; entry = entry.next) {
           if ((entry.hash == hash) && entry.key.equals(key)) {
               V old = entry.value;
               entry.value = value;
               return old;
           }
       }

       addEntry(hash, key, value, index);
       return null;
   }
```
`Hashtable` 不允许存储 `null` 的 key 和 value， `HashMap` 则可以。

## 3.迭代器

`HashMap` 的迭代器`Iterator`是 `fail-fast` 迭代器，而`Hashtable`的`enumerator`迭代器不是`fail-fast`的，所以当有其它线程改变了`Hash`Map的结构（增加或者移除元素），将会抛出`ConcurrentModificationException`，但迭代器本身的`remove()`方法移除元素则不会抛出`ConcurrentModificationException`异常。

## 4.使用性能

`HashMap`的性能无疑要好于 `Hashtable`，因为 `Hashtable` 每次使用都要同步，这会损失一定的速度，单线程运行时使用 `HashMap`。在 Java 5 之后， `ConcurrentHashMap` 成了 `Hashtable` 的替代品，所以需要同步时，请使用 `ConcurrentHashMap`。

# 面试题

### **“你用过HashMap吗？” “什么是HashMap？你为什么用到它？”**

几乎每个人都会回答“是的”，然后回答HashMap的一些特性，譬如HashMap可以接受null键值和值，而Hashtable则不能；HashMap是非synchronized;HashMap很快；以及HashMap储存的是键值对等等。这显示出你已经用过HashMap，而且对它相当的熟悉。但是面试官来个急转直下，从此刻开始问出一些刁钻的问题，关于HashMap的更多基础的细节。面试官可能会问出下面的问题

### **“你知道HashMap的工作原理吗？” “你知道HashMap的get()方法的工作原理吗？”**

你也许会回答“我没有详查标准的Java API，你可以看看Java源代码或者Open JDK。”“我可以用Google找到答案。”
但一些面试者可能可以给出答案，“HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关键点在于指出，HashMap是在bucket中储存键对象和值对象，作为Map.Entry。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道hashing以及HashMap的工作原理。但是这仅仅是故事的开始，当面试官加入一些Java程序员每天要碰到的实际场景的时候，错误的答案频现。下个问题可能是关于HashMap中的碰撞探测(collision detection)以及碰撞的解决方法：

### **“当两个对象的hashcode相同会发生什么？” **

从这里开始，真正的困惑开始了，一些面试者会回答因为hashcode相同，所以两个对象是相等的，HashMap将会抛出异常，或者不会存储它们。然后面试官可能会提醒他们有equals()和hashCode()两个方法，并告诉他们两个对象就算hashcode相同，但是它们可能并不相等。一些面试者可能就此放弃，而另外一些还能继续挺进，他们回答“因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用链表存储对象，这个Entry(包含有键值对的Map.Entry对象)会存储在链表中。”这个答案非常的合理，虽然有很多种处理碰撞的方法，这种方法是最简单的，也正是HashMap的处理方法。但故事还没有完结，面试官会继续问：

### **“如果两个键的hashcode相同，你如何获取值对象？” **

面试者会回答：当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。面试官提醒他如果有两个值对象储存在同一个bucket，他给出答案:将会遍历链表直到找到值对象。面试官会问因为你并没有值对象去比较，你是如何确定确定找到值对象的？除非面试者直到HashMap在链表中存储的是键值对，否则他们不可能回答出这一题。

其中一些记得这个重要知识点的面试者会说，找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象。完美的答案！

许多情况下，面试者会在这个环节中出错，因为他们混淆了hashCode()和equals()方法。因为在此之前hashCode()屡屡出现，而equals()方法仅仅在获取值对象的时候才出现。一些优秀的开发者会指出使用不可变的、声明作final的对象，并且采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

如果你认为到这里已经完结了，那么听到下面这个问题的时候，你会大吃一惊。“如果HashMap的大小超过了负载因子(load factor)定义的容量，怎么办？”除非你真正知道HashMap的工作原理，否则你将回答不出这道题。默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

如果你能够回答这道问题，下面的问题来了：“你了解重新调整HashMap大小存在什么问题吗？”你可能回答不上来，这时面试官会提醒你当多线程的情况下，可能产生条件竞争(race condition)。

当重新调整HashMap大小的时候，确实存在条件竞争，因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在链表中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在链表的尾部，而是放在头部，这是为了避免尾部遍历(tail traversing)。如果条件竞争发生了，那么就死循环了。这个时候，你可以质问面试官，为什么这么奇怪，要在多线程的环境下使用HashMap呢？）

# 总结

1. 注意理解存储过程中 `hashcode()` 和 `equals()` 方法的作用
2. 了解常见的哈希函数构造方法和解决冲突方法，如 `HashMap` 用的就是拉链法来解决冲突
2. `HashMap` 除了线程不安全外，其他基本都优于 `Hashtable`
3. `Hashtable` 基本已经被弃用，在 Java 5 之后用 `ConcurrentHashMap` 来替代
