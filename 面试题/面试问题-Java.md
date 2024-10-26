# Java基础

## 1、跟我简单的说说什么是java

java是一门面向对象的编程语言，可供跨平台，支持多线程。

面向对象是分析解决问题的步骤，然后用函数把这些步骤一步一步地实现，然后在使用的时候一一调用则可，性能较高

面向过程是把构成问题的事务分解成各个对象，了描述某个事物在解决整个问题的过程中所发生的行为。



### 1.1、Java的基本类型

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

- 集合类泛型只能是包装类；

  `List list = new ArrayList<Integer>();`

- 方法参数允许定义空值；

  ```java
  private void test(int status){
  	System.out.println(status);
  }
  ```

  看以上代码，方法参数定义的是基本数据类型 int，所以必须得传一个数字过来，不能传 null，很多场合我们希望是能传递 null 的，所以这种场合用包装类比较合适。

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

- 在同一个类中，如果多个方法有相同的方法名称、不同的参数类型、参数个数、参数顺序，即称为重载。说到**重载**，String 类中的 valueOf 是比较著名的重载案例，它有9个方法，可以将基本类型、数组、Object等转化为字符串。在编译器中，方法名称 + 参数列表，组成一个唯一的键，称为方法签名，JVM通过这个唯一键决定调用那种重载的方法。==注意==：返回值、访问控制符、静态标识符、final表示符并不是方法签名的一部分，所以遇到相同参数和方法名，却只有以上4中的区别，是不能重载的。



## 5、equals与`==`的区别

- ==：比较的是变量（栈）内存中存放的对象的（堆）内存地址，用来判断两个对象的地址是否相同，是否指同一个对象。
- 对于基本类型，`==`比较的是值； 对于引用类型，`==`比较的是地址；
- equals不能用于基本类型的比较；
- 如果没有重写equals，equals就相当于==；如果重写了equals方法，equals比较的是对象的内容；

### 5.1、为什么重写equals ，必须同时重写 hashCode？

若hashCode不同，将直接判定Objects不同，跳过equals，加快了冲突处理效率。可能会存在两个相同的对象。

自定义的对象作为Map的键，那么必须要覆写hashCode 和equals。

1. 做了个实验，一个对象只重写了equals，然后将三个复制相同的对象存进hashSet中，最终hashSet大小为3！

   原因：因为如果不重写hashCode方法，即使equals()相等也没意义，Object.hashCode() 的实现是默认为每一个对象生成不同的int 数值，他本身就是native 方法，一般与对象内存地址有关。从底层代码中也可知，hashCode 就是根据对象的地址进行相关计算得到int类型数值的。因为刚才对象没有重写hashCode，所以得到的是一个与对象地址相关的唯一值。

2. 再举例：ArrayList<integer> 和 LinkedList<interger> ，然后各add(1)；判断 arrayList.equals(linkedList) 为true。

   原因： **ArrayList的 equals() 只进行了是否为List子类的判断**



```java
public class Student {
    private String username;
    private String password;
    private Integer age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return Objects.equals(username, student.username) &&
                Objects.equals(password, student.password) &&
                Objects.equals(age, student.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(username, password, age);
    }
}
```





## 6、hashCode方法的作用

- 对象使用 hashcode方法 可生成哈希值。这个方法通常跟 equals 配合着使用，因为计算出的哈希值会存在冲突的情况，所以当 hashCode 相同时，还需要在调用 equals 进行一次值得比较。但是，上题答案

  **Object 类定义中对 hashCode 和 equals 要求如下：**

  1. 如果两对象的euqals 的结果相等，那么两个对象的hashCode的返回结果也必须相同的。
  2. 任何时候重写equals ，都必须同时重写 hashCode。



### 6.1、关于哈希码的相关知识点

- 解决冲突的方法：

  1. 开放式地址法

     - 线性探索再散列

       例：用线性探测再散列实现 14、5、21、16、17、15，在0 ~ 10的存储空间中的存放散列函数为H~(key)~=key%9。

       | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |
       | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
       |      |      |      |      |      | 14   | 5    | 15   |      |      |      |

       - H~(14)~=14%9=5

       - H~(5)~=5%9=5(冲突)则(5+1)%11=6

         ...（省略）

       - H~(15)~=15%9=6(冲突)则(6+1)%11=7

     - 二次探索再散列

       遇上差不多，只是平方了

       d~1~ = H~(key)~ 	d~i~ = (d~1~ + i^2^) % m 	i = 1,2,3... 

     - 随机探索再散列

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

  StringBuffer对方法添加了同步锁所以线程安全、StringBuilder没有线程不安全

### 7.1、如何让字符串反转？

将对象封装到stringBuilder中，调用reverse方法反转。

```java
String str = "abcde";
StringBuilder builder = new StringBuilder(str);
String ret = builder.reverse().toString();
```





## 8、ArrayList 和 LinkedList的区别

- ArrayList 是容量可以改变的非线程安全集合。支持对元素的随机访问，插入与删除时速度通常很慢。 

  为什么会快，用下标查的

  ```java
  // Object => Collection接口 => List接口 => ArrayList类、LinkedList类
  public class ArrayList<E> extends AbstractList<E>
          implements List<E>, RandomAccess, Cloneable, Serializable
  // ArrayList 还继承了 AbstractList抽象类，AbstractList 也实现了 List接口
  // Collection接口 继承 Iterable接口，实现此接口可for each 循环
  ```



- LinkedList  本质是双向链表，与ArrayList相比，插入和删除速度快，单随机访问速度则很慢。

  为什么会慢，一个一个作比较       

  ```java
  // LinkedList 还是实现了另一个接口Deque，即double-ended queue。
  // 这个接口同时具有队列和栈的特性
  public class LinkedList<E>
      extends AbstractSequentialList<E>
      implements List<E>, Deque<E>, Cloneable, Serializable
  ```

  优点：可以将零散的内存单元用过附加引用的方式关联起来，形成按链路顺序查找的线性结构，内存利用率较高。



### 若想线程安全的使用List，该怎么处理？

- 用过时的方法Vactor，同ArrayList一样底层是一个数组，其中大部分方法都被synchronized关键字所修饰，扩容方法与ArrayList不同，是2倍的扩容。

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



### 什么是CopyOnWrite?

COW家族，是并发的一种新思路，实行读写分离

- 如果是写操作，则复制一个集合，在新的集合内添加或删除元素。待修改完成之后，再将原集合的引用指向新的集合。

- 好处是，可以高并发地对COW进行读和遍历操作，而不需要加锁，因为之前集合不会添加任何元素。

- 使用COW需注意：

  - 尽量设置何理的容量初始值，他的扩容代价很大
  - 使用批量添加或删除方法，如addAll 或removeAll 操作，在高并发请求下，可以攒一下要添加或删除的元素，避免增加一个元素复制整个集合。
  - 原因：假如几何数据是100MB，再写入50MB，那么某个时间段内占用的就达到（200MB * 2） + 50MB = 250MB，内存的大量占用会导致GC的频繁发生，从而降低服务器的性能。

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

