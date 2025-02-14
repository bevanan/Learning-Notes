# 第七章 开源实战

## 7.1 剖析Spring框架中用到的经典设计模式

### 7.1.1 Spring中工厂模式的应用

**Spring的设计理念**

- Spring是面向Bean的编程（BOP：Bean Oriented Programming），Bean在Spring中才是真正的主角。Bean在Spring中作用就像Object对OOP的意义一样，没有对象的概念就像没有面向对象编程，Spring中没有Bean也就没有Spring存在的意义。Spring提供了IoC 容器通过配置文件或者注解的方式来管理对象之间的依赖关系。
- 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递给它。

#### 7.1.1.1 Spring中的Bean组件

Bean组件定义在Spring的**org.springframework.beans**包下，解决了以下几个问题：

这个包下的所有类主要解决了三件事：

- Bean的定义
- Bean的创建
- Bean的解析

Spring Bean的创建是典型的工厂模式，它的顶级接口是BeanFactory。

<img src=".\img\137.jpg" alt="image-20220530160637842" style="zoom: 100%;" />  



BeanFactory有三个子类：ListableBeanFactory、HierarchicalBeanFactory和AutowireCapableBeanFactory。目的是为了**区分Spring内部对象处理和转化的数据限制**。

但是从图中可以发现最终的默认实现类是DefaultListableBeanFactory，它实现了所有的接口

#### 7.1.1.2 Spring中的BeanFactory

Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得Bean对象

BeanFactory，以Factory结尾，表示它是一个工厂(接口)， 它负责生产和管理bean的一个工厂。在Spring中，BeanFactory是工厂的顶层接口，也是IOC容器的核心接口，因此BeanFactory中定义了**管理Bean的通用方法**，如 **getBean** 和 **containsBean** 等.

它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。

![image-20221025193939157](C:\Users\86187\AppData\Roaming\Typora\typora-user-images\image-20221025193939157.png) 

BeanFactory只是个接口，并不是IOC容器的具体实现，所以Spring容器给出了很多种实现，如 **DefaultListableBeanFactory**、**XmlBeanFactory**、**ApplicationContext**等，其中XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。



**1) BeanFactory源码解析** 

```java
public interface BeanFactory {
    
    /**
    	对FactoryBean的转移定义,因为如果使用bean的名字来检索FactoryBean得到的是对象是工厂生成的对象,
    	如果想得到工厂本身就需要转移
    */
    String FACTORY_BEAN_PREFIX = "&";

    //根据Bean的名字 获取IOC容器中对应的实例
    Object getBean(String var1) throws BeansException;

    
    //根据Bean的名字和class类型得到bean实例,增加了类型安全验证机制
    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    <T> T getBean(Class<T> var1, Object... var2) throws BeansException;

    <T> ObjectProvider<T> getBeanProvider(Class<T> var1);

    <T> ObjectProvider<T> getBeanProvider(ResolvableType var1);

    
   //查看Bean容器中是否存在对应的实例,存在返回true 否则返回false
    boolean containsBean(String var1);

    //根据Bean的名字 判断这个bean是不是单例
    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;

    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, ResolvableType var2) throws NoSuchBeanDefinitionException;

    boolean isTypeMatch(String var1, Class<?> var2) throws NoSuchBeanDefinitionException;

    //得到bean实例的class类型
    @Nullable
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;

    @Nullable
    Class<?> getType(String var1, boolean var2) throws NoSuchBeanDefinitionException;

    
    //得到bean的别名
    String[] getAliases(String var1);
}

```

BeanFactory的使用场景

1. 从IOC容器中获取Bean(Name or Type)
2. 检索IOC容器中是否包含了指定的对象
3. 判断Bean是否为单例

**2) BeanFactory的使用**

```java
public class User {

    private int id;

    private String name;

    private Friends friends;

    public User() {
    }

    public User(Friends friends) {
        this.friends = friends;
    }

 	//get set......
}

public class Friends {

    private List<String> names;

    public Friends() {
    }

    public List<String> getNames() {
        return names;
    }

    public void setNames(List<String> names) {
        this.names = names;
    }
}
```

配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
        http://www.springframework.org/schema/task
        http://www.springframework.org/schema/task/spring-task-4.2.xsd">
    
    <bean id="User" class="com.example.factory.User">
        <property name="friends" ref="UserFriends" />
    </bean>
    <bean id="UserFriends" class="com.example.factory.Friends">
        <property name="names">
            <list>
                <value>"LiLi"</value>
                <value>"LuLu"</value>
            </list>
        </property>
    </bean>
</beans>
```

测试

```java
public class SpringFactoryTest {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("bean.xml");
        User user = ctx.getBean("User", User.class);

        List<String> names = user.getFriends().getNames();
        for (String name : names) {
            System.out.println("FriendName: " + name);
        }

        ctx.close();
    }
}
```

#### 7.1.1.3 Spring中的FactoryBean

首先FactoryBean是一个Bean，但又不仅仅是一个Bean，这样听起来矛盾，但为啥又这样说呢？其实在Spring中，所有的Bean都是由BeanFactory（也就是IOC容器）来进行管理的。但对FactoryBean而言，**这个FactoryBean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂方法模式和修饰器模式类似** 

**1) 为什么需要FactoryBean?**

1. 在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个`org.springframework.bean.factory.FactoryBean`的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。
2. 由于第三方库不能直接注册到spring容器，于是可以实现`org.springframework.bean.factory.FactoryBean`接口，然后给出自己对象的实例化代码即可。

**2 ) FactoryBean的使用特点**

1. 当用户使用容器本身时，可以使用转义字符"&"来得到FactoryBean本身，以区别通过FactoryBean产生的实例对象和FactoryBean对象本身。

2. 在BeanFactory中通过如下代码定义了该转义字符：

   ```
    StringFACTORY_BEAN_PREFIX = "&";
   ```

3. 举例

   ```
   如果MyObject是一个FactoryBean，则使用&MyObject得到的是MyObject对象，而不是MyObject产生出来的对象。
   ```

**3) FactoryBean的代码示例**

```java
@Configuration
@ComponentScan("com.example.factory_bean")
public class AppConfig {
}

@Component("studentBean")
public class StudentBean implements FactoryBean {

    //返回工厂中的实例
    @Override
    public Object getObject() throws Exception {
        //这里并不一定要返回MyBean自身的实例，可以是其他任何对象的实例。
        return new TeacherBean();
    }

    //该方法返回的类型是在IOC容器中getBean所匹配的类型
    @Override
    public Class<?> getObjectType() {
        return StudentBean.class;
    }

    public void study(){
        System.out.println("学生学习......");
    }
}

public class TeacherBean {

    public void teach(){
        System.out.println("老师教书......");
    }
}

public class Test01 {

