# ==Java基础知识==

## 1. JVM

- 程序计数器（线程私有）

**是当前线程所执行的字节码的行号指示器**。

- 虚拟机栈（线程私有）

描述java方法执行的内存模型，每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于存储**局部变量表、操作数栈、动态链接、方法出口**等信息。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823190522986.png" alt="image-20210823190522986" style="zoom:67%;" />

- 本地方法区（线程私有）

虚拟机栈为执行Java方法服务，而本地方法栈则为Native方法服务

- 堆（Heap-线程共享） - 运行时数据区

被线程共享的一块内存区域，创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾收集的最重要的内存区域。

- 方法区/永久代（线程共享）

永久代（Permanent Generation），用于存储被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。**运行时常量池（Runtime Constant Pool)**

Java堆从GC角度可以细分为： **新生代**（Eden区、From Survivor区和To Survivor区）和**老年代**

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823195617136.png" alt="image-20210823195617136" style="zoom:67%;" />



- 新生代

用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁触发MinorGC进行垃圾回收。新生代又分为Eden区、From Survivor区和To Survivor区。

**Eden区**：Java新对象的出生地（如果新创建的对象占用内存很大，则直接分配到老年代）。当Eden区内存不够的时候就会触发MinorGC，对新生代区进行一次垃圾回收。

**ServivorFrom**：上一次GC的幸存者，作为这一次GC的被扫描者

**SurvivorTo**：保留了一次MinorGC过程中的幸存者。	

- Minor的过程（复制->清空->互换）

  - eden、servicorFrom 复制到 ServicorTo，年龄 + 1

  - 清空eden、survivorfrom
  - survivorTo和survivorfrom互换，原survivorTo成为下一次GC时的SurvivorFrom区。



- 老年代

主要存放应用程序中生命周期长的内存对象。

在进行MajorGC前一般都先进行了一次MinorGC，使得有新生代的对象晋身老年代，导致空间不够用才触发。

MajorGC采用**标记清除算法**：会产生内存碎片，一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候，就会抛出OOM（out of memory)异常。

- 永久代

指内存的永久保存区域，主要存放Class和Meta的信息，Class在被加载的时候被放入永久区域，它和存放实例的区域不同，**GC不会在主程序运行期间对永久区域进行清理。**。所以这会导致随着加载的Class的增多而胀满，最终抛出OOM异常。

*在Java8中，永久代被移除，被一个称为“元数据区（元空间）的区域所取代。*

**元空间并不在虚拟机中，而是使用本地内存。** 类的元数据放入native momory，字符串池和类的静态变量放入java堆中。

### 垃圾回收和算法

**如何确定垃圾**

- 引用计数法

如果要操作一个对象则必须用引用进行。 即**一个对象如果没有任何与之关联的引用，即他们的引用计数都不为0，则说明对象不太可能再被用到，那么这个对象就是可回收对象。**

- 可达性分析

  为了解决引用计数法的*循环引用问题*，通过一系列的“GC roots”对象作为起点搜索。 如果**在“GC roots”和一个对象之间没有可达路径，则称该对象是不可达的**。

不可达对象变为可回收对象至少要经过两次标记 过程。

#### 标记清除算法（Mark-Sweep）

标记阶段标记出所有需要回收的对象，清除阶段回收被标记的对象所占用的空间。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210824161710126.png" alt="image-20210824161710126" style="zoom:67%;" />

问题：最大的问题是内存**碎片化严重**

#### 复制算法（copying）

按内存容量将内存划分为等大小的两块。每次只使用其中一块，当这一块内存满后将尚存活的对象复制到另一块上，把已使用的内存清掉。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210824161928971.png" alt="image-20210824161928971" style="zoom:67%;" />



问题：可用内存被压缩到了原来的一半，且存活对象增多的话，Copying算法的效率会大大降低。

#### 标记整理算法（Mark-Compact）

标记阶段和Mark-Sweep算法相同，标记后不是清理对象，而是将存活对象移向内存的一端，然后清除端边界外的对象。

- 分代收集算法

其核心思想是根据对象存活的不同生命周期将内存划分为不同的域，一般情况下将GC堆划分为老生代和新生代。