首先检测数组里元素个数，通过计算Load Factor和capacity乘积得到threshold阈值，当capacity大于threshold时候，则会触发扩容。扩容的大小会是原有的乘2的N次幂，会把之前的元素再进行一次哈希运算，按照链表或者红黑树的排列方式存放到新的空间中。

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

- 源码得知新插入的节点一律赋为红色
- 不能有两个红色的节点

* 每个节点必须是红色或者黑色。
* **根节点必须是黑色**。
* 如果当前节点是红色，子节点必须是黑色
* 所有叶子节点都是黑色。
* 从任意节点到每个叶子节点的路径中，黑色节点的数量是相同的。



## 10、为什么HashMap的加载因子为0.75

- 加载因子越大,填满的元素越多,空间利用率越高，但冲突的机会加大了。
  反之,加载因子越小,填满的元素越少,冲突的机会减小,但空间浪费多了。

- 提高空间利用率和 减少查询成本的折中，主要是泊松分布，0.75的话碰撞最小，所以选择0.75作为默认的加载因子，完全是时间和空间成本上寻求的一种折衷选择 

```
加载因子过高，例如为1，虽然减少了空间开销，提高了空间利用率，但同时也增加了查询时间成本；

加载因子过低，例如0.5，虽然可以减少查询时间成本，但是空间利用率很低，同时提高了rehash操作的次数。
```





## 11、为什么下标不从1开始

- 如果是从1开始，计算偏移量就要使用当前下标减1的操作。加减法运算对于CPU 来说是一种双数运算，在数组下标使用频率极高的场景下，这种运算是十分耗时的。



## 12、HashMap 和 HashTable的区别

1. 两者父类不同

   - HashMap继承AbstractMap类
   - HashTable继承Dictionary类

   不过都同时实现了Map、Cloneable、Serializable这三个接口

2. 对外提供的接口不同

3. 对null的支持不同

   HashTabe：key和value都不能为空

   HashMap：key可以为空，但为确保唯一性只能一个；值可以多个为空。根据实现类约束为准。

4. 安全性不同

   HashMap线程不安全，在多线程并发的环境下，可能会产生死锁、扩容数据丢失等问题。

   HashTable是线程安全，每个方法都加了synchronized关键字。

   - 虽然HashMap是线程不安全的，但是它的效率远远高于Hashtable，这样设计是合理的，因为大部分的 使用场景都是单线程。
   - ConcurrentHashMap也线程安全，效率比HashTable高很多倍，采用了分段锁，并不对整个数据进行锁定。

5. 初始容量大小和每次扩容大小不同

6. 计算hash值的方法不同





### 12.2、ConcurrentHashMap







## 13、Collection、Collections的区别

- Collection是集合类的上级接口
- Collections是集合类的一个工具类，它包含有各种有关集合操作的静态多态方法，用于实现对各种集 合的搜索、排序、线程安全化等操作。此类不能实例化，就像一个工具类，服务于Java的Collection框 架。 





## 14、 Java的四种引用，强软弱虚

强度依次降低

- 强引用 Strong Reference：最为常见，比如new一个对象，这样的变量声明和定义就会产生对该对象的强引用。只要求强引用指向，并且 GC Roots 可达，那么 Java 内存回收时，即使濒临内存耗尽，也不会回收该对象。

- 软引用 Soft Reference：用SoftReference类实现，一般不会轻易回收，只有内存不够OOM之前才会回收。主要是用来缓存服务器中间计算结果及不需要实时保存的用户行为。例：

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

  由于这个机制，所以虚引用大多被用于引用销毁前的处理工作。还有就是，虚引用创建的时候，必须带有 ReferenceQueue 

  ```java
  PhantomReference<String> prf = new PhantomReference<String>(new String("str"), new ReferenceQueue<>());
  ```



### 14.1 GC Roots 有哪些





## 15、泛型常用特点

泛型是Java SE 1.5之后的特性，着编写的代码可以被不同类型的对象所重用。

1. `<? extends T>`：

   它可以赋值任何 **T 以及 T 的子类**集合，T 则为最高界限。

   注意：

   - 从中取出的类型有泛型限制，所以向上强转为 T 类型。

   - 除 null 外，**任何元素都不能添加进< ? extends T > 集合内。**

2. `<? super T>`：

   它可以赋值给任何 **T 以及 T 的父类**集合，Ｔ为最底界限。

   注意：

   - **从中读取数据会造泛型丢失。**



## 16、Java创建对象有几种方式

- new 创建新对象
- 通过反射机制
- 采用clone机制
- 通过序列化机制



## 17、深拷贝和浅拷贝的区别是什么

如何实现对象克隆？

1. 实现Cloneable接口，重写clone方法；
2. 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深克隆。
3. BeanUtils，apache和Spring都提供了bean工具，只是这都是浅克隆。

- 浅拷贝：仅仅克隆基本类型变量，**不克隆引用类型变量**；
- 深克隆：既克隆基本类型变量，**又克隆引用类型变量**；



## 18、ﬁnal有哪些用法

ﬁnal也是很多面试喜欢问的地方

- 被ﬁnal修饰的**常量**，在编译阶段会存入常量池

- 被ﬁnal修饰的**变量不可被改变**。
- 被ﬁnal修饰的**引用变量**（refvar），那么表示引用不可变，引用指向的内容可变
- 被ﬁnal修饰的**方法不可被重写，JVM会尝试将其内联，以提高运行效率**（待确认）

- 被ﬁnal修饰的**类不可被继承** 



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

静态变量只能在类主体中定义，不能在方法中定义。--在方法中不能定义static
静态变量属于类所有，而不属于方法，所以叫类变量。
类方法可以调用其他类的static方法。

static不能修饰方法中的局部变量。
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
   
   - 数组越界		    IndexOutOfBoundsException
   
   - fail-fast机制产生的   ConcurrentModificationException
   
     java.util下的所有集合类都是fail-fast，而concurrent 包中的集合类都是fail-safe。
     与fail-fast不同，fail-safe 是对遍历时频繁被修改的集合进行快照，那么再修改就无关。
   
   - 空指针异常                NullPointerException



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

- 序列化就是一种用来处理对象流的机制。将对象的内容流化，将流化后的对象传输于网络之间。

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

// 那么add的源码：
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

