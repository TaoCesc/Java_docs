

## 五种基本数据类型

### 字符串对象

```
set number 520
get number
```

- 应用场景： 共享session、分布式锁、计数器、限流
- 内部编码：`int（8字节长整型）/embstr（小于等于39字节字符串）/raw（大于39个字节字符串）`

### 散列对象

内部实现结构上与JDK1.7的HashMap一致，底层通过数据+链表实现。

- 哈希类型是指v（值）本身又是一个键值对（k-v)结构
- 举例： `hset key field value`,`hget key field`
- 内部编码： `ziplist(压缩列表)`,`hashtable(哈希表)`
- 应用场景：
- - 记录整个博客的访问人数（数据量大会考虑HyperLogLog，但是这个数据结构存在很小的误差，如果不能接受误差，可以考虑别的方案）
  - 记录博客中某个博主的主页访问量、博主的姓名、联系方式、住址

### 列表对象

简介：列表（list）类型是用来存储多个有序的字符串，一个列表最多可以存储2^32-1个元素。

简单实用举例：` lpush  key  value [value ...]` 、`lrange key start end`

内部编码：ziplist（压缩列表）、linkedlist（链表）

应用场景： 消息队列，文章列表

- lpush+lpop=Stack（栈）
- lpush+rpop=Queue（队列）
- lpsh+ltrim=Capped Collection（有限集合）
- lpush+brpop=Message Queue（消息队列）

```c++
typedef struct listNode{
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    void *value;
}listNode;
```

![image-20211115191406712](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211115191406712.png)

- 链表被用于实现Redis的各种功能，比如列表键、发布与订阅、慢查询、监视器等。

### 集合对象

**Redis的set集合相当于Java的HashSet。**

![image-20211117201357931](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211117201357931.png)

相当于

- 简介：集合（set）类型也是用来保存多个的字符串元素，但是不允许重复元素
- 简单使用举例：`sadd key element [element ...]`、`smembers key`
- 内部编码：`intset（整数集合）`、`hashtable（哈希表）`
- 应用场景： 用户标签、生成随机数抽奖、社交需求。

### 有序集合

zset为有序（优先score排序，score相同则元素字典序），自动去重的集合数据类型，其底层实现为字典和跳跃表，当数据比较少的时候用压缩列表（ziplist）编码结构存储。

**同时满足**以下两个条件采用ziplist存储：

- 有序集合保存的元素数量小于默认值128个
- 有序集合保存的所有元素的长度小于默认值64字节

#### ziplist存储方式

当ziplist作为zset的底层存储结构时候，每个集合元素使用两个紧挨在一起的压缩列表节点来保存，**第一个节点保存元素的成员，第二个元素保存元素的分值**

![image-20211122160546544](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211122160546544.png)

#### 字典+跳跃表



## 数据结构

### SDS(simple dynamic string)

```c++
struct sdshdr{
    //记录buf数组已使用字节的数量
    int len;
    //记录buf数组未使用字节的数量
    int free;
    //字节数组，用于保存字符串
    char buf[];
}
```

![image-20211115190014649](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211115190014649.png)

**SDS的好处**

**常数复杂度获取字符串长度**

通过`STRLEN`命令得到一个字符串的长度，在`SDS`结构中`len`属性记录了字符串的长度。

**杜绝缓冲区（数据）溢出**

当需要修改数据时，首先会检查当前SDS空间len是否满足，不满足则自动扩容空间至修改所需的大小，然后再实行修改

==内存重分配策略==

SDS通过两种内存重分配策略，很好的解决了字符串在增长和缩短时的内存分配问题。

1. 空间预分配

当修改字符串对SDS空间进行扩展时，不仅会为SDS分配修改所必要的空间，还会再为SDS分配额外的未使用空间`free`。**其大小一般为修改后的长度**。

2. 惰性空间释放

用于优化SDS字符串缩短操作，当缩短SDS字符串后，并不会立即执行内存重分配回收多余的空间，而是用`free`属性将这些空间记录下来。

### 链表

链表节点

```c++
typedef struct listNode{
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    void *value;
}listNode;
```

```c++
typedef struct list {
    // 链表头节点
    listNode *head;
    // 链表尾节点
    listNode *tail;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
    // 链表所包含的节点数量
    unsigned long len;
} list;
```

链表特性

- 双端链表： 带有指向前置节点和后置节点的指针
- 无环： 表头节点的prev和表尾节点的next都指向null
- 链表长度计数器：带有len属性

### 字典

用于保存键值对（key-value pair)

```sql
set msg 'hello world'
# 在数据库中创建一个键为"msg",值为"hello world"
```

当一个哈希键包含的键值对比较多，又或者键值对中的元素都是比较长的字符串时，Redis就会使用字典作为哈希键的底层实现。

==字典使用哈希表作为底层实现，一个哈希表里面又多个哈希表节点，而每个哈希表节点就保存了字典中的一个键值对。==

**哈希表**

```c++
typedef struct dictht{
    //哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小掩码，用于计算索引值
    //总是等于size - 1
    unsigned long sizemask;
    //已有节点数量
    unsigned long used;
}dictht;
```

**哈希表节点**

```c++
typedef struct dictEntry{
    //键
    void *key;
    //值
    union{
        void *val;
        uint64_tu64;
        int64_ts64;
    }v;
    //指向下一个哈希表节点，形成链表
    struct dictEntry *next;
}dictEntry;
```

