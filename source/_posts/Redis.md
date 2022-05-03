---
layout: post
title: Redis
author: liugezhou
date: 2021-06-01
updated: 2021-06-05
category: 
  服务端
tags: 
  redis
---
> > 1. NoSql = Not Only SQL
> 1. Key - Value 存储：字符串 String｜字符串列表 list｜有序集合 sorted set｜hash｜字符串集合set
> 1. 09年开发完成
> 1. C语言开发
> 1. 每秒能写8.1万次，能读11万次
> 1. 应用场景：缓存、任务队列、网站访问统计、应用排行榜、数据过期处理，分布式集群里的session处理
> 1. CentOs-6.5 安装：
>    - 下载源码后，对源码进行编译，_**编译需要一个gcc环境**_。yum install gcc-c++
>    - 将下载好的源码上传至服务器 /root下
>    - tar -zxvf redis-x.x.x.tar.gz
>    - cd  redis-x.x.x
>    - make
>    - make PREFIX=/usr/local/redis install
>    - cd /usr/local/redis/bin --存在一些可执行文件
>    - cd ~/redis.x.x.x
>    - cp redis.conf /usr/local/redis
>    - cd /usr/local/redis/bin
>    - 前端启动：_**redis-server**_(窗口不能关闭)
>    - 后端启动：修改配置文件
>    - vim redis.conf
>    - 修改 daemonize yes
>    - **_redis-server redis.conf_**
>    - 验证启动：ps -ef | grep redis
>    - 停止启动：kill -9 PID  ||  redis-cli shutdown
>    - 进入命令行交互：**_redis-cli _**
> 8. Jedis:Jedis是Redis官方首先的Java客户端开发包
> 8. Key不要过长--不建议超过1024
> 8. 存储String：
>    - 以二进制方式操作
>    - 其value数据长度最多不超过512M
>    - 赋值**_ set_**、取值**_ get_**、删值 _**del**_、**_inc_**r num、**_incrby_** num 5、**_append_** num 5
> 11. 存储Hash：
>    - String Key和String Value的map容器
>    - 每一个Hash可以存储4294967268个键值对
>    - 赋值**_：hset_** myhash username jack
>    - 赋值：**_hset_** myhash age 18
>    - **_        hmset_** myhsh username roose age 21
>    - 取值：**_hget_** myhash username ==> jack
>    - **_ hmget_** myhash username age
>    - **_hgetall_** myhash
>    - 删除一个：**_hdel_** myhash username
>    - 删除全部：**_del_** myhash
>    - 增加数字： _**hincrby**_ myhash age  5
>    - 判断属性中的某个值是否存在：**_hexists_** myhash username
>    - 获取属性数量 ： **_hlen_** myhash == 2
>    - 获取所有属性名称：**_hkeys_** myhash
>    - 获取所有值： **hvals** myhash
> 12. 存储list：
>    - ArrayList使用数组方式：
>    - LinkedList使用双向链接方式：
>    - 双向链表中增加数据
>    - 双向链表中删除数据
>    - 两端添加：
>       - lpush mylist a b c --> c,b,a
>       - lpush mylist 1 2 3 --> 3 2 1 c b a
>       - rpush mylist2 a b c
>       - rpush mylist2 1 2 3 
>       - 

>    - 查看列表: 
>       - lrange mylist  (start,end)
>       - lrange mylist 0 5 --> 3 2 1 c b a
>       - lrange mylist2 0 -2 a b c 1 2
>    - 两端弹出：
>       - lpop mylist2 --> 3
>       - lrange mylish  -->2 1 c b a
>       - rpop mylist2 --> 3
>       - lrange mylist2 --> a b  c 1 2 
>    - 获取列表元素个数:
>       - llen mylish  -->返回元素个数
>    - 扩展命令
>       - lpushx mylist2 (mylist2这个值存在才会在头部插入)
>       - lrem mylist2 2 3（从头开始删除2个3）
> 13. 存储Set：不允许出现重复的元素
>    - 添加元素
>       - sadd myset a b c
>       - sadd myset  1 2 3
>    - 删除元素
>       - srem myset 1 2
>    - 获得集合中的元素
>       - smembers myset --> c b a 3
>    - 判断元素是否存在
>       - sismember myset 3
> 14. keys通用操作
>    - **keys ***
>    - **keys my?**  ：查看以my开头的key
>    - **del** my1 my2 my3 ：删除这些key
>    - **exists** my1   ：查看my1是否存在
>    - r**ename** old new ：重命名
>    - **expire** newkey 1000 :过期时间
>    - **type key**:查看某个key的类型
> 15. redis特性
>    - 多数据库
>       - 16个数据库 0-15 （客户端默认0）
>       - select 1  --- 选择1号数据库
>       - move key 2 ---- 将1中的key移动到2
>    - Redis事务
>       - multi :开启事务
>       - exec ：提交事务
>       - discard：回滚操作
> 16. redis的持久化概述
>    - RDB持久化：默认支持不需要配置：在指定时间间隔内，将数据写入到磁盘
>       - redis.conf: save 900 /save 300 10 /save 60 10000 (默认支持)--可以查看保存的路径在 **dump.rdb**
>    - AOF持久化
>       - 优势：更高的数据安全--每一次变动都会记录到磁盘中。 
>       - 劣势：内容文件大、效率低
>       - 需要进行相应配置--redi.conf ==>appendonly yes -->写入到appendonly.aof 
>    - 无持久化
>    - 同时使用RDB和AOF
> 17. 服务器查看redis路径
> - ps -ef | grep redis
> - whereis  redis
> 18. 查看本地是否安装redis：brew info redis
> 18. 批量删除某个规则的 redisKey：
>    1. [root@node1 src]# ./redis-cli -h 127.0.0.1 -p 6379 keys "java_suisui*" | xargs ./redis-cli -h 127.0.0.1 -p 6379 del