    public static void main(String[] args) {

        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        //StudentBean studentBean = (StudentBean)context.getBean("studentBean");

        //加上&符号,返回工厂中的实例
//        StudentBean studentBean = (StudentBean)context.getBean("&studentBean");
//        studentBean.study();

        TeacherBean teacherBean = (TeacherBean) context.getBean("studentBean");
        teacherBean.teach();
    }
}
```

**3) FactoryBean源码分析** 

```java
public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    /**
    getObject()方法: 会返回该FactoryBean生产的对象实例,我们需要实现该方法,以给出自己的对象实例化逻辑
    这个方法也是FactoryBean的核心.
    */
    @Nullable
    T getObject() throws Exception;

    /**
    getObjectType()方法: 仅返回getObject() 方法所返回的对象类型,如果预先无法确定,返回NULL,
    这个方法返回类型是在IOC容器中getBean所匹配的类型
    */
    @Nullable
    Class<?> getObjectType();

    //该方法的结果用于表明 工厂方法getObject() 所生产的 对象是否要以单例形式存储在容器中如果以单例存在就返回true,否则返回false
    default boolean isSingleton() {
        return true;
    }
}
```

FactoryBean表现的是一个工厂的职责,如果一个BeanA 是实现FactoryBean接口,那么A就是变成了一个工厂,根据A的名称获取到的实际上是工厂调用getObject()方法返回的对象,而不是对象本身,如果想获取工厂对象本身,需要在名称前面加上 '&'符号

- getObject('name') 返回的是工厂中工厂方法生产的实例
- getObject('&name') 返回的是工厂本身实例

**使用场景**

- FactoryBean的最为经典的使用场景,就是用来创建AOP代理对象,这个对象在Spring中就是 ProxyFactoryBean

**BeanFactory与FactoryBean区别**

- 他们两个都是工厂,但是FactoryBean本质还是一个Bean,也归BeanFactory管理
- BeanFactory是Spring容器的顶层接口,FactoryBean更类似于用户自定义的工厂接口

**BeanFactory和ApplicationContext的区别**

- BeanFactory是Spring容器的顶层接口,而ApplicationContext应用上下文类 他是BeanFactory的子类,他是Spring中更高级的容器,提供了更多的功能
  - 国际化
  - 访问资源
  - 载入多个上下文
  - 消息发送 响应机制
- 两者的装载bean的时机不同
  - BeanFactory: 在系统启动的时候不会去实例化bean,只有从容器中拿bean的时候才会去实例化(懒加载)
    - 优点: 应用启动的时候占用的资源比较少,对资源的使用要求比较高的应用 ,比较有优势
  - ApplicationContext:在启动的时候就把所有的Bean全部实例化.
    - lazy-init= true 可以使bean延时实例化
    - 优点: 所有的Bean在启动的时候就加载,系统运行的速度快,还可以及时的发现系统中配置的问题.



### 7.1.2 Spring中观察者模式的应用

#### 7.1.2.1 观察者模式与发布订阅模式的异同

**观察者模式它是用于建立一种对象与对象之间的依赖关系,一个对象发生改变时将自动通知其他对象,其他对象将相应的作出反应.** 

> 在观察者模式中发生改变的对象称为**观察目标**,而被通知的对象称为**观察者**,一个观察目标可以应对多个观察者,而且这些观察者之间可以没有任何相互联系,可以根据需要增加和删除观察者,使得系统更易于扩展.
>

观察者模式的别名有发布-订阅(Publish/Subscribe)模式, 我们来看一下观察者模式与发布订阅模式结构上的区别

- 在设计模式结构上，发布订阅模式**继承**自观察者模式，是观察者模式的一种实现的变体。
- 在设计模式意图上，两者**关注点**不同，一个关心数据源，一个关心的是事件消息。

<img src=".\img\134.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

> 观察者模式里，只有两个角色 —— 观察者 + 被观察者; 而发布订阅模式里，却不仅仅只有发布者和订阅者两个角色，还有一个管理并执行消息队列的 “经纪人Broker”
>
> 观察者和被观察者，是松耦合的关系;发布者和订阅者，则完全不存在耦合

- 观察者模式：**数据源直接通知订阅者发生改变。**

- 发布订阅模式：**数据源告诉第三方（事件通道）发生了改变，第三方再通知订阅者发生了改变。**

#### 7.1.2.2 Spring中的观察者模式

Spring 基于观察者模式，实现了自身的事件机制也就是事件驱动模型，事件驱动模型通常也被理解成观察者或者发布/订阅模型。

spring事件模型提供如下几个角色

- **ApplicationEvent**
- **ApplicationListener**
- **ApplicationEventPublisher**
- **ApplicationEventMulticaster** 



**1) 事件：ApplicationEvent** 

- 是所有事件对象的父类。ApplicationEvent 继承自 jdk 的 EventObject, 所有的事件都需要继承 ApplicationEvent, 并且通过 source 得到事件源。

  ```java
  public abstract class ApplicationEvent extends EventObject {
      private static final long serialVersionUID = 7099057708183571937L;
      private final long timestamp = System.currentTimeMillis();
  
      public ApplicationEvent(Object source) {
          super(source);
      }
  
      public final long getTimestamp() {
          return this.timestamp;
      }
  }
  ```

- Spring 也为我们提供了很多内置事件:

  - ContextRefreshEvent，当ApplicationContext容器初始化完成或者被刷新的时候，就会发布该事件。

  - ContextStartedEvent，当ApplicationContext启动的时候发布事件.

  - ContextStoppedEvent，当ApplicationContext容器停止的时候发布事件.

  - RequestHandledEvent，只能用于DispatcherServlet的web应用，Spring处理用户请求结束后，系统会触发该事件。

​		 <img src=".\img\133.jpg" alt="image-20220530160637842" style="zoom: 50%;" />

**2) 事件监听：ApplicationListener** 

- ApplicationListener(应用程序事件监听器) 继承自jdk的EventListener,所有的监听器都要实现这个接口,这个接口只有一个onApplicationEvent()方法,该方法接受一个ApplicationEvent或其子类对象作为参数

- 在方法体中,可以通过不同对Event类的判断来进行相应的处理.当事件触发时所有的监听器都会收到消息,如果你需要对监听器的接收顺序有要求,可是实现该接口的一个实现SmartApplicationListener,通过这个接口可以指定监听器接收事件的顺序.

  ```java
  @FunctionalInterface
  public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
      void onApplicationEvent(E var1);
  }
  ```

- 实现了ApplicationListener接口之后，需要实现方法onApplicationEvent()，在容器将所有的Bean都初始化完成之后，就会执行该方法。

**3) 事件源：ApplicationEventPublisher** 

- 事件的发布者，封装了事件发布功能方法接口，是Applicationcontext接口的超类

  > 事件机制的实现需要三个部分,事件源,事件,事件监听器,在上面介绍的ApplicationEvent就相当于事件,ApplicationListener相当于事件监听器,这里的事件源说的就是ApplicationEventPublisher.

  ```java
  public interface ApplicationEventPublisher {
      default void publishEvent(ApplicationEvent event) {
          this.publishEvent((Object)event);
      }
  	//调用publishEvent方法,传入一个ApplicationEvent的实现类对象作为参数,每当ApplicationContext发布ApplicationEvent时,所有的ApplicationListener就会被自动的触发.
      void publishEvent(Object var1);
  }
  ```

- 我们常用的ApplicationContext都继承了AbstractApplicationContext,像我们平时常见ClassPathXmlApplicationContext、XmlWebApplicationContex也都是继承了它,AbstractApplicationcontext是ApplicationContext接口的抽象实现类,在该类中实现了publishEvent方法:

  ```java
     protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
          Assert.notNull(event, "Event must not be null");
  
          if (this.earlyApplicationEvents != null) {
              this.earlyApplicationEvents.add(applicationEvent);
          } else {
            //事件发布委托给applicationEventMulticaster
            this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
          }
      }
  ```

  在这个方法中,我们看到了一个getApplicationEventMulticaster().这就要牵扯到另一个类ApplicationEventMulticaster.

**4) 事件管理：ApplicationEventMulticaster**

- 用于事件监听器的注册和事件的广播。监听器的注册就是通过它来实现的，它的作用是把 Applicationcontext 发布的 Event 广播给它的监听器列表。

  ```java
  public interface ApplicationEventMulticaster {
      
      //添加事件监听器
      void addApplicationListener(ApplicationListener<?> var1);
  
      //添加事件监听器,使用容器中的bean
      void addApplicationListenerBean(String var1);
  
      //移除事件监听器
      void removeApplicationListener(ApplicationListener<?> var1);
  
      void removeApplicationListenerBean(String var1);
  
      //移除所有
      void removeAllListeners();
  
      //发布事件
      void multicastEvent(ApplicationEvent var1);
  
      void multicastEvent(ApplicationEvent var1, @Nullable ResolvableType var2);
  }
  ```

- 在AbstractApplicationcontext中有一个applicationEventMulticaster的成员变量,提供了监听器Listener的注册方法.

  ```java
  public abstract class AbstractApplicationContext extends DefaultResourceLoader
          implements ConfigurableApplicationContext, DisposableBean {
  
  　　private ApplicationEventMulticaster applicationEventMulticaster;
      
  　　protected void registerListeners() {
          // Register statically specified listeners first.
          for (ApplicationListener<?> listener : getApplicationListeners()) {
              getApplicationEventMulticaster().addApplicationListener(listener);
          }
          // Do not initialize FactoryBeans here: We need to leave all regular beans
          // uninitialized to let post-processors apply to them!
          String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
          for (String lisName : listenerBeanNames) {
              getApplicationEventMulticaster().addApplicationListenerBean(lisName);
          }
      }
  }
  ```

#### 7.1.2.3 事件监听案例

实现一个需求：当调用一个类的方法完成时，该类发布事件，事件监听器监听该类的事件并执行的自己的方法逻辑

假设这个类是Request、发布的事件是ReuqestEvent、事件监听者是ReuqestListener。当调用Request的doRequest方法时，发布事件。

代码如下

```java
/**
 * 定义事件
 * @author spikeCong
 * @date 2022/10/24
 **/
public class RequestEvent  extends ApplicationEvent {

    public RequestEvent(Object source) {
        super(source);
    }
}

/**
 * 发布事件
 * @author spikeCong
 * @date 2022/10/24
 **/
@Component
public class Request {

    @Autowired
    private ApplicationContext applicationContext;

    public void doRequest(){
        System.out.println("调用Request类的doRequest方法发送一个请求......");
        applicationContext.publishEvent(new RequestEvent(this));
    }
}

/**
 * 监听事件
 * @author spikeCong
 * @date 2022/10/24
 **/
@Component
public class RequestListener implements ApplicationListener<RequestEvent> {

    @Override
    public void onApplicationEvent(RequestEvent requestEvent) {
        System.out.println("监听到RequestEvent事件,执行本方法");
    }
}

public class SpringEventTest {

    public static void main(String[] args) {
        ApplicationContext context =
                new AnnotationConfigApplicationContext("com.mashibing.pubsub");

        Request request = (Request) context.getBean("request");

        //调用方法发布事件
        request.doRequest();
    }
}

