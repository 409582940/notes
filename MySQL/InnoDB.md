# InnoDB体系结构
## 后台线程
### Master Thread
### IO Thread
InnoDB中使用`AIO`进行处理IO请求，这样可以极大提高数据库的性能
### Purge Thread
事务被提交之后，其被使用的`undolog`已经不再被使用，所以需要`Purge Thread` 来回收已经被使用并分配的undo页
### Page Clanner Thread
进行脏页的刷新
![](https://raw.githubusercontent.com/409582940/notes/main/images/20220315165350.png)
## 内存
### 缓冲池
`InnoDB`是基于磁盘存储的，并对其中的记录使用页进行管理。

缓存池简单来说就是一块特殊的内存区域，通过内存快速读写的特性补偿磁盘读写速度比较慢对性
能的影响。

数据库在读取一条数据时，先将对应的页放到缓存池中，如果下一次还读取相同的页时候，首先判
断时候在缓存池中，在缓存池中就直接读取，否则直接读取磁盘中的页

对于数据库的修改操作，首先修改缓存池中的页，然后再将缓存池中的页按照一定频率刷到磁盘上。

并不是每一次的修改都立即刷到磁盘上，而是通过一种叫做`checkpoint`的方式刷新到磁盘

缓存池缓存的页包括数据页，索引页，undo页，插入缓冲，自适应哈希索引，锁，数据字典。
#### 什么是Checkpoint？
如果一条DML语句，造成了当前页比磁盘中的页新，此时这个页就是脏页，数据库就需要将当前页
写入磁盘。
如果每一次页发生变化，都写入磁盘，这个开销很大。如果数据集中再几个页上，性能会很差。

如果页写入到磁盘的过程中发生宕机，则容易造成数据丢失。为了防止数据丢失，当前通常采用`write Ahead log`操作，即每次事务提交时候，先写`redo log`，再修改页。如果发生了宕机，可以通过`redo log`来进行恢复，这个保证了数据库的持久性的要求

`Checkpoint`解决的问题就是：
- 缩短数据库的恢复时间
- 缓存池不够用时候，将脏页刷新到磁盘
- `redo log` 不可用时候，刷新脏页

`Checkpoint`所做的事情其实就是将脏页写入磁盘，不同的就是每次写入多少，写入磁盘时机，每次从哪里取脏页读，什么时间点触发Checkpoint等
`Checkpoint`分为以下几种：
- Sharp Checkpoint
发生在数据库关闭的时候，将所有的脏页写入数据库
- Fuzzy Checkpoint
	- Master Thread Checkpoint
	- FLUSH_LRU_LIST Checkpoint
	- Async/Sync Flash Checkpoint
	- Dirty Page too much Checkpoint


### LRU List    Free List    FLush List
### 重做日志缓冲
# InnoDB关键特性
## 插入缓冲
## 两次写
## 自适应哈希索引
## 异步IO（AIO）

## 刷新邻接页
