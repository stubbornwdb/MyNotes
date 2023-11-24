# MySQL高级

### 1.MySQL的架构介绍

1.配置文件：

- 二进制日志log-bin  主从复制
- 错误日志log-error
- 数据文件
  - frm 存放表结构
  - myd存放表数据
  - myi存放表索引
- 如何配置

2.存储引擎

- 查看：
  - show engines;
  - show variables like '%storage_engine%';
- MyISAM
- InnoDB

### 2.索引优化分析

- 性能下降SQL慢、执行时间长、等待时间长

  - 查询语句写的烂

  - 索引失效

    - 单值

      eg:

      select *from user where name = ' ';

      create index idx_user_name on user(name);

    - 复合

      eg:

      select * from user where name=' ' and email ='  ';

      create index idx_user_nameEmail on user(name,email);

  - 查询关联太多join（设计缺陷或不得已的需求）

  - 服务调优及各个参数设置（缓冲、线程数等）

- 常见的Join查询

- 索引简介

  - 索引是一种用于获取数据的数据结构，目的在于提高查找效率（排好序的快速查找数据结构）；
  - 索引分类：一张表最好不超过5个索引
    - 单值索引
    - 唯一索引
    - 复合索引

- 索引结构

  - BTree索引
  - Hash索引
  - full-text全文索引
  - R-Tree索引

  建索引情况：

  1. 主键自动建立唯一索引
  2. 频繁作为查询条件的字段应该创建索引
  3. 查询中与其他表关联的字段，外键关系建立索引
  4. 频繁更新的字段不适合创建索引，因为每次更新以后不单单是更新了记录还会更新索引
  5. where条件里用不到的字段不创建索引
  6. 单键/组合索引的选择：在高并发下倾向于创建组合索引
  7. 查询中排序的字段，排序字段若通过索引上去访问将大大提高排序速度
  8. 查询中统计或者分组的字段

  不要创建索引：

  1. 表记录太少（300万）

   	2. 经常增删改的字段
   	3. 数据重复且分布均匀的表字段，因此应该只为经常查询和最经常排序的数据列建立索引。注意，如果某个数据列包含太多重复的内容，为它建立索引就没有太大的实际效果。

