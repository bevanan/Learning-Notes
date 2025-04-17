#  Spring源码

学习想法

- 目前刚开始学习，并不太清楚每个问题所在，所以先以截图方式记下源码步骤，额外再旁边记录补充



## 一、Spring概述

![image-20221114190153159](./Spring源码.assets/image-20221114190153159.png) 



配合同名图片学习。

重点需要记住一些接口的作用：

1. BeanFactory

   用于访问Springbean容器的根接口。这是bean容器的基本客户端视图；其他接口，如ListableBeanFactory和ConfigurableBeanFactory 可配置BeanFactory可用于特定用途。

   - ListableBeanFactory

     BeanFactory接口的扩展将由可以枚举其所有bean实例的bean工厂实现，而不是按照客户机的请求逐个尝试按名称查找bean。

   - ConfigurableBeanFactory

     配置接口将由大多数bean工厂实现。除了BeanFactory接口中的bean工厂客户端方法之外，还提供了配置bean工厂的工具。

2. Aware 感知

   是很多的的父接口，例如 ApplicationContextAware、EnvironmentAware 

   作用是，对应的普通Bean对象实现了ApplicationContextAware的话，那么bean能getApplicationContext()信息

3. BeanDefinition

4. BeanDefinitionReader - 读取配置信息？

5. BeanFactoryPostProcessor - 增强beanDefinition信息

6. BeanPostProcessor - 增强bean信息 （有 before，after）

7. Environment - StandardEnviroment - 1. System.getEnv(); 2. System.getProperties();

8. FactoryBean

9. DefaultListableBeanFactory 看他类图







## 二、Debug Spring流程概述

先准备一个简单类

1. 开始debug，F7

<img src="./Spring源码.assets/image-20221114191323923.png" alt="image-20221114191323923" style="zoom:70%;" />

2. 为WebLogic的小问题，忽略 F7

<img src="./Spring源码.assets/image-20221114192226409.png" alt="image-20221114192226409" style="zoom:70%;" />

3. 回到最开始，F7

   <img src="./Spring源码.assets/image-20221114191323923.png" alt="image-20221114191323923" style="zoom:70%;" />

4.  F7进入构造方法

   ![image-20221114193837225](./Spring源码.assets/image-20221114193837225.png)

5. F7到重构去

   ![image-20221114193952034](./Spring源码.assets/image-20221114193952034.png)

   这里分为3路了

### super(parent)

   调用父类构造，做准备工作，创建集合对象，做一些初始化

   

### setConfigLocations(configLocations)

   设置配置路径

6. F8过super ，可读到配置文件路径所在位置
    ![image-20221114194524888](./Spring源码.assets/image-20221114194524888.png)

   

### refresh()


   	弄懂这里面的十几个方法，就能知道总体流程了

7. F8跳过设置配置路径 到 `refresh()`F7进入方法核心步骤流程

   ![image-20221114195430179](/Users/zhengbufeng/Library/Application Support/typora-user-images/image-20221114195430179.png)

   ```java
   	@Override
   	public void refresh() throws BeansException, IllegalStateException {
   		synchronized (this.startupShutdownMonitor) {
   			// 准备刷新，做一些最基本的准备工作
   			prepareRefresh();  
         --------------------------👇
         		protected void prepareRefresh() {
   							this.startupDate = System.currentTimeMillis();// 设置开始时间
   							this.closed.set(false);		// 设置关闭位	
   							this.active.set(true);		// 设置活跃标志位
   
   							if (logger.isDebugEnabled()) {
   									if (logger.isTraceEnabled()) {
   											logger.trace("Refreshing " + this);
   									}
   									else {
   											logger.debug("Refreshing " + getDisplayName());
   									}
   							}
   							// 里面是空，目的是为了做些扩展
   							initPropertySources();		
           					// 获取环境对象；验证环境属性    知识点：创建子类，父类也会跟着创建
   							getEnvironment().validateRequiredProperties();		
   
           					// 设置一些监听器和事件集合对象
   							if (this.earlyApplicationListeners == null) {
   									this.earlyApplicationListeners 
                       = new LinkedHashSet<>(this.applicationListeners);
   							}
   							else {
   							// Reset local application listeners to pre-refresh state.
   									this.applicationListeners.clear();
   									this.applicationListeners.addAll(this.earlyApplicationListeners);
   							}
   
   							// Allow for the collection of early ApplicationEvents,
   							// to be published once the multicaster is available...
   							this.earlyApplicationEvents = new LinkedHashSet<>();
   					}
         --------------------------
        
   
   ```



#### obtainFreshBeanFactory

7. F8跳过`prepareRefresh() ` 到 `obtainFreshBeanFactory()`

   ![image-20221114215658596](./Spring源码.assets/image-20221114215658596.png)

8. F7步入方法`obtainFreshBeanFactory()`


   ```java
   // 告诉子类刷新内部bean工厂。最后创建容器对象：DefaultListableBeanFactory，并加载xml配置文件的属性值到当前工厂中，其中最重要的就是beanDefinition
   ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   --------------------------                        👇
       	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
   				refreshBeanFactory();
   ---------------------------------------👇------------- 
       			// 只要进入到这方法里面，就是创建一个新的工厂
   				protected final void refreshBeanFactory() throws BeansException {
       					// 判断是否存在工厂并销毁
   						if (hasBeanFactory()) {
   							destroyBeans();
   							closeBeanFactory();
   						}
   						try {
                   			// 最终实例创建
   							DefaultListableBeanFactory beanFactory = createBeanFactory();
   							beanFactory.setSerializationId(getId());	// 设置序列化Id
   							customizeBeanFactory(beanFactory);			// 自定义BeanFactory一些权限
   							loadBeanDefinitions(beanFactory);			// 加载bean的定义属性值
   							synchronized (this.beanFactoryMonitor) {
   								this.beanFactory = beanFactory;
   							}
   						}
   						catch (IOException ex) {
   							throw new ApplicationContextException("I/O error parsing bean "+
                                                             "definition source for " + 
                                                             	getDisplayName(), ex);
   						}
   				}
   ---------------------------------------------------- 
       
       	return getBeanFactory();
   }
   --------------------------
   ```

10. 经过以上创建到`DefaultListableBeanFactory`，F8跳到`customizeBeanFactory(beanFactory)`F7进入方法

    ![image-20221114225248862](./Spring源码.assets/image-20221114225248862.png) 

    作用：设置一些属性值

11. 设置好后，返回到下一个方法`loadBeanDefinitions(beanFactory);`（很复杂，因为有很多个重载的方法需要被调用，但每一次重载所传入的参数是不一样的）

    目前可知，经历过这个方法后，假如配置文件配置了一个bean，那么一下图就有对应有个

    <img src="./Spring源码.assets/image-20221114231549531.png" alt="image-20221114231549531" style="zoom:50%;" /> 

    ![image-20221115195826673](./Spring源码.assets/image-20221115195826673.png)  

    后期课程讲解这个复杂的方法

    

12. 执行完后回到方法`obtainFreshBeanFactory`总结：

    ```java
    // 告诉子类刷新内部bean工厂。最后创建容器对象：DefaultListableBeanFactory，并加载xml配置文件的属性值到当前工厂中，其中最重要的就是beanDefinition
    ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
    ```



#### prepareBeanFactory

10. F7到下一个方法

    <img src="./Spring源码.assets/image-20221114233325375.png" alt="image-20221114233325375" style="zoom:50%;"/> 

    方法的作用是为了对beanFactory的一些初始化操作

    点进方法内可看到set、add、register等操作

11. F8到下一个方法

    <img src="./Spring源码.assets/image-20221114233752353.png" alt="image-20221114233752353" style="zoom:50%;"/> 

    子类覆盖方法做额外的处理,此处我们自己一般不做任何扩展工作,但是可以查看web中的代码,是有具体实现的

    空方法，做扩展用

12. F8到下一个方法

    <img src="./Spring源码.assets/image-20221114234059590.png" alt="image-20221114234059590" style="zoom:50%;" /> 

    开始执行BFPP 这些容器对象了，进入方法

    <img src="./Spring源码.assets/image-20221114234735585.png" alt="image-20221114234735585" style="zoom:50%;"/> 

    `PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());`太复杂了，后期课程讲解

    if判断为空，则直接返回结束方法了

    以上已经将流程1、2、3 完成了

    <img src="./Spring源码.assets/image-20221115000058975.png" alt="image-20221115000058975" style="zoom:40%;" />

