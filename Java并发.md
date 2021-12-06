

- 同步(Synchronous)和异步(Asynchronous)
- 并发(Concurrency)和并行(Parallelism)



# Happen-Before规则

- 程序顺序原则：一个线程内保证语义的串行性
- volatile规则：volatile变量的写先于读发生，这保证了volatile变量的可见性
- 锁规则：解锁（unlock）必然发生在随后的加锁（lock）前
- 传递性：A先于B，B先于C，那么A必然先于C
- 线程的start（）方法先于它的每一个动作
- 线程的所有操作先于线程的终结（Thread.join()）
- 线程的中断（interrupt()）先于被中断线程的代码
- 对象的构造函数的执行、结束先于finalize()方法

# Join()

join()方法的作用，是等待这个线程结束

t.join()方法**阻塞调用此方法的线程**(calling thread)进入 **TIMED_WAITING** 状态，**直到线程t完成，此线程再继续**；

通常用于在main()主线程内，等待其它线程完成再结束main()主线程。

```java
public class test implements Runnable{
    private  String name;
    public test(String name){
        this.name = name;
    }
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new test("one"));
        Thread thread2 = new Thread(new test("two"));
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println("Main thread is finished");
    }


    @Override
    public void run() {
        System.out.printf("%s begins: %s\n", name, new Date());
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.printf("%s has finished: %s\n", name, new Date());
    }
}
```

有join()方法的结果：

![image-20210728205128308](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210728205128308.png)

没有join()的结果：

![image-20210728205205894](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210728205205894.png)

# wait()、notify()详解

1） wait()、notify/notifyAll() 方法是Object的本地final方法，无法被重写。

2） wait()使当前线程阻塞，前提是必须先获得锁，一般配合**synchronized**关键字使用

3） 当线程执行wait()方法时候，会释放当前的锁，然后让出CPU，进入等待状态。

4） notify 和wait 的顺序不能错，如果A线程先执行notify方法，B线程在执行wait方法，那么B线程是无法被唤醒的。



# JAVA中CAS详解

