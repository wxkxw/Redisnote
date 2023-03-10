### 面试官： redis挂了，你们是怎么处理的？

    我： 听到这个问题的时候，我大脑突然抽了一下，说，redis挂了，重启。
    然后又说，如果是集群，主节点挂了，哨兵 sentinel 会把从节点升级为主节点
    如果是从节点挂了，重启之后会同步这一段时间的数据到从节点。
    然后又说了一些 主从节点是怎么 复制数据的。。。

### 这个时候面试官又说了一句：redis挂了，系统性能变得很差，应该怎么处理？

    我： 我们是先缓存与数据库一致性是 先写库，再删掉之前的缓存，重新加载缓存。。。

面试其实在这个问题之后已经结束了。

### 总结
其实这个问题是一个线上问题。redis挂了之后可能存在的性能问题和缓存雪崩是一样的。
大量的请求直接打到数据库，对我们的数据库的挑战是非常大的。而这个问题的处理我们之前的系统是经常发生的。

## redis挂了之后引发的系统问题
1. 响应速度变慢;
2. 并发能力下降;
3. 使用redis作为分布式锁的地方会出错，发生并发问题，产生业务问题;
4. db的压力几何倍的的增加。严重时可能导致dbCPU100%，或者db直接挂掉，从而导致整个服务不可用

## redis挂了之后如何恢复？

1. 如果发现及时，立即重启redis，并进行数据恢复;
2. 如果把db打挂了，我们的服务彻底不可用，我们需要在流量的入口进行关闭，重启数据库，并重启相关服务，并写入热点数据到redis。

## 拓展：如何避免出现redis挂？
    
缓存失效导致我们服务不可用是非常严重的事故，我们除了知道怎么去处理相关的问题之外，更重要的是避免类似的问题出现。
而我们常说的，应该避免缓存不可用的情况：
    缓存穿透，缓存击穿，缓存雪崩

### 缓存穿透：
缓存穿透：访问不存在的key，每次请求都直接请求到 db
解决办法：空值缓存
        使用布隆过滤器

### 缓存击穿：
缓存击穿：某个热点key过期，每次请求都直接请求到 db
解决办法：加锁更新
        对热点key设置永不过期

### 雪崩
redis挂了，或者大量key同一时间失效，大量请求直接打到db，db崩溃，导致整个系统崩溃。
解决办法：使用集群模式部署，引入sentinel-server
        大量key设置不同的过期时间