13. 到下一个方法开始准备BPP容器对象的准备工作了

    <img src="./Spring源码.assets/image-20221114234948076.png" alt="image-20221114234948076" style="zoom:50%;" /> 

    F7进入方法，注释：实例化并调用 

    <img src="./Spring源码.assets/image-20221115000334579.png" alt="image-20221115000334579" style="zoom:50%;" /> 





 17. 下一步到`initMessageSource()`

     ![image-20221115182527602](./Spring源码.assets/image-20221115182527602.png) 

     做国际化操作，i18n

 18. F8跳过，到`initApplicationEventMulticaster()`

     ![image-20221115183011494](./Spring源码.assets/image-20221115183011494.png) 

     初始化应用程序事件的广播器

 19. F8跳过，到`onRefresh()`

     ![image-20221115183443486](./Spring源码.assets/image-20221115183443486.png) 

     为空方法，做扩展用；在SpringBoot中有这个方法里有执行tomcat

 20. F8跳过，到`registerListeners()`，注册监听器

     ![image-20221115183738691](./Spring源码.assets/image-20221115183738691.png) 

     


#### finishBeanFactoryInitialization

17. F8跳过，到`finishBeanFactoryInitialization(beanFactory)`，实例化所有剩余的（非惰性初始化）单例

    <img src="./Spring源码.assets/image-20221115184004433.png" alt="image-20221115184004433" style="zoom:75%;" /> 

    这里面的代码非常复杂，包含步骤4的内容

    <img src="./Spring源码.assets/image-20221115184202308.png" alt="image-20221115184202308" style="zoom:70%;" />  

    **详细**

    1. F7进入方法，主要看方法的关键步骤

       ![image-20221115184540201](./Spring源码.assets/image-20221115184540201.png) 

       主要涉及类型转换的操作，`String CONVERSION_SERVICE_BEAN_NAME = "conversionService";`

       If 这块代码的判断就是将这些类型转换的操作，设置到beanFactory里

       

    2. F8跳过，到下一段代码

       ![image-20221115185154799](./Spring源码.assets/image-20221115185154799.png)

       如果之前没有任何bean后处理器（如PropertyPlaceholderConfigurer bean）注册，则注册一个默认的嵌入式值解析器：此时，主要用于解析注释属性值。PropertyPlaceholderConfigurer 容器对象 处理${}的。`spring-${username}.xml`

       根据环境对象，处理当前属性的值，然后存进beanFactory

       

    3. F8到下一个代码

       ![image-20221115185655090](./Spring源码.assets/image-20221115185655090.png)

       及早初始化LoadTimeWeaverAware bean，以便及早注册其变压器。

       织入Aware的一些操作

       

    4. F8下一段

       ```java
       // Stop using the temporary ClassLoader for type matching. 停止使用临时ClassLoader进行类型匹配。
       beanFactory.setTempClassLoader(null);
       ```

    5. F8下一段

       ```java
       // Allow for caching all bean definition metadata, not expecting further changes. 一些不动的bean配置放这里
       beanFactory.freezeConfiguration();
       ```


​    

​    

##### beanFactory.preInstantiateSingletons

1. F8下一段到达核心点

   ![image-20221115190157633](./Spring源码.assets/image-20221115190157633.png) 

   初始化所有剩余非懒加载的单例对象

   **详细**

   1. F7进入方法

      ![image-20221115190347683](./Spring源码.assets/image-20221115190347683.png) 

      日志略过

   2. F8到下一段

      ![image-20221115191520108](./Spring源码.assets/image-20221115191520108.png)

      取出来的就是上面步骤`loadBeanDefinitions(beanFactory)`的

   3. F8下一段代码，for循环    

```java
      // Trigger initialization of all non-lazy singleton beans...
      for (String beanName : beanNames) {  // size = 1 , beanName = User
          // RootBeanDefinition 出现，在loadBeanDefinition()有提到这个类
      	RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName); // 图一，获取bean的描述信息 
          
          // 经过上方描述信息的整合后，这里判断这个bean是否为这些 
      	if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) { 
      		if (isFactoryBean(beanName)) {  // 看序号19课程（bean对象实现beanFactory后 加上&的获取）
      			Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
      			if (bean instanceof FactoryBean) {
      				final FactoryBean<?> factory = (FactoryBean<?>) bean;
      				boolean isEagerInit;
      				if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
      					isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
      									((SmartFactoryBean<?>) factory)::isEagerInit,
      							getAccessControlContext());
      				}
      				else {
      					isEagerInit = (factory instanceof SmartFactoryBean &&
      							((SmartFactoryBean<?>) factory).isEagerInit());
      				}
      				if (isEagerInit) {
      					getBean(beanName);
      				}
      			}
      		}
      		else {
      			getBean(beanName); // 本次简单距离并没有beanFactory对象，所以进入此方法
      		}
      	}
      }
```

<img src="./Spring源码.assets/image-20221115211029524.png" alt="image-20221115211029524" style="zoom:50%;" /> 

  （图一） 这个方法的作用是把父类和子类的Bean的描述信息做一个整合，例如：Abstract、Singleton 、LazyInit等



   4. 进入`getBean(beanName)`

      ![image-20221115214149790](./Spring源码.assets/image-20221115214149790.png)

      do开头的方法代表正式开始干活

   4. 进入doGetBean()

      ![image-20221115220650783](./Spring源码.assets/image-20221115220650783.png) 

      第一个方法先不讲，下一个

      ![image-20221115221552064](./Spring源码.assets/image-20221115221552064.png) 

      创建好bean对象后，放入Map容器里，Map容器在放入缓存中，所以这步是去缓存中获取bean

      进入getSingleton方法

      <img src="./Spring源码.assets/image-20221115221958790.png" alt="image-20221115221958790" style="zoom:50%;"/>

      ```java
      singletonObjects         -- 一级缓存
      earlySingletonObjects    -- 二级缓存
      singletonFactories       -- 三级缓存
      ```

      由于程序刚启动，所以没有，返回null，所以`sharedInstance`为空，if 判断为 false，进入else分支

   5. 进入else分支

      <img src="./Spring源码.assets/image-20221115222845910.png" alt="image-20221115222845910" style="zoom:50%;" /> 

      if 判断作用：

      - 如果对象都是单例对象的话，则之后会尝试解决循环依赖的问题
      - 如果是原型模式且存在循环依赖，那么抛出异常（这是if判断的主要意思）

   6. 往下走，获取父类容器

      ```java
      BeanFactory parentBeanFactory = getParentBeanFactory();
      if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
          ...
      }
      ```

      由于没有父类容器，过到下一个if

   7. 往下到，判断，标志位

      <img src="./Spring源码.assets/image-20221115223521123.png" alt="image-20221115223521123" style="zoom:50%;" /> 

       进入

      <img src="./Spring源码.assets/image-20221115223957664.png" alt="image-20221115223957664" style="zoom:50%;" /> 

      将指定bean标记为已创建，返回，到代码try

   8. 进入检测

      ![image-20221115224614402](./Spring源码.assets/image-20221115224614402.png) 

      获取BeanDefinition 并 检测它    

   10. 下一步，判断依赖

       ![image-20221115225042471](./Spring源码.assets/image-20221115225042471.png) 

       如果存在依赖的bean，则优先实例化依赖的bean。

       **例子**：<bean  A、B、C 三个bean标签，没依赖的话是由上往下执行，但是如果A dependsON B ，那么B先实力，再C，再A

       由于没有依赖，代码跳过
       
       

