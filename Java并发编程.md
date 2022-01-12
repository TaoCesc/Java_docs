## 线程状态转换

![image-20220111214527184](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220111214527184.png)

**新建（New）**

创建后尚未启动。

**可运行（Runnable）**

可能正在运行，也可能正在等待CPU时间片。

包含了操作系统线程状态中的运行（Running）和就绪（Ready）。

**阻塞（Blocking）**

这个状态下，是在多个线程有同步操作的场景，比如正在等待另一个线程的`synchronized`块的执行释放，或者可重入的synchronized块里别人调用wait（）方法，也就是线程在等待进入临界区。

阻塞可以分为：等待阻塞，同步阻塞，其他阻塞

**无限期等待（Waiting）**

等待其他线程显式地唤醒，否则不会被分配CPU时间片。

| 进入方法                                   | 退出方法                             |
| :----------------------------------------- | :----------------------------------- |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | -                                    |

**限期等待（Timed Waiting）**

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。





调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用 “**使一个线程睡眠**” 进行描述。

调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用 “**挂起一个线程**” 进行描述。

**睡眠和挂起**是用来描述**行为**，而**阻塞**和等待用来描述**状态**。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

**死亡**

线程因为run方法正常退出而自然死亡

因为一个没有捕获地异常终止了run方法而意外死亡

## 实现多线程的方式

- 实现Runnable接口
- 实现Callable接口
- 继承Thread类
- 实现Runnbale和Callable接口的类只能当作一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

### 实现Runnbale接口

需要实现`run()`方法

通过Thread调用start（）方法来启动线程。

```java
public class MyRunnbale implements Runnable{
    @Override
    public void run(){
        // ...
    }
}
```

```java
public static void main(String[] args){
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

### 实现Callable接口

与Runnable相比，Callable可以有返回值，返回值通过FutureTask进行封装

```java
public class MyCallable implements Callable<Integer>{
    public Integer call(){
        return 123;
    }
}
```

```java
public static void main(String[] args){
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thead.start();
    System.out.println(fc.get());
}
```

### 继承Thread类

同样需要实现run()方法，因为Thread类也实现了Runnable接口

```java
public class MyThead extends Thread{
    @Override
    public void run(){
        //...
    }
}
```

```java
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

## 基础线程机制

### Executor（线程池）

Executor管理多个异步任务的执行，而无需显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种Executor：

- CachedThreadPool：一个任务创建一个线程。
- FixedThreadPool：所有任务只能使用固定大小的线程。
- SingleThreadExecutor：相当于为1的FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

### Daemon（守护线程）

- User Thread(用户线程)
- Daemon Thread（守护线程）

用户线程即**运行在前台的线程**，而守护线程是**运行在后台的线程**。守护线程作用是为其他前台线程的运行提供便利服务，而且仅在普通、非守护线程仍然运行时才**需要**，比如**垃圾回收线程**就是一个守护线程。

当 JVM 检测仅剩一个守护线程，而用户线程都已经退出运行时，JVM 就会退出。

用户可以用Thread的`setDaemon(true)`方法设置当前线程为守护线程。

==注意事项==：

1. `setDaemon(true)`必须在调用线程的start（）方法之前设置，否则会出现`IllegalThreadStateException`异常。
2. 在守护线程中产生的新线程也是守护线程。
3. 守护线程中**不要执行逻辑操作或读写操作**。

### Sleep()

`Thread.sleep(millisec)`方法会休眠**当前正在执行的线程**。

### yield()

对静态方法`Thread.yield()`的调用声明了**当前线程已经完成了生命周期中最重要的部分**，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

### 线程阻塞

线程可以阻塞于四种状态：

- 当线程执行Thread.sleep()时，它一直阻塞到指定的毫秒时间之后，或者阻塞被另一个线程打断。
- 当线程碰到一条wait（）语句时，它会一直阻塞到接到通知`notify()`、被终端或经过了指定毫秒时间为止。
- 线程阻塞与不同 I/O 的方式有多种。常见的一种方式是 InputStream 的 read() 方法，该方法一直阻塞到从流中读取一个字节的数据为止，它可以无限阻塞，因此不能指定超时时间。
- 线程也可以阻塞等待获取某个对象锁的排他性访问权限（即等待获得 synchronized 语句必须的锁时阻塞）。

