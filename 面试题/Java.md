# Java基础

有疑问部分：9扩容、18、26需要派生知识点



**1. 计算机存储单位的基本概念**

- **位 (bit)**：是计算机存储的最小单位，表示一个二进制位（0 或 1）。

- **字节 (Byte)**：是计算机存储的基本单位，1 字节等于 8 位（1 Byte = 8 bits）。

**2. 64 位是不是等于 8 字节？**

是的，64 位是指 **64 bits**，而 1 字节等于 8 位。

**3. 字节和兆的换算**

这里牵涉到 **进制换算**，存储单位是按照 1024 进制计算的，而不是十进制：

- **1 KB (千字节)** = 1024 字节 (Bytes)

- **1 MB (兆字节)** = 1024 KB

- **1 GB (千兆字节)** = 1024 MB

- **1 TB (太字节)** = 1024 GB

**4. 64 位计算机的含义**

“64 位”是指计算机 CPU 的字长（Word Size），即它一次可以处理 64 位数据。和内存单位的 8 字节关联如下：

- 64 位 CPU 每次可以在寄存器中存储或操作 8 个字节数据。

- 64 位系统的地址空间更大，可以支持比 32 位系统更多的内存容量。

**1. 位和字节的基本换算**

- **1 字节 (byte)** = **8 位 (bits)**

这是因为在计算机中，1 个字节由 8 个二进制位组成，形成一个数据存储的基本单位。

例如：

- 1 位可以表示 0 或 1（两个状态）。

- 8 位可以组合出 2^8^ = 256 种不同的状态，这正好适合表示一个字节的数据。

**2. 它们在数据表示中的关系**

- **位（bit）** 是最小的数据单位，用来表示 **二进制的 0 或 1**。

它用于描述单个开关的开（1）或关（0）状态。

- **字节（byte）** 是最基本的存储单位，用来存储一个字符（比如 ASCII 码中的字母、数字等）。

例如：

- 一个英文字符（如 “A”）在内存中占 **1 字节（8 位）**。

- 一个中文字符（如 “你”）在 UTF-8 编码下通常占 **3 字节（24 位）**。

**3. 在硬件中的实际作用**

- **位的作用**

位通常用于表示硬件中的基本信号，比如电流是否通过、电压是否达到某一阈值。

- **字节的作用**

字节更适合实际编程，因为它是 CPU 和内存操作的最小单位。例如，内存地址通常以字节为单位。







## 1、跟我简单的说说什么是java

java是一门面向对象的编程语言，可供跨平台，支持多线程。

面向对象是分析解决问题的步骤，然后用函数把这些步骤一步一步地实现，然后在使用的时候一一调用则可，性能较高

面向过程是把构成问题的事务分解成各个对象，了描述某个事物在解决整个问题的过程中所发生的行为。



### 1.1、Java的基本类型

共8个

byte、short、int、long、float、double、boolean、char



## 2、instanceof有什么作用

用来测试一个对象是否为一个类的实例，

编译器会检查对象是否能够转换成右边的类型，如果不能转换则直接报错，如果不能确定的类型，则通过编译，具体看运行时定



## 3、java自动装箱与拆箱

- 装箱就是自动将基本数据类型转换为包装器类型，6个包装类直接赋值时，就是调用对应的包装类的静态工厂方法`valueOf(int)`

  在JDK9直接把new的构造方法过时，推荐使用`valueof()`，合理利用缓存提升程序性能。

  以Integer为例，缓存为-128~127。因为用的广泛，所以它是唯一可以修改缓存范围的包装类。其中最小值是固定的，最大值可通过虚拟机参数`-XX:AutoBoxCacheMax=<size>`来设置

- 拆箱就是将自动将包装类型转换为基本数据类型，其中调用方法为`intValue`

  

### 3.1、包装类的使用场景

- 包装类还有一个重要的功能，就是适配器，这里以一个String类型 => 到Integer类型,我们知道要想把一个String类型转换一个int类型的数据,new Integer(String).intValue();或者直接调用它的静态方法Integer.parse(String)方法

- **集合类泛型只能是包装类**；

  `List list = new ArrayList<Integer>();`

- 方法参数**允许定义空值**；

  ```java
  private void test(int status){
  	System.out.println(status);
  }
  ```

  看以上代码，方法参数定义的是基本数据类型 int，所以**必须**得传一个数字过来，不能传 null，很多场合我们希望是能传递 null 的，所以这种场合用包装类比较合适。

## 4、重载和重写的区别

- 多态中的**重写**，主要是为解决父类定义的方法达不到子类的期望，那么子类就可以重新实现方法覆盖父类的实现。**元空间有一个方法表，保存着每个可以实例化类的方法信息，JVM可以通过方法表快速地激活实例方法。那么如果某个类重写了父类的某个方法，则方法表中的方法指向引用会指向子类的实现处。**代码通常是用这样的方式来调用子类的方法，通常这也被称作向上转型。

  - 向上转型通过父类引用执行子类方法是需要注意两点：
    - 因为元空间的方法指向引用，子类若自行添加父类不存在的变量或方法，是无法调用到的。
    - 可以调用子类中重写父类，这是一种多态实现

  - 要重写成功，要满足4点：
    - 访问权限不能变小
    - 返回类型能够向上转型成为父类的返回类型
    - 异常也是要能向上转型成父类的异常
    - 方法名、参数类型以及个数必须严格一致。