// 一个不变的值
private static final Object PRESENT = new Object();
```





## 30、关于对象创建的大小

refvar (Reference Variable)在面向对象世界中称为引用变量，或者叫引用句柄。是基本数据类型，默认值null
把引用对象指向 实际对象 (Refered Object) 简称refobj。

无论refobj是多么小的对象，最小占用的存储空间是12B（用于存储基本信息，称为对象头），但由于存储空间分配必须是8B的倍数，所以初始分配的空间至少是16B

一个refvar最多只能存储一个refobj的首地址，一个refobj可以被多个refvar存储下他的首地址，也就是一个堆内对象可以被多个refvar引用指向。如果一个refobj没有被任何refvar指向，那么迟早要被垃圾回收。而refvar的释放与其他基本数据类型类似。

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

- 对象标记（markOop）

  对象标记存储对象本身运行时的数据，哈希码、GC标记信息（三色标记算法信息）、锁信息（01，00，10，11，01）、线程关联信息等，这部分数据在64为JVM上占用8字节=64位，称为“Mark Word”。注意：为了存储更多状态信息，对象标记的存储格式是非固定的（具体与JVM实现有关）。

  细：当一个对象计算过 indentityHashCode 后，则无法进入到偏向锁。因为直接标记占用那25位地方，这样偏向的标识就无法标识。

- 类元信息（klassOop）

  存储的是对象指向它的类元数据（Klass）的首地址，占用4B，与refvar开销一致。指向的是对应类的.Class





## 31、简单介绍一下Abstract



# JVM

 JVM是Java运行基础,面试时一定会遇到JVM的有关问题,内容相对集中,但对只是深度要求较高.





## 1、把你所能知道关于JVM的知识点说出来

- 线程独占：虚拟机栈，本地方法栈，程序计数器 
  线程共享：堆，方法区 （元空间）



- 堆
  
  - OOM的发源地，存储几乎所有的实例对象，
  
  - 可设置参数：Xmn（n=new，Eden和S1 S2），Xms，Xmx
  
  - 每个垃圾回收器默认的 Threashold 不同
  
    Parallel Scavenge：15
  
    CMS：6
  
    G1：15
  
  - 除了 Threashold 可以判断外，还有动态年龄判断，重新查阅
  
- 元空间

  - MetaSpace大小默认是无限的么? 
    
    MetaSpace大小默认没有限制，一般根据系统内存的大小。JVM会动态改变此值。

  - 可以通过什么方式来指定大小?

    `-XX:MetaspaceSize`：分配给类元数据空间（以字节计）的初始大小。此值为估计值，设置的过大会延长垃圾回收时间。垃圾回收过后，引起下一次垃圾回收的类元数据空间的大小可能会变大。 

    `-XX:MaxMetaspaceSize`：分配给类元数据空间的最大值，超过此值就会触发Full GC，此值默认没有限制，但应取决于系统内存的大小。JVM会动态地改变此值。 

  - 版本不同

    - 1.7之前是PermGen

      在启动时固定大小，很难进行调优，并且FGC是会移动类元信息

      字符串常量在这里

    - 1.8之后是MateSpace

      元空间在本地内存中分配

      会FGC，字符串常量放在堆里

  - 类元信息（klass）

  - 方法表，保存着每个可以实例化类的方法信息，JVM可以通过方法表快速地激活实例方法。

- 虚拟机栈

  - 每个方法（就是一个栈帧）从开始调用到执行完成的过程，就是栈帧从入栈到出栈的过程。在线程活动中，只有位于栈顶的帧才是有效的，称为当前栈帧。正在执行的方法称为当前方法，栈帧是方法运行的基本结构。

  - **局部变量表**

    - 局部变量表是存放方法参数和局部变量的区域。

  - **操作栈**

    - 操作展示一个初始状态为空的桶式结构栈。在方法执行过程中，会有各种指令往栈中写入和提取信息。

    - JVM的执行引擎是基于栈的执行引擎，其中的栈值得就是操作栈。

    - 描述操作站与局部变量表的交互：

      ```java
      public int simpleMethod(){
          int x = 13;
          int y = 14;
          int z = x + y;
          
          return z;
      }
      ```

      局部变量表就像一个中药柜，很多抽屉编号为1、2、3、4、5、....、n。

      1. `BIPUSH 13`先将常量加载到操作栈中，并保存到局部变量表中`ISTORE_1`。
      2. `14`也是同样的操作
      3. 分别把在局部变量表中的两个元素压入操作栈中，`ILOAD_1`、`ILOAD_2`
      4. `IADD` 把上方的两个数都取出来，在CUP 里加一下，并压回到操作栈的栈顶。
      5. 栈顶的结果存储到局部变量表中`ISTORE_3`
      6. 再重新`ILOAD_3`
      7. `IRETURN` 返回栈顶元素值

      

    - 值得注意的a=i++ 和a=++i 操作中   iinc：

      **a = i ++：**

      1. 把变量压入到操作栈栈顶
      2. 然后在抽屉里实现了+1的操作，而这个操作对在栈顶元素的值没有影响。所以只是把栈顶的值赋给了a。

      **a = ++i：**

      1. 在抽屉里 + 1了。所以不同

      延伸出一个信息，i++并不是原子操作。多个线程同时写的话，也会产生数据相互覆盖的问题。

    

  - **动态连接**

    每个栈帧中包含一个在常量池中对当前方法的引用，目的是支持方法调用过程的动态链接。

  - **方法返回地址**

    下面的看有点乱，马士兵说是，A() 里执行 B() 完成后，回到哪个地方。

    方法执行时有两种退出情况：
    
    - 第一，正常退出，即字节码指令：RETURN、IRETURN、ARETURN等
    - 第二，异常退出，无论何种退出情况，都将返回至方法当前被调用的位置。
    - 方法退出的过程相当于弹出当前栈帧，弹出可能有三种方式：
      - 返回值压入上层调用栈帧
      - 异常信息抛给能够处理的栈帧
      - PC计数器指向方法调用后的下一跳指令



- 本地方法栈
  - 为Native方法服务
  - 线程开始调用本地方法时，会进入一个不在受JVM约束的区域。
  - 本地方法可以通过JNI（Java Native Interface）来访问虚拟机运行时的数据区，甚至可以调用寄存器，具有和JVM相同的能力和权限。
  - 如果大量的本地方法出现时，势必会削弱JVM对系统的控制力，因为它的出错信息都比较黑盒。
  - 内存不足的情况，本地方法栈还是会抛出natice heap OutOfMemory
  
  什么是JNI也可简单的说一下吗？
  
  - JNI类本地方法，其中最著名的本地方法应该是`System.currentTimeMillis()`，JNI使Java深度使用操作系统的特性功能，复用非Java代码。
  - 但是在项目过程中，如果大量的使用其他语言来实现JNI，就会丧失跨平台性，威胁到程序运行的稳定性。
    - 解决思想：添加中间框架进行解耦
  
  总结：执行Java方法是使用虚拟机栈，执行Native方法时使用本地方法栈。
  
- 程序技术寄存器
  
  - 每个线程在创建后，都会产生自己的程序计数器和栈帧，程序计数器用来存放执行指令的偏移量和行号指示器等，线程执行或恢复都要依赖程序计数器。



## 2、Java源文件是如何转换成字节码的？

源文件经过词法解析，将形成token信息流，传递给语法解析器；语法解析器收到token信息流后，按照Java 语法规则组装成一颗语法树；之后在语义分析阶段，检查关键字的使用是否合理、类型是否匹配、作用域是否正确等；语义分析完成后，即可生成字节码。

字节码：

```
16进制
cafe babe 0000 0034  0014
   魔法值	小号 版本号 常量池内内容个数
