## ArrayList

### ArrayList底层实现

ArrayList是基于数组实现的，是一个动态数组。它实现了List接口，底层使用数组保存所有元素；其操作基本上是对数组的操作。

1）私有属性：

```java
// ArrayList只定义类的两个私有属性
// elementData存储ArrayList内的元素
private transient Object[] elementData;
// size表示它包含的元素的数量
private int size;
```

2）构造方法：

- 构造一个默认初始容量为10的空列表
- 构造一个指定初始容量的空列表
- 构造一个包含指定collection的元素的列表

```java
// ArrayList带容量大小的构造函数。 
public ArrayList(int initiaCapacity){
    super();
    if(initiaCapacity < 0)
       throw new IllegalArgumentException("Illegal Capacity: "+    
                                               initialCapacity);
    //新建一个数组
    this.elementData = new Object[initialCapacity];
}
//ArrayList无参构造方法，默认容量是10。
public ArrayList(){
    this(10);
}
// 创建一个包含collection的ArrayList    
public ArrayList(Collection<? extends E> c){
    elementData = c.toArray();
    size = elementData.length;
    if (elementData.getClass() != Object[].class)    
            elementData = Arrays.copyOf(elementData, size, Object[].class);  
}
```

3）元素存储

`set(int index, E element)、add(E e)、add(int index, E element)、addAll(Collection<? extends E> c)、addAll(int index, Collection<? extends E> c)`

```java
// 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素。  
public E set(int index, E element){
    RangeCheck(index);
    
    E oldValue = (E)elementData[index];
    elementData[index] = element;
    return oldValue;
}

//将指定的元素添加到此列表的尾部
public boolean add(E e){
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}

//将指定的元素插入此列表中的指定位置
//如果当前位置有元素，则向右移动当前位于该位置的元素以及所有后续元素
public void add(int index, E element){
    rangeCheckForAdd(index);
    //如果数组长度不足，将进行扩容
    ensureCapacity(size + 1); //自增+1
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
}
//按照指定collection的迭代器锁返回的元素顺序，将该collection中的所有元素添加到此列表的尾部
public boolean addAll(Collection<? extends E> c){
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  //自增+1
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```

4）元素读取

```java
public E get(int index){
    RangeCheck(index);
    return (E) elementData[index];
}
```

5）元素删除

```java
//两种方式
/*移除指定位置的元素
*/
public E remove(int index){
    RangeCheck(index);
    
    modCount++;
    E oldValue = (E)elementData[index];
    
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index + 1, elementData, index, numMoved);
    elementData[--size] = null;
    
    return oldValue;
}
/*移除列表z首次出现的指定元素（如果存在）
*/
public boolean remove(Object o){
    //由于Array List种允许存放null，因此有两种情况
    // 1.
    if (o == null){
        for (int index = 0; index < size; index++)
            if(elementData[index] == null){
                //类似remove(int index)，移除列表中指定位置的元素
                fastRemove(index);
                return true;
            }
    }
    else{
        for(int index = 0; index < size; index++)
            if(o.equals(elementData[index])){
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```

**fastRemove不会判断边界，因为找到元素就相当于确定了index不会超过边界，而且fastRemove并不返回被移除的元素。**



### ArrayList底层是怎么排序的

重写sort方法

```java
@Override
public void sort(Comparator<? super E> c){
    final int expectedCount = modCount;
    Arrays.sort((E[]) elementData, 0 , size, c);
    if(modCount != expectedModCount){
        throw new ConcurrentModificationException();
    }
    modCount++;
}
//使用

arrayList.sort(Comparator.naturalOrder());
arrayList.sort(Comparator.reverseOrder());
```

### ArrayList扩容机制

