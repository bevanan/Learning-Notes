# 第五章 结构型模式(7种)

我们已经学习过了设计模式中的创建型模式. 创建型模式主要解决**对象的创建问题**,封装复杂的创建过程,解耦对象的创建代码和使用代码.

- 单例模式用来创建全局唯一对象
- 工厂模式用来创建不同但是相关类型的对象(继承同一父类或者接口的一组子类),由给定的参数来决定创建哪种类型的对象.
- 建造者模式是用来创建复杂对象,可以通过设置不同的可选参数,定制化地创建不同的对象.
- 原型模式针对创建成本比较大的对象,利用对已有对象进行复制的方式进行创建,以达到节省创建时间的目的.

从本节课开始我们来学习结构型设计模式, 结构型模式主要总结了一些类和对象组合在一起的经典结构,这些经典结构可以解决对应特定场景的问题.

一共包括七种：代理模式、桥接模式、装饰者模式、适配器模式、门面(外观)模式、组合模式、和享元模式。



## 5.1 代理模式

### 5.1.1 代理模式介绍

在软件开发中,由于一些原因,客户端不想或不能直接访问一个对象,此时可以通过一个称为"代理"的第三者来实现间接访问.该方案对应的设计模式被称为代理模式.

代理模式(Proxy Design Pattern ) 原始定义是：让你能够提供对象的替代品或其占位符。代理控制着对于原对象的访问，并允许将请求提交给对象前后进行一些处理。

- 现实生活中的代理: **海外代购**

​		<img src=".\img\65.jpg" alt="image-20220530160637842" style="zoom: 50%;" />

- 软件开发中的代理

  代理模式中引入了一个新的代理对象,代理对象在客户端对象和目标对象之间起到了中介的作用,它去掉客户不能看到的内容和服务或者增加客户需要的额外的新服务.

   

### 5.1.2 代理模式原理

代理（Proxy）模式分为三种角色：

* 抽象主题（Subject）类： 声明了真实主题和代理主题的共同接口,这样就可以保证任何使用真实主题的地方都可以使用代理主题,客户端一般针对抽象主题类进行编程。
* 代理（Proxy）类 ： 提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以在任何时候访问、控制或扩展真实主题的功能。
* 真实主题（Real Subject）类： 实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。

<img src=".\img\66.jpg" alt="image-20220530160637842" style="zoom: 80%;" /> 

###  5.1.3 静态代理实现

这种代理方式需要代理对象和目标对象实现一样的接口。

- 优点：可以在不修改目标对象的前提下扩展目标对象的功能。

- 缺点：

  1. 冗余。由于代理对象要实现与目标对象一致的接口，会产生过多的代理类。

  2. 不易维护。一旦接口增加方法，目标对象与代理对象都要进行修改。

> 举例：保存用户功能的静态代理实现

```java
//接口类： IUserDao
public interface IUserDao {
    void save();
}

//目标对象：UserDaoImpl
public class UserDaoImpl implements IUserDao {
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}

//静态代理对象：UserDaoProxy 需要实现IUserDao接口
public class UserDaoProxy implements IUserDao {
    private IUserDao target;

    public UserDaoProxy(IUserDao target) {
        this.target = target;
    }

    @Override
    public void save() {
        System.out.println("开启事务"); //扩展额外功能
        target.save();
        System.out.println("提交事务");
    }
}

//测试类
public class TestProxy {
    @Test
    public void testStaticProxy()
        //目标对象
        UserDaoImpl userDao = new UserDaoImpl();
        //代理对象
        UserDaoProxy proxy = new UserDaoProxy(userDao);
        proxy.save();
    }
}
```

### 5.1.4 JDK动态代理

#### 5.1.4.1 JDK动态代理实现

动态代理利用了JDK API,动态地在内存中构建代理对象,从而实现对目标对象的代理功能.动态代理又被称为JDK代理或接口代理.

静态代理与动态代理的`区别`:

1. `静态代理`在编译时就已经实现了,编译完成后代理类是一个实际的class文件
2. `动态代理`是在运行时动态生成的,即编译完成后没有实际的class文件,而是在运行时动态生成类字节码,并加载到JVM中.



JDK中生成代理对象主要涉及的类有

- java.lang.reflect Proxy，主要方法为

  ```java
  static Object newProxyInstance(
      ClassLoader loader,  		//指定当前目标对象使用类加载器
      Class<?>[] interfaces,      //目标对象实现的接口的类型
      InvocationHandler h         //事件处理器
  ) 
  //返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
  ```

- java.lang.reflect InvocationHandler，主要方法为

  ```java
  Object invoke(Object proxy, Method method, Object[] args) 
  // 在代理实例上处理方法调用并返回结果。
  ```

> 举例：保存用户功能的静态代理实现

```java
/**
 * 代理工厂-动态生成代理对象
 **/
public class ProxyFactory {
    private Object target; //维护一个目标对象

    public ProxyFactory(Object target) {
        this.target = target;
    }

    //为目标对象生成代理对象
    public Object getProxyInstance(){
        //使用Proxy获取代理对象
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(), //目标类使用的类加载器
                target.getClass().getInterfaces(),  //目标对象实现的接口类型
                new InvocationHandler(){            //事件处理器
                    /**
                     * invoke方法参数说明
                     * @param proxy 代理对象
                     * @param method 对应于在代理对象上调用的接口方法Method实例
                     * @param args 代理对象调用接口方法时传递的实际参数
                     * @return: java.lang.Object 返回目标对象方法的返回值,没有返回值就返回null
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开启事务");
                        //执行目标对象方法
                        method.invoke(target, args);
                        System.out.println("提交事务");
                        return null;
                    }
                }
        );
    }
}

//测试
public static void main(String[] args) {
    IUserDao target = new UserDaoImpl();
    System.out.println(target.getClass());//目标对象信息

    IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
    System.out.println(proxy.getClass()); //输出代理对象信息
    proxy.save(); //执行代理方法
}
```

#### 5.1.4.2 类是如何动态生成的

Java虚拟机类加载过程主要分为五个阶段：加载、验证、准备、解析、初始化。其中加载阶段需要完成以下3件事情：

1. 通过一个类的全限定名来获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. 在内存中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区这个类的各种数据访问入口

由于虚拟机规范对这3点要求并不具体，所以实际的实现是非常灵活的，关于第1点，**获取类的二进制字节流**（class字节码）就有很多途径：

<img src=".\img\69.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

- 从本地获取

- 从网络中获取

- **运行时计算生成**，这种场景使用最多的是动态代理技术，在 java.lang.reflect.Proxy 类中，就是用了 ProxyGenerator.generateProxyClass 来为特定接口生成形式为 `*$Proxy` 的代理类的二进制字节流

  <img src=".\img\70.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

  

所以，动态代理就是想办法，根据接口或目标对象，计算出代理类的字节码，然后再加载到JVM中使用

#### 5.1.4.3 代理类的调用过程

我们通过借用阿里巴巴的一款线上监控诊断产品 Arthas(阿尔萨斯) ,对动态生成的代理类代码进行查看

<img src=".\img\67.jpg" alt="image-20220530160637842" style="zoom: 80%;" />

<img src=".\img\68.jpg" alt="image-20220530160637842" style="zoom: 80%;" /> 



代理类代码如下:

```java
package com.sun.proxy;

import com.mashibing.proxy.example01.IUserDao;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0
extends Proxy
implements IUserDao {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler invocationHandler) {
        super(invocationHandler);
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.mashibing.proxy.example01.IUserDao").getMethod("save", new Class[0]);
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            return;
        }
        catch (NoSuchMethodException noSuchMethodException) {
            throw new NoSuchMethodError(noSuchMethodException.getMessage());
        }
        catch (ClassNotFoundException classNotFoundException) {
            throw new NoClassDefFoundError(classNotFoundException.getMessage());
        }
    }

    public final boolean equals(Object object) {
        try {
            return (Boolean)this.h.invoke(this, m1, new Object[]{object});
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final String toString() {
        try {
            return (String)this.h.invoke(this, m2, null);
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final int hashCode() {
        try {
            return (Integer)this.h.invoke(this, m0, null);
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final void save() {
        try {
            this.h.invoke(this, m3, null);
            return;
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }
}
```

简化后的代码

```java
package com.sun.proxy;

import com.mashibing.proxy.example01.IUserDao;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0
extends Proxy
implements IUserDao {
    private static Method m3;

    public $Proxy0(InvocationHandler invocationHandler) {
        super(invocationHandler);
    }

    static {
        try {
         m3 = Class.forName("com.mashibing.proxy.example01.IUserDao").getMethod("save", new Class[0]);
          return;
        }
    }

    public final void save() {
        try {
            this.h.invoke(this, m3, null);
            return;
        }
    }
}
```

