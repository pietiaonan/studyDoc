## SimpleDateFormat,Calendar 线程非安全的问题

https://www.cnblogs.com/lyy-2016/p/8638553.html

<font color='red'>可以看到format()方法先将日期存放到一个Calendar对象中，而这个Calendar在SimpleDateFormat中是以成员变量的形式存在的。随后调用subFormat()时会再次用到成员变量Calendar。这就是问题所在。同样，在parse()方法里也会存在相应的问题</font>

建议使用：

1.  Java8新增的时间日期包和时间解析包

LocalDate类：代表当前时区的日期

LocalTime类：代表当前时区的时间

LocalDateTime：代表当前时区的日期时间

DateTimeFormatter类
2. Joda-Time 

https://www.ibm.com/developerworks/cn/java/j-jodatime.html



## switch case 比 if  else 性能高

https://www.cnblogs.com/balingybj/p/5751707.html

switch...case与if...else的根本区别在于，switch...case会生成一个跳转表来指示实际的case分支的地址，而这个跳转表的索引号与switch变量的值是相等的。从而，switch...case不用像if...else那样遍历条件分支直到命中条件，而只需访问对应索引号的表项从而到达定位分支的目的。

由此看来，switch有点以空间换时间的意思，而事实上也的确如此。
1.当分支较多时，当时用switch的效率是很高的。因为switch是随机访问的，就是确定了选择值之后直接跳转到那个特定的分支，但是if。。else是遍历所以得可能值，知道找到符合条件的分支。如此看来，switch的效率确实比ifelse要高的多。
2.由上面的汇编代码可知道，switch...case占用较多的代码空间，因为它要生成跳表，特别是当case常量分布范围很大但实际有效值又比较少的情况，switch...case的空间利用率将变得很低。
3.switch...case只能处理case为常量的情况，对非常量的情况是无能为力的。例如 if (a > 1 && a < 100)，是无法使用switch...case来处理的。所以，switch只能是在常量选择分支时比ifelse效率高，但是ifelse能应用于更多的场合，ifelse比较灵活。

## 反射效率低，占用更多内存



https://www.toutiao.com/i6749042737321869838/

**1. Method#invoke 方法会对参数做封装和解封操作**

我们可以看到，invoke 方法的参数是 Object[] 类型，也就是说，如果方法参数是简单类型的话，需要在此转化成 Object 类型，例如 long ,在 javac compile 的时候 用了Long.valueOf() 转型，也就大量了生成了Long 的 Object, 同时 传入的参数是Object[]数值,那还需要额外封装object数组。

而在上面 MethodAccessorGenerator#emitInvoke 方法里我们看到，生成的字节码时，会把参数数组拆解开来，把参数恢复到没有被 Object[] 包装前的样子，同时还要对参数做校验，这里就涉及到了解封操作。

因此，在反射调用的时候，因为封装和解封，产生了额外的不必要的内存浪费，当调用次数达到一定量的时候，还会导致 GC。

**2. 需要检查方法可见性**

通过上面的源码分析，我们会发现，反射时每次调用都必须检查方法的可见性（在 Method.invoke 里）

**3. 需要校验参数**

反射时也必须检查每个实际参数与形式参数的类型匹配性（在NativeMethodAccessorImpl.invoke0 里或者生成的 Java 版 MethodAccessor.invoke 里）；

**4. 反射方法难以内联**

Method#invoke 就像是个独木桥一样，各处的反射调用都要挤过去，在调用点上收集到的类型信息就会很乱，影响内联程序的判断，使得 Method.invoke() 自身难以被内联到调用方。参见 www.iteye.com/blog/rednax…

**5. JIT 无法优化**

在 JavaDoc 中提到：

> Because reflection involves types that are dynamically resolved, certain Java virtual machine optimizations can not be performed. Consequently, reflective operations have slower performance than their non-reflective counterparts, and should be avoided in sections of code which are called frequently in performance-sensitive applications.

因为反射涉及到动态加载的类型，所以无法进行优化。

![](..\img\java\reflection.jpg)

反射原理

1. 反射类及反射方法的获取，都是通过从列表中搜寻查找匹配的方法，所以查找性能会随类的大小方法多少而变化；

  2. 每个类都会有一个与之对应的Class实例，从而每个类都可以获取method反射方法，并作用到其他实例身上；

3. 反射也是考虑了线程安全的，放心使用；

4. 反射使用软引用relectionData缓存class信息，避免每次重新从jvm获取带来的开销；

5. 反射调用多次生成新代理Accessor, 而通过字节码生存的则考虑了卸载功能，所以会使用独立的类加载器；

6. 当找到需要的方法，都会copy一份出来，而不是使用原来的实例，从而保证数据隔离；

7. 调度反射方法，最终是由jvm执行invoke0()执行；



## 创建一个对象的方式：

1、new

2、反射

3、clone

4、序列化/反序列化

5、unsafe new instance

6、字节码技术，jdk动态代理，asm，cglib



## 为什么 Lambda 表达式(匿名类) 不能访问非 final 的局部变量呢

https://www.cnblogs.com/voctrals/p/9990614.html



## cpu怎么区分引用和值