- 在同一个类中，如果多个方法有相同的方法名称、不同的参数类型、参数个数、参数顺序，即称为重载。说到**重载**，String 类中的 valueOf 是比较著名的重载案例，它有9个方法，可以将基本类型、数组、Object等转化为字符串。在编译器中，方法名称 + 参数列表，组成一个唯一的键，称为方法签名，JVM通过这个唯一键决定调用那种重载的方法。==注意==：**返回值、访问控制符、静态标识符、final**表示符并不是方法签名的一部分，所以相同参数和方法名的重载，却仅有以上4中的区别，是不能重载的。



## 5、equals与`==`的区别

- ==：比较的是变量（栈）内存中存放的对象的（堆）内存地址，用来**判断两个对象的地址是否相同**，是否指同一个对象。
- ==对于基本类型，`==`比较的是值； 对于引用类型，`==`比较的是地址；==
- equals不能用于基本类型的比较；
- 如果没有重写equals，equals就相当于==；如果重写了equals方法，equals比较的是对象的内容；

### 5.1、为什么重写equals ，必须同时重写 hashCode？

总之，Java规定了**如果**`equals`**相等，则**`hashCode`**必定相等**，这是确保Java集合类正常工作的关键规则。

这是因为`equals`和`hashCode`之间有一个重要的契约（即规则），它们的行为必须保持一致，具体原因如下：

 1. **`equals`和`hashCode`的关系：**

    **Java规定，如果两个对象通过`equals`方法判断相等（这个相等说的是对象相等），那么这两个对象的`hashCode`值也必须相等。**换句话说，当你重写了`equals`方法来定义对象相等的逻辑时，你必须确保`hashCode`方法根据相同的逻辑计算出相同的值。

 2. **Hash-based Collections的工作原理：**

    Java中的一些集合类，比如`HashSet`、`HashMap`、`Hashtable`等，依赖`hashCode`来快速查找对象。它们通过对象的`hashCode`值将对象分配到不同的桶中（比如哈希表）。如果你重写了`equals`方法，但没有同时重写`hashCode`方法，可能会导致以下问题：

    - **哈希冲突问题：** 如果两个对象在`equals`方法中认为相等，但它们的`hashCode`值不同，那么它们将被放入哈希表的不同桶中，从而影响集合的正确性，导致查找、插入、删除操作出现异常。
    - **错误的比较和存取：** 例如，在`HashSet`中，如果两个对象的`hashCode`不同，即使它们通过`equals`方法判断是相等的，集合也会认为它们是不同的对象，导致相同的元素多次添加或无法找到。

  3. **`hashCode`的约定：**
     
     - **相等对象的哈希码必须相等：** 如果两个对象通过`equals`方法比较返回`true`，那么它们的`hashCode`值也必须相等。
     - **不相等对象的哈希码不一定会不相同：** 即使两个对象通过`equals`方法比较返回`false`，它们的`hashCode`值也可以相同，只是不同的`hashCode`值能更好地减少哈希冲突，提升性能。



### 5.2、为什么不能仅依赖`hashCode`判断

`hashCode`的**主要目的**是优化存储和查找，而不是判断对象是否相等。

`hashCode`值相同并不意味着两个对象的内容一定相等，因为**哈希冲突**是常见的。

- **哈希冲突：** 不同的对象可能会生成相同的哈希值（即碰撞）。例如，如果两个对象的`hashCode`返回相同的值，哈希表需要进一步通过`equals`方法来确定它们是否确实相等。否则，哈希表可能会误认为两个对象是相同的。

  ```java
  // 假设有两个不同的对象，它们的内容不同
  Person p1 = new Person("Alice", 25);
  Person p2 = new Person("Bob", 30);
  Person p3 = new Person("Alice", 25);
  
  System.out.println(p1.hashCode() == p2.hashCode()); // 可能返回true（假设哈希值相同，但内容不同）hash冲突
  System.out.println(p1.equals(p2));      			// 结果是false，因为它们的内容不同
  System.out.println(p1.equals(p3)); 					// true
  System.out.println(p1.hashCode() == p3.hashCode()); // true equals相同hashcode必相同
  ```

- **`hashCode`只能提供“相等”的可能性，但`equals`才能决定是否真正相等。** 如果你只依赖`hashCode`，你可能错过那些内容相同但`hashCode`不同的对象（这虽然是非常少见的情况，但理论上是可能的）。例如，两个对象的`hashCode`相同，但如果它们在其他字段上有不同的内容，`equals`才会识别出它们并返回`false`。

## 6、关于哈希码的相关知识点

- 解决冲突的方法：

  1. 开放式地址法

     - **线性探索再散列**

       例：用线性探测再散列实现 14、5、21、16、17、15，在0 ~ 10的存储空间中的存放**散列函数为H~(key)~=key%9**。

       | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |
       | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
       |      |      |      |      |      | 14   | 5    | 15   |      |      |      |

       - H~(14)~=14%9=5

       - H~(5)~=5%9=5(冲突)则(5+1)%11=6

         ...（省略）

       - H~(15)~=15%9=6(冲突)则(6+1)%11=7

     - **二次探索再散列**

       与上差不多，只是平方了

       d~1~ = H~(key)~ 	d~i~ = (d~1~ + i^2^) % m 	i = 1,2,3... 

     - **随机探索再散列**

       d~1~ = H~(key)~  	d~i~=(d~1~+R)%m		R是给定的随机数

  2. 链地址法

     - 每个哈希表节点都有一个next指针,多个哈希表节点可以用next指针构成一个单向链表，被 分配到同一个索引上的多个节点可以用这个单向链表进行存储. 

     继上题

     将每个冲突的元素放置对应位置的链表后

     - 填充因子：存储的数据 / 存储空间长度 = 7 / 11 （越小冲突可能性越小，越大冲突可能性越大）

     