- 动态代理类对象 继承了 Proxy 类，并且实现了被代理的所有接口，以及equals、hashCode、toString等方法

- 代理类的构造函数，参数是`InvocationHandler`实例，`Proxy.newInstance`方法就是通过这个构造函数来创建代理实例的

- 类和所有方法都被 `public final` 修饰，所以代理类只可被使用，不可以再被继承

- 每个方法都有一个 Method 对象来描述，Method 对象在static静态代码块中创建，以 `m + 数字` 的格式命名

- 调用方法的时候通过 `this.h.invoke(this, m3, null));`  **实际上 h.invoke就是在调用ProxyFactory中我们重写的invoke方法**  

  ```java
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      System.out.println("开启事务");
      //执行目标对象方法
      method.invoke(target, args);
      System.out.println("提交事务");
      return null;
  }
  ```

### 5.1.5 cglib动态代理

#### 5.1.5.1 cglib动态代理实现

cglib (Code Generation Library ) 是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。cglib 为没有实现接口的类提供代理，为JDK的动态代理提供了很好的补充。

<img src=".\img\72.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

- 最底层是字节码
- ASM是操作字节码的工具
- cglib基于ASM字节码工具操作字节码（即动态生成代理，对方法进行增强）
- SpringAOP基于cglib进行封装，实现cglib方式的动态代理

使用cglib 需要引入cglib 的jar包，如果你已经有spring-core的jar包，则无需引入，因为spring中包含了cglib 。

- cglib 的Maven坐标

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.5</version>
</dependency>
```

**示例代码**

目标类

```java
public class UserServiceImpl {
    public List<User> findUserList(){
        return Collections.singletonList(new User("tom",18));
    }
}

public class User {
    private String name;
    private int age;
    ......
}
```

cglib代理类，需要实现MethodInterceptor接口，并指定代理目标类target

```java
public class UserLogProxy implements MethodInterceptor {
    private Object target;
    
    public UserLogProxy(Object target) {
        this.target = target;
    }

    public Object getLogProxy(){
        //增强器类,用来创建动态代理类
        Enhancer en = new Enhancer();
        //设置代理类的父类字节码对象
        en.setSuperclass(target.getClass());
        //设置回调: 对于代理类上所有的方法的调用,都会调用CallBack,而Callback则需要实现intercept()方法进行拦截
        en.setCallback(this);
        //创建动态代理对象并返回
        return en.create();
    }

    /**
     * 实现回调方法
     * @param o         代理对象
     * @param method    目标对象中的方法的Method实例
     * @param args      实际参数
     * @param methodProxy  代理对象中的方法的method实例
     * @return: java.lang.Object
     */
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Calendar calendar = Calendar.getInstance();
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(formatter.format(calendar.getTime()) + " [" +method.getName() + "] 查询用户信息...]");

        Object result = methodProxy.invokeSuper(o, args);
        return result;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        //目标对象
        UserServiceImpl userService = new UserServiceImpl();
        System.out.println(userService.getClass());

        //代理对象
        UserServiceImpl proxy = (UserServiceImpl) new UserLogProxy(userService).getLogProxy();
        System.out.println(proxy.getClass());

        List<User> userList = proxy.findUserList();
        System.out.println("用户信息: "+userList);
    }
}
```

#### 5.1.5.2 cglib代理流程

<img src=".\img\74.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

### 5.1.6 代理模式总结

#### 5.1.6.1 三种代理模式实现方式的对比

* jdk代理和CGLIB代理

  使用CGLib实现动态代理，**CGLib底层采用ASM字节码生成框架**，使用字节码技术生成代理类，在JDK1.6之前比使用Java反射效率要高。唯一需要注意的是，CGLib不能对声明为final的类或者方法进行代理，因为CGLib原理是动态生成被代理类的子类。

  在JDK1.6、JDK1.7、JDK1.8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLib代理效率，只有当进行大量调用的时候，JDK1.6和JDK1.7比CGLib代理效率低一点，但是到JDK1.8的时候，JDK代理效率高于CGLib代理。所以如果有接口使用JDK动态代理，如果没有接口使用CGLIB代理。

* 动态代理和静态代理

  动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。

  如果接口增加一个方法，静态代理模式除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。而动态代理不会出现该问题

#### 5.1.6.2 代理模式优缺点

**优点：**

- 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；
- 代理对象可以扩展目标对象的功能；
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度；

**缺点：**

* 增加了系统的复杂度；

#### 5.1.6.2 代理模式使用场景 

* 功能增强

  当需要对一个对象的访问提供一些额外操作时,可以使用代理模式

* 远程（Remote）代理

  实际上，RPC 框架也可以看作一种代理模式，GoF 的《设计模式》一书中把它称作远程代理。通过远程代理，将网络通信、数据编解码等细节隐藏起来。客户端在使用 RPC 服务的时候，就像使用本地函数一样，无需了解跟服务器交互的细节。除此之外，RPC 服务的开发者也只需要开发业务逻辑，就像开发本地使用的函数一样，不需要关注跟客户端的交互细节。

* 防火墙（Firewall）代理

  当你将浏览器配置成使用代理功能时，防火墙就将你的浏览器的请求转给互联网；当互联网返回响应时，代理服务器再把它转给你的浏览器。

* 保护（Protect or Access）代理

  控制对一个对象的访问，如果需要，可以给不同的用户提供不同级别的使用权限。

  

## 5.2 桥接模式

### 5.2.1 桥接模式介绍

桥接模式(bridge pattern) 的定义是：将抽象部分与它的实现部分分离，使它们都可以独立地变化。

> 桥接模式用一种巧妙的方式处理多层继承存在的问题,用抽象关联来取代传统的多层继承,将类之间的静态继承关系转变为动态的组合关系,使得系统更加灵活,并易于扩展,有效的控制了系统中类的个数 (避免了继承层次的指数级爆炸).

### 5.2.2 桥接模式原理

<img src=".\img\75.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

桥接（Bridge）模式包含以下主要角色：

* 抽象化（Abstraction）角色 ：主要负责定义出该角色的行为 ,并包含一个对实现化对象的引用。
* 扩展抽象化（RefinedAbstraction）角色 ：是抽象化角色的子类，实现父类中的业务方法，并通过组合关系调用实现化角色中的业务方法。
* 实现化（Implementor）角色 ：定义实现化角色的接口，包含角色必须的行为和属性,并供扩展抽象化角色调用。
* 具体实现化（Concrete Implementor）角色 ：给出实现化角色接口的具体实现。

**桥接模式原理的核心是: 首先有要识别出一个类所具有的的两个独立变化维度,将它们设计为两个独立的继承等级结构,为两个维度都提供抽象层,并建立抽象耦合.总结一句话就是: 抽象角色引用实现角色** 

比如我们拿毛笔举例, 型号和颜色是毛笔的两个维度

- 型号是其固有的维度,所以抽象出一个毛笔类,而将各种型号的毛笔作为其子类,也就是下图的左侧抽象部分内容.
- 颜色是毛笔的另一个维度, 它与毛笔之间存在一种设置的关系,因此可以提供一个抽象的颜色接口,将具体颜色作为该接口的子类.

<img src=".\img\76.jpg" alt="image-20220530160637842" style="zoom: 50%;" />



### 5.2.3 桥接模式的应用实例

模拟不同的支付工具对应不同的支付模式,比如微信和支付宝都可以完成支付操作,而支付操作又可以有扫码支付、密码支付、人脸支付等,那么关于支付操作其实就有两个维度, 包括:**支付渠道和支付方式** 

<img src=".\img\78.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  

**1) 不使用设计模式**

```java
public class PayController {
    /**
     * @param uId   		用户id
     * @param tradeId 		交易流水号
     * @param amount    	交易金额
     * @param channelType 	渠道类型 1 微信, 2 支付宝
     * @param modeType    	支付模式 1 密码, 2 人脸, 3 指纹
     */
    public boolean doPay(String uId, String tradeId, BigDecimal amount，int channelType，int modeType){
        //微信支付
        if(1 == channelType){
            System.out.println("微信渠道支付划账开始......");
            if(1 == modeType){
                System.out.println("密码支付");
            }if(2 == modeType){
                System.out.println("人脸支付");
            }if(3 == modeType){
                System.out.println("指纹支付");
            }
        }

        //支付宝支付
        if(2 == channelType){
            System.out.println("支付宝渠道支付划账开始......");
            if(1 == modeType){
                System.out.println("密码支付");
            }if(2 == modeType){
                System.out.println("人脸支付");
            }if(3 == modeType){
                System.out.println("指纹支付");
            }
        }
        return true;
    }
}

