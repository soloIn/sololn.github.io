# Java 容器

## 概况

java中的容器有 Collection 和 Map  两种，Collection 是存放对象的集合，Map是存放键值对的映射表



## Collection

![](https://soloin.github.io/galery/img/learn/collection.png)

1. Set 
   - TreeSet 基于红黑树实现，支持有序性操作（指定一个范围查找元素）,但查找效率不如HashSet.
   - HashSet 基于哈希表实现（HashMap），不支持有序性操作，但查找效率高。
   - LinkedHashSet 基于(LinkHashMap)实现，使用双向链表维护元素顺序，并有HashSet的查找效率
2. List
   - ArrayList 基于动态数组（不指定大小），支持随机访问
   - Vector 和ArrayList类似，线性安全
   - LinkedList  基于双向链表实现，方便插入删除，可以用来实现堆、栈、队列和双向队列
3. Queue
   - PriorityQueue 	基于堆实现，可以用来实现优先队列

## Map

![](https://soloin.github.io/galery/img/learn/Map.png)