- 性能分析

  - MySql Query Optimizer :负责优化Select 语句的优化器模块

  - 常见瓶颈

    - CPU：CPU在饱和的时候一般发生在数据装入内存或磁盘上读取数据时候
    - IO：磁盘I/O瓶颈发生在装入数据大于内存容量的时候
    - 服务器硬件的性能瓶颈：top free  iostat  vmstat查看系统的性能状态

  - Explain

    - 是什么？ 执行计划，使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈

    - 能干嘛？

      - 表的读取顺序
      - 数据读取操作的操作类型
      - 哪些索引可以使用
      - 哪些索引被实际使用
      - 表之间的引用
      - 每张表有多少行被优化器查询

    - 怎么用？ 

      - EXPLAIN +SQL语句   
      - 执行计划包含的信息：**id** select_type table **type** possible_keys **key** key_len ref **rows**  **Extra**

      1. **id**: select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序，有三种情况：

      id相同，执行顺序由上至下；

      id不同，如果是子查询，id的序列号会递增，id值越大优先级越高，越优先被执行；

      id有相同也有不同，id值大的优先执行，id相同的顺序执行   （DERIVED，衍生虚表）。

      

      2. **select_type**: 

      查询的类型，主要用于区别普通查询、联合查询、子查询等复杂查询。 常见几种：

      SIMPLE:简单的select查询，查询中不包含子查询或UNION；

      PRIMARY:查询中若包含任何复杂的子部分，最外层则被标记为PRIMARY;

      SUBQUERY:在SELECT或WHERE列表中包含了子查询

      DERIVE: 在FROM列表中包含的子查询被被标记为DERIVED（衍生），MYSQL会递归执行这些子查询，把结果放在临时表里。

      UNION： 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED

      UNION RESULT : 从UNION表获取结果

      3. **table**显示这一行数据是关于哪张表的
      4. **type**:

      访问类型，显示查询使用了哪种类型，常见的**从最好到最差：system>const>eq_ref>ref>range>index>ALL**，**一般得保证查询至少达到range级别，最好能达到ref**

      system：表只有一行记录（等于系统表），这是const类型的特例，平时不出现，忽略不计

      const：表示通过索引一次就找到了，const用于比较primary key 或者unique索引，因为只匹配一行数据，所以很快将主键置于where列表中，Mysql就能将该查询转换为一个常量。

      eq_ref:   唯一索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一 索引扫描

      ref: 非唯一性的索引扫描，返回匹配某个单值的所有行，本质上也是一种索引访问，他返回所有匹配某个单独值的行，然而，他可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体；

      range:  只检索给定范围的行，使用一个索引来选择行，key列显示使用了哪个索引，一般就是在你的where语句中出现了beetween、<、>、in等的查询，这种范围扫描比全表扫描要好，因为它只需要开始于索引的某一个点，而结束于索引的另一点，不用扫描全部 索引。

      index: Full Index Scan,index与all区别为index类型只遍历索引树，这通常比All快，因为索引文件通常比数据文件小，也就是说虽然all 和index都是读全表，但index是从索引中读取的，而all是从磁盘中读的

      all :  Full Table Scan 将遍历全表以找到匹配的行；

      5. **possible_keys**

      显示 可能应用在这张表中的索引，一个或多个，查询涉及的字段若存在索引，则该索引将被列出，但不一定被查询实际使用

      6. **key**

      实际使用的索引，如果为NULL，则没有被使用索引；

      查询中出现了覆盖索引，则该索引仅出现在key列表中（理论中没有，实际使用了）。

      7. **key_len** 

      表示索引中使用的字节数，可通过该列计算查询中使用索引的长度，在不损失精度的情况下，长度越短越好；

      key_len显示的值为索引字段的最大可能长度，并非实际使用的长度，即key_len是根据表定义计算而得，不是通过表内检索出的。

      8. **ref**

      显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值；

      9. **rows**

      根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

      10. **Extra**

      包含不适合在其他列中显示但十分重要的额外信息

      **Using filesort:**  ”文件排序“，说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引进行读取，MySQL中无法利用索引完成的排序操作称为“文件排序”，（出现该字段不好，尽快优化）               九死一生

      

      **Using temporary**: 使用了临时表保存中间结果Mysql在对查询结果排序时使用临时表，常见于排序order by 和分组查询 group by (问题严重)                十死无生
      
      
      
      **Using index** ：表示相应的select操作中覆盖了（Covering Index），避免了访问表的数据行，**效率不错**；
      
      如果同时出现using where,表明索引被用来执行索引键值的查找；
      
      如果没有同时出现using where，表明索引用来读取数据而非查找动作。
      
      覆盖索引Covering Index :就是select 的数据只用从索引中就能够取得，不必读取数据行，Mysql可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，换句话说查询列要被所建的索引覆盖。
      
      
      
      Using where
      
      
      
      Using join buffer 使用了连接缓存
      
      
      
      impossible where :where子句的值总是false，不能用来获取任何元组
      
      
      
      select table optimized away: 在没有group by子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT(*) 操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
      
      
      
      distinct： 优化distinct操作，在找到第一匹配的元组后即停止找同样值的操作。
      
      

