# Spring

## 1、什么是Spring？

Spring是一个轻量级的IoC和AOP容器框架。

是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。

常见的配置方式有三种：

- 基于XML的 配置
- 基于注解的配置
- 基于Java的配置





## 2、Spring的IOC和AOP机制

**spring的IoC容器是spring的核心，spring AOP是spring框架的重要组成部分。**

两者主要用到的设计模式有工厂模式和代理模式。

> IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成 可重用的功能组件。

- ### **IoC（Inversion of Control，控制反转）**

1. **概念**：
   Spring的IoC容器是框架的核心，它负责管理对象的创建、初始化、生命周期和依赖注入。通过IoC，组件之间的依赖关系不再由代码主动控制，而是交由容器来管理。
2. **实现方式**：
   - **依赖注入（DI）**：通过构造器注入、Setter注入或注解（如`@Autowired`）的方式，将组件需要的依赖传递给它。
   - **配置方式**：XML配置、Java配置（`@Configuration`）、注解方式（`@ComponentScan`等）。
3. **设计模式**：
   - 工厂模式：Spring IoC容器通过工厂模式创建和管理Bean对象。
   - 单例模式：默认情况下，Spring容器中Bean是单例的，通过配置可以调整为原型模式（Prototype）。
4. **优点**：
   - 降低耦合度，提高系统的灵活性。
   - 更容易进行单元测试，因为依赖可以通过注入进行替换。

---

- ### **AOP（Aspect-Oriented Programming，面向切面编程）**

1. **概念**：
   Spring AOP允许在不修改原始业务逻辑的情况下，为程序的不同部分添加通用功能（如日志、事务管理、安全检查）。
2. **关键术语**：
   - **切面（Aspect）**：封装横切关注点的模块。
   - **连接点（Join Point）**：程序执行过程中可插入切面的点（如方法调用）。
   - **切入点（Pointcut）**：定义特定连接点的规则。
   - **通知（Advice）**：实际执行的动作（如`@Before`、`@After`）。
   - **织入（Weaving）**：将切面应用到目标对象的过程（动态代理或编译时处理）。
3. **实现方式**：
   - 代理模式：
     - Spring AOP基于动态代理实现（JDK动态代理和CGLIB）。
     - JDK动态代理用于接口代理，CGLIB用于类代理（创建子类）。
4. **应用场景**：
   - 日志记录、性能监控。
   - 事务管理。
   - 安全认证和权限控制。

可以说是对OOP的补充和完善因为OOP以封装、继承和多态特性来建立一种对象层次结构， 用以模拟公共行为的一个集合。适合是上到下的关系，不适合左到右。不用切面的话会导致大量重复代码，不利于各模块的复用。

- 实现AOP的技术，主要分为两大类：

  - 一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；
  - 二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编 译器可以在编译期间织入有关“方面”的代码.

- 要注意要导入：AspectJ包

  AspectJ是静态代理的增强（所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强）他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的 AOP对象

------

- ### **IoC与AOP的关系**

1. **协作**：
   IoC和AOP是Spring框架的重要基石，IoC通过依赖注入管理Bean的生命周期和依赖关系，AOP则为Bean提供增强功能。两者的结合使Spring框架在复杂企业应用开发中表现出色。
2. **相辅相成**：
   - IoC容器提供了切面的配置和管理能力。
   - AOP增强的功能可以依赖IoC管理的Bean来实现。





## 3、Spring的IoC理解 

## 4、Spring的AOP理解

### 简单说说动态代理

1. **动态代理的核心组件**：

- **Proxy**：JDK 提供的代理类，用于生成动态代理对象。

- **InvocationHandler**：接口，代理对象的方法调用会被转发到 InvocationHandler 的 invoke 方法中，由它来实现具体的逻辑。

2. **动态代理的特点**：

- 代理模式是 Java 中常用的设计模式，其核心特征是代理类和委托类（被代理类）实现相同的接口。

- 动态代理是在运行时动态生成代理类，而不是在编译时定义静态代理类。

