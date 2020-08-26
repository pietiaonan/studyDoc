## 硬件架构模型

<img src="..\\..\\img\\java\\volatile_1.jpg" style="zoom:150%;" />



### 为了解决缓存不一致问题，通常来说有以下**2种解决方法：**

**1）总线锁**：在总线上发出了LOCK锁的信号,独占锁，锁住总线期间，其他CPU无法访问内存，导致效率低下。

**2）缓存锁**：不需锁定总线，只需要“锁定”被缓存的共享对象（实际为：缓存行）即可，接受到lock指令，通过**缓存一致性协议**，维护本处理器内部缓存和其他处理器缓存的一致性。相比总线锁，会提高cpu利用率。

**缓存行**（cache line）是CPU缓存中可分配、操作的最小存储单元。与CPU架构有关，通常有32字节、64字节、128字节不等。目前64位架构下，64字节最为常用。

**缓存一致性协议核心思想：**当CPU向内存写入数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，会发出信号通知其他CPU将该变量的缓存行置为无效状态，因此当其他CPU需要读取这个变量时，发现自己缓存中缓存该变量的缓存是无效的，那么它就会从内存重新读取。这类协议有MSI、**MESI**、MOSI等。

**MESI**是Modified（修改）、Exclusive（独占）、Shared（共享）、Invaild（失效）四种状态的缩写，是用来修饰**缓存行**的状态。在每个缓存行前额外使用2bit，来表示此四种状态。



<img src="..\\..\\img\\java\\MESI.jpg" style="zoom:130%;" />


- 当缓存行处于Modified状态时，会时刻监听其他cpu对该缓存行对应主内存地址的**读取**操作，一旦监听到，将本cpu的缓存行写回内存，并标记为Shared状态
- 当缓存行处于Exclusive状态时，会时刻监听其他cpu对该缓存行对应主内存地址的**读取**操作，一旦监听到，将本cpu的缓存行标记为Shared状态
- 当缓存行处于Shared状态时，会时刻监听其他cpu对使缓存行失效的指令（即其他cpu的写入操作），一旦监听到，将本cpu的缓存行标记为Invalid状态（其他cpu进入Modified状态）
- 当缓存行处于Invalid状态时，从内存中读取，否则直接从缓存读取

**总结：**当某个cpu修改缓存行数据时，其他的cpu通过监听机制获悉共享缓存行的数据被修改，会使其共享缓存行失效。本cpu会将修改后的缓存行写回到主内存中。此时其他的cpu如果需要此缓存行共享数据，则从主内存中重新加载，并放入缓存，以此完成了**缓存一致性**。



<img src="..\\..\\img\\java\storebuffer.jpeg" style="zoom:130%;" />





### 伪共享（false sharing）问题

<img src="..\\..\\img\java\falsesharing.jpg" style="zoom:130%;" />

总结来说，就是多核多线程并发场景下，多核要操作的不同变量处于同一缓存行，某cpu更新缓存行中数据，并将其写回缓存，同时其他处理器会使该缓存行失效，如需使用，还需从内存中重新加载。这对效率产生了较大的影响。



#### Disruptor：Ringbuffer环形缓冲队列，无锁并发框架，号称单线程内每秒可处理600万笔订单



```java
public long p1, p2, p3, p4, p5, p6, p7; // cache line padding

private volatile long cursor = INITIAL_CURSOR_VALUE;

public long p8, p9, p10, p11, p12, p13, p14; // cache line padding
```





#### Jdk8中自带注解@Contended
<img src="..\\..\\img\java\Contended1.jpg" style="zoom:130%;" />



<img src="..\\..\\img\java\Contended2.jpg" style="zoom:130%;" />



## JMM (Java Memory Model)

<img src="..\\..\\img\java\jmm1.jpg" style="zoom:130%;" />

<img src="..\\..\\img\java\jmm2.jpg" style="zoom:130%;" />

<img src="..\\..\\img\java\jmm3.jpg" style="zoom:130%;" />

**JMM与Java内存结构并不是同一个层次的内存划分，两者基本没有关系。如果一定要勉强对应，那从变量、主内存、工作内存的定义看，主内存主要对应Java堆中的对象实例数据部分，工作内存则对应虚拟机栈的部分区域**

