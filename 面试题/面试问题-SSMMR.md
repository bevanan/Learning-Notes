# MyBatis

## 1、什么是MyBatis?

- Mybatis是一款优秀的持久层框架，它内部封装了JDBC，开发时只需要关注SQL语句本身，免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作，不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。

- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。



## 2、MyBatis的优点和缺点

**优点：**

- 基于SQL语句编程，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除sql与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用。

- 与JDBC相比，减少了50%以上的代码量，消除了JDBC大量冗余的代码，不需要手动开关连接
- 与各种数据库兼容
- 能够与Spring很好的集成

**缺点：**

- SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求。
- SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

这两个都不怎么能算缺点，GPT查一下吧



## 3、#{}和${}的区别是什么？ 

- #{}是预编译处理，${}是字符串替换**（拼接）**。

- #可以防止sql注入，$不可以
- ${}是危险的



## 4、动态SQL

9个标签

if 、choose (when, otherwise)

trim (where, set)、foreach

SQL片段 `<sql></sql>`

```xml
<select id="findUsers" resultType="User">
    SELECT * FROM users
    WHERE 1=1
    <if test="username != null">
        AND username = #{username}
    </if>
    <choose> #类似java中的switch
       <when test="userType == 'admin'">
           AND role = 'admin'
       </when>
       <when test="userType == 'guest'">
           AND role = 'guest'
       </when>
       <otherwise>
           AND role = 'user'
       </otherwise>
    </choose>
    
    .. id IN 
    <foreach item="id" collection="ids" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

- `<set>`标签用于动态生成 `UPDATE` 语句的 `SET` 部分，自动去除多余的逗号。

  ```xml
  <update id="updateUser">
      UPDATE users
      <set>
          <if test="username != null">username = #{username},</if>
          <if test="age != null">age = #{age},</if>
      </set>
      WHERE id = #{id}
  </update>
  ```



- **`<collection>`**，它用于处理嵌套的查询结果。通常与一对多的关系查询结合使用，尤其是当查询结果中包含了一个集合属性时，比如查询一个用户及其所有订单，或者一个班级及其所有学生。

  ```java
  public class User {
      private Integer id;
      private String username;
      private List<Order> orders;  // 一对多的关系，用户有多个订单
  }
  
  public class Order {
      private Integer id;
      private String orderNumber;
      private Integer userId;
  }
  ```

  sql对应的resultMap

  ```xml
  <resultMap id="userOrderResultMap" type="User">
      <id property="id" column="userId"/>
      <result property="username" column="username"/>
      <!-- 使用 <collection> 映射用户的订单列表 -->
      <collection property="orders" ofType="Order">
          <id property="id" column="orderId"/>
          <result property="orderNumber" column="order_number"/>
      </collection>
  </resultMap>
  ```

  这个实际帮助特别大，主要非常的省事。



- `<association>`与`<collection>`相似，只是一对一。

  **实体类：**

  ```java
  public class User {
      private int id;
      private String name;
      private Address address;  // 关联的 Address 对象
  }
  
  public class Address {
      private int id;
      private String street;
      private String city;
  }
  ```

  **Mapper XML 配置：**

  你可以在 `UserMapper.xml` 中使用 `<association>` 来映射 `User` 类的 `address` 属性到 `Address` 类。

  ```xml
  <resultMap id="userResultMap" type="User">
      <id property="id" column="id"/>
      <result property="name" column="name"/>
      <!-- 使用 <association> 映射关联的 Address 对象 -->
      <association property="address" javaType="Address" column="address_id" select="selectAddressById"/>
  </resultMap>
  
  <!-- 查询 User 的 SQL -->
  <select id="selectUser" resultMap="userResultMap">
      SELECT id, name, address_id FROM user WHERE id = #{id}
  </select>
  
  <!-- 查询 Address 的 SQL -->
  <select id="selectAddressById" resultType="Address">
      SELECT id, street, city FROM address WHERE id = #{id}
  </select>
  ```



**两个组合使用**

```java
public class User {
    private int id;
    private String name;
    private Address address;  // 一对一关系
    private List<Order> orders;  // 一对多关系
}

public class Address {
    private int id;
    private String street;
    private String city;
}

public class Order {
    private int id;
    private int userId;
    private double amount;
}
```

```xml
<resultMap id="userResultMap" type="User">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    
    <!-- 映射 Address（一对一关系） -->
    <association property="address" javaType="Address" column="address_id" select="selectAddressById"/>
    <!-- 映射 Orders（一对多关系） -->
    <collection property="orders" javaType="Order" ofType="Order" column="id" select="selectOrdersByUserId"/>
</resultMap>

<!-- 查询 User -->
<select id="selectUser" resultMap="userResultMap">
    SELECT id, name, address_id FROM user WHERE id = #{id}
</select>

<!-- 查询 Address -->
<select id="selectAddressById" resultType="Address">
    SELECT id, street, city FROM address WHERE id = #{id}
</select>
<!-- 查询 Orders -->
<select id="selectOrdersByUserId" resultType="Order">
    SELECT id, user_id, amount FROM order WHERE user_id = #{userId}