######     getSingleton(beanName, () -> {

   11. 进入核心分支

       <img src="./Spring源码.assets/image-20221115230059354.png" alt="image-20221115230059354" style="zoom:50%;" /> 

       为单例，进入缓存方法 `getSingleton()`<img src="./Spring源码.assets/image-20221115230339851.png" alt="image-20221115230339851" style="zoom:50%;" /> 

       跳过中间的一些判断只直接到获取对象`getObject()`，回到上级的lambda中

        <img src="./Spring源码.assets/image-20221115230702987.png" alt="image-20221115230702987" style="zoom:50%;" /> 

          进入`createBean()`，并跳过前部分的逻辑判断直接找到`doCreateBean()`方法

          <img src="./Spring源码.assets/image-20221115230950780.png" alt="image-20221115230950780" style="zoom:50%;" /> 

          进入`doCreateBean()`后才是**步骤4**的开始

          跳过前面清除缓存步骤，通过第二个 if 判断

       ![image-20221115233822512](./Spring源码.assets/image-20221115233822512.png)

       进入`createBeanInstance()`核心实例化过程（目前只看核心代码）

       直接到方法的最后一步

       ![image-20221115234440841](./Spring源码.assets/image-20221115234440841.png) 

       看到构造器就能知道可以通过newInstance获取对象

       没有特殊处理，用简单的无参构造：`instantiateBean()`

       进入instantiateBean(beanName, mad)

       <img src="./Spring源码.assets/image-20221205105100303.png" alt="image-20221205105100303" style="zoom:50%;" /> 

       再进入 instantiate(mbd, beanName, parent)

       忽略上面的逻辑判断

       <img src="./Spring源码.assets/image-20221116000007840.png" alt="image-20221116000007840" style="zoom:50%;" />  

          代码经过以上方式到达初始化`BeanUtils.instantiateClass(constructorToUse)` ，进入

          <img src="./Spring源码.assets/image-20221116000516948.png" alt="image-20221116000516948" style="zoom:50%;" /> 

          完成后，开始逐一返回。注意：目前只是在堆中开辟了一处空间，并没有填充属性 初始化![image-20221116001842834](./Spring源码.assets/image-20221116001842834.png)

          再返回

          <img src="./Spring源码.assets/image-20221116002130311.png" alt="image-20221116002130311" style="zoom:50%;" /> 

          <img src="./Spring源码.assets/image-20221116002354557.png" alt="image-20221116002354557" style="zoom:50%;" /> 

          <img src="./Spring源码.assets/image-20221116002434126.png" alt="image-20221116002434126" style="zoom:50%;" /> 

          <img src="./Spring源码.assets/image-20221116002608588.png" alt="image-20221116002608588" style="zoom:50%;" /> 

          12. 然后一直往下走，可看到这段代码
       
              ![image-20221116002821824](./Spring源码.assets/image-20221116002821824.png)
       
              这部分代码是解决循环依赖的问题。解决方法为：提前暴露，暴露 实例化但并未初始化的bean对象
       
              进入方法看看
              
                 <img src="./Spring源码.assets/image-20221116190929994.png" alt="image-20221116190929994" style="zoom:70%;" /> 
              
                 Bean放入三级缓存， 第二参数放入lambda
              
       
          为什么要使用三级缓存？关键点是这个
       
          ![image-20221116191313107](./Spring源码.assets/image-20221116191313107.png) 
       
        里面有BPP，AOP就是靠着BPP实现的
       
          下一步

   ![image-20221116191636193](./Spring源码.assets/image-20221116191636193.png) 

   暴露bean赋值，下一步

​             ![image-20221116191742152](./Spring源码.assets/image-20221116191742152.png)  

填充属性，经过这个方法后，bean内的属性的注入了

   <img src="./Spring源码.assets/image-20221116191903541.png" alt="image-20221116191903541" style="zoom:50%;" /> 

   **进入下一个方法**（开始进入步骤4中的Aware）

   ![image-20221116192111419](./Spring源码.assets/image-20221116192111419.png) 

   ![image-20221116192237713](./Spring源码.assets/image-20221116192237713.png) 



   这个方法是，如果bean实现了某些Aware接口的话，那么就在这个方法里面进行Aware相关的注入。本例子没有实现Aware，所以下一步。

   开始进入Before操作

   ![image-20221116192723856](./Spring源码.assets/image-20221116192723856.png) 

   往下几行就能看到After方法

   ![image-20221116220905191](./Spring源码.assets/image-20221116220905191.png)

   这里执行完后就成了完整的对象



   返回上一层

   ![image-20221116221416850](./Spring源码.assets/image-20221116221416850.png) 

   一直往下到

   ![image-20221116221618277](./Spring源码.assets/image-20221116221618277.png) 

   - 这个的作用是，当容器关闭的时候要销毁对象步骤，看内部操作
   
     ![image-20221116223116064](./Spring源码.assets/image-20221116223116064.png) 

   下一步返回对象 return

   <img src="./Spring源码.assets/image-20221116223406514.png" alt="image-20221116223406514" style="zoom:50%;" /> 

   下一步再 return

   <img src="./Spring源码.assets/image-20221116223928048.png" alt="image-20221116223928048" style="zoom:50%;" /> 

   再return

   <img src="./Spring源码.assets/image-20221116224830628.png" alt="image-20221116224830628" style="zoom:50%;" />  

   到finally

   <img src="./Spring源码.assets/image-20221116224952462.png" alt="image-20221116224952462" style="zoom:50%;" /> 

   有after 上方必定有before

   before：记录当前对象的加载状态

   after：移除缓存中对该bean的正在加载状态的记录

   下一步，if 判断

   <img src="./Spring源码.assets/image-20221116231117568.png" alt="image-20221116231117568" style="zoom:50%;" /> 

   讲循环依赖的时候再讲这个部分 

   先一步，return

   <img src="./Spring源码.assets/image-20221116231552451.png" alt="image-20221116231552451" style="zoom:50%;" /> 

   从createBean回来后，sharedInstance是个完整对象，所以不是直接拿去用吗？为什么要多走个这个方法？

   - 答：得到的sharedInstance 有可能是FactoryBean，所以这个方法里面一定会有个getObject的方法操作（经过两个同名多态方法后进入do开头的同名，就能看到）
   
     另外：一个实例实现FactoryBean对象要注意，启动后会创建两个对象，1. FactoryBean对像 2.getObject的时候才会实现我们想要的对象
   
     <img src="./Spring源码.assets/image-20221116233700581.png" alt="image-20221116233700581" style="zoom:50%;" />  

   继续往下走，退出了两层try，跳过一个if，return到上一级
          <img src="./Spring源码.assets/image-20221116234218265.png" alt="image-20221116234218265" style="zoom:50%;" /> 

   再return

   <img src="./Spring源码.assets/image-20221116234307556.png" alt="image-20221116234307556" style="zoom:50%;" /> 

由于就一个bean，当前for循环结束，进入下一个for循环

   <img src="./Spring源码.assets/image-20221116234603731.png" alt="image-20221116234603731" style="zoom:50%;" /> 

   由于没有符合条件，跳过，返回上一级

   <img src="./Spring源码.assets/image-20221116234651175.png" alt="image-20221116234651175" style="zoom:50%;" /> 

   至此，步骤4 结束，再向上返回



#### finishRefresh

<img src="./Spring源码.assets/image-20221116235051312.png" alt="image-20221116235051312" style="zoom:50%;" /> 

最后一步，完成刷新，发布相应事件，进入

<img src="./Spring源码.assets/image-20221116235432766.png" alt="image-20221116235432766" style="zoom:50%;" />  

之后一直返回到

<img src="./Spring源码.assets/image-20221116235712172.png" alt="image-20221116235712172" style="zoom:50%;" /> 

整个创建过程结束





## 三、Spring启动流程细节

### super(parent) - 细讲

进入，一直到这里

<img src="./Spring源码.assets/image-20221120002742629.png" alt="image-20221120002742629" style="zoom:50%;" /> 

创建资源模式处理器

F8 下一步

<img src="./Spring源码.assets/image-20221120160540199.png" alt="image-20221120160540199" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221120160745856.png" alt="image-20221120160745856" style="zoom:50%;" /> 

继续挨个的创建

也就不继续截图的，看备注即可看懂这一些个参数。

创建完成后回到方法
 <img src="./Spring源码.assets/image-20221120161404602.png" alt="image-20221120161404602" style="zoom:50%;" />



#### getResourcePatternResolver

下一步F7进入方法，作用：创建一个资源模式解析器 

<img src="./Spring源码.assets/image-20221120162835567.png" alt="image-20221120162835567" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221121233708142.png" alt="image-20221121233708142" style="zoom:50%;" /> 



创建完成，返回至这里

<img src="./Spring源码.assets/image-20221121233916063.png" alt="image-20221121233916063" style="zoom:50%;" /> 