### Java内存模型定义了8种操作，虚拟机实现必须保证每一种操作都是原子的、不可再拆分的（double和long类型例外）。

- lock（锁定）：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
- unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
- load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
- assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
- write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

### happens-before原则

## volatile 



并发编程中有3大重要特性

- 原子性

  一个操作或者多个操作，要么全部执行成功，要么全部执行失败。满足原子性的操作，中途不可被中断。

- 可见性

  一个操作或者多个操作，要么全部执行成功，要么全部执行失败。满足原子性的操作，中途不可被中断。

- 有序性

  程序执行的顺序按照代码的先后顺序执行

  

  volatile关键字作用的是保证**可见性**和**有序性**，并不保证原子性。

  

###  可见性

- 当对volatile变量执行写操作后，JMM会把工作内存中的最新变量值强制刷新到主内存
- 写操作会导致其他线程中的缓存无效

###  有序性

volatile变量的禁止指令重排序

volatile是通过编译器在生成字节码时，在指令序列中添加“**内存屏障**”来禁止指令重排序的。


<img src="..\\..\\img\java\order.jpg" style="zoom:130%;" />

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令级并行的重排序。处理器将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
3. 内存系统的重排序。处理器使用缓存和读/写缓冲区，使得加载和存储操作看上去可能是在乱序执行。



硬件层面的“**内存屏障**”：

- **sfence**：即写屏障(Store Barrier)，在写指令之后插入写屏障，能让写入缓存的最新数据写回到主内存，以保证写入的数据立刻对其他线程可见
- **lfence**：即读屏障(Load Barrier)，在读指令前插入读屏障，可以让高速缓存中的数据失效，重新从主内存加载数据，以保证读取的是最新的数据。
- **mfence**：即全能屏障(modify/mix Barrier )，兼具sfence和lfence的功能
- **lock 前缀**：lock不是内存屏障，而是一种锁。执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。

JMM层面的“**内存屏障**”：

- **LoadLoad屏障**： 对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
- **StoreStore屏障**：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
- **LoadStore屏障**：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
- **StoreLoad屏障**： 对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。

JVM的实现会在volatile读写前后均加上内存屏障，在一定程度上保证有序性。如下所示：

> LoadLoadBarrier
>
> volatile 读操作
>
> LoadStoreBarrier


>
> StoreStoreBarrier
>
> volatile 写操作
>
> StoreLoadBarrier



### volatile的的底层实现

这一章会从**Java代码、字节码、JVM源码、汇编层面、硬件层面**去揭开volatile的面纱。

2.1、 Java代码层面

上一段最简单的代码，volatile用来修饰Java变量

public class TestVolatile {

public static volatile int counter = 1;

public static void main(String[] args){

counter = 2;

System.out.println(counter);

}

}

2.2、字节码层面

通过javac TestVolatile.java将类编译为class文件，再通过javap -v TestVolatile.class命令反编译查看字节码文件。

打印内容过长，截图其中的一部分：

<img src="..\\..\\img\\java\\volatile_2.jpg" style="zoom:150%;" />




可以看到，修饰counter字段的public、static、volatile关键字，在字节码层面分别是以下访问标志： **ACC_PUBLIC, ACC_STATIC, ACC_VOLATILE**

volatile在字节码层面，就是使用访问标志：**ACC_VOLATILE**来表示，供后续操作此变量时判断访问标志是否为ACC_VOLATILE，来决定是否遵循volatile的语义处理。

2.3、JVM源码层面

上小节图中main方法编译后的字节码，有putstatic和getstatic指令（如果是非静态变量，则对应putfield和getfield指令）来操作counter字段。那么对于被volatile变量修饰的字段，是如何实现volatile语义的，从下面的源码看起。

