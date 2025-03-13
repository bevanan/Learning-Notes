# 第四章 创建型模式(5种)

创建型模式提供创建对象的机制,能够提升已有代码的灵活性和复用性

- 常用的有：单例模式、工厂模式（工厂方法和抽象工厂）、建造者模式。 


- 不常用的有：原型模式。

## 4.1 单例模式

创建型模式提供创建对象的机制,能够提升已有代码的灵活性和复用性

- 常用的有：单例模式、工厂模式（工厂方法和抽象工厂）、建造者模式。 


- 不常用的有：原型模式。

### 4.1.1 单例模式介绍

**1 ) 定义**

单例模式（Singleton Pattern）是 Java 中最简单的设计模式之一，此模式保证某个类在运行期间，只有一个实例对外提供服务，而这个类被称为单例类。

> 单例模式也比较好理解，比如一个人一生当中只能有一个真实的身份证号，一个国家只有一个政府，类似的场景都是属于单例模式。

**2 ) 使用单例模式要做的两件事**

1. 保证一个类只有一个实例
2. 为该实例提供一个全局访问节点

**3 ) 单例模式结构**

<img src=".\img\34.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 	

### 4.1.2 饿汉式

在类加载期间初始化静态实例,保证 instance 实例的创建是线程安全的 ( 实例在类加载时实例化，有JVM保证线程安全).

- 特点: 不支持延迟加载实例(懒加载) , 此中方式类加载比较慢，但是获取实例对象比较快

- 问题: 该对象足够大的话，而一直没有使用就会**造成内存的浪费**。

```java
public class Singleton_01 {
    //1. 私有构造方法
    private Singleton_01(){ }

    //2. 在本类中创建私有静态的全局对象
    private static Singleton_01 instance = new Singleton_01();

    //3. 提供一个全局访问点,供外部获取单例对象
    public static  Singleton_01 getInstance(){
        return instance;
    }
}
```



### 4.1.3 懒汉式(线程不安全版)

此种方式的单例实现了懒加载,只有调用getInstance方法时 才创建对象.但是如果是多线程情况,会出现线程安全问题.

```java
public class Singleton_02 {
    //1. 私有构造方法
    private Singleton_02(){ }

    //2. 在本类中创建私有静态的全局对象
    private static Singleton_02 instance;

    //3. 通过判断对象是否被初始化,来选择是否创建对象
    public static Singleton_02 getInstance(){
        if(instance == null){
            instance = new Singleton_02();
        }
        return instance;
    }
}
```

> 假设在单例类被实例化之前，==有两个线程同时在获取单例对象==，线程A在执行完if (instance == null) 后，线程调度机制将 CPU 资源分配给线程B，此时线程B在执行 if (instance == null)时也发现单例类还没有被实例化，这样就会导致单例类被实例化两次。为了防止这种情况发生，需要对 getInstance() 方法同步处理。改进后的懒汉模式.

### 4.1.4 懒汉式(线程安全版)

原理: 使用同步锁 `synchronized`锁住 创建单例的方法 ，防止多个线程同时调用，从而避免造成单例被多次创建

1. 即，`getInstance（）`方法块只能运行在1个线程中

2. 若该段代码已在1个线程中运行，另外1个线程试图运行该块代码，则 **会被阻塞而一直等待** 

3. 而在这个线程安全的方法里我们实现了单例的创建，保证了多线程模式下 单例对象的唯一性

```java
public class Singleton_03 {
    //1. 私有构造方法
    private Singleton_03(){ }

    //2. 在本类中创建私有静态的全局对象
    private static Singleton_03 instance;

    //3. 通过添加synchronize,保证多线程模式下的单例对象的唯一性
    public static synchronized Singleton_03 getInstance(){
        if(instance == null){
            instance = new Singleton_03();
        }
        return instance;
    }
}
```

> 懒汉式的缺点也很明显，我们给 getInstance() 这个方法加了一把**大锁**（synchronzed），导致这个函数的并发度很低。量化一下的话，并发度是 1，也就相当于串行操作了。而这个函数是在单例使用期间，一直会被调用。如果这个单例类偶尔会被用到，那这种实现方式还可以接受。但是，如果频繁地用到，那频繁加锁、释放锁及并发度低等问题，会导致性能瓶颈，这种实现方式就不可取了。

### 4.1.5 双重校验（懒汉安全版缩小锁范围）

饿汉式不支持延迟加载，懒汉式有性能问题，不支持高并发。那我们再来看一种既支持延迟加载、又支持高并发的单例实现方式，也就是双重检测实现方式。

实现步骤:

1. 在声明变量时使用了 volatile 关键字,其作用有两个: 

- **保证变量的可见性**：当一个被volatile关键字修饰的变量被一个线程修改的时候，其他线程可以立刻得到修改之后的结果。

- **屏蔽指令重排序**：指令重排序是编译器和处理器为了高效对程序进行优化的手段，它只能保证程序执行的结果时正确的，但是无法保证程序的操作顺序与代码顺序一致。这在单线程中不会构成问题，但是在多线程中就会出现问题。

2. 将同步方法改为同步代码块. 在同步代码块中使用二次检查，以保证其不被重复实例化 同时在调用getInstance()方法时不进行同步锁，效率高。

```java
/**
 * 单例模式-双重校验
 **/
public class Singleton_04 {
    //使用 volatile保证变量的可见性
    private volatile static Singleton_04 instance = null;

    private Singleton_04(){ }

    //对外提供静态方法获取对象
    public static Singleton_04 getInstance(){
        //第一次判断,如果instance不为null,不进入抢锁阶段,直接返回实例
        if(instance == null){
            synchronized (Singleton_04.class){
                //抢到锁之后再次进行判断是否为null
                if(instance == null){
                    instance = new Singleton_04();
                }
            }
        }
        return instance;
    }
}
```

**在双重检查锁模式中为什么需要使用 volatile 关键字?**

在java内存模型中，volatile 关键字作用可以是保证可见性或者禁止指令重排。这里是因为 singleton = new Singleton() ，它并非是一个原子操作，事实上，在 JVM 中上述语句至少做了以下这 3 件事：

- 第一步是给 singleton 分配内存空间； 						-- refvar
- 第二步开始调用 Singleton 的构造函数等，来初始化 singleton；                           -- refobj
- 第三步，将 singleton 对象指向分配的内存空间（执行完这步 singleton 就不是 null 了）。     -- refvar -> refobj

这里需要留意一下 1-2-3 的顺序，因为存在指令重排序的优化，也就是说第 2 步和第 3 步的顺序是不能保证的，最终的执行顺序，可能是 1-2-3，也有可能是 1-3-2。

如果是 1-3-2，那么在第 3 步执行完以后，singleton 就不是 null 了，可是这时第 2 步并没有执行，singleton 对象未完成初始化，它的属性的值可能不是我们所预期的值。假设此时线程 2 进入 getInstance 方法，**由于 singleton 已经不是 null 了，所以会通过第一重检查并直接返回，但其实这时的 singleton 并没有完成初始化，所以使用这个实例的时候会报错.**

详细流程如下图所示：

<img src=".\img\35.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

线程 1 首先执行新建实例的第一步，也就是分配单例对象的内存空间，由于线程 1 被重排序，所以执行了新建实例的第三步，也就是把 singleton 指向之前分配出来的内存地址，在这第三步执行之后，singleton 对象便不再是 null。

这时线程 2 进入 getInstance 方法，判断 singleton 对象不是 null，紧接着线程 2 就返回 singleton 对象并使用，由于没有初始化，所以报错了。最后，线程 1 “姗姗来迟”，才开始执行新建实例的第二步——初始化对象，可是这时的初始化已经晚了，因为前面已经报错了。

使用了 volatile 之后，相当于是表明了该字段的更新可能是在其他线程中发生的，因此应确保在读取另一个线程写入的值时，可以顺利执行接下来所需的操作。在 JDK 5 以及后续版本所使用的 JMM 中，在使用了 volatile 后，会一定程度禁止相关语句的重排序，从而避免了上述由于重排序所导致的读取到不完整对象的问题的发生。





### 4.1.6 静态内部类

- 原理
  根据 **静态内部类** 的特性(外部类的加载不影响内部类)，同时解决了按需加载、线程安全的问题，同时实现简洁

> 1. 在静态内部类里创建单例，在装载该内部类时才会去创建单例（==Java规定==，换句话说就是：静态内部类在被使用的时候**（如通过** new Builder() **或调用内部类的静态成员）**才会加载到内存里）
> 2. 线程安全：类是由 `JVM`加载，而`JVM`只会加载1遍，保证只有1个单例