进入看一眼结构

<img src="./Spring源码.assets/image-20221121234227744.png" alt="image-20221121234227744" style="zoom:50%;" /> 

因为parent为空，所以不执行

继续返回（后续都是对进入super方法后执行构造）

途径，停留简单讲解，但目前用不到，就不做笔记了

<img src="./Spring源码.assets/image-20221121234843675.png" alt="image-20221121234843675" style="zoom:50%;" /> 

 最后返回到入口点





### setConfigLocations(configLocations) - 细讲

<img src="./Spring源码.assets/image-20221121235144410.png" alt="image-20221121235144410" style="zoom:50%;" /> 

进入

<img src="./Spring源码.assets/image-20221126001416081.png" alt="image-20221126001416081" style="zoom:50%;" /> 

**记住这个类 AbstractRefreshableConfigApplicationContext**

前两句断言判断后取到configLocations，进入for

**注：configLocations参数后面还会用到**

<img src="./Spring源码.assets/image-20221126001456973.png" alt="image-20221126001456973" style="zoom:50%;" /> 

解析给定的路径。-  进入resolvePath

<img src="./Spring源码.assets/image-20221126001844465.png" alt="image-20221126001844465" style="zoom:50%;" /> 

先进入getEnvironment

<img src="./Spring源码.assets/image-20221126002041901.png" alt="image-20221126002041901" style="zoom:50%;" />  

进入StandardEnvironment，会发现没看见他的构造方法

<img src="./Spring源码.assets/image-20221126002312716.png" alt="image-20221126002312716" style="zoom:50%;" /> 

进入父类AbstractEnvironment中的构造方法那里打断点

<img src="./Spring源码.assets/image-20221126002406500.png" alt="image-20221126002406500" style="zoom:50%;" /> 

这样再走F7下一步的话就能进来到父类；到了这里可以蛮看看上方的参数。

在下一步的话，就进入子类StandardEnvironment 里重写的父类customizePropertySources 方法

<img src="./Spring源码.assets/image-20221126002742067.png" alt="image-20221126002742067" style="zoom:50%;" /> 

然后方法内容将父类传过来的类赋值 （MutablePropertySources 加载对应属性资源）

走完方法

<img src="./Spring源码.assets/image-20221126003806158.png" alt="image-20221126003806158" style="zoom:50%;" />  

可看到对象已经获取到了系统的一些属性值

<img src="./Spring源码.assets/image-20221126003925638.png" alt="image-20221126003925638" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221126004149903.png" alt="image-20221126004149903" style="zoom:50%;" /> 

（课程等下要使用到电脑的名字user.name这个参数）



之后一直返回，StandardEnvironment的对象就创建好了

 <img src="./Spring源码.assets/image-20221126005148555.png" alt="image-20221126005148555" style="zoom:50%;" /> 

然回到这里，后面的方法作用就是替换掉$的占位符的，看到熟悉的user.name了吧，能想到要替换成什么了吧～

大概能想到这个方法一定是要先匹配这几个字符$ 、{、}

进入resolveRequiredPlaceholders

<img src="./Spring源码.assets/image-20221126230213860.png" alt="image-20221126230213860" style="zoom:50%;" /> 

propertyResolver里面有上面刚获取的环境数据

进入resolveRequiredPlaceholders

<img src="./Spring源码.assets/image-20221126233218922.png" alt="image-20221126233218922" style="zoom:50%;" /> 

其间的createPlaceholderHelper是创建些占位符的标识

<img src="./Spring源码.assets/image-20221126233338341.png" alt="image-20221126233338341" style="zoom:50%;" /> 

进行识别的参数，返回进入到 doResolvePlaceholders 方法

<img src="./Spring源码.assets/image-20221126234110803.png" alt="image-20221126234110803" style="zoom:50%;" /> 



#### replacePlaceholders

<img src="./Spring源码.assets/image-20221126234939439.png" alt="image-20221126234939439" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221126234808067.png" alt="image-20221126234808067" style="zoom:50%;" /> 

目前并没有多重，所以直接下一行

<img src="./Spring源码.assets/image-20221126235634668.png" alt="image-20221126235634668" style="zoom:50%;" /> 

F7进入

<img src="./Spring源码.assets/image-20221126235735223.png" alt="image-20221126235735223" style="zoom:50%;" /> 



##### getProperty

<img src="./Spring源码.assets/image-20221127000453292.png" alt="image-20221127000453292" style="zoom:50%;" /> 

然后返回

<img src="./Spring源码.assets/image-20221127001719521.png" alt="image-20221127001719521" style="zoom:50%;" /> 

已经取到了值

 <img src="./Spring源码.assets/image-20221127002206762.png" alt="image-20221127002206762" style="zoom:50%;" /> 

 之后占位符经过replace就完全被替换了

一直返回到，结束

 <img src="./Spring源码.assets/image-20221127003639149.png" alt="image-20221127003639149" style="zoom:50%;" /> 



### refresh() - 细讲

#### prepareRefresh()

<img src="./Spring源码.assets/image-20221127010607640.png" alt="image-20221127010607640" style="zoom:50%;" /> 

往下走

##### initPropertySources() 的扩展

 为空方法，留给子类覆盖，做拓展使用，初始化属性资源 

<img src="./Spring源码.assets/image-20221127011904807.png" alt="image-20221127011904807" style="zoom:50%;" /> 

然后看调用是否能输出拓展内容

```java
MyClassPathXmlApplicationContext ac = new MyClassPathXmlApplicationContext("spring-config.xml");
```

启动，发现能执行到

就算拓展成功了，



##### getEnvironment().validateRequiredProperties()

验证环境属性，可配合initPropertySources() 方法里设置的 setRequiredProperties(“user.name”)

里面就会验证，这个属性是否有值

<img src="./Spring源码.assets/image-20221127015705578.png" alt="image-20221127015705578" style="zoom:50%;" /> 

对应的属性值为空的话，就会添加进ex.add…..里面，最后抛出

<img src="./Spring源码.assets/image-20221127020107495.png" alt="image-20221127020107495" style="zoom:50%;" /> 

验证后开始创建监听器

<img src="./Spring源码.assets/image-20221127021141074.png" alt="image-20221127021141074" style="zoom:50%;" /> 

创建好准备工作后



#### obtainFreshBeanFactory()

<img src="./Spring源码.assets/image-20221127021511764.png" alt="image-20221127021511764" style="zoom:50%;" /> 

有可能在此之前就有beanFactory，随意进入这个方法前的第一步就是判断有没有，有 就干掉

 <img src="./Spring源码.assets/image-20221127021605698.png" alt="image-20221127021605698" style="zoom:50%;" /> 

下一步执行createBeanFactory() 创建工厂 

<img src="./Spring源码.assets/image-20221127155259359.png" alt="image-20221127155259359" style="zoom:50%;" /> 

留意这两个值 



下一步 setId 没什么好说的

下一步定制



##### customizeBeanFactory(beanFactory)

<img src="./Spring源码.assets/image-20221127160018825.png" alt="image-20221127160018825" style="zoom:50%;" /> 

进入后可看到刚才需要留意的两个属性判断，但出奇的是并不会进入到if里

原因：this的这两个值是为null的；如果想要有值的话，那就需要重写该方法

<img src="./Spring源码.assets/image-20221127160255862.png" alt="image-20221127160255862" style="zoom:45%;" /> 

这么重写进去后就有值了，就能给beanFactory 里的这两个值重新设值

 

##### loadBeanDefinitions(beanFactory)

加载bean的定义信息

<img src="./Spring源码.assets/image-20221127173504189.png" alt="image-20221127173504189" style="zoom:50%;" /> 

根据this创建实体解析器：用它读取本地的某些xsd或dtd文件，来完成解析工作。 

课程继续讨论了关于这META-INF/spring.schemas里的配置文件是否会被取出来，但对于目前阶段的我来说无关紧要；



## 四、Spring配置文件加载过程

继上继续讲解此方法

loadBeanDefinitions(beanFactory) 加载bean的定义信息，就是配置文件里的那些<bean>…</bean>



#### initBeanDefinitionReader(beanDefinitionReader)

初始化beanDefinitionReader，此处设置配置文件是否要进行验证

用到了适配器模式



#### loadBeanDefinitions(beanDefinitionReader)