[(4条消息) Java中CAS详解_jayxu无捷之径的博客-CSDN博客_cas](https://blog.csdn.net/ls5718/article/details/52563959)

在JDK 5之前Java语言是靠synchronized关键字保证同步的，这会导致有锁

锁机制存在以下问题：

- 在多线程竞争下，加锁、释放锁会导致比较多的调度延时，引起性能问题。
- 一个线程持有锁会导致其它所有需要此锁的线程挂起。
- 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。

### 什么是CAS

**compare and swap**的缩写

CAS操作包含3个操作数—— **内存位置（V）、预期原值（A）和新值（B）**。如果内存位置的值与预期原值相匹配，那么处理器自动将该位置的值更新为新值；否则，处理器不做任何操作。

### CAS存在的问题

- 1）ABA问题

如果一个值原来是A，变成了B，又变成了A。ABA问题的解决思路就是**使用版本号**。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。

AtomicStampedReference类

- 2）循环时间开销大

- 3）**只能保证一个共享变量的原子操作**。

# 红黑树

[(4条消息) 面试常问：什么是红黑树？_Mr.TS的博客-CSDN博客_红黑树](https://blog.csdn.net/qq_36610462/article/details/83277524)

**二叉查找树（BST）具备什么特性呢？**

1.左子树上所有结点的值均小于或等于它的根结点的值。

2.右子树上所有结点的值均大于或等于它的根结点的值。

3.左、右子树也分别为二叉排序树。

###########################################

1.节点是红色或黑色。

2.根节点是黑色。

3.每个叶子节点都是黑色的空节点（NIL节点）。

4 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)

5.从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

怎么保证一棵红黑树始终是红黑树呢

1. **变色**  2. **旋转**

- 变色

为了重新符合红黑树的规则，尝试把红色节点变为黑色，或者把黑色节点变为红色。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210808154004415.png" alt="image-20210808154004415" style="zoom:80%;" />

凭空多出的黑色节点打破了规则5，所以发生连锁反应，需要继续把节点25从黑色变成红色：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210808154035274.png" alt="image-20210808154035274" style="zoom:80%;" />

节点25和节点27又形成了两个连续的红色节点，需要继续把节点27从红色变成黑色：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210808154059298.png" alt="image-20210808154059298" style="zoom:80%;" />

- **左旋转：**

逆时针旋转红黑树的两个节点，使得父节点被自己的右孩子取代，而自己成为自己的左孩子。

- **右旋转：**

顺时针旋转红黑树的两个节点，使得父节点被自己的左孩子取代，而自己成为自己的右孩子。



# ConcurrentHashMap

[(4条消息) ConcurrentHashMap源码解析（jdk1.8）_programmer_at的专栏-CSDN博客_concurrenthashmap jdk1.8](https://blog.csdn.net/programmer_at/article/details/79715177)

## 1. 原理解析

CAS + synchronized 保证并发更新的安全

底层：**数组 + 链表 + 红黑树** 实现

### 1.1重要成员变量

- table: 默认为null，初始化发生在第一次插入操作，默认为16的数组，扩容时大小总是2的幂次方。
- nextTable：默认为null，扩容时生成的数组，大小为原数组的两倍。
- sizectl：默认为0，用来控制table的初始化和扩容操作

​     **-1**代表table正在初始化

​    **-N**表示有N-1个线程正在扩容操作

- Node：保存key、value以及key的hash值

  其中value和next都用volatile修饰，保证并发的可见性

  ```java
  static class Node<K,V> implements Map.Entry<K,V> {
          final int hash;
          final K key;
          volatile V val;
          volatile Node<K,V> next;
  ```

- ForwardingNode: 一个特殊的Node节点，hash值为-1，其中存储nextTable 的引用。只有table发生扩容的时候，ForwardNode才会发挥作用。

### 1.2 实例初始化

实例化ConcurrentHashMap时倘若声明了table的容量，在初始化时会根据参数调整table大小，==**确保table的大小总是2的幂次方**==。默认的table大小为16.

### 1.3 put操作

#### 1.3.1 put过程描述

假设table已经初始化完成，put操作采用==CAS+synchronized==实现并发插入或更新操作：

- 当前bucket为空时，使用CAS操作，将Node放入对应的bucket中。

- 出现hash冲突时，则采用synchronized关键字。如果当前hash对应的节点是链表的头节点，遍历链表，若找到对应的node节点，则修改node节点的val，否则在链表末尾添加node节点。

- 倘若当前map正在扩容`f.hash == MOVED`， 则跟其他线程一起进行扩容

#### 1.3.2 hash算法

```java
static final int HASH_BITS = 0x7fffffff;
static final int spread(int h) {return (h ^ (h>>>16)) & HASH_BITS;}
```

#### 1.3.3 定位索引

int index = (n - 1) & hash

#### 1.3.4 获取table对应的索引元素f

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
```

### 1.4 \****table扩容 

什么时候会触发扩容

- 如果新增节点之后，所在的链表的元素个数大于等于8，则会调用`treeifyBin`把链表转换为**红黑树**。
- 在转换结构时，若tab的长度小于`MIN_TREEIFY_CAPACITY`，默认值为64，则会将数组长度扩大到原来的两倍，并触发`transfer`，重新调整节点位置。（只有当`tab.length >= 64, ConcurrentHashMap`才会使用红黑树。）

- 新增节点后，`addCount`统计tab中的节点个数大于阈值（sizeCtl），会触发`transfer`，重新调整节点位置。

#### 1.4.1 addCount

#### 1.4.2 trreify

#### 1.4.3 transfer

### 1.5 get操作

读取操作，不需要同步控制

1） 空tab，直接返回null

2）计算hash值，找到相应的bucket位置，为node节点返回，否则返回null

```java
public V get(Object key) {
  Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

### 1.6 删除元素

#### 1.6.1 清空map：clear

遍历tab中每一个bucket，
\1. 当前bucket正在扩容，先协助扩容
\2. 给当前bucket上锁，删除元素
\3. 更新map的size

# CopyOnWriteArrayList

[CopyOnWriteArrayList - 简书 (jianshu.com)](https://www.jianshu.com/p/9b6a4d0b94ac)

在很多应用场景中，**读操作可能会远远大于写操作**。由于读操作根本不会修改原有的数据，因此如果每次读取都进行加锁操作，其实是一种资源浪费。我们应该允许多个线程同时访问 List 的内部数据，毕竟读操作是线程安全的。

写入也不会阻塞读取操作，只有写入和写入之间需要进行同步等待，读操作的性能得到大幅度提升。

## 如何做到的

`CopyOnWriteArrayList` 类的所有可变操作（add，set等等）都是通过创建底层数组的新副本来实现的。当List需要被修改的时候，并不直接修改原有数对象。而是对原有数据进行一次拷贝，将修改的内容写入副本中。

就是对一块内存进行修改时，**不直接在原有内存块中进行写操作**，而是将**内存拷贝一份，在新的内存中进行写操作**，写完之后，再将原来**指向的内存指针指到新的内存**，原来的内存就可以被回收。

## 写入操作的实现

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();  // 加锁
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);  // 拷贝新数组
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();  // 释放锁
        }
    }
```

# ConcurrentLinkedQueue

[Java并发编程之ConcurrentLinkedQueue详解_Stay Hungry, Stay Foolish-CSDN博客_concurrentlinkedqueue](https://blog.csdn.net/qq_38293564/article/details/80798310)

# LinkedBlockingQueue

[(1条消息) 【细谈Java并发】谈谈LinkedBlockingQueue_颤抖吧腿子-CSDN博客_linkedblockingqueue](https://blog.csdn.net/tonywu1992/article/details/83419448)

## 简介

LinkedBlockingQueue不同于ArrayBlockingQueue，它如果不指定容量，默认为`Integer.MAX_VALUE`，也就是无界队列。所以为了避免队列过大造成机器负载或者内存爆满的情况出现，我们在使用的时候建议手动传一个队列的大小.

## 源码分析

###  属性

```java
/**
 * 节点类，用于存储数据
 */
static class Node<E> {
    E item;
    Node<E> next;

    Node(E x) { item = x; }
}

/** 阻塞队列的大小，默认为Integer.MAX_VALUE */
private final int capacity;

/** 当前阻塞队列中的元素个数 */
private final AtomicInteger count = new AtomicInteger();

/**
 * 阻塞队列的头结点
 */
transient Node<E> head;

/**
 * 阻塞队列的尾节点
 */
private transient Node<E> last;

/** 获取并移除元素时使用的锁，如take, poll, etc */
private final ReentrantLock takeLock = new ReentrantLock();

/** notEmpty条件对象，当队列没有数据时用于挂起执行删除的线程 */
private final Condition notEmpty = takeLock.newCondition();

/** 添加元素时使用的锁如 put, offer, etc */
private final ReentrantLock putLock = new ReentrantLock();

/** notFull条件对象，当队列数据已满时用于挂起执行添加的线程 */
private final Condition notFull = putLock.newCondition();
```

每个添加到LinkedBlockingQueue队列中的数据都将被**封装成Node节点**，添加的链表队列中，其中`head`和`last`分别指向队列的**头结点**和**尾结点**。

LinkedBlockingQueue使用takeLock 和 putLock堆并发就行控制，也就是说，**添加和删除操作并不是互斥**的

### 构造函数

```java
public LinkedBlockingQueue() {
    // 默认大小为Integer.MAX_VALUE
    this(Integer.MAX_VALUE);
}

public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    last = head = new Node<E>(null);
}

public LinkedBlockingQueue(Collection<? extends E> c) {
    this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        int n = 0;
        for (E e : c) {
            if (e == null)
                throw new NullPointerException();
            if (n == capacity)
                throw new IllegalStateException("Queue full");
            enqueue(new Node<E>(e));
            ++n;
        }
        count.set(n);
    } finally {
        putLock.unlock();
    }
}
```

第二种构造函数可以指定队列的大小。