新生代一般采用**Copy算法**，因为新生代中每次垃圾回收都要回收大部分对象，即要复制的操作比较少。

### GC垃圾收集器

Java 堆内存被划分为新生代和年老代两部分，新生代主要使用**复制和标记-清除**垃圾回收算法； 年老代主要使用标**记-整理**垃圾回收算法.





- Serial

  最基本的垃圾收集器，使用复制算法。Serial 是**一个单线程**的收集器，它不但只会使用一个 CPU 或一条线程去完成垃圾收集工 作，并且在进行垃圾收集的同时，必须**暂停其他所有的工作线程**，直到垃圾收集结束。

  

- ParNew（Serial + 多线程）

ParNew其实是Serial的多线程版本

- Parallel Scavenge收集器（多线程复制算法、高效）

同样使用复制算法，它重点关注的是**程序达到一个可控制的吞吐量**（Thoughout，CPU用于运行用户代码的事件/CPU总消耗时间（运行用户代码时间+垃圾回收时间）。

自适应调节







## String

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
    ...
}
```

1) String类被final关键字修饰，不能被继承；字符串一旦创建就不能再修改。

2）String类实现了Serializable、CharSequence、 Comparable接口。

3） String实例是通过字符数组实现字符串存储的。

### 1. ”+“连接符

#### 1.1 实现原理

使用”+“连接字符串对象时，会创建一个StringBuilder（）对象，并调用append（）方法将数据拼接，最后调用toString（）方法返回拼接好的字符串。

### 2.字符串常量池

在Java的内存分配中，总共3种常量池，分别是**Class常量池**、**运行时常量池**、**字符串常量池**。

每当创建字符串常量时，JVM会首先检查字符串常量池，如果该字符串已经存在常量池中，那么就直接返回常量池中的实例引用。如果字符串不存在常量池中，就会实例化该字符串并且将其放到常量池中。由于String字符串的不可变性，**常量池中一定不存在两个相同的字符串**。

```java
/**
 * 字符串常量池中的字符串只存在一份！
 * 运行结果为true
 */
String s1 = "hello world!";
String s2 = "hello world!";
System.out.println(s1 == s2);
```

#### 2.1 内存区域

字符串常量池是通过一个StringTable类实现的，它是一个Hash表，默认值长度大小是**1009**；被所有的类共享。如果String非常多，就会造成Hash冲突严重，从而导致链表会很长。

JDK7版本后，字符串常量池被移到了堆中，StringTable的长度可以通过**-XX:StringTableSize=66666**参数指定。

#### 2.2 存放的内容

在JDK7以后，String Pool中也可以存放放于堆内的字符串对象的引用。

```java
/**
 * 运行结果为true false
 */