//打印日志
调用Request类的doRequest方法发送一个请求......
监听到RequestEvent事件,执行本方法
```

#### 7.1.2.4 事件机制工作流程

上面代码的执行流程

<img src=".\img\146.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

1. 监听器什么时候注册到IOC容器

   注册的开始逻辑是在AbstractApplicationContext类的refresh方法，该方法包含了整个IOC容器初始化所有方法。其中有一个registerListeners()方法就是注册系统监听者(spring自带的)和自定义监听器的。

   ```java
   public void refresh() throws BeansException, IllegalStateException {
       			//BeanFactory准备工作完成后进行的后置处理工作
                   this.postProcessBeanFactory(beanFactory);
       
       			//执行BeanFactoryPostProcessor的方法；
                   this.invokeBeanFactoryPostProcessors(beanFactory);
       
       			//注册BeanPostProcessor（Bean的后置处理器），在创建bean的前后等执行
                   this.registerBeanPostProcessors(beanFactory);
       	
       			//初始化MessageSource组件（做国际化功能；消息绑定，消息解析）；
                   this.initMessageSource();
       
       			//初始化事件派发器
                   this.initApplicationEventMulticaster();
       
       			////子类重写这个方法，在容器刷新的时候可以自定义逻辑；如创建Tomcat，Jetty等WEB服务器
                   this.onRefresh();
       
                       //注册应用的监听器。就是注册实现了ApplicationListener接口的监听器bean，这些监听器是注册到ApplicationEventMulticaster中的
                   this.registerListeners();
       
       			//初始化所有剩下的非懒加载的单例bean
                   this.finishBeanFactoryInitialization(beanFactory);
       
       			//完成context的刷新
                   this.finishRefresh();
       }
   ```
   
   看registerListeners的关键方法体，其中的两个方法`addApplicationListener和addApplicationListenerBean`，从方法可以看出是添加监听者。
   
   ```java
   protected void registerListeners() {
       Iterator var1 = this.getApplicationListeners().iterator();
   
       while(var1.hasNext()) {
           ApplicationListener<?> listener = (ApplicationListener)var1.next();
           this.getApplicationEventMulticaster().addApplicationListener(listener);
       }
   
       String[] listenerBeanNames = this.getBeanNamesForType(ApplicationListener.class, true, false);
       String[] var7 = listenerBeanNames;
       int var3 = listenerBeanNames.length;
   
       for(int var4 = 0; var4 < var3; ++var4) {
           String listenerBeanName = var7[var4];
           this.getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
       }
   }
   ```
   
   那么最后将监听者放到哪里了呢？就是ApplicationEventMulticaster接口的子类
   
   <img src=".\img\135.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 
   
   该接口主要两个职责，维护ApplicationListener相关类和发布事件。
   
   实现在默认实现类AbstractApplicationEventMulticaster，`最后将Listener放到了内部类ListenerRetriever两个set集合中`
   
   ```java
   private class ListenerRetriever {
           public final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet();
           public final Set<String> applicationListenerBeans = new LinkedHashSet();
   }
   ```

​		ListenerRetriever被称为监听器注册表。

2. Spring如何发布的事件并通知监听者

   **这个注意的有两个方法**

   **1) publishEvent方法**
   
   - `AbstractApplicationContext`实现了`ApplicationEventPublisher` 接口的`publishEvent`方法
   
   ```java
   protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
       Assert.notNull(event, "Event must not be null");
       Object applicationEvent;
       
       //尝试转换为ApplicationEvent或者PayloadApplicationEvent，如果是PayloadApplicationEvent则获取eventType
       if (event instanceof ApplicationEvent) {
           applicationEvent = (ApplicationEvent)event;
       } else {
           applicationEvent = new PayloadApplicationEvent(this, event);
           if (eventType == null) {
               eventType = ((PayloadApplicationEvent)applicationEvent).getResolvableType();
           }
       }
   
      
       if (this.earlyApplicationEvents != null) {
            //判断earlyApplicationEvents是否为空(也就是早期事件还没有被发布-说明广播器还没有实例化好)，如果不为空则将当前事件放入集合
           this.earlyApplicationEvents.add(applicationEvent);
       } else {
           //否则获取ApplicationEventMulticaster调用其multicastEvent将事件广播出去。本文这里获取到的广播器实例是SimpleApplicationEventMulticaster。
           this.getApplicationEventMulticaster().multicastEvent((ApplicationEvent)applicationEvent, eventType);
       }
   	
       //将事件交给父类处理
       if (this.parent != null) {
           if (this.parent instanceof AbstractApplicationContext) {
               ((AbstractApplicationContext)this.parent).publishEvent(event, eventType);
           } else {
               this.parent.publishEvent(event);
           }
       }
   
   }
   ```
   
    **2) multicastEvent方法**
   
   继续进入到`multicastEvent方法`，该方法有两种方式调用invokeListener，通过线程池和直接调用，进一步说就是通过异步和同步两种方式调用.
   
   ```java
   public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
       
       //解析事件类型
       ResolvableType type = eventType != null ? eventType : this.resolveDefaultEventType(event);
       
       //获取执行器
       Executor executor = this.getTaskExecutor();
       
       // 获取合适的ApplicationListener，循环调用监听器的onApplicationEvent方法
       Iterator var5 = this.getApplicationListeners(event, type).iterator();
   
       while(var5.hasNext()) {
           ApplicationListener<?> listener = (ApplicationListener)var5.next();
           if (executor != null) {
               //如果executor不为null，则交给executor去调用监听器
               executor.execute(() -> {
                   this.invokeListener(listener, event);
               });
           } else {
               //否则，使用当前主线程直接调用监听器；
               this.invokeListener(listener, event);
           }
       }
   
   }
   ```
   
   
   
   **3) invokeListener方法**
   
   ```java
   // 该方法增加了错误处理逻辑，然后调用doInvokeListener
   protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
       ErrorHandler errorHandler = this.getErrorHandler();
       if (errorHandler != null) {
           try {
               this.doInvokeListener(listener, event);
           } catch (Throwable var5) {
               errorHandler.handleError(var5);
           }
       } else {
           this.doInvokeListener(listener, event);
       }
   
   }
   
   private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
       //直接调用了listener接口的onApplicationEvent方法
       listener.onApplicationEvent(event);  
   }
   ```

### 7.1.3 结合设计模式自定义SpringIOC

#### 7.1.3.1 Spring IOC核心组件

**1) BeanFactory**  

BeanFactory作为最顶层的一个接口，定义了IoC容器的基本功能规范

<img src=".\img\137.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

从类图中我们可以发现最终的默认实现类是DefaultListableBeanFactory，它实现了所有的接口。那么为何要定义这么多层次的接口呢？
每个接口都有它的使用场合，主要是为了区分在Spring内部操作过程中对象的传递和转化，对对象的数据访问所做的限制。

例如，

- ListableBeanFactory接口表示这些Bean可列表化。
- HierarchicalBeanFactory表示这些Bean 是有继承关系的，也就是每个 Bean 可能有父 Bean
- AutowireCapableBeanFactory 接口定义Bean的自动装配规则。

这三个接口共同定义了Bean的集合、Bean之间的关系及Bean行为。

在BeanFactory里只对IoC容器的基本行为做了定义，根本不关心你的Bean是如何定义及怎样加载的。正如我们只关心能从工厂里得到什么产品，不关心工厂是怎么生产这些产品的。



**2 ) ApplicationContext** 

BeanFactory有一个很重要的子接口，就是ApplicationContext接口，该接口主要来规范容器中的bean对象是非延时加载，即在创建容器对象的时候就对象bean进行初始化，并存储到一个容器中。

```java
//延时加载
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("bean.xml"));

//立即加载
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
User user = context.getBean("user", User.class);
```

 ApplicationContext 的子类主要包含两个方面：

- ConfigurableApplicationContext 表示该 Context 是可修改的，也就是在构建 Context 中用户可以动态添加或修改已有的配置信息

- WebApplicationContext 顾名思义，就是为 web 准备的 Context 他可以直接访问到 ServletContext，通常情况下，这个接口使用少

要知道工厂是如何产生对象的，我们需要看具体的IoC容器实现，Spring提供了许多IoC容器实现，比如：

- ClasspathXmlApplicationContext : 根据类路径加载xml配置文件，并创建IOC容器对象。
- FileSystemXmlApplicationContext ：根据系统路径加载xml配置文件，并创建IOC容器对象。
- AnnotationConfigApplicationContext ：加载注解类配置，并创建IOC容器。

<img src=".\img\138.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  

总体来说 ApplicationContext 必须要完成以下几件事：

- 标识一个应用环境
- 利用 BeanFactory 创建 Bean 对象
- 保存对象关系表
- 能够捕获各种事件



**3) Bean定义：BeanDefinition** 

这里的 BeanDefinition 就是我们所说的 Spring 的 Bean，我们自己定义的各个 Bean 其实会转换成一个个 BeanDefinition 存在于 Spring 的 BeanFactory 中

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
        implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
	//DefaultListableBeanFactory 中使用 Map 结构保存所有的 BeanDefinition 信息
    private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256); 
}   
```

**BeanDefinition** 中保存了我们的 Bean 信息，比如这个 Bean 指向的是哪个类、是否是单例的、是否懒加载、这个 Bean 依赖了哪些 Bean 等等。



**4) BeanDefinitionReader**

Bean的解析过程非常复杂，功能被分得很细，因为这里需要被扩展的地方很多，必须保证足够的灵活性，以应对可能的变化。Bean的解析主要就是对Spring配置文件的解析。

这个解析过程主要通过BeanDefinitionReader来完成，看看Spring中BeanDefinitionReader的类结构图，如下图所示。

​                       <img src=".\img\140.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

BeanDefinitionReader接口定义的功能

```java
public interface BeanDefinitionReader {

	/*
		下面的loadBeanDefinitions都是加载bean定义，从指定的资源中
	*/
	int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;
	int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;
	int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;
	int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;
}
```



**5) BeanFactory后置处理器**

后置处理器是一种拓展机制，贯穿Spring Bean的生命周期

后置处理器分为两类：

**BeanFactory后置处理器：BeanFactoryPostProcessor**

实现该接口，可以在spring的bean创建之前，修改bean的定义属性

<img src=".\img\141.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

 

```java
public interface BeanFactoryPostProcessor {

    /*
     *  该接口只有一个方法postProcessBeanFactory，方法参数是ConfigurableListableBeanFactory，通过该
        参数，可以获取BeanDefinition
    */
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```



**6) Bean后置处理器：BeanPostProcessor**

BeanPostProcessor是Spring IOC容器给我们提供的一个扩展接口

实现该接口，可以在spring容器实例化bean之后，在执行bean的初始化方法前后，添加一些处理逻辑

<img src=".\img\143.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

 

```java
public interface BeanPostProcessor {
    //bean初始化方法调用前被调用
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
    //bean初始化方法调用后被调用
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

#### 7.1.3.2 IOC流程图

<img src=".\img\145.jpg" alt="image-20220530160637842" style="zoom: 100%;" />

1. 容器环境的初始化(系统、JVM 、解析器、类加载器等等)
2. Bean工厂的初始化(IOC容器首先会销毁旧工厂,旧Bean、创建新的工厂)
3. 读取：通过BeanDefinitonReader读取我们项目中的配置（application.xml）
4. 定义：通过解析xml文件内容，将里面的Bean解析成BeanDefinition（未实例化、未初始化）
5. 将解析得到的BeanDefinition,存储到工厂类的Map容器中
6. 调用 BeanFactoryPostProcessor 该方法是一种功能增强，可以在这个步骤对已经完成初始化的 BeanFactory 进行属性覆盖，或是修改已经注册到 BeanFactory 的 BeanDefinition
7. 通过反射实例化bean对象
8. 进入到Bean实例化流程,首先设置对象属性
9. 检查Aware相关接口,并设置相关依赖
10. 前置处理器,执行BeanPostProcesser的before方法对bean进行扩展
11. 检查是否有实现initializingBean 回调接口,如果实现就要回调其中的AftpropertiesSet() 方法,(通过可以完成一些配置的加载)
12. 检查是否有配置自定义的init-method ,
13. 后置处理器执行BeanPostProcesser 的after方法 -->  AOP就是在这个阶段完成的, 在这里判断bean对象是否实现接口,实现就使用JDK代理,否则选择CGLIB
14. 对象创建完成,添加到BeanFactory的单例池中

#### 7.1.3.3 自定义SpringIOC

对下面的配置文件进行解析，并自定义SpringIOC, 对涉及到的对象进行管理。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <bean id="courseService" class="com.mashibing.service.impl.CourseServiceImpl">
        <property name="courseDao" ref="courseDao"></property>
    </bean>
    <bean id="courseDao" class="com.mashibing.dao.impl.CourseDaoImpl"></bean>
</beans>
```

##### 1) 创建与Bean相关的pojo类

- **PropertyValue类**: 用于封装bean的属性，体现到上面的配置文件就是封装bean标签的子标签property标签数据。