## 7、String、StringBuffer 和 StringBuilder的区别是什么

- String是常量对象只能读，一旦被定义就无法增删改，每次对String的操作都会生成一个新的String对象。

  ```java
  private final char value[];
  ```

  隐式的在堆上new了一个跟原字符串相同的StringBuilder对象，在调用append方法拼接 + 后面的字符。

  StringBuffer、StringBuilder是变量

- StringBuffer\StringBuilder都继承了`AbstractStringBuilder`抽象类

  进入抽象类中可以看到

  ```java
  /**
  *	The value is used for character storage.
  */
  char[] value;
  ```

  他们的底层都是可变的字符数组，所以在进行频繁的字符串操作时，建议使用这两个。

  **StringBuffer对方法添加了同步锁所以线程安全、StringBuilder没有线程不安全**

### 7.1、如何让字符串反转？

将对象封装到stringBuilder中，调用reverse方法反转。

```java
String str = "abcde";
StringBuilder builder = new StringBuilder(str);
String ret = builder.reverse().toString();
```



## 8、ArrayList 和 LinkedList的区别

- ArrayList 是容量可以改变的非线程安全集合。支持对元素的随机访问，插入与删除时速度通常很慢。 

  查为什么会快，用下标查的

  ```java
  // Object => Collection接口 => List接口 => ArrayList类、LinkedList类
  public class ArrayList<E> extends AbstractList<E>
          implements List<E>, RandomAccess, Cloneable, Serializable
  // ArrayList 还继承了 AbstractList抽象类，AbstractList 也实现了 List接口
  // Collection接口 继承 Iterable接口，实现此接口可for each 循环
  ```



- LinkedList  本质是双向链表，与ArrayList相比，插入和删除速度快，单随机访问速度则很慢。

  查为什么会慢，一个一个作比较       

  ```java
  // LinkedList 还是实现了另一个接口Deque，即double-ended queue。
  // 这个接口同时具有队列和栈的特性
  public class LinkedList<E>
      extends AbstractSequentialList<E>
      implements List<E>, Deque<E>, Cloneable, Serializable
  ```

  优点：可以将零散的内存单元用过附加引用的方式关联起来，形成按链路顺序查找的线性结构，内存利用率较高。



### 若想线程安全的使用List，该怎么处理？

- 用过时的方法Vactor

  同ArrayList一样底层是一个数组，其中大部分方法都被synchronized关键字所修饰，扩容方法与ArrayList不同，是2倍的扩容。

- 用Collections工具类

  ```java
  List<String> list = Collections.synchronizedList(new ArrayList<>());
  ```

- CopyOnWriteArrayList

  ```java
  List<String> list = CopyOnWriteArrayList<>();
  ```

  比Vactor厉害在哪里，效率高是肯定的，那么主要是COW中没有同步锁



### 怎么排序

Collections内有个排序的api



### 什么是CopyOnWrite?

COW家族，是并发的一种新思路，实行读写分离

- 如果是写操作，则复制一个集合，在新的集合内添加或删除元素。待修改完成之后，再将原集合的引用指向新的集合。

- 好处是，可以高并发地对COW进行读和遍历操作，而不需要加锁，因为之前集合不会添加任何元素。

- 使用COW需注意：

  - 尽量设置何理的容量初始值，他的扩容代价很大
  - 使用批量添加或删除方法，如addAll 或removeAll 操作，在高并发请求下，可以攒一下要添加或删除的元素，避免增加一个元素复制整个集合。
  - 原因：假如几何数据是100MB，再写入50MB，那么某个时间段内占用的就达到（200MB * 2） + 50MB = 450MB，内存的大量占用会导致GC的频繁发生，从而降低服务器的性能。

- 对一段代码的执行

  ```java
  List<long> list = new CopyOnwriteArrayList<Long>();
  
  for(int i=0; i< 20*10000; i++) {
  	list.add(i);
  }
  ```

  循环20万次，不断地进行数据插入，COW用了快100秒，而ArrayList只用了40毫秒。

  由此可知：

  - COW适合用于读多写极少的场景
  - 要初始化这样的COW，建议先将数据填充到ArrayList集合中，再把ArrayList集合当成COW的参数。



## 9、ArrayList 和 HashMap是怎么扩容的

- ArrayList：在第一次add的时候分配内存

  原本空间的大小 + 原本大小按位右移1 = 新扩容大小。

  假设原本空间大小为13，二进制1101按位右移1得110，为6。

  ```java
  // 在JDK7之后是这样的算法
  oldCapacitiy + (oldCapacity>>1) = 19	// 按位运算是基于效率考虑
  // JDK7之前为
  oldCapacitiy * 3 / 2 - 1 = 20。
  ```

  当ArrayList 使用无参构造时，默认大小为10。当然，直接分配内存能减少被动扩容和数组复制的额外开销。



### HashMap

- 默认大小：16，不会在new的时候分配空间，而是在第一次put的时候完成分配的。

capacity 的默认值为16

``` java
public V put(K key, V value) {
	if (table = EMPTY_TABLE) {
		inflateTable(threshold);
    }
    ···· // 省略
}
// 第一次put时，调用如下方法，初始化table
private void inflateTable(int toSize) {
    // 找到大于参数值且最接近的2的幂值，假如输入的参数为27，则返回32
    int capacity = roundUpToPowerOf2(toSize);
    
    // threshold 在不超限制最大值的前提下等于  capacity * loadFactor
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry(capacity);
    initHashSeedAsNeeded(capacity);
}
```

**好像有点不一样了**，再GPT问问