```java
Hello hello = new HelloImpl();
Hello proxy = (Hello) Proxy.newProxyInstance(
        hello.getClass().getClassLoader(),
        hello.getClass().getInterfaces(),
        new HelloProxy(hello)
);

class HelloProxy implements InvocationHandler {
```



### 怎么实现AOP 

1. jdk ：JdkDynamicAopProxy
2. cglib ：CglibAopProxy

**实现 AOP 的两种方式**

1. **基于 JDK 动态代理** (JdkDynamicAopProxy)

- 适用于代理 **实现了接口** 的目标类。

- 通过 JDK 提供的 Proxy 和 InvocationHandler 来实现。

  `代码如上`

2. **基于 CGLIB 动态代理** (CglibAopProxy)

- 适用于代理 **没有实现接口** 的目标类。

- 使用 CGLIB 库生成目标类的子类，通过重写方法实现代理。

```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

// 目标类
class ProductService {
    public void addProduct() {
        System.out.println("Adding a product!");
    }
}

// AOP 代理逻辑
class CglibInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method: " + method.getName());
        return result;
    }
}

// 测试
public class CglibAopExample {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ProductService.class);
        enhancer.setCallback(new CglibInterceptor());

        // 创建代理对象
        ProductService proxy = (ProductService) enhancer.create();
        proxy.addProduct();
    }
}

//result
Before method: addProduct
Adding a product!
After method: addProduct
```



## 5、 Spring中Autowired和Resource关键字的区别

不同：

- 导入的包不同：
  - 一个是springframework.beans.factory.annotation.Autowired的
  - 另一个是包javax.annotation.Resource

- `@Autowired` 默认 byType 的方式实现，如果类型找不到，再找名字，两否报错【常用】
- `@Resource`  默认 byName 的方式实现，如果名字找不到，就会通过类型，两否报错

相同：

- 都是用来自动装配的，都可以放在属性字段上

但现在更好应该使用构造器注入



## 6、依赖注入的方式有几种，各是什么？

