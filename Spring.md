## Spring是什么？有什么特点

1. 方便解耦，简化开发：Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理。
2. AOP编程的支持：Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。
3. 声明式事务的支持： 只需要通过配置就可以完成对事务的管理。
4. 方便程序的测试： Spring对Junit4支持，可以通过注解方便的测试。

5. 方便集成各种优秀框架：Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持（如：Struts、Hibernate、MyBatis等）。

6. 降低JavaEE API的使用难度：Spring对JavaEE开发中非常难用的一些API（JDBC、JavaMail、远程调用等），都提供了封装，使这些API应用难度大大降低。

## Spring的模块组成

1. Spring core: 提供了框架的基本组成部分，包括控制反转（IOC）和依赖注入（DI）功能。
2. spring beans： 提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean
3. spring context： 构建于core封装包基础上的context封装包，提供了一种框架式的对象访问方法。
4. spring jdbc： 提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析， 用于简化JDBC。
5. spring aop：提供了面向切面的编程实现，让你可以自定义拦截器、切点等。
6. spring Web：提供了针对 Web 开发的集成特性，例如文件上传，利用 servlet listeners 进行 ioc 容器初始化和针对 Web 的 ApplicationContext。

7. spring test：主要为测试提供支持的，支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。







## 控制反转（IOC）和依赖注入（DI）

控制反转是一种依赖倒置原则的代码设计的思路，它主要采用依赖注入的方式来实现。

### 不使用IoC思想的传统模式

- 对象由程序员主动创建，**控制权**在程序员手里
- **如果用户需求变更**，程序员就要修改对应的代码
- **耦合性过高**

### 使用IoC思想后的模式

**IoC的主要思想是借助一个“第三方”来拆开原本耦合的对象，并将这些对象都与“第三方”建立联系，由第三方来创建、操作 这些对象，进而达到解耦的目的。**

<img src="E:\实习\Java_docs\pics\image-20211202153408080.png" alt="image-20211202153408080" style="zoom:60%;" />

**因此IoC容器也就成了整个程序的核心，对象之间没有了联系（（但都和IoC容器有联系））**

IoC的思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。第一，资源集中管理，实现资源的可配置和易管理。第二，降低了使用资源双方的依赖程度，也就是我们说的耦合度。

### 什么是控制反转

这里我们引入一个场景， `如果A对象想调用B对象`。

传统模式： 在A对象中创建一个B对象实例，就可以满足A对象调用B对象的需求。**这是我们在A对象中主动的去创建B对象**

而引入IoC后，A对象如果想调用B对象，IoC容器会创建B对象注入到A对象中。这样也可以满足A对象的调用需求。但是过程由我们的主动创建，变成了A对象**被动的去接收IoC容器注入的B对象**

控制权从程序员手中交到了IoC容器手中。A对象获得依赖的过程也由主动变为被动，这就是所谓的**控制反转**

### 什么是依赖注入

依赖注入是IoC思想最重要的**实现方式**

#### 注入方式

##### setter方法注入

我们需要在类中生成一个`set`方法和一个~~空构造~~

```java
public class Hello{
    private String name;
    //一定要有setter方法
    public void setName(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
}
```

在Spring配置文件中注入Bean（对象），在Bean中使用property标签为name属性赋值。

```xml
<bean id="hello" class="com.pojo.Hello">
	<!--setter方法注入使用property标签-->
    <property name="name" value="ABC"/>
</bean>
```

##### 构造器注入

构造器注入需要手动生成一个有参构造

```java
public class Hello {

    private String name;
    // 生成有参构造
    public Hello(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

**这个时候再切回applicationContext.xml中可以看到已经报错了**

*因为显式的定义了有参构造后，无参构造就不存在了*

我们需要将property标签改为**constructor-arg**

通过下标赋值

- ```xml
  <bean id="hello" class="com.molu.pojo.Hello">
      <constructor-arg index="0" value="陌路"/>
  </bean>
  ```

通过参数类型赋值(不推荐，参数类型容易重合)

- ```xml
  <bean id="hello" class="com.molu.pojo.Hello">
      <constructor-arg type="java.lang.String" value="陌路"/>
  </bean>
  ```

通过参数名赋值（推荐）

- ```xml
  <bean id="hello" class="com.molu.pojo.Hello">
      <constructor-arg name="name" value="陌路"/>
  </bean>
  ```



#### 补充

我们再写一个HelloTwo类，里面和Hello类一样，唯一不同是显式的定义了无参构造而不是有参构造

```java
package com.molu.pojo;

