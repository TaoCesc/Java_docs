

## 索引

InnoDB的数据文件本身就是索引文件，在InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。**这种索引叫做聚集索引。**

![image-20211123202107258](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211123202107258.png)



## 连接查询

| 类型       | 解释       | 说明                                                         |
| ---------- | ---------- | ------------------------------------------------------------ |
| INNER JOIN | 内连接     | 关键字在表中存在至少一个匹配时返回行(交集)                   |
| LEFT JOIN  | (外)左连接 | 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL |
| RIGHT JOIN | (外)右连接 | 关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL |
| FULL JOIN  | (外)全连接 | 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.(并集) |

## InoDB和MyISAM的区别

#### 存储结构

MyISAM：每张表被存放在三个文件：frm-表格定义、MYD（MYData）-数据文件、MYI（MYIndex）-索引文件

InnoDB：所有的表都保存在同一个数据文件中，InnoDB表的大小只受限于操作系统文件的大小，一般为2GB

#### 存储空间

MyISAM：可以被压缩，存储空间较小

InnoDB： InnoDB的表需要更多的内存和存储，它会在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。

#### 事务

MyISAM：不支持。

Innodb：支持。

## 主键 聚集索引 非聚集索引

**对于 InnoDB 存储引擎来说，每张表都一定有个主键（Primary Key）**

如果在创建表时没有显式地定义主键，InnoDB 存储引擎会按如下方式选择或创建主键：

- 首先判断表中是否有非空的唯一索引（Unique NOT NULL），如果有，则该列即为主键
- 如果不符合上述条件，InnoDB 存储引擎自动创建一个 6 字节大小的指针 `_rowid` 作为主键

![image-20211205210026427](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205210026427.png)

**B+ 树索引并不能找到一个给定键值的具体“行”！B+ 树索引能找到的只是被查找数据行所在的“页”。然后数据库通过把页读入到内存，再在内存中进行查找，最后得到要查找的数据**。

### 逻辑存储结构

InnoDB存储引擎中，所有数据都被逻辑地存放在一个空间中，称之为**表空间（tablespace）**,也就是说我们常说的表，可以看作是InnoDB存储引擎逻辑结构的最高层。表空间又由**段（segment）、区（extent）、页（page）组成**

![image-20211205210333984](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205210333984.png)

**页是InnoDB磁盘管理的最小单位**，在InnoDB存储引擎中，默认每个页的大小为16KB。而页里面存放的东西就是一行一行的记录。

 **聚集索引（clustered inex）和辅助索引（secondary index）其实都是一种 B+ 树索引**。也就是说不管是聚集索引还是辅助索引，其内部都是 B+树，即高度平衡的，叶子节点存放着所有的数据。（需要注意的是，索引是存储引擎负责实现的，因此不是所有的存储引擎都支持聚簇索引）



### 主键和聚集索引的关系

InnoDB 存储引擎表是索引组织表结构，即表中数据都是按照主键顺序进行存放的。而**聚集索引就是按照每张表的主键构造一棵 B+ 树，同时叶子节点中存放的即为表中一行一行的数据**，所以聚集索引的叶子节点也被称为数据节点。

![image-20211205210701845](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205210701845.png)

**聚集索引能够在B+树索引的叶子节点上直接找到数据**。

可以这么说：在聚集索引中，**索引即数据，数据即索引**。另外，由于数据页只能按照一棵 B+ 树进行查找排序，或者说无法同时把数据行存放在两个不同的地方，所以**每张表只能拥有一个聚集索引**。



⭐ 主键是一种约束，这个约束用来强制表的实体完整性，一个表中只能有一个主键约束，并且主键约束中的列值必须是非空且唯一的。

而聚集索引它作为一种索引，其目的不是为了约束啥，而是为了对数据行进行排序以提高查询的效率，换句话说它决定的是数据库的物理存储结构。

### 聚集索引和辅助索引的关系

**辅助索引（Secondary Index）也称为 非聚集索引、二级索引**。其和聚集索引的最大区别就在于，辅助索引的叶子节点并不包含行记录的全部数据。

一行记录我们可以用 “主键 + 其他数据” 这样的组合来标识，聚集索引中的叶子节点存储的就是这一整个组合，而非聚集索引中的叶子节点只存储了这个组合中的主键

**辅助索引的叶子节点包含的是：每行数据的辅助索引键 + 该行数据对应的聚集索引键**。