- ### 1. 构造器注入

  **概念**：通过构造器将依赖注入到目标对象中。
  
  **优点**：
  
  - 保证依赖对象在实例化时就完全初始化，符合不可变对象的设计理念。
  - 对依赖必需性有更强的约束。
  
  **缺点**：
  
  - 无法解决循环依赖问题。
  
  **示例代码**：
  
  ```java
  @Component
  public class UserService {
      private final UserRepository userRepository;
  
      // 构造器注入
      @Autowired
      public UserService(UserRepository userRepository) {
          this.userRepository = userRepository;
      }
  }
  ```
  
  ------
  
  ### 2. Setter方法注入
  
  **概念**：通过Setter方法将依赖注入到目标对象中。
  
  **优点**：
  
  - 灵活，可以选择性注入。
  - 可以解决循环依赖问题。
  
  **缺点**：
  
  - 依赖注入可能延迟到调用Setter方法后，增加了对象状态不一致的风险。
  
  **示例代码**：
  
  ```java
  @Component
  public class OrderService {
      private ProductService productService;
  
      // Setter方法注入
      @Autowired
      public void setProductService(ProductService productService) {
          this.productService = productService;
      }
  }
  ```
  
  ------
  
  ### 3. 接口注入
  
  **概念**：通过接口方法注入依赖，对目标对象实现特定接口，由容器调用接口方法注入依赖对象。
  
  **优点**：
  
  - 解耦合，依赖可以直接通过接口进行注入。
  - 接口的实现类无需显式了解具体依赖。
  
  **缺点**：
  
  - 使用较少，不是Spring的主流方式。
  
  **示例代码**：
  
  ```java
  public interface DependencyInjector {
      void injectDependency(Dependency dependency);
  }
  
  @Component
  public class MyService implements DependencyInjector {
      private Dependency dependency;
  
      // 接口注入
      @Override
      public void injectDependency(Dependency dependency) {
          this.dependency = dependency;
      }
  }
  
  @Configuration
  public class AppConfig {
      @Bean
      public Dependency dependency() {
          return new Dependency();
      }
  
      @Bean
      public MyService myService(Dependency dependency) {
          MyService myService = new MyService();
          myService.injectDependency(dependency);
          return myService;
      }
  }
  ```
  
  ------
  
  ### 4. 工厂注入（非标准分类，但在实际应用中常见）
  
  **概念**：通过工厂模式提供依赖对象的实例，而非直接注入。
  
  **优点**：
  
  - 用于需要复杂初始化过程的依赖。
  - 实现高定制化。
  
  **示例代码**：
  
  ```java
  public class PaymentService {
      private final String apiKey;
      private final String environment;
  
      public PaymentService(String apiKey, String environment) {
          this.apiKey = apiKey;
          this.environment = environment;
      }
  
      public void processPayment(double amount) {
          System.out.printf("Processing payment of %.2f in %s environment with API key: %s%n", 
                            amount, environment, apiKey);
      }
  }
  
  @Configuration
  public class PaymentServiceFactory {
      @Bean
      public PaymentService createPaymentService() {
          // 模拟从配置文件或环境变量中读取配置
          String apiKey = System.getenv("PAYMENT_API_KEY");
          String environment = System.getenv("PAYMENT_ENV");
  
          if (apiKey == null || environment == null) {
              throw new IllegalArgumentException("Missing required payment configuration");
          }
  
          return new PaymentService(apiKey, environment);
      }
  }
  
  @Component
  public class PaymentController {
      private final PaymentService paymentService;
  
      @Autowired
      public PaymentController(PaymentService paymentService) {
          this.paymentService = paymentService;
      }
  
      public void executePayment() {
          paymentService.processPayment(100.00);
      }
  }
  ```
  
  - @Bean 方法返回的对象在 **容器中注册为单例（默认情况下）**，可以在多个地方共享使用。
  
  
  
  ------
  
  ### 关于循环依赖问题
  
  - **构造器注入**：不能解决循环依赖，因为两个类互相依赖时会造成死循环。
  - **Setter注入**：Spring通过延迟注入（提前暴露Bean的引用）可以解决循环依赖问题。
  
  **示例代码（Setter方法解决循环依赖）**：
  
  ```java
  @Component
  public class A {
      private B b;
  
      @Autowired
      public void setB(B b) {
          this.b = b;
      }
  }
  
  @Component
  public class B {
      private A a;
  
      @Autowired
      public void setA(A a) {
          this.a = a;
      }
  }
  ```
  
  ------
  
  ### 总结
  
  - 构造器注入适合**必须依赖**，对象初始化后即完全可用。
  - Setter注入适合**可选依赖**，需要灵活配置或解决循环依赖问题时使用。
  - 接口注入较少使用，但在特殊场景下有解耦优势。
  - 工厂注入常用于复杂对象的初始化，是Spring中广泛应用的一种设计模式。





## 7、解释一下spring bean的生命周期

首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy

bean的与他相似——简单说：首先实例化后，设置对象属性，如果有Aware接口或其他的接口则调用相关的方法，没有Bean就创建完成，当Bean不在需要时候，进入就销毁了。

1. **实例化Bean：**

   对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，容器会调用createBean进行实例化。
   对于ApplicationContext容器，当容器启动结束后，通过获取BeanDeﬁnition对象中的信息，确定实例化哪一类。 

   **过程**：Spring容器根据配置（XML、注解或Java代码）加载Bean的定义，并通过**反射**或**工厂方法**创建Bean实例。

   **触发点**：在Spring容器初始化或延迟加载时，创建Bean实例。

2. **设置对象属性（依赖注入）:**

   实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDeﬁnition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。 
   源码课里描述：

   - populateBean：填充属性——在从bean定义的属性值给出BeanWrapper的填充bean实例。

   **过程**：Spring容器根据依赖注入的配置（构造器、Setter注入或注解）为Bean的字段或属性赋值。

   **工具**：使用反射或注解解析器完成属性的注入。

3. **处理Aware（联想）接口：**

   接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

   ①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值； 

   ```java
   @Component
   public class MyBean implements BeanNameAware {
       @Override
       public void setBeanName(String name) {
           System.out.println("Bean name is: " + name);
       }
   }
   ```

   ②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。 

   ③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

   等等；