//测试
public class Test_Pay {
    public static void main(String[] args) {
        PayController payController = new PayController();
        System.out.println("测试: 微信支付、人脸支付方式");
        payController.doPay("weixin_001","1000112333333",new BigDecimal(100),1，2);

        System.out.println("\n测试: 支付宝支付、指纹支付方式");
        payController.doPay("hifubao_002","1000112334567",new BigDecimal(100),2，3);
    }
}
```

从测试结果看,是满足了需求,但是这样的代码设计,后面的维护和扩展都会变得非常复杂.

**1) 桥接模式重构代码**

重构类图

**桥接模式原理的核心是: 首先有要识别出一个类所具有的的两个独立变化维度,将它们设计为两个独立的继承等级结构,为两个维度都提供抽象层,并建立抽象耦合.**

- Pay抽象类
  - 支付渠道子类: 微信支付
  - 支付渠道子类: 支付宝支付
- IPayMode接口
  - 支付模式实现: 刷脸支付
  - 支付模式实现: 指纹支付
- 支付渠道*支付模式 = 相对应的组合.

<img src=".\img\80.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

**1) 支付模式接口 (实现化角色)**

```java
/**
 * 支付模式接口
 **/
public interface IPayMode {
    //安全校验功能: 对各种支付模式进行风控校验
    boolean security(String uId);
}
```

**2) 三种具体支付模式 (具体实现化角色)**

```java
//指纹支付及风控校验
public class PayFingerprintMode implements IPayMode {
    @Override
    public boolean security(String uId) {
        System.out.println("指纹支付,风控校验-指纹信息");
        return true;
    }
}

//刷脸支付及风控校验
public class PayFaceMode implements IPayMode {
    @Override
    public boolean security(String uId) {
        System.out.println("人脸支付,风控校验-脸部识别");
        return true;
    }
}

//密码支付及风控校验
public class PayCypher implements IPayMode {
    @Override
    public boolean security(String uId) {
        System.out.println("密码支付,风控校验-环境安全");
        return false;
    }
}
```

**3) 支付抽象类 (抽象化角色)** 

```JAVA
public abstract class Pay {
    //桥接对象
    protected IPayMode payMode;

    public Pay(IPayMode payMode) {
        this.payMode = payMode;
    }

    //划账功能
    public abstract String transfer(String uId, String tradeId, BigDecimal amount);
}
```

**4) 支付渠道实现 (扩展抽象化角色)**

```java
/**
 * 支付渠道-微信划账
 **/
public class WxPay extends Pay {
    public WxPay(IPayMode payMode) {
        super(payMode);
    }

    @Override
    public String transfer(String uId, String tradeId, BigDecimal amount) {
        System.out.println("微信渠道支付划账开始......");
        boolean security = payMode.security(uId);
        System.out.println("微信渠道支付风险校验: " + uId + " , " + tradeId +" , " + security);

        if(!security){
            System.out.println("微信渠道支付划账失败!");
            return "500";
        }
        System.out.println("微信渠道划账成功! 金额: "+ amount);
        return "200";
    }
}

/**
 * 支付渠道-支付宝划账
 **/
public class ZfbPay extends Pay {
    public ZfbPay(IPayMode payMode) {
        super(payMode);
    }

    @Override
    public String transfer(String uId, String tradeId, BigDecimal amount) {
        System.out.println("支付宝渠道支付划账开始......");
        boolean security = payMode.security(uId);
        System.out.println("支付宝渠道支付风险校验: " + uId + " , " + tradeId +" , " + security);

        if(!security){
            System.out.println("支付宝渠道支付划账失败!");
            return "500";
        }
        System.out.println("支付宝渠道划账成功! 金额: "+ amount);
        return "200";
    }
}
```

**5) 测试**

```java
@Test
public void test02() {
    System.out.println("测试场景1: 微信支付、人脸方式.");
    Pay wxpay = new WxPay(new PayFaceMode());
    wxpay.transfer("wx_00100100","10001900",new BigDecimal(100));

    System.out.println();

    System.out.println("测试场景2: 支付宝支付、指纹方式");
    Pay zfbPay = new ZfbPay(new PayFingerprintMode());
    zfbPay.transfer("jlu1234567","567689999999",new BigDecimal(200));
}
```

代码重构完成后,结构更加清晰整洁, 可读性和易用性更高,外部的使用接口的用户不需要关心具体实现.桥接模式满足了单一职责原则和开闭原则,让每一部分都更加清晰并且易扩展.

### 5.2.4 桥接模式总结

**桥接模式的优点:**

1. 分离抽象接口及其实现部分.桥接模式使用"对象间的关联关系"解耦了抽象和实现之间固有的绑定关系,使得抽象和实现可以沿着各自的维度来变化.
2. 在很多情况下,桥接模式可以取代多层继承方案.多层继承方案违背了单一职责原则,复用性差,类的个数多.桥接模式很好的解决了这些问题.
3. 桥接模式提高了系统的扩展性,在两个变化维度中任意扩展一个维度都不需要修改原有系统,符合开闭原则.

**桥接模式的缺点:**

1. 桥接模式的使用会增加系统的理解和设计难度,由于关联关系建立在抽象层,要求开发者一开始就要对抽象层进行设计和编程
2. 桥接模式要求正确识别出系统中的两个独立变化的维度,因此具有一定的局限性,并且如果正确的进行维度的划分,也需要相当丰富的经验.

**桥接模式使用场景**

1. 需要提供平台独立性的应用程序时。 比如，不同数据库的 JDBC 驱动程序、硬盘驱动程序等。

2. 需要在某种统一协议下增加更多组件时。 比如，在支付场景中，我们期望支持微信、支付宝、各大银行的支付组件等。这里的统一协议是收款、支付、扣款，而组件就是微信、支付宝等。

3. 基于消息驱动的场景。 虽然消息的行为比较统一，主要包括发送、接收、处理和回执，但其实具体客户端的实现通常却各不相同，比如，手机短信、邮件消息、QQ 消息、微信消息等。

4. 拆分复杂的类对象时。 当一个类中包含大量对象和方法时，既不方便阅读，也不方便修改。

5. 希望从多个独立维度上扩展时。 比如，系统功能性和非功能性角度，业务或技术角度等。



## 5.3 装饰器模式

### 5.3.1 装饰器模式介绍

装饰模式(decorator pattern) 的原始定义是：动态的给一个对象添加一些额外的职责。就扩展功能而言，装饰器模式提供了一种比使用子类更加灵活的替代方案。

> 假设现在有有一块蛋糕，如果只有涂上奶油那这个蛋糕就是普通的**奶油蛋糕**，这时如果我们添加上一些蓝莓，那这个蛋糕就是**蓝莓蛋糕**。如果我们再拿一块黑巧克力 然后写上姓名、插上代表年龄的蜡烛，这就是变成了一块生日蛋糕 

<img src=".\img\81.jpg" alt="image-20220530160637842" style="zoom: 50%;" />



在软件设计中,装饰器模式是一种用于**替代继承**的技术，它通过一种无须定义子类的方式给对象动态的增加职责，使用对象之间的关联关系取代类之间的继承关系。

装饰器的核心思想是“**动态扩展功能**”，不直接修改原始类的代码，而是通过组合和封装的方式，在运行时为对象添加新的行为或职责。

### 5.3.2 装饰器模式原理

装饰（Decorator）模式中的角色：

* 抽象构件（Component）角色 ：它是具体构件和抽象装饰类的共同父类,声明了在具体构件中实现的业务方法.它引进了可以使客户端以一致的方式处理未被装饰的对象以及装饰之后的对象,实现客户端的透明操作
* 具体构件（Concrete  Component）角色 ：它是抽象构件类的子类,用于定义具体的构建对象,实现了在抽象构建中声明的方法,装饰类可以给它增加额外的职责(方法).
* 抽象装饰（Decorator）角色 ：它也是抽象构件类的子类,用于给具体构件增加职责,但是具体职责在其子类中实现.它维护了一个指向抽象构件对象的引用,通过该引用可以调用装饰之前构件对象的方法,并通过其子类扩展该方法,以达到装饰的目的.
* 具体装饰（ConcreteDecorator）角色 : 它是抽象装饰类的子类,负责向构件添加新的职责.每一个具体装饰类都定义了一些新的行为,它可以调用在抽象装饰类中定义的方法,并可以增加新的方法用于扩充对象的行为.

<img src=".\img\82.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

代码如下

```java
/**
 * 抽象构件类
 **/
