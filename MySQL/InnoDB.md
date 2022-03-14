# InnoDB体系结构
## 后台线程
### Master Thread
### IO Thread
InnoDB中使用AIO进行处理IO请求，这样可以极大提高数据库的性能

### Purge Thread
事务被提交之后，其被使用的undolog已经不再被使用，所以需要Purge Thread 来回收已经被使用并分配的undo页
### Page Clanner Thread
进行脏页的刷新
## 内存
### 缓冲池
### LRU List Free List FLush List
### 重做日志缓冲
# InnoDB关键特性
## 插入缓冲
## 两次写
## 自适应哈希索引
## 异步IO
## 刷新邻接页