1.单机Mysql 瓶颈

数据总量、数据索引、读写混合

2.Memcached+Mysql + 垂直拆分

3.Mysql主从读写分离 （写主读从）

4.分表分库 + 水平拆分 + mysql集群

5.mysql的扩展性瓶颈

6.为什么使用NOSQL (非关系型)

nosql:易扩展、大数据量高性能、多样灵活的数据模型

键-值对存储



**3v+3高**：海量volume  多样variety  实时velocity     高并发  高可扩  高性能



**分布式数据库 CAP原理(类似ACID）**

C：consistency 强一致性

A：availability

P：partition tolerance 分区容错性



**最多只能满足两个**

CA 传统数据库

AP 大多数网站架构选择

CP Redis、MongoDB

P一般必须，在A C间平衡



 **BASE**

BA Basically Avilable 基本可用

S Soft State 软状态

E  Eventually consistent 最终一致 



**分布式 多台机器 不同服务模块**

**集群 多台机器 相同服务模块**

------

**Redis** (Remote Dictionary Server 远程字典服务器)  

**c写的  k/v 分布式 内存存储、支持持久化  提供多种数据类型、master-slave模式的数据备份**

**场景**：

取最新N个数据

模拟HttpSession 设定过期时间功能

发布、订阅消息系统

定时器、计数器

...

redis-server

redis-cli 

redis-benchmark 性能



> **Redis基本知识**

单进程 对epoll函数包装实现

默认16个数据库（redis.config     databases 16） 选择 0-15   （select  切换）

dbsize  (4G)

keys * (keys k? 使用占位符)

flushdb 清空当前库

flushall 清空所有库

统一密码管理 

端口6379

> **Redis数据类型**

**Redis** 键 key  

```
keys*
exists [key]
move [key] db 移动
ttl [key] 查看过期时间
expire [key] 秒   设置过期时间 -1永不过期 -2已过期
del [key] 删除
types [key] 类型
重复key会覆盖
```



**String**  字符串，二进制安全，最多512M

单值单value

```
set/get/del/append/strlen
Incr/decr/incrby/decrby 操作数字加减
getrange 类似substring，范围内取值
setrange 范围内替换
setex [key] 键秒值 (set with expire)  
setnx (set if not exist)
mset/mget/msetnx  设置/获取多个
```



**List** 底层是链表，头尾可加

单值多value，值全部移除后，对应的键消失

```
lpush/rpush/lrange  (eg:lpush list01 1 2 3 4 5,lpush、rpush像栈、队列的顺序)
lpop/rpop 弹出头部（左/右）
lindex 索引位置
llen 长度
lrem key 删除N个value（lrem list01 2 3,删除两个3）
ltrim key 开始index 结束index 截取范围内值，重新复制
rpoplpush 源列表 目标列表 （类似copy）
linsert key before/after 值1 值2 
```



**Hash** 适合存储对象

k v模式不变，但v是一个键值对

```
hset/hget/hmset/hmget/hgetall/hdel
hlen
hexists key key里面的某个值的key是否存在
hkeys/hvals
hincrby/hincrbyfloat
hsetnx
```



**Set** 无序不重复，底层HashTable

单值多value

```
sadd/smenbers/sismember
scard 获取集合中元素个数
srem key value 删除集合中元素
srandmember key 整数   随机出几个数
spop key 随机出栈
smove  key1 key2 在key1里的某个值    将key1里的某个值赋给key2
sdiff/sinter/sunion  差集 交集 并集 
```



**Zset** 有序集合，使用score排序，zset成员唯一，但score可重复

```
zadd/zrange
zrangebyscore key 开始score 结束score [withscores 、Limit]
zrem key 
zcard/zcount key score区间/zrange key values值
zrevrank key values值
zrevrange
zrevrangebyscore key 
```

命令手册：redisdoc.com   



----

ps -ef|grep redis

> redis.conf 配置文件

***备份使用***



---

> 持久化

**RDB**  （redis database）

快照，写入磁盘

单独创建（fork，复制原进程并作为其子进程）一个子进程进行持久化，临时文件的替换，**主进程**不进行IO操作，高性能，数据恢复完整性不敏感可用，缺点在于最后一次持久化后的额数据可能丢失。

1分钟 1万次

5分钟 10次

15分钟1次

dump.rdb

crc64算法进行数据校验

冷拷贝

shutdown、flushall会清空并保存快照

恢复：使用冷拷贝数据

**AOF** （append only file）

以日志形式记录每个写操作（**flushall**也是写操作）

文件可能会很大

aof错误可修复：

redis-check-aof --fix appendonly.aof

同时存在，先看aof ,再看rdb

**rewrite** :重写机制，aof文件过大，超过阈值时，内容压缩 ,可使用bgrewriteaof

**重写原理：**fork新进程进行文件重写（先写临时文件，再rename），遍历新进程内存中数据，每条记录有一条的Set语句。重写AOF文件的操作，并没有读取旧的aof文件，而是将整个内存的数据库用命令方式重写一个aof文件，类似于快照。redis会记录上次重写AOF时的AOF大小，默认当AOF文件是上次Rewrite后大小的一倍且文件大于64M时触发。

**优势：**

每秒同步 appendfsync always同步持久化 每次发生数据变更会被立刻记录到磁盘 性能较差但数据完整性比较好

每修改同步：appendfsync everysec  异步操作，每秒记录  如果一秒内宕机，有数据丢失

不同步：appendfsync  no 从不同步

**劣势：**

相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb

aof运行效率要慢于rdb，每秒的同步策略效率较好，不同步效率和rdb相同



---

> Redis事务  （部分支持）

**是什么**：可一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其他命令插入，不允许加塞。

**为什么**：一个队列中，一次性、顺序性、排他性的执行一系列命令。

**怎么用**：

EXEC

DISCARD

MULTI

UNWATCH

WATCH key [ key...]

1.正常执行：multi开启  [......一堆操作]  exec执行

2.放弃执行：muti  Discar 丢弃

3.全体连坐：muti  [出错] exec报错   加入队列的时候就挂了

4.冤头债主：加入队列没问题，执行时有问题，谁出错找谁，不影响其他正常的

5.watch监控：

- 悲观锁/乐观锁/CAS:   悲观锁，认为肯定有问题（表锁、行级锁、读锁、写锁）；乐观锁，不会出啥问题（加入version字段实现，提交版本必须大于当前版本才能执行更新）；

- 先监控 Watch  key 再 Muti
- unwatch 取消对所有key监控 

三阶段：开启、队列 、执行

三特性：单独的隔离操作、无隔离级别概念、不保证原子性



---

> **Redis的发布订阅**

**是什么：**进程间的一种通信模式；发送者（pub）发送消息，订阅者（sub）接收消息

**命令：**....

**案例：**...



---

> Redis的复制（master/slave）

**是什么：**主从复制，主机更新数据后根据配置和策略，自动同步到备机的master/slaver机制，master以写为主，slave以读为主

**为什么：**读写分离  容灾恢复

**怎么做：**

1. 配从（库）不配主（库）

2. 从库配置：slaveof 主库ip  主库端口  (每次与master断开之后，都需要重新连接，除非配置进redis.conf文件)

3. 修改配置文件的细节操作

   - 拷贝多个redis.conf文件
   - 开启daemonize yes
   - Pid文件名字
   - 指定端口
   - Log文件名字
   - Dump.rdb名字

4. 常用3招：

   - 一主二仆
     - Init (启动)
     - info replication 查看角色
     - SLAVEOF  主库ip  主库端口  (从库ReadOnly ,主库挂了，从库原地待命，主机恢复继续运行；从库挂了，重启需重新连接主库，除非redis.config配置)

   - 薪火相传
     - 上一个slave可以是下一个slave的master，Slave同样可以接受其他slaves的连接和同步请求，那么slave作为链条中的下一个的master可以有效减轻master的写压力
     - 中途变更转向：会清除之前的数据，重新建立连接拷贝最新的
     - slaveof 新主库IP 新主库端口
   - 反客为主
     - Slaveof no one

**复制原理**

**哨兵模式**(反客为主自动版，稍有不同，主库恢复变成从库)

- 哨兵文件：新建touch sentinel.conf

- 配置哨兵：sentinel monitor 被监控数据库名字（主库，自定义）127.0.0.1 6379 1

上面 1 表示主机挂掉后slave投票看让谁接替成为主机，得票数多少后成为主机

- 启动哨兵：Redis-sentinel  /../(哨兵文件路径)
- 

**复制的缺点：**复制延时

