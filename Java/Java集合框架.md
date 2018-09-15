#### 1、Java集合框架的结构。
[集合框架结构](https://www.jianshu.com/p/63e76826e852)
#### 2、ArrayList源码分析。
[ArrayList工作原理](https://yikun.github.io/2015/04/04/Java-ArrayList%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)     
[ArrayList源码](https://blog.csdn.net/ns_code/article/details/35568011)
- 无参构造方法构造的ArrayList的容量默认为10。
- 当容量不足以容纳当前的元素个数时，就设置新的容量为旧的容量的1.5倍加1。当容量不够时，每次增加元素，都要将原来的元素拷贝到一个新的数组中，非常之耗时，也因此建议在事先能确定元素数量的情况下，才使用ArrayList，否则建议使用LinkedList。
- ArrayList基于数组实现，可以通过下标索引直接查找到指定位置的元素，因此查找效率高，但每次插入或删除元素，就要大量地移动元素，插入删除元素的效率低。
- 在查找给定元素索引值等的方法中，源码都将该元素的值分为null和不为null两种情况处理，ArrayList中允许元素为null。