首先检测数组里元素个数，通过计算Load Factor和capacity乘积得到threshold阈值，当capacity大于threshold时候，则会触发扩容。**扩容的大小会是原有的乘2的N次幂**，会把之前的元素再进行一次哈希运算，按照链表或者红黑树的排列方式存放到新的空间中。

- resize（）

  操作时，哈希桶内的元素被元素被逆序排列到新标中。P~207~

- 内部结构

  - table：是一个数组
  - 哈希槽slot：仅是一个位置标识，对应table数组下标
  - 哈希桶bucket：在哈希槽上形成的链表或者树上所有元素的集合。HashMap.size

  1.7数组，链表；1.8数组，链表或者红黑树

- 线程不安全 -新增对象丢失

  HashMap经历了 1.0HashTable安全、HashMap1.2不安全、ConcurrentHashMap安全

  原因：若当前线程迁移过程中，其他线程新增了元素有可能落在了已经遍历过的哈希槽上；在遍历完成后，table 数组引用指向了newTable，这时新增元素就会丢失，被垃圾回收。

  - 并发赋值时被覆盖
  - 已遍历区间新增元素会丢失
  - “新表” 被覆盖
  - 迁移丢失。在迁移过程中，有并发时，next提前置成了null。

  
  
  

### 9.1、红黑树

- **根节点必须是黑色**。
- 所有叶子节点都是黑色。
- 新插入的节点一律为红色

* 如果当前节点是红色，子节点必须是黑色（不能同时有两个红色的节点）
* 从任意节点到每个叶子节点的路径中，黑色节点的数量是相同的。



## 10、为什么HashMap的加载因子为0.75

- 加载因子越大,填满的元素越多,空间利用率越高，但冲突的机会加大了。
  反之,加载因子越小,填满的元素越少,冲突的机会减小,但空间浪费多了。

- 提高空间利用率和 减少查询成本的折中，主要是**泊松分布**，0.75的话碰撞最小，所以选择0.75作为默认的加载因子，完全是时间和空间成本上寻求的一种折衷选择 

```
加载因子过高，例如为1，虽然减少了空间开销，提高了空间利用率，但同时也增加了查询时间成本；

加载因子过低，例如0.5，虽然可以减少查询时间成本，但是空间利用率很低，同时提高了rehash操作的次数。
```





## 11、为什么下标不从1开始

- 如果是从1开始，计算偏移量就要使用当前下标减1的操作。加减法运算对于CPU 来说是一种双数运算，在数组下标使用频率极高的场景下，这种运算是十分耗时的。



## 12、HashMap 和 HashTable的区别

| 特性               | **HashMap**                          | **Hashtable**                                   |
| ------------------ | ------------------------------------ | ----------------------------------------------- |
| **线程安全性**     | 不是线程安全的，需要外部同步控制     | 线程安全的，内部使用同步机制（synchronized）    |
| **null 键和值**    | 允许一个 `null` 键和多个 `null` 值   | 不允许 `null` 键或 `null` 值                    |
| **性能**           | 性能较高，因为没有同步开销           | 性能较低，因为每个操作都需要同步                |
| **实现接口**       | 实现了 `Map` 接口                    | 继承自 `Dictionary` 类，且实现了 `Map` 接口     |
| **迭代器**         | 使用 `Iterator` 进行遍历，不保证顺序 | 使用 `Enumerator` 进行遍历，不保证顺序          |
| **操作方式**       | 常用于非线程安全的场景，如单线程应用 | 适合多线程环境，但已被 `ConcurrentHashMap` 替代 |
| **初始容量和扩容** | 默认初始容量为 16，负载因子为 0.75   | 默认初始容量为 11，负载因子为 0.75              |
| **出现时间**       | Java 1.2 引入，作为集合框架的一部分  | Java 1.0 引入，作为旧版类的一部分               |



### 12.2、ConcurrentHashMap

- #### 12.2.1、为什么HashMap的k-v允许为null，CHM不允许k-v为null？

  HashMap的设计之初，就是为了线程在局部使用的，不存在多线程操作的情况，所以存null与否都不影响当前线程自己的操作。

  ConcurrentHashMap的设计就是为了在多线程的情况下去使用。

  * **如果允许存储null，那么你在get时，获取到了一个null数据，到底是获取到了还是没获取到呢？**
  * 其次这种多线程操作的情况下，null值必然有可能会发生空指针异常的问题。







## 13、Collection、Collections的区别

- Collection是集合类的上级接口
- Collections是集合类的一个工具类，它包含有各种有关集合操作的静态多态方法，
  用于实现对各种集 合的搜索、排序、线程安全化等操作。此类不能实例化，就像一个工具类，服务于Java的Collection框架。 





## 14、 Java的四种引用，强软弱虚

强度依次降低

- 强引用 Strong Reference：最为常见，比如new一个对象，这样的变量声明和定义就会产生对该对象的强引用。只要求强引用指向，并且 GC Roots 可达，那么 Java 内存回收时，即使濒临内存耗尽，也不会回收该对象。

- 软引用 Soft Reference：用SoftReference类实现，一般不会轻易回收，只有内存不够OOM之前才会回收。主要是用来缓存服务器中间计算结果和不需要实时保存的用户行为。例：

  ```java
  // wrf这个引用是强引用，它是指向SoftReference这个对象的， 
  // 软引用指的是指向new String("str")的引用，也就是SoftReference类中T 
  SoftReference<String> wrf = new SoftReference<String>(new String("str"));
  ```

