# 0. 项目准备
1. 明确项目是做什么的
2. 明确项目的价值（为什么做这个项目，他解决了用户的什么痛点，它带来什么价值）
3. 明确项目的功能（这个项目涉及哪些功能）
4. 明确项目的技术（这个项目用到哪些技术）
5. 明确个人在项目中的位置和作用。（你在这个项目中承担的角色）
6. 明确项目的整体架构
7. 明确项目的优缺点，如果重新设计你会如何设计
8. 明确项目的亮点（这个项目有什么亮点）
9. 明确技术成长（你通过这个项目有哪些技术成长）


面试官你好，我叫xx，xx本科，21年毕业，专业是计算机科学与技术。
毕业后任职于xx，担任后台开发工程师一职，主要负责站内分拣打包项目研发，
对需求开发、性能优化、线上问题处理等方面，都有着自己深刻的理解。
在职期间，曾作为分拣、打包模块的owner 完成了项目的重构工作；
现在的话是就职于xxx 公司，在 crm team 负责消息通知平台的开发、维护工作。
谢谢。


# 1.项目介绍
## 1.1 SPX Outbound
spx 是一个完整的快递链路，一个完整的流程大致为：上游通过运单中心下物流单，下完单后会
由快递小哥进行揽收送到first mile hub（第一公里网点），网点内需要进行收货、分拣、打
包（主要会根据运单类型、目的地查询配置的路由进行分拣动作），打包完成后会交接给干线运输，
路由由多个物流网点组成，在各个物流网点都会进行收货、分拣、打包的操作，到达路由中的目的
网点完成收货后，就会交给派送小哥进行last mile 的派送，当然，也有一些包裹是我们无法派
送的，这些包裹我们会外包给第三方物流公司去完成派送，或者由买家自行取货。至此整个流程就
完成了。

而我们的团队叫站内组，主要负责物流网点的收货、分拣、打包、面单打印、交接等工作。我们的系统主要面向的终端
有web 端、PDA以及ASM。其中 web 端、PDA 都是站内操作人员进行手动作业的入口，而ASM 
是自动分拣机、也就是用机器完成收货、分拣、打包等工作的。


### 1.1.1  项目架构
终端：web PDA ASM

应用服务：app task 

下游服务： basic service\ order center \ container \ finance \ delivery

平台监控：cat 普罗米修斯 日志监控

存储层： mysql es redis ck 

大致架构（升级后）：

web \ pda 会通过http 调用的方式请求我们的站内服务， 而ASM 则以GRPC 调用的方式请求。
我们主要分为三大模块，站内作业单、入库、出库。

我们的系统下游系统包括运单中心、基础服务、 计费中心、容器服务、干线运输、派送服务，我们都通过grpc 的方式去调用这些服务。入库、出库
模块作业过程中会通过站内作业单给运单中心进行推送轨迹，（也就是物流轨迹，我们给运单中心的
kafka 队列推送作业信息，运单中心对作业信息进行消费，组装轨迹）。


### 1.1.2 亮点：(性能优化思路 -> cat 链路监控)
1.切流

架构升级后，需要保证代码能平滑替代，需要做流量灰度控制。对于分拣模块的功能，
主要就看最终的分拣目的地是否与原来一致，主要做了两件事情： 对账和切流配置

一是做数据对账，做对账的话一种是实时对账（或者说是，准实时对账），另外一种是离线对账，
因为分拣功能的特性，在我们完成分拣并将分拣轨迹推送至运单中心后，运单相关的数据就会发生变化，
所以我们对账的时机就放在分拣数据落库时，我开了个协程去跑新的分拣流程，通过日志打印的方式对
比旧流程的分拣结果，然后通过告警平台配置监控对账失败的数据。期间发现并排查解决了几个新流程中
错误分拣的逻辑，并且还找出了一些历史的错误逻辑。

在对账了一周时间后，基本没有出现分拣的问题后我们通过apollo 设计了按站点维度按比例进行灰度切流。
效果就是，整个灰度开始到完全切流，都没有出现任何分拣逻辑的问题，平稳地完成了切流。


2.原子编排

我们分拣关联许多的领域，为了完成分拣工作，我们需要调用运单中心获取运单数据、校验站点信息、校验计费信息、
校验是否在派送途中、获取分拣路由、完成打包（落库）、轨迹更新等，整个流程会涉及众多的rpc 调用，我们分拣接口的RT基本在700-800ms 之间，

这还是挺影响分拣效率的。通过 cat 链路分析，我发现了影响RT 的主要有两个问题：
- 一是分拣流程中的很多步骤其实
是可以并行的，这些操作并没有什么依赖关系，并且因为我们分拣针对不同的市场，很多校验逻辑其实是不必要的； 
- 二是我们分拣的逻辑提供给自动分拣机使用的是同一套逻辑，自动分拣机的分拣速度会依赖我们的分拣结果数据，针对分拣的（分拣以及打包）两大步骤，其实是
可以分开操作的。

对于第一个问题，我设计了一个使用DAG 图进行功能编排的框架，主要是将分拣过程中的步骤都抽出来
作为原子能力，又DAG 图对这些原子能力进行编排，编排的方式是通过apollo 进行配置的，这样就可以做到不同的市场
可以有不同的编排配置；

然后针对自动分拣机的，我与自动分拣组的同事共同讨论后，决定采用异步的方案来解决，自动分拣机通过定时任务
提前调用我们的分拣，获取分拣结果，并存到库内，在机器分拣的时候他们再根据分拣数据落入相应格口就行了。

最终达成的效果就是，我们的人工分拣RT 从700-800 ms下降到了100ms以内，部分市场达到了20-30ms


3.single flight

另外，我还做过一些优化，也是通过观察 cat 链路看到的一些问题，我们站内的各个模块都一定会涉及到站点数据查询，都需要去调用基础服务组
的接口来查询站点数据，而且有一个很突出的点就是，我们一般会出现大量的重复调用，虽然这些调用耗时很少，因为大量的调用，其实会占用较多的
网络带宽。我当时的想法就是，有没有必要再我们这边也缓存一下这些数据，因为像站点数据的更新一般是很少的，但是我很快就否定了这个方案，虽然
更新少，但是不代表没有啊，这就会涉及到同步的问题。没有什么收益。后面通过学习，我了解到了 go原因有个叫singleflight 的包。其实就是可以
用来解决这个问题（合并并发请求）。

平时开发中为了提升性能，减轻DB压力，一般会给热点数据设置缓存，如 Redis，用户请求过来后先查询 Redis，有则直接返回，没有就会去查询数据库，然后再写入缓存。
cache miss 后查询DB和将数据再次写入缓存这两个步骤是需要一定时间的，这段时间内的后续请求也会出现 cache miss，然后走同样的逻辑。
这就是缓存击穿：某个热点数据缓存失效后，同一时间的大量请求直接被打到的了DB，会给DB造成极大压力，甚至直接打崩DB。
常见的解决方案是加锁，cache miss 后请求DB之前必须先获取分布式锁，取锁失败说明是有其他请求在查询DB了，当前请求只需要循环等待并查询Redis检测取锁成功的请求把数据回写到Redis没有，如果有的话当前请求就可以直接从缓存中取数据返回了。
虽然加锁能解决问题，但是太重了，而且逻辑比较复杂，又是加锁又是等待的。
相比之下 singleflight 就是一个轻量级的解决方案。


源码：
Group 结构体由一个互斥锁和一个 map 组成，可以看到注释 map 是懒加载的，所以 Group 只要声明就可以使用，不用进行额外的初始化零值就可以直接使用。
call 保存了当前调用对应的信息，map 的键就是我们调用 Do 方法传入的 key
singleflight 内部使用 waitGroup 来让同一个 key 的除了第一个请求的后续所有请求都阻塞。直到第一个请求执行 fn 返回后，其他请求才会返回。
这意味着，如果 fn 执行需要很长时间，那么后面的所有请求都会被一直阻塞。

这时候我们可以使用 DoChan 结合 ctx + select 做超时控制
singleflight 的实现为，如果第一个请求失败了，那么后续所有等待的请求都会返回同一个 error。

调用方：我们接口超时时间不一致咋整？  DoChan