</select>
```





### Xml映射文件中，除了常见的select|insert|updae|delete标 签之外，还有哪些标签？

加上以上的几个



## 5、Mybatis缓存

### 一级缓存

- 一级缓存叫本地缓存：
  - 与数据库同一次会话期间查询到的数据会放在本地缓存中，其存储作用域为 **SqlSession**
  - 之后有需要话获取相同的数据，直接缓存中取，没必要再去查询数据库

- 有效期：
  - 就是SqlSession开启到关闭的期间
  - 默认开始，且关不掉

### 二级缓存

- 二级缓存也叫全局缓存，因为一级缓存作用域太低了，所以有了二级缓存

- 基于namespace级别的缓存。一个名称空间，对应一个二级缓存

  不同的mapper查出的数据会放在自己对应的缓存（map）中

- 通过在要开启缓存的Mapper中开启。`<cache/>`开启

- 只要开启了二级缓存，在用一个Mapper下就有效
- 所有的数据都会先放在一级缓存中
- 只有当会话提交，或者关闭的时候，才会提交到二级缓存中
- 查看顺序
  1. 先看二级缓冲中有没有
  2. 再看一级缓存中有没有
  3. 查询数据库



---

### 5.1. **一级缓存（Per-Session Cache）**

- **作用范围**：一级缓存是默认启用的，作用范围是 **SqlSession**。也就是说，在同一个 `SqlSession` 对象中，查询的结果会被缓存，直到该 `SqlSession` 被关闭。

- **工作原理**：在同一个 `SqlSession` 内，如果你执行相同的查询语句，MyBatis 会从缓存中直接返回结果，而不去查询数据库。这个缓存是自动启用的，并且 **无法配置**。

- 清空时机

  ：当执行以下操作时，一级缓存会被清空：

  - `SqlSession` 被关闭。
  - 执行了 `commit` 或 `rollback` 操作。
  - 手动调用 `clearCache()` 方法清空缓存。

#### 示例：

```xml
<select id="findUserById" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>
```

在同一个 `SqlSession` 中，如果多次查询相同的 `id`，MyBatis 会直接从缓存中返回结果，而不会再次查询数据库。

### 5.2. **二级缓存（Per-Mapper Cache）**

- **作用范围**：二级缓存的作用范围是 **SqlSessionFactory**，也就是说，在同一个 `SqlSessionFactory` 下，不同的 `SqlSession` 可以共享缓存数据。二级缓存不是默认启用的，需要手动配置。

- **工作原理**：当你执行查询时，如果查询的数据在二级缓存中，则 MyBatis 会直接从缓存中获取数据，而不去访问数据库。二级缓存主要用于跨 `SqlSession` 重用数据。

- 清空时机

  ：二级缓存的清空时机比一级缓存更灵活，通常在以下情况下被清空：

  - 执行了增、删、改等写操作时。
  - `SqlSession` 被关闭时，二级缓存中存储的数据会自动刷新。

#### 配置二级缓存：

1. **启用二级缓存**：在 MyBatis 配置文件中启用二级缓存。

   ```xml
   <settings>
       <setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. **为每个 Mapper 启用缓存**： 在每个 `Mapper` 文件中配置缓存：

   ```xml
   <mapper namespace="com.example.mapper.UserMapper">
       <cache/>
       <select id="findUserById" resultType="User">
           SELECT * FROM users WHERE id = #{id}
       </select>
   </mapper>
   ```

   这里的 `<cache/>` 标签启用了该 `Mapper` 的二级缓存。

### 3. **二级缓存的高级配置**

- **缓存清空**：可以通过配置 `<cache>` 标签来设置缓存的清空策略、过期时间等。

  - `flushInterval`：设置缓存刷新的时间间隔（单位毫秒）。
  - `size`：设置缓存的最大大小。
  - `eviction`：设置缓存的淘汰策略（如 LRU、FIFO 等）。

  例如：

  ```xml
  <cache eviction="LRU" size="512" flushInterval="60000"/>
  ```

- **自定义缓存**：如果需要定制缓存的实现，可以通过实现 `org.apache.ibatis.cache.Cache` 接口来创建自己的缓存实现。

### 4. **一级缓存和二级缓存的区别**

| 特性             | 一级缓存                             | 二级缓存                                 |
| ---------------- | ------------------------------------ | ---------------------------------------- |
| **作用范围**     | 仅限于单个 `SqlSession`              | 跨 `SqlSession`，即跨多个会话共享缓存    |
| **是否启用**     | 默认启用                             | 默认禁用，需要配置启用                   |
| **缓存生命周期** | 随 `SqlSession` 生命周期而存在       | 随 `SqlSessionFactory` 生命周期而存在    |
| **存储位置**     | 存储在 `SqlSession` 内存中           | 存储在外部缓存中（如文件、内存、数据库） |
| **清空时机**     | 关闭 `SqlSession` 或执行写操作时清空 | 执行写操作或手动刷新时清空               |

### 5. **缓存的最佳实践**

- **尽量避免在频繁修改数据的场景中使用缓存**，例如对于涉及大量数据更新、插入、删除的操作，缓存的效果较差，可能会导致缓存不一致的问题。
- **使用二级缓存时要注意缓存一致性问题**，在有写操作时，确保相关缓存及时失效，避免读取到过时数据。
- **为缓存设置合适的过期时间**，避免缓存数据过时。

### 总结：

- **一级缓存** 是 MyBatis 的默认缓存机制，作用范围是单个 `SqlSession`。
- **二级缓存** 需要手动启用，作用范围是跨多个 `SqlSession`。
- 使用缓存时要小心缓存一致性和过期策略，避免产生脏数据或缓存不一致的问题。



### 其中部分知识点

在 MyBatis 中，`SqlSession` 代表了与数据库的一个会话。通常，你会在一个方法中创建一个 `SqlSession`，然后通过这个 `SqlSession` 去访问不同的 `Mapper` 接口。例如，假设你有多个不同的 `Mapper` 接口（比如 `UserMapper` 和 `OrderMapper`），你可以使用同一个 `SqlSession` 来执行它们的数据库操作。



## 6、Mybatis如何进行分页

SQL：limit

或者RowBounds



## 7、如何执行批量插入？ 

```java
//  注意这里 executortype.batch 
sqlsession sqlsession = sqlsessionfactory.opensession(executortype.batch);    
try {     
    namemapper mapper = sqlsession.getmapper(namemapper.class);     
    for (string name : names) {         
        mapper.insertname(name);     
    }     
    sqlsession.commit();    
}catch(Exception e){
    .....
}
```



## 8、Mybatis是否支持延迟加载(懒加载)？如果支持，它的实现原理是什么？ 

ResultMap中的assosiation和collection标签具有延迟加载的功能

### 8.1. **MyBatis 中的延迟加载**

MyBatis 的延迟加载支持分为 **两种模式**：

- **懒加载（Lazy Loading）**：通过懒加载，MyBatis 会延迟加载某些对象的属性，直到你访问这些属性时才从数据库中查询数据。
- **懒加载集合**：对于关联的集合属性（如 `List` 或 `Set`），MyBatis 可以在访问集合时延迟加载其元素。

### 8.2. **延迟加载的实现原理**

MyBatis 中的延迟加载主要是通过 **代理模式** 实现的。以下是它的实现原理：

- **代理对象**：MyBatis 会为需要延迟加载的属性或对象生成代理对象。当你访问这个代理对象的属性或方法时，代理会触发实际的数据库查询，加载相关的数据。
- **SqlSession 的配置**：在 MyBatis 配置文件中，可以通过设置 `lazyLoadingEnabled` 来启用或禁用延迟加载。

#### 延迟加载的配置：

你可以在 MyBatis 的 `mybatis-config.xml` 中通过以下配置来启用延迟加载：