```java
package com.mashibing.framework.beans;

/**
 * 该类用来封装bean标签下的property子标签的属性
 *      1.name属性
 *      2.ref属性
 *      3.value属性: 给基本数据类型及string类型数据赋的值
 * @author spikeCong
 * @date 2022/10/26
 **/
public class PropertyValue {

    private String name;

    private String ref;

    private String value;

    public PropertyValue() {
    }

    public PropertyValue(String name, String ref, String value) {
        this.name = name;
        this.ref = ref;
        this.value = value;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getRef() {
        return ref;
    }

    public void setRef(String ref) {
        this.ref = ref;
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

- **MutablePropertyValues类**: 一个bean标签可以有多个property子标签，所以再定义一个MutablePropertyValues类，用来存储并管理多个PropertyValue对象。

```java
package com.mashibing.framework.beans;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
import java.util.List;

/**
 * 该类用来存储和遍历多个PropertyValue对象
 * @author spikeCong
 * @date 2022/10/26
 **/
public class MutablePropertyValues implements Iterable<PropertyValue>{

    //定义List集合,存储PropertyValue的容器
    private final List<PropertyValue> propertyValueList;

    //空参构造中 初始化一个list
    public MutablePropertyValues() {
        this.propertyValueList = new ArrayList<PropertyValue>();
    }

    //有参构造 接收一个外部传入的list,赋值propertyValueList属性
    public MutablePropertyValues(List<PropertyValue> propertyValueList) {
        if(propertyValueList == null){
            this.propertyValueList = new ArrayList<PropertyValue>();
        }else{
            this.propertyValueList = propertyValueList;
        }
    }

    //获取当前容器对应的迭代器对象
    @Override
    public Iterator<PropertyValue> iterator() {

        //直接获取List集合中的迭代器
        return propertyValueList.iterator();
    }

    //获取所有的PropertyValue
    public PropertyValue[] getPropertyValues(){
        //将集合转换为数组并返回
        return propertyValueList.toArray(new PropertyValue[0]); //new PropertyValue[0]声明返回的数组类型
    }

    //根据name属性值获取PropertyValue
    public PropertyValue getPropertyValue(String propertyName){
        //遍历集合对象
        for (PropertyValue propertyValue : propertyValueList) {
            if(propertyValue.getName().equals(propertyName)){
                return propertyValue;
            }
        }

        return null;
    }

    //判断集合是否为空,是否存储PropertyValue
    public boolean isEmpty(){
        return propertyValueList.isEmpty();
    }

    //向集合中添加
    public MutablePropertyValues addPropertyValue(PropertyValue value){
        //判断集合中存储的propertyvalue对象.是否重复,重复就进行覆盖
        for (int i = 0; i < propertyValueList.size(); i++) {
            //获取集合中每一个 PropertyValue
            PropertyValue currentPv = propertyValueList.get(i);

            //判断当前的pv的name属性 是否与传入的相同,如果相同就覆盖
            if(currentPv.getName().equals(value.getName())){
                propertyValueList.set(i,value);
                return this;
            }
        }

        //没有重复
        this.propertyValueList.add(value);
        return this;  //目的是实现链式编程
    }

    //判断是否有指定name属性值的对象
    public boolean contains(String propertyName){
        return getPropertyValue(propertyName) != null;
    }
}
```

- **BeanDefinition类**: 用来封装bean信息的，主要包含id（即bean对象的名称）、class（需要交由spring管理的类的全类名）及子标签property数据。

```java
/**
 * 封装Bean标签数据的类,包括id与class以及子标签的数据
 * @author spikeCong
 * @date 2022/10/27
 **/
public class BeanDefinition {

    private String id;

    private String className;

    private MutablePropertyValues propertyValues;

    public BeanDefinition() {
        propertyValues = new MutablePropertyValues();
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }

    public MutablePropertyValues getPropertyValues() {
        return propertyValues;
    }

    public void setPropertyValues(MutablePropertyValues propertyValues) {
        this.propertyValues = propertyValues;
    }
}
```

##### 2) 创建注册表相关的类

BeanDefinition对象存取的操作, 其实是在BeanDefinitionRegistry接口中定义的,它被称为是BeanDefinition的注册中心.

```java
//源码
public interface BeanDefinitionRegistry extends AliasRegistry {

	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	boolean containsBeanDefinition(String beanName);

	String[] getBeanDefinitionNames();
    
	int getBeanDefinitionCount();
    
	boolean isBeanNameInUse(String beanName);
}
```

BeanDefinitionRegistry继承结构图如下：

<img src=".\img\147.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

BeanDefinitionRegistry接口的子实现类主要有以下两个：

* DefaultListableBeanFactory

  在该类中定义了如下代码，就是用来注册bean

  ```java
  private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
  ```

* SimpleBeanDefinitionRegistry

  在该类中定义了如下代码，就是用来注册bean

  ```java
  private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(64);
  ```



- 自定义BeanDefinitionRegistry接口定义了注册表的相关操作，定义如下功能：

```java
public interface BeanDefinitionRegistry {

    //注册BeanDefinition对象到注册表中
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition);

    //从注册表中删除指定名称的BeanDefinition对象
    void removeBeanDefinition(String beanName) throws Exception;

    //根据名称从注册表中获取BeanDefinition对象
    BeanDefinition getBeanDefinition(String beanName) throws Exception;

    //判断注册表中是否包含指定名称的BeanDefinition对象
    boolean containsBeanDefinition(String beanName);

    //获取注册表中BeanDefinition对象的个数
    int getBeanDefinitionCount();

    //获取注册表中所有的BeanDefinition的名称
    String[] getBeanDefinitionNames();
}
```

- SimpleBeanDefinitionRegistry类, 该类实现了BeanDefinitionRegistry接口，定义了Map集合作为注册表容器。

```java
public class SimpleBeanDefinitionRegistry implements BeanDefinitionRegistry {

    private Map<String, BeanDefinition> beanDefinitionMap = new HashMap<String, BeanDefinition>();

    @Override
    public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) {
        beanDefinitionMap.put(beanName,beanDefinition);
    }

    @Override
    public void removeBeanDefinition(String beanName) throws Exception {
        beanDefinitionMap.remove(beanName);
    }

    @Override
    public BeanDefinition getBeanDefinition(String beanName) throws Exception {
        return beanDefinitionMap.get(beanName);
    }

    @Override
    public boolean containsBeanDefinition(String beanName) {
        return beanDefinitionMap.containsKey(beanName);
    }

    @Override
    public int getBeanDefinitionCount() {
        return beanDefinitionMap.size();
    }

    @Override
    public String[] getBeanDefinitionNames() {
        return beanDefinitionMap.keySet().toArray(new String[1]);
    }
}
```



##### 3) 创建解析器相关的类

BeanDefinitionReader接口

- BeanDefinitionReader用来解析配置文件并在注册表中注册bean的信息。定义了两个规范：
  * 获取注册表的功能,让外界可以通过该对象获取注册表对象
  * 加载配置文件,并注册bean数据

```java
/**
 * 该类定义解析配置文件规则的接口
 * @author spikeCong
 * @date 2022/10/28
 **/
public interface BeanDefinitionReader {

    //获取注册表对象
    BeanDefinitionRegistry getRegistry();

    //加载配置文件并在注册表中进行注册
    void loadBeanDefinitions(String configLocation) throws Exception;
}
```

XmlBeanDefinitionReader类

- XmlBeanDefinitionReader是专门用来解析xml配置文件的。该类实现BeanDefinitionReader接口并实现接口中的两个功能。

```java
/**
 * 该类是对XML文件进行解析的类
 * @author spikeCong
 * @date 2022/10/28
 **/
public class XmlBeanDefinitionReader implements BeanDefinitionReader {

    //声明注册表对象(将配置文件与注册表解耦,通过Reader降低耦合性)
    private BeanDefinitionRegistry registry;

    public XmlBeanDefinitionReader() {
        registry = new SimpleBeanDefinitionRegistry();
    }

    @Override
    public BeanDefinitionRegistry getRegistry() {
        return registry;
    }

    //加载配置文件
    @Override
    public void loadBeanDefinitions(String configLocation) throws Exception {
        //使用dom4j解析xml
        SAXReader reader = new SAXReader();

        //获取配置文件,类路径下
        InputStream is = XmlBeanDefinitionReader.class.getClassLoader().getResourceAsStream(configLocation);

        //获取document文档对象
        Document document = reader.read(is);

        Element rootElement = document.getRootElement();
        //解析bean标签
        parseBean(rootElement);
    }

    private void parseBean(Element rootElement) {

        //获取所有的bean标签
        List<Element> elements = rootElement.elements();

        //遍历获取每个bean标签的属性值和子标签property
        for (Element element : elements) {

            String id = element.attributeValue("id");
            String className = element.attributeValue("class");

            //封装到beanDefinition
            BeanDefinition beanDefinition = new BeanDefinition();
            beanDefinition.setId(id);
            beanDefinition.setClassName(className);

            //获取property
            List<Element> list = element.elements("property");

            MutablePropertyValues mutablePropertyValues = new MutablePropertyValues();

            //遍历,封装propertyValue,并保存到mutablePropertyValues
            for (Element element1 : list) {
                String name = element1.attributeValue("name");
                String ref = element1.attributeValue("ref");
                String value = element1.attributeValue("value");
                PropertyValue propertyValue = new PropertyValue(name,ref,value);
                mutablePropertyValues.addPropertyValue(propertyValue);
            }

            //将mutablePropertyValues封装到beanDefinition
            beanDefinition.setPropertyValues(mutablePropertyValues);

            System.out.println(beanDefinition);
            //将beanDefinition注册到注册表
            registry.registerBeanDefinition(id,beanDefinition);
        }
    }
}
```



##### 4) 创建IOC容器相关的类

**1) BeanFactory接口**

在该接口中定义IOC容器的统一规范和获取bean对象的方法。

```java
/**
 * IOC容器父接口
 * @author spikeCong
 * @date 2022/10/28
 **/
public interface BeanFactory {

    Object getBean(String name)throws Exception;

    //泛型方法,传入当前类或者其子类
    <T> T getBean(String name ,Class<? extends T> clazz)throws Exception;
}

```

**2) ApplicationContext接口**

该接口的所有的子实现类对bean对象的创建都是非延时的，所以在该接口中定义 `refresh()` 方法，该方法主要完成以下两个功能：

* 加载配置文件。
* 根据注册表中的BeanDefinition对象封装的数据进行bean对象的创建。

```java
/**
 * 定义非延时加载功能
 * @author spikeCong
 * @date 2022/10/28
 **/
public interface ApplicationContext extends BeanFactory {