调用方：遇到了问题，我更新了数据后，再查数据，还查到了老数据？  Forget


## 1.2 消息平台
### 1.2.1 项目描述
消息推送平台承接着站内对各种类型渠道的消息下发，每天承载亿级流量推送。项目主要对用户侧的
召回（营销）以及通知消息触达，也同时负责对内网的告警和通知消息发送。

1、消息推送平台它承接着各种消息类型的推送，比如短信、邮件、小程序、微信公众号、通知栏PUSH、
企业微信、钉钉等等。你可以简单理解为：只要发送消息的，就跟它脱不了关系。

2、发送的消息主要给两部分用户，一部分是我们站内的真实用户（比如我们给用户发短信验证码），
另一部分是我们内网的消息（比如钉钉的工作提醒、群消息助手）

### 1.2.2 架构描述
1. 在消息推送平台里，我们有个接入层消息平台-api，它是消息的统一入口，所有的消息推送都会经过该接入层进行处理。
2. 使用消息推送平台的业务方可以简单分为两种角色：运营和技术。如果是技术，他会调用我在接入层暴露的接口。如果是运营，他会使用我的消息推送后台去设置定时任务推送，所以我们会有个推送后台消息平台-admin以及定时任务模块消息平台-cron
3. 接入层干的事情比较简单，简单概括就是消息做简单的校验以及参数拼装后就写入到了消息队列
4. 写到了消息队列之后，自然就有个逻辑层对消息队列的消息进行消费，在我这边叫做消息平台-handler模块，它主要对消息做去重、夜间屏蔽等逻辑，最后就分到不同的消息类型Handler进行消息发送
5. 消息推送平台跟普通消息下发最大的不同是我们是实现对消息全链路追踪的，业务方可以通过推送后台实时查看消息下发的情况，针对消息模板和用户都是OK的（比如这个用户是否接收到消息，如果没接收到，那可能是因为什么被过滤了）
6. 所以消息推送平台会有个实时流的模块，用Flink实现的。我在消息处理的过程中对多个关键的位置进行埋点，在Flink对这些信息做清洗处理，实时的会写进Redis、离线的会落到Hive中

### 1.2.3 亮点
- 全链路追踪：<br>
 在处理层上会有不少的平台过滤规则，这些过滤规则大多都不是针对于消息模板的，而是针对于userId(接收者)的。在这个处理过程中，记录下每个消息模板中的每个用户的执行情况就尤其重要了。

1、定位和排查问题。如果客户反馈用户收不到短信，一般情况下都在这个处理的过程中导致的（可能是被去重，可能是调用接口出问题）

2、对模板执行的整体链路数据分析。一个消息模板一天发送的量级，中途被每个规则过滤的量级，成功下发的量级以及消息最后被点击的量级。除了点击数据，其他的数据都来源处理层

打点记录消息<br>
businessId=模板类型+模板ID+当天日期 eg:2000044620230129 <br>

前2位为模板类型：
10、定时类的模板(后台定时调用) <br>
20、实时类的模板(接口实时调用) <br>
中间6位为模板Id：446为模板Id，不够6位在前面补0  <br>
后8位为下发的日期：20230129 <br>
(固定16位) <br>

```text
同一批（单次）推送的消息，

0.构建messageId维度的链路信息 数据结构list:{key,list} 
key:消息平台:MessageId:{messageId}
listValue:[{timestamp,state,businessId},{timestamp,state,businessId}]  
```


```text
给某个用户发送的消息

1.构建userId维度的链路信息 数据结构list:{key,list}   <br>
key:userId
listValue:[{timestamp,state,businessId},{timestamp,state,businessId}]
```


```text
某个消息模板某天发送的消息，各种状态及其数量

2.构建消息模板维度的链路信息 数据结构hash:{key,hash}
key:businessId
hashValue:{state,stateCount}
```





基于上面的背景，我设计了一套埋点的规则，在处理关键链路上打上对应的点位 <br>
消息接收成功； <br>
消费被丢弃；<br>
夜间屏蔽；<br>
夜间屏蔽（次日早上9点发送）；<br>
消息内容被去重；<br>
白名单过滤；<br>
消息下发成功；<br>
消息下发失败；<br>

目前点位的信息还是不全的，随着系统的完善和接入各个渠道，这里的点位信息还会继续增加，只要我们认为有哪些地方是需要记录下来的，就可以增加。

=====================================

- 数据隔离：<br>
多topic <br>
生产消费模型 <br>

====================================
- 实现消息去重： <br>
去重该功能在消息平台项目里我是把它定位是：平台性功能。要理解这点很重要！不要想着把业务的各种的去重逻辑都在平台上做，这是不合理的。

这里只能是把共性的去重功能给做掉，跟业务强挂钩应由业务方自行实现。所以，我目前在这里实现的是：<br>
● 5分钟内相同用户如果收到相同的内容，则应该被过滤掉。<br>
○ 实现理由：很有可能由于MQ重复消费又或是业务方不谨慎调用，导致相同的消息在短时间内被消息平台消费，进而发送给用户。有了该去重，我们可以在一定程度下减少事故的发生。<br>
● 一天内相同的用户如果已经收到某渠道内容5次，则应该被过滤掉。（频次限流）<br>
○ 实现理由：在运营或者业务推送下，有可能某些用户在一天内会多次收到推送消息。避免对用户带来过多的打扰，从总体定下规则一天内用户只能收到N条消息。<br>

不排除随着业务的发展，还有些需要我们去做的去重功能，但还是要记住，我们这里不跟业务强挂钩。<br>

支持两种去重的类型：N分钟相同内容达到N次去重和一天内N次相同渠道频次去重。<br>
去重的逻辑可以统一抽象为：在X时间段内达到了Y阈值，还记得我曾经说过：「去重」的本质：「业务Key」+「存储」。那么去重实现的步骤可以简单分为（我这边存储就用的Redis）：<br>
● 通过Key从Redis获取记录<br>
● 判断该Key在Redis的记录是否符合条件<br>
● 符合条件的则去重，不符合条件的则重新塞进Redis更新记录<br>

为了方便调整去重的参数，我把X时间段和Y阈值都放到了配置里{"deduplication_10":{"num":1,"time":300},"deduplication_20":{"num":5}}。目前有两种去重的具体实现：<br>

1、5分钟内相同用户如果收到相同的内容，则应该被过滤掉<br>
2、一天内相同的用户如果已经收到某渠道内容5次，则应该被过滤掉<br>

从配置中心拿到配置信息了以后，Builder就是根据这两种类型去构建出DeduplicationParam。<br>
Builder和DeduplicationService都用了类似的写法（在子类初始化的时候指定类型，在父类统一接收，放到Map里管理）<br>

1. 频次去重采用普通的计数去重方法，限制的是每天发送的条数。
2. 内容去重采用的是新开发的基于redis中zset的滑动窗口去重，可以做到严格控制单位时间内的频次。
3. redis使用lua脚本来保证原子性和减少网络io的损耗
4. redis的key增加前缀做到数据隔离（后期可能有动态更换去重方法的需求）

```
// 1.移除开始时间窗口之前的数据
ZREMRANGEBYSCORE 限流key 0, 当前时间戳-限流窗口(毫秒)

// 2.统计当前元素数量 n
ZCARD 限流key   

// 3. 是否超过阈值
if n< 阈值 then zadd 限流key 当前时间戳 score对应的唯一value

为什么ARGV[4]要唯一，具体可以看看zadd这条命令，我们只需要保证每次add进窗口内的成员是唯一的，那么就不会触发有更新的操作（我认为这样设计会更加简单些），而唯一Key用雪花算法比较方便。
为什么expire？，如果这个key只被调用一次。那就很有可能在redis内存常驻了，expire能避免这种情况。
```

● 你的去重功能为什么是在发送消息之前就做了？万一你发送消息失败了怎么办？
对于这个问题，我能扯出的理由有两个：
1、假设我发送消息失败了，在该系统也不会通过回溯MQ的方式去重新发送消息（回溯MQ重新消费影响太大了）。我们完全可以把发送失败的userId给记录下来（全链路追踪现有的功能，后面会提到），
有了userId以后，我们手动批量重新发就好了。这里手动也不需要业务方调用接口，直接通过类似excel的方式导入就好了。