4. **BeanPostProcesser——Before**

   如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用 postProcessBeforeInitialization(Object obj, String s)方法。

   ```java
   @Component
   public class MyBeanPostProcessor implements BeanPostProcessor {
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) {
           System.out.println("Before Initialization: " + beanName);
           return bean;
       }
   }
   ```


5. **初始化**

   执行 init-method 方法，如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法

   - **过程**：如果Bean实现了InitializingBean接口，Spring会调用afterPropertiesSet方法。
     - 如果配置了自定义的初始化方法（`<bean init-method="init"/>` 或 `@Bean(initMethod = "init")`），会执行该方法。

   ```java
   @Component
   public class MyBean implements InitializingBean {
       @Override
       public void afterPropertiesSet() {
           System.out.println("Bean is initialized.");
       }
   }
   ```

6. **BeanPostProcesser——After**

   如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；
   由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术

   > 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

7. **清理阶段：** 

   当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的 destroy()方法；  

8. **销毁方法：**

   最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法

（7、8很少用，所以不用管）



## 8、如果我想在spring生命周期的不同阶段做不同的处理工作，我应该怎么办？

观察者模式：监听器、监听事件、多播器。（目前不懂，spring中较少，但在springboot中非常多）



## 9、解释Spring支持的几种bean的作用域

Spring容器中的bean可以分为5个范围：

1. singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。
2. prototype：为每一个bean请求提供一个实例。
3. request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
4. session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean 会随之失效。
5. global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器 中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。 





## 10、Spring框架中都用到了哪些设计模式？

1. 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；
2. 单例模式：Bean默认为单例模式。
3. 代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；
4. 模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
5. 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它 的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。 







## 其余问题

aop的三种实现方式，以及里面都有什么 aspect advice prointcut
execution语法。

事务实现方法：

事务的传播特性有哪几种：
分别是：



# SpringMVC

## 1、什么是SpringMVC

springMVC是一个MVC的**开源框架**

**springMVC和spring是什么样的关系呢？**

其实springMVC就是spring的一个子模块，就是spring在原有基础上，又提供了web应用的MVC模块，可以简单的把springMVC理解为是spring的一个模块（就像AOP， IOC这样的模块），根本不需要同spring进行整合。 



## 2、SpringMVC 的流程

简单说：

1. 用户通过视图层发送请求给前端控制器DispatcherServlet
2. 读取applicationContext.xml文件
3. 通过applicationContext.xml访问到Controller类
4. 在Controller类中将处理完的结果ModelAndView返回给applicationContext.xml中配置的ViewResolver
5. ViewResolver把结果再传递给视图解析器
6. 最后响应给视图层

详细版：

1. **DispatcherServlet** 接收并拦截视图层发来的请求。

2. **HandlerMapping** 根据请求的 URL 查找对应的处理器（Handler），将其返回给 DispatcherServlet。

3. **HandlerAdapter** 适配具体的处理器（Handler），按照特定规则执行它（如调用Controller控制器方法）。

4. 控制器（Controller）处理业务逻辑，并将结果封装为一个 **ModelAndView** 对象返回给 HandlerAdapter。

5. **HandlerAdapter** 接收 ModelAndView 对象，将视图逻辑名和模型数据返回给 DispatcherServlet。

6. **DispatcherServlet** 调用 **ViewResolver** 解析逻辑视图名，确定具体的视图路径（如 /WEB-INF/jsp/hello.jsp）。

7. DispatcherServlet 将模型数据传递给视图，渲染视图。

8. 最终，视图以 HTML 等形式呈现给用户。

   ```mermaid
   sequenceDiagram
       participant Client
       participant DispatcherServlet
       participant HandlerMapping
       participant HandlerAdapter
       participant Controller
       participant ViewResolver
       participant View
   
       Client->>DispatcherServlet: 发起请求
       DispatcherServlet->>HandlerMapping: 查找 Handler
       HandlerMapping-->>DispatcherServlet: 返回 Handler
       DispatcherServlet->>HandlerAdapter: 调用适配器
       HandlerAdapter->>Controller: 执行具体控制器
       Controller-->>HandlerAdapter: 返回 ModelAndView
       HandlerAdapter-->>DispatcherServlet: 返回视图逻辑名和模型数据
       DispatcherServlet->>ViewResolver: 调用视图解析器
       ViewResolver-->>DispatcherServlet: 返回具体视图路径
       DispatcherServlet->>View: 渲染视图
       View-->>Client: 返回渲染后的页面
   ```

   



