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

- `static Object newProxyInstance(ClassLoader loader,Class[] interface,InvocationHandler h)` :

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



## BeanFactory 和 ApplicationContext 的不同点

### BeanFactory接口

这是一个用来访问Spring容器的root接口，要访问Spring容器，我们将使用Spring依赖注入功能，使用BeanFactory接口和它的子接口特性：

- **Bean的实例化、串联**   BeanFactory 的实现是使用懒加载的方式，这意味着 beans 只有在我们通过 getBean() 方法直接调用它们时才进行实例化 实现 BeanFactory 最常用的 API 是 XMLBeanFactory 这里是如何通过 BeanFactory 获取一个 bean 的例子：

```java
public class HelloWorldApp{
    XmlBeanFactory factory = new XmlBeanFactory(new ClassPathResource("beans.xml"));
    HelloWorld obj = (HelloWorld)factory.getBean("helloWorld");
    obj.getMessage();
}
```

### ApplicationContext接口

ApplicationContext是Spring应用程序中的中央接口， 用于向应用程序提供配置信息 它继承了BeanFactory接口，所以ApplicationContext包含BeanFactory的所有功能以及更多功能 。它的主要功能是支持大型的业务应用的创建 **特性：** 

- Bean instantiation/wiring
- Bean的实例化/串联
- 自动的BeanPostProcessor注册
- 自动的BeanFactoryPostProcessor注册
- 方便的MessageSource访问

- **ApplicationEvent 的发布 与 BeanFactory 懒加载的方式不同，它是预加载，所以，每一个 bean 都在 ApplicationContext 启动之后实例化 **

```java
public class HelloWorldApp{ 
   public static void main(String[] args) { 
      ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml"); 
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");    
      obj.getMessage();    
   }
}
```

> ApplicationContext 包含 BeanFactory 的所有特性，通常推荐使用前者。

## 拦截器

### HandlerInterceptor

通常可以通过实现HandlerInterceptor接口进行自定义拦截器的定义。

```java
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class TestInterceptor implements HandlerInterceptor {
    @Override
    public void afterCompletion(HttpServletRequest request,
								HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
		// afterCompletion方法在控制器的处理请求方法执行完成后执行，即视图渲染结束之后执行
        System.out.println(" ======= >>>>  afterCompletion 方法执行");

    }

    @Override
    public void postHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
		// postHandle方法在控制器的处理请求方法调用之后，解析视图之前执行
		System.out.println(" ======= >>>>  postHandle   方法执行");
    }

    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {
		// preHandle方法在控制器的处理请求方法调用之后，解析视图之前执行
		System.out.println(" ======= >>>>  preHandle   方法执行");
        return false;
    }
}
```

在上述拦截器的定义实现了HandlerInterceptor接口：

- **preHandle方法**： 该方法在控制器的处理请求方法前执行，其返回值表示是否中断后续操作，返回true表示继续向下执行，返回false表示中断后续操作。
- **postHandle方法**： 该方法在控制器的处理请求方法调用之后、解析视图之前执行，可以通过此方法对请求域中的模型和视图做进一步的修改。
- **afterCompletion方法**： 该方法在控制器的处理请求方法执行完成后执行，即视图渲染结束后执行，可以通过此方法实现一些资源清理、记录日志信息等工作。

### 拦截器的配置

#### XML方式

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <!-- 配置一个全局拦截器，拦截所有请求 -->
    <bean class="interceptor.TestInterceptor" /> 
    <mvc:interceptor>
        <!-- 配置拦截器作用的路径 -->
        <mvc:mapping path="/**" />
        <!-- 配置不需要拦截作用的路径 -->
        <mvc:exclude-mapping path="" />
        <!-- 定义<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
        <bean class="interceptor.Interceptor1" />
    </mvc:interceptor>
    <mvc:interceptor>
        <!-- 配置拦截器作用的路径 -->
        <mvc:mapping path="/gotoTest" />
        <!-- 定义在<mvc: interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
        <bean class="interceptor.Interceptor2" />
    </mvc:interceptor>
