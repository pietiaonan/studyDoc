什么样的对象会在老年代

1. 新生代回收存活对象（默认15次minorGC ）
2. 新生对象在新生代放不下，直接放到老年代
3. 空间担保
4. 动态年龄回收
5. 从Eden空间往Survivor空间转移的时候Survivor空间不够，**直接放到老年代去**

调优：

https://blog.csdn.net/u011884671/article/details/88679457