String s1 = "AB";
String s2 = "AB";
String s3 = new String("AB");
System.out.println(s1 == s2);
System.out.println(s1 == s3);
```

由于常量池中不存在两个相同的对象，所以s1和s2都是指向JVM字符串常量池中的”AB“对象。new关键字一定会产生一个对象，并且这个对象存储在堆中。所以```java String s3 = new String("AB");```产生了两个对象：保存在栈中的s3和保存在堆中的String对象。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210718234100301.png" alt="image-20210718234100301" style="zoom:80%;" />

当执行String s1 = "AB"时，JVM首先会去字符串常量池中检查是否存在”AB“对象，如果不存在，则在字符串常量池中创建”AB“对象，并将”AB“对象的地址返回给s1；如果存在，则不创建任何对象，直接将字符串常量池中"AB"对象的地址返回给s1。

### 3.intern方法

直接使用双引号声明出来的String对象会直接存储在字符串常量池中，intern方法会从字符串常量池中**查询当前字符串是否存在**，如果存在，就直接返回当前字符串；如果不存在就会将当前字符串放入常量池中，之后再返回。

### 4.String、StringBuilder和StringBuffer

#### 4.1 继承结构

#### 4.2 主要区别

1）String是不可变字符序列，StringBuilder和StringBuffer是可变字符序列。

2）执行速度StringBuilder > StringBuffer > String

3) StringBuilder是非线程安全的，StringBuffer是线程安全的。

## hashcode和equals

### 1.hashCode和equals是什么

hashCode()方法和equals()方法都是在Java中用来对比两个对象是否相等一致。

### 2.**hashCode()和equals()的区别**

- **equals()既然已经能实现对比的功能了，为什么还要hashCode()呢？**

因为重写的equals（）里一般比较的比较全面比较复杂，这样效率就比较低，而利用hashCode()进行对比，则只要生成一个hash值进行比较就可以了，效率很高。

- **hashCode()既然效率这么高为什么还要equals()呢？**

因为hashCode()并不是完全可靠，有时候不同的对象他们生成的hashcode也会一样

### 3.hashCode()和equals使用的注意事项

首先用hashCode（）对比，如果hashCode相同再对比它们的equals

## 装箱拆箱

### 基本数据类型和包装类型的区别

1.包装类是对象，拥有方法和字段，对象的调用都是通过引用对象的地址

2.包装类型是引用的传递；基本类型是值的传递

3.存储位置不同：

​		基本数据类型直接将值保存再值栈中；

​		包装类型是把对象放在堆中，通过对象的引用来调用

4.声明方式不同：

​        基本数据类型不需要new关键字；

​        包装类型需要new在堆内存中进行new来分配内存空间 

###  装箱拆箱

给一个Integer对象赋一个int值的时候，java会自动调用Integer.ValueOf()方法，不需要调用构造方法，通过```=```Zion给把基本类型转换成类类型就叫装箱

```java
int i = 5;
Integer k = i;
```

拆箱：

```java
Integer i = new Integer();
int k = i;
```

## 接口和抽象类

### 接口

**一个类只能继承一个抽象类，但却可以实现多个接口**

#### 1.接口是什么

接口是通过interface关键字定义的，它可以包含一些常量和方法

```java
public interface Electronic{
    //常量
    String LED = "LED";
    //抽象方法
    int getElectricityUse();
    //静态方法
    static boolean isEnergyEfficient(String electtronicType) {
        return electtronicType.equals(LED);
        // 默认方法
    default void printDescription() {
        System.out.println("电子");
    }
}
```

1) 接口在定义的变量会在编译的时候自动加上```public static final```修饰符，也就是说LED变量其实是一个常量。

2）没有使用 `private`、`default` 或者 `static` 关键字修饰的方法是隐式抽象的，在编译的时候会自动加上 `public abstract` 修饰符。也就是说 `getElectricityUse()` 其实是一个抽象方法，没有方法体——这是定义接口的本意。

#### 2.定义接口

- 接口中允许定义变量
- 接口中允许定义抽象方法
- 接口中允许定义静态方法（Java 8 之后）
- 接口中允许定义默认方法（Java 8 之后）

1. 需要定义一个类去实现接口，然后再实例化

```java
public class Computer implements Electronic{
    public static void main(String[] args){
        new Computer();
    }
    