```java
public class Singleton_05 {
    private static class SingletonHandler{
        private static Singleton_05 instance = new Singleton_05();
    }

    private Singleton_05(){ }

    public static Singleton_05 getInstance(){
        return SingletonHandler.instance;
    }
}
```

### 4.1.7 反射对于单例的破坏

> 反射的概念: JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为java语言的反射机制。

反射技术过于强大，它可以通过`setAccessible()`来修改构造器，字段，方法的可见性。单例模式的构造方法是私有的，如果将其可见性设为`public`，那么将无法控制对象的创建。

```java
public class Test_Reflect {
    public static void main(String[] args) {
        try {
            //反射中，欲获取一个类或者调用某个类的方法，首先要获取到该类的Class 对象。
            Class<Singleton_05> clazz = Singleton_05.class;

            //getDeclaredXxx: 不受权限控制的获取类的成员.
            Constructor c = clazz.getDeclaredConstructor(null);

            //设置为true,就可以对类中的私有成员进行操作了
            c.setAccessible(true);

            Object instance1 = c.newInstance();
            Object instance2 = c.newInstance();

            System.out.println(instance1 == instance2);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

解决方法之一: 在单例类的构造方法中 添加判断 `instance != null` 时,直接抛出异常

```java
public class Singleton_05 {
    private static class SingletonHandler{
        private static Singleton_05 instance = new Singleton_05();
    }

    private Singleton_05(){
        if(SingletonHandler.instance != null){
            throw new RuntimeException("不允许非法访问!");
        }
    }

    public static Singleton_05 getInstance(){
        return SingletonHandler.instance;
    }
}
```

上面的这种方式使代码 简洁性 遭到破坏,设计不够优雅.

### 4.1.8 序列化对于单例的破坏

```java
/**
 * 序列化对单例的破坏
 **/
public class Test_Serializable {
    @Test
    public void test() throws Exception{
        //序列化对象输出流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("tempFile.obj"));
        oos.writeObject(Singleton.getInstance());

        //序列化对象输入流
        File file = new File("tempFile.obj");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        Singleton singleton = (Singleton) ois.readObject();

        System.out.println(singleton);
        System.out.println(Singleton.getInstance());

        //判断是否是同一个对象
        System.out.println(Singleton.getInstance() == singleton);//false
    }
}

/**
 * 单例类实现序列化接口
 */
class Singleton implements Serializable {
    private volatile static Singleton singleton;

    private Singleton() { }

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

输出结构为false，说明：

通过对Singleton的序列化与反序列化得到的对象是一个新的对象，这就`破坏了Singleton的单例性`。

解决方案

```java
/**
* 解决方案:只要在Singleton类中定义readResolve就可以解决该问题
* 程序会判断 是否有readResolve方法, 如果存在就在执行该方法, 如果不存在--就创建一个对象
*/
private Object readResolve() {
	return singleton;
}
```

问题是出在ObjectInputputStream 的readObject 方法上, 我们来看一下ObjectInputStream的readObject的调用栈:

<img src=".\img\39.jpg" alt="image-20220530160637842" style="zoom: 50%;" />	

ObjectInputStream中readObject方法的代码片段

```java
try {
    Object obj = readObject0(false); //最终会返回一个object对象,其实就是序列化对象
    return obj;
} finally {
    passHandle = outerHandle;
    if (closed && depth == 0) {
        clear();
    }
}
```

ObjectInputStream中readObject0方法的代码片段

```java
private Object readObject0(boolean unshared) throws IOException {

 case TC_OBJECT: //匹配如果是对象
        return checkResolve(readOrdinaryObject(unshared));
}
```

readOrdinaryObject方法的代码片段

```java
private Object readOrdinaryObject(boolean unshared) throws IOException {
        //此处省略部分代码

        Object obj;
        try {
            //通过反射创建的这个obj对象，就是本方法要返回的对象，也可以暂时理解为是ObjectInputStream的readObject返回的对象。
            //isInstantiable：如果一个serializable的类可以在运行时被实例化，那么该方法就返回true
            //desc.newInstance：该方法通过反射的方式调用无参构造方法新建一个对象。
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }
        return obj;
    }
```

到目前为止，也就可以解释，为什么序列化可以破坏单例了?

答: 序列化会**通过反射调用无参数的构造方法创建一个新的对象**。

我们是如何解决的呢?

答: 只要在Singleton类中定义readResolve就可以解决该问题

```java
//只要在Singleton类中定义readResolve就可以解决该问题
private Object readResolve() {
	return singleton;
}
```

实现原理

```java
if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                handles.setObject(passHandle, obj = rep);
            }
        }
```

`hasReadResolveMethod`:如果实现了serializable 接口的类中包含readResolve则返回true

`invokeReadResolve`:通过反射的方式调用要被反序列化的类的readResolve方法。

**总结: Singleton中定义readResolve方法，并在该方法中指定要返回的对象的生成策略，就可以防止单例被破坏。**

### 4.1.9  枚举(推荐方式)

枚举单例方式是`<<Effective Java>>`作者推荐的使用方式,这种方式

> 在使用枚举时，构造方法会被自动调用，利用这一特性也可以实现单例；默认枚举实例的创建是线程安全的，即使反序列化也不会生成新的实例，任何情况下都是一个单例(暴力反射对枚举方式无效)。

特点: 满足单例模式所需的 **创建单例、线程安全、实现简洁的需求** 

```java
public enum Singleton_06{
    INSTANCE;
    private Object data;
    
    public static Singleton_06 getInstance(){
        return INSTANCE;
    }

    public Object getData() {
        return data;
    }
    public void setData(Object data) {
        this.data = data;
    }
}
```



**问题1: 为什么枚举类可以阻止反射的破坏?**

1. 首先枚举类中是没有空参构造方法的,只有一个带两个参数的构造方法.

   <img src=".\img\37.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

2. 真正原因是: 反射方法中不予许使用反射创建枚举对象

   > 异常: 不能使用反射方式创建enum对象

   <img src=".\img\38.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  

**问题2: 为什么枚举类可以阻止序列化的破坏?**

`Java规范字规定，每个枚举 类型及其定义的枚举变量 在JVM中都是唯一的。`

因此在枚举类型的序列化和反序列化上，Java做了特殊的规定。在序列化的时候Java仅仅是将枚举对象的name属性输到结果中，反序列化的时候则是通过java.lang.Enum的valueOf()方法来根据名字查找枚举对象。

比如说，序列化的时候只将`INSTANCE`这个名称输出，反序列化的时候再通过这个名称，查找对应的枚举类型，因此反序列化后的实例也会和之前被序列化的对象实例相同。

```java
public enum Singleton_06{
    INSTANCE;
}
```

如果不太理解这个解释，看这里：

一个枚举有color枚举变量，他的常量数据red，green等在被序列化的时候，是序列化了这些枚举变量名称，常量并没有被序列化，依旧保存在枚举对象内。等反序列化时，color在去对象内把这些常量red，green等数据找回来。



### 4.2.0 单例模式总结

**1 ) 单例的定义**
	单例设计模式保证某个类在运行期间，只有一个实例对外提供服务，而这个类被称为单例类。

**2 ) 单例的实现**

**饿汉式**

- 饿汉式的实现方式，在类加载的期间，就已经将 instance 静态实例初始化好了，所以，instance 实例的创建是线程安全的。不过，这样的实现方式`不支持延迟加载实例`。

**懒汉式**

- 相对于饿汉式的优势是支持延迟加载。这种实现方式会导致频繁加锁、释放锁，以及并发度低等问题，频繁的调用会产生性能瓶颈。

**双重检测**

- 双重检测实现方式既支持延迟加载、又支持高并发的单例实现方式。只要 instance 被创建之后，再调用 getInstance() 函数都不会进入到加锁逻辑中。所以，这种实现方式解决了懒汉式并发度低的问题。

**静态内部类**

- 利用 Java 的静态内部类来实现单例。这种实现方式，既支持延迟加载，也支持高并发，实现起来也比双重检测简单。

**枚举方式**

- 最简单的实现方式，基于枚举类型的单例实现。这种实现方式通过 Java 枚举类型本身的特性，保证了实例创建的线程安全性和实例的唯一性(同时阻止了反射和序列化对单例的破坏)。



---

## 4.2 工厂方法模式

工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

- **在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。**

《设计模式》一书中，工厂模式被分为了三种：简单工厂、工厂方法和抽象工厂。（不过，在书中作者将简单工厂模式看作是工厂方法模式的一种特例。）

接下来我会介绍三种工厂模式的原理及使用

* 简单工厂模式（不属于GOF的23种经典设计模式）
* 工厂方法模式
* 抽象工厂模式

### 4.2.1 需求: 模拟发放奖品业务

