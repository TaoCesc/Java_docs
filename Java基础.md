## 访问权限

- private: 私有的，被其修饰的类、属性、以及方法只能被该类的对象访问，其子类不能访问。
- default： 即不加任何访问修饰符，通常称为“默认访问模式”， 该模式下，只允许在同一个包中进行访问
- protect: 称为“保护型”，被修饰的类 、属性、以及方法只能被类本身的方法即子类访问，即使子类在不同的包也可以访问。
- public：访问限制最宽的修饰符，一般称之为“公共的”，

## 基本数据类型

**整数类型**：byte、short、int、long

**浮点类型**：float、double

**字符型**：char

**布尔型**：boolean

![image-20211120203734534](E:\实习\Java_docs\pics\image-20211120203734534.png)

## 包装类

Byte、Short、Integer、Long、Float、Double、Boolean、Character

```java
Integer num = new Integer(0);  //创建一个数值为0的Integer对象
```

### 自动装箱

```java
Integer num1 = new Integer(1); //基本数据类型转为包装类
int num2 = num1.intValue();   //包装类转为基本数据类型
//等价于
int num2 = num1;  //自动拆箱
```

### 包装类的缓存机制

```java
//2、包装类中的缓存机制
Integer num3 = 10;
Integer num4 = 10;
Integer num5 = new Integer(20);
Integer num6 = new Integer(20);
Integer num7 = 128;
Integer num8 = 128;
System.out.println((num3==num4) +"	"+ num3.equals(num4));
System.out.println((num5==num6) +"	"+ num5.equals(num6));
System.out.println((num7==num8) +"	"+ num7.equals(num8));

//运行结果
true  true
flase true
flase true
```

 如果Integer类第一次被使用，Integer的静态内部类就会被加载，加载的时候创建-128到127的Integer对象，同时创建一个数组cache来缓存这些对象。当使用**valueOf（）**方法创建对象时，就直接返回已经缓存的对象，也就是说**不会再创建对象**；当使用new关键字or使用valueOf()方法创建小于-128大于127的值对象时，就会创建新对象。

由于num3、num4都小于等于127，它们指向的是同一个缓存的Integer对象，所以用==进行比较的结果是true；

num5、num6由于使用new关键字指向的是两个不同的新对象，结果为false；

num7、num8虽然是采用自动装箱的方式，但执行valueOf()方法的时候，由于不满足条件`i >= IntegerCache.low && i <= IntegerCache.high`，而同样新建了两个不同的新对象，结果同样是false。

==为什么需要包装类？==

1. Java中的基本数据类型却是不面向对象的，将每个基本数据类型设计一个对应的类进行代表，这种方式增强了Java面向对象的性质。
2. 在集合类中，我们是无法将int 、double等类型放进去的，因为集合的容器要求元素是Object类型。而包装类型的存在使得向集合中传入数值成为可能，包装类的存在弥补了基本数据类型的不足。
3. 包装类还为基本类型添加了属性和方法，丰富了基本类型的操作。

## 多态

变量的静态类型&动态类型

- 变量的静态类型 = 引用类型 = 编译时变量： 不会被改变、在编译器可知
- 变量的动态类型 = 实例对象类型 = 运行时变量： 会变化、在运行期才可知

```java
public class Test{
    static abstract class Human{
    }
    static class Man extends Human{
    }
    static class Woman extends Human{        
    }
    public static void main(String[] args){
        Human man = new Man();
        //静态类型：Human
        //实例对象类型： Man
    }
}
```

### 重载

静态分派：

​	根据变量的静态类型进行方法分派

```java
//...省略
public void sayHello(Human guy) { 
        System.out.println("hello,guy!"); 
    } 

    public void sayHello(Man guy) { 
        System.out.println("hello gentleman!"); 
    } 

    public void sayHello(Woman guy) { 
        System.out.println("hello lady!"); 
    } 

// 测试代码
    public static void main(String[] args) { 
        Human man = new Man(); 
        Man woman = new Woman(); 
        Test test = new Test(); 

        test.sayHello(man); 
        test.sayHello(woman); 
    } 
}

// 运行结果
hello,guy! 
hello gentleman!
```

### 重写

根据变量的动态类型进行方法分派

