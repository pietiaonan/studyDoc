## 结构

同步器的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作，（**模板方法模式**）。

AQS使用了一个int型成员变量state来表示线程的同步状态，通过内置的同步队列（FIFO双向队列）来完成管理线程同步状态的工作，一旦当前线程没有竞争到锁，同步队列会将当前线程以及线程的状态放在一个node节点中维护，并阻塞当前线程，等待被唤醒再次重新尝试获取锁或者被取消等待

![微信图片_20200318095846](..\\..\\img\\thread\\AQS1.png)

### Node



节点是构成同步队列的基础，其中存放了获取同步状态失败的线程引用、线程状态、前驱节点、后继节点、节点属性（共享、独占）等，同步队列的基本结构图如下。pre和next不是初始化创建，避免浪费，只有第一次有竞争（contention）才创建。



![微信图片_20200318095846](..\\..\\img\\thread\\Node1.jpg)





### 属性



```java
    //自旋时间 ，大于1000 纳秒 ，LockSupport.park(ns);
    static final long spinForTimeoutThreshold = 1000L;
    // 头节点
    private transient volatile Node head;
	// 尾节点
    private transient volatile Node tail;
    // 同步状态
    private volatile int state;
```

## ConditionObject


await、signal  没有加锁操作，里面的头尾接节点也没有加volatile，因为只有拿到锁的线程才会执行改方法，不存在竞争

```java
        /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;
```