1、openjdk8根路径/hotspot/src/share/vm/interpreter路径下的bytecodeInterpreter.cpp文件中，处理putstatic和putfield指令的代码：
```c++
CASE(_putfield):

CASE(_putstatic):

{

// ConstantPoolCacheEntry* cache; -- cache是常量池缓存实例

// cache->is_volatile() -- 判断是否有volatile访问标志修饰

int field_offset = cache->f2_as_index();

if (cache->is_volatile()) { // ****重点判断逻辑****

// volatile变量的赋值逻辑

if (tos_type == itos) {

obj->release_int_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == atos) {// 对象类型赋值

VERIFY_OOP(STACK_OBJECT(-1));

obj->release_obj_field_put(field_offset, STACK_OBJECT(-1));

OrderAccess::release_store(&BYTE_MAP_BASE[(uintptr_t)obj >> CardTableModRefBS::card_shift], 0);

} else if (tos_type == btos) {// byte类型赋值

obj->release_byte_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == ltos) {// long类型赋值

obj->release_long_field_put(field_offset, STACK_LONG(-1));

} else if (tos_type == ctos) {// char类型赋值

obj->release_char_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == stos) {// short类型赋值

obj->release_short_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == ftos) {// float类型赋值

obj->release_float_field_put(field_offset, STACK_FLOAT(-1));

} else {// double类型赋值

obj->release_double_field_put(field_offset, STACK_DOUBLE(-1));

}

// *** 写完值后的storeload屏障 ***

OrderAccess::storeload();

} else {

// 非volatile变量的赋值逻辑

if (tos_type == itos) {

obj->int_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == atos) {

VERIFY_OOP(STACK_OBJECT(-1));

obj->obj_field_put(field_offset, STACK_OBJECT(-1));

OrderAccess::release_store(&BYTE_MAP_BASE[(uintptr_t)obj >> CardTableModRefBS::card_shift], 0);

} else if (tos_type == btos) {

obj->byte_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == ltos) {

obj->long_field_put(field_offset, STACK_LONG(-1));

} else if (tos_type == ctos) {

obj->char_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == stos) {

obj->short_field_put(field_offset, STACK_INT(-1));

} else if (tos_type == ftos) {

obj->float_field_put(field_offset, STACK_FLOAT(-1));

} else {

obj->double_field_put(field_offset, STACK_DOUBLE(-1));

}

}

UPDATE_PC_AND_TOS_AND_CONTINUE(3, count);

}
```
2、重点判断逻辑cache->is_volatile()方法，调用的是openjdk8根路径/hotspot/src/share/vm/utilities路径下的accessFlags.hpp文件中的方法，**用来判断访问标记是否为volatile修饰**。
```c++
// Java access flags

bool is_public () const { return (_flags & JVM_ACC_PUBLIC ) != 0; }

bool is_private () const { return (_flags & JVM_ACC_PRIVATE ) != 0; }

bool is_protected () const { return (_flags & JVM_ACC_PROTECTED ) != 0; }

bool is_static () const { return (_flags & JVM_ACC_STATIC ) != 0; }

bool is_final () const { return (_flags & JVM_ACC_FINAL ) != 0; }

bool is_synchronized() const { return (_flags & JVM_ACC_SYNCHRONIZED) != 0; }

bool is_super () const { return (_flags & JVM_ACC_SUPER ) != 0; }

// 是否volatile修饰

bool is_volatile () const { return (_flags & JVM_ACC_VOLATILE ) != 0; }

bool is_transient () const { return (_flags & JVM_ACC_TRANSIENT ) != 0; }

bool is_native () const { return (_flags & JVM_ACC_NATIVE ) != 0; }

bool is_interface () const { return (_flags & JVM_ACC_INTERFACE ) != 0; }

bool is_abstract () const { return (_flags & JVM_ACC_ABSTRACT ) != 0; }

bool is_strict () const { return (_flags & JVM_ACC_STRICT ) != 0; }

```

3、下面一系列的if...else...对tos_type字段的判断处理，是针对java基本类型和引用类型的赋值处理。如：
```c++
obj->release_byte_field_put(field_offset, STACK_INT(-1));
```
对byte类型的赋值处理，调用的是openjdk8根路径/hotspot/src/share/vm/oops路径下的oop.inline.hpp文件中的方法：
```c++
// load操作调用的方法

inline jbyte oopDesc::byte_field_acquire(int offset) const

{ return OrderAccess::load_acquire(byte_field_addr(offset)); }

// store操作调用的方法

inline void oopDesc::release_byte_field_put(int offset, jbyte contents)

{ OrderAccess::release_store(byte_field_addr(offset), contents); }
```
赋值的操作又被包装了一层，又调用的**OrderAccess::release_store**方法。