当通过辅助索引来寻找数据时，InnoDB 存储引擎会先遍历辅助索引的 B+ 树，通过叶子节点获得某个辅助索引键对应的聚集索引键，然后再通过聚集索引来找到一个完整的行记录。



## 生产环境的B+树索引有多少层

> 一般是**2~3层**，可以存放大约**两千万行**的数据

页是 InnoDB 磁盘管理的最小单位，在 InnoDB 存储引擎中，默认每个页的大小为 16KB。而页里面存放的东西就是一行一行的记录。

假设一行数据的大小是 1k，那么一页就可以存放 16 行这样的数据。B+ 树的叶子节点存储真正的记录，而非叶子节点的存在是为了更快速的找到对应记录所在的叶子节点，所以**可以简单理解为非叶子节点存放的是键值 + 指针**。这里用指针来描其实述不是太准确，准确来说是**页的偏移量**，不过指针更好理解~

![image-20211205215025328](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205215025328.png)

==假设我们要从上图这棵 B+ 树种找到主键是 20 这行数据 `select * from table where id = 20;`==

首先找到 B+ 树的根节点，即存储的非叶子节点的页 page_number = 10，在该页上通过二分查找法以及指针定位到 id = 20 这行数据存在于 page_number = 12 这页上，然后同样的在这页上用二分查找即可快速定位 id = 20 这行记录。

假设 B+ 树只有两层，即一个根节点和若干个叶子节点，如下图：

![image-20211205215242276](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205215242276.png)

**B+ 树能够存放多少行数据，其实问的就是这棵 B+ 树的非叶子节点中存放的数据量**：

- **根节点指针数 \* 每个叶子节点存放的行记录数**

  

## 四大隔离级别

**事务**： 由一个有限的数据库操作序列构成，这些操作要么全部执行，要么全部不执行，是一个不可分割的工作单位。

**事务的特性**

- **原子性**： 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部都执行，要么都不执行。
- **一致性**： 指在事务开始之前和事务结束以后，数据不会被破坏，假如A账户给B账户转10块钱，不管成功与否，A和B的总金额是不变的。
- **隔离性**：多个事务并发访问时，事务之间是相互隔离的，一个事务不应该被其他事务干扰，多个并发事务之间要相互隔离。
- **持久性**：表示事务完成提交后，该事务对数据库所作的操作更改，将持久地保存在数据库中。

### 存在的问题

==**脏读 （dirty read）**==

假设现在有两个事务A、B：

- 假设现在A的余额是100，事务A正在准备查询Jay的余额
- 这时候，事务B先扣减Jay的余额，扣了10
- 最后A 读到的是扣减后的余额

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205221636818.png" alt="image-20211205221636818" style="zoom:50%;" />

==**不可重复读（unrepeatable read）**==

**不可重复读和脏读的区别是：脏读是读到未提交的数据，而不可重复读读到的却是已经提交的数据，但是其违反了事务一致性的要求。**

假设现在有两个事务A和B：

- 事务A先查询Jay的余额，查到结果是100
- 这时候事务B 对Jay的账户余额进行扣减，扣去10后，提交事务
- 事务A再去查询Jay的账户余额发现变成了90

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205221650361.png" alt="image-20211205221650361" style="zoom:60%;" />

==**幻读**==

幻读本质上是属于不可重复读的一种情况，区别在于，**不可重复读主要是针对数据的更新**（即事务的两次读取结果值不一样），而**幻读主要是针对数据的增加或减少**（即事务的两次读取结果返回的数量不一样）

假设现在有两个事务A、B：

- 事务A先查询id大于2的账户记录，得到记录id=2和id=3的两条记录
- 这时候，事务B开启，插入一条id=4的记录，并且提交了
- 事务A再去执行相同的查询，却得到了id=2,3,4的3条记录了。

<img src="https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205221719256.png" alt="image-20211205221719256" style="zoom:67%;" />

### 事务的隔离级别

- 读未提交（Read Uncommitted）
- 读已提交（Read Committed）
- 可重复读（Repeatable Read）
- 串行化（Serializable）