    //进行配置文件加载,并进行对象创建
    void refresh();
}
```

**3) AbstractApplicationContext类**

* 作为ApplicationContext接口的子类，所以该类也是非延时加载，所以需要在该类中定义一个Map集合，作为bean对象存储的容器。
* 声明BeanDefinitionReader类型的变量，用来进行xml配置文件的解析，符合单一职责原则。
* BeanDefinitionReader类型的对象创建交由子类实现，因为只有子类明确到底创建BeanDefinitionReader哪儿个子实现类对象。

```java
/**
 * ApplicationContext接口的子实现类
 *      创建容器对象时,加载配置文件,对bean进行初始化
 * @author spikeCong
 * @date 2022/10/28
 **/
public abstract class AbstractApplicationContext implements ApplicationContext {

    //声明解析器变量
    protected BeanDefinitionReader beanDefinitionReader;

    //定义存储bean对象的Map集合
    protected Map<String,Object> singletonObjects = new HashMap<>();

    //声明配置文件类路径的变量
    protected String configLocation;

    @Override
    public void refresh() {

        //加载beanDefinition对象
        try {
            beanDefinitionReader.loadBeanDefinitions(configLocation);
            //初始化bean
            finishBeanInitialization();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    //bean初始化
    protected  void finishBeanInitialization() throws Exception {
        //获取对应的注册表对象
        BeanDefinitionRegistry registry = beanDefinitionReader.getRegistry();

        //获取beanDefinition对象
        String[] beanNames = registry.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            //进行bean的初始化
            getBean(beanName);
        }
    };
}
```

**4) ClassPathXmlApplicationContext类** 

该类主要是加载类路径下的配置文件，并进行bean对象的创建，主要完成以下功能：

* 在构造方法中，创建BeanDefinitionReader对象。
* 在构造方法中，调用refresh()方法，用于进行配置文件加载、创建bean对象并存储到容器中。
* 重写父接口中的getBean()方法，并实现依赖注入操作。

```java
/**
 * IOC容器具体的子实现类,加载XML格式配置文件
 * @author spikeCong
 * @date 2022/10/28
 **/
public class ClassPathXmlApplicationContext extends AbstractApplicationContext{

    public ClassPathXmlApplicationContext(String configLocation) {
        this.configLocation = configLocation;
        //构建解析器对象
        this.beanDefinitionReader = new XmlBeanDefinitionReader();

        this.refresh();
    }

    //跟据bean的对象名称获取bean对象
    @Override
    public Object getBean(String name) throws Exception {
        //判断对象容器中是否包含指定名称的bean对象,如果包含就返回,否则自行创建
        Object obj = singletonObjects.get(name);
        if(obj != null){
            return obj;
        }

        //自行创建,获取beanDefinition对象
        BeanDefinitionRegistry registry = beanDefinitionReader.getRegistry();
        BeanDefinition beanDefinition = registry.getBeanDefinition(name);

        //通过反射创建对象
        String className = beanDefinition.getClassName();
        Class<?> clazz = Class.forName(className);
        Object beanObj = clazz.newInstance();

        //CourseService与UserDao存依赖,所以要将UserDao一同初始化,进行依赖注入
        MutablePropertyValues propertyValues = beanDefinition.getPropertyValues();
        for (PropertyValue propertyValue : propertyValues) {
            //获取name属性值
            String propertyName = propertyValue.getName();
            //获取Value属性
            String value = propertyValue.getValue();
            //获取ref属性
            String ref = propertyValue.getRef();

            //ref与value只能存在一个
            if(ref != null && !"".equals(ref)){
                //获取依赖的bean对象,拼接set set+Course
                Object bean = getBean(ref);
                String methodName = StringUtils.getSetterMethodFieldName(propertyName);

                //获取所有方法对象
                Method[] methods = clazz.getMethods();
                for (Method method : methods) {
                    if(methodName.equals(method.getName())){
                        //执行该set方法
                        method.invoke(beanObj,bean);
                    }
                }
            }

            if(value != null && !"".equals(value)){
                String methodName = StringUtils.getSetterMethodFieldName(propertyName);
                //获取method
                Method method = clazz.getMethod(methodName, String.class);
                method.invoke(beanObj,value);
            }
        }

        //在返回beanObj之前 ,需要将对象存储到Map容器中
        this.singletonObjects.put(name,beanObj);


        return beanObj;
    }

    @Override
    public <T> T getBean(String name, Class<? extends T> clazz) throws Exception {
        Object bean = getBean(name);
        if(bean == null){
            return null;
        }

       return clazz.cast(bean);
    }
}
```

##### 5) 自定义IOC容器测试

第一步: 将我们写好的自定义IOC容器项目,安装到maven仓库中,使其他项目可以引入其依赖

```xml
//依赖信息
<dependencies>
    <dependency>
        <groupId>com.mashibing</groupId>
        <artifactId>user_defined_springioc</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

第二步: 创建一个新的maven项目,引入上面的依赖,项目结构如下



第三步: 完成代码编写

- dao

```java
public interface CourseDao {
    public void add();
}

public class CourseDaoImpl implements CourseDao {

    //value注入
    private String courseName;

    public String getCourseName() {
        return courseName;
    }

    public void setCourseName(String courseName) {
        this.courseName = courseName;
    }

    public CourseDaoImpl() {
        System.out.println("CourseDaoImpl创建了......");
    }

    @Override
    public void add() {
        System.out.println("CourseDaoImpl的add方法执行了......" + courseName);
    }
}
```

- service

```java
public interface CourseService {

    public void add();
}

public class CourseServiceImpl implements CourseService {

    public CourseServiceImpl() {
        System.out.println("CourseServiceImpl创建了......");
    }

    private CourseDao courseDao;

    public void setCourseDao(CourseDao courseDao) {
        this.courseDao = courseDao;
    }

    @Override
    public void add() {
        System.out.println("CourseServiceImpl的add方法执行了......");
        courseDao.add();
    }
}
```

- applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <bean id="courseService" class="com.mashibing.test_springioc.service.impl.CourseServiceImpl">
        <property name="courseDao" ref="courseDao"></property>
    </bean>
    <bean id="courseDao" class="com.mashibing.test_springioc.dao.impl.CourseDaoImpl">
        <property name="courseName" value="java"></property>
    </bean>
</beans>
```

- Controller

```java
public class CourseController{

    public static void main(String[] args) {
        //1.创建Spring的容器对象
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        //2.从容器对象中获取CourseService对象
        CourseService courseService = context.getBean("courseService", CourseService.class);

        //3.调用UserService的add方法
        courseService.add();
    }
}
```

##### 6) 案例中使用到的设计模式

* 工厂模式。这个使用工厂模式 + 配置文件的方式。
* 单例模式。Spring IOC管理的bean对象都是单例的，此处的单例不是通过构造器进行单例的控制的，而是spring框架对每一个bean只创建了一个对象。
* 模板方法模式。AbstractApplicationContext类中的finishBeanInitialization()方法调用了子类的getBean()方法，因为getBean()的实现和环境息息相关。
* 迭代器模式。对于MutablePropertyValues类定义使用到了迭代器模式，因为此类存储并管理PropertyValue对象，也属于一个容器，所以给该容器提供一个遍历方式。

## 7.2 剖析MyBatis框架中用到的经典设计模式

### 7.2.1 MyBatis回顾

#### 7.2.1.1 MyBatis与ORM框架

MyBatis 是一个 ORM（Object Relational Mapping，对象 - 关系映射）框架。ORM 框架主要是根据类和数据库表之间的映射关系，帮助程序员自动实现对象与数据库中数据之间的互相转化。

> ORM 负责将程序中的对象 存储到数据库中、将数据库中的数据转化为程序中的对象

<img src=".\img\149.jpg" alt="image-20220530160637842" style="zoom: 100%;" />
**Mybatis架构** 

<img src=".\img\150.jpg" alt="image-20220530160637842" style="zoom: 50%;" />   

```txt
1、mybatis配置

SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。

mapper.xml文件即sql映射文件，文件中配置了操作数据库的sql语句。此文件需要在SqlMapConfig.xml中加载。

2、通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂

3、由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行。

4、mybatis底层自定义了Executor执行器接口操作数据库，Executor接口有两个实现，一个是基本执行器、一个是缓存执行器。

5、Mapped Statement也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个Mapped Statement对象，sql的id即是Mapped statement的id。

6、Mapped Statement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数。

7、Mapped Statement对sql执行输出结果进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程。
```



#### 7.2.1.1 MyBatis的基础使用

因为 MyBatis 依赖 JDBC 驱动，所以，在项目中使用 MyBatis，除了需要引入 MyBatis 框 架本身（mybatis.jar）之外，还需要引入 JDBC 驱动（比如，访问 MySQL 的 JDBC 驱动 实现类库 mysql-connector-java.jar）。

1) 导入MyBatis的坐标和其他相关坐标

```xml
<!--mybatis坐标-->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.4</version>
</dependency>
<!--mysql驱动坐标-->
<dependency>    
    <groupId>mysql</groupId>   
    <artifactId>mysql-connector-java</artifactId>    
    <version>5.1.6</version>    
    <scope>runtime</scope>
</dependency>
<!--单元测试坐标-->
<dependency>    
    <groupId>junit</groupId>    
    <artifactId>junit</artifactId>    
    <version>4.12</version>    
    <scope>test</scope>
</dependency>
<!--日志坐标-->
<dependency>    
    <groupId>log4j</groupId>    
    <artifactId>log4j</artifactId>    
    <version>1.2.17</version>
</dependency>
```

2) 创建user数据表

```sql
CREATE DATABASE `mybatis_db`;
USE `mybatis_db`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` datetime default NULL COMMENT '生日',
  `sex` char(1) default NULL COMMENT '性别',
  `address` varchar(256) default NULL COMMENT '地址',
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- insert.... 略
```

3. 编写User实体

```java
public class User {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    // getter/setter 略
}
```

4)  编写UserMapper映射文件

```java
public interface UserMapper {

    public List<User> findAll();
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mashibing.mapper.UserMapper">
    <!--查询所有-->
    <select id="findAll" resultType="com.mashibing.domain.User">
        select * from user
    </select>
</mapper>
```

