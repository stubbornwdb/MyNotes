# 1.排行榜
有1亿用户和1亿短视频，设计一个实时的日排行榜，展示top100个热门视频，
热门视频的统计方法为统计视频的实时观看用户数，根据用户数排行。
设计方案后计算使用多少内存

使用Redis作为我们的内存数据存储，使用其内置的有序集合（sorted set）功能来实现排行榜。
在这个方案中，我们将视频ID作为成员（member），视频的实时观看用户数作为分数（score）。
当一个用户开始观看一个视频时，我们将视频ID添加到Redis中的有序集合，并将其观看用户数加1。
我们可以使用Redis的ZINCRBY命令来实现这个功能。
`ZINCRBY daily_ranking 1 video_id`
每次有新的观看用户时，我们可以使用ZRANK命令来获取当前视频在排行榜中的实时排名。

`ZRANK daily_ranking video_id`
我们可以使用ZREVRANGE命令来获取排行榜中的前100名热门视频。

`ZREVRANGE daily_ranking 0 99 WITHSCORES`
在每天的凌晨，我们可以使用DEL命令来清空当天的排行榜，并开始新的一天的排名统计。

`DEL daily_ranking`
接下来，我们来计算内存的使用情况。假设视频ID是64位整数（8字节），实时观看用户数也是64位整数（8字节）。
在Redis的有序集合中，每个成员（视频ID）和分数（观看用户数）的存储开销约为40字节（这是一个经验值，实
际存储开销可能略有不同）。

假设所有1亿个视频都有至少一个观看用户，那么我们需要存储1亿个成员和分数。因此，内存使用量大约为：
100,000,000 (videos) x 40 bytes (per video) = 4,000,000,000 bytes ≈ 3.73 GiB
但实际上，并非所有视频都会有观看用户，因此实际内存使用量可能会低于这个估算值。总之，使用Redis
实现实时日排行榜的内存开销是可以接受的。

# 2.短链服务
## 2.1 场景
根据 Short URL 还原 Long URL，并跳转
问题：
Long Url 和 Short Url 之间必须是一一对应的关系么? 
Short Url 长时间没人用需要释放么?

qps 存储
1. 询问面试官微博日活跃用户
   • 约100M
2. 推算产生一条Tiny URL的QPS
• 假设每个用户平均每天发 0.1 条带 URL 的微博
• Average Write QPS = 100M * 0.1 / 86400 ~ 100
• Peak Write QPS = 100 * 2 = 200
3. 推算点击一条Tiny URL的QPS
• 假设每个用户平均点1个Tiny URL
• Average Read QPS = 100M * 1 / 86400 ~ 1k
• Peak Read QPS = 2k
4. 推算每天产生的新的 URL 所占存储
• 100M * 0.1 ~ 10M 条
• 每一条 URL 长度平均 100 算，一共1G
• 1T 的硬盘可以用 3 年

2k QPS
一台 SSD支持 的MySQL完全可以搞定

## 2.2 服务
该系统比较简单，只有一个 Service
URL Service

TinyUrl只有一个UrlService
本身就是一个小Application 
无需关心其他的

• 函数设计
UrlService.encode(long_url) • UrlService.decode(short_url)

访问端口设计
• GET /<short_url>
• return a Http redirect response
• POST /data/shorten/
• Data = {url: http://xxxx }
• Return short url

## 2.3 Storage 数据存取
可以直接考虑使用Mysql

Base62
• 将 6 位的short url看做一个62进制数(0-9, a-z, A-Z)
• 每个short url 对应到一个整数
• 该整数对应数据库表的Primary Key —— Sequential ID
• 6 位可以表示的不同 URL 有多少?
• 5 位 = 625 = 0.9B = 9 亿
• 6 位 = 626 = 57 B = 570 亿
• 7 位 = 627 = 3.5 T = 35000 亿

• 优点:效率高
• 缺点:依赖于全局的自增ID

因为需要用到自增ID(Sequential ID)，因此只能选择使用 SQL 型数据库。
表单结构如下(id +  long_url)，shortURL 可以不存储在表单里，因为可以根据 id 来进行换算

## 2.4 Scale
如何提高响应速度?
利用缓存提速(Cache Aside) • 缓存里需要存两类数据:
• long to short(生成新 short url 时需要)
• short to long(查询 short url 时需要)

• 利用地理位置信息提速

•优化服务器访问速度
不同的地区，使用不同的 Web 服务器
通过DNS解析不同地区的用户到不同的服务器

• 优化数据访问速度
使用Centralized MySQL+Distributed Memcached
一个MySQL配多个Memcached，Memcached跨地区分布

• 什么时候需要多台数据库服务器?
Cache 资源不够
写操作越来越多
越来越多的请求无法通过 Cache 满足

• 增加多台数据库服务器可以优化什么?
解决“存不下”的问题 —— Storage的角度 • 解决“忙不过”的问题 —— QPS的角度

Tiny URL 主要是什么问题?