需求: 为了让我们的案例更加贴近实际开发, 这里我们来模拟一下互联网电商中促销拉新下的业务场景, **新用户注册立即参与抽奖活动** ,奖品的种类有: 打折券, 免费优酷会员,小礼品.

<img src=".\img\42.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 



### 4.2.2 原始开发方式

**不考虑设计原则,不使用设计模式的方式进行开发** 

在不考虑任何代码的可扩展性的前提下,只为了尽快满足需求.我们可以这样去设计这个业务的代码结构:

**1) 实体类**

| 名称           | 描述                     |
| -------------- | :----------------------- |
| AwardInfo      | 获奖信息对应实体类       |
| YouKuMember    | 优酷会员对应实体类       |
| SmallGiftInfo  | 小礼品信息对应实体类     |
| DiscountInfo   | 打折券信息对应实体类     |
| DiscountResult | 打折券操作响应结果封装类 |

```java
public class AwardInfo {
    private String 	uid; 			//用户唯一ID
    private Integer awardType;  	//奖品类型: 1 打折券 ,2 优酷会员,3 小礼品
    private String 	awardNumber; 	//奖品编号
    Map<String, String> extMap; 	//额外信息
}

public class DiscountInfo {
    //属性信息省略......
}

public class YouKuMember {
    //属性信息省略......
}

public class SmallGiftInfo {
    private String userName;              // 用户姓名
    private String userPhone;             // 用户手机
    private String orderId;               // 订单ID
    private String relAddress;            // 收货地址
}

public class DiscountResult {
    private String status; // 状态码
    private String message; // 信息
}
```

**2) 服务层**

| 名称               | 功能                                                  | 描述                 |
| ------------------ | ----------------------------------------------------- | -------------------- |
| DiscountService    | DiscountResult sendDiscount(String uid,String number) | 模拟打折券服务       |
| YouKuMemberService | void openMember(String bindMobile , String number)    | 模拟赠送优酷会员服务 |
| SmallGiftService   | Boolean giveSmallGift(SmallGiftInfo smallGiftInfo)    | 模拟礼品服务         |

```java
public class DiscountService {
    public DiscountResult sendDiscount(String uid, String number){
        System.out.println("向用户发放打折券一张: " + uid + " , " + number);
        return new DiscountResult("200","发放打折券成功");
    }
}

public class YouKuMemberService {
    public void openMember(String bindMobile , String number){
        System.out.println("发放优酷会员: " + bindMobile + " , " + number);
    }
}

public class SmallGiftService {
    public Boolean giveSmallGift(SmallGiftInfo smallGiftInfo){
        System.out.println("小礼品已发货,获奖用户注意查收! " + JSON.toJSON(smallGiftInfo));
        return true;
    }
}
```

**3) 控制层**

| 名称              | 功能                                            | 描述                                                         |
| ----------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| DeliverController | ResponseResult awardToUser(AwardInfo awardInfo) | 按照类型的不同发放商品<br/>奖品类型: 1 打折券 ,2 优酷会员,3 小礼品 |

```java
public class DeliverController {
    /**
     * 按照类型的不同发放商品
     * 奖品类型: 1 打折券 ,2 优酷会员,3 小礼品
     */
    public void awardToUser(AwardInfo awardInfo){
        if(awardInfo.getAwardType() == 1){ 				//打折券
            DiscountService discountService = new DiscountService();
            DiscountResult result = discountService.sendDiscount(awardInfo.getUid(), awardInfo.getAwardNumber());
            System.out.println("打折券发放成功!"+ JSON.toJSON(result));
        }else if(awardInfo.getAwardType() == 2){ 		//优酷会员
            //获取用户手机号
            String bindMobile = awardInfo.getExtMap().get("phone");
            //调用service
            YouKuMemberService youKuMemberService = new YouKuMemberService();
            youKuMemberService.openMember(bindMobile,awardInfo.getAwardNumber());
            System.out.println("优酷会员发放成功!");
        }else if(awardInfo.getAwardType() == 3){		// 小礼品 封装收货用户信息
            SmallGiftInfo smallGiftInfo = new SmallGiftInfo();
            smallGiftInfo.setUserName(awardInfo.getExtMap().get("username"));
            smallGiftInfo.setOrderId(UUID.randomUUID().toString());
            smallGiftInfo.setRelAddress(awardInfo.getExtMap().get("adderss"));

            SmallGiftService smallGiftService = new SmallGiftService();
            Boolean isSuccess = smallGiftService.giveSmallGift(smallGiftInfo);
            System.out.println("小礼品发放成功!" + isSuccess);
        }
    }
}
```

**4) 测试**

通过单元测试,来对上面的接口进行测试,验证代码质量.

```java
public class TestApi01 {
    //测试发放奖品接口
    @Test
    public void test01(){
        DeliverController deliverController = new DeliverController();

        //1. 发放打折券优惠
        AwardInfo info1 = new AwardInfo();
        info1.setUid("1001");
        info1.setAwardType(1);
        info1.setAwardNumber("DEL12345");
        deliverController.awardToUser(info1);

        //2. 发放优酷会员
        AwardInfo info2 = new AwardInfo();
        info2.setUid("1002");
        info2.setAwardType(2);
        info2.setAwardNumber("DW12345");
        Map<String,String> map = new HashMap<>();
        map.put("phone","13512341234");
        info2.setExtMap(map);
        deliverController.awardToUser(info2);

        //2. 发放小礼品
        AwardInfo info3 = new AwardInfo();
        info3.setUid("1003");
        info3.setAwardType(3);
        info3.setAwardNumber("SM12345");
        Map<String,String> map2 = new HashMap<>();
        map2.put("username","大远");
        map2.put("phone","13512341234");
        map2.put("address","北京天安门");
        info3.setExtMap(map2);
        deliverController.awardToUser(info3);
    }
}
```

对于上面的实现方式,如果我们有想要添加的新的奖品时,势必要改动DeliverController的代码,违反开闭原则.而且如果有的抽奖接口出现问题,那么对其进行重构的成本会非常高.

除此之外代码中有一组if分支判断逻辑,现在看起来还可以,但是如果经历几次迭代和拓展,后续ifelse肯定还会增加.到时候接手这段代码的研发将会十分痛苦.



### 4.2.3 简单工厂模式

#### 4.2.3.1 简单工厂模式介绍

简单工厂不是一种设计模式，反而比较像是一种编程习惯。简单工厂模式又叫做静态工厂方法模式（static Factory Method pattern）,它是通过使用静态方法接收不同的参数来返回不同的实例对象.

**实现方式:**  定义**一个**工厂类，根据传入的参数不同返回不同的实例，被创建的实例具有共同的父类或接口。

**适用场景：**
　　（1）需要创建的对象较少。
　　（2）客户端不关心对象的创建过程。

#### 4.2.3.2 简单工厂原理

简单工厂包含如下角色：

* 抽象产品 ：定义了产品的规范，描述了产品的主要特性和功能。
* 具体产品 ：实现或者继承抽象产品的子类
* 具体工厂 ：提供了创建产品的方法，调用者通过该方法来获取产品。

<img src=".\img\43.jpg" alt="image-20220530160637842" style="zoom: 100%;" />    

#### 4.2.3.3 简单工厂模式重构代码

**1) service** 

```java
/**
 * 免费商品发放接口
 **/
public interface IFreeGoods {
    ResponseResult sendFreeGoods(AwardInfo awardInfo);
}
```

```java
/**
 * 模拟打折券服务
 **/
public class DiscountFreeGoods implements IFreeGoods {
    @Override
    public ResponseResult sendFreeGoods(AwardInfo awardInfo) {
        System.out.println("向用户发放一张打折券: " + awardInfo.getUid() + " , " + awardInfo.getAwardNumber());
        return new ResponseResult("200","打折券发放成功!");
    }
}

/**
 * 小礼品发放服务
 **/
public class SmallGiftFreeGoods implements IFreeGoods {
    @Override
    public ResponseResult sendFreeGoods(AwardInfo awardInfo) {
        SmallGiftInfo smallGiftInfo = new SmallGiftInfo();
        smallGiftInfo.setUserPhone(awardInfo.getExtMap().get("phone"));
        smallGiftInfo.setUserName(awardInfo.getExtMap().get("username"));
        smallGiftInfo.setAddress(awardInfo.getExtMap().get("address"));
        smallGiftInfo.setOrderId(UUID.randomUUID().toString());
        
        System.out.println("小礼品发放成,请注意查收: " + JSON.toJSON(smallGiftInfo));
        return new ResponseResult("200","小礼品发送成功",smallGiftInfo);
    }
}

/**
 * 优酷 会员服务
 **/
public class YouKuMemberFreeGoods implements IFreeGoods {
    @Override
    public ResponseResult sendFreeGoods(AwardInfo awardInfo) {
        String phone = awardInfo.getExtMap().get("phone");
        System.out.println("发放优酷会员成功,绑定手机号: " + phone);
        return new ResponseResult("200","优酷会员发放成功!");
    }
}
```