5. 编写MyBatis核心文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--环境配置-->
    <environments default="mysql">
        <!--使用MySQL环境-->
        <environment id="mysql">
            <!--使用JDBC类型事务管理器-->
            <transactionManager type="JDBC"></transactionManager>
            <!--使用连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"></property>
                <property name="url" value="jdbc:mysql:///mybatis_test"></property>
                <property name="username" value="root"></property>
                <property name="password" value="123456"></property>
            </dataSource>
        </environment>
    </environments>

    <!--加载映射配置-->
    <mappers>
        <mapper resource="com/mashibing/mapper/UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

6. 编写测试类

```java
public class MainApp {

    public static void main(String[] args) throws IOException {

        // 加载核心配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        // 获取SqlSessionFactory工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        // 获取SqlSession会话对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 执行sql
        List<User> list = sqlSession.selectList("com.mashibing.mapper.UserMapper.findAll");
        for (User user : list) {
            System.out.println(user);
        }
        // 释放资源
        sqlSession.close();
    }
}
```

### 7.2.2 MyBatis中使用到的设计模式

#### 7.2.2.1 Builder模式

Builder模式的定义是“将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。”，它属于创建类模式，建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。

《effective-java》中第2条也提到：**遇到多个构造器参数时，考虑用构建者(Builder)模式**。

<img src=".\img\52.jpg" alt="image-20220530160637842" style="zoom: 50%;" />

在Mybatis环境的初始化过程中，`SqlSessionFactoryBuilder`会调用`XMLConfigBuilder`读取所有的`MybatisMapConfig.xml`和所有的`*Mapper.xml`文件，构建Mybatis运行的核心对象`Configuration`对象，然后将该`Configuration`对象作为参数构建一个`SqlSessionFactory`对象。

<img src=".\img\154.jpg" alt="image-20220530160637842" style="zoom: 100%;" />  

其中`XMLConfigBuilder`在构建`Configuration`对象时，也会调用`XMLMapperBuilder`用于读取`*.Mapper`文件，而`XMLMapperBuilder`会使用`XMLStatementBuilder`来读取和build所有的SQL语句。

<img src=".\img\155.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

<img src=".\img\156.jpg" alt="image-20220530160637842" style="zoom: 100%;" />

<img src=".\img\157.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

在这个过程中，有一个相似的特点，就是这些Builder会读取文件或者配置，然后做大量的XpathParser解析、配置或语法的解析、反射生成对象、存入结果缓存等步骤，这么多的工作都不是一个构造函数所能包括的，因此大量采用了Builder模式来解决。

对于builder的具体类，方法都大都用`build*`开头，比如`SqlSessionFactoryBuilder`为例，它包含以下方法：

<img src=".\img\151.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

从建造者模式的设计初衷上来看，SqlSessionFactoryBuilder 虽然带有 Builder 后缀，但 不要被它的名字所迷惑，它并不是标准的建造者模式。一方面，原始类 SqlSessionFactory 的构建只需要一个参数，并不复杂。

另一方面，Builder 类SqlSessionFactoryBuilder 仍然定义了多包含不同参数列表的构造函数。 实际上，SqlSessionFactoryBuilder 设计的初衷只不过是为了简化开发。因为构建 SqlSessionFactory 需要先构建 Configuration，而构建 Configuration 是非常复杂的，需 要做很多工作，比如配置的读取、解析、创建 n 多对象等。为了将构建 SqlSessionFactory 的过程隐藏起来，对程序员透明，MyBatis 就设计了 SqlSessionFactoryBuilder 类封装这些构建细节。

#### 7.2.2.2 工厂模式

在Mybatis中比如`SqlSessionFactory`使用的是工厂模式，该工厂没有那么复杂的逻辑，是一个简单工厂模式。

简单工厂模式(Simple Factory Pattern)：又称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。在简单工厂模式中，可以根据参数的不同返回不同类的实例。简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

<img src=".\img\158.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

`SqlSession`可以认为是一个Mybatis工作的核心的接口，通过这个接口可以执行执行SQL语句、获取Mappers、管理事务。类似于连接MySQL的`Connection`对象。

<img src=".\img\159.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

可以看到，该Factory的`openSession（）`方法重载了很多个，分别支持`autoCommit`、`Executor`、`Transaction` 等参数的输入，来构建核心的`SqlSession`对象。

在`DefaultSqlSessionFactory`的默认工厂实现里，有一个方法可以看出工厂怎么产出一个产品：

<img src=".\img\160.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

openSessionFromDataSource方法,

<img src=".\img\161.jpg" alt="image-20220530160637842" style="zoom: 50%;" />

这是一个openSession调用的底层方法，该方法先从configuration读取对应的环境配置，然后初始化`TransactionFactory`获得一个`Transaction`对象，然后通过`Transaction`获取一个`Executor`对象，最后通过configuration、Executor、是否autoCommit三个参数构建了`SqlSession`。

#### 7.2.2.3 单例模式

单例模式(Singleton Pattern)：单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。

单例模式的要点有三个：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。单例模式是一种对象创建型模式。单例模式又名单件模式或单态模式。

<img src=".\img\34.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 	

在Mybatis中有两个地方用到单例模式，`ErrorContext`和`LogFactory`，其中`ErrorContext`是用在每个线程范围内的单例，用于记录该线程的执行环境错误信息，而`LogFactory`则是提供给整个Mybatis使用的日志工厂，用于获得针对项目配置好的日志对象。

```java
public class ErrorContext {

    private static final ThreadLocal<ErrorContext> LOCAL = new ThreadLocal<>();


    private ErrorContext() {
    }

    public static ErrorContext instance() {
        ErrorContext context = LOCAL.get();
        if (context == null) {
            context = new ErrorContext();
            LOCAL.set(context);
        }
        return context;
    }

}
```

构造函数是private修饰，具有一个static的局部instance变量和一个获取instance变量的方法，在获取实例的方法中，先判断是否为空如果是的话就先创建，然后返回构造好的对象。

只是这里有个有趣的地方是，LOCAL的静态实例变量使用了`ThreadLocal`修饰，也就是说它属于每个线程各自的数据，而在`instance()`方法中，先获取本线程的该实例，如果没有就创建该线程独有的`ErrorContext`。

#### 7.2.2.4 代理模式

代理模式可以认为是Mybatis的核心使用的模式，正是由于这个模式，我们只需要编写`Mapper.java`接口，不需要实现，由Mybatis后台帮我们完成具体SQL的执行。

代理模式(Proxy Pattern) ：给某一个对象提供一个代 理，并由代理对象控制对原对象的引用。代理模式的英 文叫做Proxy，它是一种对象结构型模式。

<img src=".\img\66.jpg" alt="image-20220530160637842" style="zoom: 80%;" /> 

这里有两个步骤，第一个是提前创建一个Proxy，第二个是使用的时候会自动请求Proxy，然后由Proxy来执行具体事务；

当我们使用`Configuration`的`getMapper`方法时，会调用`mapperRegistry.getMapper`方法，而该方法又会调用`mapperProxyFactory.newInstance(sqlSession)`来生成一个具体的代理：

<img src=".\img\162.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

<img src=".\img\163.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

```java
/**
 * Mapper Proxy 工厂类
 *
 * @author Lasse Voss
 */
public class MapperProxyFactory<T> {

    /**
     * Mapper 接口
     */
    private final Class<T> mapperInterface;
    /**
     * 方法与 MapperMethod 的映射
     */
    private final Map<Method, MapperMethod> methodCache = new ConcurrentHashMap<>();

    public MapperProxyFactory(Class<T> mapperInterface) {
        this.mapperInterface = mapperInterface;
    }

    public Class<T> getMapperInterface() {
        return mapperInterface;
    }

    public Map<Method, MapperMethod> getMethodCache() {
        return methodCache;
    }

    @SuppressWarnings("unchecked")
    protected T newInstance(MapperProxy<T> mapperProxy) {

        return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[]{mapperInterface}, mapperProxy);
    }

   //MapperProxyFactory类中的newInstance方法
    public T newInstance(SqlSession sqlSession) {
        // 创建了JDK动态代理的invocationHandler接口的实现类mapperProxy
        final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
        // 调用了重载方法
        return newInstance(mapperProxy);
    }

}
```

在这里，先通过`T newInstance(SqlSession sqlSession)`方法会得到一个`MapperProxy`对象，然后调用`T newInstance(MapperProxy<T> mapperProxy)`生成代理对象然后返回。

而查看`MapperProxy`的代码，可以看到如下内容：

```java
public class MapperProxy<T> implements InvocationHandler, Serializable {

    private static final long serialVersionUID = -6424540398559729838L;

    /**
     * SqlSession 对象
     */
    private final SqlSession sqlSession;
    /**
     * Mapper 接口
     */
    private final Class<T> mapperInterface;
    /**
     * 方法与 MapperMethod 的映射
     *
     * 从 {@link MapperProxyFactory#methodCache} 传递过来
     */
    private final Map<Method, MapperMethod> methodCache;

    // 构造，传入了SqlSession，说明每个session中的代理对象的不同的！
    public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethod> methodCache) {
        this.sqlSession = sqlSession;
        this.mapperInterface = mapperInterface;
        this.methodCache = methodCache;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            // 如果是 Object 定义的方法，直接调用
            if (Object.class.equals(method.getDeclaringClass())) {
                return method.invoke(this, args);

            } else if (isDefaultMethod(method)) {
                return invokeDefaultMethod(proxy, method, args);
            }
        } catch (Throwable t) {
            throw ExceptionUtil.unwrapThrowable(t);
        }
        // 获得 MapperMethod 对象
        final MapperMethod mapperMethod = cachedMapperMethod(method);
        // 重点在这：MapperMethod最终调用了执行的方法
        return mapperMethod.execute(sqlSession, args);
    }

}
```

非常典型的，该`MapperProxy`类实现了`InvocationHandler`接口，并且实现了该接口的`invoke`方法。

通过这种方式，我们只需要编写`Mapper.java`接口类，当真正执行一个`Mapper`接口的时候，就会转发给`MapperProxy.invoke`方法，而该方法则会调用后续的`sqlSession.cud>executor.execute>prepareStatement`等一系列方法，完成SQL的执行和返回。

#### 7.2.2.5 组合模式

组合模式(Composite Pattern) 的定义是：将对象组合成树形结构以表示整个部分的层次结构.组合模式可以让用户统一对待单个对象和对象的组合.

组合模式其实就是将一组对象(文件夹和文件)组织成树形结构,以表示一种'部分-整体' 的层次结构,(目录与子目录的嵌套结构). 组合模式让客户端可以统一单个对象(文件)和组合对象(文件夹)的处理逻辑(递归遍历).