- 索引优化

  - 索引分析

    范围（> 、< ）以后的索引失效

    - 单表

      show index from table_name;

      Alter table table_name add Index index_name(字段,..,..)

      drop index index_name on table_name;

      新建索引（where查询字段，联合索引，解决全表扫描）+删除索引（索引不合适，绕开范围查询，删除重建）

    - 两表

      添加索引优化

      左连接：加到右表上

      右连接：加到左表上

    - 三表

    尽可能减少join语句中的nestedloop的循环总数，永远用小结果集驱动大的结果集

    优先优化nestedloop的内层循环；

    保证join语句中被驱动表上join条件字段已经被索引

    当无法保证被驱动表的join字段条件被索引且内存资源充足的前提下，不要太吝啬joinbuffer的设置；

  - 索引失效(应该避免)

    - 全值匹配我最爱

    - 最左前缀法则

      带头大哥不能死，中间兄弟不能断（中间没了，只使用到开头那个）

    - 不在索引列上做任何操作，（计算、函数、手动或自动类型转换），会导致索引失效而转向全表扫描

       索引之上不计算

    - 存储引擎不能使用索引中的范围条件右边的列

      范围后边全失效

    - 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少select *

    - 使用！=  或者 <>的时候无法使用索引会导致全表扫描

    - is null, is not null 也无法使用索引

    -  like 以通配符开头（‘%abc...’）mysql索引失效会变成全表扫描的操作（解决方法：1.%仅出现在右边，2.使用覆盖索引）

      like百分加右边

    - 字符串不加单引号失效（对照第3条，这里会隐式转换字符串）

      varchar引号不能缺

    - 少用or，用它来连接时会导致索引失效

  - 一般性建议

    - 对于单键索引，尽量选择针对query过滤性更好的索引
    - 在选择组合索引的时候，当前query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。
    - 在选择组合索引的时候，尽量选择能够包含当前query中where字句中更多字段的索引
    - 尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的

    总结口诀：

    全值匹配我最爱，最左前缀要遵守；

    带头大哥不能死，中间兄弟不能断；

    索引列上少计算，范围之后全失效；

    like百分写最右，  覆盖索引不写星；

    不等空值还有or，索引失效要少用；

    VAR引号不可丢，SQL高级也不难。

### 3.查询截取分析

-------------------分析----------------------

1. 观察，至少跑1天，看看生产的慢sql情况

2. 开启慢查询日志，设置阈值，比如超过5秒的就是慢SQL，并将它抓取出来

3. explain + 慢SQL分析

4. show profile

5. 运维或DBA进行SQL数据库服务器的参数调优

-------------------总结------------------------

1. 慢查询的开启并捕获；

2. explain+慢SQL分析

3. show profile 查询sql 在mysql服务器里面的执行细节和生命周期情况
4. SQL数据库服务的参数调优。

------------------------------------------------------------------------------------------------------------------

- 查询优化

  - 永远小表驱动大表（小的数据集驱动大数据集）类似嵌套循环

    select * from A where id in (select id from B)

    等价于:

    for select id from B

    for  select * from A where A.id = B.id

    **当表B的数据集必须小于A表的数据集是，用in优于exists**

    select * from A where exists (select 1 from B where B.id = A.id)

    等价于：

    for select * from A

    for select *from B where B.id = A.id

    **当A表的数据集小于B表的数据集时，用exists优于in**

    **注意：**A表与B表ID字段应建立索引

  IN：后面的子查询 是返回结果集的,换句话说执行次序和Exists()不一样.子查询先产生结果集,然后主查询再去结果集里去找符合要求的字段列表去.符合要求的输出,反之则不输出.

  Exists：后面的子查询被称做相关子查询, 他是不返回列表的值的.只是返回一个ture或false的结果(这也是为什么子查询里是 **"select  1 "的原因,当然也可以select任何东西**)（select清单会被忽略）其运行方式是先运行主查询一次。再去子查询里查询与其对应的结果，如果是ture则输出,反之则不输出.再根据主查询中的每一行去子查询里去查询。

  - order by排序优化

    - ORDER BY子句，尽量使用Index 方式排序，避免使用FileSort方式排序

      Mysql支持两种方式的排序FileSort和Index ,Index效率高，它指Mysql扫描索引本身完成排序，FileSort方式效率低

      OrderBy满足两种情况，会使用Index方式排序：

      1. OrderBy语句使用索引最左前列
      2. 使用Where子句与OrderBy 子句条件组合满足索引最左前列

    - 尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

    - 如果不在索引列上，filesort有两种算法：mysql就要启动**双路排序**和**单路排序**

      1.双路排序： 从磁盘读取排序字段，在buffer进行排序，再从磁盘取其他字段，取一批数据，要对磁盘进行两次扫描（Mysql4.1前使用该方法）

      2.单路排序：从磁盘读取要查询的所有列，按照order by列在buffer对他们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据 （Mysql4.1后使用该方法），但如果sort_buffer不足，一次取不完数据那性能反而会下降,此时只能服务器调优。

      **Order By时select * 是一个大忌，只Query 需要的字段，这点非常重要。**

  - 优化策略：

    增大sort_buffer_size参数的设置

    增大max_length_for_sort_data参数的设置

  - 总结

    key a_b_c(a,b,c);

    1. order by能使用索引最左前缀

    2. 如果where 使用索引的最左前缀定义为常量，则order by能使用索引

    3. 不能使用索引进行排序：排序升降不一致、丢失大哥、丢失中间、不是索引的一部分、对于排序来说，多个相等条件也是范围查询。

       