**2) factory** 

```java
/**
 * 具体工厂: 生成免费商品
 **/
public class FreeGoodsFactory {
    public static IFreeGoods getInstance(Integer awardType){
        IFreeGoods iFreeGoods = null;
        if(awardType == 1){  		// 打折券
            iFreeGoods = new DiscountFreeGoods();
        }else if(awardType == 2){ 	// 优酷会员
            iFreeGoods = new YouKuMemberFreeGoods();
        }else if(awardType == 3){ 	// 小礼品
            iFreeGoods = new SmallGiftFreeGoods();
        }
        return iFreeGoods;
    }
}
```

**3）controller**

```java
public class DeliverController {
    //发放奖品
    public ResponseResult awardToUser(AwardInfo awardInfo){
        try {
            IFreeGoods freeGoods = FreeGoodsFactory.getInstance(awardInfo.getAwardTypes());
            ResponseResult responseResult = freeGoods.sendFreeGoods(awardInfo);
            return responseResult;
        } catch (Exception e) {
            e.printStackTrace();
            return new ResponseResult("201","奖品发放失败!");
        }
    }
}
```

**4) 测试** 

通过单元测试,来对上面的接口进行测试,验证代码质量.

```java
public class TestApi02 {
    DeliverController deliverController = new DeliverController();
    @Test
    public void test01(){
        //1. 发放打折券优惠
        AwardInfo info1 = new AwardInfo();
        info1.setUid("1001");
        info1.setAwardTypes(1);
        info1.setAwardNumber("DEL12345");

        ResponseResult result = deliverController.awardToUser(info1);
        System.out.println(result);
    }

    @Test
    public void test02(){
        //2. 发放优酷会员
        AwardInfo info2 = new AwardInfo();
        info2.setUid("1002");
        info2.setAwardTypes(2);
        info2.setAwardNumber("DW12345");
        Map<String,String> map = new HashMap<>();
        map.put("phone","13512341234");
        info2.setExtMap(map);

        ResponseResult result1 = deliverController.awardToUser(info2);
        System.out.println(result1);
    }

    @Test
    public void test03(){
        //3. 发放小礼品
        AwardInfo info3 = new AwardInfo();
        info3.setUid("1003");
        info3.setAwardTypes(3);
        info3.setAwardNumber("SM12345");
        Map<String,String> map2 = new HashMap<>();
        map2.put("username","大远");
        map2.put("phone","13512341234");
        map2.put("address","北京天安门");
        info3.setExtMap(map2);

        ResponseResult result2 = deliverController.awardToUser(info3);
        System.out.println(result2);
    }
}
```

#### 4.2.3.4 简单工厂模式总结

**优点：**

封装了创建对象的过程，可以通过参数直接获取对象。把对象的创建和业务逻辑层分开，这样以后就避免了修改客户代码，如果要实现新产品直接修改工厂类，而不需要在原代码中修改，这样就降低了客户代码修改的可能性，更加容易扩展。

**缺点：**

增加新产品时还是需要修改工厂类的代码，违背了“开闭原则”。



### 4.2.4 工厂方法模式

#### 4.2.4.1 工厂方法模式介绍

工厂方法模式 `Factory Method pattern`,属于创建型模式. 

概念: 定义一个用于创建对象的接口，让子类决定实例化哪个产品类对象。工厂方法使一个产品类的实例化延迟到其工厂的子类。

#### 4.2.4.2 工厂方法模式原理

工厂方法模式的目的很简单，就是**封装对象创建的过程，提升创建对象方法的可复用性**。

工厂方法模式的主要角色：

* 抽象工厂：提供了创建产品的接口，调用者通过它访问具体工厂的工厂方法来创建产品。
* 具体工厂：主要是实现抽象工厂中的抽象方法，完成具体产品的创建。
* 抽象产品：定义了产品的规范，描述了产品的主要特性和功能。
* 具体产品：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间一一对应。

我们直接来看看工厂方法模式的 UML 图：

<img src=".\img\44.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

####  4.2.4.3 工厂方法模式重构代码

为了提高代码扩展性,我们需要将简单工厂中的if分支逻辑去掉,通过增加抽象工厂(**生产工厂的工厂**)的方式,让具体工厂去进行实现,由具体工厂来决定实例化哪一个具体的产品对象。

**抽象工厂**

```java
public interface FreeGoodsFactory {
    IFreeGoods getInstance();
}
```

**具体工厂**

```java
public class DiscountFreeGoodsFactory implements FreeGoodsFactory {
    @Override
    public IFreeGoods getInstance() {
        return new DiscountFreeGoods();
    }
}

public class SmallGiftFreeGoodsFactory implements FreeGoodsFactory {
    @Override
    public IFreeGoods getInstance() {
        return new SmallGiftFreeGoods();
    }
}
```

**Controller**

```java
public class DeliverController {
    // 按照类型的不同发放商品
    public ResponseResult awardToUser(AwardInfo awardInfo){
        FreeGoodsFactory freeGoodsFactory = null;
        if(awardInfo.getAwardType() == 1){
            freeGoodsFactory = new DiscountFreeGoodsFactory();
        }else if(awardInfo.getAwardType() == 2){
            freeGoodsFactory = new SmallGiftFreeGoodsFactory();
        }
        IFreeGoods freeGoods = freeGoodsFactory.getInstance();
        
        System.out.println("=====工厂方法模式========");
        ResponseResult result = freeGoods.sendFreeGoods(awardInfo);
        return result;
    }
}
```

从上面的代码实现来看，工厂类对象的创建逻辑又耦合进了 awardToUser() 方法中，跟我们**最初的代码版本非常相似**,引入工厂方法非但没有解决问题，反倒让设计变得更加复杂了。

那怎么 来解决这个问题呢？

我们可以为工厂类再创建一个简单工厂，也就是工厂的工厂，用来创建工厂类对象。

```java
/**
 * 用简单方法模式实现: 工厂的工厂,作用是不需要每次创建新的工厂对象
 **/
public class FreeGoodsFactoryMap {
    private static final Map<Integer,FreeGoodsFactory> cachedFactories = new HashMap<>();

    static{
        cachedFactories.put(1, new DiscountFreeGoodsFactory());
        cachedFactories.put(2, new SmallGiftFreeGoodsFactory());
    }

    public static FreeGoodsFactory getParserFactory(Integer type){
        if(type == 1){
            FreeGoodsFactory freeGoodsFactory = cachedFactories.get(1);
            return freeGoodsFactory;
        }else if(type ==2){
            FreeGoodsFactory freeGoodsFactory = cachedFactories.get(2);
            return freeGoodsFactory;
        }
        return null;
    }
}
```

**Controller**

```java
/**
 * 发放奖品接口
 **/
public class DeliverController {
    // 按照类型的不同发放商品
    public ResponseResult awardToUser(AwardInfo awardInfo){
        //根据类型获取工厂
        FreeGoodsFactory goodsFactory = FreeGoodsFactoryMap.getParserFactory(awardInfo.getAwardType());
        //从工厂中获取对应实例
        IFreeGoods freeGoods = goodsFactory.getInstance();

        System.out.println("=====工厂方法模式========");
        ResponseResult result = freeGoods.sendFreeGoods(awardInfo);
        return result;
    }
}
```

现在我们的代码已经**基本上符合**了开闭原则,当有新增的产品时,我们需要做的事情包括:

1. 创建新的产品类,并且让该产品实现抽象产品接口
2. 创建产品类对应的具体工厂（自家工厂建自家商品）,并让具体工厂实现抽象工厂
3. 将新的具体工厂对象,添加到 FreeGoodsFactoryMap 的 cachedFactories 中即可,**需要改动的代码改动的非常少**.



简单工厂是，工厂管理所有产品

工厂模式是，一个工厂只生产一个产品

后面的优化其实与上面的偏差就是 新增原本要改动业务代码，变成改动工厂代码。



#### 4.2.4.4 工厂方法模式总结

**工厂方法模优缺点**

**优点：**

- 用户只需要知道具体工厂的名称就可得到所要的产品，无须知道产品的具体创建过程；
- 在系统增加新的产品时只需要添加具体产品类和对应的具体工厂类，无须对原工厂进行任何修改，满足开闭原则；

**缺点：**

* 每增加一个产品就要增加一个具体产品类和一个对应的具体工厂类，这增加了系统的复杂度。

**什么时候使用工厂方法模式**