```java
// 定义类
    class Human { 
        public void sayHello(){ 
            System.out.println("Human say hello"); 

        } 
    } 

// 继承类Human 并 重写sayHello()
    class Man extends Human { 
        @Override 
        protected void sayHello() { 
            System.out.println("man say hello"); 

        } 
    } 

    class Woman extends Human { 
        @Override 
        protected void sayHello() { 
            System.out.println("woman say hello"); 

        } 
    } 

// 测试代码
    public static void main(String[] args) { 

        // 情况1
        Human man = new man(); 
        man.sayHello(); 

        // 情况2
        man = new Woman(); 
        man.sayHello(); 
    } 
}

// 运行结果
man say hello
woman say hello
```

## Object方法

**getClass**

获取对象运行时class对象

**hashCode**

主要用于获取对象的散列值。Object中该方法默认返回的是**对象的堆内存地址**

**equals**

```java
public boolean equals(Object obj){
    return (this == obj);
}
```

用于比较两个对象，如果这两个对象引用的是**同一个对象**，则返回true，否则返回false。一般 equals 和 == 是不一样的，但是在Object中两者是一样的。子类一般都要重写这个方法。

**clone**

```java
protected native Object clone() throws CloneNotSupportedException;
```

该方法是保护方法，实现对象的浅复制，只有实现了Cloneable接口才可以调用该方法，

默认的clone方法是浅拷贝。指的是对象内属性引用的对象只会拷贝引用地址，而不会将引用的对象重新分配内存。

**toString**

```java
public String toString(){
    return getClass().getName() +"@" + Integer.toHexString((hashCode));
}
```

**notify**

用于唤醒在该对象上等待的某个线程。

**notifyAll**

用于唤醒在该对象上等待的所有线程。

**wait(long timeout)**

```java
public final native void wait(long timeout) throws InterruptedException;
```

使当前线程等待该对象的锁，当前线程必须是该对象的拥有者。

**wait(long timeout, int nanos)**

**wait**

**finalize**

## == 和 equals的区别

### ==

- 引用类型： `==`是直接比较两个对象的堆内存地址，如果相等，则说明两个引用实际上指向同一个对象地址的。
- 基本类型：对于基本类型（8个）和直接声明的`String s1 = "abc";`，都是作为字面量存在常量池中以HashSet策略存储起来的，在常量池中，一个常量只会对应一个地址，所以它们的引用都是指向的同一块地址。

### equals

-  **Object的通用方法**，在没有重写之前，与==是没有区别的；

```java
public boolean equals(Object obj){
    return (this == obj);
}
```

- String和基本类型封装类就重写了equals，从而进行的是内容的比较；
- 一边实现：

```java
public class Student{
    private String num;
    private String name;
    
    @Override
    public boolean equals(Object o){
        if(this == o){
            return true;
        }
        if(o == null || getClass() != o.getClass()){
            return false;
        }
        Student student = (Student) o;
        return Objects.equals(num, student.num) && Objects.equals(name, student.name);
    }
    @Override
    public int hashCode(){
        return Objects.hash(num, name);
    }
}
```

- - 检查是否为同一个对象的引用，如果是直接返回true
  - 检查 是否为空，是否为同一类型，为空或者类型不一致，返回false
  - 将Object对象强制转型
  - 判断每个属性的值是否相等

