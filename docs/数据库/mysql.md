## 1.锁

相对其他数据库而言，MySQL的锁机制比较简单，其最 显著的特点是不同的存储引擎支持不同的锁机制。比如，**MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking**）；BDB存储引擎采用的是页面锁（page-level locking），但也支持表级锁；**InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。** 
**表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 
**行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。 
**页面锁：**开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般 

### MyISAM表锁

MySQL的表级锁有两种模式：**表共享读锁（Table Read Lock）**和**表独占写锁（Table Write Lock）**。 



>l 读锁，对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求
>l 读锁，对MyISAM表的读操作，不会阻塞当前session对表读，当对表进行修改会报错
>l 读锁，一个session使用LOCK TABLE命令给表f加了读锁，这个session可以查询锁定表中的记录，但更新或访问其他表都会提示错误；

>l 写锁，对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；
>l 写锁，对 MyISAM表的写操作，当前session可以对本表做CRUD,但对其他表进行操作会报错



对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM表的读操作与写操作之间，以及写操作之间是串行的



MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行更新操作 （UPDATE、DELETE、INSERT等）前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此，用户一般不需要直接用LOCK TABLE命令给MyISAM表显式加锁



```mysql
## 加锁之后session只能操作当前锁定的表
## 读锁
lock table t1 read;
unlock tables;
##写锁
lock table t1 write;
unlock tables;
## 多个表同时加锁
Lock tables orders read local, order_detail read local; 
Select sum(total) from orders;
Select sum(subtotal) from order_detail; 
Unlock tables;
## LOCK TABLES时加了“local”选项，其作用就是在满足MyISAM表并发插入条件的情况下，允许其他用户在表尾并发插入记录,插入的数据当前持有读锁用户查询不到，插入数据用户再次查询进入等待锁状态

```

### 查询表级锁争用情况

```mysql
mysql> show status like 'table%';
```

- 1

> **Variable_name | Value** 
> *Table_locks_immediate | 2979* 
> *Table_locks_waited | 0* 
> 2 rows in set (0.00 sec))

如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。

### MyISAM的锁调度

MyISAM存储引擎的读锁和写锁是互斥的，读写操作是串行的。那么，一个进程请求某个 MyISAM表的读锁，同时另一个进程也请求同一表的写锁，MySQL如何处理呢？答案是**写进程先获得锁**。不仅如此，即使读请求先到锁等待队列，写请求后 到，写锁也会插到读锁请求之前！这是因为MySQL认为写请求一般比读请求要重要。这也正是**MyISAM表不太适合于有大量更新操作和查询操作应用的原因**，因为，大量的更新操作会造成查询操作很难获得读锁，从而可能永远阻塞。这种情况有时可能会变得非常糟糕！幸好我们可以通过一些设置来调节MyISAM 的调度行为。

两种方式修改：

- low-priority-updates  ，指定INSERT、UPDATE、DELETE语句的LOW_PRIORITY属性，降低该语句的优先级
- 系统参数`max_write_lock_count`设置一个合适的值，当一个表的读锁达到这个值后，MySQL就暂时将写请求的优先级降低，给读进程一定获得锁的机会。

## 2. InnoDB锁

**InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）；二是采用了行级锁。**

<img src="..\\img\mysql\mysql_1.png" style="zoom:150%;" />

### 获取InonoD行锁争用情况

可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况：

```mysql
mysql> show status like 'innodb_row_lock%';
```
<img src="..\\img\mysql\mysql_2.png" style="zoom:150%;" />

如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高，还可以通过设置InnoDB Monitors来进一步观察发生锁冲突的表、数据行等，并分析锁争用的原因。

查具体数据的所情况
```mysql
select * from information_schema.INNODB_LOCKS;
```
<img src="..\\img\mysql\mysql_4.png" style="zoom:150%;" />




```mysql
select * from sys.innodb_lock_waits
```
<img src="..\\img\mysql\mysql_5.png" style="zoom:150%;" />




### InnoDB的行锁模式及加锁方法

- **共享锁（s）：又称读锁。**允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，**其他事务只能再对A加S锁**，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
- **排他锁（Ｘ）：又称写锁。**允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，**其他事务不能再对A加任何锁**，直到T释放A上的锁。