[一文彻底读懂MySQL事务的四大隔离级别 - 掘金 (juejin.cn)](https://juejin.cn/post/6844904115353436174)

![image-20211205223134119](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205223134119.png)

#### 读未提交

```sql
set session transaction isolation level read uncommitted;
//开启事务A
begin;
select * from account where id = 1;
//开启事务B
begin;
update account set balance = labance + 20 where id = 1;
//回到事务A
select * from account where id = 1;
//会看到 balance + 20
```

不能解决脏读问题。



#### 已提交读 (不可重复读)

```sql
set session transaction isolation level read committed;
//开启事务A
begin;
select * from account where id = 1;
//开启事务B
begin;
update account set balance = balance + 20 where id = 1;
//回到事务A，数据没有改变
select * from account where id = 1;
//到事务B执行事务
commit;
// 再回到事务A查询，发现数据变成balance + 20
```

不可重复读和脏读的区别是：脏读是读到未提交的数据，而不可重复读读到的却是已经提交的数据，但是其违反了事务一致性的要求。

可以解决脏读问题，但是不能解决**不可重复读**

#### 可重复读

![image-20211111203331561](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211111203331561.png)

#### 串行化（Serializable）

## 隔离级别的实现原理

- 读写锁
- 一致性快照， MVCC

MySql使用不同的锁策略(Locking Strategy)/MVCC来实现四种不同的隔离级别。

Repeatable Read、Read Committed的实现与MVCC有关， Read Uncommitted、串行化与锁有关。



**读未提交**

读未提交，采取的是读不加锁原理。

- 事务读不加锁，不阻塞其他事务的读和写
- 事务写阻塞其他事务写，但不阻塞其他事务读；

**串行化（Serializable)**

- 所有SELECT语句会隐式转化为`SELECT ... FOR SHARE`，即加共享锁。
- 读加共享锁，写加排他锁，读写互斥。如果有未提交的事务正在修改某些行，所有select这些行的语句都会阻塞。

### MVCC的实现原理

MVCC，中文叫**多版本并发控制**，它是通过读取历史版本的数据，来降低并发事务冲突，从而提高并发性能的一种机制。它的实现依赖于**隐式字段、undo日志、快照读&当前读、Read View**

#### **隐式字段**

对于InnoDB存储引擎，每一行记录都有两个隐藏列**DB_TRX_ID、DB_ROLL_PTR**，如果表中没有主键和非NULL唯一键时，则还会有第三个隐藏的主键列**DB_ROW_ID**。

- DB_TRX_ID，记录每一行最近一次修改（修改/更新）它的事务ID，大小为6字节；
- DB_ROLL_PTR，这个隐藏列就相当于一个指针，指向回滚段的undo日志，大小为7字节；
- DB_ROW_ID，单调递增的行ID，大小为6字节；

#### undo日志

- 事务未提交的时候，修改数据的镜像（修改前的旧版本），存到undo日志里。以便事务回滚时，恢复旧版本数据，撤销未提交事务数据对数据库的影响。

- undo日志是逻辑日志。可以这样认为，当delete一条记录时，undo log中会记录一条对应的insert记录，当update一条记录时，它记录一条对应相反的update记录。

- 存储undo日志的地方，就是**回滚段**。

多个事务并行操作某一行数据时，不同事务对该行数据的修改会产生多个版本，然后通过回滚指针（DB_ROLL_PTR）连一条**Undo日志链**。

==举例==

- 假设表accout现在只有一条记录，插入该该记录的事务Id为100
- 如果事务B（事务Id为200），对id=1的该行记录进行更新，把balance值修改为90

![image-20211111223840302](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211111223840302.png)

#### 快照读&当前读

**快照读：**

读取的是记录数据的可见版本（有旧的版本），不加锁,普通的select语句都是快照读,如：

```sql
select * from account where id > 2;
```

**当前读：**

读取的是记录数据的最新版本，显示加锁的都是当前读

```sql
select * from account where id>2 lock in share mode;
select * from  account where id>2 for update;
```

#### Read View

- Read View就是事务执行**快照读**时，产生的读视图。
- 事务执行快照读时，会生成数据库系统当前的一个快照，记录当前系统中还有哪些活跃的读写事务，把它们放到一个列表里。
- Read View主要是用来做可见性判断的，即判断当前事务可见哪个版本的数据



- **`m_ids`：生成 ReadView 时有哪些事务在执行但是还没提交的（称为 ”活跃事务“），这些活跃事务的 id 就存在这个字段里**
- **min_trx_id:m_ids事务列表中，最小的事务ID**
- **`max_trx_id`：**生成 ReadView 时 InnoDB 将分配给下一个事务的 ID 的值（事务 ID 是递增分配的，越后面申请的事务 ID 越大）
- **`creator_trx_id`：当前创建 ReadView 事务的 ID**