2、在业务上，很多发送消息的场景即便真的丢了几条数据，都不会被发现。有的消息很重要，但有更多的消息并没那么重要，并且我们即便在调用接口才把数据写入Redis，但很多渠道的消息其实在调用接口后，也不知道是否真正发送到用户上了。


● 你的去重功能存在并发的问题吧？
假设我有两条一样的消息，消费的线程有多个，然后该两个线程同时查询Redis，发现都不在Redis内，那这不就有并发的问题吗（对于一天内同一个用户频次去重场景）
如果我们要仅靠Redis来实现去重的功能，去重的实现需要依赖两个操作：查询和插入。查询后如果没有，则需要添加，那查询和插入需要保持原子性才能避免并发的问题（我们的去重功能还要考虑过期时间）

如果不上lua脚本，简单分析下可能用于去重的redis方案：

1、Redis setNx命令 可以原子性设置值，但是该命令没有过期时间的，但我们的去重场景是有时间限制的。

2、Redis incr命令 ，同样没有过期时间，且不能用pipeline批量操作

（这两个命令好像都不太行）

重新把视角拉回到我们为什么要实现去重功能：
当存在事故的时候，我们去重能一定保障到绝大多数的消息不会重复下发。对于整体性的规则，并发消息发送而导致规则被破坏的概率是非常的低。


技术是离不开业务的，有可能我们设计或实现的代码对于强一致性是有疏漏的，但如果系统的整体是更简单和高效，且业务可接受的时候，这不是不可以的。

这是一种trade-off权衡，要保证数据不丢失和不重复一般情况是需要编写更多的代码和损耗系统性能等才能换来的。我可以在消费消息的时候实现at least once语义，保证数据不丢失。我可以在消费消息的时候，实现真正的幂等，下游调用的时候不会重复。

但这些都是有条件的，要实现at least once语义，需要手动ack。要实现幂等，需要用redis lua或者把记录写入MySQL构建唯一key并把该key设置唯一索引。在订单类的场景是必须的，但在一个核心发消息的系统里，可能并没那么重要。




====================================
- 限流：
服务器能处理的请求数是有限的，如果请求量特别大，我们就可能需要做限流。
限流处理的姿势：要么就让请求等待，要么就把请求给扔了
从系统架构来看，我们的统一处理入口在接入层上，
接入层做完简单的参数校验以及参数拼接后，就将请求转发到消息队列上了
按正常来说，因为接了消息队列且接入层没有什么耗时的操作，那对外的接口压力不大的。

没错的，要接入限流也并不是在接入层上做，
是在消息处理下发层。消息处理下发层我们是用线程池去隔离不同的消息渠道不同的消息类型。
在系统本身上其实没有性能相关的问题，但我们下发的渠道可能就需要我们去控制调用的速率。
牛信云短信默认限制3000次/秒调用下发接口

在保证下发速度的前提下，为了让业务方所下发的消息其用户能正常接收到和下游渠道的稳定性，我们需要给某些渠道进行限流
于是在这个背景下，我目前定义了两种限流策略：
1、按照请求数限流（服务提供商QPS 限制）

2、按照下发用户数限流(比如我们内部使用的钉钉群机器人，会限制每个机器人每分钟最多发送20条)





- 动态流量配置：
不同渠道按比例进行分流

### 1.2.3 项目角色
项目主要负责人

### 1.2.4 项目亮点
1、全类型渠道消息的生命周期链路追踪：在每个关键处理的阶段上进行埋点，将点位收集到Kafka，Flink统一清洗处理。实时数据写入Redis，离线数据写入Hive，固化出实时和离线的统一推送基础模型

2、消息资源隔离：不同的渠道不同的消息类型互不影响并且利用动态线程池可配置化地对消费能力进行调控

3、拥有完备的消息管理平台基础建设：对系统和应用资源有完整的监控和告警体系、消息模板工单审核、各种消息模板的素材管理等等

### 1.2.5 接口性能
我们的QPS和RT的指标都来源于日志，我们在接口被调用时打印出了耗时（RT）以及记录了一条日志（QPS）。有了这日志，我们通过GrayLog就能配置出QPS和RT的监控了。所以，我们的接口对QPS和RT都是有监控的哈
在接入层部署了4台4C8G的机器，通过监控可以看到，日常的QPS大概百级（200左右），大促时记录的峰值是2000，RT大概在20ms。、
发送接口是提供了单发和批量接口的。如果有批量推送消息的需求，我们这边是建议业务走批量接口的，这样就能一定程度上减少网络的IO，我这边所承接的QPS并不会太大。
但话又说回来，在接口层面的压力并不大。回到我们的系统架构上，我们有接入层->MQ->发送逻辑层，接入层做的工作仅仅是组装参数，然后把消息发给MQ了。
至于压测的话，我这边没有操作过，一般是提给测试去搞的，这块我就不太清楚了。不过系统成型了以后，发送消息这种确实不太好压，压接入层并没有什么太大的意义（只要MQ能顶得住，那接入层就不是问题）。

发送消息的速率瓶颈一般在下游渠道侧，我们调用短信/邮件等渠道都会限制速度，不同的渠道还不一样。

有大概几个指标：
● 腾讯云短信3000QPS（不过我们会负载到几个短信渠道方）<br>
● 腾讯企业邮箱大概只支持百级以下QPS（当时调高了，就会限制发送失败了，没公布出具体的QPS）<br>
● 个推PUSH我们按发送人数（8000人/QPS）进行限制 <br>
● 而IM是我们自研的（通知类的都需要进MySQL和过风控），限制人数在600人/QPS <br>
● ...

在消费MQ的时候，我们是每个消息渠道的每种类型都会有对应的线程池进行消费，而且这个是动态的线程池（不用重启发布就能调整线程池的参数）
随着业务增长，当QPS真的上来了（连接数变多），我们只要横向扩容就好了，对于接入层来说就是无状态的。

### 1.2.6 项目瓶颈
接入层，接收到了消息下发的请求之后，做些简单的校验和处理就把请求发送至MQ了。理论上我们的接口瓶颈就在于MQ的吞吐量，只要MQ能承受，我们的系统是没什么性能压力的。

而又因为MQ一般支持的TPS都很高，我们的消息平台-api接入层又有提供批量下发接口，所以发消息的瓶颈往往不在于内部系统层面上的性能。

我们都是调用外部的渠道接口进行消息下发的。比如短信有腾讯云/云片这种渠道商，邮件可能会经由腾讯邮件服务，钉钉会经由阿里服务器下发....

无论是哪种渠道，对应的开放平台其接口往往都会限制下发的QPS。那其实下发消息的速率，瓶颈就是对应渠道所支持的并发量。比如，腾讯邮件一秒可能最多我们去调用30QPS（官方没给出具体的数值，但调用频繁它就直接报错了，这是我们线上遇到过的）。正因为如此，我们系统里才会有限流这个功能。

我们系统是做了消息隔离的，每种渠道消费MQ都是互不影响的，我们做的就是让每个渠道的消费能力达到其开放平台所限制的最大值就OK了（这个并不难实现，都是网络IO，况且我们每个渠道消费MQ的时候，设计成动态线程池的）。那如果是系统瓶颈的话，就是机器的网络IO能力了（消息平台是可以横向扩展的）。

### 1.2.7 怎么保证生产者不重复发送消息，消费者不重复消费消息？
在消息推送平台里，我是没有实现生产者不重复发送消息的，因为没啥必要。

在Kafka里，我印象中生产者会有幂等的姿势，同时比较高版本的Kafka也提供了事务的支持，这或许能一定程度上去避免重复发送消息。但无论是哪种实现，肯定会带来性能的损耗。

消费者不重复消费消息这就得在消费端自己做幂等的处理，而在消息推送平台里，我是实现是用Redis做的消息去重功能的。

**幂等是怎么设计的**
我理解下的幂等（去重），由两部分组成：Key+存储

像我处理过的业务（订单类），用的是Redis+MySQL唯一索引的方式去做的。唯一key就是订单Id号+订单状态。Redis主要做前置的判断，而MySQL做最后的判断（反正最后订单的数据还是会落到数据库里）

消息推送平台里，我们是平台类的去重，为了性能，用的是Redis存储，会根据不同的业务对Key进行构建