public abstract class Component {
    //抽象方法
    public abstract void operation();
}

/**
 * 具体构建类
 **/
public class ConcreteComponent extends Component {
    @Override
    public void operation() {
        //基础功能实现(复杂功能通过装饰类进行扩展)
    }
}
```

```java
/**
 * 抽象装饰类-装饰者模式的核心
 **/
public class Decorator extends Component{
    //维持一个对抽象构件对象的引用
    private Component component;
    //注入一个抽象构件类型的对象
    public Decorator(Component component) {
        this.component = component;
    }

    @Override
    public void operation() {
        //调用原有业务方法(这里并没有真正实施装饰,而是提供了一个统一的接口,将装饰过程交给子类完成)
        component.operation();
    }
}


/**
 * 具体装饰类
 **/
public class ConcreteDecorator extends Decorator {
    public ConcreteDecorator(Component component) {
        super(component);
    }

    @Override
    public void operation() {
        super.operation(); //调用原有业务方法
        addedBehavior();   //调用新增业务方法
    }

    //新增业务方法
    public void addedBehavior(){
        //......
    }
}
```

### 5.3.3 装饰器模式应用实例

我们以一个文件读写器程序为例, 演示一下装饰者模式的使用,下面是该程序的UML类图

<img src=".\img\84.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

- **DataLoader** 
  - 抽象的文件读取接口DataLoader 
- **BaseFileDataLoader**
  - 具体组件BaseFileDataLoader，重写组件 DataLoader 的读写方法
- **DataLoaderDecorator**
  - 装饰器DataLoaderDecorator，这里要包含一个引用 DataLoader 的对象实例 wrapper，同样是重写 DataLoader 方法，不过这里使用 wrapper 来读写,并不进行扩展
- **EncryptionDataDecorator**
  - 读写时有加解密功能的具体装饰器EncryptionDataDecorator，它继承了装饰器 DataLoaderDecorator 重写读写方法

导入IO工具类

```xml
<dependencies>
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
</dependencies>
```

1 ) DataLoader

```java
/**
 * 抽象的文件读取接口DataLoader
 **/
public interface DataLoader {
    String read();
    void write(String data);
}
```

2 ) BaseFileDataLoader

```java
/**
 * 具体组件,重写读写方法
 **/
public class BaseFileDataLoader implements DataLoader {
    private String filePath;
    
    public BaseFileDataLoader(String filePath) {
        this.filePath = filePath;
    }