```

可以写个简单的类练习一下。编译后打开class文件，可以用sublime，也可以用idea插件。根据马士兵的XMind去读16进制的的字节码。

往深点就是根据字节码进行java汇编，比如 (0x2a) aload_0，在 自己码中就是

## 3、字节码在JVM中经历了什么？

解释执行，JIT （JustInTime）编译执行

## 4、类加载过程

父加载器不是“ 类加载器的加载器 ”！也不是“ 类加载器的父类加载器 ”

复述书中的内容，够全

加载一类，从下往上询问是否加载过类，都查后发现自己的类加载器中并没有加载，然后就有要求子加载器此类。子加载器发现在自己里面并没有此class文件，那么就有向下推，直到加载上类。若找不到就抛出ClassNotFund

若想把一个类加载到内存中，调用对应的类加载器的 loadClass，返回一个 class 对象。也就是一层层的 findClass，找到就返回。

双亲委派模式：优点：

1. 避免类的重复加载 

2. 避免Java核心API被修改



## 5、怎么自定类加载器？

1. 继承`ClassLoader`

2. 重写`findClass()`方法

   源码中可以看到，protected 的 findClass 方法内只返回了一个ClassNotFund 异常，而需要重写补全功能。

   其中用到的是：模板方法。其他功能都实现了，就是中间部分需子类自己实现。

3. 调用方法中的`definClass()`方法。

   defineClass 最终把二进制流转换为 Class 类对象。

   ```java
   File f = new File(...);
   FileInputStream fis = new FileInputStream(f);
   ByteArrayOutStream baos = new ByteArrayoutStream();
   int b = 0;
   if (b = fis.read() != 0) {
       baos.write(b);
       byte[] bytes = baos.toByteArray();
       baos.close();
       fis.close();
   }
   return definClass(name, bytes, 0, bytes.length);
   
   mian(String[] args) {
       ClassLoader cl = new 刚定一个的类加载器();
       Class clazz = cl.getClass("全限定名");
       类名 test = (类名) clazz.getInstance();
       // 这样创建了一个 test对象，用到的是刚才自己创建的类加载器
   }
   
   ```

   

**插入一个不知道放哪里的 JVM 知识点**

## 缓存行（`caceh line`）

需要了解一下硬件层数据一致性，一致性的协议有很多种，因为 inter 用的是 MESI 协议，所以就了解这个即可。

`CPU`中每个缓存行（`caceh line`) 使用4种状态进行标记（使用额外的两位(`bit`)表示)

**MESI：**

- Modified	修改

  该缓存行只被缓存在该`CPU`的缓存中，并且是被修改过的，即与主存中的数据不一致，该缓存行中的内存需要在未来的某个时间点（允许其它`CPU`读取请主存中相应内存之前）写回主存。

  当被写回主存之后，该缓存行的状态会变成独享状态。

- Exclusive	独享

  该缓存行只被缓存在该`CPU`的缓存中，它是未被修改过的，与主存中数据一致。该状态可以在任何时刻当有其它`CPU`读取该内存时变成共享状态。

  同样地，当`CPU`修改该缓存行中内容时，该状态可以变成修改状态。

- Shared	共享

  该状态意味着该缓存行可能被多个`CPU`缓存，并且各个缓存中的数据与主存数据一致，当有一个`CPU`修改该缓存行中，其它`CPU`中该缓存行可以被作废（变成无效状态）。

- Invalid    无效

  该缓存是无效的（可能有其它`CPU`修改了该缓存行）。



正式讲一下缓存行的问题：提交的时候是一块内存的提交，**通常是64个byte**。

举个例子：

- 现在有两个变量 a和b 要被提交。
- a和b 在被提交前放在同一小块内存中，也就是64个byte，进行提交。
- 这时候只想要 a 变量的 cpu 的那一条线程中内存 b 被修改了，那么这一块内存就被标为 Invalid，那么这一块内存就不得不回去重读一下获取新的值。但是 b 的修改与只想要 a 变量的 cpu 无关，这么一来就浪费时间了。

改进方法：

```java
long p1, p2, p3, p4, p5, p6, p7;
long p8 = 1000;
long p1, p2, p3, p4, p5, p6, p7;
```

这么一来，p8无论是前边还是后边组合，都能合成独立的一块64字节的空间。



## 6、对象实例化

用最简单的例子举例`Object obj = new Object();`

字节码角度看：

- 通过javap -verbose -p命令可以看到对象创建的字节码

  ```java
  stact=2, locals=1, args_size=0
      NEW java/lang/Object
      DUP
      INVOKESPECIAL java/lang/Object.<init> ()V
      ASTORE_1
      LocalVariableTable:
  		Start	Length	Slot	Name	Signature
            8		   1	 0		 obj	Ljava/lang/Object
  ```

- NEW：如果找不到Class对象，则进行类加载。加载成功后，则在队中分配内存，从Object开始到本类路径上的所有属性值都要分配内存。==在分配过程中，注意引用是占据存储空间的，他是一个变量，占用4个字节。==

  ```
  refvar(Reference Variable)在面向对象世界中称为引用变量，或者叫引用句柄。是基本数据类型，默认值null
  把引用对象指向 实际对象(Refered Object) 简称refobj。
  ```

- 指令完毕后，将指向实例对象的引用变量压入虚拟机栈顶。

- DUP：在栈顶复制该引用变量，这是的栈顶的两个指向堆内实例对象的引用变量。两个引用变量的目的不同，其中压在底下的用于**赋值**，或者保存到局部变量表，另一个栈顶的引用变量作为**句柄**调用相关方法。

- INVOKESPECIAL：调用对象实例方法，通过栈顶的引用变量调用< init >方法。
  < clinit > 是类初始化执行的方法，而< init > 是对象初始化时执行的方法。

执行步骤角度来分析：

- 确认元信息是否存在

  在JVM 接收到new 的指令时，首先会在元空间（Metaspace）内检查需要创建的 类元信息 是否存在。
  如果不存在，那么在双亲委派模式下，使用当前类加载器，以 ClassLoader + 包名 + 类名 为Key 进行查找对应的.class文件。
  如果没有找到，则抛出`ClassNoFoundException`异常。
  如果找到，则进行类加载，并生成对应的Class类对象。

- 分配对象内存

  首先计算对象占用空间大小，如果实例成员变量是引用变量，仅分配引用变量空间即可，接着在堆中划分一块内存给新对象。
  在分配内存空间时，需要进行同步操作，比如采用CAS（Compare And Swap）失败重试、区域加锁等方式保证分配操作的原子性。

- 设定默认值

  成员变量值都是需要设定为默认值，即各种不同形式的零值。

- 设置对象头

  设置型对象的哈希码、GC信息、锁信息、对象所属的 类元信息 等。这个过程具体设置方式取决于JVM 实现。

- 执行 init 方法

  初始化成员变量，执行实例化代码块，调用类的构造方法，并把对内对象的首地址赋值给引用变量。





## 7、简述Java的对象结构（存储布局）

三部分组成：对象头、实例数据、对齐填充

然后就把这些知识点复述一遍





## 8、如何判断对象可以被回收的

- 根据GC Roots判断。

  如果一个对象与GC Roots 之间没有直接或间接的引用的引用关系

  - 比如某个失去任何引用的对象，或者两个互相环岛状循环引用的对象等，判断这些对象可被回收

- 什么对象可以作为GC Roots？

  - 类静态属性中引用的对象
  - 常量引用的对象
  - 虚拟机栈中引用的对象
  - 本地方法栈中引用的对象





## 9、垃圾回收的相关算法

- 最基础的“ 标记 - 清除算法 “ ：

  - 两遍扫描，在存活对象比较多的时候有效。
  - 该算法从每个GC Roots出发，依次标记有引用关系的对象，最后将没有被标记的对象清除。
  - 但这种算法会带来大量的空间碎片，导致需要分配一个较大连续空间时容易出发FGC。

- ” 标记 - 整理算法 ”：

  - 两遍扫描
  - 该算法类似计算机磁盘整理
  - 从GC Roots出发标记存活的对象，然后将存活对象整理到内容空间一端，形成连续的已使用空间，最后把已使用空间外的部分全部清除，这样就不会产生空间碎片的问题。

- “ Mark-Copy ”：

  堆中的一个回收方式

  现在作为主流的YGC算法进行新生代的垃圾回收

  这样一来还得重新给复制好的对象引用

- 分代搜集算法：

  根据各个年代的特点采用最适当的收集算法



先写进来，还不太理解

JVM 内存分代模型（用于垃圾回收算法）

- 部分垃圾回收器使用

  除 Epsilon、ZGC、shanandoah 之外的 GC，都是使用逻辑分代模型，G1是逻辑分代模型，物理不分代。除此之外不仅逻辑分代，且物理分代。



## 10、垃圾回收器

Serial、Serial Old、CMS、ParNew、Parallel Scavenge、Parallel Old、G1、ZGC、Shanandoah、Epsilon

常见三种组合：

1. Serial、Serial Old

2. Parallel Scavenge、Parallel Old 简称 PS + PO 没指定，默认开启

3. CMS、ParNew

   

“Stop The World”：简称STW，即垃圾回收的某个阶段会暂停整个应用程序的执行，当然不是直接停止，线程会找到safe point上停止。

FGC时触发的时间 STW 相对较长，频繁的FGC会严重影响应用程序的性能。

举例：

- 10G 内存用 PS + PO 来清：87W 耗十几秒
- CMS 到最后因为碎片问题触发的 FGC 长达十几小时



主讲Serial、CMS、G1三种。

- **Serial**：主要应用于YGC的垃圾回收器，采用串行单线程的方式完成GC任务。

- **CMS：**（Concurrent Mark Sweep Collector）是回收停顿时间比较短、目前比较常用的垃圾回收器

  通过：

  1. 初始标记（Initial Mark）
  2. 并发标记（Consurrent Mark）会很耗时
  3. 重新标记（Remark）STW时间较短，因为2操作
  4. 并发清除（Concurrent Sweep）

  四个步骤完成垃圾回收工作。

  - 第1、3步骤的依然会引发STW
  - 而2、4步骤可以和应用程序并发执行，也是比较耗时的操作，但不影响程序的正常执行。

  由于“CMS”是“标记-清除算法”，随意会**产生大量的空间碎片**。

  - 解决方法：可通过配置`-XX:+UseCMSCompactAtFullCollection`，强制JVM在FGC完成后对老年代进行压缩，执行一次空间碎片整理，但空间碎片整理阶段也会引发STW。
    - 再解决：减少STW的次数，CMS还可以通过配置`-XX:+CMSFullGCsBeforeCompactioin=n`参数，在执行n次FGC后，JVM再在老年代执行空间碎片整理。

  在最后并发清除的阶段，又产生了新的垃圾，被称为“ 浮动垃圾 ”，CMS + Serial Old 又是组合。出现：

  - 问题1：当 CMS 到后期不能动弹时候，Serial Old 这个单线程垃圾回收器，拿着扫把慢慢悠悠的扫天安门前的垃圾 

  - 问题2：并发清除期间新生成的浮动垃圾快塞满了Old，这时Servious 那边又来新对象塞不下了，就会请Serial Old 单线程老祖宗慢慢清理，清理期间会触发STW。这就是`PromotionFailed`晋升失败问题，

    解决方法：降低触发CMS的阈值，参数：-XX:CMSInitiatingOccupancyFraction 92% 把92降到50，60就可以。



- **G1：**HotSpot在JDK7 中推出了新一代G1（Garbage-First Garbage Collector）垃圾回收。

  - 采用“Mark-Copy”，非常好的空间整理能力，不会产生大量的空间碎片。

  - 通过`-XX:+UseG1GC`参数启动。

  - 与CMS相比，G1具备压缩用能，能避免面碎片问题，G1的暂停时间更加可控。

  - 运行原理：

    - G1将Java堆空间分割成了若干相同大小的区域，即region，包括Eden、Survivor、Old、Humongous（是特俗的Old类型，专门存放大型对象）四种类型。
    - 这样的划分方式意味着不需要一个连续的内存空间管理对象。
    - 优势：可预测停顿时间，能够尽可能快地在指定时间内完成垃圾回收任务。

  - 在JDK11，将G1设为默认垃圾回收器。

    



## 11、你知道哪些JVM性能调优命令

Sun JDK监控和故障处理命令有jps jstat jmap jhat jstack jinfo

1. jps：显示指定系统内所有的HotSpot虚拟机进程。 
2. jstat：用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机 进程中的类装载、内存、垃圾收集、JIT编译等运行数据。 
3. jmap：JVM Memory Map命令用于生成heap dump文件
4. jhat：是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了 一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看 
5. jstack：生成java虚拟机当前时刻的线程快照
6. jinfo：实时查看和调整虚拟机运行参数



- 设置堆内存大小`-Xms  -Xmx`，设置成一样大小，避免在GC后调整堆大小时带来的额外压力。
- 设置新生代大小，如果太小，那么会有大量的对象涌入老年区。





## 12、简述Java垃圾回收机制

在Java中，程序员是不需要显示的去释放一个对象的内存的，而是由虚拟机自行执行。在JVM中，有一 个垃圾回收线程，它是低优先级的，在正常情况下是不会执行的，只有在虚拟机空闲或者当前堆内存不 足时，才会触发执行，扫描那些没有被任何引用的对象，并将它们添加到要回收的集合中，进行回收



## 13、什么时候会触发FullGC

JVM内存不够的时候发生FGC 

1. 老年代空间不足
2.  CMS GC时出现promotion failed和concurrent mode failure
   - `promotionfailed`是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的
   - concurrent mode failure是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的
3.  统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间





## 14、如何启动系统的时候设置JVM的启动参数

其实很简单，比如采用“Java -jar”的方式启动一个jar包里的系统，那么就可以采用类似下面那的格式

`java -Xms512M -Xmx512M -Xss1M -XX:PermSize=128M -XX:MaxPermSize=128M -jar App.jar`





## 15、调优 JVM

-X:printGC



## 16、说说你所知道的几种参数





1. DCL需不需要volatile

   




# 多线程&并发

## 1、Java中实现多线程有几种方法

- 继承Thread类

- Runnable接口

- 实现Callable接口通过FutureTask（Runnable的实现类）包装器来创建Thread线程；

使用ExecutorService、Callable、Future实现有返回结果的多线程（也就是使用了ExecutorService来管理前面的三种方式）。 



#### 实现Runnable接口和Callable接口的区别

Callable（简单）与Runnable的区别：

1. 可以有返回值 
2. 可以抛出异常
3. 方法不同，run()/  call()

所以用Callable要好

```java
class MyThread implements Callable<String> {

