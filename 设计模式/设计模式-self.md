# 设计模式

所有的设计模式到大成时，将融汇贯通

## 1. 单例模式

## 2. 策略模式

## 3. 工厂模式

## 4. 门面者模式Facade

![image-20221103220233567](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103220233567.png)

对外封装同一个接口，供外人调用

### 调停者模式Mediator

![image-20221103215511499](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103215511499.png)

在实际中有大用：消息消息中间件

门面者与调停者相似



## 5. 装饰器

![image-20221103221003731](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103221003731.png)

`new RectDecorator(bullet)`起到装饰作用

## 6. 责任链

Chain Of Responsibility

倒数第二难

### 1. 初步描述问题

1. 制造场景<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103221757310.png" alt="image-20221103221757310" style="zoom:50%;" />  

2. 代码简单实现

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103222048692.png" alt="image-20221103222048692" style="zoom:50%;" />

   其中问题：发表的言论中如果存在尖括号，有可能会执行代码，为了避免这种情况，需要将文本进行处理；

   ![image-20221103222339137](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103222339137.png)

   现在是将文本进行处理了，但是未来还需要处理其他的文本，就得进行扩展，如何操作便于日后扩展 - 简单的东西复杂化，建立个过滤器

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103222941042.png" alt="image-20221103222941042" style="zoom:50%;" />

   有需要什么类型的过滤器就创建

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103223141122.png" alt="image-20221103223141122" style="zoom:50%;" />

   然后创建功能类调用处理文本

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103223215442.png" alt="image-20221103223215442" style="zoom:50%;" />

   现在将调用方式进行调整 - 把这些类串起来

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103223503424.png" alt="image-20221103223503424" style="zoom:50%;" />

   形成图片这种情况

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103223540164.png" alt="image-20221103223540164" style="zoom:50%;" />  

​		

### 2. 形成初步的责任链

1. 将上一个简单的例子进行问题处理，重新写一个类`FilterChain`

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103224558537.png" alt="image-20221103224558537" style="zoom:50%;" />

   Main的调用为

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221103225032036.png" alt="image-20221103225032036" style="zoom:50%;" />

   ==区别== ：如果存在另一个FilterChain，那么他们是能够连在一起

   小技巧：

   ```java
   public FilterChain add(Filter f) {
     	filters.add(f);
     	return this;
   }
   
   main() {
       FilterChain fc = new FilterChain();
     	fc.add(new HTMLFilter()).add(new SensitiveFilter());
   }
   
   // 返回自身可以进行链式调用
   ```

   若想与另一条链条连在一起，步骤：

   1. 增加新链条

      <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221104102414140.png" alt="image-20221104102414140" style="zoom:50%;" />

      这么做有点蠢，那么做些改动，让链条自己再实现下过滤器

      <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221104102704147.png" alt="image-20221104102704147" style="zoom:50%;" />

      然后调用处就能直接将添加另一个链条

      <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221104103040533.png" alt="image-20221104103040533" style="zoom:50%;" />





### 3. 进一步要求

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221104104833714.png" alt="image-20221104104833714" style="zoom:50%;" />

1. 其中重点问题是，由Filter判断是否再继续执行

2. 对Filter进行改造

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221104105556401.png" alt="image-20221104105556401" style="zoom:50%;" />

   - 当链条中过滤器返回ture，那么就继续执行
   - 当链条中过滤器返回false，那么就终止

   调用处做的修改为：

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221104110043027.png" alt="image-20221104110043027" style="zoom:50%;" />

   



### 4. 正式责任链

从java.serverlt中要求的责任链的模式是 request请求经过过滤器 1，2，3处理后到达后端，之后返回的response经过3，2，1 处理

可用递归处理

- 在filterChain中加入位子的处理，同时在filter中加入第三个参数

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221105235928126.png" alt="image-20221105235928126" style="zoom:50%;" />

---

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106000100892.png" alt="image-20221106000100892" style="zoom:50%;" />

---

![image-20221106000244364](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106000244364.png)

`chain.doFilter`递归调用；request先处理完成后，进入下一个递归。等到最终返回执行完所有就是倒着处理response



## 观察者

Observer

事件处理模型

### 1. 初步模拟

问题：小朋友睡醒了哭，饿！

- 版本一

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106001206574.png" alt="image-20221106001206574" style="zoom:50%;" />

  直接系统观察小孩，看小孩什么时候哭，哭了就处理

