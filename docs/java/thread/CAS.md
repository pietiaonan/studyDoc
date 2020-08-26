## 概念 

CAS,compare and swap的缩写，在intel的CPU中，使用cmpxchg指令。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。

整个J.U.C都是建立在CAS之上的，因此对于synchronized阻塞算法，J.U.C在性能上有了很大的提升。

CAS适合轻量级的并发操作，也就是并发量并不多，而且等待时间不长的情况，否则就应该使用普通锁，进入阻塞状态，避免CPU空转。

## 原理

利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。



处理器自动保证基本内存操作的原子性

首先说明，处理器会自动保证基本的内存操作是原子性的。处理器保证从系统内存中读取或写入一个字节是原子的。意思是，当一个处理器读取一个字节时，其他处理器不能访问这个字节的内存地址

当然， long和 double类型在32位操作系统中的读写操作不是原子的，因为 long和 double占64位，需要分成2个步骤来处理，在读写时分别拆成2个字节进行读写。因此 long和 double类型的数据在进行计算时需要注意这个问题。

- 使用总线锁保证原子性

当多个处理器调度线程时，同时读取主内存中的共享变量，有安全问题，处理器使用总线锁就是解决这个问题的。总线锁就是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该处理器可以独占使用共享内存。

- 使用缓存锁保证原子性 （缓存一致性协议实现，如MESI）

总线锁把CPU和内存之间通信锁住，使得锁定期间，其他处理器不能操作其他内存地址的数据，所以总线锁的开销比较大。所以在某些场合使用缓存锁替代总线锁。

缓存锁是指通过锁住CPU缓存，在CPU缓存区实现共享变量的原子性操作。



在两种情况下处理器不会使用缓存锁。

1、当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行（cache line），则处理器会调用总线锁定。

2、有些处理器不支持缓存锁定。

CAS 底层是靠调用 CPU 指令集的 cmpxchg 完成的，它是 x86 和 Intel 架构中的 compare and exchange 指令。在多核的情况下，这个指令也不能保证原子性，需要在前面加上 lock 指令。lock 指令可以保证一个 CPU 核心在操作期间独占一片内存区域







- CAS缺点

在Java并发包中有一些并发框架也使用了自旋CAS的方式实现了原子操作，比如：LinkedTransferQueue类的Xfer方法。CAS虽然很高效的解决了原子操作，但是CAS仍然存在三大问题：**ABA问题、循环时间长开销大、只能保证一个共享变量的原子操作**

**1.ABA问题解决方法**

1、使用版本号

ABA问题的解决思路是使用版本号，每次变量更新的时候版本号加1，那么A->B->A就会变成1A->2B->3A

2、jdk自带原子变量

从jdk1.5开始，jdk的Atomic包里就提供了一个类AtomicStampedReference来解决ABA问题，这个类中的compareAndSet方法的作用就是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值更新为指定的新值

**2.循环时间长开销大**

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果jvm能支持处理器提供的pause指令，那么效率会有一定的提升。pause指令有两个作用：

第一，它可以延迟流水线执行指令（de-pipeline），使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。

第二，它可以避免在退出循环的时候因内存顺序冲突（Memory Order Violation）而引起CPU流水线被清空（CPU Pipeline Flush），从而提高CPU的执行效率。

**3.只能保证一个共享变量的原子操作**

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。还有一个方法，就是把多个共享变量合并成一个共享变量来操作。比如，有两个共享变量i=2,j=a合并一下ij=2a，然后用CAS来操作ij。从java1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

## Java中实现

unsafe  java8中无法直接获取实例，只有bootstrap类加载器加载的类，即java核心类才有权限获取单例的实例。java11中可以直接获取单例。

unsafe支持的操作：

1）cas  compareAndSwapObject 、compareAndSwapInt、compareAndSwapLong

2）原子操作 基于cas实现

getAndAddInt、getAndAddLong、getAndSetInt、getAndSetLong、getAndSetObject

3）分配和回收堆外内存

 allocateMemory  freeMemory

4）类似反射，获取Class对象，生成对象实例，设置对象属性

5）线程同步  LockSupport

park 、unpark  

@deprecated：monitorEnter、monitorExit

6) 内存屏障相关 

loadFence、storeFence、fullFence



```java
	// java 8 中的源码
    @CallerSensitive
    public static Unsafe getUnsafe() {
        Class var0 = Reflection.getCallerClass();
       //判断classloader是否为null，为null证明是系统类加载器，才有权限返回
        //如果想获取只能通过反射机制获取
        if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
            throw new SecurityException("Unsafe");
        } else {
            return theUnsafe;
        }
    }
/**
 最开始是由BootStrap ClassLoader加载rt.jar下的文件，也就是java最最核心的部分；
 然后由Extension ClassLoader加载ext下的文件；再有App ClassLoader加载用户自己的文件。
 由于BootStrap ClassLoader是用c++写的，所以在返回该ClassLoader时会返回null
 */

```



```java
// var1 改值的对象
// var2 偏移量
// var4 要改变的值
public final int getAndSetInt(Object var1, long var2, int var4) {
    int var5;
    do {
        //根据便宜量，去除属性值
        var5 = this.getIntVolatile(var1, var2);
        //出去的属性值作为老值，执行cas，不成功说明期间有人修改过，重新取值，在cas，直到成功为止	
    } while(!this.compareAndSwapInt(var1, var2, var5, var4));

    return var5;
}
```





指令重排序     

|            代码级             |                            JMM级                             |                            CPU级                             |
| :---------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|       编译器优化重排序        |                       指令级并行重排序                       |                        内存系统重排序                        |
| volatile、synchronized、final | LoadLoad Barriers、StoreStore Barriers、LoadStore Barriers、StoreLoadBarriers、 | 以在 CPU 层面提供了 **memory barrier**(内存屏障)的指令，从硬件层面来看这个 memroy barrier 就是 CPU flush store bufferes中的指令。X86 memory barrier指令包括lfence(读屏障) sfence(写屏障) mfence(全屏障) |



参考 https://www.toutiao.com/a6735777944611848711/