- 弱引用 Weak Reference：用WeakReference类实现，一旦垃圾回收已经启动发现了它，就会回收。当对象只存在弱引用这条路线的时候，那么会在下一次 YGC 时被回收。但由于 YGC 时间不确定性，所以弱引用何时被回收也不确定。

  ```java
  WeakReference<String> wrf = new WeakReference<String>(new String("str"));
  ```

- 虚引用 Phantom Reference：不能单独存在，==必须==和引用队列联合使用，==主要作用是跟踪对象被回收的状态==。PhantomReference

  虚引用的回收机制跟弱引用差不多，但是它被回收之前，会被放入 `ReferenceQueue` 中。而其它引用是被JVM回收后才被传入 ` ReferenceQueue` 中的。

  由于这个机制，所以**虚引用大多被用于引用销毁前的处理工作**。还有就是，虚引用创建的时候，必须带有 ReferenceQueue 

  ```java
  PhantomReference<String> prf = new PhantomReference<String>(new String("str"), new ReferenceQueue<>());
  ```



### 14.1 GC Roots 都有哪些

| **GC Roots 类型**                | **说明**                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| **虚拟机栈（栈帧中的局部变量）** | 方法调用过程中，栈帧中的局部变量表保存的对象引用，作为根对象。通常指当前线程的栈中的局部变量。 |
| **方法区中的类静态变量**         | 静态变量在方法区中存储，在类加载时就存在，不会被销毁，除非类卸载。 |
| **方法区中的常量**               | 存储在方法区常量池中的常量引用，比如字符串常量池中的对象。   |
| **当前线程**                     | 当前运行的线程对象是 GC Roots 之一，因为它是直接与执行过程相关的对象。 |
| **JNI 引用（Native 方法）**      | 本地方法（通过 JNI 调用的）中持有的引用（例如 C++ 等本地代码中保存的引用）。 |
| **Java 虚拟机内部数据结构**      | Java 虚拟机内部所维护的某些对象引用，例如线程管理、垃圾回收等相关的内部对象。 |



## 15、泛型常用特点

泛型是Java SE 1.5之后的特性，着编写的代码可以被不同类型的对象所重用。

1. `<? extends T>`：添加限制

   它可以赋值任何 **T 以及 T 的子类**集合，T 则为最高界限。

   注意：

   - 从中取出的类型有泛型限制，所以向上强转为 T 类型。

   - 除 null 外，**任何元素都不能添加进< ? extends T > 集合内。**

2. `<? super T>`：读取限制

   它可以赋值给任何 **T 以及 T 的父类**集合，Ｔ为最底界限。

   注意：

   - **从中读取数据会造泛型丢失（Object）。**



## 16、Java创建对象有几种方式

```java
// 1. new 关键字（最常见方式）
// 2. Class.forName() 反射方式
Class<?> cls = Class.forName("com.example.MyClass");
Object obj = cls.getDeclaredConstructor().newInstance();
// 3. clone() 方法（复制对象）
MyClass obj1 = new MyClass();
MyClass obj2 = (MyClass) obj1.clone();
// 4. 反序列化恢复对象
ObjectInputStream in = new ObjectInputStream(new FileInputStream("object.ser"));
MyClass obj = (MyClass) in.readObject();
// 5. 工厂方法模式
MyClass obj = MyClassFactory.createMyClass();
// 6. Optional.of()（包装对象）
Optional<MyClass> optional = Optional.of(new MyClass());
// 7. enum 类型（自动创建单例对象，单例）
public enum MyEnum {
    INSTANCE;
}
MyEnum obj = MyEnum.INSTANCE;
```



## 17、深拷贝和浅拷贝的区别是什么

如何实现对象克隆？

1. 实现Cloneable接口，重写clone方法；
2. 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深克隆。
3. BeanUtils，apache和Spring都提供了bean工具，只是这都是浅克隆。**常用工具基本是浅克隆**

- 浅拷贝：仅仅克隆基本类型变量，**不克隆引用类型变量**；
- 深克隆：既克隆基本类型变量，**有克隆引用类型变量**；



## 18、ﬁnal有哪些用法

ﬁnal也是很多面试喜欢问的地方

- 被ﬁnal修饰的**常量**，在编译阶段会存入常量池

- 被ﬁnal修饰的**变量不可被改变**。
- 被ﬁnal修饰的**引用变量**（refvar），那么表示引用不可变，引用指向的内容可变
- 被ﬁnal修饰的**方法不可被重写，JVM会尝试将其内联，以提高运行效率**（待确认）

- 被ﬁnal修饰的**类不可被继承** 

所以线程安全、约等于可见性等，也可以一同说



## 19、static都有哪些用法

类变量、类方法、

静态导包即 import static，import static是在JDK 1.5之后引入的新特性，可以用来指定导入某个类中的静态资源，并且不需要使用类名，可以直接使用资源名。

执行顺序：