- 版本二

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106001554601.png" alt="image-20221106001554601" style="zoom:50%;" />

  看到需求抽象出小孩类定义方法

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106001654227.png" alt="image-20221106001654227" style="zoom:50%;" />

  主程序也是在一直观察，直到有人调用了方法`wakeUp`

- 版本三

  加入一个观察者

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106002040052.png" alt="image-20221106002040052" style="zoom:50%;" />

  Dad

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106002119514.png" alt="image-20221106002119514" style="zoom:50%;" />

  现在情况是，当小孩醒了后，Dad会喂小孩

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106002230255.png" alt="image-20221106002230255" style="zoom:50%;" />

- 版本四

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106002630387.png" alt="image-20221106002630387" style="zoom:50%;" />

  类、方法、调用 同上

  其中问题来了：

  - 再加入新的观察者，那么又要再Dog后面new新的类，方法在调用。写死的方式不好，扩展性也不好

  - 观察者做出的反应不一定得耦合到某一个特定的被观察者身上，例如：狗不一定就看到小孩哭了就叫，也可以看到其他的事务会叫

  

- 版本五

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106003318166.png" alt="image-20221106003318166" style="zoom:50%;" />

  定义一个观察者接口，为醒来的动作做出行动的方法

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106003423068.png" alt="image-20221106003423068" style="zoom:50%;" />

  爸爸妈妈分别实现接口

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106003600735.png" alt="image-20221106003600735" style="zoom:50%;" />

  小孩做的处理

  这里能发现与责任链有些相似：

  - 当有事件触发后，观察者list一次执行。若是其中的一个观察者停止了，就变成了责任链

    <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106003923444.png" alt="image-20221106003923444" style="zoom:50%;" />

  责任链能中断的，但观察者一般是不能的

  

- 版本六

  观察者模式刚刚开始

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106155035305.png" alt="image-20221106155035305" style="zoom:50%;" />

  需要将事件的本身抽象出来 - Event

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106153402915.png" alt="image-20221106153402915" style="zoom:50%;" />

  那么当小孩醒的时候，就会发出这个事件

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106153747248.png" alt="image-20221106153747248" style="zoom:50%;" />



- 版本七

  很多时候观察者需要根据事件的具体情况来进行处理，处理事件的时候，需要事件源对象。

  那么事件源对象如何传递给观察者

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106160214978.png" alt="image-20221106160214978" style="zoom:50%;" />

  ---

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106160941814.png" alt="image-20221106160941814" style="zoom:50%;" />

  在以上的基础上增加个数据源 source ，看是否处理这个原数据



- 版本八

  在以上基础上，事件也可以形成继承关系

  简单的模拟下情况

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106161503153.png" alt="image-20221106161503153" style="zoom:50%;" />

  ---

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106161544580.png" alt="image-20221106161544580" style="zoom:50%;" />



- 版本九

  直接看 Java 内部是怎么应用观察者模式的

  ![image-20221106161819608](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106161819608.png)

  ---

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106163557342.png" alt="image-20221106163557342" style="zoom:50%;" />

  ---

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106163935623.png" alt="image-20221106163935623" style="zoom:50%;" />

  ---

  这里简单讲了下Hook callBack Observer Listener 其实都是一回事





## 组合模式

Composite

树状结构专用模式

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106202548499.png" alt="image-20221106202548499" style="zoom:50%;" />

模拟简单情况

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106202707584.png" alt="image-20221106202707584" style="zoom:50%;" />

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106203722261.png" alt="image-20221106203722261" style="zoom:50%;" />

遍历以上的树结构

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106204332604.png" alt="image-20221106204332604" style="zoom:50%;" />



## 享元模式

Flyweight

重复利用对象 - 共享元数据

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106230002889.png" alt="image-20221106230002889" style="zoom:50%;" />

可以联想到 字符串的创建

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106230551705.png" alt="image-20221106230551705" style="zoom:50%;" />

## 代理

Proxy

### 静态代理

简单模拟情况

1. 版本一

   描述问题

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106231629343.png" alt="image-20221106231629343" style="zoom:50%;" />

2. 版本二

   简单的做出修改，加上系统时间，相减得出

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106231819308.png" alt="image-20221106231819308" style="zoom:50%;" />