1、五分钟相同的文案发送相同的人去重，key就是 md5（发送模板ID+接收人+文案内容）。这里md5主要就是为了减小key的长度

2、一天内一个用户只能收到某个渠道的消息 N 次。key就是  发送人+发送渠道。这里的key其实并不唯一，因为要根据次数进行判断。

### 1.2.8 能复用某个消息模板来发送营销消息吗？
可以，但不建议复用某个模板来发送营销消息，消息后台会有提供复制模板的功能，不会麻烦的。

1、营销消息的推送内容应该是要留存的，如果重复使用模板很可能会将原有的模板内容改掉，运维侧/商家就不太好翻阅以前的记录

2、部分的渠道是支持消息撤回的（比如说站内的IM/钉钉），而消息撤回是目前是基于整个模板去做的，所以很有可能会撤回到之前的内容消息。

从功能上是没问题的，但是为了维护性，营销类的消息最好是新建出新的消息模板，或者在原有的模板进行复制（产出新的模板ID）进行消息推送。

### 1.2.9 链路追踪是什么
作为一个平台，我们是需要知道每一条发送的消息的生命周期的，因为前面提到，我们下发一条消息会经过很多个系统（从接口触发，接入层，下发层逻辑层，再到调用下游的发送接口)，每一步都有可能导致消息下发失败。

那么当业务方问起为什么某条消息没有下发到某个用户手上时，我们怎么去给业务方去解释了？翻看Log是一方面，我们基于Log做了全链路追踪，可以在后台就能直接看到某条消息的整个下发情况（现在可以基于用户维度和模板维度去查看

在关键的位置上记录日志，所谓的关键位置就是有可能导致这条消息下发失败的位置，或者下发成功的位置。（比如消费MQ/经过去重/夜间屏蔽/下发成功)这些都是关键位置。

我们打的日志会写到Kafka，由Flink去消费Kafka的日志，明细的日志会进Hive，实时经过ETL的数据会写到Redis

**为什么这个日志不存DB，是基于什么考虑的**
因为量很大，每一条消息下发一次可能会有3~4条消息，某些渠道可能还会更多。明细我们直接落到Hive就行了，在我们公司Hive的数据一般20分钟就能跟上消费速度，而实时可供业务查看的的链路数据我们会进到Redis，由消息推送后台提供对应的功能 进行查询Redis。

**那Redis的内存满了呢？数据是不是被挤掉了？那为什么要存Redis，我存DB行不行**

我们的Redis在公司是有专门的人员维护的，而每个具体的业务场景都是用专门的实例去存储的，也是有监控的。如果内存快满了，我们会有监控告知，可以及时地去做扩容。

如果真的满了，因为有Redis内存淘汰机制，那数据当然是会被挤掉的。假设业务真的发展到一定的程度下，认为内存真的消耗很大，我们可以考虑换HBase这种存储，或者基于磁盘的“Redis”Pika进行存储，这些都是OK的。

DB不太合适，我们存进Redis并不是明细数据，是经过聚合的，而Redis本身就是带有数据结构的存储，比如我们就用到了Hash和List结构，在检索的时候就非常方便了，不用再做调整直接透出给前台。

再说了，DB一般的IUD（增删改）并不会很快，像我们公司一般的配置下也就1300，而且存入DB是永久的，而像HBase和Pika这种都是会有过期的机制，不用我们自己去管理数据的有效期（因为这种链路数据我们也没必要存永久，万一真的要追溯以前的数据，我们还有Hive呢）

### 1.2.10 有没有可能出现这条消息被处理了，但是写Redis失败了？
有可能，如果被处理了，写Redis失败了，那大概率说明Redis集群有问题（不稳定），那此时系统的去重功能会受到一定的影响（但其实甚少）

消息推送平台里的去重逻辑可以理解为就是兜底逻辑，正常的业务强一致去重都是让业务自身去保证的，而不是依赖平台的能力。

那写入redis成功了，消息被处理失败了呢？
也会有这种情况，但是目前消息推送平台下没有程序上的重试的功能，所以现状是不需要处理的。如果回答有重试的功能，那得想办法绕开这次的重试。比如使用（retry=1）标识是重试的逻辑，无需走去重的判断。

### 1.2.11 消息下发失败怎么办（重试），那如果一直失败呢
消息下发失败目前我们是不做策略的，最多会告警让业务方自身通过平台的能力再去重试。

而问题本身，也可以提出重试的技术方案。
1、首先确保消息推送平台本身是没有挂的（只是某个渠道或者某个中间件短时间出了异常导致下发失败了）

2、而重试就可以把消息写到其他的MQ或者调用其他的程序进行下发兜底补充（这块需要关联业务来聊，因为很多消息的下发是有时效性的，并且必须确保是不是真的下发失败，不然下发多了消息比丢了消息这个问题是会更严重的）

### 1.2.12 一直失败上游业务方会收到什么通知？ 你这是做业务告警的系统，告警消息发送失败不是业务有损的吗，这块怎么解决
一直失败上游业务方会收到什么通知？可以有下发失败的通知，告知该模板的消息下发失败了（一般是整个模板的消息失败，而不是某条失败），由于消息内容我们在系统里都可以查看，所以这块直接告知业务即可。

告警消息发送失败不是业务有损的吗，这块怎么解决？方案：可以把某些模板确认为是核心消息，平台可针对这类的消息做重点的关注和处理（回到上面的重试方案上）。

经验值：消息推送平台作为一个下发消息的平台，绝大数下发失败都跟接收者本身有关（push可能是手机的各种权限，短信可能是已停号，邮件可能是业务没提供正确的邮箱地址，IM可能是用户已经关闭了提醒等等）

### 1.2.13 消息积压怎么处理
当出现消息积压的时候，我们的肯定会收到告警（因为一般公司里本身会对消费的进度进行监控），那有了告警了以后我们就可以得出是哪一个渠道的哪种消息在积压中。

得出了哪个渠道的哪种消息在积压我们就可以找正在积压的消息体是什么，通过这一步就能查出是不是在这个时间点上有运营的操作（有可能运营在做活动推送，推了4000W的人群，那这时候的积压是正常的，可以忽略这次的问题）

如果不是运营的活动问题，那这时候就需要去排查是不是调用下发的中间件存在问题，这时候也是去找监控，看各种中间件有无明显的异常。如果有，则及时反馈给中间件的团队去修复。

如果不是中间件的问题，思考自身在近期有没变动改过（大多数的问题都来自于发布变动），如果有，是否可以直接回退原始版本恢复线上的正常运行。

如果不是改动的问题，思考是否需要调整线程池的参数/服务器配置，进而提高消费的能力（系统持续的运行，消费端的正常处理下，很少由于这个问题所带来，一般前期已调试好）

### 1.2.13 消息隔离之后，如果说消费消息的那几个渠道是在不同服务器上，某一个挂了，那么消息队列那个服务器中的消息堆积怎么处理（前提是这个消费服务器第一时间无法正常重启）。
1、首先要根据业务判断这种消息有无时效性，如果时效性很强的消息（比如验证码短信），那可能我们直接就丢弃了。

2、如果时效性是能接受，但我们又想服务恢复后快速消费完所有的消息。我们可以考虑临时扩容，方案大概就是：写一个程序把现有topic的消息写入另一个topic（这个topic比原有的topic扩展出更多的partition，来提高它的并行性）。然后临时写个程序消费这个新建的topic消息（开更多的消费者去干这个事）。当然了，做这个事的前提是我们的下游发送消息的处理能力是跟得上的。

### 1.2.14 消费数据能保证不丢失吗？
现在消息平台接入层是把消息发到mq，下发逻辑层从mq消费数据，随后调用对应渠道接口来下发消息。

消息丢弃一般我们考虑的是消费端，于是重点看的是下发逻辑层。

（因为对于mq使用方来说：生产端只要配置mq相关的参数，在调用下发时有回调重试机制。那就足够了，生产端能做的东西确实不多）

目前为止，下发逻辑层(消费端)使用的是自动提交offset策略。只要消费端存在系统重启或者进程被kill掉，那就会有丢消息的情况。

