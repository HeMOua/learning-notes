# Java集合

## List, Set, Map 区别

+ List：有序集合，可以存放相同的元素
+ Set：无序集合，不可重复存储元素
+ Map：以键值对的方式存储元素，key 是无序且唯一的，value 不唯一，不同的 key 可以对应相同的 value

## 常用集合的底层数据结构

+ List
  + ArrayList：使用数组实现
  + LinkedList：双向链表
+ Set
  + HashSet：基于 HashMap 实现，存入数据的方式和 HashMap 的 key 是一致的
  + TreeSet：红黑树
+ Map
  + HashMap：数组+链表/红黑树
    + 当链表长度大于 8，则链表转化为红黑树
    + 当长度小于6，则红黑树转化为链表
  + HashTable：数组+链表
  + TreeMap：红黑树

## 那些集合类是线程安全的？

+ `vector`
+ `stack`
+ `HashTable`

## 迭代器 `Iterator`

Java 迭代器，可以用来遍历任何 `Collection` 接口

## 集合的快速失败和安全失败机制

1、快速失败

当多个线程对集合中的内容进行修改时可能会抛出 `ConcurrentModificationException` 异常，还有在单线程中如果使用增强 for 对集合一边遍历一遍修改也会产生这个异常。

这种机制可以提醒开发者在多线程环境下避免出现意外的并发修改。

2、安全失败

采用安全失败机制的集合容器，会对原集合进行拷贝，在拷贝的集合上进行遍历，因此对原集合的修改不会影响到迭代器，就不会发生`ConcurrentModificationException` 这个异常了。缺点是遍历集合时，原集合发生修改，迭代器也无法访问修改后的内容。

java.util.concurrent 包下的容器都是安全失败，可以在多线程下并发使用。