![image-20211115192436628](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211115192436628.png)

**字典实现**

```c++
typedef struct dict{
    //类型指定
    dictType * type;
    //私有属性
    void * privdata;
    //哈希表
    dictht ht[2];
    //rehash索引
    //当不在rehash时，值为-1
    int rehashidx;
}dict;
```

- ht属性是一个包含两个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht [0]哈希表,ht [1]哈希表只会在对ht[0]哈希表进行rehash时使用。
- 除了ht[1]之外，另一个和rehash有关的属性就是rehashidx，它记录了rehash目前的进度，如果目前没有在进行rehash，那么它的值为-1。

Rehash过程

1. 为字典的ht[1]分配空间，取决于ht[0].used属性的值

- 如果执行的是扩展操作，那么ht[1]的大小为第一个大于等于ht[0].used * 2的 2^n
- 如果执行的收缩操作，那么ht[1]的大小为第一个大于等于ht[0].used 的2^n

2. 将保存在ht[0]中的所有键值对rehash到ht[1]上面：rehash是指重新计算哈希值和索引值。
3. 当ht[0]包含的所有键值对都迁移到了ht[1]之后（ht[0]为空表时), 释放ht[0]， 将ht[1]设置为ht[0]，并且ht[1]新创建一个空白哈希表。

==扩展和收缩的条件==

当以下条件中的任意一个被满足时，程序会自动开始对哈希表执行扩展操作：

1. 服务器目前没有执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的**负载因子大于等于1**
2. 服务器目前执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的**负载因子大于等于5.**

收缩： **负载因子小于0.1**



### 跳跃表

跳跃表是有序集合（Sorted Set）的底层实现之一，如果有序集合包含的元素比较多，或者元素的成员是比较长的字符串时，Redis会使用跳跃表做有序集合的底层实现

跳跃表的定义

==多层的链表==

- **多层**的结构组成，每层是一个有序的链表
- 最底层（level 1）的链表包含所有的元素
- 跳跃表的查找次数近似于层数
- 跳跃表是一种随机化的数据结构（通过抛硬币来决定层数）

![image-20211122153123500](E:\实习\Java_docs\pics\image-20211122153123500.png)

### 整数集合

```c++
typedef struct intset{
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
}intset;
```

### 压缩列表

是由一系列特殊编码的连续内存块组成的顺序性（sequential）数据结构，一个压缩列表可以包含多个节点，每个节点可以保存一个字节数组或者一个整数值。

压缩列表是列表（List）和散列（Hash）的底层实现实现之一，一个列表只包含少量列表项，并且每个列表项是小整数值或比较短的字符串，会使用压缩列表作为底层实现

`zlbytes`：记录整个压缩列表占用的内存字节数，在压缩列表内存重分配，或者计算`zlend`的位置时使用

`zltail`：记录压缩列表表尾节点距离压缩列表的起始地址有多少字节，通过该偏移量，可以不用遍历整个压缩列表就可以确定表尾节点的地址

`zllen`：记录压缩列表包含的节点数量，但该属性值小于UINT16_MAX（65535）时，该值就是压缩列表的节点数量，否则需要遍历整个压缩列表才能计算出真实的节点数量

`entryX`：压缩列表的节点

`zlend`：特殊值0xFF（十进制255），用于标记压缩列表的末端

压缩列表的节点构成

![image-20211122154047229](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211122154047229.png)

- `previous_entry_ength`：记录压缩列表前一个字节的长度
- `encoding`：节点的encoding保存的是节点的content的内容类型
- `content`：content区域用于保存节点的内容，节点内容类型和长度由encoding决定

## RDB/AOF持久化

### RDB快照持久化

RDB持久化是通过**快照**的方式，即在指定的时间间隔内将内存中的数据集快照写入磁盘。在创建快照之后，用户可以备份该快照，可以将快照复制到其他服务器以创建相同数据的服务器副本，或者在重启服务器后恢复数据。RDB是Redis**默认的持久化方式**

RDB文件默认为当前工作目录下的`dump.rdb`，可以根据配置文件中的`dbfilename`和`dir`设置RDB的文件名和文件位置

==触发快照的时机==

- 执行save和bgsave命令
- 配置文件设置`save<seconds><changes>`规则，自动间隔性执行bgsave命令
- 主从复制时，从库全量复制同步主库数据，主库会执行bgsave
- 执行flushall命令清空服务器数据
- 执行shutdown命令关闭Redis时，会执行save命令

#### save和bgsave命令

执行save和bgsave命令，可以手动触发快照，生成RDB文件

使用save命令会**阻塞Redis服务器进程**，服务器进程在RDB文件创建完成之前是不能处理任何请求

而bgsave命令，会**fork**一个子进程，然后该子进程会负责创建RDB文件。

```c++
127.0.0.1:6379>bgsave
Background saving started
```

`fork`一个子进程，子进程会把数据集先写入临时文件，写入成功之后，再替换之前的RDB文件，用二进制压缩存储，这样可以保证RDB文件始终存储的是完整的持久化内容

### AOF持久化

AOF持久化会把被执行的写命令写到AOF文件的末尾，记录数据的变化。默认情况下，Redis是没有开启AOF持久化的，开启后，每执行一条更改Redis数据的命令，都会把该命令追加到AOF文件中，

==AOF需要记录Redis的每个写命令，步骤为：命令追加（append）、文件写入（write）和文件同步（sync）==