## String

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
    
    //...
}
```

- String类被final关键字修饰，表示String类不能被继承，并且它的成员方法都默认为final方法。
- String类实现了Serializable、CharSequence、 Comparable接口。
- String类的值是通过char数组存储的，并且char数组被private和final修饰，字符串一旦创建就不能再修改。

**问题一**

上面说字符串一旦创建就不能再修改，String类提供的`replace()`方法不就可以替换修改字符串的内容吗？

`replace()`方法并没有对原字符串进行修改，**而是创建了一个新的字符串返回**

```java
public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */
        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        if (i < len) {
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            //创建一个新的字符串返回
            return new String(buf, true);
        }
    }
    return this;
}
```

==其他方法也是一样，无论是sub、concat还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。==

**问题二**

**为什么要使用final关键字修饰String类？**

String类被final修饰主要基于安全性和效率两点考虑。

- 安全性

因为字符串是不可变的，所以**是多线程安全的**，同一个字符串实例可以被多个线程共享。这样便不用因为线程安全问题而使用同步。字符串自己便是线程安全的。

- 效率

**字符串不变性保证了hash码的唯一性**，因此可以放心的进行缓存，这也是一种性能优化手段，意味着不必每次都取计算新的哈希码。

### 字符串常量池

当创建字符串时，JVM会首先检查字符串常量池，如果该字符串已经存在在常量池中，那么就直接返回常量池中的实例引用。如果字符串不存在于常量池中，就会实例化该字符串并且将其放到常量池中。

```java
String s1 = "abc";
String s2 = "abc";
System.out.println(s1 == s2); //true!
```

<img src="E:\实习\Java_docs\pics\image-20211121154147529.png" alt="image-20211121154147529" style="zoom:70%;" />

 `new String("ABC");`创建了几个对象

如果之前“ABC"字符串没有使用过，创建了两个对象，堆中创建了一个String对象，字符串常量池创建一个。

**StringBuffer**

StringBuffer是**可变类**，和**线程安全**的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。每个StringBuffer对象都有一定的**缓冲区容量**，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量。

**StringBuilder**

StringBuilder是**可变类**，和**线程不安全**的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。每个StringBuilder对象都有一定的缓冲区容量，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量。

### StringBuffer和StringBuilder区别

**StringBuffer**： 线程不安全，执行速度快

**StringBuffer**： 线程安全，执行速度慢

### StringBuffer和StringBuilder的初始容量及扩容

StringBuffer和StringBuilder初始的空闲容量都是16

- StringBuffer()的初始容量可以容纳16个字符，当该对象的实体存放的字符的长度大于16时，实体容量就自动增加。StringBuffer对象可以通过length()方法获取实体中存放的字符序列长度，通过capacity()方法来获取当前实体的实际容量。

- StringBuffer(int capacity)可以指定分配给该对象的实体的初始容量参数为**参数size指定的字符个数**。当该对象的实体存放的字符序列的长度大于size个字符时，实体的容量就自动的增加。以便存放所增加的字符。

- StringBuffer(String str)可以指定给对象的实体的初始容量为参数字符串s的长度**额外再加16个字符**。当该对象的实体存放的字符序列长度大于size个字符时，实体的容量自动的增加，以便存放所增加的字符。

### StringBuffer线程安全

很多方法都是用**synchronized**修饰的

```java
public StringBuffer(CharSequence seq){
    this(seq.length() + 16);
    append(seq);
}
public synchronized int length(){
    return count;
}
public synchronized int capacity(){
    return value.length;
}
```

### String拼接字符串

除了直接使用=赋值，也会用到字符串拼接，字符串拼接又分为变量拼接和已知字符串拼接。

只要拼接内容存在变量，那么该拼接后的新变量就是在堆内存中新建的一个对象实体。

实例：

```java
String str = "abc";//在常量池中创建abc

String str1 = "abcd";//在常量池中创建abcd

String str2 = str+"d";//拼接字符串，此时会在堆中新建一个abcd的对象，因为str2编译之前是未知的

String str3 = "abc"+"d";//拼接之后str3还是abcd，所以还是会指向字符串常量池的内存地址

System.out.println(str1==str2);//false