</mvc:interceptors>
```

#### 注解方式

```java
@Configuration
public class TestConfiguration implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new TestInterceptor()).addPathPatterns("/app/*");
    }
}
```

### 单个拦截器的执行过程

运行程序时，拦截器的执行时是有一定顺序的，该顺序与配置文件中所定义的拦截的顺序相关。

![image-20211203150303610](E:\实习\Java_docs\pics\image-20211203150303610.png)

程序首先执行拦截器类中的preHandle（）方法，如果该方法返回的是true， 则程序会继续向下执行处理器中的方法，否则不再向下执行； 在业务控制类Controller处理完请求后，会执行postHandle（）方法，而后会通过DispathcerServlet向客户端返回响应；在DispathcherServlet处理完请求后，才会执行afterCompletion（）方法。

### 多个拦截器的执行流程

当多个拦截器同时工作时，它们的preHandle()方法会按照配置文件中拦截器的配置顺序执行，而它们的postHandle()方法和afterCompletion()方法则会按照配置顺序的反序执行

### 预备知识

#### BeanDefinition

BeanDefinition是用来描述一个Bean的，直译过来就是Bean的定义。可以看到接口定义了各种方法来描述Bean的各种属性。Spring容器在初始化Bean的时候，不是直接从配置文件到Bean的，而是从配置文件解析为BeanDefinition，然后根据BeanDefinition实例化Bean。

#### BeanFactoryPostProcessor

针对BeanFactory的后置处理器。接口只有一个方法，入参就是beanFactory。在postProcessBeanFactory()方法中可以对beanFactory进行各种修改。这是Spring对外提供的修改beanFactory的机会。实现了这个接口的类会在ApplicationContext实例化的过程中被调用，实现修改。

```java
@FuncationalInterface
public interface BeanFactoryPostProcessor{
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

#### BeanPostProcessor

针对Bean的后置处理器。这是Spring对外提供的修改已实例化bean的机会。在bean初始化前、后各有一个回调方法，默认是不做任何操作，可以实现此接口，然后做自己想要的操作。

后置处理器机制在Spring中使用非常广泛。熟知的AOP就是利用了这个机制。

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

### 监听器引导初始化Spring容器

ServletContext初始化完成后，通知监听器。ContextLoaderListener进行Spring容器的初始化操作。主要逻辑在`COntextLoader`的`initWebApplicationContext()`方法完成。

<img src="E:\实习\Java_docs\pics\image-20211203125407398.png" alt="image-20211203125407398" style="zoom:80%;" />

#### 获取容器类





















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





## Spring的事务实现原理和传播机制

[美团二面：Spring的@Transactional如何实现的？ - 掘金 (juejin.cn)](https://juejin.cn/post/6987924251751743518#comment)

1. 编程式事务，在代码中硬编码（不推荐使用）
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解`@Transactional`的声明式事务

### @Transactional注解

`@Transactional`是spring中声明式事务管理的注解配置方式，`@Transactional`注解可以帮助我们把事务开启、提交或者回滚的操作，通过aop的方式进行管理。

通过`Transactional`注解就能让spring为我们管理事务，免去了重复的事务管理逻辑，减少对业务代码的侵入。

![image-20211202225411976](E:\实习\Java_docs\pics\image-20211202225411976.png)

实现@Transactional原理是基于spring aop，aop又是动态代理模式的实现





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

## JdbcTemplate常用方法

`JdbcTemplate`是`Spring JDBC`的核心类，借助该类提供的方法可以很方便的实现数据的增删改查。

`Spring`对数据库的操作在`jdbc`上面做了深层次的封装，使用spring的注入功能，可以把`DataSource`注册到`JdbcTemplate`之中。

其全限定命名为`org.springframework.jdbc.core.JdbcTemplate`。要使用`JdbcTemlate`还需一个这个包包含了事务和异常控制

### JdbcTemplate主要提供以下五类方法：

1.  **execute方法**：可以用于执行任何SQL语句，一般用于执行DDL语句；
2.  **update方法及batchUpdate方法**：`update`方法用于执行新增、修改、删除等语句；`batchUpdate`方法用于执行批处理相关语句；
3.  **query方法及queryForXXX方法**：用于执行查询相关语句；
4.  **call方法**：用于执行存储过程、函数相关语句。

### xml中的配置

```xml
<!-- 扫描 -->
<context:component-scan base-package="com.zzj.*"></context:component-scan>

<!-- 不属于自己工程的对象用bean来配置 -->
<!-- 配置数据库连接池 -->
<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close" p:username="root" p:password="qw13579wq">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3306/test"></property>
</bean>

<!-- 配置jdbcTemplate -->
<bean class="org.springframework.jdbc.core.JdbcTemplate" p:dataSource-ref="dataSource"></bean>
```

```java
package com.zzj.vo;

public class UserInfo {

    private int id;
    private String userName;
    private String password;

    // getter and setter

    @Override
    public String toString() {
        return "UserInfo [id=" + id + ", userName=" + userName + ", password=" + password + "]";
    }
    
}
```

```java
@Repository
public class UserInfoDao {

    @Autowired
        //从容器中自动扫描获取jdbcTemplate
    private JdbcTemplate jdbcTemplate;

        //update()实现增加数据
    public boolean insert(int id,String userName,String password){
        String sql = "insert into user_info values (?,?,?)";
        return jdbcTemplate.update(sql,id,userName,password)>0;
    }
    
    //update()实现修改
    public boolean update(int id,String userName,String password){
        String sql = "update user_info set user_name=?,password=? where id=?";
        return jdbcTemplate.update(sql,userName,password,id)>0;
    } 
    
    //update()实现删除
    public boolean delete(int id){
        String sql = "delete from user_info where id=?";
        return jdbcTemplate.update(sql,id)>0;
    }

}
```

## autowired作用

- `@Autowired`: 自动装配，默认按照Bean的类型进行装配，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false

- `@Resource`：与`@Autowired`类似，但是是按照名称进行装配，当找不到与名称匹配的Bean时才按照类型进行装配，这是JDK的注解，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。
- @Bean：方法上的注解，用于产生一个Bean，然后交由Spring管理
- @Component：表示一个组件对象，加上了该注解就能实现自动装配，默认的Bean的id为使用小驼峰命名法的类

## Bean的作用域和生命周期

### Bean作用域

`<bean>`中的`scope`可以指定的作用域如下：

- `singleton`：默认作用域，在`Spring`容器只有一个`Bean`实例
- `prototype`：每次获取`Bean`都会返回一个新的实例
- `request`：在一次`HTTP`请求中只返回一个`Bean`实例，不同`HTTP`请求返回不同的`Bean`实例，仅在`Spring Web`应用程序上下文使用
- `session`：在一个`HTTP Session`中，容器将返回同一个`Bean`实例，仅在`Spring Web`应用程序上下文中使用
- `application`：为每个`ServletContext`对象创建一个实例，即同一个应用共享一个`Bean`实例，仅在`Spring Web`应用程序上下文使用
- `websocket`：为每个`WebSocket`对象创建一个`Bean`实例，仅在`Spring Web`应用程序上下文使用

### Bean生命周期

`Spring`可以管理作用域为`singleton`的生命周期，在此作用域下`Spring`能精确知道`Bean`何时被创建，何时初始化完成以及何时被摧毁。`Bean`的整个生命周期如下：

1. 实例化`Bean`

2. 进行依赖注入

3. 如果`Bean`实现了`BeanNameAware`，调用`setBeanName`

4. 如果`Bean`实现了`BeanFactoryAware`，调用`setBeanFactory`

5. 如果`Bean`实现了`ApplicationContextAware`，调用`setApplicationContext`

6. 如果`Bean`实现了`BeanPostProcessor`，调用`postProcessBeforeInitialization`

7. 如果`Bean`实现了`InitializingBean`，调用`afterPropertiesSet`

8. 如果配置文件配置了`init-method`属性，调用该方法

9. 如果实现了`BeanPostProcessor`，调用`postProcessAfterInitialization`，注意接口与上面的相同但是方法不一样

10. 不需要时进入销毁阶段

11. 如果`Bean`实现了`DisposableBean`，调用`destroy`如果配置文件配置了`destroy-method`，调用该方法

    

```java
public class TestBean implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, BeanPostProcessor, DisposableBean {
    public TestBean() {
        System.out.println("调用构造方法");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("调用BeanFactoryAware的setBeanFactory方法");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("调用BeanNameAware的setBeanName方法");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("调用ApplicationContextAware的setApplicationContext方法");
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用BeanPostProcessor的postProcessBeforeInitialization");
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用BeanPostProcessor的postProcessAfterInitialization");
        return null;
    }

    public void initMethod() {
        System.out.println("调用XML配置中的init-method");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("调用DisposableBean的destroy方法");
    }

    public void destroyMethod(){
        System.out.println("调用XML配置的destroy-method");
    }
}
```

配置文件如下，指定了`init-method`以及`destroy-method`：

```xml
<bean id="testBean" class="TestBean" init-method="initMethod" destroy-method="destroyMethod"/>
复制代码
```

测试：

```java
public static void main(String[] args) {
    ConfigurableApplicationContext context = new FileSystemXmlApplicationContext("classpath:applicationContext.xml");
    TestBean test = (TestBean) context.getBean("testBean");
    ((BeanDefinitionRegistry) context.getBeanFactory()).removeBeanDefinition("testBean");
}
```

输出：

![image-20211203160732594](E:\实习\Java_docs\pics\image-20211203160732594.png)

如果没有最后一行的手动删除`Bean`定义是不会看见最后两行的输出的

## Spring如何解决循环依赖

![image-20211203162917823](E:\实习\Java_docs\pics\image-20211203162917823.png)

```java
public class A{
    private B b;
}
public class B{
    private A a;
}
**********************
<bean id="beanA" class="xyz.coolblog.BeanA">
    <property name="beanB" ref="beanB"/>
</bean>
<bean id="beanB" class="xyz.coolblog.BeanB">
    <property name="beanA" ref="beanA"/>
</bean>
```

IOC 按照上面所示的 <bean> 配置，实例化 A 的时候发现 A 依赖于 B 于是去实例化 B（此时 A 创建未结束，处于创建中的状态），而发现 B 又依赖于 A ，于是就这样循环下去，最终导致 OOM

### 循环依赖发生的时机

> Bean实例化主要分为三步：

<img src="E:\实习\Java_docs\pics\image-20211203163436558.png" alt="image-20211203163436558" style="zoom:80%;" />

问题出现在：第一步和第二步的过程中，也就是填充属性 / 方法的过程中

### Spring如何解决的

- Spring为了解决单例的循环依赖问题，使用了三级缓存，递归调用时发现Bean还在创建中即为循环依赖。
- 单例模式的 Bean 保存在如下的数据结构中：

```java
/** 一级缓存：用于存放完全初始化好的 bean **/
private final Map<String, Object> singletonObjects = new ConcurrentHash<String, Object>(256);
/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
/** 三级缓存：存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
/**
bean 的获取过程：先从一级获取，失败再从二级、三级里面获取

创建中状态：是指对象已经 new 出来了但是所有的属性均为 null 等待被 init
**/
```

检测循环依赖的过程如下：

- A创建过程中需要B，于是**A将自己放到三级缓存中**，去实例化B
- B实例化的时候发现需要A，于是B先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到A
  - **然后把三级缓存里面的这个A放到二级缓存里面，并删除三级缓存里的A**
  - B顺利初始化后，将自己放到**一级缓存**中（此时B里面的A依然时创建中状态）
- 然后回来接着创建A，此时B已经创建结束，直接从一级缓存里面拿到B，然后完成创建，**并将自己放到一级缓存里面**。

### 一级缓存能解决吗？

<img src="E:\实习\Java_docs\pics\image-20211203193529311.png" alt="image-20211203193529311" style="zoom:85%;" />

- 因为 A 的成品创建依赖于 B，B的成品创建又依赖于 A，当需要补全B的属性时 A 还是没有创建完，所以会出现死循环。

### 二级缓存能解决吗

<img src="E:\实习\Java_docs\pics\image-20211203193613101.png" alt="image-20211203193613101" style="zoom:85%;" />

- 有了二级缓存其实这个事处理起来就容易了，一个缓存用于存放成品对象，另外一个缓存用于存放半成品对象。
- A在创建半成品对象后存放到缓存中，接下来补充A对象中依赖B的属性。
- B 继续创建，创建的半成品同样放到缓存中，在补充对象的 A 属性时，可以从半成品缓存中获取，现在 B 就是一个完整对象了，而接下来像是递归操作一样 A 也是一个完整对象了。



==三级缓存==

有了二级缓存都能解决 Spring 依赖了，怎么要有三级缓存呢。其实我们在前面分析源码时也提到过，三级缓存主要是解决 Spring AOP 的特性。AOP 本身就是对方法的增强，是 `ObjectFactory<?>` 类型的 lambda 表达式，而 Spring 的原则又不希望将此类类型的 Bean 前置创建，所以要存放到三级缓存中处理。

其实整体处理过程类似，唯独是 B 在填充属性 A 时，先查询[成品缓存](https://www.zhihu.com/search?q=成品缓存&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1908173247})、再查半成品缓存，最后在看看有没有单例工程类在三级缓存中。最终获取到以后调用 getObject 方法返回代理引用或者原始引用。

## Spring启动流程

spring的启动过程其实就是其IoC容器的启动过程，对于web程序，IoC容器启动过程即是建立上下文的过程。

首先看web.xml中的配置：

```xml
<servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
    </context-param>
<listener>      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
=================分析===========================================
<context-param>
        <param-name>contextConfigLocation</param-name>
       <param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
    </context-param>
<listener>      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
```

这段是加载spring配置文件，初始化上下文，ContextLoaderListener是一个实现了ServletContextListener接口的监听器，在启动项目时会触发contextInitialized方法（该方法主要完成ApplicationContext对象的创建），在关闭项目时会触发contextDestroyed方法（该方法会执行ApplicationContext清理操作）

1. 启动项目时触发contextInitialized方法，通过父类contextLoader得initWebApplicationContext方法创建Spring上下文对象。
2. initWebApplicationContext方法做了三件事： ①创建WebApplicationContext； ②加载对应的Spring文件创建里面的Bean实例； ③将WebApplicationContext放入ServletContext（就是Java Web的全局变量中）。
3. createWebApplicationContext创建上下文对象，支持用户自定义的上下文对象，但是必须继承自ConfigurableWebApplicationContext，而Spring MVC默认使用ConfigurableWebApplicationContext作为ApplicationContext（它仅仅是一个接口）的实 现。
4. configureAndRefreshWebApplicationContext方法用 于封装ApplicationContext数据并且初始化所有相关Bean对象。它会从web.xml中读取名为 contextConfigLocation的配置，这就是spring xml数据源设置，然后放到ApplicationContext中，最后调用传说中的refresh方法执行所有Java对象的创建。
5. 完成ApplicationContext创建之后就是将其放入ServletContext中，注意它存储的key值常量。

### DispatcherServlet初始化顺序

1. HttpServletBean继承HttpServlet，因此在Web容器启动时将调用它的init方法，该初始化方法的主要作用：将Servlet初始化参数（init-param）设置到该组件上（如contextAttribute、contextClass、namespace、contextConfigLocation），通过BeanWrapper简化设值过程，方便后续使用；提供给子类初始化扩展点，initServletBean()，该方法由FrameworkServlet覆盖。

2. **FrameworkServlet****继承HttpServletBean****，**通过initServletBean()进行Web上下文初始化，该方法主要覆盖一下两件事情：初始化web上下文；提供给子类初始化扩展点。

3. DispatcherServlet继承FrameworkServlet，并实现了onRefresh()方法提供一些前端控制器相关的配置。

   整个DispatcherServlet初始化的过程和做了些什么事情，具体主要做了如下两件事情：

   1、初始化Spring Web MVC使用的Web上下文，并且指定父容器为WebApplicationContext（ContextLoaderListener加载了的根上下文）；

   2、初始化DispatcherServlet使用的策略，如HandlerMapping、HandlerAdapter等。

## SpringBoot 自动配置

[SpringBoot | 是如何实现自动配置的？ - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1522642)