当前下发逻辑层(消费端)有可能放大了这个丢弃消息的问题，因为现在是消费到mq数据后，会把消息给到线程池去处理。线程池会指定一个阻塞队列，那队列数量越大，可能由重启所丢弃的消息就越多。

这里我的策略是：当应用重启的时候，系统里的线程池是优雅关闭的(尽可能等待一段时间，等阻塞队列里没有消息了，再关闭线程池)。
但回到问题的本质上，只要消费端是自动提交offset策略，就一定会有丢消息的问题。所以要做到消费端的消息不丢，我们就要设置为手动提交offset，这个是必要条件。

### 1.2.15 有没有必要保证不丢
在探讨具体的技术实现方案之前，我们来看看在业务上有没有必要保证消息不丢。我刚接触到消息推送平台的时候，当时那个交接的哥们告诉我和我学长：消息少发比多发要好。

1、重要的消息用户很可能会手动重试触发。

一个发送各类渠道消息的平台，从我的经验来说，这里面最重要的是短信渠道。经过消息通知平台下发很可能是登陆验证码，银行卡提现验证码，这类消息从全局上看是最重要的。

而其他渠道，例如push通知栏的通知消息，微信渠道的营销消息，这种消息即便用户没收到，也不会对用户带来很大的使用体验问题。这种消息或许对绝大数用户都是无感知的（少发几条，用户可能更乐意）。

我们先假设用户的某一次银行卡提现的验证码恰好因为我们重启系统而丢弃。这时候，绝大数用户可能怀疑自己的信号问题，会继续操作，重新发送一次。

(因为客服经常找我排查这种问题，每次都能看到有好几条下发记录。当然了，能到技术的，99%的问题都不是由系统重启丢失消息导致的，更多可能是用户的客户端本身确实就存在问题)

2、消息是有时效性的。

比如验证码这种短信一般就5min的时效性，由于系统的问题，你超过这个时间给用户发送，对用户的体验是非常差的。

3、消息推送平台是有全链路追踪的

我们是可以知道下发的消息有没有到达到用户手上，至少都可以知道在我们的系统内部执行过程中有没有丢。如果这条消息真的那么重要，那可以单独为丢弃的消息单独做重发处理，这些功能在消息推送平台都是支持的。
由于实现难度以及业务的问题，到目前为止消息通知平台都没有实现消费至少一次语义。

### 1.2.16 如果要保证数据不丢需要做什么？
保证数据不丢简单来说，就是我们要在消费端手动ack offset，不能再用自动提交策略了。这样当我们系统重启时，kafka会自动从未ack的offset中拉取。

如果要实现消息推送平台不丢消息的话，有几个问题是需要考虑的：

1、消息少发比多发要好，那么要实现消息不丢，就必须要在系统内实现幂等。因为现在的消息不丢，一般都是基于【至少一次]消费语义去做的。

2、那实现幂等的逻辑是在调用渠道下发接口前，还是渠道下发接口后？

如果做在下发接口前，那是不是会有可能第一次下发记录写入了，但实际调用下发接口却失败了，后面的重试都被幂等处理掉了。

如果做在下发接口后，那是不是会有可能调用调用下发接口成功了，但写入幂等处理的消息失败了，后面的重试就会导致消息多发

3、消息是有时效性的，那如果重试的处理时间过长，那是不是要考虑把这条消息给丢弃掉，不再重试了。

4、重试的消息不应该影响到正常消息的下发，他得作为一种补偿的机制，而非主流程。

稍微细想下技术实现，应该不太好搞，还有很多细节的地方得关注到。比如业务上的：应该是不需要所有的渠道的所有类型消息都得实现消息不丢吧？现在的设计是追求高性能的，能在短时间内下发批量的消息。而如果做到所有消息不丢，肯定会影响到下发的速率。

### 1.2.17 你这数据库表有多大的量级啊？
数据库表并不多哈，主要就是消息模板配置表和短信记录表。

模板配置表的数量级在千级左右，技术侧调用的模板一般都是可以复用的，而运营下发活动营销消息，一般都是要创建新的模板（因为我们会根据模板的维度去看下发的情况，不建议复用模板，会导致分析数据不准）

短信记录表的数据量级，之前我们线上环境大概有1亿4的量，我们没有做分库分表（这一点主要是最开始没考虑进去吧）。但是，其实一亿多的量，在数据库建好索引，我们后台其实只有一天时间+手机号查的功能，是够用的。另外，因为我们hive里会备份历史的数据，所以我们其实可以删除MySQL的历史数据来有更好的查询性能。
当然了，如果公司里有条件，最好是分库分表啦。（就单单短信业务下发记录而言，没有分库分表也是可以的）

### 1.2.18 你这接口能支持多大的QPS啊？RT多少？你部署了多少台机器？
在接入层部署了2台 4C8G 的机器，通过监控可以看到，日常的QPS 七八十左右，大促时记录的峰值是300，RT大概在20ms。

由于接入层接收到请求了之后，是发到MQ的，所以理论上接口是没什么性能压力的，接入层也可以横向扩展。

当我们负责一个系统时，对外需要提供接口给业务方调用，我们是需要了解这个接口的指标以及对应的上下游。这样在出现的问题的时候，就可以根据历史的指标去找问题，去找上下游提醒有什么风险。

面试官问到接口的性能/QPS或者压测主要是想看你是不是真的了解你所负责的内容，如果是搜索/推荐/流量的接口还比较好压测，但发送消息/订单/支付类似这类接口就不太好压测了。

我们能表达出对接口的指标以及相关业务的细节，那么一般面试官也不会纠着你啦（除非这个面试官也刚好做这块业务）。面试官就想看看你有没有思考过自己所负责的业务的一些基础指标。 如果不是TO C的业务，而且用户量没这么大的话，其实接口的QPS都不会高的，减个10倍也是很合理的，日常20 QPS，峰值200 很正常。

### 1.2.19 你这Redis有多大的量级啊？
在消息推送平台里用到Redis的两个核心场景是：

1、内容去重和频次去重

2、全链路追踪的维度数据（用户维度、模板维度）

而问Redis一般是想问你容量多大，比如我们当时的Redis用的是32g做去重，全链路存储用了16G。

面试官有可能会问到：你现在这容量的话，由于业务膨胀，有没有可能会有问题？（其实就是在考你，如果redis满了，会怎么样）

这时候就是理论的问题了，redis本身就会带有内存淘汰机制，如果满了就会把别的key给淘汰掉。

你就说：目前根据现有的业务，根据监控都是够用的。即便满了，其实消息推送平台用redis只是作为去重的兜底方案，不会影响太大。而全链路追踪，我们还会把明细存到hive，对核心功能影响并没那么大。

这时候也可能会问你有没有考虑过别的存储做去重或者全链路追踪的存储。

● 去重用redis是为了下发消息的高性能，其他的存储未必能做到，又因为redis自带过期机制（我们的去重都是有一定的时间限制的），所以很合适。

● 全链路追踪是因为redis有很好使的数据结构，list和hash我们都是flink清洗完直接通过该数据结构写上去，业务层不用再根据单独去聚合。


------
# 2.工作遇到的问题
## 2.1 MySQL 遇到的问题
### 2.1.1 死锁的问题
性能下降：死锁会导致事务相互等待，直到超时或者MySQL选择一个事务作为牺牲者回滚，

这样会导致事务的执行时间变长，从而影响服务的性能。

```sql
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE id=xxxx;
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE task_id=xxxxx;
```


MySQL死锁是指两个或多个事务相互等待对方释放资源而无法继续执行的情况。当发生死锁时，MySQL会自动选择一个事务作为死锁牺牲者进行回滚，以结束死锁。

为了排查MySQL死锁，可以执行以下步骤：

查看MySQL错误日志：在MySQL错误日志中，可以查看到死锁的详细信息，包括死锁的事务ID、死锁发生的表和行号等。可以通过查看错误日志来确定死锁发生的具体原因。

执行SHOW ENGINE INNODB STATUS命令：在MySQL命令行中执行SHOW ENGINE INNODB STATUS命令，可以查看到当前正在运行的事务、锁信息和等待情况等详细信息，通过这些信息可以判断哪些事务出现了死锁。