- 慢查询日志

  1. 是什么？Mysql提供的一种日志记录，用来记录在Mysql中响应时间超过阈值的语句，具体指运行时间超过long_query_time值的SQL语句，则会被记录到慢查询日志中，long_query_time 默认为10，意思是运行10秒以上的语句。

     由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒，就算SQL，收集超过5秒的SQL，结合explain进行全面分析；

  2. 怎么做? 默认mysql没有开启慢查询日志（会影响性能），慢查询日志支持写入文件

     默认：SHOW VARIABLES LIKE "%slow_query_log%";

     开启：set global slow_query_log=1;  该指令仅对当前数据库生效，数据库重启后会失效

     要永久生效，需修改配置文件：  .cnf

     设置时间：set global long_query_time = 3 （大于3才保存），设置后看不到修改，重新连接或开启新的会话才可看到。

  3. 日志分析工具

     mysqldumpslow

     mysqldumpslow --help 查看使用帮助

  

- 批量数据脚本

  1. 设置参数 log_bin_trust_function_creators 避免慢查询记录

  2. 创建函数，保证每条数据都不同

     随机产生字符串：

     ```
     DELIMITER $$
     
     CREATE FUNCTION rand_string (n INT) RETURN VARCHAR(255)
     
     BEGIN
     ​	DECLARE char_str VARCHAR(100) DEFAULT  'abcdefghijklmnopqrstuvwxyzABCD';
     ​	DECLARE return_str VARCHAR(255)  DEFAULT  ' ';
     ​	DECLARE i INT DEFAULT 0;
     ​	WHILE i<n DO
     ​		SET return_str  = CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
     ​		SET i = i+1;
     ​	END WHILE;
     RETURN return_str;
     END $$
     ```

     随机产生编号

     ```
     DELIMITER $$
     
     CREATE FUNCTION rand_num () RETURN INT(5)
     
     BEGIN
     	DECLARE i INT DEFAULT 0;
     	SET i = FLOOR(100+RNAD()*10);
     RETURN i;
     END $$
     ```

  3. 创建存储过程

     ```
     DELIMITER $$
     
     CREATE PROCEDURE inser_pcd (IN START INT(10),IN max_num INT(10) )
     BEGIN
     	DECLARE i INT DEFAULT 0;
     	#set autocommit = 0  把autocommit设置成0
     	SET autocommit = 0;
     	REPEAT 
     		SET i = i+1;
     		INSERT INTO table_name(...) VALUES(调用创建函数...);
     		UNTIL i=max_num
     	END REPEAT;
     COMMIT;
     END $$
     ```

  4. 调用存储过程

     ```
     DELIMITER;  //改回分割符为分号
     CALL insert_pcd(100,10);
     ```

     