1. 主调类的静态代码块       static{ //静态代码块又称为静态初始化块 }
2. 父类的静态代码块和静态成员变量，按出现顺序初始化。
3. 子类的静态代码块和静态成员变量，按出现顺序初始化。
4. 父类的非静态代码块 { }
5. 父类的构造函数
6. 子类的非静态代码块 { }
7. 子类的构造函数

静态变量只能在类主体中定义，不能在方法中定义。（**在方法中不能定义static**）
静态变量属于类所有，而不属于方法，所以叫类变量。

类方法可以调用其他类的static方法。

static静态成员属性==不能==使用this关键词调用

接口中不允许有static类型的方法。
Serializable是不会序列化static变量和transient修饰的变量。

静态成员变量在类加载时就分配的内存。



## 20、3*0.1== 0.3 返回的值是什么

false，浮点数不能精确





## 21、 Exception与Error包结构

Java可抛出(Throwable)的结构分为三种类型：被检查的异常(CheckedException)，运行时异常 (RuntimeException)，错误(Error)。 

1. 运行时异常

   定义：RuntimeException及其子类unchecked都被称为运行时异常。 不需要程序进行显示的try-catch捕捉和处理throws，unchecked可细分为3类：

   - 可预测异常（Predicted Exception）
   
     常见的是数组越界，未找到类的异常，基于对代码的性能和稳定型要求，此类一场不应该被产生或者抛出，而应该提前做好检查的处理。

   - 需捕捉异常（Caution Exception）

   - 可透出异常（Ignored Exception）
   
   常见5种运行时异常：
   
   - 数组越界	      IndexOutOfBoundsException
   
   - fail-fast机制产生的   ConcurrentModificationException
   
     java.util下的所有集合类都是fail-fast，而concurrent 包中的集合类都是fail-safe。
     与fail-fast不同，fail-safe 是对遍历时频繁被修改的集合进行快照，那么再修改就无关。
   
   - 空指针异常            NullPointerException



2. 被检查异常

   定义：Exception类本身，以及Exception的子类中除了"运行时异常"之外的其它子类都属于被检查异常。checked异常需要在在代码中显示处理的异常，否则会编译出错；如果无法处理，则继续向调用方抛出异常对象。checked可细分为2类：

   - 无能为力、引起注意型

     如字段超长等导致的SQLException

   - 力所能及、坦然处理型

     如发生未授权异常（UnAuthorizedException），程序回跳转至授权页面

   常见的：

   - SQLException
   - ClassNotFoundException
   - IOException
   - 等

   

3. error

   Error是非常特殊的异常类型，他的出现标志着系统发生了不可控的错误，如StackOverflowError、OutOfMemoryError。针对此类错误，程序无法处理，只能人工介入。





## 22、OOM你遇到过哪些情况，SOF你遇到过那些情况

**OOM：**程序计数器外，虚拟机内存的其他几个运行时区域都有发生OutOfMemoryError(OOM)异常的可能。

1. Java Heap 溢出： 可以将堆的造成的OOM情况描述出来
2. 虚拟机栈和本地方法栈溢出
   如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常
3. 运行时常量池溢出
4. 方法区溢出

常见原因：

- 内存加载的数据量太大
- 一次性从数据库取太多数据
- 集合类中有对对象的引用，使用后未清空，GC不能进行回收
- 代码中存在循环产生过多的重复对象
- 启动参数堆内存值小。 



**SOF（堆栈溢出）：**当应用程序递归太深而发生堆栈溢出时，抛出该错误。 

- 栈溢出的原因：递归调用，大量循环或死循环，全局变量是否过多，数组、List、map数据过大。

- 为栈一般默认为1-2m，一旦出现死循环或者是大量的递归调用，在不断的压栈过程中，造成栈容量超过1m而导致溢出（这个弄懂了栈结构的1-2m，再说）。 



## 23、线程有几个状态

```java
public enum State {    
    // 新生, 被构建，但还没调用start()方法
    NEW,
    
    // 运行, Java线程将操作系统中的就绪和运行两种状态庞统地称作“运行”
    RUNNABLE,
    
    // 阻塞, 线程阻塞于锁    
    BLOCKED,
    
    // 等待，死死地等    
    WAITING,
    
    // 超时等待, 不同WAITING，他是可以在指定的时间自行返回的    
    TIMED_WAITING,
    
    // 终止，线程执行完毕
    TERMINATED; 
}
```





## 24、Java 可以开启线程吗？

通过源码可以得知：开不了

点开`new Thread().start`的start查看源码，方法加了同步锁，把当前线程加入线程组后调用了`start0()`方法

```java
public synchronized void start() {
    
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    // 把当前线程加入线程组
    group.add(this);

    boolean started = false;
    try {
        // 注意这个方法，在下方
        start0();
        started = true;
        
    	.....
            
    }
}
// 本地方法，底层的C++ ，Java 无法直接操作硬件 
private native void start0();
```





## 25、Java 序列化中如果有些字段不想进行序列化，怎么办？

对于不想进行序列化的变量，使用 transient 关键字修饰。 类变量也不会。

- transient 只能修饰变量，不能修饰类和方法。
- 当对象被反序列化时，被 transient 修饰的变量值不会被持久化和恢复。



### 25.1、什么是 java 序列化？

- `序列化就是一种用来处理对象流的机制`。将对象的内容流化，**将流化后的对象传输于网络之间**。

- 通过实现serializable接口，该接口没有需要实现的方法，implement Serializable只是为了标注该对象是可被序列化的。

- 序列化是将对象转换为容易传输的格式的过程。

  ### 什么情况下需要序列化？

  - 序列化一个对象，通过HTTP通过Internet在客户端和服务器之间传输该对象。在另一端，反序列化将从流中心构造成对象。
  - 对象序列化的最主要目的就是传递和保存对象，保存对象的完整性和可传递性。
  - 网络传输或者把一个对象保存成本地一个文件的时候，需要使用序列化。





## 26、Java 中 IO 流

Java 中 IO 流分为几种?

- 按照流的流向分，可以分为输入流和输出流
- 按照操作单元划分，可以划分为字节流和字符流
- 按照流的角色划分为节点流和处理流。

 Java IO 流的 40 多个类都是从如下 4 个抽象类基类中派生出来的。 

- InputStream/Reader: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。

- OutputStream/Writer: 所有输出流的基类，前者是字节输出流，后者是字符输出流

需要派生知识点。





## 27、BIO、NIO、AIO 有什么区别？

1. 同步阻塞BIO

   一个连接一个线程。

   JDK1.4之前，建立网络连接的时候采用BIO模式，先在启动服务端socket，然后启动客户端socket，对服务端通信，客户端发送请求后，先判断服务端是否有线程响应，如果没有则会一直等待或者遭到拒绝请求，如果有的话会等待请求结束后才继续执行。

2. 同步非阻塞NIO

   NIO主要是想解决BIO的大并发问题，BIO是每一个请求分配一个线程，当请求过多时，每个线程占用一定的内存空间，服务器瘫痪了。

   JDK1.4开始支持NIO，适用于连接数目多且连接比较短的架构，比如聊天服务器，并发局限于应用中。

   一个请求一个线程。

3. 异步非阻塞AIO

   一个有效请求一个线程。

   JDK1.7开始支持AIO，适用于连接数目多且连接比较长的结构，比如相册服务器，充分调用OS参与并发操作。



- NIO效率比IO效率会高出很多。
- NIO的通道是可以双向的，但是IO中的流只能是单向的。
- 更多知识点：https://mp.weixin.qq.com/s/N1ojvByYmary65B6JM1ZWA 





## 28、java反射

**射的作用于原理** 

1. **定义**

   反射机制是在运行时，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意个对象，都能够调用它的任意一个方法。

2. **哪里会用到反射机制？** 

   jdbc就是一个典型的反射

   ```java
   Class.forName('com.mysql.jdbc.Driver.class');
   ```

3. **反射的实现方式：** 

   例：

   - `Class.forName("com.baven.pojo.Student");`
   - `Student.Class`
   - `student.getClass`
   - 基本类型的包装类，可以调用包装类的Type属性来获得该包装类的Class对象 

4. **实现Java反射的类：**

   - Class：表示正在运行的Java应用程序中的类和接口  
     注意：所有获取对象的信息都需要Class类来实现。 
   - Field：提供有关类和接口的属性信息，以及对它的动态访问权限。
   - Constructor：提供关于类的单个构造方法的信息以及它的访问权限 
   - Method：提供类或接口中某个方法的信息

5. **反射机制的优越点：**

   优点：

   - 能够运行时动态获取类的实例，提高灵活性
   - 与动态编译结合

   缺点：

   - 相对不安全，破坏了封装性（因为通过反射可以获得私有方法和属性） 
   - 使用反射性能较低，需要解析字节码，将内存中的对象进行解析
   - 解决方案：
     - 通过setAccessible(true)关闭JDK的安全检查来提升反射速度
     - 多次创建一个类的实例时，有缓存会快很多
     - ReﬂectASM工具类，通过字节码生成的方式加快反射速度（可不说）

   



## 29、说说List，Set，Map三者的区别

- **List(对付顺序的好帮手)：** List接口存储一组不唯一（可以有多个元素引用相同的对象），
- **有序的 对象 Set(注重独一无二的性质):** 不允许重复的集合。不会有多个元素引用相同的对象。 
- **Map(用Key来搜索的专家):** 使用键值对存储。Map会维护与Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，但也可以是任何对象。



hashSet的底层就是hashMap，源码：

```java
public HashSet() {
	map = new HashMap<>();
}

// 对应add的源码：
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

// 一个不变的值
private static final Object PRESENT = new Object();
```





## 30、关于对象创建的大小

`refvar` (Reference Variable)在面向对象世界中称为**引用变量**，或者叫引用句柄。`是基本数据类型，默认值null`
把引用对象指向 实际对象 (Refered Object) 简称refobj。

无论 refobj 是多么小的对象，最小占用的存储空间是**12B**（对象头，用于存储基本信息），但由于存储空间分配必须是8B的倍数，所以初始分配的空间至少是16B

一个 refvar 最多只能存储一个 refobj 的首地址，一个 refobj 可以被多个 refvar 存储下他的`首地址`，也就是一个堆内对象可以被多个 refvar 引用指向。如果一个 refobj 没有被任何 refvar 指向，那么迟早要被垃圾回收。而 refvar 的释放与其他基本数据类型类似。

---

类定义中的方法代码不占用实例对象的任何空间。IntergerCache 是Interger的静态内部类，容量占用也与实例对象无关。

一个MyNumber类里有一个基本数据类型int，那么这个类的对象就是12B + 4B = 16，若是double那么是24B（8的倍数）。

```java
class MyNumber(){
    // 对象头最小占用12B空间
    
    // int占用4B，到此对象共占用16B
    int i;
    
    // 每个引用变量4B
    Object obj;
    Object obj;
    
    // MyNumber2 实例占用空间并不计算在本对现象内，依然只计算引用变量大小的4B
    MyNumber2 my2 = new MyNumber2();
    
    // 综上，12B + 4B + (4B * 2) + 4B = 26个字节
    // 去8倍数为32个字节
}
```

Integer 的实例成员只有一个private int value，占用4个字节，加上对象头为16个字节；
再入上述的MyNumber 对象大小为32个字节，一个子类Test继承MyNumber，即使子类内部是空的，new Test的对象也是32个字节。



### 为什么会是8的倍数呢？

对齐填充（Padding）

对象的内存空间分配单位是8个字节。

如果一个对象占用16字节，增加一个成员变量byte类型，此时需要占用17个字节，但是也会分配24个字节进行对其填充操作。



### 对象头最小占用空间12B，其内部存储的是什么信息？

存储内容包括：

- 对象标记（**markOop**）markword

  对象标记存储对象本身运行时的数据，`哈希码、GC标记信息（三色标记算法信息）、锁信息（01，00，10，11，01）、线程关联信息等`，这部分数据在64位JVM上占用8字节=64位，称为“Mark Word”。注意：为了存储更多状态信息，对象标记的存储格式是非固定的（具体与JVM实现有关）。

  细：当一个对象计算过 indentityHashCode 后，则无法进入到偏向锁。因为直接标记占用那25位地方，这样偏向的标识就无法标识。

- 类元信息（**klassOop**）

  存储的是对象指向它的类元数据（Klass）的首地址，占用4B，与refvar开销一致。指向的是对应类的.Class





## 31、简单介绍一下Abstract







# Collections 工具类API

## 1. 排序和查找相关方法

- **sort(List<T> list)**

  * 用于对 List 进行升序排序。排序是基于 List 元素的自然顺序（即 Comparable 接口的实现）。

  * 示例：

    ```java
    List<Integer> list = Arrays.asList(4, 2, 5, 1, 3);
    Collections.sort(list); // 排序
    System.out.println(list); // 输出: [1, 2, 3, 4, 5]
    ```

* **sort(List<T> list, Comparator<? super T> c)**

  * 用于根据自定义的 Comparator 对 List 进行排序。

  * 示例：

    ```java
    List<String> list = Arrays.asList("apple", "banana", "cherry");
    Collections.sort(list, (a, b) -> b.compareTo(a)); // 按降序排序
    System.out.println(list); // 输出: [cherry, banana, apple]
    ```

- **binarySearch(List<? extends Comparable<? super T>> list, T key)**

  * 在已排序的 List 中使用二分查找来查找元素。

  * 示例：

    ```java
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    int index = Collections.binarySearch(list, 3);
    System.out.println(index); // 输出: 2
    ```

- **shuffle(List<?> list)**

  * 随机打乱 List 中的元素顺序。

  * 示例：

    ```java
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    Collections.shuffle(list);
    System.out.println(list); // 输出顺序可能不同
    ```

- **reverse(List<?> list)**

  * 反转 List 中元素的顺序。

  * 示例：

    ```java
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    Collections.reverse(list);
    System.out.println(list); // 输出: [5, 4, 3, 2, 1]
    ```

## **2.** **线程安全相关方法**

- **synchronizedList(List<T> list)**

​	* 返回一个线程安全的 List，即对原始 List 的所有操作都被同步。

​	* 示例：

```java
List<Integer> list = new ArrayList<>();
List<Integer> syncList = Collections.synchronizedList(list);
```

- **synchronizedSet(Set<T> s)**

​	* 返回一个线程安全的 Set。

- **synchronizedMap(Map<K, V> m)**

​	* 返回一个线程安全的 Map。

## **3.** **常量和空集合**

- **emptyList()**

​	* 返回一个空的不可修改的 List。

​	* 示例：

```java
List<String> emptyList = Collections.emptyList();
```

- **emptySet()**

​	* 返回一个空的不可修改的 Set。

​	* 示例：

```java
Set<String> emptySet = Collections.emptySet();
```

- **emptyMap()**

​	* 返回一个空的不可修改的 Map。

## **4.** **集合元素操作**

- **copy(List<? super T> dest, List<? extends T> src)**

​	* 将 src 中的元素复制到 dest 中。 dest 必须足够大，或者会抛出 IndexOutOfBoundsException。

​	* 示例：

```java
List<Integer> src = Arrays.asList(1, 2, 3);
List<Integer> dest = new ArrayList<>(Arrays.asList(0, 0, 0));
Collections.copy(dest, src);
System.out.println(dest); // 输出: [1, 2, 3]
```

- **fill(List<? super T> list, T obj)**

​	* 将 List 中的所有元素替换为指定的 obj。

​	* 示例：

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));
Collections.fill(list, 0);
System.out.println(list); // 输出: [0, 0, 0]
```

- **frequency(Collection<?> c, Object o)**

​	* 计算某个元素在集合中出现的次数。

​	* 示例：

```java
List<Integer> list = Arrays.asList(1, 2, 3, 1, 1, 4);
int count = Collections.frequency(list, 1);
System.out.println(count); // 输出: 3
```

- **max(Collection<? extends T> coll)**

​	* 返回集合中的最大元素，前提是集合中的元素实现了 Comparable 接口。

​	* 示例：

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
int max = Collections.max(list);
System.out.println(max); // 输出: 5
```