```java
private void grow(int minCapacity){
    //考虑溢出
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); //扩大1.5倍
    if(newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if(newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### ArrayList多线程访问

#### ArrayIndexOutOfBoundsException

容量是10，现在有9个元素，然后有两个线程同时向队列中增加元素：

1. `线程1`开始进入`add`方法，获取size=9，调用`ensureCapacityInternal` 方法进行容量判断，此时数组容量是10，不需要扩容。
2. `线程2`也进入`add`方法，获取size=9，调用`ensureCapacityInternal`方法进行容量判断，此时数组容量还是10，也不需要扩容。
3. `线程1`开始赋值，也就是`elementData[size++] = e`，此时**size变成10**，达到数组容量极限。
4. `线程2`此次开始执行赋值操作，使用的size=10，也就是`elementData[10] = e`，因为下标从0开始，目前数组容量是10，直接报**数组越界**`ArrayIndexOutOfBoundsException`。

#### null元素

假设还是有两个线程要赋值，此时数组长度还比较富裕，比如数组长度是10，目前size=5：

1. 线程1开始进入`add`方法，获取size=5，调用`ensureCapacityInternal`方法进行容量判断，此时数组容量是10，不需要扩容
2. 线程2也进入`add`方法，获取size=5，调用`ensureCapacityInternal`方法进行容量判断，此时数组容量还是10，也不需要扩容
3. 线程1开始赋值，执行`elementData[size] = e`，此时size=5，在执行`size++`之前，线程2开始赋值了
4. 线程2开始赋值，执行`elementData[size] = e`，此时size还是5，所以线程2把线程1赋的值覆盖了
5. 线程1开始执行`size++`，此时size=6
6. 线程2开始执行`size++`，此时size=7

也就是说，添加了2个元素，队列长度+2，但是真正加入队列的元素只有1个，有一个被覆盖了。





Java 8 `add`方法：

```java
public boolean add(E e){
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}
```

Java 11 `add`方法：

```java
public boolean add(E e){
    modCount++;
    add(e, elementData, size);
    return true;
}