<img src=".\img\96.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

Mybatis支持动态SQL的强大功能，比如下面的这个SQL：

```xml
<update id="update" parameterType="org.format.dynamicproxy.mybatis.bean.User">
    UPDATE users
    <trim prefix="SET" prefixOverrides=",">
        <if test="name != null and name != ''">
            name = #{name}
        </if>
        <if test="age != null and age != ''">
            , age = #{age}
        </if>
        <if test="birthday != null and birthday != ''">
            , birthday = #{birthday}
        </if>
    </trim>
    where id = ${id}
</update>
```

在这里面使用到了trim、if等动态元素，可以根据条件来生成不同情况下的SQL；

在`DynamicSqlSource.getBoundSql`方法里，调用了`rootSqlNode.apply(context)`方法，`apply`方法是所有的动态节点都实现的接口：

```java
/**
 * SQL Node 接口，每个 XML Node 会解析成对应的 SQL Node 对象
 * @author Clinton Begin
 */
public interface SqlNode {

    /**
     * 应用当前 SQL Node 节点
     *
     * @param context 上下文
     * @return 当前 SQL Node 节点是否应用成功。
     */
    boolean apply(DynamicContext context);
}
```

对于实现该`SqlSource`接口的所有节点，就是整个组合模式树的各个节点：

<img src=".\img\164.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

组合模式的简单之处在于，所有的子节点都是同一类节点，可以递归的向下执行，比如对于TextSqlNode，因为它是最底层的叶子节点，所以直接将对应的内容append到SQL语句中：

<img src=".\img\165.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

但是对于IfSqlNode，就需要先做判断，如果判断通过，仍然会调用子元素的SqlNode，即`contents.apply`方法，实现递归的解析。

<img src=".\img\166.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

#### 7.2.2.6 模板方法模式

**模板方法模式(template method pattern)原始定义是：在操作中定义算法的框架，将一些步骤推迟到子类中。模板方法让子类在不改变算法结构的情况下重新定义算法的某些步骤。**

模板方法中的算法可以理解为广义上的业务逻辑,并不是特指某一个实际的算法.定义中所说的算法的框架就是模板, 包含算法框架的方法就是模板方法.

模板类定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

<img src=".\img\109.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

在Mybatis中，sqlSession的SQL执行，都是委托给Executor实现的，Executor包含以下结构：

<img src=".\img\167.jpg" alt="image-20220530160637842" style="zoom: 100%;" />  

其中的BaseExecutor就采用了模板方法模式，它实现了大部分的SQL执行逻辑，然后把以下几个方法交给子类定制化完成：

<img src=".\img\177.jpg" alt="image-20220530160637842" style="zoom: 100%;" />  

```java
protected abstract int doUpdate(MappedStatement ms, Object parameter) throws SQLException;
```

该模板方法类有几个子类的具体实现，使用了不同的策略：

- 简单`SimpleExecutor`：每执行一次`update`或`select`，就开启一个`Statement`对象，用完立刻关闭`Statement`对象。（可以是`Statement`或`PrepareStatement`对象）
- 重用`ReuseExecutor`：执行`update`或`select`，以sql作为key查找`Statement`对象，存在就使用，不存在就创建，用完后，不关闭`Statement`对象，而是放置于`Map`内，供下一次使用。（可以是`Statement`或`PrepareStatement`对象）
- 批量`BatchExecutor`：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（`addBatch()`），等待统一执行（`executeBatch()`），它缓存了多个Statement对象，每个Statement对象都是`addBatch()`完毕后，等待逐一执行`executeBatch()`批处理的；`BatchExecutor`相当于维护了多个桶，每个桶里都装了很多属于自己的SQL，就像苹果蓝里装了很多苹果，番茄蓝里装了很多番茄，最后，再统一倒进仓库。（可以是Statement或PrepareStatement对象）

比如在SimpleExecutor中这样实现doUpdate方法：

<img src=".\img\168.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

模板模式基于继承来实现代码复用。如果抽象类中包含模板方法，模板方法调用有待子类实 现的抽象方法，那这一般就是模板模式的代码实现。而且，在命名上，模板方法与抽象方法 一般是一一对应的，抽象方法在模板方法前面多一个“do”，比如，在 BaseExecutor 类 中，其中一个模板方法叫 update()，那对应的抽象方法就叫 doUpdate()。

#### 7.2.2.7 适配器模式

适配器模式(Adapter Pattern) ：将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

<img src=".\img\88.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

在Mybatsi的logging包中，有一个Log接口：

<img src=".\img\169.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

该接口定义了Mybatis直接使用的日志方法，而Log接口具体由谁来实现呢？Mybatis提供了多种日志框架的实现，这些实现都匹配这个Log接口所定义的接口方法，最终实现了所有外部日志框架到Mybatis日志包的适配：

<img src=".\img\170.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

比如对于`Log4jImpl`的实现来说，该实现持有了`org.apache.log4j.Logger`的实例，然后所有的日志方法，均委托该实例来实现。

```java
/**
 * Copyright 2009-2017 the original author or authors.
 * <p>
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 * <p>
 * http://www.apache.org/licenses/LICENSE-2.0
 * <p>
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.apache.ibatis.logging.log4j;

import org.apache.ibatis.logging.Log;
import org.apache.log4j.Level;
import org.apache.log4j.Logger;

/**
 * @author Eduardo Macarron
 */
public class Log4jImpl implements Log {

    private static final String FQCN = Log4jImpl.class.getName();

    private final Logger log;

    public Log4jImpl(String clazz) {
        log = Logger.getLogger(clazz);
    }

    @Override
    public boolean isDebugEnabled() {
        return log.isDebugEnabled();
    }

    @Override
    public boolean isTraceEnabled() {
        return log.isTraceEnabled();
    }

    @Override
    public void error(String s, Throwable e) {
        log.log(FQCN, Level.ERROR, s, e);
    }

    @Override
    public void error(String s) {
        log.log(FQCN, Level.ERROR, s, null);
    }

    @Override
    public void debug(String s) {
        log.log(FQCN, Level.DEBUG, s, null);
    }

    @Override
    public void trace(String s) {
        log.log(FQCN, Level.TRACE, s, null);
    }

    @Override
    public void warn(String s) {
        log.log(FQCN, Level.WARN, s, null);
    }
}
```

在适配器模式中，传递给适配器构造函数的是被适配的类对象，而这里是 clazz（相当于日志名称 name），所以，从代码实现上来讲，它并非标准的适配器模式。但是，从应用 场景上来看，这里确实又起到了适配的作用，是典型的适配器模式的应用场景。

#### 7.2.2.8 装饰者模式

装饰模式(Decorator Pattern) ：动态地给一个对象增加一些额外的职责(Responsibility)，就增加对象功能来说，装饰模式比生成子类实现更为灵活。其别名也可以称为包装器(Wrapper)，与适配器模式的别名相同，但它们适用于不同的场合。根据翻译的不同，装饰模式也有人称之为“油漆工模式”，它是一种对象结构型模式。

<img src=".\img\82.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

在mybatis中，缓存的功能由根接口`Cache（org.apache.ibatis.cache.Cache）`定义。整个体系采用装饰器设计模式，数据存储和缓存的基本功能由`PerpetualCache（org.apache.ibatis.cache.impl.PerpetualCache）`永久缓存实现，然后通过一系列的装饰器来对`PerpetualCache`永久缓存进行缓存策略等方面的控制。如下图：

<img src=".\img\171.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

用于装饰PerpetualCache的标准装饰器共有8个（全部在org.apache.ibatis.cache.decorators包中）：

<img src=".\img\172.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

1. `FifoCache`：先进先出算法，缓存回收策略
2. `LoggingCache`：输出缓存命中的日志信息
3. `LruCache`：最近最少使用算法，缓存回收策略
4. `ScheduledCache`：调度缓存，负责定时清空缓存
5. `SerializedCache`：缓存序列化和反序列化存储
6. `SoftCache`：基于软引用实现的缓存管理策略
7. `SynchronizedCache`：同步的缓存装饰器，用于防止多线程并发访问
8. `WeakCache`：基于弱引用实现的缓存管理策略

之所以 MyBatis 采用装饰器模式来实现缓存功能，是因为装饰器模式采用了组合，而非继 承，更加灵活，能够有效地避免继承关系的组合爆炸。



##### MyBatis一级缓存

缓存就是内存中的数据，常常来自对数据库查询结果的保存。使用缓存，我们可以避免频繁的与数据库进行交互，进而提高响应速度MyBatis也提供了对缓存的支持，分为一级缓存和二级缓存，可以通过下图来理解：

<img src=".\img\179.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

①、一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

②、二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的



**一级缓存默认是开启的**

①、我们使用同一个sqlSession，对User表根据相同id进行两次查询，查看他们发出sql语句的情况

```java
   public static void main(String[] args) throws IOException {

        // 加载核心配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");

        // 获取SqlSessionFactory工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

        // 获取SqlSession会话对象
        SqlSession sqlSession = sqlSessionFactory.openSession();

        User user1 = sqlSession.selectOne("com.mashibing.mapper.UserMapper.findById",1);

        User user2 = sqlSession.selectOne("com.mashibing.mapper.UserMapper.findById",1);
        System.out.println(user1);
        System.out.println(user2);

        System.out.println(user1 == user2);

        // 释放资源
        sqlSession.close();
    }


//打印结果
User{id=1, username='tom', birthday=Sat Oct 01 17:27:27 CST 2022, sex='0', address='北京'}
User{id=1, username='tom', birthday=Sat Oct 01 17:27:27 CST 2022, sex='0', address='北京'}
true
```

②  同样是对user表进行两次查询，只不过两次查询之间进行了一次update操作。

```java
public static void main(String[] args) throws IOException {

    // 加载核心配置文件
    InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");

    // 获取SqlSessionFactory工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);

    // 获取SqlSession会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();


    User user1 = sqlSession.selectOne("com.mashibing.mapper.UserMapper.findById",1);
    System.out.println(user1);

    sqlSession.delete("com.mashibing.mapper.UserMapper.deleteById",3);
    sqlSession.commit();

    User user2 = sqlSession.selectOne("com.mashibing.mapper.UserMapper.findById",1);

    System.out.println(user2);

    System.out.println(user1 == user2);

    // 释放资源
    sqlSession.close();
}

//结果
User{id=1, username='tom', birthday=Sat Oct 01 17:27:27 CST 2022, sex='0', address='北京'}
User{id=1, username='tom', birthday=Sat Oct 01 17:27:27 CST 2022, sex='0', address='北京'}
false
```