    @Override
    public String read() {
        try {
            String result = FileUtils.readFileToString(new File(filePath), "utf-8");
            return result;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    @Override
    public void write(String data) {
        try{
            FileUtils.writeStringToFile(new File(filePath), data, "utf-8");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

3 ) DataLoaderDecorator

```java
/**
 * 装抽象饰者类
 **/
public class DataLoaderDecorator implements DataLoader {
    private DataLoader wrapper;

    public DataLoaderDecorator(DataLoader wrapper) {
        this.wrapper = wrapper;
    }

    @Override
    public String read() {
        return wrapper.read();
    }

    @Override
    public void write(String data) {
        wrapper.write(data);
    }
}
```

4 ) EncryptionDataDecorator

```java
/**
 * 具体装饰者-对文件内容进行加密和解密
 **/
public class EncryptionDataDecorator extends DataLoaderDecorator {
    public EncryptionDataDecorator(DataLoader wrapper) {
        super(wrapper);
    }

    @Override
    public String read() {
        return decode(super.read());
    }

    @Override
    public void write(String data) {
        super.write(encode(data));
    }

    //加密操作
    private String encode(String data) {
        try {
             Base64.Encoder encoder = Base64.getEncoder();
             byte[] bytes = data.getBytes("UTF-8");
             String result = encoder.encodeToString(bytes);
             return result;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    //解密
    private String decode(String data) {
        try {
            Base64.Decoder decoder = Base64.getDecoder();
            String result = new String(decoder.decode(data), "UTF-8");
            return result;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

5 ) 测试

```java
public class TestDecorator {
    public static void main(String[] args) {
        String info = "name:tom，age:15";
        DataLoaderDecorator decorator = new EncryptionDataDecorator(new BaseFileDataLoader("demo.txt"));
        decorator.write(info);

        String data = decorator.read();
        System.out.println(data);
    }
}
```

**例子2**

假设我们有一个饮料类，我们希望在不修改原始类的情况下给它加上不同的调料（比如牛奶、糖等）。这个场景非常适合使用装饰器模式。

```java
// Component
interface Beverage {
    double cost(); // 计算饮料的价格
}

// ConcreteComponent
class Coffee implements Beverage {
    @Override
    public double cost() {
        return 5.0; // 咖啡的基础价格
    }
}

// Decorator
abstract class BeverageDecorator   {
    protected Beverage beverage;
    
    public BeverageDecorator(Beverage beverage) {
        this.beverage = beverage;
    }
}

// ConcreteDecorator 1
class MilkDecorator extends BeverageDecorator {
    public MilkDecorator(Beverage beverage) {
        super(beverage);
    }

    @Override
    public double cost() {
        return beverage.cost() + 1.0; // 加牛奶的额外费用
    }
}

// ConcreteDecorator 2
class SugarDecorator extends BeverageDecorator {
    public SugarDecorator(Beverage beverage) {
        super(beverage);
    }

    @Override
    public double cost() {
        return beverage.cost() + 0.5; // 加糖的额外费用
    }
}

// 测试代码
public class Main {
    public static void main(String[] args) {
        Beverage coffee = new Coffee();
        System.out.println("基础咖啡价格: " + coffee.cost());

        coffee = new MilkDecorator(coffee); // 给咖啡加牛奶
        System.out.println("加牛奶的咖啡价格: " + coffee.cost());

        coffee = new SugarDecorator(coffee); // 再加糖
        System.out.println("加牛奶和糖的咖啡价格: " + coffee.cost());
    }
}
```

### 5.3.4 装饰器模式总结

 **装饰器模式的优点:**

1. 对于扩展一个对象的功能,装饰模式比继承更加灵活,不会导致类的个数急剧增加
2. 可以通过一种动态的方式来扩展一个对象的功能,通过配置文件可以在运行时选择不同的具体装饰类,从而实现不同的行为.
3. 可以对一个对象进行多次装饰,通过使用不同的具体装饰类以及这些装饰类的排列组合可以创造出很多不同行为的组合,得到更加强大的对象.
4. 具体构建类与具体装饰类可以独立变化,用户可以根据需要增加新的具体构建类和具体装饰类,原有类库代码无序改变,符合开闭原则.

**装饰器模式的缺点:**

1. 在使用装饰模式进行系统设计时将产生很多小对象,这些对象的区别在于它们之间相互连接的方式有所不同,而不是它们的类或者属性值不同,大量的小对象的产生势必会占用更多的系统资源,在一定程度上影响程序的性能.
2. 装饰器模式提供了一种比继承更加灵活、机动的解决方案,但同时也意味着比继承更加易于出错,排错也更加困难,对于多次装饰的对象,在调试寻找错误时可能需要逐级排查,较为烦琐.

**装饰器模式的适用场景**

1. 快速动态扩展和撤销一个类的功能场景。 比如，有的场景下对 API 接口的安全性要求较高，那么就可以使用装饰模式对传输的字符串数据进行压缩或加密。如果安全性要求不高，则可以不使用。

2. 不支持继承扩展类的场景。 比如，使用 final 关键字的类，或者系统中存在大量通过继承产生的子类。



## 5.4 适配器模式

### 5.4.1 适配器模式介绍

适配器模式(adapter pattern )的原始定义是：将类的接口转换为客户期望的另一个接口，适配器可以让不兼容的两个类一起协同工作。

> 如果去欧洲国家去旅游的话，他们的插座如下图最左边，是欧洲标准。而我们使用的插头如下图最右边的。因此我们的笔记本电脑，手机在当地不能直接充电。所以就需要一个插座转换器，转换器第1面插入当地的插座，第2面供我们充电，这样使得我们的插头在当地能使用。生活中这样的例子很多，手机充电器（将220v转换为5v的电压），读卡器等，其实就是使用到了适配器模式。

<img src=".\img\85.jpg" alt="image-20220530160637842" style="zoom: 100%;" />

适配器模式是用来做适配，它将不兼容的接口转换为可兼容的接口，让原本由于接口不兼容而不能一起工作的类可以一起工作。适配器模式有两种实现方式：类适配器和对象适配器。其中，类适配器使用继承关系来实现，对象适配器使用组合关系来实现。

**类适配器模式的耦合度比后者高，且要求程序员了解现有组件库中的相关组件的内部结构，所以应用相对较少些。**

### 5.4.2 适配器模式原理

适配器模式（Adapter）包含以下主要角色：

* 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口。
* 适配者（Adaptee）类：适配者即被适配的角色,它是被访问和适配的现存组件库中的组件接口。
* 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

<img src=".\img\87.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  

<img src=".\img\88.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

### 5.4.3 适配器模式应用实例

#### 5.4.3.1 类适配器模式

假设现有一台电脑目前**只能读取**SD卡的信息，这时我们想要使用电脑读取TF卡的内容, 就需要将TF卡加上卡套，转换成SD卡!

创建一个读卡器，将TF卡中的内容读取出来。

类图如下：

<img src=".\img\89.jpg" alt="image-20220530160637842" style="zoom: 70%;" /> 

代码如下:

```java
/**
 * SD卡接口
 **/
public interface SDCard {
    //读取SD卡方法
    String readSD();
    //写入SD卡功能
    void writeSD(String msg);
}

/**
 * SD卡实现类
 **/
public class SDCardImpl implements SDCard {
    @Override
    public String readSD() {
        String msg = "sd card reading data";
        return msg;
    }

    @Override
    public void writeSD(String msg) {
        System.out.println("sd card write data : " + msg);
    }
}

/**
 * TF卡接口
 **/
public interface TFCard {
    //读取TF卡方法
    String readTF();
    //写入TF卡功能
    void writeTF(String msg);
}

/**
 * TF卡实现类
 **/
public class TFCardImpl implements TFCard {
    @Override
    public String readTF() {
        String msg = "tf card reading data";
        return msg;
    }

    @Override
    public void writeTF(String msg) {
        System.out.println("tf card write data : " + msg);
    }
}

/**
 * 定义适配器类(SD兼容TF)
 **/
public class SDAdapterTF extends  TFCardImpl implements SDCard{
    @Override
    public String readSD() {
        System.out.println("adapter read tf card ");
        return readTF();
    }

    @Override
    public void writeSD(String msg) {
        System.out.println("adapter write tf card");
        writeTF(msg);
    }
}

public class Client {
    public static void main(String[] args) {
        Computer computer = new Computer();
        SDCard sdCard = new SDCardImpl();
        System.out.println(computer.read(sdCard));

        System.out.println("========================");
        SDAdapterTF adapterTF = new SDAdapterTF();
        System.out.println(computer.read(adapterTF));
    }
}
```

#### 5.43.2 对象适配器模式

实现方式：对象适配器模式可釆用将现有组件库中已经实现的组件引入适配器类中，该类同时实现当前系统的业务接口。

<img src=".\img\90.jpg" alt="image-20220530160637842" style="zoom: 70%;" /> 



代码如下：

类适配器模式的代码，我们只需要修改适配器类（SDAdapterTF）和测试类。

```java
public class SDAdapterTF implements SDCard{
    private TFCard tfCard;

    public SDAdapterTF(TFCard tfCard) {
        this.tfCard = tfCard;
    }

    @Override
    public String readSD() {
        System.out.println("adapter read tf card ");
        return tfCard.readTF();
    }

    @Override
    public void writeSD(String msg) {
        System.out.println("adapter write tf card");
        tfCard.writeTF(msg);
    }
}

public class Client {
    public static void main(String[] args) {
        Computer computer = new Computer();
        SDCard sdCard = new SDCardImpl();
        System.out.println(computer.read(sdCard));

        System.out.println("========================");
        TFCard tfCard = new TFCardImpl();
        SDAdapterTF adapterTF = new SDAdapterTF(tfCard);
        System.out.println(computer.read(adapterTF));
    }
}
```



### 5.4.3 适配器模式总结

**适配器模式的优点**

1. 将目标类和适配者类解耦,通过引入一个适配器类来重用现有的适配者类,无序修改原有结构
2. 增加了类的透明性和复用性,将具体业务实现过程封装在适配者类中,对于客户端类而言是透明的,而且提高了适配者的复用性,同一个适配者类可以在多个不同的系统中复用.
3. 灵活性和扩展性都非常好,通过使用配置文件可以很方便的更换适配器,也可以在不修改原有代码的基础上增加新的适配器类,符合开闭原则.

**适配器模式的缺点**

- 类适配器的缺点
  1. 对于Java等不支持多重继承的语言,一次最多只能适配一个适配者类,不能同时适配多个适配者
  2. 适配者类不能为最终类
- 对象适配器的缺点
  1. 与类适配器模式相比较,在该模式下要在适配器中置换适配者类的某些方法比较麻烦.

**适配器模式适用的场景**

- 统一多个类的接口设计时

  > 某个功能的实现依赖多个外部系统（或者说类）。通过适配器模式，将它们的接口适配为统一的接口定义

- 需要依赖外部系统时

  > 当我们把项目中依赖的一个外部系统替换为另一个外部系统的时候，利用适配器模式，可以减少对代码的改动

- 原有接口无法修改时或者原有接口功能太老旧但又需要兼容；

  > JDK1.0 Enumeration 到 Iterator 的替换,适用适配器模式保留 Enumeration 类，并将其实现替换为直接调用 Itertor.

- 适配不同数据格式时；

  > Slf4j 日志框架,定义打印日志的统一接口,提供针对不同日志框架的适配器

**代理、桥接、装饰器、适配器 4 种设计模式的区别**

代理、桥接、装饰器、适配器，这 4 种模式是比较常用的结构型设计模式。它们的代码结构非常相似.但其各自的用意却不同,简单说一下它们之间的关系

- 代理模式：代理模式在不改变原始类接口的条件下，为原始类定义一个代理类，主要目的是控制访问，而非加强功能，这是它跟装饰器模式最大的不同。

- 桥接模式：桥接模式的目的是将接口部分和实现部分分离，从而让它们可以较为容易、也相对独立地加以改变。

- 装饰器模式：装饰者模式在不改变原始类接口的情况下，对原始类功能进行增强，并且支持多个装饰器的嵌套使用。

- 适配器模式：将一个类的接口转换为客户希望的另一个接口.适配器模式让那些不兼容的类可以一起工作.



## 5.5 外观模式

### 5.5.1 外观模式介绍

外观模式( Facade Pattern)，也叫门面模式, 外观模式的原始定义是：为子系统中的一组接口提供统一的接口。它定义了一个更高级别的接口，使子系统更易于使用。

外观模式，是一种通过为多个复杂的子系统提供一个一致的接口，而使这些子系统更加容易被访问的模式。该模式对外有一个统一接口，外部应用程序不用关心内部子系统的具体的细节，这样会大大降低应用程序的复杂度，提高了程序的可维护性。

门面模式有点类似之前讲到的迪米特法则（最少知识原则）和接口隔离原则：两个有交互的系统，只暴露有限的必要的接口

<img src=".\img\91.jpg" alt="image-20220530160637842" style="zoom: 70%;" /> 

门面类充当了系统中的"服务员",它为多个业务类的调用提供了一个统一的入口,简化了类与类之间的交互,如果没有门面类,每个客户类需要和多个子系统之间进行复杂的交互,系统的耦合度将会很大.

### 5.5.2 外观模式原理

<img src=".\img\92.jpg" alt="image-20220530160637842" style="zoom: 50%;" />

外观（Facade）模式包含以下主要角色：

* 外观（Facade）角色：为多个子系统对外提供一个共同的接口。

  > 外观角色中可以知道多个相关的子系统中的功能和责任.在正常情况下,它将所有从客户端发来的请求委派到相应的子系统,传递给相应的子系统对象处理

* 子系统（Sub System）角色：实现系统的部分功能，客户可以通过外观角色访问它。

  > 每一个子系统可以是一个类也可以是多个类的集合.每一个子系统都可以被客户端直接调用,或者被外观角色调用.子系统并不	知道外观的存在,对于子系统而言,外观角色仅仅是另一个客户端而已.

代码示例

```java
public class SubSystemA {
    public void methodA(){
        //业务代码
    }
}

public class SubSystemB {
    public void methodB(){
        //业务代码
    }
}

public class SubSystemC {
    public void methodC(){
        //业务代码
    }
}

public class Facade {
    private SubSystemA obj1 = new SubSystemA();
    private SubSystemB obj2 = new SubSystemB();
    private SubSystemC obj3 = new SubSystemC();

    public void method(){
        obj1.methodA();
        obj2.methodB();
        obj3.methodC();
    }
}

public class Client {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.method();
    }
}
```



### 5.5.3 外观模式应用实例

智能家电控制

- 通过智能音箱来控制室内的 灯、电视、空调.本来每个设备都需要进行独立的开关操作,现在通过智能音箱完成对这几个设备的统一控制. 

类图如下

<img src=".\img\93.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

代码如下

```java
public class Light {
    public void on(){
        System.out.println("打开灯......");
    }

    public void off(){
        System.out.println("关闭灯......");
    }
}

public class TV {
    public void on(){
        System.out.println("打开电视......");
    }

    public void off(){
        System.out.println("关闭电视......");
    }
}

public class AirCondition {
    public void on(){
        System.out.println("打开空调......");
    }

    public void off(){
        System.out.println("关闭空调......");
    }
}

public class SmartAppliancesFacade {
    private Light light;
    private TV tv;
    private AirCondition airCondition;

    public SmartAppliancesFacade() {
        this.light =new Light();
        this.tv = new TV();
        this.airCondition = new AirCondition();
    }

    public void say(String message){
        if(message.contains("打开")){
            on();
        }else if(message.contains("关闭")){
            off();
        }else{
            System.out.println("对不起没有听清楚您说什么! 请重新再说一遍");
        }
    }

    //起床后 语音开启 电灯 电视 空调
    private void on() {
        System.out.println("起床了!");
        light.on();
        tv.on();
        airCondition.on();
    }

    //睡觉前 语音关闭 电灯 电视 空调
    private void off() {
        System.out.println("睡觉了!");
        light.off();
        tv.off();
        airCondition.off();
    }
}

public class Client {
    public static void main(String[] args) {
        //创建外观对象
        SmartAppliancesFacade facade = new SmartAppliancesFacade();

        facade.say("打开家电");
        facade.say("关闭家电");
    }
}
```



### 5.5.4 外观模式总结

**外观模式的优点:**

1. 它对客户端屏蔽了子系统组件,减少了客户端所需要处理的对象数目,并使子系统使用起来更加的容易.通过引入外观模式,客户端代码将变得很简单,与之关联的对象也很少.
2. 它实现了子系统与客户端之间的松耦合关系,这使得子系统的变化不会影响到调用它的客户端,只需要调整外观类即可
3. 一个子系统的修改对其他子系统没有任何影响,而子系统内部变化也不会影响到外观对象. 

**外观模式缺点:**

1. 不能很好的控制客户端直接使用子系统类,如果客户端访问子系统类做太多的限制则减少了可变性和灵活性.
2. 如果设计不当,增加新的子系统可能需要修改外观类的源代码,违背了开闭原则.

**使用场景分析:** 

- 简化复杂系统。 比如，当我们开发了一整套的电商系统后（包括订单、商品、支付、会员等系统），我们不能让用户依次使用这些系统后才能完成商品的购买，而是需要一个门户网站或手机 App 这样简化过的门面系统来提供在线的购物功能。
- 减少客户端处理的系统数量。 比如，在 Web 应用中，系统与系统之间的调用可能需要处理 Database 数据库、Model 业务对象等，其中使用 Database 对象就需要处理打开数据库、关闭连接等操作，然后转换为 Model 业务对象，实在是太麻烦了。如果能够创建一个数据库使用的门面（其实就是常说的 DAO 层），那么实现以上过程将变得容易很多。
- 让一个系统（或对象）为多个系统（或对象）工作。 比如，线程池 ThreadPool 就是一个门面模式，它为系统提供了统一的线程对象的创建、销毁、使用等。
- 联合更多的系统来扩展原有系统。 当我们的电商系统中需要一些新功能时，比如，人脸识别，我们可以不需要自行研发，而是购买别家公司的系统来提供服务，这时通过门面系统就能方便快速地进行扩展。
- 作为一个简洁的中间层。 门面模式还可以用来隐藏或者封装系统中的分层结构，同时作为一个简化的中间层来使用。比如，在秒杀、库存、钱包等场景中，我们需要共享有状态的数据时（如商品库存、账户里的钱），在不改变原有系统的前提下，通过一个中间的共享层（如将秒杀活动的商品库存总数统一放在 Redis 里），就能统一进行各种服务（如，秒杀详情页、商品详情页、购物车等）的调用。



## 5.6 组合模式

我们很容易将“组合模式”和“组合关系”搞混。组合模式最初只是用于解决树形结构的场景，更多的是处理对象组织结构之间的问题。而组合关系则是通过将不同对象封装起来完成一个统一功能.

### 5.6.1 组合模式介绍

组合模式(Composite Pattern) 的定义是：将对象组合成树形结构以表示整个部分的层次结构.组合模式可以让用户统一对待单个对象和对象的组合.

比如: windows操作系统中的目录结构,其实就是树形目录结构,通过tree命令实现树形结构展示. 

<img src=".\img\94.jpg" alt="image-20220530160637842" style="zoom: 50%;" />	

在上图中包含了文件夹和文件两类不同元素,其中在文件夹中可以包含文件,还可以继续包含子文件夹.子文件夹中可以放入文件,也可以放入子文件夹. 文件夹形成了一种容器结构(树形结构),递归结构.

<img src=".\img\95.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

接着我们再来思考虽然文件夹和文件是不同类型的对象,但是他们有一个共性,就是 **都可以被放入文件夹中**. 其实文件和文件夹可以被当做是同一种对象看待.

组合模式其实就是将一组对象(文件夹和文件)组织成树形结构,以表示一种'部分-整体' 的层次结构,(目录与子目录的嵌套结构). 组合模式让客户端可以统一单个对象(文件)和组合对象(文件夹)的处理逻辑(递归遍历).

组合模式更像是一种数据结构和算法的抽象,其中数据可以表示成树这种数据结构,业务需求可以通过在树上的递归遍历算法来实现. 

### 5.6.2 组合模式原理

<img src=".\img\96.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

组合模式主要包含三种角色：

* 抽象根节点（Component）：定义系统各层次对象的共有方法和属性，可以预先定义一些默认行为和属性。

  > 在该角色中可以包含所有子类共有行为的声明和实现.在抽象根节点中定义了访问及管理它的子构件的方法,如增加子节点、删除子节点、获取子节点等.

* 树枝节点（Composite）：定义树枝节点的行为，存储子节点，组合树枝节点和叶子节点形成一个树形结构。

  > 树枝节点可以包含树枝节点,也可以包含叶子节点,它其中有一个集合可以用于存储子节点,实现了在抽象根节点中定义的行为.包括那些访问及管理子构件的方法,在其业务方法中可以递归调用其子节点的业务方法.

* 叶子节点（Leaf）：叶子节点对象，其下再无分支，是系统层次遍历的最小单位。

  > 在组合结构中叶子节点没有子节点,它实现了在抽象根节点中定义的行为.

### 5.6.3 组合模式实现

组合模式的关键在于定义一个抽象根节点类,它既可以代表叶子,又可以代表树枝节点,客户端就是针对该抽象类进行编程,不需要知道它到底表示的是叶子还是容器,可以对其进行统一处理.

树枝节点对象和抽象根节点类之间建立了一个聚合关联关系,在树枝节点对象中既可以包含叶子节点,还可以继续包含树枝节点,以此实现递归组合,形成一个树形结构.

代码实现

```java
/**
 * 抽象根节点
 * 对于客户端而言将针对抽象编程,无需关心其具体子类是容器构件还是叶子构件.
 **/
public abstract class Component {
    public abstract void add(Component c); //增加成员
    public abstract void remove(Component c); //删除成员
    public abstract Component getChild(int i); //获取成员
    public abstract void operation(); //业务方法
}

/**
 * 叶子节点
 * 叶子节点中不能包含子节点
 **/
public class Leaf extends Component {
    @Override
    public void add(Component c) {
        //具体操作
    }

    @Override
    public void remove(Component c) {
        //具体操作
    }

    @Override
    public Component getChild(int i) {
        //具体操作
        return new Leaf();
    }

    @Override
    public void operation() {
        //叶子节点具体业务方法
    }
}

/**
 * 树枝节点
 * 容器对象,可以包含子节点
 **/
public class Composite extends Component {
    private ArrayList<Component> list = new ArrayList<>();

    @Override
    public void add(Component c) {
        list.add(c);
    }

    @Override
    public void remove(Component c) {
        list.remove(c);
    }

    @Override
    public Component getChild(int i) {
        return (Component) list.get(i);
    }

    @Override
    public void operation() {
        //在树枝节点中的业务方法,将递归调用其他节点中的operation() 方法
        for (Component component : list) {
            component.operation();
        }
    }
}
```

### 5.6.4 组合模式应用实例

下面我们通过一段程序来演示一下组合模式的使用. 程序的功能是列出某一目录下所有的文件和文件夹.类图如下: 

<img src=".\img\98.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

我们按照下图的表示,进行文件和文件夹的构建.

<img src=".\img\97.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

Entry类: 抽象类,用来定义File类和Directory类的共性内容

```java
/**
 * Entry抽象类,表示目录条目(文件+文件夹)的抽象类
 **/
public abstract class Entry {
    public abstract String getName(); //获取文件名
    public abstract int getSize(); //获取文件大小
    //添加文件夹或文件
    public abstract Entry add(Entry entry);
    //显示指定目录下的所有信息
    public abstract void printList(String prefix);

    @Override
    public String toString() {
        return getName() + "(" +getSize() + ")";
    }
}
```

File类,叶子节点,表示文件.

```java
/**
 * File类 表示文件
 **/
public class File extends Entry {
    private String name; //文件名
    private int size; //文件大小

    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public int getSize() {
        return size;
    }

    @Override
    public Entry add(Entry entry) {
        return null;
    }

    @Override
    public void printList(String prefix) {
        System.out.println(prefix + "/" + this);
    }
}
```

Directory类,树枝节点,表示文件

```java
/**
 * Directory表示文件夹
 **/
public class Directory extends Entry{
    //文件的名字
    private String name;
    //文件夹与文件的集合
    private ArrayList<Entry> directory = new ArrayList();

    //构造函数
    public Directory(String name) {
        this.name = name;
    }

    //获取文件名称
    @Override
    public String getName() {
        return this.name;
    }

    /**
     * 获取文件大小
     *      1.如果entry对象是File类型,则调用getSize方法获取文件大小
     *      2.如果entry对象是Directory类型,会继续调用子文件夹的getSize方法,形成递归调用.
     */
    @Override
    public int getSize() {
        int size = 0;

        //遍历或者去文件大小
        for (Entry entry : directory) {
            size += entry.getSize();
        }
        return size;
    }

    @Override
    public Entry add(Entry entry) {
        directory.add(entry);
        return this;
    }

    //显示目录
    @Override
    public void printList(String prefix) {
        System.out.println("/" + this);
        for (Entry entry : directory) {
            entry.printList("/" + name);
        }
    }
}
```

测试

```java
public class Client {
    public static void main(String[] args) {
        //根节点
        Directory rootDir = new Directory("root");

        //树枝节点
        Directory binDir = new Directory("bin");
        //向bin目录中添加叶子节点
        binDir.add(new File("vi",10000));
        binDir.add(new File("test",20000));

        Directory tmpDir = new Directory("tmp");
        Directory usrDir = new Directory("usr");
        Directory mysqlDir = new Directory("mysql");
        mysqlDir.add(new File("my.cnf",30));
        mysqlDir.add(new File("test.db",25000));
        usrDir.add(mysqlDir);

        rootDir.add(binDir);
        rootDir.add(tmpDir);
        rootDir.add(mysqlDir);

        rootDir.printList("");
    }
}
```

### 5.6.5 组合模式总结

**1 ) 组合模式的分类**

* 透明组合模式

  透明组合模式中，抽象根节点角色中声明了所有用于管理成员对象的方法，比如在示例中 `Component` 声明了 `add`、`remove` 、`getChild` 方法，这样做的好处是确保所有的构件类都有相同的接口。透明组合模式也是组合模式的标准形式。

  透明组合模式的缺点是不够安全，因为叶子对象和容器对象在本质上是有区别的，叶子对象不可能有下一个层次的对象，即不可能包含成员对象，因此为其提供 add()、remove() 等方法是没有意义的，这在编译阶段不会出错，但在运行阶段如果调用这些方法可能会出错（如果没有提供相应的错误处理代码）

  <img src=".\img\99.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

* 在安全组合模式中，在抽象构件角色中没有声明任何用于管理成员对象的方法，而是在树枝节点类中声明并实现这些方法。安全组合模式的缺点是不够透明，因为叶子构件和容器构件具有不同的方法，且容器构件中那些用于管理成员对象的方法没有在抽象构件类中定义，因此客户端不能完全针对抽象编程，必须有区别地对待叶子构件和容器构件。

​		<img src=".\img\100.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 



**2 ) 组合模式优点** 

* 组合模式可以清楚地定义分层次的复杂对象，表示对象的全部或部分层次，它让客户端忽略了层次的差异，方便对整个层次结构进行控制。
* 客户端可以一致地使用一个组合结构或其中单个对象，不必关心处理的是单个对象还是整个组合结构，简化了客户端代码。
* 在组合模式中增加新的树枝节点和叶子节点都很方便，无须对现有类库进行任何修改，符合“开闭原则”。
* 组合模式为树形结构的面向对象实现提供了一种灵活的解决方案，通过叶子节点和树枝节点的递归组合，可以形成复杂的树形结构，但对树形结构的控制却非常简单。

**3 ) 组合模式的缺点**

- 使用组合模式的前提在于，你的**业务场景必须能够表示成树形结构**。所以，组合模式的应用场景也 比较局限，它并不是一种很常用的设计模式。

**4 ) 组合模式使用场景分析**

- 处理一个树形结构，比如，公司人员组织架构、订单信息等；

- 跨越多个层次结构聚合数据，比如，统计文件夹下文件总数；

- 统一处理一个结构中的多个对象，比如，遍历文件夹下所有 XML 类型文件内容。

  

## 5.7 享元模式

### 5.7.1 享元模式介绍

享元模式 (flyweight pattern) 的原始定义是：摒弃了在每个对象中保存所有数据的方式，通过共享多个对象所共有的相同状态，从而让我们能在**有限的内存容量**中载入更多对象。

从这个定义中你可以发现，享元模式要解决的核心问题就是**节约内存空间**，使用的办法是找出相似对象之间的共有特征，然后复用这些特征。所谓“享元”，顾名思义就是被**共享的单元**。

比如: 一个文本字符串中存在很多重复的字符,如果每一个字符都用一个单独的对象来表示,将会占用较多的内存空间,我们可以使用享元模式解决这一类问题.

​												<img src=".\img\101.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

享元模式通过共享技术实现相同或者相似对象的重用,在逻辑上每一个出现的字符都有一个对象与之对应,然而在物理上他们却是共享同一个享元对象.

### 5.7.2 享元模式原理

享元模式的结构较为复杂,通常会结合工厂模式一起使用,在它的结构图中包含了一个享元工厂类.

<img src=".\img\102.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

享元模式的主要有以下角色：

* 抽象享元角色（Flyweight）：通常是一个接口或抽象类，在抽象享元类中声明了具体享元类公共的方法，这些方法可以向外界提供享元对象的内部数据（内部状态），同时也可以通过这些方法来设置外部数据（外部状态）。

  享元（Flyweight）模式中存在以下两种状态：

  1. 内部状态，即不会随着环境的改变而改变的可共享部分。
  2. 外部状态，指随环境改变而改变的不可以共享的部分。享元模式的实现要领就是区分应用中的这两种状态，并将外部状态外部化。

* 可共享的具体享元（Concrete Flyweight）角色 ：它实现了抽象享元类，称为享元对象；在具体享元类中为内部状态提供了存储空间。通常我们可以结合单例模式来设计具体享元类，为每一个具体享元类提供唯一的享元对象。

* 非共享的具体享元（Unshared Flyweight)角色 ：并不是所有的抽象享元类的子类都需要被共享，不能被共享的子类可设计为非共享具体享元类；当需要一个非共享具体享元类的对象时可以直接通过实例化创建。

* 享元工厂（Flyweight Factory）角色 ：负责创建和管理享元角色。当客户对象请求一个享元对象时，享元工厂检査系统中是否存在符合要求的享元对象，如果存在则提供给客户；如果不存在的话，则创建一个新的享元对象。

  

### 5.7.3 享元模式实现

- 抽象享元类可以是一个接口也可以是一个抽象类，作为所有享元类的公共父类，主要作用是提高系统的可扩展性.

```java
/**
 * 抽象享元类
 **/
public abstract class Flyweight {
    public abstract void operation(String extrinsicState);
}
```

- 具体享元类，具体享元类中要将内部状态和外部状态分开处理，内部状态作为具体享元类的成员变量，而外部状态通过注入的方式添加到具体享元类中。

```java
/**
 * 可共享-具体享元类
 **/
public class ConcreteFlyweight extends Flyweight {
    //内部状态 intrinsicState作为成员变量,同一个享元对象的内部状态是一致的
    private String intrinsicState;

    public ConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }

    /**
     * 外部状态在使用时由外部设置,不保存在享元对象中,即使是同一个对象
     * @param extrinsicState  外部状态,每次调用可以传入不同的外部状态
     */
    @Override
    public void operation(String extrinsicState) {
        //实现业务方法
        System.out.println("=== 享元对象内部状态" + intrinsicState +",外部状态:" + extrinsicState);
    }
}
```

- 非共享享元类，不复用享元工厂内部状态，但是是抽象享元类的子类或实现类

```java
/**
 * 非共享具体享元类
 **/
public class UnsharedConcreteFlyweight extends Flyweight {
    private String intrinsicState;

