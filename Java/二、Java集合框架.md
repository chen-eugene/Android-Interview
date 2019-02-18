#### [1、Java集合框架的结构。](https://www.jianshu.com/p/63e76826e852)

![Java集合框架](https://github.com/chen-eugene/Android-Interview/blob/master/image/2243690-9cd9c896e0d512ed.jpg)

#### 2、ArrayList实现原理。
[ArrayList工作原理](https://yikun.github.io/2015/04/04/Java-ArrayList%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)     
[ArrayList源码](https://blog.csdn.net/ns_code/article/details/35568011)
- 无参构造方法构造的ArrayList的容量默认为10。
- 当容量不足以容纳当前的元素个数时，就设置新的容量为旧的容量的1.5倍加1。当容量不够时，每次增加元素，都要将原来的元素拷贝到一个新的数组中，非常之耗时，也因此建议在事先能确定元素数量的情况下，才使用ArrayList，否则建议使用LinkedList。
- ArrayList基于数组实现，可以通过下标索引直接查找到指定位置的元素，因此查找效率高，但每次插入或删除元素，就要大量地移动元素，插入删除元素的效率低。
- 在查找给定元素索引值等的方法中，源码都将该元素的值分为null和不为null两种情况处理，ArrayList中允许元素为null。
#### 3、LinkedList实现原理。
[LinkedList工作原理](https://yikun.github.io/2015/04/05/Java-LinkedList%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)   
[LinkedList源码](https://blog.csdn.net/ns_code/article/details/35787253)
- LinkedList的实现是基于双向循环链表的，因此不存在容量不足的问题，且头结点中不存放数据。
- LinkedList是基于链表实现的，因此插入删除效率高，查找效率低。
 #### 4、HashMap实现原理。
[HashMap工作原理](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)        
[HashMap源码](https://blog.csdn.net/ns_code/article/details/36034955)
- 在Java 8之前的实现中是用链表解决冲突的，在产生碰撞的情况下，进行get时，两步的时间复杂度O(1)+O(n)。因此，当碰撞很厉害的时候n很大，O(n)的速度显然是影响速度的。因此在Java 8中，利用红黑树替换链表，这样复杂度就变成了O(1)+O(logn)了，这样在n很大的时候，能够比较理想的解决这个问题。
- 当put时，如果发现目前的bucket占用程度已经超过了Load Factor所希望的比例，那么就会发生resize。在resize的过程，简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中。
#### 5、LinkedHashMap实现原理。
[LinkedHashMap工作原理](https://yikun.github.io/2015/04/02/Java-LinkedHashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)    
[LinkedHashMap源码](https://blog.csdn.net/ns_code/article/details/37867985)   
![LinkedHashMap结构图](https://github.com/chen-eugene/Interview/blob/master/image/v2-a821a76fb84ce6223598c89ae8ebe7b0_hd.jpg)


#### [6、ConcurrentHashMap的原理。](https://www.cnblogs.com/dolphin0520/p/3932905.html)
  - get方法没有加锁。
  - 默认情况下分为16个segment，segment之间不存在竞争关系，只要线程不是访问的同一个segment，那么就没有锁竞争，默认情况下允许16条线程并发的访问不会回发生竞争。
  - 访问ConcurrentHashMap会进程两次的hash计算，第一次确定是在哪一个segment中，第二次是确定key的索引位置。
  

#### [7、什么是fast-fail机制。](https://blog.csdn.net/javazejian/article/details/53073995#%E7%90%86%E8%A7%A3iterator%E6%8E%A5%E5%8F%A3)

 **先说说为什么需要迭代器(Iterator)？**
 
 试想一下，我们在遍历ArrayList等集合类的时候通常可以采用for循环，但是如果随便的一个集合类，在不知道其api或者内部结构的情况下如何进行遍历呢？这时就需要迭代器。
 
 迭代器是一种标准化遍历容器类的工具类。用于遍历各种集合类。它可以避免暴露集合的内部结构，只要该集合内部实现了迭代器，那么就可以在不知道API或者集合类内部结构的情况下通过迭代器遍历该集合的所有元素。
 
 **fast-fail机制：**
 
 迭代器和集合类之间还存在着一个严重的问题，那就是怎么保证数据的一致性，比如说当我们通过迭代器遍历ArraryList的时候，已经通过hashNext方法判断了，正在调用next方法，但是我们有调用了ArrayList本身的remove方法，这样就会造成迭代器获取的数据和ArrayList的数据不一致的情况。
 
 fast-fail机制是java集合保证迭代器数据和集合本身数据是否一致的一种机制，当多个线程直接修改集合的结构而没有通知迭代器的时候就有可能触发fast-fail机制，这里是有可能触发，而不是一定触发。
 

#### [8、SparseArray的原理，与HashMap的区别。](https://extremej.itscoder.com/sparsearray_source_analyse/)
 
 **原理：**
 - 在Android中，SparseArray是用来取代HashMap的，它的key默认是int类型。
 - 在它的内部使用的int[]数组来存放key，使用Object[]数组来存放value。
 - 存放key的数组是有序的，采用二分查找法计算key应该插入的位置。
 - 如果有冲突，就直接覆盖掉原来的值，并不会返回原值（HashMap会返回原值）。
 - 如果当前要插入的 key 的索引上的值为DELETE，直接覆盖。
 - 前几步都失败了，检查是否需要gc()并且在该索引上插入数据。
 
 **与HashMap的区别：**
 - 使用int[]数组存放key，避免了HashMap中基本数据类型装箱的消耗。
 - 没有使用HashMap中的结构体Entry，节省了内存。
