## 3、SpringMVC怎样设定重定向和转发？

- 重定向：在返回值前面加"redirect:"，譬如"redirect:http://www.baidu.com"

- 转发：在返回值前面加"forward:"，譬如"forward:user.do?name=method4"





## 4、SpringMVC常用的注解有哪些？

@RequestMapping：用于处理请求 url 映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径。
@RequestBody：注解实现接收http请求的json数据，将json转换为java对象。

@ResponseBody：注解实现将conreoller方法返回对象转化为json对象响应给客户。

@RestController：



# SpringBoot

## 1、什么是SpringBoot？为什么要用SpringBoot？

一开始的spring春天很好，但后来发展成了配置地狱所以有了现在的springboot

他用来简化spring应用的初始搭建以及开发过程，使用特定的方式来进行配置（properties或yml文件)

- 创建独立的spring引用程序 main方法运行 
- 嵌入的Tomcat 无需部署war文件 
- 简化maven配置 
- 自动配置spring添加对应功能starter自动化配置 



优点非常多：

1. 独立运行

   Spring Boot而且内嵌了各种servlet容器，Tomcat、Jetty等，现在不再需要打成war包部署到容器中， Spring Boot只要打成一个可执行的jar包就能独立运行，所有的依赖包都在一个jar包内。

2. 简化配置

   spring-boot-starter-web启动器自动依赖其他组件，简少了maven的配置。 

3. 自动配置

   Spring Boot能根据当前类路径下的类、jar包来自动配置bean，如添加一个spring-boot-starter-web启动器就能拥有web的功能，无需其他配置。 

4. 无代码生成和XML配置

   Spring Boot配置过程中无代码生成，也无需XML配置文件就能完成所有配置工作，这一切都是借助于 条件注解完成的，这也是Spring4.x的核心功能之一。

5. 应用监控

   Spring Boot提供一系列端点可以监控服务及应用，做健康检测。





## 2、Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下3个注解：

- @SpringBootConﬁguration：

  **作用**：这是一个特殊的 **@Configuration** 注解，标志着当前类是一个 Spring 配置类，可以定义 Bean。

  **本质**：等同于 **@Configuration**，使其具备了传统 Java 配置类的功能。

  

- @EnableAutoConﬁguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： 

  `@SpringBootApplication(exclude = { DataSourceAutoConﬁguration.class })`

  使用`@EnableAutoConfiguration(excludeName = {"com.example.MyAutoConfiguration"})` 排除特定的自动配置类。

  **核心机制**：通过 spring.factories 文件加载 META-INF/spring.factories 中定义的自动配置类。

  补充：@EnableAutoConfiguration **优先级低于用户自定义配置**。如果用户在配置文件中显式配置了某些 Bean，自动配置会被覆盖。

  

- @ComponentScan：

  **作用**：启用组件扫描，自动发现并注册被 **@Component**、**@Service**、**@Repository** 等注解标记的 Spring 组件。 





## 3、运行Spring Boot有哪几种方式？

1）打包用命令或者放到容器中运行

2）用 Maven/Gradle 插件运行

3）直接执行 main 方法运行 



## 4、如何理解 SpringBoot 中的 Starters？

Starters是什么：

- Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成Spring 及其他技术，而不需要到处找示例代码和依赖包。

  如你想使用Spring thymeleaf访问数据库，只要加入spring-boot-starter-thymeleaf启动器依赖就能使用了

- 第三方的 启动器不能以spring-boot开头命名，它们都被Spring Boot官方保留。一般一个第三方的应该这样命名，像mybatis的mybatis-spring-boot-starter



## 5、Springboot常用的starter有哪些？ 