3. 版本三

   在此基础上进行继承，然后调用父方法解决不能修改源代码的问题

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106232211927.png" alt="image-20221106232211927" style="zoom:50%;" />

   但慎用继承 - 耦合度太高

4. 版本四

   利用代理

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221106232910461.png" alt="image-20221106232910461" style="zoom:50%;" />

   上方简单的修改下，在`new TankTimeProxy()`的时候把 tank 传进来，代理类的构造函数去接这个 tank。

   创建代理类实现 Movable，进行聚合。那么有关Movable 接口的类都可以进行代理

5. 版本五

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107001529580.png" alt="image-20221107001529580" style="zoom:50%;" />

   可以在以上基础上，实现多种代理，而不动到源代码

6. 版本六

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107003312403.png" alt="image-20221107003312403" style="zoom:50%;" />

   将引用改成 Movable 

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107003521714.png" alt="image-20221107003521714" style="zoom:50%;" />

   达到这样的目的是为了什么 - 代理做嵌套：先记录时间后再输出日志，或者先日志再时间

   <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107004013674.png" alt="image-20221107004013674" style="zoom:50%;" />

   这么调用即可

### 动态代理

- 版本七

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107004721431.png" alt="image-20221107004721431" style="zoom:50%;" />

  描述现在问题：以上版本的代理只能代理 Movable 接口的类，没办法实现所有的接口都可以代理。

  如何解决：这种请款就进行动态代理 - 代理的实体类都不是自己手写的

  实现：

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107154019580.png" alt="image-20221107154019580" style="zoom:50%;" />

  与前期的版本不同是在调用处利用Proxy类进行反射获取

  执行完 `m.move()`后，Handler里的`invoke`也被触发了，不过执行顺序是：先输出日志，在执行tank，再输出日志

  ---

  可把Handler单独取出来

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107155918072.png" alt="image-20221107155918072" style="zoom:50%;" />

- 版本八

  但是以上为什么执行 `m.move()`后会执行`invoke`方法？

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107160103481.png" alt="image-20221107160103481" style="zoom:50%;" />

  为了搞清楚后面执行了什么，配置输出文件反编译文件

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107162125256.png" alt="image-20221107162125256" style="zoom:50%;" />

  ---

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107163904018.png" alt="image-20221107163904018" style="zoom:50%;" />

  ---

  图解释

  <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107164117962.png" alt="image-20221107164117962" style="zoom:50%;" />

  其中：Proxy -> Proxy$0如何生成。 asm包



除了自带的动态，还可以用带cglib包的intercepter生成

 

## 迭代器

Iterator

容器与容器遍历

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107210041765.png" alt="image-20221107210041765" style="zoom:50%;" />

自己模拟写一个ArrayList、LinkedList

如果想两者达到替换效果，那么共同实现接口Collection

---

如果想达到

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107214511437.png" alt="image-20221107214511437" style="zoom:50%;" />

创建一个实现Iterator接口的内部类

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107214640850.png" alt="image-20221107214640850" style="zoom:50%;" />

实际调用



<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107215007960.png" alt="image-20221107215007960" style="zoom:50%;" />

## 访问者

Visitor （工作一般用不到）

在结构不变的情况下动态改变对于内部元素的动作



暂不听



## 构建器

Builder

构建复杂对象

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107230610333.png" alt="image-20221107230610333" style="zoom:50%;" />



理解：

- 就是有一个复杂的类，有50个参数需要构造函数初始化，那么就很麻烦。

- 解决方法就是分批构造

  1. 首先创建个接口

     <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107231132661.png" alt="image-20221107231132661" style="zoom:50%;" />

  2. 然后实现接口

     <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107231215263.png" alt="image-20221107231215263" style="zoom:50%;" /> 

  3. 初始化

     <img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107231506809.png" alt="image-20221107231506809" style="zoom:50%;" />

---

另一个例子

<img src="/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221107231907137.png" alt="image-20221107231907137" style="zoom:50%;" />

```java
// 继上没截全的代码
public Person build() {
    return p;
}
```



这个Person类有很多参数，每次new的时候构造函数得写一堆，有时候不传还得空着，麻烦

图中就创建了静态内部类，专门来创建各种情况的构造函数。



## 适配器

Adapter

接口转换器

如果一个类不能直接访问另一个类的时候，中间加一层类，那就是转换器



## 桥接







## 命令

## 原型

## 备忘录

## 模版方法

## 状态

## 解释器