4、OrderAccess是定义在openjdk8根路径/hotspot/src/share/vm/runtime路径下的orderAccess.hpp头文件下的方法，具体的实现是根据不同的操作系统和不同的cpu架构，有不同的实现。

**强烈建议大家读一遍orderAccess.hpp文件中30-240行的注释！！！**你就会发现本文1.2章所介绍内容的来源，也是网上各种雷同文章的来源。

<img src="..\\..\\img\\java\\volatile_3.jpg" style="zoom:150%;" />



orderAccess_linux_x86.inline.hpp 是linux系统下x86架构的实现：

<img src="..\\..\\img\\java\\volatile_4.jpg" style="zoom:150%;" />




可以从上面看到，到c++的实现层面，又使用c++中的volatile关键字，用来修饰变量，通常用于建立语言级别的memory barrier。在《C++ Programming Language》一书中对volatile修饰词的解释：

> A volatile specifier is a hint to a compiler that an object may change its value in ways not specified by the language so that aggressive optimizations must be avoided.

含义就是：

- volatile修饰的类型变量表示可以被某些编译器未知的因素更改（如：操作系统，硬件或者其他线程等）
- 使用 volatile 变量时，避免激进的优化。即：系统总是重新从内存读取数据，即使它前面的指令刚从内存中读取被缓存，防止出现未知更改和主内存中不一致

5、步骤3中对变量赋完值后，程序又回到了2.3.1小章中第一段代码中一系列的if...else...对tos_type字段的判断处理之后。有一行关键的代码：**OrderAccess::storeload();** 即：只要volatile变量赋值完成后，都会走这段代码逻辑。

它依然是声明在orderAccess.hpp头文件中，在不同操作系统或cpu架构下有不同的实现。orderAccess_linux_x86.inline.hpp是linux系统下x86架构的实现：

<img src="..\\..\\img\\java\\volatile_5.jpg" style="zoom:150%;" />




代码**lock; addl $0,0(%%rsp)** 其中的addl $0,0(%%rsp) 是把寄存器的值加0，相当于一个空操作（之所以用它，不用空操作专用指令nop，是因为lock前缀不允许配合nop指令使用）

**lock前缀，会保证某个处理器对共享内存（一般是缓存行cacheline，这里记住缓存行概念，后续重点介绍）的独占使用。它将本处理器缓存写入内存，该写入操作会引起其他处理器或内核对应的缓存失效。通过独占内存、使其他处理器缓存失效，达到了“指令重排序无法越过内存屏障”的作用**

2.4、汇编层面

运行2.1章的代码时，加上JVM的参数：-XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly，就可以看到它的汇编输出。（如果运行报错，参见上篇文章：synchronized底层原理（从Java对象头说到即时编译优化）,拉到文章最底部有解决方案）

打印的汇编代码较长，仅截取其中的关键部分：

<img src="..\\..\\img\\java\\volatile_6.jpg" style="zoom:150%;" />




又看到了lock addl $0x0,(%rsp)指令，熟悉的配方熟悉的味道，和上面2.3章中的**步骤5**一摸一样，其实这里就是步骤5中代码的体现。

2.5、硬件层面

为什么会有上述如此复杂问题？为什么会有并发编程？为什么会产生可见性、有序性、原子性的线程或内存问题？

归根结底，还是计算机硬件告诉发展的原因。如果是单核的cpu，肯定不会出现多线程并发的安全问题。正是因为多核CPU架构，以及CPU缓存才导致一系列的并发问题。



### 举例  DCL单例
```java
class Singleton {
    private volatile static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    } 
}
```



参考：
-  https://www.toutiao.com/a6819214673057939975/
- https://www.toutiao.com/a6762409071548039693/
- https://www.toutiao.com/i6817840703884755469/
- https://www.toutiao.com/a6636600759246914052/
- https://www.jianshu.com/p/c6f190018db1

