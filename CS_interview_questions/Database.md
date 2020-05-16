# 数据库面试题

> 除以下原理知识外，leet code 有SQL表达式题目，大家可以去做一下。

### 1.索引的底层实现：
**B树：**  
B Tree 是平衡查找树，所有叶子节点到根节点的距离相等，节点间使用指针相连。每个节点内的数据按顺序存储，同时为其子节点的划分阈值。

**B+树：**  
B+ Tree是B Tree的变种，其所有数据存储于叶子节点，中间节点只存储指针。
- 相同大小的磁盘页使用B+树能存储更多的节点元素，降低树的高度，使得IO次数更少。
- B+树每次查找都查找到叶子节点，查询性能更稳定。
- B+树所有叶子节点形成有序链表，便于范围查询。B树查找范围时先通过二分查找找到下限，再不断中序遍历找到上限。而B+树先二分查找找到下限，再顺序遍历叶子节点链表即可找到上限。

**哈希索引：**  
哈希索引是基于hash表实现的，hash索引本身只需要存储每个键值的hash码，因此其结构十分紧凑，查询效率也很高（能以O(1)的复杂度进行查找）。结构【键值，hash码，指针】。
**哈希索引缺陷：**
- hash索引只能等值查找，不能进行范围查找和部分查找。
- 无法用于排序与分组。
- 当大量值的hash码存在冲突时，其效率不一定比B+树高。
- 二次扫描：第一次找到满足hash值得数据，第二次对比键值取出数据。



### 2.聚簇索引、非聚簇索引(辅助索引)：
聚簇索引：

	• 索引和数据存放在一起，索引结构的叶子节点保存了行数据。
	• 索引顺序与物理顺序相同，更适合between and和order by操作；
	• 每张表只能有一个聚集索引；
	• [InnoDB使用聚簇索引]

非聚簇索引：

	• 索引和数据分开，索引结构的叶子节点指向数据。
	• 索引顺序与物理顺序无关；
	• 每新建一个索引就会建立一个非聚集索引，因此大量建立索引需要更多的资源和开销，并影响insert和update性能；
	• [MyISAM使用非聚簇索引]


### 3.关系模型的三类完整性约束：
- 实体完整性（主键）：每个元组都是唯一可标识的；
- 参照完整性（外键）：将有关联的表使用外键联系起来，保证修改一个表中的数据时，对应的另一个表中数据及时更新。
- 用户定义完整性：又叫域完整性或语义完整性，指明关系中属性的取值范围，防止属性的值与应用定义矛盾。


### 4.一致性哈希：
一致性Hash算法将整个哈希值空间组织成一个虚拟的圆环，一致性Hash算法对于节点的增减都只需重定位环空间中的一小部分数据，具有较好的容错性和可扩展性。


### 5.Redis：
**Redis:** 基于内存的高性能key-value数据库。  
优点：

	• 速度快：基于内存；
	• 支持事务：处理都是原子性的；
	• 支持丰富的数据类型：string，list，set等；
	• 丰富的特性：可用于缓存、消息、按key设置过期时间。

为什么快：

	• 基于内存：数据存在内存中，绝大部分的请求都是内存操作，速度很快；
	• 单进程单线程：避免了进程线程的竞争和进程切换带来的开销，也不必考虑锁的问题；
	• IO多路复用：使得单个线程可以处理多个接口的请求。

### 6.ACID：
	• 原子性（atomicity）：事务被视为不可分割的最小单元，事务中的操作要么全部成功提交，要么失败回滚。反向执行回滚日志中的操作可实现回滚；
	• 一致性（consistency）：数据库在事务执行前后都保持一致性状态，在一致性状态下，所有事务对同一数据的访问结果是相同的；
	• 隔离性（isolation）：事务提交前，其所做的修改对其他事务不可见；
	• 持久性（durable）：事务提交后，其所做的修改会永久保存到数据库中，即使数据库发生崩溃也不会失效。


### 7. 并发一致性问题：
	• 丢失修改：两个事务同时修改一个数据，后修改的结果覆盖了前一个修改的结果。
	• 读脏数据：事务1修改了数据后事务2读取数据，但紧接着事务1撤回了修改，事务2读取的数据就是脏数据。
	• 不可重复读：事务1多次读取同一个数据，在事务1执行过程中，事务2修改数据并提交，之后事务1读取的数据和之前的结果不同。
	• 幻影读：事务1读取一个范围数据，事务2在其中插入数据后，事务1再次读取范围数据和之前不同。

并发一致性问题主要是破坏了事务的隔离性，解决方法是保证事务的隔离性。可以通过封锁来实现隔离性，但是封锁需要用户自己控制，相当复杂。数据库管理系统提供了不同的隔离级别，可以更方便地管理。



