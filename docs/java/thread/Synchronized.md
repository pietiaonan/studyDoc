Markword



synchronized

linux内核采用的是mutex 互斥量实现，cpu级别待补充

monitorenter

monitoerexit (2个 ，一个正常结束，一个异常是退出)

ReentrantLock

1、直接cas比较，cas在intel的CPU中，使用cmpxchg指令。

2、自旋

3、超过自旋LockSupport.park()完成，而LockSupport.park()则调用sun.misc.Unsafe.park()本地方法，再进一步，HotSpot在Linux中中通过调用pthread_mutex_lock函数把线程交给系统内核进行阻塞。