**命令追加**：

开启AOF持久化功能后，服务器每执行一个写命令，都会把该命令以协议格式先追加到`aof_buf`缓存区的末尾，而不是直接写入文件，减少硬盘IO次数。

**文件写入(write)和文件同步(sync)**

对于何时把`aof_buf`缓冲区的内容写入保存在AOF文件中，Redis提供了多种策略

- `appendsync always`: 将`aof_buf`缓冲区的所有内容写入并同步到AOF文件，每个写命令同步写入磁盘
- `appendfsync everysec`: 将`aof_buf`缓冲区的内容写入AOF文件，每秒同步一次，该操作由一个线程单独完成。
- `appendfsync no`：将`aof_buf`缓存区的内容写入AOF文件，什么时候同步由操作系统来决定

`appendfsync`选项的默认配置为`everysec`，即每秒执行一次同步

#### AOF重写（rewrite）

```sql
127.0.0.1:6379> set s1 hello
OK
127.0.0.1:6379> set s2 world
OK
```

==appendonly.aof==

```
*3
$3
set
$2
s1
$5
hello
*3
$3
set
$2
s2
$5
world
```

该命令格式为Redis的序列化协议（RESP）。`*3`代表这个命令有三个参数，`$3`表示该参数长度为3

AOF重写的目的就是减小AOF文件的体积，不过值得注意的是：**AOF文件重写并不需要对现有的AOF文件进行任何读取、分享和写入操作，而是通过读取服务器当前的数据库状态来实现的**

文件重写可分为手动触发和自动触发，手动触发执行`bgrewriteaof`命令，该命令的执行跟`bgsave`触发快照时类似的，都是先`fork`一个子进程做具体的工作

自动触发会根据`auto-aof-rewrite-percentage`和`auto-aof-rewrite-min-size 64mb`配置来自动执行`bgrewriteaof`命令

- 重写会有大量的写入操作，所以服务器进程会`fork`一个子进程来创建一个新的AOF文件

- 在重写期间，服务器进程继续处理命令请求，如果有写入的命令，追加到`aof_buf`的同时，还会追加到`aof_rewrite_buf`AOF重写缓冲区

- 当子进程完成重写之后，会给父进程一个信号，然后父进程会把AOF重写缓冲区的内容写进新的AOF临时文件中，再对新的AOF文件改名完成替换，这样可以保证新的AOF文件与当前数据库数据的一致性

### RDB和AOF优缺点

**RDB**

优点：

- RDB快照是一个压缩过的非常紧凑的文件，保存着某个时间点的数据集，适合做数据的备份，灾难恢复
- 可以最大化Redis性能，在保存RDB文件，服务器进程只需fork一个子进程来完成RDB文件的创建，父进程不需要做IO操作
- 与AOF相比，恢复大数据集的时候会更快

缺点：

- RDB的数据安全性不如AOF，保存整个数据集的过程比较繁重，如果服务器宕机，可能丢失几分钟的数据
- Redis数据集比较大时，fork的子进程会比较消耗CPU、耗时



**AOF**

优点：

- 数据更完整，安全性更高，（取决于fsync策略）

- AOF文件只是一个追加的日志文件，内容是可读的，适合误删紧急恢复

缺点：

- 对于相同的数据集，AOF文件体积要大于RDB文件

- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB。 不过在一般情况下， 每秒 fsync 的性能依然非常高

## 内存回收

1. 在Redis中，set指令可以指定key的过期时间，当过期时间到达后，key就失效了
2. Redis是基于内存操作的，所有数据都是保存在内存中的。

**内存回收机制**

Redis的内存回收主要分为过期删除策略和内存淘汰策略两部分。

### 过期删除策略

删除达到过期时间的key

**1.定时删除**

对于每一个设置了过期时间的key都会创建一个定时器，一旦到达过期时间就会立即删除。该策略可以立即清除过期的数据，对内存较友好，但是缺点是占用了大量的CPU资源去处理过期的数据，会影响Redis的吞吐量和响应时间。

**2.惰性删除**

当访问一个key时，才判断该key是否过期，过期则删除。该策略能最大限度地节省CPU资源。有一种极端的情况是可能出现大量的过期key没有被再次访问，因为不会被清除，导致占用了大量的内存。

**3.定期删除**

每隔一段时间，扫描Redis中过期key字典，并且清除部分过期的key。这是**折中方案。**

*redisDb结构体定义*

```c++
typedef struct redisDb{
    dict *dict; 	//键空间，保存所有键值对
    dict *expires;	//保存所有过期的key
    dict * blocking_keys;	
    dict *ready_keys;
    dict *watched_keys;
    int id; 	//数据库ID字段，代表不同的数据库
    long long avg_ttl; 
}
```

**expires属性**

它的类型也是字典，Redis会把所有的过期键值对加入到expires，之后再通过定期删除清理expires里面的值，加入expires的场景有：

1. set指定过期时间expire

如果设置key的时候指定了过期时间，Redis会将这个key直接加入到expires字典中并将超时时间设置到该字典元素。

2. 调用expire命令

显式指定某个key的过期时间

3. 恢复或者修改数据

从Redis持久化文件中恢复文件或者修改key，如果数据中的key已经设置过期时间，那么就将他加入expires

**Redis清理过期key的时机**

1. Redis在启动的时候，会注册两种事件，一种是时间事件，另一种是文件事件。