### 8. 隔离级别：
	• 未提交读：事务未提交之前，其所做的修改对其他事务也可见；
	• 提交读：事务未提交之前，其所做的修改对其他事务不可见；
	• 可重复读：保证同一个事务中对同一个数据的读取结果是一致的；【MySQL的默认隔离级别】
	• 可串行化：强制事务串行执行，事务间就可以相互不影响。需要加锁保证同一时间只有一个事务执行。


### 9. 视图：
视图是虚拟的表，不包含数据，只包含动态检索数据的查询（即SQL查询语句）。
使用场景：
	• 重用SQL语句；
	• 简化SQL操作，不必知道包含的基本查询的细节；
	• 使用表的组成部分而不是整个表；
	• 保护数据，可以给用户授权表的部分访问权限而非全部；
	• 更改数据和表示：返回与底层表示和格式不同的数据；
规则和限制：
	• 命名唯一；
	• 数量没有限制，可以嵌套（从视图创建新视图）；
	• 不能索引，也不能有关联的触发器和默认值；
	• 可以和表一起使用。



### 10. InnoDB与MyISAM比较：
- 事务：InnoDB是事务性的，可以有Commit和RollBack操作；
- 并发：MyISAM只支持表级锁，InnoDB还支持行级锁；
- 外键：InnoDB支持外键；
- 备份：InnoDB支持在线热备份（系统运行时备份）；
- 崩溃恢复：MyISAM更容易崩溃，且修复很慢；
- 其他特性：MyISAM支持表压缩和空间数据索引。
- MyISAM适合只读数据或表较小、可以容忍修复操作的场景。InnoDB适合大数据量的情况，故障可RollBack。


### 11. InnoDB 4大特性：
	1. 插入缓冲：
		a. 要求：非聚簇索引，索引不唯一；
		b. 插入前先检查是否在缓冲区，若是则直接插入，否则先放入insert buffer对象中。再以一定的频率进行insert buffer和非聚簇索引的子节点进行合并，可以将多个处于同一索引页的插入合并到一个操作中，大大提高了非聚簇索引的插入性能，减少了随机IO带来的性能损耗。
	2. 二次写：二次写缓存是位于系统表空间的存储，用来缓存从缓冲池到数据文件中的数据页，当数据库宕机时可以从中找到备份进行恢复；
	3. 自适应哈希：经常访问的索引会被自动生成到哈希索引中去，通过缓冲池的B+树构造而来，建立速度很快。
	4. 预读：（extent，page两种单位）
		a. 线性预读：将下一个extent读入buffer；
		b. 随机预读：将同一extent中的剩余page读入buffer。


### 12. 范式（解决增删改异常）
- 第一范式：属性不可分；
- 第二范式：所有非主属性依赖于主键；
- 第三范式：所有非主属性不传递函数依赖于主键。


### 13. 数据库优化：
结构优化：

	• 分解字段很多的表：有些字段使用频率低；
	• 增加中间表：经常需要联合查询的表建立中间表，将联合查询改为对中间表的查询；
	• 为频繁使用和查询的字段建立索引；
	• 尽可能使用not null，给空字段定义默认值；
SQL语句优化：

	• 避免使用select *，将需要查询的字段列出来；
	• 使用连接（join）代替子查询；
	• 使用limit对查询结果进行限定；
	• 避免在where子句中进行null判断，使用or和（!=， <>），否则数据库会放弃索引进行全表扫描。


### 14. Union 和 Union all 的区别：
	• 重复数据：Union会去掉重复值，Union all不会；
	• 顺序：Union按照字段排序结果，Union all不会；
	• 效率：Union all比Union快很多。


### 15. MySQL索引：
- 唯一索引：索引列的所有值必须唯一，可以为空；
- 主键索引：是一种唯一索引，一张表只能有一个主键索引，且不能为空；
- 普通索引：基本索引类型，没有唯一性限制，可以为空；
- 全文索引：针对文件、文本的检索，MyISAM和高版本InnoDB支持；fulltext索引配合match against操作检车或过滤文本中的关键字，而不是直接与索引中的值相比较。
- 组合索引：一个索引中包含多个列。只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀集合。


### 16. 存储过程：
- 存储过程（Stored Procedure）是一种在数据库中存储复杂程序，以便外部程序调用的一种数据库对象。
- 存储过程是为了完成特定功能的SQL语句集，经编译创建并保存在数据库中，用户可通过指定存储过程的名字- 并给定参数(需要时)来调用执行。
- 存储过程思想上很简单，就是数据库 SQL 语言层面的代码封装与重用。

触发器是一种特殊类型的存储过程，主要是通过事件进行触发而被执行的，而存储过程可以通过存储过程名字而被直接调用。当对某一表进行诸如Update、 Insert、 Delete 这些操作时，SQL Server就会自动执行触发器所定义的SQL 语句，从而确保对数据的处理必须符合由这些SQL 语句所定义的规则。

### 17. 联结JOIN：
- CROSS JOIN:返回笛卡尔积；
- INNER JOIN在CROSS JOIN的基础上筛选不符合条件的数据；
- 自然联结在普通的INNER JOIN的基础上删除了重复列。