分析死锁日志：MySQL可以将死锁信息写入到死锁日志中，通过分析死锁日志可以找出死锁的原因和发生时间等信息。可以通过设置参数innodb_print_all_deadlocks=1来开启死锁日志记录。

使用锁监控工具：MySQL提供了一些锁监控工具，如Percona Toolkit等，可以帮助我们更方便地分析死锁问题。使用这些工具可以从多个维度查看死锁信息，包括锁等待时间、锁争用次数等。

优化查询语句和事务设计：死锁问题往往与查询语句和事务设计有关。优化查询语句和事务设计可以减少死锁的发生。例如，可以使用合适的索引、分批次操作数据、尽量避免长事务等。

## 2.3 职业规划
1、1年内，我会先熟悉工作环境，学习公司文化并融入其中，
同时对自己的不足加以改进，学习并提升自己的专业技能，向
领导和老员工虚心请教，争取做好自己的本职工作。

2、1-2年，不断地学习和丰富自己的专业知识，通过自己的
努力成为岗位的技术能手，同时希望通过带新人，锻炼和提升
自己的管理和综合能力。

3、2-3年，通过学习和努力，在专业、综合能力都能符合公司
要求的情况下，逐渐承担一定的管理职责，并继续学习和提升自
己的大局观、沟通和团队领导能力，为公司做出更大的贡献;


## 2.4 微服务治理
微服务治理技术是一种用于管理和协调微服务架构中各个服务的方法。微服务治理技术的目标是确保系统的稳定性、可扩展性、可维护性和高性能。以下是一些常见的微服务治理技术：

1. 服务注册与发现：通过服务注册中心，各个微服务实例可以注册自己的信息，同时查询其他服务的信息。常见的服务注册与发现工具有Eureka、Consul和Zookeeper。

2. API网关：API网关是微服务架构中的入口点，负责请求的路由、负载均衡、认证授权和限流等功能。常见的API网关有Zuul、Kong和Spring Cloud Gateway。

3. 负载均衡：在微服务架构中，负载均衡可以分为客户端负载均衡和服务端负载均衡。客户端负载均衡通常由API网关或服务调用者实现，如Ribbon。服务端负载均衡则由服务提供者实现，如Nginx。

4. 服务熔断与降级：服务熔断是一种防止服务雪崩的机制，当某个服务出现故障时，熔断器会拦截请求，防止故障扩散。服务降级是在服务出现故障或者响应过慢时，返回一个预设的默认响应。常见的熔断与降级工具有Hystrix和Resilience4j。

5. 分布式追踪：分布式追踪用于收集和分析微服务之间的调用关系和性能数据，帮助开发者定位问题和优化性能。常见的分布式追踪工具有Zipkin、Jaeger和OpenTracing。

6. 配置中心：配置中心用于集中管理微服务的配置信息，实现配置的动态更新和版本控制。常见的配置中心有Spring Cloud Config、Apollo和Consul。

7. 容器化与编排：容器化技术（如Docker）可以简化微服务的部署和管理，而编排工具（如Kubernetes）则负责自动化部署、扩缩容、滚动更新等任务。

8. 日志管理：在微服务架构中，统一收集、存储和分析日志信息至关重要。常见的日志管理工具有ELK（Elasticsearch、Logstash、Kibana）和EFK（Elasticsearch、Fluentd、Kibana）。

9. 监控与告警：监控微服务的运行状态和性能指标，及时发现和处理问题。常见的监控与告警工具有Prometheus、Grafana和Alertmanager。

这些技术可以根据项目的需求和团队的技术栈进行选择和组合，以实现高效的微服务治理。

## 2.5. 你一般如何定位线上问题
1、如果线上出现了问题，我们更多的是希望由监控告警发现我们出了线上问题，而不是等到业务侧反馈。所以，我们需要对核心接口做好监控告警的功能。

2、如果是业务代码层面的监控报警，那我们应该是可以很快地定位出是哪儿的问题，毕竟告警逻辑都是我们写的嘛。如果是服务器资源/所依赖的中间件告警，那我们可能就要花点时间去排查啦。

3、不管怎么样，无论是系统告警还是是业务侧反馈系统或者接口出了问题。我们要想想在近期有没有发布过系统，如果近期发布过系统，判断能不能立马回滚到上一个版本，恢复系统平稳正常运行（在线上环境下，可用性是相当重要的）。回滚的时候要考虑接口有无依赖性，是否需要跟业务侧同步此次的回滚以及做相关的配合。

4、因为线上大多数的问题都来源于系统的变更，可能我们只是变更了很少的代码，但只要有一丝的逻辑没留意到，就真的很可能会导致出现问题，回滚很可能是最快能恢复线上正常运行的办法。

5、如果近期都没发布过系统，是系统告的警，那追踪下告警和报错日志，应该是可以很快地就能定位出问题。

6、如果不是系统告的警，是业务侧反馈出了问题，那这时候需要业务侧明确是哪个具体的功能/接口出了问题，有没有保留请求入参，有没有返回错误的信息，有何现象

7、知道了问题的现象之后，就需要根据经验排查可能是哪块出了问题了。我的经验一般是：先查存储侧有没有瓶颈(MySQL 的CPU有没有飙高，主从同步延迟是否很大，有没有慢SQL。Redis是不是内存满了，走了淘汰策略。搜索引擎有没有慢Query)，把该服务所依赖的中间件的指标看一遍，这个过程中也要去看看服务接口的QPS/RT相关的监控。如果有某项指标不对劲，那顺着写入逻辑也应该很快能看出来

8、一般到这里，大多数的问题都能查出来。可能是逻辑本身的问题，可能是请求入参导致慢查询，可能是中间件的网络抖动，可能是突发或者异常请求的问题。

9、如果都不是，回归到应用和机器本身的监控：应用GC的表现、机器本身的网络/磁盘/内存/CPU 各种的指标有没有发现异常的情况。这里可能是需要运维侧一起配合看看有没有做过改动。

10、要是还定位不出来，看能不能复现，能复现都好说，肯定是能解决的。

11、要是不能复现，只能在怀疑的地方打上详细的日志再好好观察（问题定位不出来，很多时候就是日志不够详细，而日志在正常情况下也不应该打太多）

## 2.6.什么是幂等和去重
面试官：要不你来讲讲你最近在看的点呗？可以拉出来一起讨论下

候选者：最近在看「去重」和「幂等」相关的内容

面试官：那你就先来聊聊你对「去重」和「幂等」的理解吧

候选者：我认为「幂等」和「去重」它们很像，我也说不出他们之间的严格区别

候选者：我说下我个人的理解，我也不知道对不对

候选者：「去重」是对请求或者消息在「一定时间内」进行去重「N次」

候选者：「幂等」则是保证请求或消息在「任意时间内」进行处理，都需要保证它的结果是一致的

候选者：不论是「去重」还是「幂等」，都需要对有一个「唯一 Key」，并且有地方对唯一Key进行「存储」

候选者：以项目举例，我维护的「消息管理平台」是有「去重」的功能的：「5分钟相同内容消息去重」「1小时内模板去重」「一天内渠道达到N次阈值去重」…

候选者：再次强调下「幂等」和「去重」的本质：「唯一Key」+「存储」

面试官：那你是怎么做的呢

候选者：不同的业务场景，唯一Key是不一样的，由业务决定

候选者：存储选择挺多的，比如「本地缓存」/「Redis」/「MySQL」/「HBase」等等，具体选取什么，也跟业务有关

候选者：比如说，在「消息管理平台」这个场景下，我存储选择的「Redis」（读写性能优越），Redis也有「过期时间」方便解决「一定时间内」的问题

候选者：而唯一Key，自然就是根据不同的业务构建不同的。

候选者：比如说「5分钟相同内容消息去重」，我直接MD5请求参数作为唯一Key。「1小时模板去重」则是「模板ID+userId」作为唯一Key，「一天内渠道去重」则是「渠道ID+userId」作为唯一Key…

面试官：既然提到了「去重」了，你听过布隆过滤器吗？

候选者：自然是知道的啦

面试官：来讲讲布隆过滤器吧，你为什么不用呢？

候选者：布隆过滤器的底层数据结构可以理解为bitmap，bitmap也可以简单理解为是一个数组，元素只存储0和1，所以它占用的空间相对较小