在时间事件中，redis注册的回调函数是`serverCron`，在定时任务回调函数中，通过调用databasesCron清理部分过期key（*定期删除的实现*）

```c++
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData)
{
    databasesCron();
}
```

2. 每次访问key的时候，都会调用`expireIfNeeded`函数判断key是否过期，如果是，清理key*（惰性删除的实现）*

```c++
robj *lookupKeyRead(redisDb *db, robj *key){
    robj *val;
    expireIfNeeded(db, key);
    val = lookupKey(db, key);
    
    return val;
}
```

**删除key**

Redis4.0以前，使用`del`指令删除，del会直接释放对象的内存。

Redis4.0版本引入了`unlink`指令，能对删除操作进行‘懒’处理，将删除操作丢给后台线程，由后台线程来异步回收内存。

==实际上，在判断key需要过期之后，真正删除key 的过程是先广播expire事件到从库和AOF文件中，然后根据Redis的配置决定立即删除还是异步删除。==

如果是立即删除，Redis会立即释放key和value占用的的内存空间，否则，Redis会在另一个bio线程中释放需要延迟删除的空间。

### 内存淘汰策略

是指内存达到**maxmemory**极限时，使用某种算法来决定清理掉哪些数据，以保证新数据的存入。

- noeviction：当内存不足以容纳新写入数据时，新写入操作会报错
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间`(server.db[i].dict)`中，移除最近最少使用的key**（最常用）**

- allkeys-random：当内存不足以容纳新写入数据时，在键空间（`server.db[i].dict`）中，随机移除某个 key。

- volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间（`server.db[i].expires`）中，移除最近最少使用的 key。

- volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间（`server.db[i].expires`）中，随机移除某个 key。

- volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间（`server.db[i].expires`）中，有更早过期时间的 key 优先移除。

==在配置文件中，通过maxmemory-policy可以配置要使用哪一个淘汰机制。==

```c++
int processCommand(client *c)
{
    ...
    if (server.maxmemory) {
        int retval = freeMemoryIfNeeded();  
    }
    ...
}
```

**LRU实现原理**

在淘汰key时，Redis默认最常用的是LRU算法（Latest Recently Used）。Redis通过在每一个redisObject保存lru属性来保存key最近的访问时间，在实现LRU算法时直接读取key的lru属性。

具体实现时，Redis遍历每一个db，从每一个db中随机抽取一批样本key，默认是3个key，再从这3个key中，删除最近最少使用的key。

## 三大缓存问题

### 缓存穿透

缓存穿透是指查询一条数据库和缓存都没有的一条数据，就会一直查询数据库，对数据库的访问压力就会增大。

解决方案：

1. 缓存空对象：代码维护比较简单，但是效果不好
2. 布隆过滤器：代码维护复杂，但是效果很好

**缓存空对象**

缓存空对象是指当一个请求过来缓存和数据库中都不存在该请求的数据，第一次请求就会掉过缓存进行数据库的访问，并且访问数据库后返回为空，此时也将该对象进行缓存。

若是再次进行访问该空对象的时候，就会直接**击中缓存**，而不是再次**数据库**，缓存空对象实现的原理图如下：

![image-20211116213555543](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211116213555543.png)

缓存空对象的实现代码很简单，但是缓存空对象会带来比较大的问题，就是缓存中会存在很多空对象，占用**内存的空间**，浪费资源，一个解决的办法就是设置空对象的**较短的过期时间**

**布隆过滤器**

布隆过滤器是一种基于**概率**的**数据结构**，主要用来判断某个元素是否在集合内，它具有**运行速度快**（时间效率），**占用内存小**的优点（空间效率），但是有一定的**误识别率**和**删除困难**的问题。它只能告诉你某个元素一定不在集合内或可能在集合内。

在实际项目中会启动一个**系统任务**或者**定时任务**，来初始化布隆过滤器，将热点查询数据的id放进布隆过滤器里面，当用户再次请求的时候，使用布隆过滤器进行判断，该订单的id是否在布隆过滤器中存在，不存在直接返回null

### 缓存击穿

缓存击穿是指一个key非常热点，在不停的扛着大并发，**大并发**集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，瞬间对数据库的访问压力增大。

缓存击穿这里强调的是**并发**，造成缓存击穿的原因有两个：

1. 该数据没有人查询过，第一次就大并发的访问
2. 添加到了缓存，但是**失效**了，大并发访问

对于缓存击穿的解决方案是加锁：

当用户出现**大并发**访问的时候，在查询缓存的时候和查询数据库的过程加锁，只能第一个进来的请求进行执行，当第一个请求把该数据放进缓存中，接下来的访问就会直接集中缓存，防止了**缓存击穿**。

业界比较普遍的一种做法，即根据key获取value值为空时，锁上，从数据库中`load`数据后再释放锁。若其它线程获取锁失败，则等待一段时间后重试。

以一个获取商品库存的案例进行代码的演示，**单机版**的锁实现具体实现的代码如下：

```java
//获取库存数量
public String getProduceNum(String key){
    try{
        synchronized(this){
            //加锁
            //缓存中取数据，并存入缓存中
            int num = Integer.parseInt(redisTemplate.opsForValue().get(key));
            
            if(num &gt; 0){
                //每查一次库存 - 1
                redisTemplate.opsForValue().set(key, (num - 1)+"");
                System.out.println("剩余库存为" + (num - 1));
            }
            else{
                System.out.println("库存为0");
            }
            
        }
    }
}
```