- 共享锁又称：读锁。当一个事务对某几行上读锁时，允许其他事务对这几行进行读操作，但不允许其进行写操作，也不允许其他事务给这几行上排它锁，但允许上读锁。
- 排它锁又称：写锁。当一个事务对某几个上写锁时，不允许其他事务写，但允许读。更不允许其他事务给这几行上任何锁。包括写锁


- 共享锁（S）：`SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE`。
- 排他锁（X）：`SELECT * FROM table_name WHERE ... FOR UPDATE`。


另外，为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB还有两种内部使用的**意向锁（Intention Locks）**，这两种意向锁都是表锁

- 意向共享锁（IS）：事务打算给数据行共享锁，事务在给一个数据行加共享锁前必须先取得该表的IS锁。
- 意向排他锁（IX）：事务打算给数据行加排他锁，事务在给一个数据行加排他锁前必须先取得该表的IX锁。
- <img src="..\\img\mysql\mysql_3.png" style="zoom:150%;" />

如果一个事务请求的锁模式与当前的锁兼容，InnoDB就请求的锁授予该事务；反之，如果两者两者不兼容，该事务就要等待锁释放。 
意向锁是InnoDB自动加的，不需用户干预。对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排他锁（X)；对于普通SELECT语句，InnoDB不会加任何锁。



### InnoDB行锁实现方式

InnoDB行锁是通过给索引上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！** 
在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能。下面通过一些实际例子来加以说明。

（1）在不通过索引条件查询的时候，InnoDB确实使用的是表锁，而不是行锁。

（2）由于MySQL的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现锁冲突的。应用设计的时候要注意这一点。 

（3）当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，另外，不论是使用主键索引、唯一索引或普通索引，InnoDB都会使用行锁来对数据加锁。 

（4）即便在条件中使用了索引字段，但是否使用索引来检索数据是由MySQL通过判断不同执行计划的代价来决 定的，如果MySQL认为全表扫描效率更高，比如对一些很小的表，它就不会使用索引，这种情况下InnoDB将使用表锁，而不是行锁。因此，在分析锁冲突 时，别忘了检查SQL的执行计划，以确认是否真正使用了索引。 
比如，在tab_with_index表里的name字段有索引，但是name字段是varchar类型的，检索值的数据类型与索引字段不同，虽然MySQL能够进行数据类型转换，但却不会使用索引，从而导致InnoDB使用表锁。通过用explain检查两条SQL的执行计划，我们可以清楚地看到了这一点。 

### 间隙锁（Next-Key锁）

当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的 索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁 （Next-Key锁）。 
举例来说，假如emp表中只有101条记录，其empid的值分别是 1,2,…,100,101，下面的SQL：

```mysql
Select * from  emp where empid > 100 for update;
```

是一个范围条件的检索，InnoDB不仅会对符合条件的empid值为101的记录加锁，也会对empid大于101（这些记录并不存在）的“间隙”加锁。

InnoDB使用间隙锁的目的，一方面是为了防止幻读，以满足相关隔离级别的要求，对于上面的例子，要是不使 用间隙锁，如果其他事务插入了empid大于100的任何记录，那么本事务如果再次执行上述语句，就会发生幻读；另外一方面，是为了满足其恢复和复制的需 要。有关其恢复和复制对锁机制的影响，以及不同隔离级别下InnoDB使用间隙锁的情况，在后续的章节中会做进一步介绍。

很显然，在使用范围条件检索并锁定记录时，InnoDB这种加锁机制会阻塞符合条件范围内键值的并发插入，这往往会造成严重的锁等待**。因此，在实际应用开发中，尤其是并发插入比较多的应用，我们要尽量优化业务逻辑，尽量使用相等条件来访问更新数据，避免使用范围条件。**

### 避免死锁，这里只介绍常见的三种

- 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
- 选择合理的事务大小，小事务发生锁冲突的几率也更小；
- 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
- 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
- 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
- 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。






- https://www.cnblogs.com/leedaily/p/8378779.html
- https://www.cnblogs.com/Soy-technology/p/11064307.html
- https://zhuanlan.zhihu.com/p/29150809
- https://blog.csdn.net/qq_40378034/article/details/90904573