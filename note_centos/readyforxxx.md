###java 
- hashMap  
数组结构+链表结构  (java 8 增加红黑树)

- arrayList, linkedList
- hash  ,B树，B+树，二叉搜索树，平衡二叉树，红黑树  
前中序遍历 

- lock 和 synchronized  
0.volatile/ 

1.synchronized 
关键字 jvm层



2.ReentranLock

Lock 接口   ReentranLock  readLock  WriteLock

lock(): 获取锁
unlock():释放锁
tryLock():尝试获取锁  boolean


Thread 状态  new ready blocked running  dead

新建  就绪  阻塞 运行  死亡 

Thread    sleep 方法

Object 的wait() 和 notify()  配置何用，线程的挂起 和 恢复

wait()  会释放对象锁  sleep() 不会释放




 

- 线程
ThreadPoolExecutor (核心线程数，最大线程数，空闲线程存活时间，时间单位，工作队列，拒绝策略)

### jvm  
.java->.class->jvm
方法区，堆区，栈区，本地方法栈，

GC:minor GC(新生代  )  /  FUll GC（老年代）           
 




### spring

- bean
    1. 默认单例
    2. 生命周期  // todo

- ioc aop


### mysql
- acid  
原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）

- innodb 和 MyISAM  
MyISAM和InnoDB两者的应用场景：
1) MyISAM管理非事务表。它提供高速存储和检索，以及全文搜索能力。如果应用中需要执行大量的SELECT查询，那么MyISAM是更好的选择。
2) InnoDB用于事务处理应用程序，具有众多特性，包括ACID事务支持。如果应用中需要执行大量的INSERT或UPDATE操作，则应该使用InnoDB，这样可以提高多用户并发操作的性能。

- 事务    
    1.隔离级别  
    * 
- 乐观锁 悲观锁  
    1.乐观锁 - 总是假设最好的情况，每次去拿数据都会认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此之间别人有没有去更新这个数据，可以使用版本号和CAS算法实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量。
  
    应用：数据库中的write_condition ,java.util.concurrent.atomic 使用cas实现  
    2.悲观锁- 总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次拿数据的时候都会上锁
    ，这样别人想拿这个数据就会阻塞直到它拿到锁(共享资源每次只给一个线程使用，其他的线程阻塞，用完后把资源转给其他线程)。  
    应用：传统的关系型数据库就用了这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在操作之前先上锁。java中的synchronized 和 ReentrantLock 等独占锁就是悲观锁的思想实现。

- 共享锁，排他锁

### redis

- 数据类型
  1. string 
  2. list
  3. hash (filed - value)
  4. set
  5. sortSet

- 单线程

- 

### 分布式

- zookeeper

- rpc

- 分布式事务


### 


###kafka 

- 消息队列



### 计算机网络

- 7层模型  
▪ 应用层
▪ 表示层
▪ 会话层
▪ 传输层
▪ 网络层
▪ 数据链路层
▪ 物理层

- 5层模型

1 应用层
2 传输层
3 网络层
4 数据链路层
5 物理层



### 自我介绍

    您好，我的名字叫做郭宏明，17年从苏州大学毕业之后直接来南京途牛工作了，在无线网站部门的时候，主要负责pc/m站列表页服务端的开发,现在在社群中心，主要负责苔客小程序的服务端开发。
    
   
1.读的接口并发量很高的话 1.redis 做缓存  2.依赖多个服务 网络io多的话，可以异步处理

2.写接口的话，可以先写redis 或者队列，然后做最终一致性

分布式 事务： 二段式提交，消息队列，实现最终一致性

分布式锁 ：分布式锁，redis ，数据库 ，zk

可重入锁 是什么 



树的复杂度

红黑树

快速排序
稳定: 否
时间复杂度:
最优时间: O(nlog(n))
最坏时间: O(n^2)
平均时间: O(nlog(n))

归并排序
归并排序是典型的分治算法，它不断地将某个数组分为两个部分，分别对左子数组与右子数组进行排序，然后将两个数组合并为新的有序数组。
稳定: 是
时间复杂度:
最优时间: O(nlog(n))
最坏时间: O(nlog(n))
平均时间: O(nlog(n))

桶排序
桶排序将数组分到有限数量的桶子里。每个桶子再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）。
时间复杂度:
最优时间: Ω(n + k)
最坏时间: O(n^2)
平均时间:Θ(n + k)



深度优先搜索
深度优先算法是一种优先遍历子节点而不是回溯的算法。
时间复杂度: O(|V| + |E|)

广度优先搜索
广度优先搜索是优先遍历邻居节点而不是子节点的图遍历算法。
时间复杂度: O(|V| + |E|)


运算符 