### 缓存雪崩

缓存雪崩是指在某一个时间段，缓存集中过期失效，此刻无数的请求直接绕开缓存，直接请求数据库。

造成缓存雪崩的原因，有两点：

1. redis宕机
2. 大部分数据失效

比如天猫双11，马上就要到双11零点，很快就会迎来一波抢购，这波商品在23点集中的放入了缓存，假设缓存一个小时，那么到了凌晨24点的时候，这批商品的缓存就都过期了。

而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰，对数据库造成压力，甚至压垮数据库。

**对于缓存雪崩的解决方案有以下两种**：

1. 搭建高可用的集群，防止单机的redis宕机。
2. 设置不同的过期时间，防止同意之间内大量的key失效。

## 数据一致性

### KV、DB读写模式

缓存+数据库读写模式，就是**Cache Aside Pattern**

- 读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回相应。
- 更新的时候，先**更新数据库，然后再删除缓存。**

==为什么是删除缓存，而不是更新缓存==

很多时候，在复杂的缓存场景，缓存不单单是数据库中直接取出来的值。

比如可能更新了某个表的一个字段，然后其对应的缓存，是需要查询另外两个表的数据并进行运算，才能计算出缓存最新的值的。

## Redis线程模型

Redis内部使用文件事件处理器`file event handler`，这个文件事件处理器是单线程的，所以Redis才叫做单线程的模型。它采用IO多路复用机制同时监听多个**Socket**，根据Socket上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含4个部分：

- 多个Socket
- IO多路复用程序
- 文件事件派发器
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

## 什么是热Key问题，如何解决热Key问题

在Redis中，我们把访问频率高的Key，称为热点key

如果某一热点key的请求到服务器主机时，由于请求量特别大，可能会导致主机资源不足，甚至宕机，从而影响正常的服务。

**热点key的产生**

- 用户消费的数据远大于生产的数据，如秒杀、热点新闻等读多写少的场景。
- 请求分片集中，超过单Redis服务器的性能，比如固定名称key，Hash落入同一台服务器。

**如何解决热Key问题**

- Redis集群扩容：增加分片副本，均衡读流程。

- 将热key分散到不同的服务器中；
- 使用二级缓存，即JVM本地缓存,减少Redis的读请求。

## 集群方案

### 主从复制模式

![image-20211118230939850](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211118230939850.png)

`Redis`主从复制，主从库模式一个`Master`主节点多`Slave`从节点的模式，将一份数据保存在多`Slave`个实例中，**增加副本冗余量**，当某些出现宕机后，`Redis`服务还可以使用。

但是这会存在数据不一致问题，那redis的副本集是如何数据一致性？

==`Redis`为了保证数据副本的一致，主从库之间采用读写分离的方式==：

- **读操作：主库、从库都可以执行处理；**
- **写操作：先在主库执行，再由主库将写操作同步给从库。**

使用读写分离方式的好处，可以避免当主从库都可以处理写操作时，主从库处理写操作加锁等一系列巨额的开销。

主从库是同步数据方式有两种：

#### **全量同步**

- 当一次从库启动时，从库给主库发送`psync`命令进行数据同步（`psync` 命令包含：主库的 `runID` 和复制进度 `offset` 两个参数）
- 当主库接收到ysync命令后将会保存RDB文件并发送给从库，发送期间会使用缓存区`（replication buffer)`记录后续的所有写操作，从库收到数据后，会先**清空**当前数据库，然后加载从主库获取的RDB文件。
- 当主库完成RDB文件发送后，也会把将保存发送RDB文件期间期间写操作的`replication buffer`发送给从库，从库再重新执行这些操作，这样主从库就实现同步了。

另外，为了分担主库生成 RDB 文件和传输 RDB 文件压力，提高效率，可以使用 **“主 - 从 - 从”模式将主库生成 RDB 和传输 RDB 的压力，以级联的方式分散到从库上。**

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211118232056333.png" alt="image-20211118232056333" style="zoom:67%;" />

#### 增量同步

增量同步，基于环形缓冲区`repl_backlog_buffer`缓存区实现

在环形缓冲区，主库会记录自己写到的位置`master_repl_offset`，从库会记录自己已经读到的位置`slave_repl_offset`,主库并通过`master_repl_offset`和`slave_repl_offset`的差值的数据同步到从库。

==主从库间网络断了，主从库会采用增量复制的方式继续同步==，主库会把断连期间收到的写操作命令，写入`replication_buffer`，同时也会把这些操作命令也写入 `repl_backlog_buffer` 这个缓冲区，然后主库并通过`master_repl_offset` 和 `slave_repl_offset`的差值数据同步到从库。

因为`repl_backlog_buffer` 是一个环形缓冲区，当在缓冲区写满后，主库会继续写入，此时，会出现什么情况呢？

覆盖掉之前写入的操作。如果从库的读取速度比较慢，就有可能导致从库还未读取的操作被主库新写的操作覆盖了，这会导致主从库间的数据不一致。因此需要关注 `repl_backlog_size`参数，调整合适的缓冲空间大小，避免数据覆盖，主从数据不一致。

### Sentinel 哨兵模式

哨兵机制是实现主从库自动切换的关键机制，其主要分为三个阶段:

- 监控：哨兵进程会周期性地给所有的主从库发送PING命令，检测它们是否仍然在线运行。
- 选主（选择主库）：主库挂了以后，哨兵基于一定规则评分选举出来一个从库实例新的主库。
- 通知：哨兵会将新主库的信息发送给其他从库，让它们和新主库建立连接，并进行数据复制。**同时，哨兵会把新主库的信息广播通知给客户端，让它们把请求操作发到新主库上。**

![image-20211118232900067](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211118232900067.png)

**其中，在监控中如何判断主库是否处于下线状态？**

- **主观下线**： **哨兵进程会使用 PING 命令检测它自己和主、从库的网络连接情况，用来判断实例的状态，** 如果单哨兵发现主库或从库对 PING 命令的响应超时了，那么，哨兵就会先把它标记为“主观下线”
- 客观下线：在哨兵集群中，基于少数服从多数，多数实例都判定主库已“主观下线”，则认为主库“客观下线”。

==哨兵之间时如何互相通信的呢==

哨兵集群中哨兵实例之间可以相互发现，**基于Redis提供的发布/订阅机制（pub/sub机制）**

哨兵可以在主库中发布/订阅消息，在主库上有一个名为`\_sentinel_:hello`的频道，不同哨兵就是通过它来相互发现，实现互相通信，而且只有订阅了同一个频道的应用，才能通过发布的消息进行信息交换。

==如何选举新的主库==

- 从库的当前在线状态
- 判断它之前的网络连接状态，通过`down-after-milliseconds * num`（断开连接次数），当断开连接次数超过阈值，不适合作为新的主库。

==如何选举leader哨兵==

基于少数服从多数原则“投票仲裁”选举出来

- 当任何一个从库判定主库“主观下线”后，发送命令`s-master-down-by-addr`命令发送想要称为leader的信号。
- 其他哨兵根据于主机连接情况作出相应的相应，而且如果有多个哨兵发起请求，每个哨兵的赞成票只能投给其中一个，其他只能为反对票。

想要成为Leader 的哨兵，要满足两个条件：

- 第一，获得半数以上的赞成票；
- 第二，获得的票数同时还需要大于等于哨兵配置文件中的`quorum`值。

==选举完leader哨兵并新主库切换完毕之后，那么leader哨兵怎么通知客户端？==

基于哨兵自身的 pub/sub 功能，实现了客户端和哨兵之间的事件通知，客户端订阅哨兵自身消息频道 

#### 数据丢失-主从异步复制

因为`master` 将数据复制给`slave`是异步实现的，在复制过程中，这可能存在master有部分数据还没复制到slave，master就宕机了，此时这些部分数据就丢失了。

#### 数据丢失-脑裂

何为脑裂？当一个集群中的 master 恰好网络故障，导致与 sentinal 通信不上了，sentinal会认为master下线，且sentinal选举出一个slave 作为新的 master，此时就存在两个 master了。

可能存在client还没来得及切换到新的master，还继续写向旧master的数据，当master再次恢复的时候，会被作为一个slave挂到新的master 上去，自己的数据将会清空，重新从新的master 复制数据，这样就会导致数据缺失。

**总结：主库的数据还没有同步到从库，结果主库发生了故障，等从库升级为主库后，未同步的数据就丢失了。**

#### 数据丢失解决方案

数据丢失可以通过合理地配置参数 min-slaves-to-write 和 min-slaves-max-lag 解决，比如

- `min-slaves-to-write` 1
- `min-slaves-max-lag 10` 

如上两个配置：要求至少有 1 个 slave，数据复制和同步的延迟不能超过 10 秒，如果超过 1 个 slave，数据复制和同步的延迟都超过了 10 秒钟，那么这个时候，master 就不会再接收任何请求了。

###### 数据不一致

在主从异步复制过程，当从库因为网络延迟或执行复杂度高命令阻塞导致滞后执行同步命令，这样就会导致数据不一致

解决方案： 可以开发一个外部程序来监控主从库间的复制进度（`master_repl_offset` 和 `slave_repl_offset` ），通过监控 `master_repl_offset` 与`slave_repl_offset`差值得知复制进度，当复制进度不符合预期设置的Client不再从该从库读取数据。

## Redis分布式锁 *****************