    // 返回值与泛型相同
    @Override
    public String call() throws Exception {
        return "123";
    }
}

main { 
    MyThread my = new MyThread();
    // 实现Callable接口通过FutureTask（Runnable的实现类）包装器来创建Thread线程
    FutureTask<String> task = new FutureTask<>(myThread);
     // 结果会被缓存，效率高
    new Thread(task,"A").start();
    // 这个Get 方法可能会产生阻塞，把他方法最后
    String s = task.get();
}
```

Callable 不认识Thread 则通过Runnable

细节：
1、有缓存
2、结果可能需要等待，会阻塞！ 



## 2、如何停止一个正在运行的线程 

如何关闭一个线程？不要关闭线程，关闭线程的概念已经被否定了。所以就直接让它正常运行完就行。

1. 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop方法强行终止，但是**不推荐**这个方法，因为stop和suspend及resume一样都是过期作废的 方法。
3. 使用interrupt方法中断线程，使用会抛出异常。暂没见过用来控制业务逻辑的



## 3、notify()和notifyAll()有什么区别？

- notify可能会导致死锁，而notifyAll则不会 
- 任何时候只有一个线程可以获得锁，也就是说只有一个线程可以运行synchronized 中的代码 
- 使用notifyall,可以唤醒 所有处于wait状态的线程，使其重新进入锁的争夺队列中，而notify只能唤醒一个。
- notify() 是对notifyAll()的一个优化，但它有很精确的应用场景，并且要求正确使用。不然可能导致死锁。
- wait() 应配合while循环使用，不应使用if，务必在wait()调用前后都检查条件，如果不满足，必须调用 notify()唤醒另外的线程来处理，自己继续wait()直至条件满足再往下执行。



## 4、sleep()和wait() 有什么区别？

**1、来自不同的类** 

wait => Object
sleep => Thread 

在公司中一般都不用，用TimeUtil，在JUC包中

**2、关于锁的释放** 

`sleep()`方法导致了程序暂停执行指定的时间，让出cpu该其他线程，但是他的监控状态依然保持者，当 指定的时间到了又会自动恢复运行状态。在调用sleep()方法的过程中，线程不会释放对象锁.

调用`wait()`方法的时候，线程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用 notify()方法后本线程才进入对象锁定池准备，获取对象锁进入运行状态。

**3、使用的范围是不同的** 

wait：必须在Synchronize同步代码块中

sleep：任何地方都可以

**4、是否需要捕获异常**
wait   不需要捕获异常 

sleep 必须要捕获异常 





## 5、volatile 是什么?

计算机并不会根据代码顺序按部就班地执行相关指令。

在字节码层面：只加了ACC_VOLAILE

在 VM 层面：读写都加了层屏障

```java
StoreStroeBarrier		LoadLoadBarrier
	volatile 写			  volatile 读