System.out.println(str1==str3);//true
```

## 接口和抽象类

### 抽象类

1. 由abstract修饰符声明
2. 无法实例化
3. 可以声明抽象方法（即，使用abstract修饰符声明的其他方法）
4. 如果一个类包含一个抽象方法，那么必须将其声明为abstract

```java
public abstract class vehicle{
    private String description;
    public abstract void accelerate();
    //constructors and setters and getters methods
}
```

### 接口

1. 使用interface关键字声明
2. 无法实例化
3. 能够扩展其他接口
4. 一个类可以实现的多个接口之一
5. 能够声明：

- 公共抽象方法 - 不需要使用public 和 abstact修饰符
- 公共默认方法， 即用default修饰符标记的具体方法
- 具体的私有方法 -  只能由默认方法调用
- 公共或私有静态方法 - 编译器将隐式认为没有访问说明符的静态方法是公告的

```java
public interface weighable{
    public static final String UNIT_OF_MEASURE = "kg";
    public abstract double getWeight();
}
//等价于
public interface weighable {
    String UNIT_OF_MEASURE = “kg”;
    double getWeight ( );
}
```

## Finally 

[finally代码块一定会执行吗？_浅末年华的博客-CSDN博客_finally一定会执行吗](https://blog.csdn.net/qq_39135287/article/details/78455525)

## static关键字

static的主要意义在于创建独立于具体对象的域变量或者方法。**以至于即使没有创建对象，也能使用属性和调用方法。**

形成静态代码块以优化程序性能。static块可以置于类中的任何地方，类中可以有多个static块。在类初次被加载的时候，会按照static块的顺序来执行每个static块。



1. 被static修饰的变量或者方法是独立于该类的任何对象，也就是说，这些变量和方法**不属于任何一个实例对象，而是被类的实例对象所共享**。

2. 在该类被第一次加载的时候，就会去加载被static修饰的部分，而且只在类第一次使用时加载并进行初始化，注意这是第一次用就要初始化，后面根据需要是可以再次赋值的。
3. static变量值在类加载的时候分配空间，以后创建类对象的时候不会重新分配。
4. 被static修饰的变量或者方法是优先于对象存在的，也就是说当一个类加载完毕之后，即便没有创建对象，也可以去访问。

### 应用场景

因为static是被类的实例对象所共享，因此如果某个成员变量是被所有对象所共享的话，**那么这个成员变量就应该被定义为静态变量**。

1、修饰成员变量 

2、修饰成员方法 

3、静态代码块 

4、修饰类【只能修饰内部类也就是静态内部类】 

5、静态导包

### 静态变量和实例变量

静态变量： static修饰的成员变量，静态变量属于这个类，而不是属于对象。

实例变量：没有被static修饰的成员变量叫做实例变量，实例变量是属于这个类的实例对象。

==访问方式==

```java
public class StaticDemo {
        static int value = 666;
        public static void main(String[] args) throws Exception{
            new StaticDemo().method();
        }
        private void method(){
            int value = 123;
            System.out.println(this.value);
        }
}
// 结果 666
```

###  执行顺序

基本上代码块分为三种：Static静态代码块、构造代码块、普通代码块

代码块执行顺序**静态代码块——> 构造代码块 ——> 构造函数——> 普通代码块**

继承中代码块执行顺序：**父类静态块——>子类静态块——>父类代码块——>父类构造器——>子类代码块——>子类构造器**

## 类加载器

### 加载器种类

- **启动类加载器**（Bootstrap ClassLoader）：负责将存放在`<JAVA_HOME>\lib`目录中的，并且能被虚拟机识别的类库加载到虚拟机内存中。
- **扩展类加载器**（Extension ClassLoader）：负责加载<JAVA_HOME>\lib\ext目录中的所有类库
- 应用程序类加载器（Application ClassLoader）：
- 由于这个类加载器是 ClassLoader 中的 `getSystemClassLoader()` 方法的返回值，所以一般也称它为“系统类加载器”。它负责加载用户类路径（classpath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

### 双亲委派

双亲委派模型是描述类加载器之间的层次关系。它要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。（父子关系一般不会以继承的关系实现，而是以组合关系来复用父加载器的代码）

#### 工作过程

如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因为所有的**加载请求最终都应该送到顶层的启动类加载器**中，只有当父类加载器反馈自己无法完成这个加载请求，子加载器才会尝试自己去加载。

在java.lang.ClassLoader中的`loadClass`方法中实现该过程

#### 为什么使用双亲委派

像`java.lang.Object`这些存放在 rt.jar 中的类，无论使用哪个类加载器加载，最终都会委派给最顶端的启动类加载器加载，从而使得不同加载器加载的 Object 类都是同一个。

相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为 java.lang.Object 的类，并放在 classpath 下，那么系统将会出现多个不同的 Object 类，Java 类型体系中最基础的行为也就无法保证。

## 反射

Java 反射的主要组成部分有4个：

- **Class**： 任何运行在内存中的所有类都是该Class类的实例对象，每个Class类对象内部都包括了本来的**所有信息**

- **`Field`**：描述一个类的**属性**，内部包含了该属性的所有信息，例如**数据类型，属性名，访问修饰符**

- **`Constructor`**：描述一个类的**构造方法**，内部包含了构造方法的所有信息，例如**参数类型，参数名字，访问修饰符**
- **`Method`**：描述一个类的**所有方法**（包括抽象方法），内部包含了该方法的所有信息，与`Constructor`类似，不同之处是 Method 拥有**返回值类型**信息，因为构造方法是没有返回值的。

![image-20211206002248468](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211206002248468.png)

新建一个类（**属性，构造函数和方法）**

```java
public class SmallPineapple {
    public String name;
    public int age;
    private double weight;

    public SmallPineapple() {
    }