private void add(E e, Object[] elementData, int s){
    if(s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = size + 1;
}
```

两段逻辑的差异在于**数组下标是否确定**：

- `elementData[size++] e`， Java 8中直接使用`size`定位并赋值，然后通过`size++`自增。
- `elementData[s] = e; size = s + 1`，Java 11 借助临时变量`s`定位并赋值，然后通过`size = s + 1`给`size`赋新值

## Vector

### 同步

与ArrayList类似，但是使用了`synchronized`进行同步

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
} 

public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
}
```

### ArrayList与Vector的比较

- Vector是同步的，因此开销就比ArrayList要大，访问速度更慢。
- Vector每次扩容请求其大小的2倍空间，而ArrayList是1.5倍。

### Vector替代方法

**synchronizedList**

为了获得线程安全的ArrayList，可以使用`Collections.synchronizedList()`得到一个线程安全的ArrayList。

```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchroziedList(list);
```

**CopyOnWriteArrayList**

CopyOnWrite容器即写时复制的容器。**当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行 Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器**。

这样做的好处是我们可以对 CopyOnWrite 容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以 CopyOnWrite 容器也是一种**读写分离**的思想，读和写不同的容器。

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

读的时候不需要加锁，如果读的时候有多个线程正在向ArrayLIst添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的ArrayList。

```java
public E get(int index) {
    return get(getArray(), index);
}
```

CopyOnWrite的缺点

**内存占用问题**

- 写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象。那么这个时候很有可能造成频繁的 Yong GC 和 Full GC。

**数据一致性问题**

- CopyOnWrite 容器只能保证数据的最终一致性，不能保证数据的实时一致性。

## LinkedList

### LinkedList的底层实现

**Node类**

```java
private static class Node<E>{
    E item;
    Node<E> next;
    Node<E> prev;
    
    Node(Node<E> prev, E element, Node<E> next){
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

**属性**

```java
//节点数  transient关键字：不参与对象的序列化
transient int size = 0;
//指向第一个节点的引用
transient Node<E> first;
//指向最后一个节点的引用
transient Node<E> last;
```

**构造方法**

```java
//无参构造
public LinkedList(){
}
//集合构造
public LinkedList(Collection<? extends E> c){
    this();
    addAll(c);
}
```

**核心方法**

```java
//返回列表中指定位置的元素
public E get(int index){
    checkElementIndex(index);
    return node(index).item;
}
```

```java
//返回指定元素索引处的节点
Node<E> node(int index){
    //二分法，选择从前向后遍历还是从后向前遍历
    if (index < (size >> 1)){
        Node<E> x = first;
        for(int i = 0; i < index; i++)
            x = x.next;
    }
    else{
        Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
    }
}
```

```java
//向指定位置添加元素
public void add(int index, E element){
    checkPostionIndex(index);
    
    if(index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

//添加的元素是在末尾
void linkLast(E e){
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if(l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

//正常添加
void linkBefore(E e, Node<E> succ){
    final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
}
```

```java
//删除元素
public E remove(int index){
    checkElementIndex(index);
    return unlink(node(index));
}

//真正的删除方法
E unlink(Node<E> x){
    final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
}
```

## HashMap

1. 了解底层如何存储数据的
2. HashMap的几个主要方法
3. HashMap时如何确定元素存储位置的以及如果处理哈希冲突的
4. HashMap扩容机制是怎么样的
5. JDK1.8在扩容和解决哈希冲突上对HashMap源码做了哪些改动？有什么好处

### HashMap底层实现

**关键变量**

```java
//初始容量大小  必须是2的幂
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; //  = 16
//最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;

//负载因子，因为统计学中hash冲突符合泊松分布，7-8之间冲突最小
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//链表大于这个值就会树化
static final int TREEIFY_THRESHOLD = 8;

//小于这个值就会反树化
static final int UNTREEIFY_THRESHOLD = 6;
```

**构造方法**

- 指定初始容量大小，负载因子

```java
public HashMap(int initialCapacity, float loadFactor){
    if(initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY) //1<<30 最大容量是 Integer.MAX_VALUE;
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
	this.loadFactor = loadFactor;
    //tableSizeFor 可以找到最小的2的幂
    this.threshold = tableSizeFor(initialCapacity);  
}
```

- *负载因子给的默认值0.75*

```java
public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

- 空参构造，均使用默认值

```java
public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

- 初始化

```java
public HashMap<Map<? extends K, ? extends V> m){
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

### HashMap计算Hash值

**hash方法**

```java
static final int hash(Object key){
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

`h = key.hashCode()`表示`h`是key对象的hashCode返回值；

`h >>> 16`是h右移16位，因为int是4字节，32位，所以右移16位后变成：左边16个0 + 右边原h的高16位；

**异或**：二进制位运算。如果一样返回 0，不一样则返回 1。

例：两个二进制 110 和 100 进行异或

​        	  110

​          ^ 100

结果     010

**putVal（）中寻址部分**

`tab[i = (n - 1) & hash]`

**为什么不直接用hashCode（）%length?**

**寻址为什么不用取模**

HashMap中规定了哈希表长度为2的幂，而2 ^k - 1转为二进制就是k个连续的1，那么`hash & (k个连续的1)返回的就是hash的低k个位，该计算结果就是0到2^k-1。

为什么不直接用 hashCode() 而是用它的高 16 位进行异或计算新 hash 值？

int 类型占 32 位，可以表示 2^32 种数（范围：-2^31 到 2^31-1），而哈希表长度一般不大，在 HashMap 中哈希表的初始化长度是 16（HashMap 中的 DEFAULT_INITIAL_CAPACITY），如果直接用 hashCode 来寻址，那么相当于只有低 4 位有效，其他高位不会有影响。这样假如几个 hashCode 分别是 210、220、2^30，那么寻址结果 index 就会一样而发生冲突，所以哈希表就不均匀分布了。

为了减少这种冲突，HashMap 中让 hashCode 的高位也参与了寻址计算（进行扰动），即把 hashCode 高 16 位与 hashCode 进行异或算出 hash，然后根据 hash 来做寻址。

### HashMap的put操作

![image-20220111190618268](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220111190618268.png)

```java
public V put(K key, V value){
    //把key先hash得到hash值
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict){
    //数组+链表+红黑树，链表型（Node泛型)数组，每一个元素代表一条链表，则每个元素称为桶
    //HashMap 的每一个元素，都是链表的一个节点（Entry<K,V>）这里也就是Node<K,V>
    //tab:桶 p:桶 n:哈希表数组大小 i:数组下标(桶的位置)
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //1.判断当前是否为0，空就调用resize()方法（resize中会判断是否进行初始化）
    if((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //2.判断是否有hash冲突，根据入参key和key的hash值找到具体的桶并且判空，
    if((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
    //3.以下表示有冲突，处理hash冲突
    else{
        Node<K,V> e; K k; //均为临时变量
        //4.判断当前桶的key是否与入参key一致，一致则存在，把当前桶p赋值给e,覆盖原 value 在步骤10进行
        if(p.hash == hash && ((k = p.key) == key ||(key != null && key.equals(k))))
            e = p;
        //5.如果当前的桶为红黑树，用putTreeVal方法写入
        else if(p instanceof TreeNode)
            e = ((TreeNode<K,V>) p).putTreeVal(this, tab, hash, key, value);
        //当前的桶是链表 遍历链表
        else{
            for(int binCount = 0; ; ++binCount){
                if((e = p.next) == null){
                    //7.尾插法 链表下一个节点是null
                    p.next = newNode(hash, key, value, null);
                    //8.判断是否大于阈值，是否需要转成红黑树
                    if(binCount >= TreeIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                //9. 如果链表中key存在，则直接跳出
                if(e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //10. 存在相同的key的Node节点，覆盖原来的value
        if(e != null){
            V oldValue = e.value;
            if(!onlyIfAbsent || oldValue == null)
                e.value = value;
            //LinkedHashMap用到的回调方法
                    afterNodeAccess(e);
                    return oldValue;
        }
    }
    ++modCount;
    if(++size > threshold)
        resize();
    //LinkedHashMap用到的回调方法
            afterNodeInsertion(evict);
            return null;
}

/**
1、开始，入参key、value
2、判断当前table是否为空或者length=0？
    是，去扩容，（resize()方法中有判断是否初始化）
    否，根据key算出hash值并得到插入的数组的索引
        判断找到的这个table[i]是否为空？
            是，直接插入，再到步骤
            否，判断key是否存在？
                是，直接覆盖对应的value,再到步骤3
                否，去判断当前这个table[i]是不是treeNode？
                    是，使用红黑树的方式插入key、value
                    否，开始遍历链表准备插入
                        判断链表长度是不是大于8？
                            是，链表转红黑树插入key、value
                            否，以链表的方式插入key、value如果key存在就直接覆盖对应value

3、判断map的size()是否大于阈值？
	    是 就去扩容resize()
4、结束
**/

```

### HashMap的get操作

```java
public V get(Object key){
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}


final Node<K,V> getNode(int hash, Object key){
    Node<K,V>[] tab; Node<K,V> first, e, int n; K k;
    //1.判断当前数组不为空并且长度大于0 && 由key的hash值找到对应数组下的桶
    if((tab = table) != null &&  (n = table.length) > 0 &&
      (first = tab[(n - 1)& hash]) != null){
        //2.先判断桶的第一个节点 如果key一致 返回
        if(first.hash == hash &&
          ((k = first,key) == key || (key != null && key.equals(k))))
            return first;
        //3.再判空下一个节点不为空 && 判断是红黑树还是链表
        if((e = first.next) != null){
            //4.如果是红黑树 则按红黑树方式取值
            if(first instanceof TreeNode)
                return ((TreeNode<K,V> first).getTreeNode(hash,key));
            //否则就是链表
            do{
                if(e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            }
            while((e = e.next) != null);
        }
        
    }
    return null;
}
```

```java
/**1、开始，入参key
2、判断当前的数组长度不为空&&length>0
    是 return null;
    否，去判断第一个节点，如果key符合，返回
        再去判断下一个节点是否为空
            是，return null
            否，判断否是红黑树？
                是，按红黑树的方式取值
                否，遍历链表取值

3、结束
**/
```

### HashMap的扩容机制

```java
final Node<K,V>[] resize(){
    Node<K,V>[] oldtab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int OldThr = threshold;
    int newCap, newThr = 0;
    //1.原数组扩容
    if(oldCap > 0){
        //如果原数组长度大于最大容量，把阈值调最大，return
        if(oldCap >= MAXIMUM_CAPACITY){
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //把原数组大小、阈值都扩大一倍
        else if((newCap = oldCap << 1) < MAXIMUM_CAPACITY && 
               oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;   // 现阈值 = 原阈值的两倍
    }
    //使用指定initialCapacity的构造方法，则用原阈值作为新容量
    else if(oldThr > 0)
        newCap = oldThr;
    //使用空参构造，用默认值
    else{
        newCap = DEFAULT_INITIAL_CAPACITY; //16
        newThr = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_INITIAL_LOAD_FACTOR); //0.75*16 = 12
    }
    //使用指定了initialcapacity的构造方法，新阈值为0，则计算新的阈值
    if(newThr == 0){
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE); 
    }
    threshold = newThr;
    //2.用新的数组容量大小初始化数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //如果仅仅是初始化过程，到此结束
    table = newTab;
    //3.开始扩容的主要工作，数据迁移
    if(oldTab != null){
        //遍历原数组开始复制旧数据
        for(int j = 0; j < oldCap; j++){
            Node<K,V> e;
            if((e = oldTab[j]) != null){
                oldTab[j] = null;
                //原数组中单个元素，直接复制到新表
                if(e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //如果该元素类型是红黑树，则按红黑树处理
                else if(e instanceof TreeNode)
                    ((TreeNode<K,V>)e.split(this, newTab,j, oldCap));
                else{
                    //先定义了两种类型的链表，以及头尾节点，高位链表和低位链表
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //按顺序遍历原链表的节点
                    do{
                        next = e.next;
                        //核心判断条件，
                        // = 0 则放到低位链表
                        if(e.hash & oldCap) == 0){
                            if(loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                    }
                    while((e = next) != null);
                    //把整个低位链表放到新数组的j位置
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        //把整个高位链表放到新数组的j+oldCap位置上
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                }
            }
        }
    }
        return newTab;
}
```

```java
/**1、开始，拿到原数组
2、对原数组扩容
	  2.1如果原数组中的容量到最大，不再扩容，return原数组
	  2.2把原数组容量大小与阈值都扩大一倍
3、如初始化用的指定initialCapacity的构造方法，则用原阈值作为新容量
4、如初始化时候用的空参构造，用默认容量与默认阈值
5、如初始化用的指定initialCapacity的构造方法，阈值=0，计算新的阈值
6、用新的容量初始化数组，如果是初始化，结束返回新数组
7、开始扩容，做数据迁移
	  7.1遍历原数组copy数据到新数组
		    7.1.1如数组中只有一个元素，则直接复制
		    7.1.2如元素是红黑树数类型，则按红黑树的方式处理
		    7.1.3对原数组的链表进行处理
				      定义一个高位链表、一个低位链表（对原链表拆分）
				      开启一个循环，遍历原链表
					        判断条件e.hash & oldCap == 0?
						          是，把这些链表节点放到低位链表
						          否，放到高位链表
				      循环结束，遍历链表完成
				      把整个低位链表放到新数组j位置
				      把整个高位链表放到新数组j+oldCap位置
	  7.2循环结束，遍历旧数组完成
8、返回新数组
**/
```



### HashMap扩容时(e.hash & oldCap) == 0推导

[(1条消息) HashMap 在扩容时为什么通过位运算 (e.hash & oldCap) 得到新数组下标_Luke.Du的博客-CSDN博客](https://blog.csdn.net/qq_45369827/article/details/114960370)

1. HashMap计算key所对应数组下标的公式是`(length - 1) & hash`，这个公式等价于`hash % length`(当length是2的n次幂)
2. 如下图，`hash % length`的结果只取决于小于数组长度的部分，这个key的hash值的低四位就是当前所在数组的下标。扩容后 新数组长度 = 旧数组长度 * 2，也就是左移 1 位，而此时 hash % length 的结果只取决于 hash 值的低五位，前后两者之间的**差别就差在了第五位**上。

![image-20211123183145042](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211123183145042.png)

3. 如果第五位是 0，那么只要看低四位 (也就是当前下标)；如果第五位是 1，只要把二进制数 1 0 0 0 0 + 1 1 1 0 ，就可以得到新数组下标。
4. 那为什么根据 (e.hash & oldCap) == 0 来做判断条件呢？是因为旧数组的长度 length 的二进制数的第五位刚好是 1 其它位全为 0，而 & 运算相同为 1 不同为 0，==因此 hash & length 的目的就是为了计算 hash 值的第五位是 0 还是 1。==

### HashMap为什么要使用红黑树而不使用AVL树

**红黑树**

- 根节点是黑色的
- 每个叶子节点都是黑色的空节点（NIL），叶子节点不存储数据
- 任何相邻的节点都不能同时为红色
- 每个节点，从该节点到其可达叶子节点的所有路径，都包含相同数目的黑色节点

AVL树和红黑树有几点比较和区别：
（1）AVL树是更加严格的平衡，因此可以提供更快的查找速度，一般读取查找密集型任务，适用AVL树。
（2）红黑树更适合于插入修改密集型任务。
（3）通常，AVL树的旋转比红黑树的旋转更加难以平衡和调试。

在CurrentHashMap中是加锁了的，实际上是读写锁，如果写冲突就会等待，如果插入时间过长必然等待时间更长，而红黑树相对AVL树他的插入更快！在AVL树中，从根到任何叶子的最短路径和最长路径之间的差异最多为1。在红黑树中，差异可以是2倍。

在AVL树中查找通常更快，但这是以更多旋转操作导致更慢的插入和删除为代价的，红黑树在添加，删除，查找相对较好。

### HashMap 的 tableSizeFor

```java
static final int tableSizeFor(int cap){
    //part1
    int n = cap - 1;
    //part2
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    //part3
    return (n < 0) ? 1: (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

| **操作** | **原始值**           | **结果值** |
| -------- | -------------------- | ---------- |
| n >>> 1  | 1xxxxxxx             | 01xxxxxx   |
| 位或操作 | 1xxxxxxx \| 01xxxxxx | 11xxxxxx   |
| n >>> 2  | 11xxxxxx             | 0011xxxx   |
| 位或操作 | 11xxxxxx \| 0011xxxx | 1111xxxx   |
| n >>> 4  | 1111xxxx             | 00001111   |
| 位或操作 | 1111xxxx \| 00001111 | 11111111   |
| n >>> 8  | 11111111             | 00000000   |
| 位或操作 | 11111111 \| 00000000 | 11111111   |
| n >>> 16 | 11111111             | 00000000   |
| 位或操作 | 11111111 \| 00000000 | 11111111   |

## ConcurrentHashMap

![image-20220111192531120](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220111192531120.png)

### 存储结构

ConcurrentHashMap采用“分段锁“策略，ConcurrentHashMap的主干是一个Segment数组。

```java
final Segment<K, V>[] segments;
```

Segment继承了`ReentrantLock`，它是一种可重入锁。在ConcurrentHashMap，一个Segment就是一个子哈希表，Segment里**维护了一个HashEntry数组**，并发环境下，对于不同 Segment 的数据进行操作是不用考虑锁竞争的。

**所以，对于同一个 Segment 的操作才需考虑线程同步，不同的 Segment 则无需考虑。**

一个 ConcurrentHashMap 维护一个 Segment 数组，一个 Segment 维护一个 HashEntry 数组。

### ConcurrentHashMap底层实现

数组 + 链表 + 红黑树

- table： 默认为null，初始化发生在第一次插入操作，默认为16的数组。
- nextTable：默认为null，扩容时新生成的数组，其大小为原数组的两倍。
- sizeCtl： 默认为0，用来控制table的初始化和扩容操作
  - **-1** 代表table正在初始化
  - **-N** 表示有N - 1个线程正在进行扩容操作
- Node：保存key，value及key的hash值

其中value和next都用volatile修饰，保证并发的可见性

```java
static class Node<K,V> implements Map.Entry<K,V>{
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
}
```

- ForwardingNode: 一个特殊的Node节点，hash值为-1，其中存储nextTable的引用。

只有table发生扩容的时候，ForwardingNode才会发挥作用，作为一个占位符放在table中表示当前节点为null或则已经被移动。

```java
final class ForwardingNode<K,V> extends Node<K,V>{
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab){
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }
}
```

**实例初始化**

实例化ConcurrentHashMap时倘若声明了table的容量，在初始化时会根据参数调整table大小，==确保table的大小总是2的幂次方==。默认的table大小为16.

table的初始化操作回延缓到第一次put操作再进行，并且初始化只会执行一次。

```java
private final Node<K,V>[] initTable(){
    Node<K,V>[] tab; int sc;
    while((tab = table) == null || tab.length == 0){
         // 别的线程已经初始化好了或者正在初始化 sizeCtl 为 -1
        if((sc = sizeCtl) < 0)
            Thread.yield   // 让出线程的执行权
		// CAS 一下，将 sizeCtl 设置为 -1，代表抢到了锁 基本在这就相当于同步代码块
        else if(U.compareAndSwapInt(this, SIZECTL, sc, -1)){
            try{
                if((tab = table) == null || tab.length == 0){
                    // DEFAULT_CAPACITY 默认初始容量是 16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACTIY;
                    // 初始化数组，长度为16或者初始化时提供的长度
                    Node<K,V>[] nt = (Node<K,V>[]) new Node<?, ?>[n];
                    //将这个数组赋值给table，table时volatile的，它的写发生在别人的读之前
                    table = tab = nt;
                    //如果n为16的化，那么这里sc其实就是0.75 * n
                    sc = n -(n >>> 2);
                }
            }
            finally{
                设置下次扩容时的阈值
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

### table扩容

什么时候会触发扩容

- 如果新增节点后，所在的链表的元素个数大于等于8，则会调用treeifyBin把链表转换为红黑树。在转换结构时，若tab的长度小于==MIN_TREEIFY_CAPACITY==，默认值为64，则会将数组长度扩大到原来的两倍，并触发`transfer`,重新调整节点位置。（只有当`tab.length >= 64, ConcurrentHashMap`才会使用红黑树。）

- 新增节点后，`addCount`统计tab中的节点个数大于阈值（sizeCtl），会触发`transfer`，重新调整节点位置。



### 如何在扩容时，并发地复制与插入？

1. 遍历整个table，当前节点为空，则采用CAS的方式在当前位置放入fwd
2. 当前节点已经为fwd(with hash field “MOVED”)，则已经有有线程处理完了了，直接跳过 ，这里是控制并发扩容的核心
3. 当前节点为链表节点或红黑树，重新计算链表节点的hash值，移动到nextTable相应的位置（构建了一个反序链表和顺序链表，分别放置在i和i+n的位置上）。移动完成后，用`Unsafe.putObjectVolatile`在tab的原位置赋为为fwd, 表示当前节点已经完成扩容。

### transfer

1. 如果当前的 `nextTab` 是空，也就是说需要进行扩容的数组还没有初始化，那么初始化一个大小是原来两倍的数组出来，作为扩容后的新数组。 
2. 我们分配几个变量，来把原来的数组拆分成几个完全相同的段，你可以把他们想象成一个个大小相同的短数组，每个短数组的长度是 `stride` 。 
3. 我们先取最后一个短数组，用 `i` 表示一个可变的指针，可指向短数组的任意一个位置，最开始指向的是短数组的结尾。`bound` 表示短数组的下界，也就是开始的位置。也就是我们在短数组选择的时候是采用从后往前进行的。 
4. 然后使用了一个全局的属性 `transferIndex`（线程共享），来记录当前已经选择过的短数组和还没有被选择的短数组之间的分隔。
5. 那么当前的线程选择的这个短数组其实就是当前线程应该进行的数据迁移任务，也就是说当前线程就负责完成这一个小数组的迁移任务就行了。那么很显然在 `transferIndex` 之前的，没有被线程处理过的短数组就需要其他线程来帮忙进行数据迁移，其他线程来的时候看到的是 `transferIndex` 那么他们就会从 `transferIndex` 往前数 `stride` 个元素作为一个小数组当做自己的迁移任务。

### ConcurrentHashMap怎么保证写数据安全

通过自旋锁 + CAS + sychronized + 分段锁

Ⅰ. 先判断散链表是否已经初始化，如果没有初始化则先初始化数组

```java
if(tab == null || (n = tab.length) == 0)
    tab = initTable();
```

Ⅱ. 向桶中加入数据，需要先判断桶中是否为空，如果为空就通过CAS算法将新增数据加入到桶中。

如果写入失败，说明其他线程已经在当前桶位写入了数据，当前线程竞争失败，回到自旋位置，进行等待。

```java
// 如果对应key的哈希值对应table数组下标的位置没有node，则通过cas操作创建一个node放入table
else if((f = tabAt(tab, i = (n - 1) & hash) == null)){
    if(casTabAt(tab,i, null, new Node<K,V>(hash, key, value,null)))
        break;
}
```

Ⅲ. 如果桶中不为空，就需要判断当前桶中头节点的类型：如果桶中头节点值为- 1则表示当前桶位的头节点为fed节点，目前散链表正处于扩容状态，这时候当前线程需要协助扩容。

```java
// 如果table正在扩容，则得到扩容后的table，然后再重新开始一个循环
else if((fh == f.hash) == MOVED) //MOVED = -1
    tab = helpTransfer(tab, f)
```

### 获取size

[ConcurrentHashMap 1.8 计算 size 的方式 - 简书 (jianshu.com)](https://www.jianshu.com/p/971ee45597ac)

```java
//方法一 但不推荐
public int size(){
    long n = sumCount();
    return ((n < 0L) ? 0 : (n > (long)Integer.MAX_VALUE)? Integer.MAX_VALUE):
    (int)n);
}
//更推荐使用这个方法计算
public long mappingCount() {
        long n = sumCount();
        return (n < 0L) ? 0L : n; // ignore transient negative values
    }


//核心方法 sumCount
final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
```

==baseCount就是记录容器数量，为什么sumCount()还要遍历counterCells数组，累加对象的值呢==

```java
//counterCells是个全局的变量，表示的是CounterCell类数组。CounterCell是ConcurrentHashmap的内部类，它就是存储一个值。
private transient volatile CounterCell[] counterCells;
```

JDK1.8中使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据put()或则删除数据remove()时，**会通过addCount()方法更新baseCount:**

```java

```

### 为什么 ConcurrentHashMap 的读操作不需要加锁？

volatile关键字来保证可见性、有序性。但不保证原子性。

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
    
    //Node的值 用了volatile修饰
        volatile V val;
        volatile Node<K,V> next;
```

- 数组用volatile修饰主要是保证在数组扩容的时候保证可见性。

## 容器中的设计模式

### 迭代器模式

![image-20220111200417969](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20220111200417969.png)

Collection实现了Iterable接口，其中的`iterator()`方法能够产生一个Iterator对象，通过这个对象就可以迭代遍历Collection中的元素。

```java
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
for (String item : list) {
    System.out.println(item);
}
```

### 适配器模式

`java.util.Arrays.asList()`可以把数组类型转成List类型

```java
Integer[] arr = {1, 2, 3};
List list = Arrays.asList(arr);
```