StoreLoadBarrier		LoadStoreBarrier
```



volatile修饰之后，那么就具备了两层语义：

1. 保证了不同线程对这个变量进行操作时的可见性，volatile关键字会强制将修改的值立即写入主存。

   将可见性的时候，就要谈JMM

2. 禁止进行指令重排序。

   工作原理：**内存屏障**，可以保证避免指令重排的现象产生！

   invokespeclie 与 astroe_1 的重排，避免位于栈顶的句柄引用
   
   invoke是调用初始化init
   
   astroe是把对内的首地址赋给引用变量 
   
   

不保证原子性

一般用于 状态标记量 和 **单例模式的双检锁**（手写）

1. 为 uniqueInstance 分配内存空间 
2. 初始化 uniqueInstance 
3. 将 uniqueInstance 指向分配的内存地址 



### 5.1、什么是JMM?

java memory model

JMM ： Java内存模型，不存在的东西，概念！约定！ 



**关于JMM的一些同步的约定：** 

1. 线程解锁前，必须把共享变量立刻刷回主存。
2. 线程加锁前，必须读取主存中的新值到工作内存中！
3. 注意：加锁和解锁是同一把锁 



线程a要到主存中拿变量，并不会仅仅直接读取，而是拷贝一份到自己线程的工作内存中，真正操作的是自己拷贝的一份。
线程解锁，立马把共享变量刷回去。不会直接操作主存

线程：工作内存，主内存









## 6、volatile与synchronized的区别

- volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。 
- volatile仅能使用在变量级别；synchronized则可以使用在变量、方法 
- volatile仅能实现变量的修改可见性，并不能保证原子性；synchronized则可以保证变量的修改可见性和原子性。
-  volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。 



## 6、Thread 类中的start() 和 run() 方法有什么区别？

start()方法被用来启动新创建的线程，而且start()内部调用了run()方法 

和直接调用run()方法的效果不一样。

当你调用run()方法的时候，只会是在原来的线程中调用，没有新的线程启动，start()方法才会启动新线程。





## 7、常用辅组类（必会）

1. CountDownLatch 计数器 Down知道是减法，算倒计时

   允许一个或多个线程等待，知道在其他线程中执行的一组操作完成同步辅助

   ```java
   package com.kuang.add;
   import java.util.concurrent.CountDownLatch;
   // 计数器 
   public class CountDownLatchDemo {    
       public static void main(String[] args) throws InterruptedException {        
           // 总数是6，必须要执行任务的时候，再使用！        
           CountDownLatch countDownLatch = new CountDownLatch(6);
           for (int i = 1; i <=6 ; i++) {            
               new Thread(()->{                							  		 					System.out.println(Thread.currentThread().getName()+" Go out");       				  countDownLatch.countDown(); // 数量-1            
            },String.valueOf(i)).start();        }
           
           countDownLatch.await(); // 等待计数器归零，然后再向下执行
           
           System.out.println("Close Door");
       } 
   }
   ```

   原理：
   `countDownLatch.countDown();` // 数量-1

   `countDownLatch.await();` // 等待计数器归零，然后再向下执行
   每次有线程调用 countDown() 数量-1，假设计数器变为0，countDownLatch.await() 就会被唤醒，继续执行

2. CyclicBarrier  加法计数器

   ```java
   CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{         
       System.out.println("召唤神龙成功！"); 
   });                                                     
   // 等待数字到7的时候才激活                                                       
   cyclicBarrier.await();    
   // 激活后才会执行“召唤神龙”                                                        
   ```

   

3. Semmaphore：一个计数信号量。在概念上信号量维持一组许可证。如果有必要，每个acquire()都会阻塞，知道许可证可用，然后才能使用。

   在并发用的非常的多

   限制线程数量，限流。距离：停车位

   ```java
   public class Test {
       public static void main(String[] args) {
           // 线程数量：停车位! 限流！
           Semaphore semaphore = new Semaphore(3);
           for (int i = 1; i <=6 ; i++) {
               new Thread(()->{
                   // acquire() 得到
                   try {
                       semaphore.acquire();
                       System.out.println(Thread.currentThread().getName()+"抢到车位");
                       TimeUnit.SECONDS.sleep(2);
                       System.out.println(Thread.currentThread().getName()+"离开车位");
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   } finally {
                       semaphore.release();
                       // release() 释放
                   }
                },String.valueOf(i)).start();
           }
       }
   }
   ```

   原理：
   `semaphore.acquire(); `获得，假设如果已经满了，则等待，等待有线程被释放

   `semaphore.release(); `释放，会将当前的信号量释放 + 1，然后唤醒等待的线程
   作用： 多个共享资源互斥的使用！并发限流，控制大的线程数！
   
   定义了资源总量state=permits，当state > 0 时就能获得锁，并将state 减1，当state = 0 时只能等待其他线程释放锁，当释放锁时 state 加1，其他等待线程又能获得这个锁。当permits 定义为1 时，就是互斥锁，当permits > 1 就是共享锁。





## 8、读写锁

一个用于只写操作，一个用于只读操作

独占锁（写锁） 一次只能被一个线程占有 

共享锁（读锁） 多个线程可以同时占有 

```java
ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