- show proflile

  1. 是什么： 用来分析当前会话中语句执行的资源消耗情况，可以用于SQL调优的测量
  2. 默认情况下参数处于关闭状态，并保存最近15次的运行结果
  3. 分析步骤

- 全局查询日志

  只允许在测试环境中使用

### 4.MySQL锁机制

脏读：事务A读取到了事务B**已修改但未提交**的数据，还在这个基础上做了操作，此时如果事务B如果回滚，A读取的数据就是无效的，不符合一致性要求；

不可重复读：事务A读到了事务B已经提交的修改数据，不符合隔离性

脏读是事务B里面修改了数据，幻读是事务B里面新增了数据

查看隔离级别：show variables like 'tx_isolation';

- 锁是计算机用于协调多个进程或线程并发访问某一资源的机制

- 锁的分类

  1. 从数据的操作的类型（读/写）分：

     读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会相互影响

     写锁（排他锁）：当前写操作没有完成前，他会阻断其他写锁和读锁

  2. 从数据操作的粒度分：

     **行锁**（偏写）：偏向INNODB存储引擎，开销大，加锁慢；会出现死锁；锁粒度最小，发生冲突的概率最低，并发度也最高

     INNODB与MyISAM最大的不同点：INNODB**支持事务Transaction**和**采用了行级锁**

     - 查看：show status like 'innodb_row_lock%';   状态

     - **无索引行锁升级为表锁**（eg:varchar没加引号，没走上索引）

     

     - **间隙锁的危害**：当使用范围条件而不是相等条件检索数据，并请求共享或排他锁时INNODB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做间隙；INNODB也会对这个间隙加锁，这种锁机制就是所谓的间隙锁， next-key.   query执行过程中通过范围查找的话，它会所动这个范围内的索引键值，即使这个键不存在，造成在锁定的时候无法插入锁定键值范围内的任何数据，在某些场景下可能会对性能造成很大危害。

     

     - **如何锁定一行？**begin；   select * from table_name where id="  "  for update;   commit;

     - **如果存在记录则插入，否则更新**？INSERT INTO `student`(`name`, `age`) VALUES('Jack', 19)  ON DUPLICATE KEY   UPDATE `age`=19;

     - 优化建议：

       尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁；

       合理设计索引，尽量缩小锁的范围；

       尽可能减少检索条件，避免间隙锁；

       尽量控制事务大小，减少锁定资源量和时间长度；

       尽可能低级别是无隔离

     

     **表锁**（偏读）：偏向MyISAM存储引擎，开销小、加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发读最低；

     ​	查看：show open tables;

     ​	show status like "table%";     其中Table_lock_immediate 指产生表级锁的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1； Table_locks_waited 表示出现表级锁定争用而发生等待的次数，此值高则表示存在着较严重的标记锁争用情况

     ​	手动增加表锁：lock table table_name read(write) , table_name read(write);

     ​	释放：unlock tables;

     ​	MyISAM的读写调度是写优先，这也是MyISAM不适合做写为主的表引擎的原因，因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。

     ​	**读锁会阻塞写但不阻塞读，写锁会把读和写都堵塞；**

     **页锁**：开销和加锁时间介于表锁和行锁之间，会出现死锁，锁定粒度介于表锁和行锁之间，并发度一般

### 5.主从复制

- 复制的基本原理

  slave会从master读取binlog来进行数据同步

  步骤：

  1. master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志时间事件，binary log events
  2. slave 将master的binary log events 拷贝到它的中继日志（relay log）
  3. slave重做中继日志中的事件，将改变应用到自己的数据库中，Mysql复制是异步的且串行化的

- 复制的基本规则

  1. 每个slave只能有一个master
  2. 每个slave只能有一个唯一的服务器ID
  3. 每个master可以有多个slave

- 复制的最大问题

  延时

- 一主一从的常见配置

  