候选者：当一个元素要存入bitmap时，其实是要去看存储到bitmap的哪个位置，这时一般用的就是哈希算法，存进去的位置标记为1

候选者：标记为1的位置表示存在，标记为0的位置标示不存在

候选者：布隆过滤器是可以以较低的空间占用来判断元素是否存在进而用于去重，但是它也有对应的缺点

候选者：只要使用哈希算法离不开「哈希冲突」，导致有存在「误判」的情况

候选者：在布隆过滤器中，如果元素判定为存在，那该元素「未必」真实存在。如果元素判定为不存在，那就肯定是不存在

候选者：这应该不用我多解释了吧？（结合「哈希算法」和「标记为1的位置表示存在，标记为0的位置标示不存在」这两者就能得出上面结论）

候选者：布隆过滤器也不能「删除」元素（也是哈希算法的局限性，在布隆过滤器中是不能准确定位一个元素的）

候选者：如果要用的话，布隆过滤器的实现可以直接上Guava已经实现好的，不过这个是单机的

候选者：而分布式下的布隆过滤器，一般现在会用Redis，但也不是没个公司都会部署布隆过滤器的Redis版（还是有局限，像我以前公司就没有）

候选者：所以，目前我负责的项目都是没有用布隆过滤器的（：

候选者：如果「去重」开销比较大，可以考虑建立「多层过滤」的逻辑

候选者：比如，先看看『本地缓存』能不能过滤一部分，剩下「强校验」交由『远程存储』（常见的Redis或者DB）进行二次过滤

面试官：嗯，那我就想起你上一次回答Kafka的时候了

面试官：当时你说在处理订单时实现了at least one + 幂等

面试官：幂等处理时：前置过滤使用的是Redis，强一致校验时使用的是DB唯一索引，也是为了提高性能，对吧？

面试官：唯一Key 好像就是 「订单编号 + 订单状态」

候选者：面试官你记性真的好！

候选者：一般我们需要对数据强一致性校验，就直接上MySQL（DB），毕竟有事务的支持

候选者：「本地缓存」如果业务适合，那可以作为一个「前置」判断

候选者：Redis高性能读写，前置判断和后置均可（：

候选者：而HBase则一般用于庞大数据量的场景下（Redis内存太贵，DB不够灵活也不适合单表存大量数据）

候选者：至于幂等，一般的存储还是「Redis」和「数据库」

候选者：最最最最常见的就是数据库「唯一索引」来实现幂等（我所负责的好几个项目都是用这个）

候选者：构建「唯一Key」是业务相关的事了（：一般是用自己的业务ID进行拼接，生成一个”有意义”的唯一Key

候选者：当然，也有用「Redis」和「MySQL」实现分布式锁来实现幂等的（：

候选者：但Redis分布式锁是不能完全保证安全的，而MySQL实现分布式锁（乐观锁和悲观锁还是看业务吧，我是没用到过的）

候选者：网上有很多实现「幂等」的方案，本质上都是围绕着「存储」和「唯一Key」做了些变种，然后取了个名字…

候选者：总的来说，换汤不换药（：

## 2.7. DDD
### 2.7.1 实操
大概以下几点：
1. 通过公共平台大概梳理出系统之间的调用关系（一般中等以上公司都具备 RPC 和 HTTP 调用关系，无脑的挨个系统查询即可），画出来的可能会很乱，也可能会比较清晰，但这就是现状。 
2. 分配组员每个人认领几个项目，来梳理项目维度关系，这些关系包括：对外接口、交互、用例、MQ 等的详细说明。个别核心系统可以画出内部实体或者聚合根。 
3. 小组开会，挨个 review 每个系统的业务概念，达到组内统一语言。 
4. 根据以上资料，即可看出哪些不合理的调用关系（比如循环调用、不规范的调用等），甚至不合理的分层。 
5. 根据主线业务自顶向下细分领域，以及限界上下文。此过程可能会颠覆之前的系统划分。 
6. 根据业务复杂性，指定领域模型，选择贫血或者充血模型。团队内部最好实行统一习惯，以免出现交接成本过大。 
7. 分工进行开发，并设置 deadline，注意，不要单一的设置一个 deadline，要设置中间 check 时间，比如 dealline 是 1 月 20 日，还要设置两个 check 时间，分别沟通代码风格及边界职责，以免 deadline 时延期。

### 2.7.2 DDD介绍
DDD 全程是 Domain-Driven Design，中文叫领域驱动设计，是一套应对复杂软件系统分析和设计的面向对象建模方法论。

同时期，随着互联网的兴起，Rod Johnson 这大哥以轻量级极简风格的 Spring Cloud 抢占了所有风头，虽然 Spring 推崇的失血模式并非 OOP 的皇家血统，但是谁用关心这些呢？毕竟简化开发的成本才是硬道理。

就在我们用这张口闭口 Spring 的时候，我们意识到了一个严重的问题，我们应对复杂业务场景的时候，Spring 似乎并不能给出更合理的解决方案，于是分而治之的思想下应生了微服务，一改以往单体应用为多个子应用，一下子让人眼前一亮，于是我们没日没夜地拆分服务，加之微服务提供的注册中心、熔断、限流等解决方案，我们用得不亦乐乎。

人们在踩过诸多拆分服务的坑（拆分过细导致服务爆炸、拆分不合理导致频分重构等）之后，开始死锁原因了，到底有没有一种方法论可以指导人们更加合理地拆分服务呢？众里寻他千百度，DDD 却在灯火阑珊处，有了 DDD 的指导，加之微服务的事件，才是完美的架构。
背景中我们说到，有 DDD 的指导，加之微服务的事件，才是完美的架构，这里就详细说下它们的关系。

系统的复杂度越来越来高是必然趋势，原因可能来自自身业务的演进，也有可能是技术的创新，然而一个人和团队对复杂性的认知是有极限的，就像一个服务器的性能极限一样，解决的办法只有分而治之，将大问题拆解为小问题，最终突破这种极限。微服务在这方面都给出来了理论指导和最佳实践，诸如注册中心、熔断、限流等解决方案，但微服务并没有对“应对复杂业务场景”这个问题给出合理的解决方案，这是因为微服务的侧重点是治理，而不是分。

我们都知道，架构一个系统的时候，应该从以下几方面考虑：

功能维度
质量维度（包括性能和可用性）
工程维度
微服务在第二个做得很好，但第一个维度和第三个维度做的不够。这就给 DDD 了一个“可乘之机”，DDD 给出了微服务在功能划分上没有给出的很好指导这个缺陷。所以说它们在面对复杂问题和构建系统时是一种互补的关系。

从架构角度看，微服务中的服务所关注的范围，正是 DDD 所推崇的六边形架构中的领域层，和整洁架构中的 entity 和 use cases 层。

知道了 DDD 与微服务还不够，我们还需要知道他们是怎么协作的。

一个系统（或者一个公司）的业务范围和在这个范围里进行的活动，被称之为领域，领域是现实生活中面对的问题域，和软件系统无关，领域可以划分为子域，比如电商领域可以划分为商品子域、订单子域、发票子域、库存子域 等，在不同子域里，不同概念会有不同的含义，所以我们在建模的时候必须要有一个明确的边界，这个边界在 DDD 中被称之为限界上下文，它是系统架构内部的一个边界

所以复杂系统划分的第一要素就是划分系统内部架构边界，也就是划分上下文，以及明确之间的关系，这对应之前说的第一维度（功能维度），这就是 DDD 的用武之处。其次，我们才考虑基于非功能的维度如何划分，这才是微服务发挥优势的地方。

我们可以在一个进程内部署单体应用，也可以通过远程调用来完成功能调用，这就是目前的微服务方式，更多的时候我们是两种方式的混合，比如 A 和 B 在一个部署单元内，C 单独部署，这是因为 C 非常重要，或并发量比较大，或需求变更比较频繁，这时候 C 独立部署有几个好处：

C 独立部署资源：资源更合理的倾斜，独立扩容缩容。
弹力服务：重试、熔断、降级等，已达到故障隔离。
技术栈独立：C 可以使用其他语言编写，更合适个性化团队技术栈。
团队独立：可以由不同团队负责。
架构是可以演进的，所以拆分需要考虑架构的阶段，早期更注重业务逻辑边界，后期需要考虑更多方面，比如数据量、复杂性等，但即使有这个方针，也常会见仁见智，没有人能一下子将边界定义正确，其实这里根本就没有明确的对错。