## 中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

### InterruptedException

通过调用一个线程的`interrupt()`来中断该线程，如果该线程处于**阻塞、限期等待或者无限期等待**状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
```

### interrupted()

## volatile

Java线程安全（volatile & synchronized）

- volatile **不能保证线程安全**而 synchronized 可以保证线程安全。

- volatile 只能保证被其修 饰变量的**内存可见性**，但如果对该变量执行的是非原子操作线程依旧是不安全的。
- synchronized 既可以保证其修饰范围内存可见性和操作的原子性，所以 synchronized 是线程安全的

保证了共享变量的可见性，可见性 就是在一个线程修改一个共享变量的时候，另一个线程可以看到修改后的值

线程对 volatile 变量的修改会立刻被其他 线程所感知，即不会出现数据脏读的现象，从而保证数据的“可见性”

### 内存屏障指令

内存屏障指令是一组处理指令，用来限制内存操作的顺序。

volatile变量写，汇编指令会多出`Lock前缀`:

- 将当前处理器缓存行的数据写回主存。
- 令其他CPU里缓存该内存地址的数据无效。

**针对编译器重排序**：JMM针对编译器指定了`volatile重排序规则表`:

![image-20220112140511744](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220112140511744.png)

**针对处理器重排序**：编译器在生成字节码指令时，通过在指令序列中`插入内存屏障指令`来禁止特定类型的处理器重排序： **volatile底层通过内存屏障指令实现**

![image-20220112140739982](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220112140739982.png)

- 在每个**volatile变量写**操作之前插入StoreStore屏障，之后插入StoreLoad屏障：
  - 之前插入StoreStore屏障：禁止volatile写之前的写操作与其重排序，保证之前的所有写操作都写回主存，对volatile写可见。
  - 之后插入StoreLoad屏障：禁止volatile写之后的读写操作与其重排序，实现volatile写结果对后续操作可见；
- 在每个volatile变量读操作之后，接连插入LoadLoad屏障，LoadStore屏障；
  - 插入LoadLoad屏障：禁止**volatile**变量读之后的读操作与其重排序；
  - 插入LoadStore屏障：禁止**volatile**变量读之后的写操作与其重排序；
  - 通过插入两次内存屏障，实现volatile读结果对后续操作可见；

**volatile**用来修饰共享变量（成员变量，static变量）表明：

- volatile变量写：当写一个**volatile**变量时，JMM会把所有线程本地内存的对应变量副本刷新回主存；

- - **volatile**写和解锁内存语义相同；

- volatile变量读：当读一个**volatile**变量时，JMM会设置该线程的volatile变量副本（本地内存中）无效，线程只能从主存中读取该变量；

- - 保证了volatile变量读，总能看见对该volatile变量最后的修改；
  - **volatile**变量读和加锁内存语义相同；

通过上述机制，**volatile**保证共享变量一旦被修改，新值对所有线程可见；

## 进程、线程的区别

**进程** 

一个在内存中运行的应用程序。每个进程都有自己独立的一块内存空间，一个进程可以有多个线程，比如在Windows系统中，一个运行的xx.exe就是一个进程。

**线程**

进程中的一个执行任务（控制单元），负责当前进程中程序的执行。一个进程至少有一个线程，一个进程可以运行多个线程，多个线程可共享数据。

与进程不同的是同类的多个线程共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

==根本区别==：进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位

## 死锁出现的原因和如何避免

是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象。

==产生的四个必要条件==

**互斥条件：**资源是独占的且排他使用，进程互斥使用资源，即任意时刻一个资源只能给 一个进程使用，其他进程若申请一个资源，而该资源被另一进程占有时，则申请者等待 直到资源被占有者释放。

**不可剥夺条件：**进程所获得的资源在未使用完毕之前，不被其他进程强行剥夺，而只能 由获得该资源的进程资源释放

**请求和保持条件：**进程每次申请它所需要的一部分资源，在申请新的资源的同时，继续 占用已分配到的资源

**环路等待条件：**在发生死锁时必然存在一个进程等待队列{P1,P2,…,Pn},其中 P1 等待 P2 占有的资源，P2 等待 P3 占有的资源，…，Pn 等待 P1 占有的资源，形成一个进程等待 环路，环路中每一个进程所占有的资源同时被另一个申请，也就是前一个进程占有后一 个进程所深情地资源

==如何解决死锁==

*预防死锁*

*避免死锁*

*检测死锁*

*解除死锁*

## sychronized

synchronized 是 Java 的关键字，是一种同步锁，==被 synchronized 修饰的代码块及 方法，在同一时间，只能被单个线程访问==。

三种使用场景

### 修饰实例方法

 synchronized修饰实例方法只需要在方法上加上synchronized关键字即可。

```java
public class Synchronized {
public synchronized void add(){
       i++;
	}
}
```

此时，synchronized加锁的对象就是这个方法所在实例的本身。

### 同步静态方法

synchronized修饰静态方法的使用与实例方法并无差别，在静态方法上加上synchronized关键字即可

```java
public static synchronized void func(){
       i++;
}
```

此时，synchronized加锁的对象为当前静态方法所在类的Class对象。

### 同步代码块

synchronized修饰代码块需要传入一个对象。指定一个加锁的对象，**给对象加锁**

```java
public void func() {
    synchronized (this) {
        i++;
    }
}
```

它

### Java对象头和Monitor对象

- **实例数据 **存放类的属性数据信息，包括父类的属性信息，这部分内存按4字节对齐。
- **填充数据** 由于虚拟机要求对象起始位置必须是8字节的整数倍。填充数据不是必须存在的，只是为了字节对齐。
- **对象头** 对象头被分为两部分，分别为： **Mark Word**（标记字段）、**Class Pointer**（类型指针）。如果是数组，那么还会有数组长度。
  - **Mark Word（标记字段）**：默认存储对象的HashCode，分代年龄和锁标志位信息。
  - **Class Point（类型指针）**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

![image-20211207165932569](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211207165932569.png)

#### 对象头

在对象头的Mark Word中主要存储了对象自身的**运行时数据**，例如哈希码、GC分代年龄、锁状态、线程持有的锁、偏向线程ID以及偏向时间戳等。

![image-20211126203020789](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211126203020789.png)

==当为重量级锁时，对象头的MarkWord中存储了指向Monitor对象的指针。==

#### Monitor对象

Monitor对象被称为管程或者监视器锁。在Java中，每一个对象实例都会关联一个Monitor对象。这个Monitor对象既可以与对象一起创建销毁，也可以在线程试图获取对象锁时自动生成。当这个Monitor对象被线程持有后，它便处于锁定状态。

**锁升级**

==偏向锁==:认为锁不存在多线程竞争，总是由同一线程获得。。所以当一个线程获取锁的时候，会在对象头和栈帧中的锁记录里存放锁偏向的线程 ID，之后该线程进入和退出该同步 代码块不需要通过 cas 操作获得锁和释放锁，只需要比较对象头内存储的线程 ID 是不是当 前线程，是的话就获得了锁。

**当多个线程竞争锁时，偏向锁就会撤销，偏向锁撤销之后会升级为轻量级锁**

Monitor是由[ObjectMonitor](https://link.juejin.cn/?target=https%3A%2F%2Fhg.openjdk.java.net%2Fjdk8u%2Fjdk8u%2Fhotspot%2Ffile%2F782f3b88b5ba%2Fsrc%2Fshare%2Fvm%2Fruntime%2FobjectMonitor.hpp)实现的,它是一个使用C++实现的类

ObjectMonitor中有五个重要部分，分别为_ower,_WaitSet,_cxq,_EntryList和count。

- **_ower** 用来指向持有monitor的线程
- **_WaitSet** 调用了锁对象的wait方法后的线程会被加入到这个队列中
- **_cxq** 是一个阻塞队列，线程被唤醒后根据决策判断是放入cxq还是EntryList
- **_EntryList** 没有抢到锁的线程会被放到这个队列
- **coun**t 用于记录线程获取锁的次数，成功获取到锁后count加1，释放锁时count减1

**_WaitSet存放的是处于WAITING状态等待被唤醒的线程。而_EntryList队列中存放的是等待锁的BLOCKED状态。_cxq队列仅仅是临时存放，最终还是会被转移到_EntryList中等待获取锁。**

### synchronized底层实现原理

**同步代码块**

通过javap -v来反汇编下面的一段代码。

```java
public void add() {
    synchronized (this) {
        i++;
    }
}
```

可以得到如下的字节码指令：

```java
public class com.zhangpan.text.TestSync {
  public com.zhangpan.text.TestSync();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void add();
    Code:
       0: aload_0
       1: dup
       2: astore_1
       3: monitorenter    // synchronized关键字的入口
       4: getstatic     #2                  // Field i:I
       7: iconst_1
       8: iadd
       9: putstatic     #2                  // Field i:I
      12: aload_1
      13: monitorexit  // synchronized关键字的出口
      14: goto          22
      17: astore_2
      18: aload_1
      19: monitorexit // synchronized关键字的出口
      20: aload_2
      21: athrow
      22: return
    Exception table:
       from    to  target type
           4    14    17   any
          17    20    17   any
}
```

由此可以得出在字节码中会在同步代码块的入口和出口加上monitorenter和moniterexit指令。当执行到monitorenter指令时，线程就会去尝试获取该对象对应的Monitor的所有权，即尝试获得该对象的锁。

当该对象的 monitor 的计数器count为0时，那线程可以成功取得 monitor，并将计数器值设置为 1，取锁成功。如果当前线程已经拥有该对象monitor的持有权，那它可以重入这个 monitor ，计数器的值也会加 1。而当执行monitorexit指令时，锁的计数器会减1。

### 偏向锁

经研究发现，**在大多数情况下锁不仅不存在多线程竞争关系，而且大多数情况都是被同一线程多次获得**。因此，为了减少同一线程获取锁的代价而引入了偏向锁的概念。

偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也变为偏向锁结构，即将对象头中Mark Word的第30bit的值改为1，并且在Mark Word中记录该线程的ID。当这个线程再次请求锁时，无需再做任何同步操作，即可获取锁的过程。

### 轻量级锁

轻量级锁优化性能的依据是**对于大部分的锁，在整个同步生命周期内都不存在竞争。** 当升级为轻量级锁之后，MarkWord的结构也会随之变为轻量级锁结构。JVM会利用CAS尝试把对象原本的Mark Word 更新为Lock Record的指针，成功就说明加锁成功，改变锁标志位为00，然后执行相关同步操作。

### 自旋锁

自旋锁是基于**在大多数情况下，线程持有锁的时间都不会太长**。因此自旋锁会假设在不久将来，当前的线程可以获得锁，因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因)，不断的尝试获取锁。如果还不能获得锁，那就会将线程在操作系统层面挂起，即进入到重量级锁。

### synchronized锁升级过程

1. 当没有被当成锁时，就是一个普通的对象，Mark Word记录对象的HashCode。锁标志位时01，是否偏向锁那一位是0；
2. 当对象被当做同步锁并有一个线程A抢到锁时，锁标志位还是01，但是`是否偏向锁`那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态。
3. 当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步中的代码;
4. 当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用*CAS操作试图获得锁*。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步代码。如果抢锁失败，则继续执行步骤5;
5. 偏向锁状态抢锁失败，代表当前所有一定的竞争，**偏向锁将升级为轻量级锁**。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时会在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤6;
6. 轻量级锁抢锁失败，JVM会使用自旋锁，不断地重试，尝试抢锁。
7. 自旋锁重试之后如果抢锁依然失败，同步锁会升级到重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。

## AQS源码

[AQS源码详细解读 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/65349219)

```
AbstractQueuedSynchronizer  //队列同步器
```

使用锁时，线程获取锁是一种**悲观锁策略**，即假设每一次执行临界区代码都会产生冲突，所以当前线程获取到锁的时候同时也会阻塞其他线程获取该锁。 而CAS操作（又称为无锁操作）是一种**乐观锁策略**，它假设所有线程访问共享资源的时候不会出现冲突。

###  成员变量

```java
private volatile int state;