接下来会进入多个同名但不同参数的方法

<img src="./Spring源码.assets/image-20221129002346007.png" alt="image-20221129002346007" style="zoom:50%;" /> 

获取配置文件的路径，往下走，由于我们是String格式的，则进入到第二个if判断 

继续进入下一个

**loadBeanDefinitions(configLocations)**

**loadBeanDefinitions(location)**

**loadBeanDefinitions(location, null)**

<img src="./Spring源码.assets/image-20221129004041466.png" alt="image-20221129004041466" style="zoom:50%;" /> 

**loadBeanDefinitions(resources)**

**loadBeanDefinitions(resource)**

**loadBeanDefinitions(new EncodedResource(resource))**

```
String[] -> String -> Resource[] -> resources -> 包装成EncodedResource    参数的变化
```

<img src="./Spring源码.assets/image-20221129004808475.png" alt="image-20221129004808475" style="zoom:50%;" /> 

根据输入流，获取到配置文件inputStream后， 字节码，传给do处理



**doLoadBeanDefinitions(inputSource, encodedResource.getResource())**

<img src="./Spring源码.assets/image-20221129005709894.png" alt="image-20221129005709894" style="zoom:50%;" /> 

- 文档对象内部的创建，getValidationModeForResource(resource)方法进行了xsd、dtd或其他文件的判断操作，方法实际返回数字。

- 可以蛮看看doc对象，里面包含的是各Node信息 

- 在解析自定义标签的时候会用到这些内容

 

**registerBeanDefinitions(doc, resource)** ⬇️

**registerBeanDefinitions(doc, createReaderContext(resource))⬇️**

**doRegisterBeanDefinitions(doc.getDocumentElement())⬇️**

### **parseBeanDefinitions(root, this.delegate)**



##### 1⃣️if 判断为**默认的命名空间**则进入

默认的命名空间：import、alias、bean、beans

**parseDefaultElement(ele, delegate)**

<img src="./Spring源码.assets/image-20221129224647908.png" alt="image-20221129224647908" style="zoom:50%;" /> 

bean

<img src="./Spring源码.assets/image-20221129224939760.png" alt="image-20221129224939760" style="zoom:50%;" /> 

进入 delegate.parseBeanDefinitionElement(ele);

<img src="./Spring源码.assets/image-20221129235417926.png" alt="image-20221129235417926" style="zoom:50%;" /> 

 接着在进入多态同名方法进行详细的解析 parseBeanDefinitionElement(ele, beanName, containingBean);

<img src="./Spring源码.assets/image-20221129235947055.png" alt="image-20221129235947055" style="zoom:50%;" /> 

解析完这两个其实就可以进行反射实例化创建对象了，有id 有class

<img src="./Spring源码.assets/image-20221130000622539.png" alt="image-20221130000622539" style="zoom:50%;" /> 

方法进入后主要创建了以下，然后存入了些值

```
GenericBeanDefinition bd = new GenericBeanDefinition()
```

之后就是一些设置工作了

<img src="./Spring源码.assets/image-20221130000735567.png" alt="image-20221130000735567" style="zoom:50%;" /> 

图中的解析都是在<bean></bean>标签中能用到的属性

其中 parseConstructorArgElements(ele, bd) 构造函数内部可以看看，略微复杂

开始返回到

<img src="./Spring源码.assets/image-20221130002407595.png" alt="image-20221130002407595" style="zoom:50%;" /> 

这里的beanDefinition就已经是完整的解析完xml文件属性值的 

回到上级

<img src="./Spring源码.assets/image-20221203134721756.png" alt="image-20221203134721756" style="zoom:50%;" /> 

delegate.decorateBeanDefinitionIfRequired(ele, bdHolder)

这一步装饰也只是为了对一些属性值的替换工作而已

下一步

<img src="./Spring源码.assets/image-20221203135137079.png" alt="image-20221203135137079" style="zoom:50%;" />  

进入

<img src="./Spring源码.assets/image-20221203135308885.png" alt="image-20221203135308885" style="zoom:50%;" /> 

进入

<img src="./Spring源码.assets/image-20221203135449302.png" alt="image-20221203135449302" style="zoom:50%;" /> 

方法内有很多的逻辑判断，判断完成后可看到

<img src="./Spring源码.assets/image-20221203135727216.png" alt="image-20221203135727216" style="zoom:50%;" /> 

如果上面的if 进不去，就由下面的else做处理

<img src="./Spring源码.assets/image-20221203135943492.png" alt="image-20221203135943492" style="zoom:50%;" /> 

Map是在这里进行存值的

这样就把bean对象注册进来了

返回上级

<img src="./Spring源码.assets/image-20221203140409203.png" alt="image-20221203140409203" style="zoom:50%;" /> 

跟前面的步骤同理，经过判断然后将别名放进别名map（aliasMap）里

返回上级

<img src="./Spring源码.assets/image-20221203140647251.png" alt="image-20221203140647251" style="zoom:50%;" /> 

小结：

- 从**xml文件**里解析到对应的**标签、元素**， 然后将这些**标签、元素**放进**BeanDefinition**里。再把**BeanDefinition**放进**BeanFactory**里，在这步骤中就包含了bean的定义信息。
- 完成了流程图中，xml到beanDefinition的一步；下一步就轮到该进行实例化了 





##### 2⃣️继上 if 的 Else自定义标签

**delegate.parseCustomElement(ele)**

<img src="./Spring源码.assets/image-20221203134231598.png" alt="image-20221203134231598" style="zoom:50%;" /> 

进入 <img src="./Spring源码.assets/image-20221129234953241.png" alt="image-20221129234953241" style="zoom:50%;" /> 





## 五、Spring自定义标签解析过程

继以上分支else内容，context、tx等这些都属于自定义标签

解析xsd、dtd文件



那么代码中是如何获取到标签的？

<img src="./Spring源码.assets/image-20221203143804515.png" alt="image-20221203143804515" style="zoom:50%;" /> 

在META-INF中即可找到对应的配置文件



继续源码解读

parseCustomElement()方法中

先获取对应命名空间

再获取handler

![image-20221203151641201](./Spring源码.assets/image-20221203151641201.png)



###### 其中readerContext是从哪里获取的？

且，还有那些内容？

答：在之前调用的 registerBeanDefinitions(doc, **createReaderContext(resource)**)中返回的

进入方法createReaderContext(resource)

<img src="./Spring源码.assets/image-20221203152218855.png" alt="image-20221203152218855" style="zoom:50%;" /> 

在进入createDefaultNamespaceHandlerResolver()

<img src="./Spring源码.assets/image-20221203152334778.png" alt="image-20221203152334778" style="zoom:50%;" /> 

查看DefaultNamespaceHandlerResolver(cl)

<img src="./Spring源码.assets/image-20221203152425821.png" alt="image-20221203152425821" style="zoom:50%;" /> 

由此可以得到的信息是，会从这个路径下获取配置文件；所以，如果想自定义一个标签的话，就在这个路径下编写自己的配置文件

点击常量，再往下翻能看到 getHandlerMappings() ，这个于之前讲的

<img src="./Spring源码.assets/image-20221203153436699.png" alt="image-20221203153436699" style="zoom:50%;" /> 

一样，下面也有个 getSchemaMappings()， 都会默认调用最后的toString方法

那么进行返回的时候也能看到参数内都已将配置文件读取到了

<img src="./Spring源码.assets/image-20221203154031307.png" alt="image-20221203154031307" style="zoom:50%;" /> 



回到自定标签分支处

readerContext也就是由上半部分创建好的xmlReaderContext

<img src="./Spring源码.assets/image-20221203151641201.png" alt="image-20221203151641201" style="zoom:50%;" /> 

.getNamespaceHandlerResolver()获取到对应的处理器

###### resolve(namespaceUri)

进入.resolve(namespaceUri)

<img src="./Spring源码.assets/image-20221203154843699.png" alt="image-20221203154843699" style="zoom:50%;" /> 

根据key再Mappings里找到handler文件对象

往下走

<img src="./Spring源码.assets/image-20221203160152131.png" alt="image-20221203160152131" style="zoom:50%;" /> 

假设我们现在处理的是context的自定义标签，那么根据上面所获遇到的handler，进入