- spring-boot-starter-web 嵌入tomcat和web开发需要servlet与jsp支持 
- spring-boot-starter-data-jpa 数据库支持
- spring-boot-starter-data-redis redis数据库支持 
- spring-boot-starter-data-solr solr支持 
- mybatis-spring-boot-starter 第三方的mybatis集成starter





## 6、如何在SpringBoot启动的时候运行一些特定的代码？ 

如果你想在Spring Boot启动的时候运行一些特定的代码，你可以实现接口**ApplicationRunner**或者 **CommandLineRunner**

这两个接口实现方式一样，它们都只提供了一个run方法。

CommandLineRunner：启动获取命令行参数 





## 7、SpringBoot 需要独立的容器运行吗？ 

 可以不需要，内置了 Tomcat/ Jetty 等容器。 





## 8、Spring Boot中的监视器是什么？ 

Spring boot actuator是spring启动框架中的重要功能之一。

Spring boot监视器可帮助您访问生产环境 中正在运行的应用程序的当前状态。

有几个指标必须在生产环境中进行检查和监控。即使一些外部应用 程序可能正在使用这些服务来向相关人员触发警报消息。

监视器模块公开了一组可直接作为HTTP URL 访问的REST端点来检查状态。





## 9、如何使用Spring Boot实现异常处理？

Spring提供了一种使用ControllerAdvice处理异常的非常有用的方法。 

我们通过实现一个 ControllerAdvice类，来处理控制器类抛出的所有异常。 

@ControllerAdvice



## 10、SpringBoot 实现热部署有哪几种方式？ 

主要有两种方式： 

- Spring Loaded 
- Spring-boot-devtools





## 11、如何理解 SpringBoot 配置加载顺序？

- 同一个名称的配置文件存在优先级 properties > yaml > yml

- 在官方文档中可知

  从中可了解到

  1：项目路径下的config文件夹配置文件
  2：项目路径下配置文件
  3：资源路径下的config文件夹配置文件
  4：资源路径下配置文件

优先级由高到底，高优先级的配置会覆盖低优先级的配置





## 12、SpringBoot 的核心配置文件有哪几个？它们的区别是什么？ 

spring Boot 的核心配置文件是 application 和 bootstrap 配置文件。 

application 配置文件这个容易理解，主要用于 SpringBoot 项目的自动化配置。

bootstrap 配置文件有以下几个应用场景。 

- 使用 Spring Cloud Conﬁg 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的 配置属性来加载外部配置中心的配置信息；

- 一些固定的不能被覆盖的属性；

- 一些加密/解密的场景； 





## 13、如何集成 SpringBoot 和 ActiveMQ？ 

使用 spring-boot-starter-activemq 依赖关系。 

它只需要很少的配置，并且不需要样板代码。 



## 14、SpringBoot、Spring MVC 和 Spring 有什么区别

1. Spring

   Spring重要的特征是依赖注入。所有 SpringModules 不是依赖注入就是 IOC 控制反转。 恰当的使用 DI 或是 IOC 的时候，可以开发松耦合应用。松耦合应用的单元测试可以很容易的进行。

2. Spring MVC 

   Spring MVC 提供了一种分离式的方法来开发 Web 应用。通过运用像 DispatcherServelet， MoudlAndView 和 ViewResolver 等一些简单的概念，开发 Web 应用将会变的非常简单。

3. SpringBoot

   由于Spring 和 SpringMVC 的问题在于需要配置大量的参数。

   SpringBoot 通过一个自动配置和启动的项来目解决这个问题。





## 15、能否举一个例子来解释更多 Staters 的内容？ 

用Spring Boot Stater Web 为例子。

依赖项可以被分为：

- Spring - core，beans，context，aop 
- Web MVC - （Spring MVC） 
- Jackson - for JSON Binding 
- Validation - Hibernate,Validation API 
- Enbedded Servlet Container - Tomcat 
- Logging - logback,slf4j 任何经典的 Web 应用程序都会使用所有这些依赖项。Spring Boot Starter Web 预先打包了这些依赖项。

作为一个开发者，我不需要再担心这些依赖项和它们的兼容版本





## 讲讲 SpringBoot 里的自动配置问题