③、**总结**

1、第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从 数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。

2、 如果中间sqlSession去执行commit操作（执行插入、更新、删除），则会清空SqlSession中的 一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

3、 第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直 接从缓存中获取用户信息



**1. 一级缓存 底层数据结构到底是什么？**

之前说`不同SqlSession的一级缓存互不影响`,所以我从SqlSession这个类入手

<img src=".\img\180.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

可以看到，`org.apache.ibatis.session.SqlSession`中有一个和缓存有关的方法——`clearCache()`刷新缓存的方法，点进去，找到它的实现类`DefaultSqlSession`

```java
  @Override
  public void clearCache() {
    executor.clearLocalCache();
  }
```

再次点进去`executor.clearLocalCache()`,再次点进去并找到其实现类`BaseExecutor`，

```java
  @Override
  public void clearLocalCache() {
    if (!closed) {
      localCache.clear();
      localOutputParameterCache.clear();
    }
  
```

进入`localCache.clear()`方法。进入到了`org.apache.ibatis.cache.impl.PerpetualCache`类中

```java
package org.apache.ibatis.cache.impl;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import org.apache.ibatis.cache.Cache;
import org.apache.ibatis.cache.CacheException;
/**
 * @author Clinton Begin
 */
public class PerpetualCache implements Cache {
  private final String id;

  private Map<Object, Object> cache = new HashMap<Object, Object>();

  public PerpetualCache(String id) {
    this.id = id;
  }

  //省略部分...
  @Override
  public void clear() {
    cache.clear();
  }
  //省略部分...
}

```

我们看到了`PerpetualCache`类中有一个属性`private Map<Object, Object> cache = new HashMap<Object, Object>()`，很明显它是一个HashMap，我们所调用的`.clear()`方法，**实际上就是调用的Map的clear方法**

<img src=".\img\181.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

**得出结论：**

一级缓存的数据结构确实是HashMap

<img src=".\img\182.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

##### 一级缓存的执行流程 

我们进入到`org.apache.ibatis.executor.Executor`中
看到一个方法`CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql)` ，见名思意是一个创建CacheKey的方法
找到它的实现类和方法`org.apache.ibatis.executor.BaseExecuto.createCacheKey`

<img src=".\img\183.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

我们分析一下创建CacheKey的这块代码：

```java
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    //初始化CacheKey
    CacheKey cacheKey = new CacheKey();
    //存入statementId
    cacheKey.update(ms.getId());
    //分别存入分页需要的Offset和Limit
    cacheKey.update(rowBounds.getOffset());
    cacheKey.update(rowBounds.getLimit());
    //把从BoundSql中封装的sql取出并存入到cacheKey对象中
    cacheKey.update(boundSql.getSql());
    //下面这一块就是封装参数
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();

    for (ParameterMapping parameterMapping : parameterMappings) {
      if (parameterMapping.getMode() != ParameterMode.OUT) {
        Object value;
        String propertyName = parameterMapping.getProperty();
        if (boundSql.hasAdditionalParameter(propertyName)) {
          value = boundSql.getAdditionalParameter(propertyName);
        } else if (parameterObject == null) {
          value = null;
        } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
          value = parameterObject;
        } else {
          MetaObject metaObject = configuration.newMetaObject(parameterObject);
          value = metaObject.getValue(propertyName);
        }
        cacheKey.update(value);
      }
    }
    //从configuration对象中（也就是载入配置文件后存放的对象）把EnvironmentId存入
        /**
     *     <environments default="development">
     *         <environment id="development"> //就是这个id
     *             <!--当前事务交由JDBC进行管理-->
     *             <transactionManager type="JDBC"></transactionManager>
     *             <!--当前使用mybatis提供的连接池-->
     *             <dataSource type="POOLED">
     *                 <property name="driver" value="${jdbc.driver}"/>
     *                 <property name="url" value="${jdbc.url}"/>
     *                 <property name="username" value="${jdbc.username}"/>
     *                 <property name="password" value="${jdbc.password}"/>
     *             </dataSource>
     *         </environment>
     *     </environments>
     */
    if (configuration.getEnvironment() != null) {
      // issue #176
      cacheKey.update(configuration.getEnvironment().getId());
    }
    //返回
    return cacheKey;
  }
```

我们再点进去`cacheKey.update()`方法看一看

```java

public class CacheKey implements Cloneable, Serializable {
  private static final long serialVersionUID = 1146682552656046210L;
  public static final CacheKey NULL_CACHE_KEY = new NullCacheKey();
  private static final int DEFAULT_MULTIPLYER = 37;
  private static final int DEFAULT_HASHCODE = 17;

  private final int multiplier;
  private int hashcode;
  private long checksum;
  private int count;
  //值存入的地方
  private transient List<Object> updateList;
  //省略部分方法......
  //省略部分方法......
  public void update(Object object) {
    int baseHashCode = object == null ? 1 : ArrayUtil.hashCode(object); 
    count++;
    checksum += baseHashCode;
    baseHashCode *= count;
    hashcode = multiplier * hashcode + baseHashCode;
    //看到把值传入到了一个list中
    updateList.add(object);
  }
 
  //省略部分方法......
}
```

我们知道了那些数据是在CacheKey对象中如何存储的了。下面我们返回`createCacheKey()`方法。

<img src=".\img\1.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

我们可以看到，创建CacheKey后调用了query()方法，我们再次点进去：

<img src=".\img\2.jpg" alt="image-20220530160637842" style="zoom: 100%;" />

在执行SQL前如何在一级缓存中找不到Key，那么将会执行sql，我们来看一下执行sql前后会做些什么，进入`list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);`

```java
 private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    //1. 把key存入缓存，value放一个占位符
	localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      //2. 与数据库交互
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
      //3. 如果第2步出了什么异常，把第1步存入的key删除
      localCache.removeObject(key);
    }
      //4. 把结果存入缓存
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
      localOutputParameterCache.putObject(key, parameter);
    }
    return list;
  }
```

总结

1. 一级缓存的数据结构是一个`HashMap<Object,Object>`，它的value就是查询结果，它的key是`CacheKey`，`CacheKey`中有一个**list**属性，`statementId,params,rowbounds,sql`等参数都存入到了这个**list**中
2. 先创建`CacheKey`，会首先根据`CacheKey`查询缓存中有没有，如果有，就处理缓存中的参数，如果没有，就执行sql，执行sql后执行sql后把结果存入缓存

#### 7.2.2.9 迭代器模式

迭代器（Iterator）模式，又叫做游标（Cursor）模式。GOF给出的定义为：提供一种方法访问一个容器（container）对象中各个元素，而又不需暴露该对象的内部细节。

<img src=".\img\121.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

在软件系统中,容器对象拥有两个职责: 一是存储数据,而是遍历数据.从依赖性上看,前者是聚合对象的基本职责.而后者是可变化的,又是可分离的.因此可以将遍历数据的行为从容器中抽取出来,封装到迭代器对象中,由迭代器来提供遍历数据的行为,这将简化聚合对象的设计,更加符合单一职责原则.

Java的`Iterator`就是迭代器模式的接口，只要实现了该接口，就相当于应用了迭代器模式：

<img src=".\img\174.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

比如Mybatis的`PropertyTokenizer`是property包中的重量级类，该类会被reflection包中其他的类频繁的引用到。这个类实现了`Iterator`接口，在使用时经常被用到的是`Iterator`接口中的`hasNext`这个函数。

```java
/**
 * 属性分词器
 *
 * @author Clinton Begin
 */
public class PropertyTokenizer implements Iterator<PropertyTokenizer> {

    /**
     * 当前字符串
     */
    private String name;
    /**
     * 索引的 {@link #name} ，因为 {@link #name} 如果存在 {@link #index} 会被更改
     */
    private final String indexedName;
    /**
     * 编号。
     *
     * 对于数组 name[0] ，则 index = 0
     * 对于 Map map[key] ，则 index = key
     */
    private String index;
    /**
     * 剩余字符串
     */
    private final String children;

    public PropertyTokenizer(String fullname) {
        // 初始化 name、children 字符串，使用 . 作为分隔
        int delim = fullname.indexOf('.');
        if (delim > -1) {
            name = fullname.substring(0, delim);
            children = fullname.substring(delim + 1);
        } else {
            name = fullname;
            children = null;
        }
        // 记录当前 name
        indexedName = name;
        // 若存在 [ ，则获得 index ，并修改 name 。
        delim = name.indexOf('[');
        if (delim > -1) {
            index = name.substring(delim + 1, name.length() - 1);
            name = name.substring(0, delim);
        }
    }

    public String getName() {
        return name;
    }

    public String getIndex() {
        return index;
    }

    public String getIndexedName() {
        return indexedName;
    }

    public String getChildren() {
        return children;
    }

    @Override
    public boolean hasNext() {
        return children != null;
    }

    @Override
    public PropertyTokenizer next() {
        return new PropertyTokenizer(children);
    }

    @Override
    public void remove() {
        throw new UnsupportedOperationException("Remove is not supported, as it has no meaning in the context of properties.");
    }

}
```

可以看到，这个类传入一个字符串到构造函数，然后提供了iterator方法对解析后的子串进行遍历，是一个很常用的方法类。

实际上，PropertyTokenizer 类也并非标准的迭代器类。它将配置的解析、解析之后的元 素、迭代器，这三部分本该放到三个类中的代码，都耦合在一个类中，所以看起来稍微有点 难懂。不过，这样做的好处是能够做到惰性解析。我们不需要事先将整个配置，解析成多个 PropertyTokenizer 对象。只有当我们在调用 next() 函数的时候，才会解析其中部分配 置。

## 7.3 总结

上面给大家讲解的就是Spring和MyBatis框架中所使用到的设计模式. 要再次强调的是, 对于同学们来说不需要去记忆哪个类用到了哪个模式, 死记硬背是没有意义的,同学们最好是下载一些优秀框架的源码,比如Spring或者MyBatis,然后抽出时间好好的阅读一下源码,锻炼自己阅读理解源码的能力.

除此之外同学们应该也有发现,其实框架对很多设计模式的实 现，都并非标准的代码实现，都做了比较多的自我改进。实际上，这就是所谓的灵活应用, 只借鉴不照搬, 根据具体问题针对性地去解决。