- 需要使用很多重复代码创建对象时，比如，DAO 层的数据对象、API 层的 VO 对象等。
- 创建对象要访问外部信息或资源时，比如，读取数据库字段，获取访问授权 token 信息，配置文件等。
- 创建需要统一管理生命周期的对象时，比如，会话信息、用户网页浏览轨迹对象等。
- 创建池化对象时，比如，连接池对象、线程池对象、日志对象等。这些对象的特性是：有限、可重用，使用工厂方法模式可以有效节约资源。
- 希望隐藏对象的真实类型时，比如，不希望使用者知道对象的真实构造函数参数等。

## 4.3 抽象工厂模式

### 4.3.1 抽象工厂模式介绍

> 抽象工厂模式比工厂方法模式的**抽象程度更高。在工厂方法模式中每一个具体工厂只需要生产一种具体产品，但是**在抽象工厂模式中一个具体工厂可以生产一组相关的具体产品**，这样一组产品被称为产品族。产品族中的每一个产品都分属于某一个产品继承等级结构。

**1) 产品等级结构与产品族** 

为了更好的理解抽象工厂, 我们这里先引入两个概念:

- **产品等级结构** ：产品等级结构即产品的继承结构，如一个抽象类是电视机，其子类有海尔电视机、海信电视机、TCL电视机，则抽象电视机与具体品牌的电视机之间构成了一个产品等级结构，抽象电视机是父类，而具体品牌的电视机是其子类。

- **产品族** ：在抽象工厂模式中，产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品，如海尔电器工厂生产的海尔电视机、海尔电冰箱，海尔电视机位于电视机产品等级结构中，海尔电冰箱位于电冰箱产品等级结构中。

<img src=".\img\46.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

> 在上图中,每一个具体工厂可以生产属于一个产品族的所有产品,例如海尔工厂生产海尔电视机、海尔空调和海尔冰箱,所生产的产品又位于不同的产品等级结构中. 如果使用工厂方法模式,上图所示的结构需要提供9个具体工厂,而使用抽象工厂模式只需要提供3个具体工厂,极大减少了系统中类的个数.

**2) 抽象工厂模式概述**

抽象工厂模式(Abstract Factory Pattern)  原始定义：**提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。**

抽象工厂模式为创建一组对象提供了解决方案.与工厂方法模式相比,**抽象工厂模式中的具体工厂不只是创建一种产品,而是负责创建一个产品族**.如下图:

<img src=".\img\47.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

### 4.3.2 抽象工厂模式原理

在抽象工厂模式中,每一个具体工厂都提供了多个工厂方法,用于产生多种不同类型的产品.这些产品构成了一个产品族.

<img src=".\img\49.jpg" alt="image-20220530160637842" style="zoom: 100%;" />  

抽象工厂模式的主要角色如下：

* 抽象工厂（Abstract Factory）：它声明了一种用于创建一族产品的方法,每一个方法对应一种产品.
* 具体工厂（Concrete Factory）：主要是实现抽象工厂中的多个抽象方法，完成具体产品的创建.
* 抽象产品（Product）：定义了产品的规范，描述了产品的主要特性和功能，抽象工厂模式有多个抽象产品。
* 具体产品（ConcreteProduct）：实现了抽象产品角色所定义的接口，由具体工厂来创建，它 同具体工厂之间是多对一的关系。

### 4.3.3 抽象工厂模式实现

抽象工厂

```java
// 抽象工厂: 在一个抽象工厂中可以声明多个工厂方法,用于创建不同类型的产品
public interface AppliancesFactory {
    AbstractTV createTV();
    AbstractFreezer createFreezer();
}
```

具体工厂: 每一个具体工厂方法,可以返回一个特定的产品对象,而同一个具体工厂所创建的产品对象构成了一个产品族.

```java
public class HairFactory implements AppliancesFactory {
    @Override
    public AbstractTV createTV() {
        return new HairTV();
    }
    @Override
    public AbstractFreezer createFreezer() {
        return new HairFreezer();
    }
}

public class HisenseFactory implements AppliancesFactory {
    @Override
    public AbstractTV createTV() {
        return new HisenseTV();
    }
    @Override
    public AbstractFreezer createFreezer() {
        return new HisenseFreezer();
    }
}
```

抽象产品

```java
public interface AbstractFreezer {}
public interface AbstractTV {}
```

具体产品

```java
public class HairFreezer implements AbstractFreezer {}
public class HisenseFreezer implements AbstractFreezer {}
public class HairTV implements AbstractTV {}
public class HisenseTV implements AbstractTV {}
```

客户端

```java
public class Client {
    private AbstractTV tv;
    private AbstractFreezer freezer;
    
    public Client(AppliancesFactory factory){
        //在客户端看来就是使用抽象工厂来生产家电
        this.tv = factory.createTV();
        this.freezer = factory.createFreezer();
    }

    // getter setter
    ...

    public static void main(String[] args) {
        Client client = new Client(new HisenseFactory());
        AbstractTV tv = client.getTv();
        AbstractFreezer freezer = client.getFreezer();
        System.out.println(tv);
        System.out.println(freezer);
    }
}
```

### 4.3.4 抽象工厂模式总结

从上面代码实现中我们可以看出，抽象工厂模式向使用（客户）方隐藏了下列变化：

- 程序所支持的实例集合（具体工厂）的数目；
- 当前是使用的实例集合中的哪一个实例；
- 在任意给定时刻被实例化的具体类型；

所以说，在理解抽象工厂模式原理时，你一定要牢牢记住“如何找到某一个类产品的正确共性功能”这个重点。

**抽象工厂模式优点**

1) 对于不同产品系列有比较多共性特征时，可以使用抽象工厂模式，有助于提升组件的复用性.

2) 当需要提升代码的扩展性并降低维护成本时，把对象的创建和使用过程分开，能有效地将代码统一到一个级别上


3. 解决跨平台带来的兼容性问题


**抽象工厂模式缺点**

   **增加新的产品等级结构麻烦,需要对原有结构进行较大的修改,**甚至需要修改抽象层代码,这显然会带来较大不变,违背了开闭原则.





## 4.4 建造者模式

### 4.4.1 建造者模式介绍

**建造者模式** (builder pattern), 也被称为**生成器模式** , 是一种创建型设计模式.

- 定义: 将一个复杂对象的构建与表示分离，使得同样的构建过程可以创建不同的表示。

**建造者模式要解决的问题 **

建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。

> 比如: 一辆汽车是由多个部件组成的,包括了车轮、方向盘、发动机等等.对于大多数用户而言,并不需要知道这些部件的装配细节,并且几乎不会使用单独某个部件,而是使用一辆完整的汽车.而建造者模式就是负责将这些部件进行组装让后将完整的汽车返回给用户.

​													<img src=".\img\50.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 



### 4.4.2 建造者模式原理

建造者（Builder）模式包含以下4个角色 :

* 抽象建造者类（Builder）：这个接口规定要实现复杂对象的哪些部分的创建，并不涉及具体的部件对象的创建。 

* 具体建造者类（ConcreteBuilder）：实现 Builder 接口，完成复杂产品的各个部件的具体创建方法。在构造过程完成后，提供一个方法,返回创建好的负责产品对象。 

* 产品类（Product）：要创建的复杂对象 (包含多个组成部件).

* 指挥者类（Director）：调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建(客户端一般只需要与指挥者进行交互)。 

<img src=".\img\52.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  

### 4.4.3 建造者模式实现方式1

**创建共享单车**

生产自行车是一个复杂的过程，它包含了车架，车座等组件的生产。而车架又有碳纤维，铝合金等材质的，车座有橡胶，真皮等材质。对于自行车的生产就可以使用建造者模式。

这里Bike是产品，包含车架，车座等组件；Builder是抽象建造者，MobikeBuilder和HelloBuilder是具体的建造者；Director是指挥者。类图如下：

<img src=".\img\53.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

具体产品

```java
public class Bike {
    // 车架
    private String frame;
    // 座椅
    private String seat;
	// getter setter
    ...
}
```

构建者类

```java
public abstract class Builder {
    protected Bike mBike = new Bike();

    public abstract Bike createBike();
    public abstract void buildFrame();
    public abstract void buildSeat();
}

public class HelloBuilder extends Builder {
    @Override
    public void buildFrame() {
        mBike.setFrame("碳纤维车架");
    }
    @Override
    public void buildSeat() {
        mBike.setSeat("橡胶车座");
    }
    @Override
    public Bike createBike() {
        return mBike;
    }
}

public class MobikeBuilder extends Builder {
    @Override
    public void buildFrame() {
        mBike.setFrame("铝合金车架");
    }
    @Override
    public void buildSeat() {
        mBike.setSeat("真皮车座");
    }
    @Override
    public Bike createBike() {
        return mBike;
    }
}
```