[redis系列：基于redis的分布式锁 - 云枭zd - 博客园 (cnblogs.com)](https://www.cnblogs.com/fixzd/p/9479970.html)

### **最基础的版本1**

假设有两个客户端A和B，A获取到分布式的锁。A执行了一会，突然A所在的服务器断电了（或者其他什么的），也就是客户端A挂了。这时出现一个问题，这个锁一直存在，且不会被释放，其他客户端永远获取不到锁。

![image-20211118202121288](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211118202121288.png)

### **设置锁的过期时间**

1. 客户端A获取锁成功，过期时间30秒。
2. 客户端A在某个操作上阻塞了50秒。
3. 30秒时间到了，锁自动释放了。
4. 客户端B获取到了对应同一个资源的锁。
5. 客户端A从阻塞中恢复过来，释放掉了客户端B持有的锁。

![image-20211118202316538](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211118202316538.png)

==这时会有两个问题==

1. 过期时间如何保证大于业务执行时间?
2. 如何保证锁不会被误删除?

### **设置锁的value**

### **具有原子性的释放锁**

### **确保过期时间大于业务执行时间**

## Redis为什么这么快

- **基于内存**：Redis是使用内存存储的，没有磁盘IO上的开销，数据存在内存中，读写速度快。
- **单线程实现**（Redis6.0以前）：Redis使用单个线程处理请求，避免了多个线程之间线程切换和锁资源争用的开销。
- **IO多路复用模型**： Redis采用IO多路复用技术，Redis使用单线程来轮询描述符，将数据库的操作都转换成了事件，不在网络I/O上浪费太多的事件
- **高效的数据结构**： Redis为每种数据结构底层都做了优化，目的就是为了追求更快的速度。

## 布谷鸟过滤器

### 布谷鸟哈希

两个不同的hash算法将新来的元素映射到数组的两个位置。如果两个位置中有一个位置为空，那么就可以直接将元素放进去。**但是**如果两个位置都满了， ==随机踢走一个==，然后自己霸占这个位置。

```java
p1 = hash1(x) % l
p2 = hash2(x) % l
```

被挤走的那个元素去查看自己的另一个位置，如果为空，就自己挪过去。如果这个位置还被别人占了，那就再来一次【鸠占鹊巢】。

布谷鸟哈希会设置一个阈值，当连续占巢行为超出了某个阈值，就认为这个数组已经几乎满了。这时候就需要对它进行扩容，重新放置所有元素。

**挤兑循环**。比如两个不同的元素，hash 之后的两个位置正好相同，这时候它们一人一个位置没有问题。但是这时候来了第三个元素，它 hash 之后的位置也和它们一样，很明显，这时候会出现挤兑的循环。

**优化**

原始的布谷鸟哈希算法的平均空间利用率大概只有50%，改良的方案之一是增加hash函数，这样可以大大降低碰撞的概率。

另一种方案是在数组的每个位置上挂上多个多个座位，这样即使两个元素被hash在了同一个位置，也不必立即【鸠占鹊巢】，这种方案的空间利用率只有85%，但是查询效率会很高，同一个位置上的多个座位在内存空间上是连续的，可以有效利用 CPU 高速缓存。

### 布谷鸟过滤器

它也是一维数组，但是布谷鸟哈希会存储整个元素，而布谷鸟过滤器中只会存储元素的指纹信息（几个bit，类似于布隆过滤器）。

首先布谷鸟过滤器还是只会选用两个hash函数，但是每个位置可以放置多个座位。这两个hash函数选择的比较特殊，因为过滤器只能存储指纹信息。当这个位置上的指纹被挤兑之后，它需要计算出另一个对偶位置。而计算这个对偶位置是需要元素本身的。

```java
fp = fingerprint(x)
p1 = hash1(x) % l
p2 = hash2(x) % l
//我们知道了p1和x的指纹，是没有办法直接算出p2的    
```

**特殊的hash函数**

使得可以根据p1和元素指纹直接计算出p2，而不需要完整的x元素。

```java
fp = fingerprint(x)
p1 = hash(x)
p2 = p1 ^ hash(fp)  // 异或
p1 = p2 ^ hash(fp)  
```

**数据结构**

```java
type bucket [4]byte  // 一个桶，4个座位
type cuckoo_filter struct {
  buckets [size]bucket // 一维数组
  nums int  // 容纳的元素的个数
  kick_max  // 最大挤兑次数
}
```

```java
class Solution {
    public int integerBreak(int n) {
        int[] dp = new int[n + 1];
        for (int i = 2; i <= n; i++) {
            int curMax = 0;
            for (int j = 1; j < i; j++) {
                curMax = Math.max(curMax, Math.max(j * (i - j), j * dp[i - j]));
            }
            dp[i] = curMax;
        }
        return dp[n];
    }
}

```

## Redis 实现分布式锁

```c++
SETNX key value
```

`setnx`是  *SET if Not eXists*的简写。

如果不存在set成功返回int的1，这个key存在了返回0。

![image-20211129153218866](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129153218866.png)

```c++
SETEX key seconds value
```

将值`value`关联到`key`, 并将`key`的生存时间设为`seconds`

如果`key`已经存在，`setex`命令将覆写旧值。

![image-20211129153707935](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129153707935.png)

## Redis、DB数据一致性

一致性就是数据保持一致，在分布式系统中，可以理解为多个节点中数据的值是一致的。

- **强一致性**：这种一致性级别是最符合用户直觉的，它要求系统写入什么，读出来的也会是什么，用户体验好，但实现起来往往对系统的性能影响大
- **弱一致性**：这种一致性级别约束了系统在写入成功后，不承诺立即可以读到写入的值，也不承诺多久之后数据能够达到一致，但会尽可能地保证到某个时间级别（比如秒级别）后，数据能够达到一致状态
- **最终一致性**：最终一致性是弱一致性的一个特例，系统会保证在一定时间内，能够达到一个数据一致的状态。这里之所以将最终一致性单独提出来，是因为它是弱一致性中非常推崇的一种一致性模型，也是业界在大型分布式系统的数据一致性上比较推崇的模型

### 集中式redis缓存的三个经典的缓存模式

缓存可以提升性能、缓解数据库压力

- Cache-Aside Pattern
- Read-Through/Write through
- Write behind

#### Cache-Aside Pattern

旁路缓存模式，它的提出是为了尽可能地解决缓存与数据库的数据不一致问题。

**Cache-Aside的读流程**

![image-20211129160302076](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129160302076.png)

==读的时候，先读缓存，缓存命中的话，直接返回数据==

==缓存没有的命中的话，就去读数据库，从数据库取出数据，放入缓存后，同时返回响应。==

**Cache-Aside的写流程**

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129160826046.png" alt="image-20211129160826046" style="zoom:67%;" />

==更新的时候，先更新数据库，然后再删除缓存==

#### Read-Through/Write-Through(读写穿透)

Read/Write Through模式中，服务端把缓存作为主要数据存储。应用程序跟数据库缓存交互，都是通过抽象缓存层完成的。

**Read-Through读流程**

![image-20211129161149318](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129161149318.png)

==从缓存读取数据，读到直接返回==

==如果读取不到的话，从数据库加载，写入缓存后，再返回响应==

这个简要流程是不是跟Cache-Aside很像呢？

其实Read-Through就是多了一层Cache-Provider，流程如下：

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129161555560.png" alt="image-20211129161555560" style="zoom:65%;" />

**Write-Through写流程**

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129161859318.png" alt="image-20211129161859318" style="zoom:67%;" />

#### Write behind(异步缓存写入)

Write behind跟Read-Through/Write-Through有相似的地方，都是由Cache Provider来负责缓存和数据库的读写。它两又有个很大的不同：Read/Write Through是同步更新缓存和数据的，Write Behind则是只更新缓存，不直接更新数据库，通过批量异步的方式来更新数据库。

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129162118163.png" alt="image-20211129162118163" style="zoom:67%;" />

这种方式下，缓存和数据库的一致性不强，对一致性要求高的系统要谨慎使用。

> 但是它适合频繁写的场景，MySQL的InnoDB Buffer Pool机制就使用到这种模式。

### 三种模式的比较

Cache Aside 更新模式实现起来比较简单，但是需要维护两个数据存储：

- 一个是缓存（Cache）
- 一个是数据库（Repository）

Read/Write Through 的写模式需要维护一个数据存储（缓存），实现起来复杂

Write Behind  更新模式和Read/Write Through 更新模式类似，区别是Write Behind  更新模式的数据持久化操作是**异步的**，但是Read/Write Through 更新模式的数据持久化操作是**同步的**。

Write Behind Caching 的优点是直接**操作内存速度快**，多次操作可以合并持久化到数据库。缺点是数据可能会丢失，例如系统断电等。

### Cache-Aside的问题

==更新数据的时候，是删除缓存呢，还是更新缓存==

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129162957074.png" alt="image-20211129162957074" style="zoom:67%;" />

操作的次序如下：

> 线程A先发起一个写操作，第一步先更新数据库
> 线程B再发起一个写操作，第二步更新了数据库

**现在，由于网络等原因，线程B先更新了缓存, 线程A再更新缓存。**

这时候，缓存保存的是A的数据（老数据），数据库保存的是B的数据（新数据），数据不一致了，脏数据出现啦。如果是删除缓存取代更新缓存则不会出现这个脏数据问题。

> 更新缓存相对于删除缓存，还有两点劣势：

> 1 如果你写入的缓存值，是经过复杂计算才得到的话。 更新缓存频率高的话，就浪费性能啦。

> 2 在写多读少的情况下，数据很多时候还没被读取到，又被更新了，这也浪费了性能呢(实际上，写多的场景，用缓存也不是很划算了)

==双写的情况下，先操作数据库还是先操作缓存==

> 美团二面：Redis与MySQL双写一致性如何保证？

Cache-Aside缓存模式中，在写入请求的时候，为什么是先操作数据库呢？为什么不先操作缓存呢？
假设有A、B两个请求，请求A做更新操作，请求B做查询读取操作。

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129163727439.png" alt="image-20211129163727439" style="zoom:67%;" />

A、B两个请求的操作流程如下：

1. 线程A发起一个写操作，第一步del cache
2. 此时线程B发起一个读操作，cache miss
3. 线程B继续读DB，读出来一个老数据
4. 然后线程B把老数据设置入cache
5. 线程A写入DB最新的数据

> 缓存保存的是老数据，数据库保存的是新数据。因此，Cache-Aside缓存模式，选择了先操作数据库而不是先操作缓存。

### redis分布式缓存与数据库的数据一致性

**重要：\*缓存是通过牺牲强一致性来提高性能的\*。**

这是由**CAP理论**决定的。缓存系统适用的场景就是非强一致性的场景，它属于CAP中的AP。

**3种保证数据库与缓存的一致性的方案**

- 延迟双删策略
- 删除缓存重试机制
- 读取biglog异步删除缓存

#### 延迟双删

步骤：

1. 先删除缓存
2. 再更新数据库
3. 休眠一会（比如1秒），再次删除缓存

#### 删除缓存重试机制

如果第二步删除缓存失败？

删除失败就多删除几次呀,保证删除缓存成功呀~  所以可以引入删除缓存重试机制

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129182749426.png" alt="image-20211129182749426" style="zoom:67%;" />、

**删除缓存重试机制的大致步骤：**

- 写请求更新数据库
- 缓存因为某些原因，删除失败
- 把删除失败的key放到消息队列
- 消费消息队列的消息，获取要删除的key
- 重试删除缓存操作

#### 同步biglog异步删除缓存

[Redis与DB的数据一致性解决方案（史上最全） - 疯狂创客圈 - 博客园 (cnblogs.com)](https://www.cnblogs.com/crazymakercircle/p/14853622.html)

重试删除缓存机制会造成好多业务代码入侵。

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211129183757654.png" alt="image-20211129183757654" style="zoom:67%;" />