    @Override
    public int getElectricityUse(){
        return 0;
    }
}
```

2. 接口可以是空的，既不定义变量，也不定义方法
3. 不要在定义接口的时候使用 final 关键字
4. 接口的抽象方法不能是 private、protected 或者 final

**Java 原则上只支持单一继承，但通过接口可以实现多重继承的目的**

![image-20210719181259857](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210719181259857.png)

ClassC 同时继承了 ClassA 和 ClassB，ClassC 的对象在调用 ClassA 和 ClassB 中重载的方法时，就不知道该调用 ClassA 的方法，还是 ClassB 的方法

**实现多态**

### 3.接口与抽象类的区别

1）语法上：

- 接口中不能有public和protected修饰的方法，抽象类可以有
- 接口中的变量只能是隐式的常量，抽象类中可以有任意类型的变量。
- 一个类只能继承一个抽象类，但却可以实现多个接口

2）设计层面：

抽象类是对类的一个抽象，继承抽象类的类和抽象类本身是一种**```is-a```**的关系

接口时对类的某种行为的一种抽象，接口和类之间并没有很强的关联关系

## BIO NIO AIO

[2020年，如何理解BIO NIO AIO？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/111816019)

**同步、异步**描述的是：**客户端在请求数据的过程中，能否做其他事情。**

**阻塞、非阻塞**描述的是：**客户端与服务端是否从头到尾始终都有一个持续连接，以至于占用了通道，不让其他客户端成功连接。**

- BIO（同步阻塞）：客户端在请求数据的过程中，**保持一个连接**，**不能做其他事情**。
- NIO（同步非阻塞）：客户端在请求数据的过程中，**不用保持一个连接**，不能做其他事情。（不用保持一个连接，而是用许多个小连接，也就是轮询）

- AIO（异步非阻塞）：客户端在请求数据的过程中，不用保持一个连接，**可以做其他事情**。（客户端做其他事情，数据来了等服务端来通知。）
  - 同步的意思是：客户端与服务端 相同步调。就是说 服务端 没有把数据给 客户端 之前，客户端什么都不能做。它们做同样一件事情，就是说它们有相同步调，即同步。
  - 阻塞的意思是：客户端与服务端之间是否始终有个东西占据着它们中间的通道。就是说 客户端与服务端中间，始终有一个连接。导致其他客户端不能继续建立新通道连接服务器。

### 1. 能处理什么问题

**BIO（同步阻塞）**

定义：客户端在请求数据的过程中，**保持一个连接**，**不能做其他事情**。

由于**连接是双向**的，“始终保持一个连接”，则说明，对于客户端和服务端而言，都需要一个线程来维护这个连接，如果服务端没有数据给客户端，则客户端需要一直等待，该连接也需要一直维持。

**NIO（同步非阻塞）**

定义：客户端在请求数据的过程中，**不用保持一个连接**，不能做其他事情。

上面提到BIO，当有很多个客户端同时向服务端请求数据时，其连接所花费的开销就极大。

客户端发送一个请求，并建立一个连接，服务端接收到了。如果服务端没有数据，就告知客户端“没有数据”；如果有数据，则返回数据。客户端接到了服务端回复的“没有数据”就断开连接，过了一段时间后，客户端重新问服务端是否有数据。服务器重复以上步骤。

客户端反复建立连接询问，如果没有数据则断开连接。这个过程称为“轮询”。**NIO用轮询代替了始终保持一个连接。**

**AIO（异步非阻塞）**

定义：客户端在请求数据的过程中，不用保持一个连接，**可以做其他事情**

AIO用了一个通知机制，其流程如下：

客户端向服务端请求数据。服务端若有，则返回数据；若无，则告诉客户端“没有数据”。客户端收到“没有数据”的回复后，就做自己的其他事情。服务端有了数据之后，就主动通知客户端，并把数据返回去

**但是**服务端需要主动通知客户端，关于“通知”的业务逻辑肯定是需要消耗资源的。客户端本来在做别的事情，突然前面的事情又插过来要做了，必然引入了一个多线程的协调工作。

## JVM的内存划分

- 堆：存放对象
- 方法区：存放已被加载的类信息、常量、静态变量、JIT编译后的代码
- 虚拟机栈：
- 本地方法栈：为native方法服务
- 程序计数器

## JAVA中的引用

### 1. 值类型和引用类型

- 变量初始化

```java
int num = 10;
String str = "hello"
```

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210818143145627.png" alt="image-20210818143145627" style="zoom:50%;" />

- 变量赋值

num是int基本类型变量，值就直接保存在变量中。

str是String引用类型变量，变量中保存的只是实际对象对应的地址信息，而不是实际对象数据。

```java
num = 20;
str = "java"
```

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210818143257093.png" alt="image-20210818143257093" style="zoom:50%;" />

对于基本类型变量num，赋值运算符会直接修改变量的值，原来的数据会被覆盖，被替换为新的值。引用类型只会改变变量中所保存的对象的地址信息。

### 2.数据存储方式

- 局部变量/方法参数

在jvm中的存储方式是相同的，都是存储在栈上开辟的空间中。

当在方法中声明一个int变量i=0或Object变量obj=null时，此时仅仅在**栈上分配空间**，不影响到**堆空间**。**当new Object()时**，将会在堆中开辟一段内存空间并初始化Object对象。

- 数组类型引用和对象

当声明数组时，```int[] arr=new int[2]```; 数组也是对象，arr实际上是引用，栈上占用4个字节大小的存储空间. 会在堆中开辟相应大小空间进行存储，然后arr变量指向它。

```int[][] arr2 = new int[2][4]```

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210818143917477.png" alt="image-20210818143917477" style="zoom:50%;" />

所以当传递一个数组给一个方法时，数组的元素在方法内部是可以被修改的，但是无法让数组引用指向新的数组。

- String类型数据

对象内部需要维护3个成员变量，```char[] chars, int startIndex, int length```。

char是存储字符串数据的真正位置，例如```String str = new String("hello")```,内存分布如下：

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210818150549355.png" alt="image-20210818150549355" style="zoom:50%;" />

### 3.JAVA引用类型

[(2条消息) JAVA四种引用方式_我是一个包子-CSDN博客_java引用](https://blog.csdn.net/u014086926/article/details/52106589)

[JAVA中的引用 - 追求沉默者 - 博客园 (cnblogs.com)](https://www.cnblogs.com/czx1/p/10665327.html)

在JAVA中提供了四种引用类型：强引用、软引用、软引用和虚引用。在四种引用类型中，**只有强引用FinalReference类型变量是包内可见的**，其他三种引用类型均为public，可以在程序中直接使用。

#### 1）强引用

强引用是使用最普遍的引用。如果一个对象具有强引用，那么垃圾回收器绝不会回收它。

例如:StringBuilder sb = new StringBuilder("test");变量str指向StringBuffer实例所在的堆空间，通过str可以操作该对象。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210818150912650.png" alt="image-20210818150912650" style="zoom:50%;" />

强引用特点：

- 强引用可以直接访问目标对象
- 只要有引用变量存在，垃圾回收器永远不会回收
- 强引用可能导致内存泄漏问题

#### 2）软引用

一个持有软引用的对象，不回被JVM很快回收，JVM会根据当前堆的使用情况来判断何时回收。当堆使用率**临近阈值**时，才会去回收软引用的对象。

软引用可用来实现内存敏感的高速缓存，比如<font color='red'>网页缓存、图片缓存</font>等。**使用软引用可以防止内存泄漏，增强程序的健壮性。**

另外，一旦垃圾线程回收该java对象后，get（）方法将返回null

```java
MyObject aRef = new MyObject();
SoftReference aSoftRef = new SoftReference(aRef);
```

对于这个MyObject对象，有两个引用路径，一个是来自SoftRefence对象的软引用，一个来自

aReference的强引用，所以这个MyObject对象是强引用对象。

随即，我们结束aReference对这个MyObject实例的强引用：

```java
aRef = null
```

此后，这个MyObject对象成了一个软引用对象。

#### 3）弱引用

弱引用也是用来描述非必需对象，当JVM进行垃圾回收时，无论内存是否充足，都会回收被弱引用关联的对象。

```java
public static void main(String[] args) {
		WeakReference<People>reference=new WeakReference<People>(new People("zhouqian",20));
		System.out.println(reference.get());
		System.gc();//通知GVM回收资源
		System.out.println(reference.get());
	}
}
class People{
	public String name;
	public int age;
	public People(String name,int age) {
		this.name=name;
		this.age=age;
	}
	@Override
	public String toString() {
		return "[name:"+name+",age:"+age+"]";
	}
}
```

Input:

```
[name:zhouqian,age:20]
null
```

说明只要JVM进行垃圾回收，被弱引用关联的对象必定会被回收掉。

**这里所说的被弱引用关联的对象是指只有弱引用与之关联，如果存在强引用同时与之关联，则进行垃圾回收时也不会回收该对象（软引用也是如此）。**

```java
WeakReference<People>reference=new WeakReference<People>(new People("zhouqian",20)); 
//改成如下语句
People people = new People("zhouqian",20);
WeakReference<People> reference = new WeakReference<>(people);
//则不再会被GC回收
```

#### 4）虚引用

PhantomReference

如果一个对象与虚引用关联，则跟没有引用与之关联一样，在任何时候都可能被垃圾回收器回收。

要注意的是，虚引用必须和引用队列关联使用，当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会把这个虚引用加入到与之关联的引用队列中。

```java
public static void main(String[] args) {
        ReferenceQueue<String> queue = new ReferenceQueue<String>();
        PhantomReference<String> pr = new PhantomReference<String>(new String("hello"), queue);
        System.out.println(pr.get());
    }
```

