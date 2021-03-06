MySQL体系图如下：
![](https://raw.githubusercontent.com/409582940/notes/main/images/20220314144620.png)
可以看出，MySQL主要由以下几部分组成
- 连接池组件
- 管理服务和工具组件
- SQL接口组件
- 查询分析器组件
- 优化器组件
- 缓冲组件
- 插件式存储引擎
- 物理文件
# MySQL存储引擎
## InnoDB
特点：天生支持事务，行锁。支持外键。使用MVCC来获取高并发性。实现了SQL的四种隔离级别
此外，还提供了插入缓冲，自适应哈希索引，二次读预读等功能、
## MyISAM
特点：不支持事务，没有行锁，只有表锁。支持全文索引，支持256TB的单表数据

## NDB
特点：集群存储引擎，数据存储到内存中。主键查找的速度极快
不过它连接操作是在MySQL数据库层完整的，不是通常的存储引擎层。所以复杂的连接需要巨大的网络开销，导致查询速度很慢
## Memory
表中数据存储到内存中，索引使用的是哈希索引不是B+树
只支持表锁，并发性比较差
# 不同存储引擎的对比
如图：
![](https://raw.githubusercontent.com/409582940/notes/main/images/20220314151803.png)