    public SmallPineapple(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void getInfo() {
        System.out.println("[" + name + " 的年龄是：" + age + "]");
    }
}
```

### 获取类的Class对象

获取 Class 对象的方法有`3`种：

- `类名.class`：这种获取方式只有在`编译`前已经声明了该类的类型才能获取到 Class 对象

```java
Class clazz = SmallPineapple.class;
```

- `实例.getClass()`：通过实例化对象获取该实例的 Class 对象

```java
SmallPineapple sp = new SmallPineapple();
Class clazz = sp.getClass();
```

- `Class.forName(className)`：通过类的**全限定名**获取该类的 Class 对象

```java
Class clazz = Class.forName("com.bean.smallpineapple");
```

在程序中，每个类的 Class 对象只有一个通过三种方式打印各个 `Class` 对象都是相同的。

> 内存中只有一个 Class 对象的原因要牵扯到 `JVM 类加载机制`的`双亲委派模型`，它保证了程序运行时，`加载类`时每个类在内存中仅会产生一个`Class对象`。

### 构造类的实例化对象

通过反射构造一个类的实例方法有`2`种：

- Class对象调用`newInstance()`方法

```java
Class clazz = Class.forName("com.bean.SmallPineapple");
SmallPineapple smallpineapple = (SmallPineapple)clazz.newInstance();
smallpineapple.getInfo();
//打印输出
// [null 的年龄是：0]
```

即使 SmallPineapple 已经显式定义了构造方法，通过 newInstance() 创建的实例中，所有属性值都是对应类型的`初始值`，因为 newInstance() 构造实例会**调用默认无参构造器**。

- Constructor构造器调用`newInstance()`方法

```java
Class clazz = Class.forName("com.bean.SmallPineapple");
Constructor constructor = clazz.getConstructor(String.class, int.class);
constructor.setAccessible(true);
SmallPineapple smallpineapple = (SmallPineapple) constructor.newInstance("ABC",21);
smallpineapple.getInfo();
//print
// [小菠萝 的年龄是：21]
```

### 获取一个类的所有信息

**Class 对象中包含了该类的所有信息，在编译期我们能看到的信息就是该类的变量、方法、构造器，在运行时最常被获取的也是这些信息。**

![image-20211206155857123](https://cdn.jsdelivr.net/gh/TaoCesc/blogImages/imgs/image-20211206155857123.png)

#### 获取类中的变量（Field）

- Field[] getFields()：获取类中所有被`public`修饰的所有变量
- Field getField(String name)：根据**变量名**获取类中的一个变量，该**变量必须被public修饰**
- Field[] getDeclaredFields()：获取类中所有的变量，但**无法获取继承下来的变量**
- Field getDeclaredField(String name)：根据姓名获取类中的某个变量，**无法获取继承下来的变量**

#### 获取类中的方法（Method）

- Method[] getMethods()：获取类中被`public`修饰的所有方法
- Method getMethod(String name, Class...<?> paramTypes)：根据**名字和参数类型**获取对应方法，该方法必须被`public`修饰
- Method[] getDeclaredMethods()：获取`所有`方法，但**无法获取继承下来的方法**
- Method getDeclaredMethod(String name, Class...<?> paramTypes)：根据**名字和参数类型**获取对应方法，**无法获取继承下来的方法**

#### 获取类的构造器（Constructor）

- Constuctor[] getConstructors()：获取类中所有被`public`修饰的构造器
- Constructor getConstructor(Class...<?> paramTypes)：根据`参数类型`获取类中某个构造器，该构造器必须被`public`修饰
- Constructor[] getDeclaredConstructors()：获取类中所有构造器
- Constructor getDeclaredConstructor(class...<?> paramTypes)：根据`参数类型`获取对应的构造器

获取SmallPineapple获取类中**所有的方法**

```java
Class clazz = Class.forName("SmallPineapple");
        Constructor constructor = clazz.getConstructor(String.class, int.class);
        constructor.setAccessible(true);
        SmallPineapple smallPineapple = (SmallPineapple) constructor.newInstance("ABC", 21);
        smallPineapple.getInfo();
        Method[] methods = clazz.getMethods();
        Set<Method> set = new HashSet<>();
        set.addAll(Arrays.asList(methods));
        System.out.println(set);
```

### 通过反射调用方法

通过反射获取某个Method类对象后，可以通过调用`invoke`方法执行。

- `invoke(Object obj, Object... args)`:  **obj指定调用该方法的对象**， **args是方法的参数列表值**

如果调用的方法是**静态方法**，参数1只要传入`null`，因为静态方法不与某个对象有关，只与某个类有关。

```java
        Class clazz = Class.forName("SmallPineapple");
        Constructor constructor = clazz.getConstructor(String.class, int.class);
        constructor.setAccessible(true);
        SmallPineapple smallPineapple = (SmallPineapple) constructor.newInstance("ABC", 21);
        smallPineapple.getInfo();
        Method[] methods = clazz.getMethods();
        Set<Method> set = new HashSet<>();
        set.addAll(Arrays.asList(methods));
        System.out.println(set);
        Method method = clazz.getMethod("setName", String.class);
        if(method != null){
            method.invoke(smallPineapple,"CBA");
        }
        smallPineapple.getInfo();
//first print
//[ABC 的年龄是：21]
//modify and print
//[CBA 的年龄是：21]
```

### 反射的应用场景

- **Spring实例化对象**，当程序启动时，Spring会读取配置文件`applicationContext.xml`并解析里面所有的标签实例化到`IOC容器`中。
- **反射+工厂模式**，通过`反射`消除工厂中的多个分支，如果需要生产新的类，无需关注工厂类，工厂类可以应对各种新增的类，`反射`可以使得程序更加健壮。
- **JDBC连接数据库**：使用JDBC连接数据库时，指定连接数据库的`驱动类`时用到反射加载驱动类。

#### Spring的IOC容器

在Spring中，上下文配置文件`applicationContext.xml`，里面就是关于bean的配置，程序启动时会读取该 xml 文件，解析出所有的 `<bean>`标签，并实例化对象放入`IOC`容器中。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="smallpineapple" class="com.bean.SmallPineapple">
        <constructor-arg type="java.lang.String" value="ABC"/>
        <constructor-arg type="int" value="21"/>
    </bean>
</beans>

```

在定义好上面的文件后，通过`ClassPathXmlApplicationContext`加载该配置文件，程序启动时，Spring 会将该配置文件中的所有`bean`都实例化，放入 IOC 容器中，IOC 容器本质上就是一个工厂，通过该工厂传入 <bean> 标签的`id`属性获取到对应的实例。

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext ac =
                new ClassPathXmlApplicationContext("applicationContext.xml");
        SmallPineapple smallPineapple = (SmallPineapple) ac.getBean("smallpineapple");
        smallPineapple.getInfo(); // [ABC的年龄是：21]
    }
}