- **min(Collection<? extends T> coll)**

​	* 返回集合中的最小元素。

## **5.** **不可变集合**

- **unmodifiableList(List<? extends T> list)**

​	* 返回一个不可修改的 List，对其进行修改将抛出 UnsupportedOperationException。

- **unmodifiableSet(Set<? extends T> s)**

​	* 返回一个不可修改的 Set。

- **unmodifiableMap(Map<? extends K, ? extends V> m)**

​	* 返回一个不可修改的 Map。

## **6.** **其他常用方法**

- **disjoint(Collection<?> c1, Collection<?> c2)**

​	* 检查两个集合是否没有交集（即没有共同的元素）。

​	* 示例：

```java
List<Integer> list1 = Arrays.asList(1, 2, 3);
List<Integer> list2 = Arrays.asList(4, 5, 6);
boolean disjoint = Collections.disjoint(list1, list2);
System.out.println(disjoint); // 输出: true
```

- **rotate(List<?> list, int distance)**

​	* 将 List 中的元素按照给定的距离进行旋转。

​	* 示例：

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
Collections.rotate(list, 2);
System.out.println(list); // 输出: [4, 5, 1, 2, 3]
```

这些方法可以帮助你在 Java 中高效地操作和管理集合，提供了灵活的功能和高度的可定制性。