```xml
<settings>
  <!-- 启用延迟加载 -->
  <setting name="lazyLoadingEnabled" value="true"/>
  
  <!-- 配置延迟加载的程度，true 表示所有关联属性都使用懒加载 -->
  <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

#### 关键配置项：

- **lazyLoadingEnabled**：启用或禁用延迟加载功能，默认为 `false`。如果启用，MyBatis 会创建代理对象。
- **aggressiveLazyLoading**：决定是否对所有的关联属性都启用懒加载。默认为 `false`，表示只有在明确指定懒加载的情况下，才会延迟加载；如果设置为 `true`，则所有关联属性都会启用懒加载。

### 8.3. **延迟加载的工作流程**

当延迟加载开启时，MyBatis 会为映射的对象创建代理（通常是使用 `CGLIB` 或 `JDK` 动态代理），代理对象会拦截对属性或方法的访问请求。当你访问懒加载的属性时，代理对象会触发实际的数据库查询，从而加载数据。

#### 举个例子：

假设你有一个 `User` 类，它有一个关联的 `Order` 类：

```java
public class User {
    private Integer id;
    private String username;
    private List<Order> orders;  // 这个属性将会被懒加载
}

public class Order {
    private Integer id;
    private Integer userId;
    private String product;
}
```

对应的映射文件 `UserMapper.xml` 可能会是这样的：

```xml
<resultMap id="userResultMap" type="User">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <association property="orders" column="user_id" javaType="List" select="selectOrdersByUserId"/>
</resultMap>

<select id="selectOrdersByUserId" resultType="Order">
    SELECT * FROM orders WHERE user_id = #{userId}