```

Spring 在实例化对象的过程经过简化之后，可以理解为反射实例化对象的步骤：

- **获取Class对象的构造器**
- 通过构造器**调用newInstance（）**实例化对象

#### 反射+抽象工厂模式

传统的工厂模式，如果需要生产新的子类，**需要修改工厂类，在工厂类中增加新的分支**；

```java
public class MapFactory {
    public Map<Object, object> produceMap(String name) {
        if ("HashMap".equals(name)) {
            return new HashMap<>();
        } else if ("TreeMap".equals(name)) {
            return new TreeMap<>();
        } // ···
    }
}
```

利用反射和工厂模式相结合，在产生新的子类时，**工厂类不用修改任何东西**，可以专注于子类的实现，**当子类确定下来时，工厂也就可以生产该子类了。**

反射 + 抽象工厂的**核心思想**是：

- **在运行时通过参数传入不同子类的全限定名获取到不同的 Class 对象，调用 newInstance() 方法返回不同的子类。**

```java
public class MapFactory {
    public Map<Object, Object> produceMap(String className) {
        Class clazz = Class.forName(className);
        Map<Object, Object> map = clazz.newInstance();
        return map;
    }
}

```

#### JDBC加载数据驱动类

```java
private static final String DRIVER_CLASS_NAME = "com.mysql.jdbc.Driver";
    public static Connection getConnection() throws SQLException, ClassNotFoundException {
        Connection conn = null;
        Class.forName(DRIVER_CLASS_NAME);
        conn = DriverManager.getConnection("jdbc:mysql://···", "root", "root");
        return conn;
    }
```

### 反射的优势及缺陷

优点：

- **增加程序的灵活性**，面对需求变更时，可以灵活地实例化不同对象

缺点：

- **破坏类的封装性**：可以强制访问private修饰的信息
- **性能损耗**：反射相比直接实例化对象、调用方法、访问变量，中间需要非常多的**检查步骤和解析步骤**，JVM对他们无法优化



## 深拷贝与浅拷贝

- 对象拷贝(Object Copy)就是将一个对象的属性拷贝到另一个有着相同类类型的对象中去。

- Java中有三种类型的对象拷贝：浅拷贝(Shallow Copy)、深拷贝(Deep Copy)、延迟拷贝(Lazy Copy)。

### 浅拷贝

- 浅拷贝是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。
  - 如果属性是基本类型，拷贝的就是**基本类型的值**；如果属性是内存地址（引用类型），**拷贝的就是内存地址** ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

### 深拷贝

- 深拷贝会拷贝所有的属性，并拷贝属性指向的动态分配的内存，当对象和它所引用的对象一起拷贝时即发生深拷贝。