readWriteLock.writeLock().lock();
readWriteLock.writeLock().unlock();

readWriteLock.readLock().lock();
readWriteLock.readLock().unlock();
```

* 读-读  可以共存
* 读-写  不能共存
* 写-写  不能共存



## 9、阻塞队列

一个队列：

- 写入：如果队列满了，就必须阻塞等待
- 获取：如果队列是空的，就必须阻塞等待生产 



阻塞

队列



**四组API**

 `ArrayBlockingQueue blockingQueue = new ArrayBlockingQueue<>(3);`

| 方式         | 抛出异常 | 有返回值，不抛出异常 | 阻塞 等待 | 超时等待    |
| ------------ | -------- | -------------------- | --------- | ----------- |
| 添加         | add      | oﬀer()               | put()     | oﬀer( , , ) |
| 移除         | remove   | poll()               | take()    | poll( , )   |
| 检测队首元素 | element  | peek                 | -         | -           |



SynchronousQueue 同步队列

- 进去一个元素，必须等待取出来之后，才能再往里面放一个元素！
- ` BlockingQueue<String> blockingQueue = new SynchronousQueue<>(); `// 同步队列
- 存：`blockingQueue.put("1")` 取：`blockingQueue.take()`





## 10、线程池

线程池：5大方法、7大参数、4种拒绝策略

> 线程池：5大方法

```java
// 单个线程 
ExecutorService threadPool = Executors.newSingleThreadExecutor();	
// 创建一个固定的线程池的大小 
ExecutorService threadPool = Executors.newFixedThreadPool(5); 
// 可伸缩的，遇强则强，遇弱则弱，依赖于操作系统能够创建的大线程大小
ExecutorService threadPool = Executors.newCachedThreadPool(); 	
// 与上相同，线程池支持定时以及周期性执行任务的需求。 
ExecutorService threadPool = Executors.newScheduledThreadPool();
// JDK8引入，创建持有足够线程的线程池支持给定的并行度，并使用多个对象减少竞争。
ExecutorService threadPool = Executors.newWorkStealingPool();
 
