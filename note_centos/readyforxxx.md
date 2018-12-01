###java 
- hashMap  
数组结构+链表结构  (java 8 增加红黑树)

- arrayList, linkedList

- lock 和 synchronized

- 线程


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