<img src="./Spring源码.assets/image-20221203155938124.png" alt="image-20221203155938124" style="zoom:50%;" /> 

把这些解析类全部放在`Map<String, BeanDefinitionParser> parsers`里面

返回

把结果记录在缓存中

返回，F8下一步

<img src="./Spring源码.assets/image-20221203160346569.png" alt="image-20221203160346569" style="zoom:50%;" /> 

F7进入.parse

<img src="./Spring源码.assets/image-20221203161025106.png" alt="image-20221203161025106" style="zoom:50%;" /> 

进入findParserForElement()

<img src="./Spring源码.assets/image-20221203161253176.png" alt="image-20221203161253176" style="zoom:50%;" /> 

获取到类，返回

<img src="./Spring源码.assets/image-20221203161424899.png" alt="image-20221203161424899" style="zoom:50%;" /> 

拿着获取到的类进行解析，F7进入

<img src="./Spring源码.assets/image-20221203162227423.png" alt="image-20221203162227423" style="zoom:50%;" /> 

再F7进入，前边部分都是一些判断和赋值，到下面，看到doParse，正式干活方法

<img src="./Spring源码.assets/image-20221203162326123.png" alt="image-20221203162326123" style="zoom:50%;" /> 

方法里面都是做一堆属性的的判断后就返回了

然后将beanDefinition返回上级

<img src="./Spring源码.assets/image-20221203162227423.png" alt="image-20221203162227423" style="zoom:50%;" /> 

然后一直往下走到

<img src="./Spring源码.assets/image-20221203164036499.png" alt="image-20221203164036499" style="zoom:50%;" /> 

进入

**registerBeanDefinition(holder, parserContext.getRegistry());**

**registerBeanDefinition(definition, registry);**

**registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());**

<img src="./Spring源码.assets/image-20221203164346528.png" alt="image-20221203164346528" style="zoom:50%;" /> 

在这里做了存储操作（Map 和 Names），注册到beanFactory里，然后开始返回

途径

<img src="./Spring源码.assets/image-20221203164555985.png" alt="image-20221203164555985" style="zoom:50%;" /> 

最终返回到最开始的else

<img src="./Spring源码.assets/image-20221203164807868.png" alt="image-20221203164807868" style="zoom:50%;" /> 





##### 开始自定义一个标签

<img src="./Spring源码.assets/image-20221203170432516.png" alt="image-20221203170432516" style="zoom:50%;" /> 

创建个  msb:user 这么个标签 需要哪些类

<img src="./Spring源码.assets/image-20221203170823057.png" alt="image-20221203170823057" style="zoom:50%;" /> 



<img src="./Spring源码.assets/image-20221203171259773.png" alt="image-20221203171259773" style="zoom:50%;" /> 



<img src="./Spring源码.assets/image-20221203171504663.png" alt="image-20221203171504663" style="zoom:50%;" /> 



 <img src="./Spring源码.assets/image-20221203173512019.png" alt="image-20221203173512019" style="zoom:50%;" />  

handlers

<img src="./Spring源码.assets/image-20221203172810034.png" alt="image-20221203172810034" style="zoom:50%;" /> 

Schemas

<img src="./Spring源码.assets/image-20221203172834739.png" alt="image-20221203172834739" style="zoom:50%;" /> 

user.xsd

<img src="./Spring源码.assets/image-20221203172724828.png" alt="image-20221203172724828" style="zoom:50%;" />   

然后再配置文件上增加

<img src="./Spring源码.assets/image-20221203172627787.png" alt="image-20221203172627787" style="zoom:50%;" /> 

最后运行看是否能执行

<img src="./Spring源码.assets/image-20221203172946931.png" alt="image-20221203172946931" style="zoom:50%;" /> 







## 六、Spring的bean工厂准备工作

refresh() 的13方法的第三步

### prepareBeanFactory(beanFactory)

<img src="./Spring源码.assets/image-20221205100258713.png" alt="image-20221205100258713" style="zoom:50%;" /> 

F7进入看看new的具体是什么

- <img src="./Spring源码.assets/image-20221205100830937.png" alt="image-20221205100830937" style="zoom:50%;" /> 

  #{} 就知道这是什么了，但目前阶段只是准备工作

返回下一行

#### addPropertyEditorRegistrar(PropertyEditorRegistrar registrar)

<img src="./Spring源码.assets/image-20221205101418668.png" alt="image-20221205101418668" style="zoom:50%;" /> 

这个方法有什么作用？

例子：业务中出现的情况，Adress是由省市区按道理是分成3个变量来处理的，但是如果输入只用一个Adress来处理：`省_市_区`，就可以在这个方法上进行拓展个属于这数据的方法，**属性编辑注册器**

进入**new ResourceEditorRegistrar(this, getEnvironment())**

<img src="./Spring源码.assets/image-20221205102125349.png" alt="image-20221205102125349" style="zoom:50%;" /> 

如果单单看构造方法仅仅是赋值，没什么意思；向上看他的视线类

<img src="./Spring源码.assets/image-20221205102249778.png" alt="image-20221205102249778" style="zoom:50%;" /> 

注册自定义的编辑器，这下清楚了些什么吧

而Spring默认的注册器，是准备在数据填充（填充属性）的时候才调用到

- 具体处理位置

  createBean -> doCreateBean -> createBeanInstance -> instantiateBean ->

  <img src="./Spring源码.assets/image-20221205104557219.png" alt="image-20221205104557219" style="zoom:50%;" /> 

  - 进入BeanWrapperImpl的构造函数

    <img src="./Spring源码.assets/image-20221205104718606.png" alt="image-20221205104718606" style="zoom:50%;" /> 

    看的眼熟吧

    <img src="./Spring源码.assets/image-20221205104753908.png" alt="image-20221205104753908" style="zoom:50%;" /> 

    将默认编辑器的标识为激活

  返回进入 **initBeanWrapper(bw)**

  <img src="./Spring源码.assets/image-20221205105535736.png" alt="image-20221205105535736" style="zoom:50%;" /> 

  - 其中ConversionService 是在最开始进入实例对象的时候设置的

    <img src="./Spring源码.assets/image-20221205105808847.png" alt="image-20221205105808847" style="zoom:50%;" /> 

  - 在这时候调用registerCustomEditors(bw)

    里面先获取有多少个这样的编辑器，然后遍历调用方法，就看进入到默认的编辑器

    <img src="./Spring源码.assets/image-20221205110501774.png" alt="image-20221205110501774" style="zoom:50%;" /> 



##### 拓展自定义属性编辑器

 <img src="./Spring源码.assets/image-20221205111609546.png" alt="image-20221205111609546" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221205112418597.png" alt="image-20221205112418597" style="zoom:50%;" /> 

Address

<img src="./Spring源码.assets/image-20221205111803806.png" alt="image-20221205111803806" style="zoom:50%;" /> 

Customer

<img src="./Spring源码.assets/image-20221205111834006.png" alt="image-20221205111834006" style="zoom:50%;" /> 

AddressPropertyEditor

<img src="./Spring源码.assets/image-20221205112058434.png" alt="image-20221205112058434" style="zoom:50%;" /> 

AddressPropertyEditorRegistrar

<img src="./Spring源码.assets/image-20221205112329751.png" alt="image-20221205112329751" style="zoom:50%;" /> 



编写自定义配置类

<img src="./Spring源码.assets/image-20221205112704434.png" alt="image-20221205112704434" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221205112731364.png" alt="image-20221205112731364" style="zoom:50%;" /> 

或另一种配置方式

<img src="./Spring源码.assets/image-20221205144838790.png" alt="image-20221205144838790" style="zoom:50%;" /> 

测试

<img src="./Spring源码.assets/image-20221205112922631.png" alt="image-20221205112922631" style="zoom:50%;" /> 

拓展结束



#### addBeanPostProcessor(new ApplicationContextAwareProcessor(this))

添加BPP，ApplicationContextAwareProcessor - 此类用来完成某些Aware对象的注入

根据后部分的ApplicationContextAwareProcessor(this) 里的 Aware 做联想，invokeAwareMethods方法，但这个方法只会识别3种类型：BeanNameAware、BeanClassLoaderAware、BeanFactoryAware

当然，是想Aware接口的肯定不止这些，那么除了这三个的其他类会在哪里调用呢？ 