假设表中已经被之前的事务 A（id = 100）插入了一条行记录（id = 1, username = "Jack", age = 18），如图所示：

![image-20211205220131733](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205220131733.png)

接下来，有两个事务B（id = 200） 和 C（id = 300）过来**并发执行**，事务B想要更新（update）这行id = 1的记录，而事务C想要查询这行数据，这两个事务都执行了相应的操作但是还没有进行提交：

如果现在事务B开启了一个ReadView，在这个ReadView里面：

- `m_ids` 就包含了当前的活跃事务的 id，即事务 B 和事务 C 这两个 id，200 和 300
- `min_trx_id` 就是 200
- `max_trx_id` 是下一个能够分配的事务的 id，那就是 301
- `creator_trx_id` 是当前创建 ReadView 事务 B 的 id 200

此时，事务B进行查询，会**把这行记录的隐藏字段 `trx_id` 和 ReadView 的 `min_trx_id` 进行下判断**，此时，发现 trx_id 是 100，小于 ReadView 里的 `min_trx_id`（200），这说明在事务 B 开始之前，修改这行记录的事务 A 已经提交了，所以**开始于事务 A 提交之后的事务 B、是可以查到事务 A 对这行记录的更新的**。

```sql
row.trx_id < ReadView.min_trx_id
```

事务 C 过来修改这行记录，把 age = 18 改成了 age = 20，所以这行记录的 `trx_id` 就变成了 300，同时 `roll_pointer` 指向了事务 C 修改之前生成的 undo log：

![image-20211205220611722](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205220611722.png)

这时事务B再次进行查询操作，会发现**这行记录的 `trx_id`（300）大于 ReadView 的 `min_trx_id`（200），并且小于 `max_trx_id`（301）**。

```sql
row.trx_id > ReadView.min_trx_id && row.trx_id < max_trx_id
```

***这就说明***  更新这行记录的事务很有可能也存在于 ReadView 的 m_ids（活跃事务）中。所以事务 B 会去判断下 ReadView 的 m_ids 里面是否存在 `trx_id = 300` 的事务，显然是存在的，这就表示这个 id = 300 的事务是跟自己（事务 B）在同一时间段并发执行的事务，也就说明这行 age = 20 的记录事务 B 是不能查询到的。







## MySQL的锁

### 表锁与行锁

表锁（Table Lock），就是会锁定整张表，表锁是开销最小的策略（因为粒度比较大）

行锁（Row Lock），也称为记录锁，**仅仅锁住一行**，需要的注意的是，MySQL 服务器层并没有实现行锁机制，**行级锁只在存储引擎层实现** ！！！

**对于 InnoDB 引擎来说，读锁和写锁可以加在表上，也可以加在行上**。

对于并发读和并发写的问题，可以通过实现一个由两种类型的锁组成的锁系统来解决。这两种类型的锁通常被称为 **共享锁（Shared Lock，S Lock）** 和 **排他锁（Exclusive Lock，X Lock）**，也叫 **读锁（readlock）** 和 **写锁（write lock）**：

- 共享锁 / 读锁：允许持锁事务读取（`select`)数据。
- 排他锁 / 写锁：允许事务删除（`delete`）或更新（`update`）数据

**读锁是共享的**，或者说是相互不阻塞的。多个事务在同一时刻可以同时读取同一个资源，而互不干扰。**写锁是排他的**，也就是说一个写锁会阻塞其他的读锁和写锁，这样就能确保在给定的时间里，只有一个事务能执行写入，并防止其他用户读取正在写入的同一资源。

### 意向锁

InnoDB 存储引擎支持 **多粒度（granular）锁定**，就是说**允许事务在行级上的锁和表级上的锁同时存在**。

意向锁是一个**表级锁**，其作用就是指明接下来的事务将会用到哪种锁。

有两种意向锁：

- **意向共享锁（IS Lock）**：当事务想要获得一张表中某几行的共享锁行级锁）时，InnoDB 存储引擎会自动地先获取该表的意向共享锁（表级锁）
- **意向排他锁（IX Lock）**：当事务想要获得一张表中某几行的排他锁（行级锁）时，InnoDB 存储引擎会自动地先获取该表的意向排他锁（表级锁）

### 如何加锁