</select>
```

#### 延迟加载工作过程：

1. 当你查询 `User` 时，`orders` 属性不会立即加载。
2. 当你访问 `user.getOrders()` 时，代理对象会拦截这个方法调用，并触发 `selectOrdersByUserId` 查询，加载相关的 `Order` 数据。

### 8.4. **懒加载和急加载的区别**

- **懒加载（Lazy Loading）**：只有在访问关联属性时，MyBatis 才会执行 SQL 查询。例如，查询 `User` 数据时，不会立即查询 `orders` 数据，只有当你访问 `user.getOrders()` 时，才会查询 `Order` 表。
- **急加载（Eager Loading）**：在查询主对象时，会同时加载所有的关联属性。例如，在查询 `User` 时，`orders` 数据会被立即加载。

你可以通过 `fetchType` 属性来控制是否使用懒加载：

```xml
<association property="orders" fetchType="lazy" javaType="List" select="selectOrdersByUserId"/>
```

`fetchType` 的默认值是 `eager`，表示急加载。如果设置为 `lazy`，则会启用懒加载。

### 8.5. **延迟加载的优缺点**

**优点：**

- **减少不必要的数据库查询**：如果关联的属性并不是每次都需要，懒加载可以避免不必要的 SQL 查询，提高性能。
- **提高响应速度**：初始查询时，不会加载所有的关联数据，减少了数据库访问的开销。

 **缺点：**

- **潜在的 N+1 查询问题**：如果懒加载的关联属性被频繁访问，可能会导致多个数据库查询，尤其是集合属性的懒加载，可能会造成 N+1 查询问题，即查询一次主对象时，随后又会执行多次查询来加载关联的集合。

  **解决方法**：可以使用 `@One` 或 `@Many` 的 `fetchType` 配置为 `EAGER`，或者使用 `join fetch` 或 `select` 语句进行批量加载。

- **性能开销**：每次访问懒加载的属性时，都会执行一次查询，这可能导致性能开销。



当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。 



## 9、MyBatis实现一对一有几种方式?具体怎么操作的？

有联合查询和嵌套查询

- 联合查询是几个表联合查询，只查询一次，通过在resultMap里面配置association 节点配置一对一的类就可以完成
- 嵌套查询是先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据，也是通过 association配置，但另外一个表的查询通过select属性配置。 





## 10、Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

1. 使用标签，逐一定义数据库列名和对象属性名之间的映射关系。
2. 使用sql列的别名功能，将列的别名书写为对象属性名。 有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并 返回，那些找不到映射关系的属性，是无法完成赋值的。 





# Spring

## 1、什么是Spring？

Spring是一个轻量级的IoC和AOP容器框架。

是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。

常见的配置方式有三种：

- 基于XML的 配置
- 基于注解的配置
- 基于Java的配置





## 2、 Spring的IOC和AOP机制

**spring的IoC容器是spring的核心，spring AOP是spring框架的重要组成部分。**

两者主要用到的设计模式有工厂模式和代理模式。

> IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成 可重用的功能组件。





## 3、Spring的IoC理解 

IoC是控制反转，依赖注入，其中利用了工厂模式 

- 之前创建对象的主动权和时机都是自己把控的，而现在把这种控制权的转移给Spring容器中，容器根据配置文件去创建实例和管理各个实例之间的依赖关系，对象与对象之间松散耦合，这样利于功能的复用。
- 最明显的感受是，使用IoC，创建对象就不用再new了，可以由spring自动生产。
- 原理是使用java的反射机制，根据配置文件在运行时动态的去创建对象。





## 4、Spring的AOP理解

AOP面向切面就是典型的代理模式



- 可以说是对OOP的补充和完善因为OOP以封装、继承和多态特性来建立一种对象层次结构， 用以模拟公共行为的一个集合。适合是上到下的关系，不适合左到右。不用切面的话会导致大量重复代码，不利于各模块的复用。
- 实现AOP的技术，主要分为两大类：
  - 一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；
  - 二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编 译器可以在编译期间织入有关“方面”的代码.

- 要注意要导入：AspectJ包

  AspectJ是静态代理的增强（所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强）他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的 AOP对象



### 简单说说动态代理

- 代理模式讲解，需要了解两个类：Proxy：代理，InvocationHandler：由代理实例的*调用处理程序实现的接口*

- 代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类 预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。



### 怎么实现AOP 

1. jdk ：JdkDynamicAopProxy
2. cglib ：CglibAopProxy

——笔记Spring



## 5、 Spring中Autowired和Resource关键字的区别

不同：

- 导入的包不同：
  - 一个是springframework.beans.factory.annotation.Autowired的
  - 另一个是包javax.annotation.Resource

- @Autowired 默认byType的方式实现 ，找不到类型，再找名字，两否报错【常用】
- @Resource 默认byName 的方式实现，如果名字找不到，就会通过类型 ，两否报错

相同：

- 都是用来自动装配的，都可以放在属性字段上





## 6、依赖注入的方式有几种，各是什么？

（设置属性）：接口注入不确定，在源码课里说到的是工厂

- 构造器注入（无法解决循环依赖的问题）

  优点： 对象初始化完成后便可获得可使用的对象

- setter方法注入 （可以解决循环依赖问题）【重点】

  优点： 灵活。可以选择性地注入需要的对象

- 接口注入 

  优点： 接口注入中，接口的名字、函数的名字都不重要，只要保证函数的参数是要注入的对象类型即可





## 7、解释一下spring bean的生命周期

首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy

bean的与他相似——简单说：首先实例化后，设置对象属性，如果有Aware接口或其他的接口则调用相关的方法，没有Bean就创建完成，当Bean不在需要时候，进入就销毁了。

1. 实例化Bean：

   对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，容器会调用createBean进行实例化。
   对于ApplicationContext容器，当容器启动结束后，通过获取BeanDeﬁnition对象中的信息，确定实例化哪一类。 

2. 设置对象属性（依赖注入）:

   实例化后的对象被封装在BeanWrapper对象中，紧接着，Spring根据BeanDeﬁnition中的信息 以及 通过BeanWrapper提供的设置属性的接口完成依赖注入。 
   源码课里描述：

   - populateBean：填充属性——在从bean定义的属性值给出BeanWrapper的填充bean实例。

3. 处理Aware（联想）接口：

   接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：

   ①如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值； 

   ②如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的是Spring工厂自身。 

   ③如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文；

   等等；

4. BeanPostProcesser——Before

   如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用 postProcessBeforeInitialization(Object obj, String s)方法。

5. 执行 init-method 方法，如果Bean在Spring配置文件中配置了 init-method 属性，则会自动调用其配置的初始化方法

6. BeanPostProcesser——After

   如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术

   > 以上几个步骤完成后，Bean就已经被正确创建了，之后就可以使用这个Bean了。

7. 清理阶段：

   当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的 destroy()方法；  

8. 销毁方法：

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

1. DispatchedServlet接收并拦截视图层发来的请求。

2. HandlerMapping处理器映射，DispatcherServlet调用它，HandlerMapping根据请求URL去查找Handler（处理器)

3. HandlerExecution表示具体的Handler，作用是根据URL查找控制器

4. HandlerExecution将解析后的信息传递给DispatcherServlet，如解析控制器等。

5. HanderlAdapter表示处理器适配器，其按照特定的规则去执行Handeler

6. Handler让具体的Controller执行

7. Controller将具体的执行信息ModelView返回给HanderAdapter

8. HandlerAdapter将视图逻辑名和模型传递给DispatcherServlet。

9. DispatcherServlet调用视图解析器ViewResolver来解析HandlerAdapter传递的逻辑视图名
   获取了Model的数据
   解析了View的视图名字
   拼接前缀后缀视图名字，找到对应的视图 /WEB-INF/jsp/hello.jsp
   将数据渲染到这个视图上

10. 视图解析器将解析的逻辑视图名传给DispatcherServlet

11. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图

12. 最终视图呈现给用户



## 3、SpringMVC怎样设定重定向和转发？

- 转发：在返回值前面加"forward:"，譬如"forward:user.do?name=method4"
- 重定向：在返回值前面加"redirect:"，譬如"redirect:http://www.baidu.com"



## 4、SpringMVC常用的注解有哪些？

@RequestMapping：用于处理请求 url 映射的注解，可用于类或方法上。用于类上，则表示类中的所有响应请求的方法都是以该地址作为父路径。
@RequestBody：注解实现接收http请求的json数据，将json转换为java对象。

@ResponseBody：注解实现将conreoller方法返回对象转化为json对象响应给客户。

@RestController：



# SpringBoot

## 1、什么是SpringBoot？为什么要用SpringBoot ？

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

- @SpringBootConﬁguration：组合了 @Conﬁguration 注解，实现配置文件的功能。

- @EnableAutoConﬁguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： 

  @SpringBootApplication(exclude = { DataSourceAutoConﬁguration.class })。 

- @ComponentScan：Spring组件扫描。 





## 3、运行Spring Boot有哪几种方式？

1）打包用命令或者放到容器中运行

2）用 Maven/Gradle 插件运行

3）直接执行 main 方法运行 



## 4、如何理解 Spring Boot 中的 Starters？

Starters是什么：

- Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成Spring 及其他技术，而不需要到处找示例代码和依赖包。

  如你想使用Spring thymeleaf访问数据库，只要加入spring-boot-starter-thymeleaf启动器依赖就能使用了

- 第三方的 启动器不能以spring-boot开头命名，它们都被Spring Boot官方保留。一般一个第三方的应该这样命 名，像mybatis的mybatis-spring-boot-starter



## 5、 springboot常用的starter有哪些？ 

- spring-boot-starter-web 嵌入tomcat和web开发需要servlet与jsp支持 
- spring-boot-starter-data-jpa 数据库支持
- spring-boot-starter-data-redis redis数据库支持 
- spring-boot-starter-data-solr solr支持 
- mybatis-spring-boot-starter 第三方的mybatis集成starter





## 6、 如何在Spring Boot启动的时候运行一些特定的代码？ 

如果你想在Spring Boot启动的时候运行一些特定的代码，你可以实现接口**ApplicationRunner**或者 **CommandLineRunner**

这两个接口实现方式一样，它们都只提供了一个run方法。

CommandLineRunner：启动获取命令行参数 





## 7、 Spring Boot 需要独立的容器运行吗？ 

 可以不需要，内置了 Tomcat/ Jetty 等容器。 





## 8、 Spring Boot中的监视器是什么？ 

Spring boot actuator是spring启动框架中的重要功能之一。

Spring boot监视器可帮助您访问生产环境 中正在运行的应用程序的当前状态。

有几个指标必须在生产环境中进行检查和监控。即使一些外部应用 程序可能正在使用这些服务来向相关人员触发警报消息。

监视器模块公开了一组可直接作为HTTP URL 访问的REST端点来检查状态。





## 9、 如何使用Spring Boot实现异常处理？

Spring提供了一种使用ControllerAdvice处理异常的非常有用的方法。 

我们通过实现一个 ControllerAdvice类，来处理控制器类抛出的所有异常。 





## 10、 SpringBoot 实现热部署有哪几种方式？ 

主要有两种方式： 

- Spring Loaded 
- Spring-boot-devtools





## 11、如何理解 Spring Boot 配置加载顺序？

- 同一个名称的配置文件存在优先级 properties > yaml > yml

- 在官方文档中可知

  从中可了解到

  1：项目路径下的config文件夹配置文件
  2：项目路径下配置文件
  3：资源路径下的config文件夹配置文件
  4：资源路径下配置文件

​		优先级由高到底，高优先级的配置会覆盖低优先级的配置





## 12、 Spring Boot 的核心配置文件有哪几个？它们的区别是什么？ 

spring Boot 的核心配置文件是 application 和 bootstrap 配置文件。 

application 配置文件这个容易理解，主要用于 Spring Boot 项目的自动化配置。

bootstrap 配置文件有以下几个应用场景。 

- 使用 Spring Cloud Conﬁg 配置中心时，这时需要在 bootstrap 配置文件中添加连接到配置中心的 配置属性来加载外部配置中心的配置信息；

- 一些固定的不能被覆盖的属性；

- 一些加密/解密的场景； 





## 13、如何集成 Spring Boot 和 ActiveMQ？ 

使用 spring-boot-starter-activemq 依赖关系。 

它只需要很少的配置，并且不需要样板代码。 



## 14、Spring Boot、Spring MVC 和 Spring 有什么区别

1. Spring

   Spring重要的特征是依赖注入。所有 SpringModules 不是依赖注入就是 IOC 控制反转。 恰当的使用 DI 或是 IOC 的时候，可以开发松耦合应用。松耦合应用的单元测试可以很容易的进行。

2. Spring MVC 

   Spring MVC 提供了一种分离式的方法来开发 Web 应用。通过运用像 DispatcherServelet， MoudlAndView 和 ViewResolver 等一些简单的概念，开发 Web 应用将会变的非常简单。

3. SpringBoot

   由于Spring 和 SpringMVC 的问题在于需要配置大量的参数。

   Spring Boot 通过一个自动配置和启动的项来目解决这个问题。





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





## 讲讲SpringBoot里的自动配置问题









# MySQL

## 1、数据库的三范式是什么 

1. 第一范式（1NF）：（列不可分）字段具有原子性,不可再分。(所有关系型数据库系 统都满足第一范式数据库表中的字段都是单一属性的，不可再分) 
2. 第二范式（2NF）是在第一范式（1NF）的基础上建立起来的，*即满足 第二范式（2NF）必须先满足第一范式（1NF）*。要求数据库表中的每 个实例或行必须可以被惟一地区分。通常需要为表加上一个列，以存储 各个实例的惟一标识。这个惟一属性列被称为主关键字或主键。（不能存在传递依赖）
3. 满足第三范式（3NF）必须先满足第二范式（2NF）。简而言之，第三 范式（3NF）要求一个数据库表中不包含已在其它表中已包含的非主关 键字信息。 >所以第三范式具有如下特征： >>1. 每一列只有一个 值 >>2. 每一行都能区分。 >>3. 每一个表都不包含其他表已经包含 的非主关键字信息。

## 2、数据库引擎有哪些 

如何查看mysql提供的所有存储引擎

```
mysql> show engines;
```



mysql常用引擎包括：MYISAM、Innodb、Memory、MERGE

- MYISAM：全表锁，拥有较高的执行速度，不支持事务，不支持外键，并发性能差，占用空间相对较小，对事务完整性没有要求，以select、insert为主的应用基本上可以使用这引擎 
- Innodb：行级锁，提供了具有提交、回滚和崩溃回复能力的事务安全，支持自动增长列，支持外键约束，并发能力强，占用空间是MYISAM的2.5倍，处理效率相对会差一些 
- Memory：全表锁，存储在内容中，速度快，但会占用和数据量成正比的内存空间且数据在mysql重启时会丢失，默认使用HASH索引，检索效率非常高，但不适用于精确查找，主要用于那些内容变化不频繁的代码表 
- MERGE：是一组MYISAM表的组合 



## 3、InnoDB与MyISAM的区别

1. InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交， 这样会影响速度，所以好把多条SQL语言放在begin和commit之间，组成一个事务； 
2. InnoDB支持外键，而MyISAM不支持。对一个包含外键的InnoDB表转为MYISAM会失败； 
3. InnoDB是聚集索引，数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但 是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大， 因为主键太大，其他索引也都会很大。而MyISAM是非聚集索引，数据文件是分离的，索引保存的 是数据文件的指针。主键索引和辅助索引是独立的。 
4. InnoDB不保存表的具体行数，执行select count(*) from table时需要全表扫描。而MyISAM用一 个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快； 
5. Innodb不支持全文索引，而MyISAM支持全文索引，查询效率上MyISAM要高；



**如何选择引擎？**

如果没有特别的需求，使用默认的 Innodb 即可。

MyISAM：以读写插入为主的应用程序，比如博客系统、新闻门户网站。 

Innodb：更新（删除）操作频率也高，或者要保证数据的完整性；并发量高，支持事务和外键。比如 OA自动化办公系统。



## 4、数据库的事务 

**什么是事务？**： 多条sql语句，要么全部成功，要么全部失败。 

**事务的特性：**
**数据库事务特性：原子性(Atomic)、一致性(Consistency)、隔离性(Isolation)、持久性(Durabiliy)。 简称ACID。**

- 原子性：组成一个事务的多个数据库操作是一个不可分割的原子单元，只有所有操作都成功，整个事务才会提交。任何一个操作失败，已经执行的任何操作都必须撤销，让数据库返回初始状态。
-  一致性：事务操作成功后，数据库所处的状态和它的业务规则是一致的。即数据不会被破坏。如 A 转账100元给 B，不管操作是否成功，A 和 B 的账户总额是不变的。 
- 隔离性：在并发数据操作时，不同的事务拥有各自的数据空间，它们的操作不会对彼此产生干扰 
- 持久性：一旦事务提交成功，事务中的所有操作都必须持久化到数据库中





## 5、索引问题 

> 索引是对数据库表中一个或多个列的值进行排序的结构，建立索引有助于快速获取信息。 

你也可以这样理解：索引就是加快检索表中数据的方法。数据库的索引类似于书籍的索引。在书籍中，索引允许用户不必翻阅完整个书就能迅速地找到所需要的信息。在数据库中，索引也允许数据库程序迅速地找到表中的数据，而不必扫描整个数据库。 

`mysql `有4种不同的索引：

- 主键索引（PRIMARY） 

  数据列不允许重复，不允许为NULL，一个表只能有一个主键。 

- 唯一索引（UNIQUE） 

  数据列不允许重复，允许为NULL值，一个表允许多个列创建唯一索引。 

  - 可以通过 ` ALTER TABLE table_name ADD UNIQUE (column); ` 创建唯一索引 
  - 可以通过 `ALTER TABLE table_name ADD UNIQUE (column1,column2); ` 创建唯一组合索引 

- 普通索引（INDEX） 
  - 可以通过 `ALTER TABLE table_name ADD INDEX index_name (column);` 创建普通索引 
  - 可以通过 `ALTER TABLE table_name ADD INDEX index_name(column1, column2, column3);` 创建组合索引

- 全文索引（FULLTEXT） 

  可以通过 `ALTER TABLE table_name ADD FULLTEXT (column);` 创建全文索引 



 **索引并非是越多越好，创建索引也需要耗费资源，一是增加了数据库的存储空间，二是在插入和删除时 要花费较多的时间维护索引**



- 索引加快数据库的检索速度 
- 索引降低了插入、删除、修改等维护任务的速度 
- 唯一索引可以确保每一行数据的唯一性 
- 通过使用索引，可以在查询的过程中使用优化隐藏器，提高系统的性能 
- 索引需要占物理和数据空间 







## 6、SQL优化 

1. 查询语句中不要使用select  * 
2. 尽量减少子查询，使用关联查询（left join,right join,inner  join）替代
3. 减少使用IN或者NOT IN ,使用exists，not exists或者关联查询语句替代 
4. or 的查询尽量用 union或者union all 代替(在确认没有重复数据或者不用剔除重复数据时，union all会更好) 
5. 应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎==放弃使用索引==而进行全表扫描。
6. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎==放弃使用索引==而进行全表扫 描，如： select id from t where num is null 可以在num上设置默认值0，确保表中num列没有null 值，然后这样查询： select id from t where num=0 
7. 除此之外还有**不要在索引列上进行运算或使用函数**（id+1=5）、**前导模糊查询不会使用索引**（%李）、**小心隐式类型转换**都会导致==放弃使用索引==





## 7、drop、delete与truncate的区别 

- SQL中的drop、delete、truncate都表示删除，但是三者有一些差别 

- delete和truncate只删除表的数据不删除表的结构 

- 速度,一般来说: drop> truncate >delete 

- delete语句是dml,这个操作会放到rollback segement中,事务提交之后才生效; 

- 如果有相应的trigger，执行的时候将被触发. truncate,drop是ddl, 操作立即生效,原数据不放到rollback segment中，不能回滚。操作不触发trigger. 







## 8、什么是视图 ？

视图是一种虚拟的表，具有和物理表相同的功能。可以对视图进行增，改，查，操作，试图通常是有一 个表或者多个表的行或列的子集。对视图的修改不影响基本表。它使得我们获取数据更容易，相比多表查询。

虚拟的表和普通的表一样，通过普通的表动生成的数据，只保存sql逻辑，不保存查询结果

create view 别名 as

sql语句 
优点：重用了sql 简化了复制的sql语句 保护数据的安全性

- 视图的修改

  create or replace view 别名 as 

- 删除视图

  drop view 视图名1，视图名2

- 查看视图

  desc 视图名 show create view 视图

  

视图的更行：更改视图中的数据
增加、修改、删除、
具备一下特点的视图不能更新：分组函数、distinct、group by 、having、union、union all
常量视图（只有一个结果）
视图和表的区别
创建语法不一样
视图只是保存了sql逻辑空间 表实际占用物理空间

**视图一般用于查  表可以增删改查**



## 9、 什么是内联接、左外联接、右外联接？

- 内联接（Inner Join）：匹配2张表中相关联的记录。 
- 左外联接（Left Outer Join）：除了匹配2张表中相关联的记录外，还会匹配左表中剩余的记录， 右表中未匹配到的字段用NULL表示。 
- 右外联接（Right Outer Join）：除了匹配2张表中相关联的记录外，还会匹配右表中剩余的记录， 左表中未匹配到的字段用NULL表示。在判定左表和右表时，要根据表名出现在Outer Join的左右 位置关系





## 10、并发事务带来哪些问题？

在典型的应用程序中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个用户对同一 数据进行操作）。并发虽然是必须的，但可能会导致以下的问题。

- **脏读（Dirty read）**: 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到 数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提 交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确 的。 
- **丢失修改（Lost to modify）**: 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 例如：事务1读取某表中的数据A=20，事务2也读取A=20，事 务1修改A=A-1，事务2也修改A=A-1，终结果A=19，事务1的修改被丢失。 
- **不可重复读（Unrepeatableread）**: 指在一个事务内多次读同一数据。在这个事务还没有结束 时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改 导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样 的情况，因此称为不可重复读。 
- **幻读（Phantom read）**: 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接 着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了 一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。



**不可重复读和幻读区别：**

不可重复读的重点是修改比如多次读取一条记录发现其中某些列的值被修改，幻读的重点在于新增或者 删除比如多次读取一条记录发现记录增多或减少了。





## 11、事务隔离级别有哪些? MySQL的默认隔离级别是?

**SQL 标准定义了四个隔离级别：** 

- **READ-UNCOMMITTED(读取未提交)：** 低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读。** 
- **READ-COMMITTED(读取已提交)：** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生。** 
- **REPEATABLE-READ(可重复读)：** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。** 
- **SERIALIZABLE(可串行化)：** 高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执 行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读。**



| 隔离级别         | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| READ-UNCOMMITTED | √    | √          | √    |
| READ-COMMITTED   | ×    | √          | √    |
| REPEATABLE-READ  | ×    | ×          | √    |
| SERIALIZABLE     | ×    | ×          | ×    |



MySQL InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）**。我们可以通过 `SELECT @@tx_isolation;` 命令来查看 



这里需要注意的是：与 SQL 标准不同的地方在于 InnoDB 存储引擎在 **REPEATABLE-READ（可重读）** 事务隔离级别下使用的是Next-Key Lock 锁算法，因此可以避免幻读的产生，这与其他数据库系统(如 SQL Server) 是不同的。所以说InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）** 已经可以完全保证事务的隔离性要求，即达到了 SQL标准的 **SERIALIZABLE(可串行化)** 隔离级 别。因为隔离级别越低，事务请求的锁越少，所以大部分数据库系统的隔离级别都是 **READCOMMITTED(读取提交内容)** ，但是你要知道的是InnoDB 存储引擎默认使用 **REPEATABLE-READ（可重读）** 并不会有任何性能损失。 

InnoDB 存储引擎在分布式事务 的情况下一般会用到 **SERIALIZABLE(可串行化)** 隔离级别。







## 12、大表如何优化？

当MySQL单表记录数过大时，数据库的CRUD性能会明显下降，一些常见的优化措施如下： 

**1.限定数据的范围** 

**禁止**不带任何限制数据范围条件的查询语句。比如：我们当用户在查询订单历史的时候，我们可以控制在一个月的范围内； 

**2. 读/写分离** 

经典的数据库拆分方案，主库负责写，从库负责读； 

**3. 垂直分区** 

**根据数据库里面数据表的相关性进行拆分。**  例如，用户表中既有用户的登录信息又有用户的基本信息， 可以将用户表拆分成两个单独的表，甚至放到单独的库做分库。

**简单来说垂直拆分是指数据表列的拆分，把一张列比较多的表拆分为多张表。** 



- **垂直拆分的优点：** 可以使得列数据变小，在查询时减少读取的 Block 数，减少 I/O次 数。此外，垂 直分区可以简化表的结构，易于维护。 
- **垂直拆分的缺点：** 主键会出现冗余，需要管理冗余列，并会引起 Join 操作，可以通过在应用层进行 Join 来解决。此外，垂直分区会让事务变得更加复杂； 



**4. 水平分区** 

**保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了 分布式的目的。 水平拆分可以支撑非常大的数据量。**水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成 多张表来存放。

水平拆分可以支持非常大的数据量。需要注意的一点是：分表仅仅是解决了单一表数据过大的问题，但由于表的数据还是在同一台机器上，其实对于提升MySQL并发能力没有什么意义，所以**水平拆分最好分库 。**

水平拆分能够 **支持非常大的数据量存储，应用端改造也少，但分片事务难以解决** ，跨节点Join性能较差，逻辑复杂。《Java工程师修炼之道》的作者推荐**尽量不要对数据进行分片，因为拆分会带来逻辑、 部署、运维的各种复杂度** ，一般的数据表在优化得当的情况下支撑千万以下的数据量是没有太大问题 的。如果实在要分片，尽量选择客户端分片架构，这样可以减少一次和中间件的网络I/O。

**下面补充一下数据库分片的两种常见方案：**

- **客户端代理： 分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。** 当当网的 Sharding-JDBC 、阿里的TDDL是两种比较常用的实现。 
- **中间件代理： 在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。** 我们现在 谈的 Mycat 、360的Atlas、网易的DDB等等都是这种架构的实现。



## 13、分库分表之后,id 主键如何处理？

因为要是分成多个表之后，每个表都是从 1 开始累加，这样是不对的，我们需要一个全局唯一的 id 来 支持。

生成全局 id 有下面这几种方式：

- **UUID：**不适合作为主键，因为太长了，并且无序不可读，查询效率低。比较适合用于生成唯一的名字的标示比如文件的名字。
- **数据库自增 id :**  两台数据库分别设置不同步长，生成不重复ID的策略来实现高可用。这种方式生成 的 id 有序，但是需要独立部署数据库实例，成本高，还会有性能瓶颈。 
- **利用 redis 生成 id :** 性能比较好，灵活方便，不依赖于数据库。但是，引入了新的组件造成系统更 加复杂，可用性降低，编码更加复杂，增加了系统成本。

- **Twitter的snowﬂake算法 ：**Github 地址：https://github.com/twitter-archive/snowﬂake。 
- **美团的Leaf分布式ID生成系统 ：**Leaf 是美团开源的分布式ID生成器，能保证全局唯一性、趋势递 增、单调递增、信息安全，里面也提到了几种分布式方案的对比，但也需要依赖关系数据库、 Zookeeper等中间件。感觉还不错。美团技术团队的一篇文章：https://tech.meituan.com/2017/ 04/21/mt-leaf.html 。 



## 14、mysql有关权限的表都有哪几个 

MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库里，由 mysql_install_db脚本初始化。这些权限表分别user，db，table_priv，columns_priv和host。下面分 别介绍一下这些表的结构和内容：

- user权限表：记录允许连接到服务器的用户帐号信息，里面的权限是全局级的。 
- db权限表：记录各个帐号在各个数据库上的操作权限。
- table_priv权限表：记录数据表级的操作权限。 
- columns_priv权限表：记录数据列级的操作权限。 
- host权限表：配合db权限表对给定主机上数据库级操作权限作更细致的控制。这个权限表不受 GRANT和REVOKE语句的影响。 





## 15、mysql有哪些数据类型 

**1、整数类型**，包括TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示1字节、2字节、3 字节、4字节、8字节整数。任何整数类型都可以加上UNSIGNED属性，表示数据是无符号的，即非负整数。 

长度：整数类型可以被指定长度，例如：INT(11) 表示长度为 11 的 INT 类型。长度在大多数场景是没有意 义的，它不会限制值的合法范围，只会影响显示字符的个数，而且需要和UNSIGNED ZEROFILL属性配 合使用才有意义。 

例子，假定类型设定为 INT(5)，属性为 UNSIGNED ZEROFILL，如果用户插入的数据为 12 的话，那么数 据库实际存储数据为 00012。

**2、实数类型**，包括FLOAT、DOUBLE、DECIMAL。 

DECIMAL可以用于存储比BIGINT还大的整型，能存储精确的小数。 
而 FLOAT 和 DOUBLE 是有取值范围的，并支持使用标准的浮点进行近似计算。 
计算时 FLOAT 和 DOUBLE 相比 DECIMAL 效率更高一些，DECIMAL 你可以理解成是用字符串进行处理。 

**3、字符串类型**，包括VARCHAR、CHAR、TEXT、BLOB 

VARCHAR用于存储可变长字符串，它比定长类型更节省空间。 
VARCHAR使用额外1或2个字节存储字符串长度。列长度小于255字节时，使用1字节表示，否则使用2 字节表示。 
VARCHAR存储的内容超出设置的长度时，内容会被截断。 
CHAR是定长的，根据定义的字符串长度分配足够的空间。 
CHAR会根据需要使用空格进行填充方便比较。 
CHAR适合存储很短的字符串，或者所有值都接近同一个长度。 
CHAR存储的内容超出设置的长度时，内容同样会被截断。

**使用策略：**
对于经常变更的数据来说，CHAR比VARCHAR更好，因为CHAR不容易产生碎片。 
对于非常短的列，CHAR比VARCHAR在存储空间上更有效率。 
使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。 
尽量避免使用TEXT/BLOB类型，查询时会使用临时表，导致严重的性能开销。 

**4、枚举类型（ENUM）**，把不重复的数据存储为一个预定义的集合。 

有时可以使用ENUM代替常用的字符串类型。 
ENUM存储非常紧凑，会把列表值压缩到一个或两个字节。 
ENUM在内部存储时，其实存的是整数。 
尽量避免使用数字作为ENUM枚举的常量，因为容易混乱。 
排序是按照内部存储的整数 

**5、日期和时间类型**，尽量使用timestamp，空间效率高于datetime， 

用整数保存时间戳通常不方便处理。
如果需要存储微妙，可以使用bigint存储。 
看到这里，这道真题是不是就比较容易回答了。



## 16、创建索引的三种方式，删除索引 

第一种方式：在执行CREATE TABLE时创建索引



第二种方式：使用ALTER TABLE命令去增加索引 



第三种方式：使用CREATE INDEX命令创建



删除索引
根据索引名删除普通索引、唯一索引、全文索引： `alter table 表名 drop KEY 索引名`



删除主键索引： `alter table 表名 drop primary key `（因为主键只有一个）。这里值得注意的是， 如果主键自增长，那么不能直接执行此操作（自增长依赖于主键索引）： 

```sql
alter table t_user drop primary key
```

 需要取消自增长再行删除：  

```sql
alter table t_user
-- 重新定义字段 
MODIFY id int, 
drop PRIMARY KEY
```

 但通常不会删除主键，因为设计主键一定与业务逻辑无关。 







# Redis

## 1、Redis持久化机制 

Redis是一个支持持久化的内存数据库，通过持久化机制把内存中的数据同步到硬盘文件来保证数据持 久化。当Redis重启后通过把硬盘文件重新加载到内存，就能达到恢复数据的目的。 实现：单独创建fork()一个子进程，将当前父进程的数据库数据复制到子进程的内存中，然后由子进程 写入到临时文件中，持久化的过程结束了，再用这个临时文件替换上次的快照文件，然后子进程退出， 内存释放。

**RDB：**Redis默认的持久化方式。按照一定的时间周期策略把内存的数据以快照的形式保存到硬盘的二 进制文件。即Snapshot快照存储，对应产生的数据文件为dump.rdb，通过配置文件中的save参数来定 义快照的周期。（ 快照可以是其所表示的数据的一个副本，也可以是数据的一个复制品。） 
**AOF：**Redis会将每一个收到的写命令都通过Write函数追加到文件后，类似于MySQL的binlog。当 Redis重启是会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。 

当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复。



## 2、缓存雪崩、缓存穿透、缓存预热、缓存更新、缓存降级等问题 

**1、缓存雪崩**

我们可以简单的理解为：由于原有缓存失效，新缓存未到期间 (例如：我们设置缓存时采用了相同的过期时间，在同一时刻出现大面积的缓存过期)，所有原本应该访 问缓存的请求都去查询数据库了，而对数据库CPU和内存造成巨大压力，严重的会造成数据库宕机。从 而形成一系列连锁反应，造成整个系统崩溃。 

**解决办法：** 大多数系统设计者考虑用加锁（ 多的解决方案）或者队列的方式保证来保证不会有大量的线程对数据 库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。还有一个简单方案就时讲缓 存失效时间分散开



**2、缓存穿透** 

缓存穿透是指用户查询数据，在数据库没有，自然在缓存中也不会有。这样就导致用户查询的时候，在 缓存中找不到，每次都要去数据库再查询一遍，然后返回空（相当于进行了两次无用的查询）。这样请 求就绕过缓存直接查数据库，这也是经常提的缓存命中率问题。

**解决办法：**

- 常见的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存 在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。 

- 另外也有一个更为简单粗暴的方法，如果一个查询返回的数据为空（不管是数据不存在，还是系统故 障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，长不超过五分钟。通过这个直接设 置的默认值存放到缓存，这样第二次到缓冲中获取就有值了，而不会继续访问数据库，这种办法简单粗暴。 

5TB的硬盘上放满了数据，请写一个算法将这些数据进行排重。如果这些数据是一些32bit大小的数据该如何解决？如果是64bit的呢？

对于空间的利用到达了一种极致，那就是Bitmap和布隆过滤器(Bloom Filter)。 
Bitmap： 典型的就是哈希表 
缺点是，Bitmap对于每个元素只能记录1bit信息，如果还想完成额外的功能，恐怕只能靠牺牲更多的空 间、时间来完成了。



**布隆过滤器（推荐）** 

就是引入了k(k>1)k(k>1)个相互独立的哈希函数，保证在给定的空间、误判率下，完成元素判重的过程。 

它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。 

Bloom-Filter算法的核心思想就是利用多个不同的Hash函数来解决“冲突”。 

Hash存在一个冲突（碰撞）的问题，用同一个Hash得到的两个URL的值有可能相同。为了减少冲突， 我们可以多引入几个Hash，如果通过其中的一个Hash值我们得出某元素不在集合中，那么该元素肯定 不在集合中。只有在所有的Hash函数告诉我们该元素在集合中时，才能确定该元素存在于集合中。这便是Bloom-Filter的基本思想。 

Bloom-Filter一般用于在大数据量的集合中判定某元素是否存在



**3、缓存预热** 

缓存预热这个应该是一个比较常见的概念，相信很多小伙伴都应该可以很容易的理解，缓存预热就是系 统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据 库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！ 

解决思路： 

- 直接写个缓存刷新页面，上线时手工操作下
- 数据量不大，可以在项目启动的时候自动进行加载
- 定时刷新缓存



**4、缓存更新** 

除了缓存服务器自带的缓存失效策略之外（Redis默认的有6中策略可供选择），我们还可以根据具体的业务需求进行自定义的缓存淘汰，常见的策略有两种： 

- 定时去清理过期的缓存；

- 当有用户请求过来时，再判断这个请求所用到的缓存是否过期，过期的话就去底层系统得到新数 据并更新缓存。

两者各有优劣，第一种的缺点是维护大量缓存的key是比较麻烦的，第二种的缺点就是每次用户请求过 来都要判断缓存失效，逻辑相对比较复杂！具体用哪种方案，大家可以根据自己的应用场景来权衡。







**5、缓存降级** 

当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

降级的终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

以参考日志级别设置预案： 

- 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级
- 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级， 并发送告警； 

- 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的 大阀值，此时可以根据情况自动降级或者人工降级
- 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。



服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要 的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。



## 3、热点数据和冷数据是什么 

**热点数据，缓存才有价值** 

对于冷数据而言，大部分数据可能还没有再次访问到就已经被挤出内存，不仅占用内存，而且价值不大。频繁修改的数据，看情况考虑使用缓存 

对于上面两个例子，寿星列表、导航信息都存在一个特点，就是信息修改频率不高，读取通常非常高的场景。 

对于热点数据，比如我们的某IM产品，生日祝福模块，当天的寿星列表，缓存以后可能读取数十万次。 再举个例子，某导航产品，我们将导航信息，缓存以后可能读取数百万次



**数据更新前至少读取两次**，缓存才有意义。这个是基本的策略，如果缓存还没有起作用就失效了，那 就没有太大价值了。 

那存不存在，修改频率很高，但是又不得不考虑缓存的场景呢？有！比如，这个读取接口对数据库的压 力很大，但是又是热点数据，这个时候就需要考虑通过缓存手段，减少数据库的压力，比如我们的某助 手产品的，点赞数，收藏数，分享数等是非常典型的热点数据，但是又不断变化，此时就需要将数据同 步保存到Redis缓存，减少数据库压力。 



## 4、Memcache与Redis的区别都有哪些？ 

1. 存储方式 Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。 Redis 有部份存在硬盘上，redis可以持久化其数据
2. 数据支持类型 memcached所有的值均是简单的字符串，redis作为其替代者，支持更为丰富的数据 类型 ，提供list，set，zset，hash等数据结构的存储
3. 使用底层模型不同 它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。 Redis直接自 己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
4. value 值大小不同：Redis 大可以达到 1gb；memcache 只有 1mb。
5. redis的速度比memcached快很多 
6. Redis支持数据的备份，即master-slave模式的数据备份。



## 5、单线程的redis为什么这么快 

1. 纯内存操作
2. 单线程操作，避免了频繁的上下文切换
3. 采用了非阻塞I/O多路复用机制