答：那么就是在 ApplicationContextAwareProcessor(this) 类里的一个方法

<img src="./Spring源码.assets/image-20221205152547015.png" alt="image-20221205152547015" style="zoom:50%;" /> 

这里在匹配看你是否实现了这其中六种接口，有的话，就调用最后的 invokeAwareInterfaces(bean)

<img src="./Spring源码.assets/image-20221205152918865.png" alt="image-20221205152918865" style="zoom:50%;" /> 



##### 拓展属于自己的Aware

根据以上判断最终是在ApplicationContextAwareProcessor(this) 里的 postProcessBeforeInitialization() 方法

<img src="./Spring源码.assets/image-20221205153736807.png" alt="image-20221205153736807" style="zoom:50%;" /> 

MyAwareProcessor

<img src="./Spring源码.assets/image-20221205154155787.png" alt="image-20221205154155787" style="zoom:50%;" /> 

最终会在这里被调用

<img src="./Spring源码.assets/image-20221205154715493.png" alt="image-20221205154715493" style="zoom:50%;" /> 





#### ignoreDependencyInterface()

<img src="./Spring源码.assets/image-20221205155930260.png" alt="image-20221205155930260" style="zoom:50%;" /> 

这里面需要忽略的接口是刚才上方在BPP里的那几个Aware接口 



#### registerResolvableDependency()

<img src="./Spring源码.assets/image-20221205162613837.png" alt="image-20221205162613837" style="zoom:50%;" /> 

后面不讲了



<img src="./Spring源码.assets/image-20221205165708522.png" alt="image-20221205165708522" style="zoom:50%;" /> 





## 七、Spring的BeanFactoryPostProcessors执行

### invokeBeanFactoryPostProcessors(beanFactory)

调用各种beanFactory处理器；BFPP可以理解为增强器，也可以理解为，后置处理器

BFPP在进行修改的时候，修改的是BF内的所有对象；而BDRPP修改的是BD的

还得注意一个BeanFactoryPostProcessor的子接口BeanDefinitionRegistryPostProcessor

beanFactory 中包含beanDefinition

<img src="./Spring源码.assets/image-20221205142940499.png" alt="image-20221205142940499" style="zoom:50%;" /> 

#### 如何自定义一个BFPP

- 自定义

  <img src="./Spring源码.assets/image-20221205172347332.png" alt="image-20221205172347332" style="zoom:50%;" /> 

  注册方法一：

  <img src="./Spring源码.assets/image-20221205172429220.png" alt="image-20221205172429220" style="zoom:50%;" /> 

  注册方法二：

  <img src="./Spring源码.assets/image-20221205172501022.png" alt="image-20221205172501022" style="zoom:50%;" /> 

  采用任意一种即可调用到sout

  当自定义的BFPP注入到Spring后，getBeanFactoryPostProcessors()这个方法里面就能看到我们自定的BFPP（原本默认为空）



进入invokeBeanFactoryPostProcessors()

<img src="./Spring源码.assets/image-20221205143040117.png" alt="image-20221205143040117" style="zoom:50%;" /> 

这里面是主要处理逻辑的地方

运行期间可以有多个BFPP

<img src="./Spring源码.assets/image-20221207074144234.png" alt="image-20221207074144234" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221207074400862.png" alt="image-20221207074400862" style="zoom:50%;" /> 

 看懂上方的图，这个方法就看的比较轻松了

<img src="./Spring源码.assets/image-20221209072453143.png" alt="image-20221209072453143" style="zoom:50%;" /> 

父子类分别要实现的方法



开始步骤

上几行理解简单

<img src="./Spring源码.assets/image-20221207074930807.png" alt="image-20221207074930807" style="zoom:50%;" /> 

为什么要创建两个集合？因为这个调用这个方法需要处理两个接口，看注释

<img src="./Spring源码.assets/image-20221207220646562.png" alt="image-20221207220646562" style="zoom:50%;" /> 

下一步for循环将传进来的BFPP数据进行分类，registryProcessors方便执行后续实现了BDRPP接口里的PPBF

跳过for

<img src="./Spring源码.assets/image-20221207221127853.png" alt="image-20221207221127853" style="zoom:50%;" /> 

这里再次创建个了同类不同名的集合，保留本次要执行的

下一步postProcessorNames只获取名字

for循环看注释理解，PriorityOrdered.class是上面er图显示的

下一步

<img src="./Spring源码.assets/image-20221207221821127.png" alt="image-20221207221821127" style="zoom:50%;" /> 

如果有多个，那就对他排个优先级

下一步是添加一个集合

上下两张图看作一个整体，是处理实现了PriorityOrdered排序方式的BF

而后面两大段与这段差不多，分别处理的是实现了Orderd的接口和非排序接口的BF

为什么要重复的验证呢？因为在执行invokeBeanDefinitionRegistryPostProcessors后可能会生成新的BDPP

<img src="./Spring源码.assets/image-20221208070707144.png" alt="image-20221208070707144" style="zoom:50%;" /> 



其中进入sortPostProcessors(currentRegistryProcessors, beanFactory)

<img src="./Spring源码.assets/image-20221208071446302.png" alt="image-20221208071446302" style="zoom:50%;" /> 

如果比较器也为空，使用默认的比较器，进入INSTANCE

<img src="./Spring源码.assets/image-20221208071701997.png" alt="image-20221208071701997" style="zoom:50%;" /> 

在进入类里

<img src="./Spring源码.assets/image-20221208071737882.png" alt="image-20221208071737882" style="zoom:50%;" /> 

doCompare

<img src="./Spring源码.assets/image-20221208071853617.png" alt="image-20221208071853617" style="zoom:50%;" /> 





上方都排序好后，准备开始干活

<img src="./Spring源码.assets/image-20221208072205127.png" alt="image-20221208072205127" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221208074953013.png" alt="image-20221208074953013" style="zoom:50%;" /> 

最后再把所有收到的 统一执行postProcessBeanFactory



**if else 执行完后，继续往下**

<img src="./Spring源码.assets/image-20221211203146344.png" alt="image-20221211203146344" style="zoom:50%;" />  

<img src="./Spring源码.assets/image-20221208073625602.png" alt="image-20221208073625602" style="zoom:50%;" /> 

获取到3个集合对象

进入到for

<img src="./Spring源码.assets/image-20221208073726705.png" alt="image-20221208073726705" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221208073937936.png" alt="image-20221208073937936" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221208074019461.png" alt="image-20221208074019461" style="zoom:50%;" /> 

两小节没认真听



## 八、Spring的BeanFactoryPostProcessors执行2

继上再次分析流程

<img src="./Spring源码.assets/image-20221208223508738.png" alt="image-20221208223508738" style="zoom:50%;" /> 

第一节回顾上节的知识点

第二节课解释为什么需要重复获取实现不同排序接口的BDRPP，因为invokeBeanDefinitionRegistryPostProcessors方法中有可能会产生新的BDRPP，所以需要多次获取

例子：

<img src="./Spring源码.assets/image-20221209074602063.png" alt="image-20221209074602063" style="zoom:50%;" /> 

配置文件直接`<bean class:”这个类的路径”></bean>`即可

自定义的BDRPP的方法postProcessBeanDefinitionRegistry里再次注册了个BD，所以会出现这情况

- 也可以使用另一种方式来注册进去

  <img src="./Spring源码.assets/image-20221209074940402.png" alt="image-20221209074940402" style="zoom:50%;" /> 

可以以上方式自定义两个BDRPP提供测试，了解这里面具体情况



### ConfigurationClassPostProcessor

连老师讲这个实际并不是为了让我们了解这个流程，而是想让我们了解个类`ConfigurationClassPostProcessor`后续简称CCPP

![image-20221212221854214](./Spring源码.assets/image-20221212221854214.png) 

这些注解都是在这个类里面解析的，实现了BDRPP

可以看看执行过程

怎么开启让他在invokeBFPP中获取到这个类我忘了（补：增加ComponentScan注解即可有internal注入），似乎是在讲关于internal相关的知识点的时候提到过

流程是在invokeBFPP中的 postProcessorNames 字符串数组可以看到被获取的名字

然后再invokeBeanDefinitionRegistryPostProcessors方法中for循环轮到CCPP执行他的postProcessBeanDefinitionRegistry