对于 InnoDB 来说，随时都可以加锁，但是并非随时都可以解锁。具体来说，InnoDB 采用的是**两阶段锁定协议（two-phase locking protocol）**：即在事务执行过程中，随时都可以执行加锁操作，但是**只有在事务执行 COMMIT 或者 ROLLBACK 的时候才会释放锁**，并且所有的锁是在同一时刻被释放。

如何加表级锁：

1）隐式锁定：对于常见的 DDL 语句（如 `ALTER`、`CREATE` 等），InnoDB 会自动给相应的表加表级锁

2）显示锁定：在执行 SQL 语句时，也可以明确显示指定对某个表进行加锁（`lock table user read(write)`）

如何加行级锁：

1）对于常见的 DML 语句（如 `UPDATE`、`DELETE` 和 `INSERT` ），InnoDB 会自动给相应的记录行加写锁

2）默认情况下对于普通 `SELECT` 语句，InnoDB 不会加任何锁，但是在 Serializable 隔离级别下会加行级读锁

上面两种是隐式锁定，InnoDB 也支持通过特定的语句进行显式锁定，不过这些语句并不属于 SQL 规范：

3）`SELECT * FROM table_name WHERE ... FOR UPDATE`，加行级写锁

4）`SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE`，加行级读锁

# InnoDB 存储引擎中行锁的三种算法

### 记录锁（Record Locks）

记录锁是最简单的行锁，**仅仅锁住一行**。如：`SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE`

c1 为 10 的记录行会被锁住。

需要注意的是：`id` 列必须为`唯一索引列`或`主键列`，否则上述语句加的锁就会变成`临键锁`。

同时查询语句必须为`精准匹配`（`=`），不能为 `>`、`<`、`like`等，否则也会退化成`临键锁`

- 记录锁**永远都是加在索引上**的，即使一个表没有索引，InnoDB也会隐式的创建一个索引，并使用这个索引实施记录锁。
- 会阻塞其他事务对其插入、更新、删除

在通过 `主键索引` 与 `唯一索引` 对数据行进行 UPDATE 操作时，也会对该行数据加`记录锁`：

```sql
-- id 列为主键列或唯一索引列
update set age = 50 where id = 1;
```



### 间隙锁（Gap Lock）

- 间隙锁是一种**加在两个索引之间的锁，或者加在第一个索引之前，或最后一个索引之后的间隙**。

- 使用**间隙锁锁住的是一个区间**，而不仅仅是这个区间中的每一条数据。

- 间隙锁只阻止其他事务插入到间隙中，他们不阻止其他事务在同一个间隙上获得间隙锁，所以 gap x lock 和 gap s lock 有相同的作用。

```sql
select * from table where id between 1 and 10 for update;
```

即所有在`（1，10）`区间内的记录行都会被锁住，所有id 为 2、3、4、5、6、7、8、9 的数据行的插入会被阻塞，**但是 1 和 10 两条记录行并不会被锁住。**



### 临键锁（Next-Key Lock）

**其主要目的是为了解决幻读问题**。

 每个数据行上的`非唯一索引列`上都会存在一把**临键锁**，当某个事务持有该数据行的**临键锁**时，会锁住一段**左开右闭区间**的数据。也就是说它是**包含当前被操作的索引记录**的。

`InnoDB` 中`行级锁`是基于索引实现的，**临键锁**只与`非唯一索引列`有关，在`唯一索引列`（包括`主键列`）上不存在**临键锁**。

例如一个索引有 10，11，13 和 20 这四个值，分别对这个 4 个索引进行加锁操作，那么这四个操作分别对应的 Next-Key Lock 锁住的区间是：

- `(-∞, 10]`
- `(10, 11]`
- `(11, 13]`
- `(13, 20]`
- `(20, +∞]`

在 InnoDB 默认的隔离级别 REPEATABLE-READ 下，行锁默认使用的算法就是 Next-Key Lock。但是，**如果操作的索引是唯一索引或主键，InnoDB 会对 Next-Key Lock 进行优化，将其降级为 Record Lock**，即仅锁住索引本身，而不是范围。

示例：