//设置期望值，想修改的值，通过CAS操作实现
protected final boolean compareAndSetState(int expect, int update){
    return unsafe.compareAndSwapInt(this, stateOffset,expect, update);
}
//维护了等待队列（也叫CHL队列，同步队列）的头节点和尾节点
private transient volatile Node head;
private transient volatile Node tail;
//CHL队列由链表实现，以自旋的方式获取资源，是可阻塞的先进先出的双向队列，通过自旋和CAS操作保证节点插入和移除的原子性，当有线程获取锁失败，就被添加到队列==末尾==
```

### 内部类Node

AQS的工作模式分为独占模式和共享模式，记录在节点的信息中。

一般地，它的实现类只实现一种模式，ReentrantLock就实现了独占模式；但也有例外，ReentrantReadAndWriteLock实现了独占模式和共享模式。

```java
//当前节点处于共享模式的标记
static final Node SHARED = new Node();
//当前节点处于独占模式的标记
static final Node EXCLUSIVE  = null;
//线程被取消了
static final int CANCELLED =  1;
//释放资源后需唤醒后继节点
static final int SIGNAL    = -1;
//等待condition唤醒
static final int CONDITION = -2;
//工作于共享锁状态，需要向后传播，
//比如根据资源是否剩余，唤醒后继节点
static final int PROPAGATE = -3;