// 关闭线程
threadPool.shutdown();
```

> 七大参数

```java
public ThreadPoolExecutor(int corePoolSize,			// 核心线程池大小 
                          int maximumPoolSize,		// 大核心线程池大小 
                          long keepAliveTime,		// 超时了没有人调用就会释放 
                          TimeUnit unit,			// 超时单位 
                          BlockingQueue<Runnable> workQueue,// 阻塞队列 
                          ThreadFactory threadFactory,// 线程工厂：创建线程的，一般不用动 
                          RejectedExecutionHandler handler) {// 拒绝策略
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
        null :
    AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```



> 四大拒绝策略

```java
new ThreadPoolExecutor.AbortPolicy() // 银行满了还有人进来，不处理这个人的，抛出异常 
new ThreadPoolExecutor.CallerRunsPolicy() // 哪来的回哪去！ 
new ThreadPoolExecutor.DiscardPolicy() //队列满了，丢掉任务，不会抛出异常！ 
new ThreadPoolExecutor.DiscardOldestPolicy() //队列满了，尝试去和早的竞争，也不会抛出异常！ 
```





### 10.1、简述一下你对线程池的理解

（如果问到了这样的问题，可以展开的说一下线程池如何用、线程池的好处、线程池的启动策略）

- 合理利用线程池能够带来三个好处：

  1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
  2. 提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。
  3. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系 统的稳定性，使用线程池可以进行统一的分配，调优和监控。

  







## 11、为什么wait, notify 和 notifyAll这些方法不在thread类里面？ 

- 明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。
- 线程需要等待 某些锁 那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不清除了
- 简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中，因为锁属于对象。 





## 12、为什么wait和notify方法要在同步块中调用？ 

1. 只有在调用线程拥有某个对象的独占锁时，才能够调用该对象的wait(),notify()和notifyAll()方法。
2. 如果你不这么做，你的代码会抛出IllegalMonitorStateException异常。
3. 还有一个原因是为了避免wait和notify之间产生竞态条件。

问题8，没写完，没看懂



## 13、Java中interrupted 和 isInterrupted方法的区别？ 

interrupted() 和 isInterrupted()的主要区别是前者会将中断状态清除而后者不会。

Java多线程的中断机 制是用内部标识来实现的，调用Thread.interrupt()来中断一个线程就会设置中断标识为true。

当中断线程调用静态方法Thread.interrupted()来检查中断状态时，中断状态会被清零。

而非静态方法 isInterrupted()用来查询其它线程的中断状态，且不会改变中断状态标识。





## 14、说一说自己对于 synchronized 关键字的了解 

synchronized关键字解决的是多个线程之间访问资源的同步性，它可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。 

在 Java 早期1.2版本中，synchronized属于重量级锁，效率低，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统OS帮忙完成。而操作系统实现线程之间的切换时需要从用户态转换到内核状态，这个状态之间的转换需要较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。

在 JDK6 后不断优化是的 synchronized 提供三种锁的实现，包括偏向锁、轻量级锁、重量级锁，还提供自动的升级和降级机制。

**偏向锁**，并不是锁，是一种优化，标签。--永远偏向第一个加锁的对象。

- 因为像Vactor、StringBuffer、HashTable大多情况下只有单条线程来访问，所以 出现锁竞争的情况很少，那么就干脆的设置个偏向锁，省去竞争。所以是一种优化。

当前线程是第一个执行sync同步方法时候，对方法贴上自己的标签也就是把自己的线程id写到对象头上markword，那么下次以来便可在直接使用，省的再申请锁，这样少了很多锁竞争的过程。一旦有其他的线程过来，那么在外阻塞的线程会把标记抹掉。此时，会升级到轻量级锁--自旋锁。如果说有线程在方法内停留了很久，而在外的线程一直自旋，然而while是很消耗的资源的，所以在

1.6之前两种方式升级重量级：

- 自旋超过10次，可通过JVM调优参数修改 `-XX:PreBlockSpin`
- 等待的线程超过CUP核的二分之一。例：CPU32核，等待已经16个。现在又来了一个，那么升级。

1.6之后增加了一个：自适应自旋 Adapative Self Spinning

- 自旋就不用再自己设置，由JVM自己决定。



**偏向锁的延迟打开理论**，4秒后打开，没打开上自旋锁，打开上偏向。

- 因为在JVM启动时分配内存的时候默认会有11个线程在争夺，那么这是上偏向锁就是浪费资源。所以等4秒，让线程都完成工作后在打开偏向锁。

匿名偏向，虽然已经是偏向锁，但其实是一个匿名对象。



#### 打开偏向锁是否会提高效率？为什么？

不一定会提高，因为产生锁竞争时发生偏向锁的锁撤销，会消耗资源。

当明确有很多的线程会对同一个方法产生竞争时，就没必要打开偏向锁。



#### 轻量级锁一定比重量级锁效率高吗？

自旋时的while必然会造成资源消耗，所以在线程多且执行时间长的情况下，轻量级锁效率低，重量级则就很不错。

相反，线程少执行短，那么轻量级效率高，重量级低。



### 14.1、项目中用到了吗synchronized关键字最主要的三种使用方式

- **修饰实例方法:** 作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁
- **修饰静态方法:** 访问静态 synchronized 方法占用的锁 是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁
- **修饰代码块:** 指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。 



## 15、Java中synchronized 和 ReentrantLock 有什么不同？ 

相似点：都是加锁方式同步，而且都是阻塞式的同步

- 当如果一 个线程获得了对象锁，进入了同步块，其他访问该同步块的线程都必须阻塞在同步块外面等待，而进行线程阻塞和唤醒的代价是比较高的。

区别：

- Synchronized：---其中monitor是监视锁
  - 是java语言的关键字，是原生语法层面的互斥，需要jvm实现。
  - 进过编译，会在**同步代码块**的前后分别形成 monitorenter 和 monitorexit 这个两个字节码指令，然后执行两指令获取和释放monitor。
    - 如果使用 monitorenter 进入时 monitor 为 0，标识该线程可以持有 monitor 后续代码，并将monitor 加1；如果当前线程已经持有了monitor，那么monitor 继续加1；如果monitor 非0，其他线程就会进入阻塞状态。
    - 两个 monitorexit，一个是正常退出，另一个遇到异常退出。
  -  修饰方法的的情况，**同步方法**使用ACC_SYNCHRONIZED来标识，而没有monitorenter和monitorexit。JVM 通过访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。
- ReentrantLock：需要lock()和unlock()方法配合try/ﬁnally语句块来完成。
  - JUC报下提供的互斥锁，相比Synchronized，RenntrantLock类提供了一些高级功能。
  - 等待可中断，持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于 Synchronized来说可以避免出现死锁的情况。 
  - 公、非公平锁，多个线程等待同一个锁时，必须按照申请锁的时间顺序获得锁，Synchronized锁非公平锁， ReentrantLock默认的构造函数是创建的非公平锁，可以通过参数true设为公平锁，但公平锁表现的性 能不是很好。

- Synchronized  内置的Java关键字，Lock 是一个Java类 
- Synchronized  无法判断获取锁的状态，Lock  可以判断是否获取到了锁
- Synchronized  会自动释放锁，lock 必须要手动释放锁！如果不释放锁，死锁
- Synchronized  线程 1（获得锁，阻塞）、线程2（等待，傻傻的等）；Lock锁就不一定会等待下去
- Synchronized  可重入锁，不可以中断的，非公平；Lock 可重入锁，可以判断锁，非公平（可以自己设置）；
- Synchronized  适合锁少量的代码同步问题，Lock  适合锁大量的同步代码！ 



## 16、有三个线程T1,T2,T3,如何保证顺序执行？ 

有很多种方式可以保证，其中一种用Join。在线程3中`t2.join`，在线程2中`t1.join`





## 17、SynchronizedMap和ConcurrentHashMap有什么区别

- SynchronizedMap()和Hashtable一样，实现上在调用map所有方法时，都对整个map进行同步。

- ConcurrentHashMap的实现却更加精细，它对map中的所有桶加了锁。所以，只要有一个线程访问 map，其他线程就无法进入map，而如果一个线程在访问ConcurrentHashMap某个桶时，其他线程， 仍然可以对map执行某些操作。 

  在遍历map时，如果其他线程试图对map进行数据修 改，也不会抛出ConcurrentModiﬁcationException。





## 18、什么是线程安全

- 线程安全就是说多线程访问同一代码，不会产生不确定的结果。

- 在多线程环境中，当各线程不共享数据的时候，即都是私有（private）成员，那么一定是线程安全的。 但这种情况并不多见。

- 代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运 行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。





## 19、Thread类中的yield方法有什么作用？

- yield方法可以暂停当前正在执行的线程对象，让其它有相同优先级的线程执行。

- 它是一个静态方法，只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行，因为没有优先级高的线程执行。





## 20、Java线程池中submit() 和 execute()方法有什么区别？

两个方法都可以向线程池提交任务。

- `execute()`方法的返回类型是void，它定义在Executor接口中。

-  `submit()`方法可以返回持有计算结果的Future对象，它定义在ExecutorService接口中，它扩展了 Executor接口。

  其它线程池类像ThreadPoolExecutor和ScheduledThreadPoolExecutor都有这些方 法





## 21、如何手动创建一个线程池

开发手册建议使用 `ThreadPoolExecutor`来创建线程池，目的是让大家能更好的使用线程池。

过去的`Executors`局限性大。

配置对应的7大参数即可。





## 22、CAS

CAS ： 比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么则执行操作！如果不是就 一直循环

缺点：
1、 循环会耗时
2、一次性只能保证一个共享变量的原子性3、ABA问题

 