```sql
CREATE TABLE `test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `class` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `index_class` (`class`) USING BTREE COMMENT '非唯一索引'
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
```

![image-20211205204330893](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211205204330893.png)

开启一个事务1：

```sql
select * from test where class = 3 for update;
```

在这种情况下，InnoDB会加上三种行锁（`select * ... from update` 加的是行级写锁即 X 锁）

1. 给主键索引id = 105 加上Record Lock记录锁
2. 对于非唯一索引class = 3，其加上的是Next-Key Lock临键锁，锁定的范围是`(1,3]`
3. InnoDB存储引擎还会为非唯一索引class的**下一个键值**加上Gap Lock间隙锁（表中class =3 的下一个键值是6），所以还有class索引为`(3,6)`的间隙锁

最终，**InnoDB锁定的class索引范围为`(1,6)`**



## 主从复制

主从复制是指将主数据库的DDL和DML操作通过二进制日志传到从数据库上，然后再从数据库上对这些日志进行重新执行，从而使从数据库和主数据库的数据保持一致。

#### 主从复制的原理

- MySql主库在事务提交时会把数据变更作为事件记录在二进制日志Binlog中；
- 主库推送二进制日志文件Binlog中的事件到从库的中继日志Relay Log中，之后从库根据中继日志重做数据变更操作，通过逻辑复制来达到主库和从库数据的一致性。
- MySql通过三个线程完成主从库间的数据复制，其中BinLog Dump线程跑在主库上，I/O线程和SQL线程

跑在从库上。

- 当在从库上启动复制时，首先创建I/O线程连接主库，主库随后创建Binlog Dump线程读取数据库事件并发送给I/O线程，I/O线程获取到事件数据后更新到从库的中继日志Relay Log中去，之后从库上的SQL线程读取中继日志Relay Log中更新的数据库事件并应用。

![image-20211112202602539](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211112202602539.png)

## 临时表

MySql在执行SQL语句的时候会临时创建一些存储中间结果集的表，这种表被称为**临时表**，临时表只对当前连接可见，在连接关闭后，临时表会被删除并释放空间。

临时表主要分为内存临时表和磁盘临时表两种。内存临时表使用的是MEMORY存储引擎，磁盘临时表使用的是MyISAM存储引擎。

会产生临时表的情况：

- From中的子查询
- Distinct 查询并加上Order by
- Order by 和 Group by 的子句不一样时
- 使用Union查询

## 慢查询

#### 慢查询日志

1. MySql的慢查询日志是MySql提供的一种日志记录，它用来记录MySql中查询时间超过设置阈值（long_query_time)的语句，记录到慢查询日志中。
2. long_query_time的默认值是10

#### 开启慢查询日志

**默认情况下，MySql没有开启慢查询日志**。需要手动开启。

```sql
-- 查看慢查询日志是否开启
show variables like '%slow_query_log%'
-- 开启慢查询日志，只对当前数据库生效，并且重启数据库后失效
set global slow_query_log = 1;
-- 查看慢查询日志的阈值，默认为10s
shown variables like '%long_query_time%';
-- 设置阈值
set long_query_time = 3;

```



#### 8.7.3 对慢查询优化

- 分析语句的执行情况，查询SQL语句的索引是否命中
- 优化数据库的结构，将字段很多的表分解成多个表，或者考虑建立中间表
- 优化LIMIT 分页

## SQL的执行顺序

```sql
SELECT DISTINCT     select_list FROM     left_table LEFT JOIN     right_table ON join_condition WHERE     where_condition GROUP BY     group_by_list HAVING     having_condition ORDER BY     order_by_condition
```

![image-20211113202004093](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211113202004093.png)

## 索引失效

### 使用！=或者<>

```sql
SELECT * FROM `user` WHERE `name` != '冰峰';
```

我们给name字段建立了索引，但是如果!= 或者 <> 这种都会导致索引失效，进行全表扫描

### 类型不一致

```sql
SELECT * FROM `user` WHERE height= 175;
```

height 类型为 varchar，但是这里使用int类型，类型不一致会导致索引失效。

### 函数导致的索引失效

```sql
select * from 'user' where date(create_time) = '2020-09-03';
```

### 运算符

```sql
SELECT * FROM 'user' where age - 1 = 20;
```

### OR

```sql
SELECT * FROM 'user' WHERE 'name' = 'zhangsan' OR height = '175';
```

### 模糊搜索

```sql
SELECT * FROM 'user' WHERE 'name' LIKE '%kk';
```

### NOT IN、NOT EXISTS

```sql
SELECT s.* FROM `user` s WHERE NOT EXISTS (SELECT * FROM `user` u WHERE u.name = s.`name` AND u.`name` = '冰峰')
```

### IS NULL不走索引，IS NOT NULL走索引