<img src="./Spring源码.assets/image-20221212222913813.png" alt="image-20221212222913813" style="zoom:50%;" /> 

前面都好理解，最重要的最后一个方法，来进行解析的，F7进入

<img src="./Spring源码.assets/image-20221212223243452.png" alt="image-20221212223243452" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221212223106014.png" alt="image-20221212223106014" style="zoom:50%;" /> 

可算是重点了；有使用红色注解的类进入后会筛选出来，在for循环中会

`ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)`里，进入看看

- 方法

  <img src="./Spring源码.assets/image-20221212224636054.png" alt="image-20221212224636054" style="zoom:50%;" /> 

  第二个if是判断是否是从注解生成的；metadata元数据——注解内的数据，如 @Import(“元数据”)

  <img src="./Spring源码.assets/image-20221213072020264.png" alt="image-20221213072020264" style="zoom:50%;" /> 

  <img src="./Spring源码.assets/image-20221213072344406.png" alt="image-20221213072344406" style="zoom:50%;" /> 

  <img src="./Spring源码.assets/image-20221213072448551.png" alt="image-20221213072448551" style="zoom:50%;" /> 

  if 先判断当前的类是否有@Configuration注解，没有的话进入else if里看有没有其他的候选项

  `isConfigurationCandidate`

  - 进入看看

    <img src="./Spring源码.assets/image-20221213073024137.png" alt="image-20221213073024137" style="zoom:50%;" /> 

    <img src="./Spring源码.assets/image-20221214224120665.png" alt="image-20221214224120665" style="zoom:50%;" /> 
    
    进入`candidateIndicators`
    
    <img src="./Spring源码.assets/image-20221213073054058.png" alt="image-20221213073054058" style="zoom:50%;" /> 
    
    返回上级
  
  <img src="./Spring源码.assets/image-20221214224745097.png" alt="image-20221214224745097" style="zoom:50%;" /> 
  
  其中常数`ONFIGURATION_CLASS_ATTRIBUTE`对应的值事`configurationClass`；后面的常数上下就分别是 full、lite；
  
  - 可蛮进入看看
  
    <img src="./Spring源码.assets/image-20221214225151340.png" alt="image-20221214225151340" style="zoom:50%;" /> 
  
  将的这些方法目的都在确定是否有这些注解，如果没有后return false了，如果哪怕只包含一个，也会return true；
  
  <img src="./Spring源码.assets/image-20221213072146850.png" alt="image-20221213072146850" style="zoom:50%;" /> 
  
  返回上一级

符合条件进入

<img src="./Spring源码.assets/image-20221213073501537.png" alt="image-20221213073501537" style="zoom:50%;" /> 

以上动作都是为了给configCandidates添加符合条件的对象值

继续往下

<img src="./Spring源码.assets/image-20221213073651452.png" alt="image-20221213073651452" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221213073724237.png" alt="image-20221213073724237" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221213073820700.png" alt="image-20221213073820700" style="zoom:50%;" /> 

if里判断的是 是否使用自定了命名生成器，如果没有，这用默认的

<img src="./Spring源码.assets/image-20221213073947596.png" alt="image-20221213073947596" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221213074039887.png" alt="image-20221213074039887" style="zoom:50%;" /> 

这个类开始重要

<img src="./Spring源码.assets/image-20221213074115912.png" alt="image-20221213074115912" style="zoom:50%;" /> 

do第一行中 进入parse解析的步骤与上面工具类ConfigurationClassUtils.checkConfigurationClassCandidate里的差不多

- 进入parse后，在进入一个第一个if的parse

  <img src="./Spring源码.assets/image-20221213074631356.png" alt="image-20221213074631356" style="zoom:80%;" /> 

  进入再进入

  <img src="./Spring源码.assets/image-20221213074743780.png" alt="image-20221213074743780" style="zoom:50%;" /> 

  - 进入跳过解析看看那些bean符合条件进行递归调用

    <img src="./Spring源码.assets/image-20221213075014188.png" alt="image-20221213075014188" style="zoom:50%;" /> 

    最后的两个return都是递归调用

    <img src="./Spring源码.assets/image-20221213075216474.png" alt="image-20221213075216474" style="zoom:50%;" /> 

    <img src="./Spring源码.assets/image-20221213075301906.png" alt="image-20221213075301906" style="zoom:50%;" /> 

    返回

  下一个逻辑判断：处理Imprted的情况，当前类是否被别的类Import

  <img src="./Spring源码.assets/image-20221215073331049.png" alt="image-20221215073331049" style="zoom:50%;" /> 

  进不去，下一个

  <img src="./Spring源码.assets/image-20221213075608156.png" alt="image-20221213075608156" style="zoom:50%;" /> 

  这行看不懂就算了
  
  进入doProcessConfigurationClass
  
  - 实际干活
  
    <img src="./Spring源码.assets/image-20221214072527945.png" alt="image-20221214072527945" style="zoom:50%;" /> 
  
    内部类理解，类似：
  
    <img src="./Spring源码.assets/image-20221215073801818.png" alt="image-20221215073801818" style="zoom:50%;" /> 
  
    <img src="./Spring源码.assets/image-20221215074930992.png" alt="image-20221215074930992" style="zoom:50%;" /> 
  
    <img src="./Spring源码.assets/image-20221215075022695.png" alt="image-20221215075022695" style="zoom:50%;" /> 
  
    看出了第一段if 之外段的注解，都是明确表明是为了解析单个注解，其中最重要的是Import注解
  
    <img src="./Spring源码.assets/image-20221214072930920.png" alt="image-20221214072930920" style="zoom:50%;" /> 
  
    getImports方法是获取Import注解内的值
  
    - 进入看看processImports
  
      <img src="./Spring源码.assets/image-20221214073057982.png" alt="image-20221214073057982" style="zoom:50%;" /> 
  
      <img src="./Spring源码.assets/image-20221214073340989.png" alt="image-20221214073340989" style="zoom:50%;" /> 
  
      这里还有个processImports递归，因为带入的元数据里可能还有impoert
  
      <img src="./Spring源码.assets/image-20221215222549953.png" alt="image-20221215222549953" style="zoom:50%;" /> 
  
      <img src="./Spring源码.assets/image-20221215222605605.png" alt="image-20221215222605605" style="zoom:50%;" /> 
  
      <img src="./Spring源码.assets/image-20221215222802049.png" alt="image-20221215222802049" style="zoom:50%;" /> 
  
      



## 九、Spring的ConfigurationClassPostProcessor的讲解

实现了BDRPP接口的类

回顾上节课的内容

<img src="./Spring源码.assets/image-20221215223033275.png" alt="image-20221215223033275" style="zoom:50%;" /> 

根据这个自定的类再去细讲 parse（**会先解析外面的类，其中如果递归到内部类有包含的注解，那么就会把内部类全部解析完后，再回到外面继续**）；

目的就是为了解析被注解修饰的BeanDefinition （里面的xml、Configuration）

<img src="./Spring源码.assets/image-20221215223229094.png" alt="image-20221215223229094" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221215223602652.png" alt="image-20221215223602652" style="zoom:50%;" /> 

进入

<img src="./Spring源码.assets/image-20221215223804107.png" alt="image-20221215223804107" style="zoom:50%;" /> 

<img src="./Spring源码.assets/image-20221215223927845.png" alt="image-20221215223927845" style="zoom:50%;" /> 

由于当前解析的类还包含个内部类，所以会在进入processConfigurationClass后进行一个递归操作



后面的部分就是invokeBean…里的流程做的消息讲解，对部分跳过的类，通过debug进入讲解。对目前的我来说用处不大，听个乐即可



## 十、注册BeanPostProcesser

补充上节课没讲完的Import注解

<img src="./Spring源码.assets/image-20221221073138328.png" alt="image-20221221073138328" style="zoom:50%;" /> 

- getImports(sourceClass) 识别到对应的类

  <img src="./Spring源码.assets/image-20221221073341089.png" alt="image-20221221073341089" style="zoom:50%;" /> 

- collectImports

  <img src="./Spring源码.assets/image-20221221073506721.png" alt="image-20221221073506721" style="zoom:50%;" /> 

  返回，再返回

进入processImports

里面做了 导入一些额外的配置类，同时完成了具体类的实例化工作