    public UnsharedConcreteFlyweight(String intrinsicState) {
        this.intrinsicState = intrinsicState;
    }

    @Override
    public void operation(String extrinsicState) {
        System.out.println("=== 使用不共享对象,内部状态: " + intrinsicState +",外部状态: " + extrinsicState);
    }
}
```

- 享元工厂类，管理一个享元对象类的缓存池。它会存储享元对象之间需要传递的共有状态，比如，按照大写英文字母来作为状态标识，这种只在享元对象之间传递的方式就叫内部状态。同时，它还提供了一个通用方法 getFlyweight()，主要通过内部状态标识来获取享元对象。

```java
/**
 * 享元工厂类
 *      作用: 作为存储享元对象的享元池.用户获取享元对象时先从享元池获取,有则返回,没有创建新的
 *      享元对象返回给用户,并在享元池中保存新增的对象.
 **/
public class FlyweightFactory {
    //定义一个HashMap用于存储享元对象,实现享元池
    private Map<String，Flyweight> pool = new HashMap();
    
    public FlyweightFactory() {
        //添加对应的内部状态
        pool.put("A",new ConcreteFlyweight("A"));
        pool.put("B",new ConcreteFlyweight("B"));
        pool.put("C",new ConcreteFlyweight("C"));
    }

    //根据内部状态来进行查找
    public Flyweight getFlyweight(String key){
        //对象存在,从享元池直接返回
        if(pool.containsKey(key)){
            System.out.println("===享元池中存在,直接复用,key:" + key);
            return pool.get(key);
        }else{
            //如果对象不存在,先创建一个新的对象添加到享元池中,然后返回
            System.out.println("===享元池中不存在,创建并复用,key:" + key);
            Flyweight fw = new ConcreteFlyweight(key);
            pool.put(key，fw);
            return fw;
        }
    }
}
```

### 5.7.4 享元模式应用实例

五子棋中有大量的黑子和白子，它们的形状大小都是一样的,只是出现的位置不同，所以一个棋子作为一个独立的对象存储在内存中，会导致大量的内存的浪费，我们使用享元模式来进行优化

<img src=".\img\103.jpg" alt="image-20220530160637842" style="zoom: 90%;" /> 

类图如下

<img src=".\img\104.jpg" alt="image-20220530160637842" style="zoom: 90%;" /> 

代码如下

```java
/**
 * 抽象享元类: 五子棋类
 **/
public abstract class GobangFlyweight {
    public abstract String getColor();
    public void display(){
        System.out.println("棋子颜色: " + this.getColor());
    }
}

/**
 * 共享享元类-白色棋子
 **/
public class WhiteGobang extends GobangFlyweight{
    @Override
    public String getColor() {
        return "白色";
    }
}

/**
 * 共享享元类-黑色棋子
 **/
public class BlackGobang extends GobangFlyweight {
    @Override
    public String getColor() {
        return "黑色";
    }
}

/**
 * 享元工厂类-生产围棋棋子,使用单例模式进行设计
 **/
public class GobangFactory {
    private static GobangFactory factory = new GobangFactory();
    private static Map<String，GobangFlyweight> pool;

