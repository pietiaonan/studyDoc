什么样的对象会在老年代

1. 新生代回收存活对象（默认15次minorGC ）
2. 新生对象在新生代放不下，直接放到老年代
3. 空间担保
4. 动态年龄回收
5. 从Eden空间往Survivor空间转移的时候Survivor空间不够，**直接放到老年代去**

调优：

https://blog.csdn.net/u011884671/article/details/88679457





标记复制算法的优点在于标记阶段和复制阶段可以同时进行

标记-压缩算法缺点就是GC暂停的时间会增 长，因为你需要将所有的对象都拷贝到一个新的地方，还得更新它们的引用地址。相对于标记-清除算法，它的优点也是显而易见的——经过整理之后，新对象的分 配只需要通过**指针碰撞**便能完成



**Java堆对象的内存分配**

在类加载检查通过之后，虚拟机就会为新生对象分配内存，分配方式因取决于Java堆中内存是否规整而分为“指针碰撞”和“空闲列表”。在Java堆中的内存是规整的情况下，内存分配方式采用“指针碰撞”，反之采用“空闲列表


内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，分配内存的方式就是将这个指针往指向空闲空间那边挪动一段与对象大小相等的距离

假设Java堆中的内存不是规整的，已使用的内存和空闲的内存相互交错，这样就无法进行简单地指针碰撞了，这时候虚拟机会维护一张列表，列表中记录哪些内存块是可用的，在分配内存的时候就从列表中找到一块足够大的空间划分给对象实例，同时更新列表上的记录，这种内存分配的方式叫做“空闲列表”



https://www.toutiao.com/a6718908781431882251/

**“-XX:PretenureSizeThreshold”这个参数对Serial+Serial Old垃圾收集器组合有效而对Parallel+Serial Old垃圾收集器组合无效**



### **动态对象年龄判断**

对象的年龄到达了MaxTenuringThreshold可以进入老年代，同时，如果在survivor区中相同年龄所有对象大小的总和大于survivor区的一半，年龄大于等于该年龄的对象就可以直接进入老年代。无需等到MaxTenuringThreshold中要求的年龄



### [服务刚启动就 Old GC，要闹哪样？](https://www.cnblogs.com/javaadu/p/11742570.html)

https://www.cnblogs.com/javaadu/p/11742570.html

在CMS中，full gc 也叫 The foreground collector，对应的 cms gc 叫 The background collecto



### PLAB

https://blog.csdn.net/snowy_sakura/article/details/8455502



### 多态是怎么实现的

https://blog.csdn.net/huangrunqing/article/details/51996424

https://www.cnblogs.com/loveincode/p/7230448.html