指挥者类

```java
public class Director {
    private Builder mBuilder;

    public Director(Builder builder) {
        this.mBuilder = builder;
    }

    public Bike construct() {
        mBuilder.buildFrame();
        mBuilder.buildSeat();
        return mBuilder.createBike();
    }
}
```

客户端

```java
public class Client {
    public static void main(String[] args) {
        showBike(new HelloBuilder());
        showBike(new MobikeBuilder());
    }

    private static void showBike(Builder builder) {
        Director director = new Director(builder);
        Bike bike = director.construct();
        System.out.println(bike.getFrame());
        System.out.println(bike.getSeat());
    }
}
```

### 4.4.4 建造者模式实现方式2

建造者模式除了上面的用途外，在开发中还有一个常用的使用方式，就是当一个类构造器需要传入很多参数时，如果创建这个类的实例，代码可读性会非常差，而且很容易引入错误，此时就可以利用建造者模式进行重构。

**1 ) 构造方法创建复杂对象的问题**

- 构造方法如果参数过多,代码的可读性和易用性都会变差. 在使用构造函数时,很容易搞错参数的顺序,传递进去错误的参数值,导致很有隐蔽的BUG出现.

```java
package com.mashibing.example02;

/**
 * MQ连接客户端
 **/
public class RabbitMQClient1 {
    private String host = "127.0.0.1";
    private int port = 5672;
    private int mode;
    private String exchange;
    private String queue;
    private boolean isDurable = true;
    int connectionTimeout = 1000;
    
    //构造方法参数过多,代码的可读性和易用性太差,在使用构造函数时,很容易搞错顺序,传递错误的参数值,导致很有隐蔽的BUG
    public RabbitMQClient1(String host, int port, int mode, String exchange, String queue, boolean isDurable, int connectionTimeout) {
        this.host = host;
        this.port = port;
        this.mode = mode;
        this.exchange = exchange;
        this.queue = queue;
        this.isDurable = isDurable;
        this.connectionTimeout = connectionTimeout;

        if(mode == 1){ //工作队列模式不需要设计交换机,但是队列名称一定要有
            if(exchange != null){
                throw new RuntimeException("工作队列模式无需设计交换机");
            }
            if(queue == null || queue.trim().equals("")){
                throw new RuntimeException("工作队列模式名称不能为空");
            }
            if(isDurable == false){
                throw new RuntimeException("工作队列模式必须开启持久化");
            }
        }else if(mode == 2){ //路由模式必须设计交换机,但是不能设计队列
            if(exchange == null){
                throw new RuntimeException("路由模式下必须设置交换机");
            }
            if(queue != null){
                throw new RuntimeException("路由模式无须设计队列名称");
            }
        }
        //其他验证方式,
    }

    public void sendMessage(String msg){
        System.out.println("发送消息......");
    }

    public static void main(String[] args) {
        //每一种模式,都需要根据不同的情况进行实例化,构造方法会变得过于复杂.
        RabbitMQClient1 client1 = new RabbitMQClient1("192.168.52.123",5672,
                2,"sample-exchange",null,true,5000);

        client1.sendMessage("Test-MSG");
    }
}
```

**2) set方法创建复杂对象的问题** 

- set方式设置对象属性时,存在中间状态,并且属性校验时有前后顺序约束,逻辑校验的代码找不到合适的地方放置.

  > 比如下面的代码,  创建对象后使用set 的方式，那就会导致在第一个 set 之后，对象处于无效状态
  >
  > Rectangle r = new Rectangle ();  //无效状态
  >
  > r.setWidth(2);  //无效状态
  >
  > r.setHeight(3);  //有效状态

- set方法还破坏了"不可变对象"的密闭性 .

  > 不可变对象:  对象创建好了,就不能再修改内部的属性值,下面的client类就是典型的不可变对象,创建好的连接对象不能再改动

```java
/**
 * MQ连接客户端
 **/
public class RabbitMQClient2 {
    private String host = "127.0.0.1";
    private int port = 5672;
    private int mode;
    private String exchange;
    private String queue;
    private boolean isDurable = true;
    int connectionTimeout = 1000;

    //私有化构造方法
    private RabbitMQClient2() { }

    public void setExchange(String exchange) {
        if(mode == 1){ //工作队列模式不需要设计交换机,但是队列名称一定要有
            if(exchange != null){
                throw new RuntimeException("工作队列模式无需设计交换机");
            }
            if(queue == null || queue.trim().equals("")){
                throw new RuntimeException("工作队列模式名称不能为空");
            }
            if(isDurable == false){
                throw new RuntimeException("工作队列模式必须开启持久化");
            }
        }else if(mode == 2){ //路由模式必须设计交换机,但是不能设计队列
            if(exchange == null){
                throw new RuntimeException("路由模式下必须设置交换机");
            }
            if(queue != null){
                throw new RuntimeException("路由模式无须设计队列名称");
            }
        }
        //其他验证方式,
        this.exchange = exchange;
    }

    public void setMode(int mode) {
        if(mode == 1){ //工作队列模式不需要设计交换机,但是队列名称一定要有
            if(exchange != null){
                throw new RuntimeException("工作队列模式无需设计交换机");
            }
            if(queue == null || queue.trim().equals("")){
                throw new RuntimeException("工作队列模式名称不能为空");
            }
            if(isDurable == false){
                throw new RuntimeException("工作队列模式必须开启持久化");
            }
        }else if(mode == 2){ //路由模式必须设计交换机,但是不能设计队列
            if(exchange == null){
                throw new RuntimeException("路由模式下必须设置交换机");
            }
            if(queue != null){
                throw new RuntimeException("路由模式无须设计队列名称");
            }
        }
        this.mode = mode;
    }
    
    // 其余 getter setter
    ...

    public void sendMessage(String msg){
        System.out.println("发送消息......");
    }

    /**
     * set方法的好处是参数的设计更加的灵活,但是通过set方式设置对象属性时,对象有可能存在中间状态(无效状态),
     * 并且进行属性校验时有前后顺序约束.
     * 怎么保证灵活设置参数又不会存在中间状态呢? 答案就是: 使用建造者模式
     */
    public static void main(String[] args) {
        RabbitMQClient2 client2 = new RabbitMQClient2();
        client2.setHost("192.168.52.123");
        client2.setQueue("queue");
        client2.setMode(1);
        client2.setDurable(true);
        client2.sendMessage("Test-MSG2");
    }
}
```



**3) 建造者方式实现**

建造者使用步骤如下:

1. 目标类的构造方法要传入Builder对象
2. Builder建造者类位于目标类内部,并且使用static修饰
3. Builder建造者对象提供内置的各种set方法,注意set方法返回的是builder对象本身
4. Builder建造者类提供build()方法实现目标对象的创建

```java
public class 目标类{
    //目标类的构造方法需要传入Builder对象
    public 目标类(Builder builder){ }

    public 返回值 业务方法(参数列表){ }
    
    //Builder建造者类位于目标类内部,并且使用static修饰
    public static class Builder(){
        //Builder建造者对象提供内置的各种set方法,注意set方法返回的是builder对象本身
        private String xxx;
        public Builder setXxx(String xxx){
            this.xxx = xxx;
            return this;
        }
        
        //Builder建造者类提供build()方法实现目标对象的创建
        public 目标类 build(){
            //校验
            return new 目标类(this);
        }
    }
}
```

**重写案例代码**

```java
/**
 * 建造者模式
 **/
public class RabbitMQClient {
    //私有构造方法
    private RabbitMQClient(Builder builder) { }

    public static class Builder{
        //属性密闭性,保证对象不可变
        private String host = "127.0.0.1";
        private int port = 5672;
        private int mode;
        private String exchange;
        private String queue;
        private boolean isDurable = true;
        int connectionTimeout = 1000;

        public Builder setHost(String host) {
            this.host = host;
            return this;
        }
        public Builder setPort(int port) {
            this.port = port;
            return this;
        }
        public Builder setMode(int mode) {
            this.mode = mode;
            return this;
        }
        public Builder setExchange(String exchange) {
            this.exchange = exchange;
            return this;
        }
        public Builder setQueue(String queue) {
            this.queue = queue;
            return this;
        }
        public Builder setDurable(boolean durable) {
            isDurable = durable;
            return this;
        }
        public Builder setConnectionTimeout(int connectionTimeout) {
            this.connectionTimeout = connectionTimeout;
            return this;
        }


        //返回构建好的复杂对象
        public RabbitMQClient build(){
            //首先进行校验
            if(mode == 1){ //工作队列模式不需要设计交换机,但是队列名称一定要有
                if(exchange != null){
                    throw new RuntimeException("工作队列模式无需设计交换机");
                }
                if(queue == null || queue.trim().equals("")){
                    throw new RuntimeException("工作队列模式名称不能为空");
                }
                if(isDurable == false){
                    throw new RuntimeException("工作队列模式必须开启持久化");
                }
            }else if(mode == 2){ //路由模式必须设计交换机,但是不能设计队列
                if(exchange == null){
                    throw new RuntimeException("路由模式下必须设置交换机");
                }
                if(queue != null){
                    throw new RuntimeException("路由模式无须设计队列名称");
                }
            }
            return new RabbitMQClient(this);
        }	
    }

    public void sendMessage(String msg){
        System.out.println("发送消息......");
    }
}
```