    //设置共享对象的内部状态,在享元对象中传递
    private GobangFactory() {
        pool = new HashMap<String，GobangFlyweight>();
        GobangFlyweight black = new BlackGobang(); //黑子
        GobangFlyweight white = new WhiteGobang(); //白子
        pool.put("b",black);
        pool.put("w",white);
    }

    //返回享元工厂类唯一实例
    public static final GobangFactory getInstance(){
        return SingletonHolder.INSTANCE;
    }

    //静态内部类-单例
    private static class SingletonHolder{
        private static final GobangFactory INSTANCE = new GobangFactory();
    }

    //通过key获取集合中的享元对象
    public GobangFlyweight getGobang(String key){
        return pool.get(key);
    }
}

public class Client {
    public static void main(String[] args) {
        //获取享元工厂对象
        GobangFactory instance = GobangFactory.getInstance();

        //获取3颗黑子
        GobangFlyweight b1 = instance.getGobang("b");
        GobangFlyweight b2 = instance.getGobang("b");
        GobangFlyweight b3 = instance.getGobang("b");
        System.out.println("判断两颗黑子是否相同: " + (b1 == b2));

        //获取2颗白子
        GobangFlyweight w1 = instance.getGobang("w");
        GobangFlyweight w2 = instance.getGobang("w");
        System.out.println("判断两颗白子是否相同: " + (w1 == w2));

        //显示棋子
        b1.display();
        b2.display();
        b3.display();
        w1.display();
        w2.display();
    }
}
```

三颗黑子(两颗白子)对象比较之后内存地址**都是一样的**.说明它们是同一个对象.在实现享元模式时使用了单例模式和简单工厂模式,保证了享元工厂对象的唯一性,并提供工厂方法向客户端返回享元对象.

### 5.7.5 享元模式总结

**1) 享元模式的优点**

- 极大减少内存中相似或相同对象数量，节约系统资源，提供系统性能

  > 比如，当大量商家的商品图片、固定文字（如商品介绍、商品属性）在不同的网页进行展示时，通常不需要重复创建对象，而是可以使用同一个对象，以避免重复存储而浪费内存空间。由于通过享元模式构建的对象是共享的，所以当程序在运行时不仅不用重复创建，还能减少程序与操作系统的 IO 交互次数，大大提升了读写性能。

- 享元模式中的外部状态相对独立，且不影响内部状态

**2) 享元模式的缺点**  

- 为了使对象可以共享，需要将享元对象的部分状态外部化，分离内部状态和外部状态，使程序逻辑复杂

**3) 使用场景**

- 一个系统有大量相同或者相似的对象，造成内存的大量耗费。

  注意: 在使用享元模式时需要维护一个存储享元对象的享元池，而这需要耗费一定的系统资源，因此，应当在需要多次重复使用享元对象时才值得使用享元模式。

- 在 Java 中，享元模式一个常用的场景就是，使用数据类的包装类对象的 valueOf() 方法。比如，使用 Integer.valueOf() 方法时，实际的代码实现中有一个叫 IntegerCache 的静态类，它就是一直缓存了 -127 到 128 范围内的数值，如下代码所示，你可以在 Java JDK 中的 Integer 类的源码中找到这段代码。

  ```java
  public class Test1 {
      public static void main(String[] args) {
          Integer i1 = 127;
          Integer i2 = 127;
          System.out.println("i1和i2对象是否是同一个对象？" + (i1 == i2));
  
          Integer i3 = 128;
          Integer i4 = 128;
          System.out.println("i3和i4对象是否是同一个对象？" + (i3 == i4));
      }
  }
  
  //传入的值在-128 - 127 之间,直接从缓存中返回
  public static Integer valueOf(int i) {
  	if (i >= IntegerCache.low && i <= IntegerCache.high)
  		return IntegerCache.cache[i + (-IntegerCache.low)];
      return new Integer(i);
  }
  ```
  
  可以看到 `Integer` 默认先创建并缓存 `-128 ~ 127` 之间数的 `Integer` 对象，当调用 `valueOf` 时如果参数在 `-128 ~ 127` 之间则计算下标并从缓存中返回，否则创建一个新的 `Integer` 对象。
  
  其实享元模式本质上就是找到对象的不可变特征，并缓存起来，当类似对象使用时从缓存中读取，以达到节省内存空间的目的。