即使边界定义的不太合适，通过聚合根可以保障我们能够演进出更合适的上下文，在上下文内部通过实体和值对象来对领域概念进行建模，一组实体和值对象归属于一个聚合根。

按照 DDD 的约束要求：

第一，聚合根来保证内部实体规则的正确性和数据一致性；
第二，外部对象只能通过 id 来引用聚合根，不能引用聚合根内部的实体；
第三，聚合根之间不能共享一个数据库事务，他们之间的数据一致性需要通过最终一致性来保证。
有了聚合根，再基于这些约束，未来可以根据需要，把聚合根升级为上下文，甚至拆分成微服务，都是比较容易的。

### 2.7.3 概念
**领域**
映射概念：切分的服务。

领域就是范围。范围的重点是边界。领域的核心思想是将问题逐级细分来减低业务和系统的复杂度，这也是 DDD 讨论的核心。

**子域**
映射概念：子服务。

领域可以进一步划分成子领域，即子域。这是处理高度复杂领域的设计思想，它试图分离技术实现的复杂性。这个拆分的里面在很多架构里都有，比如 C4。

**核心域**
映射概念：核心服务。

在领域划分过程中，会不断划分子域，子域按重要程度会被划分成三类：核心域、通用域、支撑域。

决定产品核心竞争力的子域就是核心域，没有太多个性化诉求。

桃树的例子，有根、茎、叶、花、果、种子等六个子域，不同人理解的核心域不同，比如在果园里，核心域就是果是核心域，在公园里，核心域则是花。有时为了核心域的营养供应，还会剪掉通用域和支撑域（茎、叶等）。

**通用域**
映射概念：中间件服务或第三方服务。

被多个子域使用的通用功能就是通用域，没有太多企业特征，比如权限认证。

**支撑域**
映射概念：企业公共服务。

对于功能来讲是必须存在的，但它不对产品核心竞争力产生影响，也不包含通用功能，有企业特征，不具有通用性，比如数据代码类的数字字典系统。

**统一语言**
映射概念：统一概念。

定义上下文的含义。它的价值是可以解决交流障碍，不管你是 RD、PM、QA 等什么角色，让每个团队使用统一的语言（概念）来交流，甚至可读性更好的代码。

通用语言包含属于和用例场景，并且能直接反应在代码中。

可以在事件风暴（开会）中来统一语言，甚至是中英文的映射、业务与代码模型的映射等。可以使用一个表格来记录。

**限界上下文**
映射概念：服务职责划分的边界。

定义上下文的边界。领域模型存在边界之内。对于同一个概念，不同上下文会有不同的理解，比如商品，在销售阶段叫商品，在运输阶段就叫货品。


理论上，限界上下文的边界就是微服务的边界，因此，理解限界上下文在设计中非常重要。

**聚合**
映射概念：包。

聚合概念类似于你理解的包的概念，每个包里包含一类实体或者行为，它有助于分散系统复杂性，也是一种高层次的抽象，可以简化对领域模型的理解。

拆分的实体不能都放在一个服务里，这就涉及到了拆分，那么有拆分就有聚合。聚合是为了保证领域内对象之间的一致性问题。

在定义聚合的时候，应该遵守不变形约束法则：

聚合边界内必须具有哪些信息，如果没有这些信息就不能称为一个有效的聚合；
聚合内的某些对象的状态必须满足某个业务规则：
一个聚合只有一个聚合根，聚合根是可以独立存在的，聚合中其他实体或值对象依赖与聚合根。
只有聚合根才能被外部访问到，聚合根维护聚合的内部一致性。

**聚合根**
映射概念：包。

一个上下文内可能包含多个聚合，每个聚合都有一个根实体，叫做聚合根，一个聚合只有一个聚合根。

**实体**
映射概念：Domain 或 entity。

《领域驱动设计模式、原理与实践》一书中讲到，实体是具有身份和连贯性的领域概念，可以看出，实体其实也是一种特殊的领域，这里我们需要注意两点：唯一标示（身份）、连续性。两者缺一不可。

你可以想象，文章可以是实体，作者也可以是，因为它们有 id 作为唯一标示。

**值对象**
映射概念：Domain 或 entity。

为了更好地展示领域模型之间的关系，制定的一个对象，本质上也是一种实体，但相对实体而言，它没有状态和身份标识，它存在的目的就是为了表示一个值，通常使用值对象来传达数量的形式来表示。

比如 money，让它具有 id 显然是不合理的，你也不可能通过 id 查询一个 money。

定义值对象要依照具体场景的区分来看，你甚至可以把 Article 中的 Author 当成一个值对象，但一定要清楚，Author 独立存在的时候是实体，或者要拿 Author 做复杂的业务逻辑，那么 Author 也会升级为聚合根。

### 2.7.4 领域模型
**失血模型**
Domain Object 只有属性的 getter/setter 方法的纯数据类，所有的业务逻辑完全由 business object 来完成。

**贫血模型**
简单来说，就是 Domain Object 包含了不依赖于持久化的领域逻辑，而那些依赖持久化的领域逻辑被分离到 Service 层。
注意这个模式不在 Domain 层里依赖 DAO。持久化的工作还需要在 DAO 或者 Service 中进行。
这样做的优缺点
优点：各层单向依赖，结构清晰。
缺点：
Domain Object 的部分比较紧密依赖的持久化 Domain Logic 被分离到 Service 层，显得不够 OO
Service 层过于厚重

**充血模型**
充血模型和第二种模型差不多，区别在于业务逻辑划分，将绝大多数业务逻辑放到 Domain 中，Service 是很薄的一层，封装少量业务逻辑，并且不和 DAO 打交道：
Service (事务封装) —> Domain Object <—> DAO

所有业务逻辑都在 Domain 中，事务管理也在 Item 中实现。这样做的优缺点如下。
优点：

更加符合 OO 的原则；
Service 层很薄，只充当 Facade 的角色，不和 DAO 打交道。
缺点：
DAO 和 Domain Object 形成了双向依赖，复杂的双向依赖会导致很多潜在的问题。
如何划分 Service 层逻辑和 Domain 层逻辑是非常含混的，在实际项目中，由于设计和开发人员的水平差异，可能 导致整个结构的混乱无序。

**胀血模型**
基于充血模型的第三个缺点，有同学提出，干脆取消 Service 层，只剩下 Domain Object 和 DAO 两层，在 Domain Object 的 Domain Logic 上面封装事务。
Domain Object (事务封装，业务逻辑) <—> DAO
似乎 Ruby on rails 就是这种模型，它甚至把 Domain Object 和 DAO 都合并了。
这样做的优缺点：
简化了分层
也算符合 OO
该模型缺点：
很多不是 Domain Logic 的 Service 逻辑也被强行放入 Domain Object ，引起了 Domain Object 模型的不稳定；
Domain Object 暴露给 Web 层过多的信息，可能引起意想不到的副作用。

---------
## 网络
> [计算机网络](https://stubbornwdb.github.io/MyNotes/#/notes/计算机网络3.md) 

## 操作系统
> [计算机操作系统](https://stubbornwdb.github.io/MyNotes/#/notes/计算机操作系统.md) </br>
> [Linux](https://stubbornwdb.github.io/MyNotes/#/notes/Linux.md)

## 数据库
> [MySQL](https://stubbornwdb.github.io/MyNotes/#/mynotes/mysql/mysql.md) </br>
> [Redis面经](https://stubbornwdb.github.io/MyNotes/#/mynotes/redis/redis_question_v2.md)

## 消息队列
> [kafka](https://stubbornwdb.github.io/MyNotes/#/mynotes/mq/kafka.md)

## Go
> [Go](https://stubbornwdb.github.io/MyNotes/#/mynotes/Go/go-interview.md)

## 系统设计
> [系统设计](https://stubbornwdb.github.io/MyNotes/#/mynotes/SystemDesign/SystemDesign.md)

## 分布式
> [分布式](https://stubbornwdb.github.io/MyNotes/#/mynotes/DistributedSystem/distributed-system.md)