其中有几个知识点：

1. 外部类没有对静态内部类的构造，所以每次在想调用静态内部类的时候，就需要new

   ```java
   RabbitMQClient client = new RabbitMQClient.Builder().setxxxx().build();
   // 除非外部类有个静态的内部类构造，但就破坏了
   public static Bilder getBuilder(){
       return new Builder();
   }
   ```

2. 创建出来的cilent对象无法修改参数，没办法访问到内部的参数。因为外部类与内部类是隔离的，除内部类有对内的getter。

测试

```java
public class MainAPP {
    public static void main(String[] args) {
        //使用链式编程设置参数
        RabbitMQClient client = new RabbitMQClient.Builder()
            .setHost("192.168.52.123")
            .setMode(2)
            .setExchange("text-exchange")
            .setPort(5672)
            .setDurable(true).build();

        client.sendMessage("Test");
    }
}
```

先设置一个内部类，然后对这个内部类进行赋值，

### 4.4.5 建造者模式总结

**1) 建造者模式与工厂模式区别** 

- 工厂模式是用来创建不同但是相关类型的对象（继承同一父类或者接口的一组子类），由给定的参数来决定创建哪种类型的对象。
- 建造者模式是用来创建一种类型的复杂对象，通过设置不同的可选参数，“定制化”地创建不同的对象。

> 举例: 顾客走进一家餐馆点餐，我们利用工厂模式，根据用户不同的选择，来制作不同的食物，比如披萨、汉堡、沙拉。
>
> 对于披萨来说，用户又有各种配料可以定制，比如奶酪、西红柿、起司，我们通过建造者模式根据用户选择的不同配料来制作披萨。



**2) 建造者模式的优缺点**

- 优点

  - 建造者模式的封装性很好。使用建造者模式可以有效的封装变化，在使用建造者模式的场景中，一般产品类和建造者类是比较稳定的，因此，将主要的业务逻辑封装在指挥者类中对整体而言可以取得比较好的稳定性。
  - 在建造者模式中，客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。
  - 可以更加精细地控制产品的创建过程 。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。
  - 建造者模式很容易进行扩展。如果有新的需求，通过实现一个新的建造者类就可以完成，基本上不用修改之前已经测试通过的代码，因此也就不会对原有功能引入风险。符合开闭原则。

- 缺点

  - 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似，如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。

  

**3) 应用场景**

- 建造者（Builder）模式创建的是复杂对象，其产品的各个部分经常面临着剧烈的变化，但将它们组合在一起的算法却相对稳定，所以它通常在以下场合使用。
  - 创建的对象较复杂，由多个部件构成，各部件面临着复杂的变化，但构件间的建造顺序是稳定的。
  - 创建复杂对象的算法独立于该对象的组成部分以及它们的装配方式，即产品的构建过程和最终的表示是独立的。



## 4.5 原型模式

### 4.5.1 原型模式介绍

定义: 原型模式(Prototype Design Pattern)用一个已经创建的实例作为原型，通过复制该原型对象来创建一个和原型对象相同的新对象。

> 西游记中的孙悟空 拔毛变小猴,孙悟空这种根据自己的形状复制出多个身外化身的技巧,在面向对象软件设计领域被称为原型模式.孙悟空就是原型对象.

<img src=".\img\54.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 



**原型模式主要解决的问题**

- 如果创建对象的成本比较大，比如对象中的数据是经过复杂计算才能得到，或者需要从RPC接口或者数据库等比较慢的IO中获取，这种情况我们就可以使用原型模式，从其他已有的对象中进行拷贝，而不是每次都创建新对象，进行一些耗时的操作.



### 4.5.2 原型模式原理

原型模式包含如下角色：

* 抽象原型类(Prototype)：它是声明克隆方法的接口,是所有具体原型类的公共父类,它可以是抽象类也可以是接口.
* 具体原型类(ConcretePrototype)：实现在抽象原型类中声明的克隆方法,在克隆方法中返回自己的一个克隆对象.
* 客户类(Client)：在客户类中,让一个原型对象克隆自身从而创建一个新的对象.由于客户类针对抽象原型类Prototype编程.因此用户可以根据需要选择具体原型类,系统具有较好的扩展性,增加或者替换具体原型类都比较方便.

<img src=".\img\55.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

### 4.5.3 深克隆与浅克隆

根据在**复制原型对象的同时是否复制包含在原型对象中引用类型的成员变量** 这个条件,原型模式的克隆机制分为两种,即浅克隆(Shallow Clone)和深克隆(Deep Clone)

**1) 什么是浅克隆**

被复制对象的所有变量都含有与原来的对象相同的值，而`所有的对其他对象的引用仍然指向原来的对象`(克隆对象与原型对象共享引用数据类型变量)。

​								<img src=".\img\56.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

**2) 什么是深克隆**

除去那些引用其他对象的变量，被复制对象的所有变量都含有与原来的对象相同的值。那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。换言之，深复制把要复制的对象所引用的对象都复制了一遍。

​								<img src=".\img\57.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 



Java中的Object类中提供了 `clone()` 方法来实现**浅克隆**。需要注意的是要想实现克隆的Java类必须实现一个标识接口 Cloneable ,来表示这个Java类支持被复制.

Cloneable接口是上面的类图中的抽象原型类，而实现了Cloneable接口的子实现类就是具体的原型类。代码如下：

**3) 浅克隆代码实现：**  

```java
public class ConcretePrototype implements Cloneable {
    public ConcretePrototype() {
        System.out.println("具体的原型对象创建完成!");
    }

    @Override
    protected ConcretePrototype clone() throws CloneNotSupportedException {
        System.out.println("具体的原型对象复制成功!");
        return (ConcretePrototype)super.clone();
    }
}
```

测试

```java
    @Test
    public void test01() throws CloneNotSupportedException {
        ConcretePrototype c1 = new ConcretePrototype();
        ConcretePrototype c2 = c1.clone();

        System.out.println("对象c1和c2是同一个对象？" + (c1 == c2));
    }
```

**4) 深克隆代码实现**  

在ConcretePrototype类中添加一个对象属性为Person类型

```java
public class ConcretePrototype implements Cloneable {
    private Person person;

    public Person getPerson() {
        return person;
    }

    public void setPerson(Person person) {
        this.person = person;
    }

    void show(){
        System.out.println("嫌疑人姓名: " +person.getName());
    }

    public ConcretePrototype() {
        System.out.println("具体的原型对象创建完成!");
    }

    @Override
    protected ConcretePrototype clone() throws CloneNotSupportedException {
        System.out.println("具体的原型对象复制成功!");
        return (ConcretePrototype)super.clone();
    }

}

public class Person {
    private String name;

   	// 无参 有参 getter setter
}
```

测试

```java
    @Test
    public void test02() throws CloneNotSupportedException {
        ConcretePrototype c1 = new ConcretePrototype();
        Person p1 = new Person();
        c1.setPerson(p1);

        //复制c1
        ConcretePrototype c2 = c1.clone();
        //获取复制对象c2中的Person对象
        Person p2 = c2.getPerson();
        p2.setName("峰哥");

        //判断p1与p2是否是同一对象
        System.out.println("p1和p2是同一个对象？" + (p1 == p2));

        c1.show();
        c2.show();
    }
```

打印结果

<img src=".\img\61.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

**说明: p1与p2是同一对象,这是浅克隆的效果,也就是对具体原型类中的引用数据类型的属性进行引用的复制.**

如果有需求场景中不允许共享同一对象,那么就需要使用**深拷贝**,如果想要进行深拷贝需要使用到对象序列化流 (对象序列化之后,再进行反序列化获取到的是不同对象).  代码如下:

```java
    @Test
    public void test03() throws Exception {
        ConcretePrototype c1 = new ConcretePrototype();
        Person p1 = new Person("峰哥");
        c1.setPerson(p1);

        //创建对象序列化输出流
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("c.txt"));

        //将c1对象写到文件中
        oos.writeObject(c1);
        oos.close();

        //创建对象序列化输入流
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("c.txt"));

        //读取对象
        ConcretePrototype c2 = (ConcretePrototype) ois.readObject();
        Person p2 = c2.getPerson();
        p2.setName("凡哥");

        //判断p1与p2是否是同一个对象
        System.out.println("p1和p2是同一个对象?" + (p1 == p2));

        c1.show();
        c2.show();
    }
```

打印结果:

<img src=".\img\63.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  

> 注意：ConcretePrototype类和Person类必须实现Serializable接口，否则会抛NotSerializableException异常。



其实现在不推荐大家用Cloneable接口，实现比较麻烦，现在借助Apache Commons或者springframework可以直接实现：

- 浅克隆：`BeanUtils.cloneBean(Object obj);BeanUtils.copyProperties(S,T);`
- 深克隆：`SerializationUtils.clone(T object);` 

BeanUtils是利用反射原理获得所有类可见的属性和方法，然后复制到target类。
SerializationUtils.clone()就是使用我们的前面讲的序列化实现深克隆，当然你要把要克隆的类实现Serialization接口。

### 4.5.4 原型模式应用实例

模拟某银行电子账单系统的广告信发送功能,广告信的发送都是有一个模板的,从数据库查出客户的信息,然后放到模板中生成一份完整的邮件,然后交给发送机进行发送处理.

<img src=".\img\58.jpg" alt="image-20220530160637842" style="zoom: 100%;" /> 

**发送广告信邮件UML类图** 

<img src=".\img\59.jpg" alt="image-20220530160637842" style="zoom: 50%;" />  



**代码实现**

- **广告模板代码** 

```java
/**
 * 广告信模板代码
 **/
@Getter
public class AdvTemplate {
    //广告信名称
    private String advSubject = "xx银行本月还款达标,可抽iPhone 13等好礼!";

    //广告信内容
    private String advContext = "达标用户请在2022年3月1日到2022年3月30参与抽奖......";
}
```

- **邮件类代码**

```java
/**
 * 邮件类
 **/
@Data
public class Mail {
    //收件人
    private String receiver;
    //邮件名称
    private String subject;
    //称谓
    private String appellation;
    //邮件内容
    private String context;
    //邮件尾部, 一般是"xxx版权所有"等信息
    private String tail;

    //构造函数
    public Mail(AdvTemplate advTemplate) {
        this.context = advTemplate.getAdvContext();
        this.subject = advTemplate.getAdvSubject();
    }
}
```

- **客户类**

```java
/**
 * 业务场景类
 **/
public class Client {
    //发送信息的是数量,这个值可以从数据库获取
    private static int MAX_COUNT = 6;

    //发送邮件
    public static void sendMail(Mail mail){
        System.out.println("标题: " + mail.getSubject() + "\t收件人: " + mail.getReceiver()
        + "\t..发送成功!");
    }

    public static void main(String[] args) {
        //模拟邮件发送
        int i = 0;
        //把模板定义出来,数据是从数据库获取的
        Mail mail = new Mail(new AdvTemplate());
        mail.setTail("xxx银行版权所有");
        while(i < MAX_COUNT){
            //下面是每封邮件不同的地方
            mail.setAppellation(" 先生 (女士)");
            Random random = new Random();
            int num = random.nextInt(9999999);
            mail.setReceiver(num+"@"+"liuliuqiu.com");
            //发送 邮件
            sendMail(mail);
            i++;
        }
    }
}
```

- **运行结果** 

<img src=".\img\60.jpg" alt="image-20220530160637842" style="zoom: 50%;" /> 

上面的代码存在的问题: 

- 发送邮件需要重复创建Mail类对象,而且Mail类的不同对象之间差别非常小,这样重复的创建操作十分的浪费资源.

  （上面这句有问题，证实过，**Mail只创建了一次**，while内重复使用的是同一个mail，只有Random倒是不断的重复创建了，**只能说假装是这么回事**）

- 这种情况我们就可以使用原型模式,从其他已有的对象中进行拷贝,而不是每次都创建新对象,进行一些耗时的操作.

**代码重构**

- Mail类

```java
/**
 * 邮件类 实现Cloneable接口,表示该类的实例可以被复制
 **/
@Data
public class Mail implements Cloneable{
    //收件人
    private String receiver;
    //邮件名称
    private String subject;
    //称谓
    private String appellation;
    //邮件内容
    private String context;
    //邮件尾部, 一般是"xxx版权所有"等信息
    private String tail;

    //构造函数
    public Mail(AdvTemplate advTemplate) {
        this.context = advTemplate.getAdvContext();
        this.subject = advTemplate.getAdvSubject();
    }

    @Override
    public Mail clone(){
        Mail mail = null;
        try {
            mail = (Mail)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return mail;
    }
}
```

- Client类


```java
/**
 * 业务场景类
 **/
public class Client {
    //发送信息的是数量,这个值可以从数据库获取
    private static int MAX_COUNT = 6;

    //发送邮件
    public static void sendMail(Mail mail){
        System.out.println("标题: " + mail.getSubject() + "\t收件人: " + mail.getReceiver()
        + "\t..发送成功!");
    }

    public static void main(String[] args) {
        //模拟邮件发送
        int i = 0;
        //把模板定义出来,数据是从数据库获取的
        Mail mail = new Mail(new AdvTemplate());
        mail.setTail("xxx银行版权所有");
        while(i < MAX_COUNT){
            //下面是每封邮件不同的地方
            Mail cloneMail = mail.clone();
            cloneMail.setAppellation(" 先生 (女士)");
            Random random = new Random();
            int num = random.nextInt(9999999);
            cloneMail.setReceiver(num+"@"+"liuliuqiu.com");
            //发送 邮件
            sendMail(cloneMail);
            i++;
        }
    }
}
```

反而多此一举了。。。。

只能说记住怎么回事就是了。

### 4.5.5 原型模式总结

**原型模式的优点**

1. 当创建新的对象实例较为复杂时,使用原型模式可以简化对象的创建过程, 通过复制一个已有实例可以提高新实例的创建效率.

   > 比如，在 AI 系统中，我们经常需要频繁使用大量不同分类的数据模型文件，在对这一类文件建立对象模型时，不仅会长时间占用 IO 读写资源，还会消耗大量 CPU 运算资源，如果频繁创建模型对象，就会很容易造成服务器 CPU 被打满而导致系统宕机。通过原型模式我们可以很容易地解决这个问题，当我们完成对象的第一次初始化后，新创建的对象便使用对象拷贝（在内存中进行二进制流的拷贝），虽然拷贝也会消耗一定资源，但是相比初始化的外部读写和运算来说，内存拷贝消耗会小很多，而且速度快很多

2. 原型模式提供了简化的创建结构,工厂方法模式常常需要有一个与产品类等级结构相同的工厂等级结构(具体工厂对应具体产品),而原型模式就不需要这样,原型模式的产品复制是通过封装在原型类中的克隆方法实现的,无须专门的工厂类来创建产品.

3. 可以使用深克隆的方式保存对象状态,使用原型模式将对象复制一份并将其状态保存起来,以便在需要的时候使用,比如恢复到某一历史状态,可以辅助实现撤销操作.

   > 在某些需要保存历史状态的场景中，比如，聊天消息、上线发布流程、需要撤销操作的程序等，原型模式能快速地复制现有对象的状态并留存副本，方便快速地回滚到上一次保存或最初的状态，避免因网络延迟、误操作等原因而造成数据的不可恢复。			

**原型模式缺点**

- 需要为每一个类配备一个克隆方法,而且该克隆方法位于一个类的内部,当对已有的类进行改造时需要修改源代码,违背了开闭原则.



**使用场景**

原型模式常见的使用场景有以下六种。

- 资源优化场景。也就是当进行对象初始化需要使用很多外部资源时，比如，IO 资源、数据文件、CPU、网络和内存等。

- 复杂的依赖场景。 比如，F 对象的创建依赖 A，A 又依赖 B，B 又依赖 C……于是创建过程是一连串对象的 get 和 set。

- 性能和安全要求的场景。 比如，同一个用户在一个会话周期里，可能会反复登录平台或使用某些受限的功能，每一次访问请求都会访问授权服务器进行授权，但如果每次都通过 new 产生一个对象会非常烦琐，这时则可以使用原型模式。

- 同一个对象可能被多个修改者使用的场景。 比如，一个商品对象需要提供给物流、会员、订单等多个服务访问，而且各个调用者可能都需要修改其值时，就可以考虑使用原型模式。

- **需要保存原始对象状态的场景。 比如，记录历史操作的场景中，就可以通过原型模式快速保存记录。**