public class HelloTwo {
    private String name;
    // 显式的定义了无参构造
    public HelloTwo() {
        // 简单写一个测试输出语句
        System.out.println("HelloTwo的无参构造被调用了");
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

**在Application Context.xml中注册bean，不对其进行其他任何操作。**

```xml
<bean id="helloTwo" class="com.pojo.HelloTwo"></bean>
```

**使用刚刚用过的Hello类测试方法 原封不动进行测试**

```java
public class MyTest {
    @Test
    public void test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Hello hello = (Hello) context.getBean("hello");
        System.out.println(hello.getName());
    }
}
```

***控制台输出了我们在HelloTwo无参构造中写的输出语句，奇怪的是我们并没有在测试类中写任何关于HelloTwo的代码。。***



**由此能够得到一些信息：“注册进applicationContext.xml中的bean，无论你调用与否，他都会被初始化”**

##  AOP编程

面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。

### 代理模式

它分为动态代理和静态代理，代理模式可以使客户端的访问对象从真实对象变为代理对象

**代理模式可以屏蔽用户对真实对象的访问，这样可以避免一些安全上的问题。也能够做到不改变真实对象，对真实对象的功能进行扩展（代理对象实现附加操作进行扩展）。真实对象的功能更加纯粹，业务的分工更加明确。**

==如何实现代理模式==

- 首先需要一个抽象主题（接口或者抽象类）
- 创建代理对象和真实对象
- 代理对象和真实对象都实现该抽象主题
- 客户端访问代理对象

#### 静态代理

引入场景： `我喜欢一双鞋，但在中国地区买不到，需要托朋友从国外代购。`

这里“我”可以理解为客户端、朋友是代理对象、出售鞋的商店为真实对象、抽象主题为卖这双鞋。

```java
// 抽象主题，卖鞋
public interface Subject{
    public void sellShoes();
}
```

```java
// 商店
public class Store implements Subject{
    public void sellShoes(){
        System.out.println("鞋子售价90")；
    }
}
```

```java
// 朋友
public class Friend implements Subject{
    // 朋友拿到商店对象，对应朋友去商店这一场景
    // 代理对象拿到真实对象
    private Store store;
    
    public void setStore(Store store){
        this.store = store;
    }
    
    //代理对象附加对象
    public void returnHome(){
        System.out.println("朋友回到国内，到我家");
    }
    
    public void giveMe(){
        System.out.println("我付了朋友100");
    }
    public void sellShoes(){
         // 朋友在商店里买下了这双鞋子（代理对象调用真实对象的方法）
        store.sellShoes();
        // 朋友回国
        returnHome();
        // 朋友把这双鞋交给我，我付给它相应的费用（含关税）
        giveMe();
    } 
}
```

可以看到，朋友和商店都实现了Subject这个接口。原本我想买到这双鞋应该直接访问商店对象。但因为没办法访问到该对象，我只能通过访问“朋友”对象来实现我拿到这双鞋的需求。

**访问“朋友”对象**

```java
public class Me(){
    public static void main(String[] args){
        //创建真实对象
        Store store = new Store();
        //创建代理对象
        Friend friend = new Friend();
        //将真实对象传给代理对象
        friend.setStore(store);
        
        //调用代理方法
        friend.sellShoes();
    }
}
```

**静态代理模式中，每有一个真实对象 就会有一个代理对象，如果真实对象十分多的话.......😥**

#### 动态代理

```
动态代理可以根据需要，通过反射机制在程序运行时，动态的为目标对象生成代理对象。
动态代理主要分为两大类，一种是基于接口的（JDK），一种是基于类的（CGLIB）
```

##### jdk动态代理

需要了解`java.lang.reflect.Proxy`和`java.lang.reflect.InvocationHandler`这两个类

**InvocationHandler**: 该接口仅定义了一个方法

- `public object invoke(Object proxy, Method method, Object[] args)`
  - 第一个参数为调用该方法的代理实例
  - 第二个参数为目标对象的方法
  - 第三个参数为目标对象方法的参数
- 当我们使用Proxy的静态方法生成动态代理实例后，使用该实例调用接口中的任意方法，都会将调用的方法替换为**invoke方法**

**Proxy：**该类就是为我们生成动态代理的 类

- `static Object newProxyInstanc(ClassLoader loader,Class[] interface,InvocationHandler h)` :

  该静态方法会返回一个Object，

  - 返回的Object就可以被当做代理类使用
  - 三个参数(ClassLoader loader,Class[] interface,InvocationHandler h)
    - loader：一个类加载器对象，我们通过反射来获取目标对象（真实）的类加载器
    - Class[] interface:  接口对象数组，也是通过反射获取的，生成的代理对象会实现这些接口，并可以调用接口中声明的所有方法。
    - h： InvocationHandler的对象实例，如果我们用来的生成代理类 的 类（Friend）实现了这个接口（InvocationHandler），可以直接传入这个类本身（this）。



1. 使用静态代理中的Subject公共主题和Store真实对象
2. 创建一个实现了InvocationHandler接口的类（Friend），它必须实现Invoke方法，在Invoke方法中写附加操作。

```java
//首先是先InvocationHandler接口
Public class Friend implements InvocationHandler{
    //被代理的接口对象
    private Object target;
    
    public Friend(Object target){
        this.target = target;
    }
    
    //写一个获取代理对象实例的方法
    public Object getProxy(){
        // Proxy中的newProxyInstance方法会创建一个动态的代理类
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), target.getClass().getInterfaces(),this);
        //InvocationHandler接口中的invoke方法：
        // 该方法在使用getProxy方法 生成代理类并调用接口中的方法时会被自动调用
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            // 附加操作
        returnHome();
        // method的invoke方法，通过反射获取到目标对象中的方法。
        // 由于我们没有将目标对象写死，所有我们传入动态的target。
        Object object = method.invoke(target,args);
        // 附加操作
        giveMe();
        return object;
    }
    //附加操作
    public void returnHome(){
        System.out.println("朋友回国后来到我家");
    }
    // 同上
    public void giveMe(){

        System.out.println("我付给了朋友指定的钱");
    }
        }
    }
}
```



**测试：**

```java
public class Me {
    public static void main(String[] args) {
        // 创建真实对象
        Store store = new Store();
        // 创建 InvocationHandler对象的实例,并传入目标对象（真实对象）
        Friend friend = new Friend(store);
        // 通过InvocationHandler的实例（friend）调用getProxy方法
        // 该方法会返回一个代理对象的实例，我们只需要将我们写好的Object类型转换为需要的接口类型即可
        Subject proxy = (Subject) friend.getProxy();
        // invoke方法会在我们调用接口中的方法时，将该方法替换为它。
        // invoke方法会通过method.invoke拿到目标对象中的方法
        // 也就是Store中的方法，从而实现代理的操作。
        proxy.sellShoes();
    }
}
```

##### jdk动态代理原理

==分析Friend类中的具体实现==

1. `getProxy`方法，主要使用了Proxy类的**newProxyInstance**方法
   - 返回的对象通过**Proxy的静态方法**生成，生成后就可以被当作一个代理对象来使用。
2. InvocationHandler中的invoke方法
   - 三个参数分别是代理对象的实例，目标对象的方法，方法的参数
   - 我们如果通过getProxy来生成代理实例，使用该实例调用接口中的方法——就会执行invoke方法。
   - invoke通过反射拿到真实对象中的方法。真正执行的也就是这个通过反射拿到的方法。
3. Me类

​	1. 首先我们创建



##### cglib动态代理

```
在jdk动态代理生成的代理对象实例（$Proxy0）的源码中我们看到，jdk动态代理必须要有接口实现才能使用。这就造成了一定的局限性，所以在目标类没有接口实现的情况下我们就会使用cglib动态代理
```

cglib动态代理采用的是继承思想，它针对**类**来实现代理，它会给目标类生成一个对应的子类，并覆盖其方法。

简单来说： 代理类会继承目标类，并重写目标类中的方法（由于使用了继承，所以要避免使用final来修饰目标类

==cglib实现动态代理首选需要准备一个目标对象和一个生成动态代理的类==

**这里我们使用MacStore来充当目标对象，唯一的不同是没有再继承一个公共主题接口。**

```java
public class MacStore{
    public void sellMac(){
        System.out.println("Mac 1800");
    }
}
```

**编写Friend类，写一个生成代理类的方法，重写拦截器方法**

### AspectJ

Spring使用AspectJ提供的用于切入点解析和匹配的库来解释与AspectJ 5相同的注释。但是，AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或编织器。

横切关注点：跨越应用程序多个模块的方法或者功能，即是 与我们业务逻辑毫无关系的部分 也是我们需要关注的部分。如日志、安全、缓存、事务等等.....

切面（ASPECT）：横切关注点 被模块化 的特殊对象。即 它是一个类。

通知（Advice）：切面必须要完成的工作 即 它是类中的一个方法

目标（Target）： 被通知的对象

代理（Proxy）：向目标对象应用通知之后创建的对象

切入点（PointCut）：切面通知执行的"地点"的定义

连接点（JoinPoint）：与切入点匹配的执行点

SpringAOP中， 通过 **Advice(通知) **定义横切逻辑，Spring支持五种类型的Advice

- 前置通知**[Before advice]**：方法（连接点）前执行的通知，它会不阻止执行流程前进到连接点（除非它引发异常）
- 正常返回（后置）通知**[After returning advice]**：方法（连接点）正常执行完后运行的通知（没有引发异常的情况）
- 环绕通知**[Around advice]**：环绕通知围绕在方法（连接点）执行前后运行。这是最强大的通知类型，能在方法调用前后自定义一些操作。
- 异常返回通知**[After throwing advice]**：方法（连接点）抛出异常时运行的通知
- 最终通知**[Final advice]**：在方法（连接点）执行完成后执行的通知，与后置通知不同的是，它会无视抛出异常的情况，即抛出异常仍然会执行该通知，用人话说就是："无论如何都会执行的通知" 。`补充，后置通知可以通过配置得到返回值，而最终通知不行`



**业务接口**

```java
public interface UserService{
    public void add();
    public void delete();
    public void update();
    public void select();
}
```

**接口实现类**

```java
public class UserServiceImpl implements UserService{
    public void add() { System.out.println("增加了一个用户"); }
    public void delete() { System.out.println("删除了一个用户"); }
    public void update() { System.out.println("更新了用户"); }
    public void select() { System.out.println("查询用户"); }

}
```

写一个前置通知，这个通知类只做一件事情：“在我们调用接口实现类中的方法时 打印当前时间和调用的方法名”

```java
```



























## @Component 和 @Bean的区别

1. 作用对象不同：`@Component`注解作用于类，`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来震动侦测以及自动装配到Spring容器中。`@Bean`注解通常是我们在标有该注解的方法中定义产生这个Bean

3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

`@Bean` 注解：

```java
@Configuration
public class AppConfig{
    @Bean
    public TransferService transferService(){
        return new TransferServiceImpl();
    }
}
```

上面代码相当于如下的xml配置

```xml
<beans>
	<bean id = "transferService" class="com.TransferServiceImpl"/>
</beans>
```

下面这个例子是通过 `@Component` 无法实现的。

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

## 讲一个类声明为Spring的bean的注解有哪些

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- `Component`:  通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于拿个层，可以使用`@Component` 注解标注。
- `@Repository`：对应持久层即Dao层，主要用于数据库相关操作。
- `Service`： 对应服务层，主要涉及一些复杂的逻辑，需要用到Dao层。
- `Controller`: 对应Spring MVC控制层，主要接收用户请求并调用Service层返回数据给前端。

## Spring管理事务的方式有几种

1. 编程式事务，在代码中硬编码（不推荐使用）
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

## Spring事务中的隔离级别有哪几种

**TransactionDefinition接口中定义了五个表示隔离级别的常量：**

- **TransactionDefinition.ISOLATION_DEFAULT:** 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.

- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**

- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**

- **TransactionDefinition.ISOLATION_SERIALIZABLE:**   最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## 设计模式在Spring中的应用

### 工厂方法模式

Spring中提供了FactoryBean接口，用于创建各种不同的Bean。

![image-20211201230942908](E:\实习\Java_docs\pics\image-20211201230942908.png)

开发人员也可以自己实现该接口，常用于框架集成。比如SqlSessionFactoryBean就是如此。

### 模板方法模式

Spring针对JDBC、JMS、JPA等规范，都提供了相应的模板方法类，如JdbcTemplate,JmsTemplate, JpaTemplate。 例如JdbcTemplate,它提供了很多常用的增加，删除，查询，修改方法模板。而JMSTemplate则提供了对于消息的发送，接收方法等。

### 代理模式

Spring中AOP，事务等都运用了代理模式

### 观察者模式

Spring中提供了一种事件监听机制，即ApplicationListener，可以实现Spring容器内的事件监听。

主要是以下两个接口：

- 发布消息
- 监听消息

### 单例模式

Spring默认的创建Bean的作用域就是单例，即每个Spring容器中只存在一个该类的实例。可以通过`@Scope("prototype")`来修改成*prototype*模式，*prototype*在设计模式中叫做原型模式，实际上，Spring中对于@Scope(“prototype”)标记的Bean的处理的确是原型模式。

### 职责链模式

在SpringMVC中，当存在多个拦截器（HandlerInterceptor）的时候，所有的拦截器就构成了一条拦截器链。SpringMVC中使用HandlerExecutionChain类来将所有的拦截器组装在一起。