//等待状态，有1,0,-1,-2,-3五个值。分别对应上面的值
volatile int waitStatus;

//前驱节点
volatile Node prev;

//后继节点
volatile Node next;

//等待锁的线程
volatile Thread thread;

//等待条件的下一个节点，ConditonObject中用到
Node nextWaiter;
```

### 获取资源（锁）

获取释放资源是对state变量的修改，

获取锁的方法有**acquire(),acquiredShared()**.

- acquire()独占模式获取资源，忽略中断

```java
public final void acquire(int arg){
    if(!tryAcquire(arg) && 
      //让线程处于一种自旋状态
      //尝试让该线程重新获取锁！ 当条件满足获取到了锁可以从自旋过程中退出，否则继续
      acquireQueued(addWait(Node.EXCULSIVE), arg))
      selfInterrrupt();
}
```

addWaiter()：将当前线程插入至队尾，返回在等待队列中的节点（就是处理了它的前驱后继）

```java
  private Node addWaiter(Node mode) {
        //把当前线程封装为node,指定资源访问模式
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        //如果tail不为空,把node插入末尾
        if (pred != null) {
            node.prev = pred;
            //此时可能有其他线程插入,所以使用CAS重新判断tail
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //如果tail为空，说明队列还没有初始化，执行enq()
        enq(node);
        return node;
    }
```

### CAS的操作过程

**V 内存地址存放的实际值； O 预期的值（旧值）；N 更新的新值**

当V和O相同时，也就是说旧值和内存中实际的值相同表明该值没有被其他线程更改过，即该旧值O就是目前来说最新的值了，自然而然可以将新值N赋值给V。

### 存在的问题

#### ABA问题

解决方案可以宴席数据库中常用的乐观锁方式，添加一个版本号

#### 自旋时间过长

使用CAS时非阻塞同步，也就是说不会将线程挂起，会自旋进行下一次尝试，如果自旋时间过长对性能有很大的消耗。

#### 只能保证一个共享变量的原子操作

## Reentrantlock

重入锁（ReentrantLock）是一种递归无阻塞的同步机制。

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁

### ReentrantLock使用

ReentrantLock是一种显式锁，需要我们**手动编写加锁和释放锁**的代码。

```java
public class ReentrantLockDemo{
    // 实例化一个非公平锁，构造方法的参数为true表示公平锁，false为非公平锁。
    private final ReentrantLock lock = new ReentrantLock(false);
    private int i;
    
    public void testLock(){
        //获取锁，如果获取不到就一直等待
        lock.lock();
        try{
            // 再次尝试获取锁（可重入），拿锁最多等待100ms
            if(lock.trylock(100, TimeUnit.MILLISECONDS))
                i++;
        }	catch(InterruptedException e){
            e.printStackTrace();
        }finally{
            // 释放锁
            lock.unlock();
            lock.unlock();
        }
    }
}
```

### 源码解析

```java
public class ReentrantLock implements Lock, java.io.Serializable {

    private final Sync sync;
    
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    
    // ...省略其它代码
}
```

它实现了Lock和Serializable两个接口，同时有两个构造方法，在无参构造方法中初始化了一个非公平锁，在有参构造方法中根据参数决定是初始化公平锁还是非公平锁。

```java
public interface Lock(){
    //获取锁
    void lock();
    void lockInterruptibly() throws InterruptedException;
    // 尝试获取锁，成功返回true，失败返回false
    boolean tryLock();
    // 在给定时间内尝试获取锁，成功返回true，失败返回false
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    // 释放锁
    void unlock();
    // 等待与唤醒机制
    Condition newCondition();
}
```

**NonfairSync非公平锁的实现：**

```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

**lock方法**

```java
// ReentrantLock
public void lock(){
    sync.acquire(1);
}
   // AbstractQueuedSynchronizer
    
public final void acquire(int arg) {
	if (!tryAcquire(arg) &&acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```



## Lock接口

实现类： ReentrantLock，ReentrantReadWriteLock

简单使用：

```java
Lock Lock = new ReentrantLock();
lock.lock();
try{
    
}finally{
    lock.unlock();
}
//最好不要把获取锁的过程写在try语句块中，因为如果在获取锁时发生了异常，异常抛出的同时也会导致锁无法被释放
```

Condition newCondition()

==Condition接口==

synchronized关键字与wait()和notify/notifyAll()方法相结合可以实现等待/通知机制

ReentrantLock需要借助于Condition接口与newCondition（）方法，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。

```java
//使用Condition实现等待、通知机制
//使用单个Condition实例实现等待/通知机制

```

## 线程池

### 线程池的主要参数

```java
public ThreadPoolExecutor(int corePoolSize,
                         int maxmumPoolsize,
                         long keepAliveTime,
                         TimeUnit unit,
                         BlockingQueue<Runnable workQueue,
                         ThreadFactory threadFactory,
                         RejectedExecutionHandler handler) 
```

**corePoolSize**: 线程池中核心线程的数量。当有任务提交到线程池时，如果线程池中的线程数小于corePoolSize,那么则直接创建新的线程来执行任务。

**maximumPoolSize**: 线程池中最大线程数量。当一个任务提交到线程池时，线程池中的线程数大于corePoolSize,并且workQueue已满，那么则会创建新的线程执行任务，但是线程数要小于等于maximumPoolSize。

**keepAliveTime**：非核心线程的存活时间 

**TimeUnit unit**：存活时间单位 

**workQueue**：任务队列。它是一个阻塞队列，用于存储来不及执行的任务的队列。当有任务提交到线程池的时候，如果线程池中的线程数大于等于corePoolSize，那么这个任务则会先被放到这个队列中，等待执行。

**threadFactory**：线程工厂，用于创建线程，一般用默认的即可 

**handler**：拒绝策略，当队列满了并且工作线程大于等于线程池的最大线程数

### 线程池的生命周期

线程池从诞生到死亡，中间会经历RUNNING、SHUTDOWN、STOP、TIDYING、TERMINATED五个生命周期状态。

- **RUNNING** 表示线程池处于运行状态，能够接收新提交的任务且能对已添加的任务进行处理。RUNNING状态是线程池的初始化状态，线程池一旦被创建就处于RUNNING状态。

- **SHUTDOWN** 线程处于关闭状态，不接受新任务，但可以处理已添加的任务。RUNNING状态的线程池调用shutdown后会进入SHUTDOWN状态。

- **STOP** 线程池处于停止状态，不接受任务，不处理已添加的任务，且会中断正在执行任务的线程，RUNNING状态的线程池调用了shutdownNow后会进入STOP状态。

- **TIDYING**  当所有任务已终止，且任务数量为0时，线程池会进入TIDYING。当线程池处于SHUTDOWN状态时，阻塞队列中的任务被执行完了，且线程池中没有正在执行的任务了，状态会由SHUTDOWN变为TIDYING。当线程处于STOP状态时，线程池中没有正在执行的任务时则会由STOP变为TIDYING。

- **TERMINATED** 线程终止状态。处于TIDYING状态线程执行terminated（）后进入该状态。

  ![image-20211127145041171](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211127145041171.png)

  

### 线程池的执行流程



![image-20211108220501033](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211108220501033.png)

1） 在创建线程池后，等待提交过来的任务请求。

2） 当调用execute()方法添加一个请求任务时，线程池会做如下判断：

- 如果正在运行的线程数量小于corePoolSize，那么马上创建核心线程运行这个线程。
- 如果正在运行的线程数量大于或者等于corePoolSize，且此时没有空闲的线程，那么则会将任务存储到workQueue中。
- 如果任务队列满了且正在运行的线程数量小于maximumPoolSize(最大线程数)，那么创建一个非核心线程立刻运行这个任务；

- 如果任务队列满了且正在运行的线程数量大于或等于maximumPoolSize，线程池会执行拒绝策略；

3） 当一个线程完成任务时，会在队列中取下一个任务来执行；

### 任务拒绝策略

线程池有一个最大的容量，当线程池的任务缓存队列已满，并且线程池中的线程数目达到maximumPoolSize时，就需要拒绝该任务。

拒绝策略是一个接口：

```java
public interface RejectedExecutionHandler{
    void rejectedExecution(Runnable r, ThreadPoolExecutor, executor);
}
//可以通过实现这个接口去定制拒绝策略，也可以选择JDK提供的4种已有策略

```

![image-20211109195944246](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211109195944246.png)

### 线程池的创建方式

```java
ThreadPoolExecutor//：最原始的创建线程池的方式，它包含了 7 个参数可供设置
//1. 创建一个固定大小的线程池
newFixedThreadPool(int nThreads)
//2. 创建单个线程的线程池
newSingleThreadExecutor()
//3. 创建一个可缓存的线程池，若线程数超过处理所需，缓存一段时间后会回收，若线程数不够，则新建线程。
newCachedThreadPool()
//4. 创建一个可以执行延迟任务的线程池。
newScheduledThreadPool(int corePoolSize)
```

## Java内存模型（JMM）

Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

### 主内存与工作内存

处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

![image-20220112001213961](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220112001213961.png)

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

所有的变量都存储在**主内存**中，每个线程还有自己的**工作内存**，工作内存存储在**高速缓存或者寄存器**中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![image-20220112001300651](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220112001300651.png)

### 内存间交互操作

Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作。

![image-20220112001405260](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220112001405260.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在read之后执行，把read得到的值放入工作内存的变量副本中
- use：把工作内存中的变量的值传递给执行引擎
- assgin：把一个从执行引擎接收到的值赋给工作内存中的变量
- store：把工作内存中的一个变量的值传送到主内存中
- write：在store之后执行，把store得到的值放入主内存的变量中
- lock：作用于主内存的变量，把一个变量标识为一条线程独占状态
- unlock：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定

如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行 **read 和 load** 操作，如果把变量从工作内存中同步回主内存中，就要按顺序地执行 **store 和 write** 操作。Java内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。也就是 read 和 load 之间，store 和 write 之间是可以插入其他指令的，如对主内存中的变量a、b进行访问时，可能的顺序是read a，read b，load b， load a。

### 重排序和happen-before规则

![image-20220112132752330](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220112132752330.png)

- **编译器重排序**：编译器保证不改变单线程执行结果的前提下，可以调整多线程语句执行顺序。
- **处理器重排序**：如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。

**JMM通过happen-before规则，底层禁止特定类型的编译器重排序和处理器重排序，保证内存的可见性和有序性**
