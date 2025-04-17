# Kafka生产与消费全流程

Kafka是一款消息中间件，消息中间件本质就是收消息与发消息，所以这节课我们会从一条消息开始生产出发，去了解生产端的运行流程，然后简单的了解一下broker的存储流程，最后这条消息是如何被消费者消费掉的。其中最核心的有以下内容。

1. Kafka客户端是如何去设计一个非常优秀的生产级的保证高吞吐的一个缓冲机制
2. 消费端的原理：每个消费组的群主如何选择，消费组的群组协调器如何选择，分区分配的方法，分布式消费的实现机制，拉取消息的原理，offset提交的原理。

# Kafka一条消息发送和消费的流程(非集群)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/35d3bd71bad045dd92d5a117c71e1368.png)

## 一、简单入门

我们这里使用Kafka内置的客户端API开发kafka应用程序。因为我们是Java程序员，所以这里我们使用Maven，使用较新的版本

```xml
<dependency>
   <groupId>org.apache.kafka</groupId>
   <artifactId>kafka-clients</artifactId>
   <version>3.3.1</version>
</dependency>
```

### 一、生产者

先创建一个主题，推荐在消息发送时创建对应的主题。当然就算没有创建主题，Kafka也能自动创建。

- **auto.create.topics.enable**

  是否允许自动创建主题。如果设为true，那么produce（生产者往主题写消息），consume（消费者从主题读消息）或者fetch metadata（任意客户端向主题发送元数据请求时）一个不存在的主题时，就会自动创建。缺省为true。

- **num.partitions**

  每个新建主题的分区个数（分区个数只能增加，不能减少 ）。这个参数默认值是1（最新版本），

  若想要消息具有顺序，那么分区只能有一个。否者顺序就不能保证。

#### 1.1 必选属性

创建生产者对象时，有三个属性必须指定（代码中）。

- **bootstrap.servers**

  该属性指定broker的地址清单，地址的格式为host:port。

  清单里不需要包含所有的broker地址，生产者会从给定的broker里查询其他broker的信息。不过最少提供2个broker的信息(用逗号分隔，比如:127.0.0.1:9092,192.168.0.13:9092)，一旦其中一个宕机，生产者仍能连接到集群上。

- **key.serializer**

  生产者接口允许使用参数化类型，可以把Java对象作为键和值传broker，但是broker希望收到的消息的键和值都是字节数组，所以，必须提供将对象序列化成字节数组的序列化器。

  key.serializer必须设置为实现org.apache.kafka.common.serialization.Serializer的接口类

  Kafka的客户端默认提供了ByteArraySerializer,IntegerSerializer,StringSerializer，也可以实现自定义的序列化器。

- **value.serializer**

  同 key.serializer。

#### 1.2 三种发送方式

我们通过生成者的send方法进行发送。send方法会返回一个包含RecordMetadata的Future对象。RecordMetadata里包含了目标主题，分区信息和消息的偏移量。

##### 1.1.1 发送并忘记

忽略send方法的返回值，不做任何处理。大多数情况下，消息会正常到达，而且生产者会自动重试，但有时会丢失消息。

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * 类说明：kafak生产者
 */
public class HelloKafkaProducer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 补充一下： 配置多台的服务，用’,‘来分割。作用：若其中一个宕机了，生产者依然可以连接上（集群）
        // 设置String的序列化 StringSerializer （将对象 =》 二进制数组：能在网络上传输）
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);

        // 构建kafka生产者对象
        KafkaProducer<String,String> producer = new KafkaProducer<String, String>(properties);
        try {
            ProducerRecord<String,String> record;
            try {
                // 构建消息
                record = new ProducerRecord<String,String>("msb", "teacher","lijin");
                // 发送消息
                producer.send(record);
                System.out.println("message is sent.");
            } catch (Exception e) {
                e.printStackTrace();
            }
        } finally {
            // 释放连接
            producer.close();
        }
    }
}
```

##### 1.1.2 同步发送

获得send方法返回的Future对象，在合适的时候调用Future的get方法，查看消息去了哪个分区。参见代码。

```java
package com.msb.producer;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.Future;

/**
 * 类说明：发送消息--同步模式
 */
public class SynProducer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的序列化
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);

        // 构建kafka生产者对象
        KafkaProducer<String,String> producer  = new KafkaProducer<String, String>(properties);
        try {
            ProducerRecord<String,String> record;
            try {
                // 构建消息
                record = new ProducerRecord<String,String>("msb", "teacher2333","lijin");
                // 发送消息
                Future<RecordMetadata> future =producer.send(record);
                RecordMetadata recordMetadata = future.get();
                if(null!=recordMetadata){
                    System.out.println("offset:"+recordMetadata.offset()+","
                            +"partition:"+recordMetadata.partition());
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        } finally {
            // 释放连接
            producer.close();
        }
    }
}
```

##### 1.1.3 异步发送

实现接口org.apache.kafka.clients.producer.Callback，然后将实现类的实例作为参数传递给send方法。

```java
package com.msb.producer;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.Future;

/**
 * 类说明：发送消息--异步模式
 */
public class AsynProducer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的序列化
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);

        // 构建kafka生产者对象
        KafkaProducer<String,String> producer  = new KafkaProducer<String, String>(properties);
        try {
            ProducerRecord<String,String> record;
            try {
                // 构建消息
                record = new ProducerRecord<String,String>("msb", "teacher","lijin");
                // 发送消息
                producer.send(record, new Callback() {
                    @Override
                    public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                        if (e == null){
                            // 没有异常，输出信息到控制台
                            System.out.println("offset:"+recordMetadata.offset()+"," +"partition:"+recordMetadata.partition());
                        } else {
                            // 出现异常打印
                            e.printStackTrace();
                        }
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        } finally {
            // 释放连接
            producer.close();
        }
    }
}
```

### 二、消费者

消费者的含义，同一般消息中间件中消费者的概念。在高并发的情况下，生产者产生消息的速度是远大于消费者消费的速度，单个消费者很可能会负担不起，此时有必要对消费者进行横向伸缩，于是我们可以使用多个消费者从同一个主题读取消息，对消息进行分流。

#### 2.1 必选属性

创建消费者对象时一般有四个属性必须指定。

bootstrap.servers、value.Deserializer、 key.Deserializer 其含义同生产者

#### 2.2 可选属性

group.id （**ConsumerConfig.GROUP_ID_CONFIG**） 并非完全必需。

作用：指定了消费者属于哪一个群组。

虽然创建一个不属于任何群组的消费者也没有问题，但是绝大部分情况下还是会使用群组来消费。

对应代码：

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

/**
 * 类说明：消费者入门
 */
public class HelloKafkaConsumer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的反序列化
        properties.put("key.deserializer", StringDeserializer.class);
        properties.put("value.deserializer", StringDeserializer.class);
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // 构建kafka消费者对象
        KafkaConsumer<String,String> consumer = new KafkaConsumer<String, String>(properties);
        try {
            consumer.subscribe(Collections.singletonList("msb"));
            // 调用消费者拉取消息
            while(true){
                // 设置1秒的超时时间
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                for(ConsumerRecord<String, String> record : records){
                    String key = record.key();
                    String value = record.value();
                    System.out.println("接收到消息: key = " + key + ", value = " + value);
                }
            }
        } finally {
            // 释放连接
            consumer.close();
        }
    }
}
```



#### 2.3 消费者群组

Kafka里消费者从属于消费者群组，一个群组里的消费者订阅的都是同一个主题，每个消费者接收主题一部分分区的消息。

负载均衡是建立在分区级别

| 主题T有4个分区，群组中只有一个消费者，则该消费者将收到主题T1全部4个分区的消息。 |
| :----------------------------------------------------------- |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/5693e8540e2b4b31b73c8b6ac6ee7038.png) |

| 在群组中增加一个消费者2，那么每个消费者将分别从两个分区接收消息，上图中就表现为消费者1接收分区1和分区3的消息，消费者2接收分区2和分区4的消息。 |
| ------------------------------------------------------------ |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/dc2400aa869a414abe8909c1e59190e5.png) |

| 群组中有4个消费者，那么每个消费者将分别从1个分区接收消息。   |
| ------------------------------------------------------------ |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/1a2816e2b9b94ffa88598d5e7e021841.png) |

| 增加更多的消费者，超过了主题的分区数量，就会有一部分的消费者被闲置，不会接收到任何消息。 |
| ------------------------------------------------------------ |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/6992d6eacdbb4e31b1ab2facaf09dd0f.png) |

| 不同群组，各消费各的                                         |
| ------------------------------------------------------------ |
| <img src="/Users/zhengbufeng/Documents/学习笔记/Learning-Notes/kafka/2、Kafka生产与消费全流程.assets/image-20241015121847304.png" alt="image-20241015121847304" style="zoom:50%;" /> |

往消费者群组里增加消费者是进行横向伸缩能力的主要方式。所以我们有必要为主题设定合适规模的分区，在负载均衡的时候可以加入更多的消费者。但是要记住，**一个群组里消费者数量超过了主题的分区数量，多出来的消费者是没有用处的。**

具体代码在comsumergroup包内，验证以上结果。群组内增加消费者是动态的增加，对应的消费者模块是可以不用重启的（通过动态再平衡的方式分配分区）



**自己的的疑惑**：

1. 怎么消费消息的，群组A消费者1消费完数据xx，群组B里面的消费者1还能消费到同一条数据吗？

2. 刚才看到一部份的讲解是说按偏移量。比如消费者1在分区1消费了数据，那么ack回去分区内的偏移量会+1，那么下次就不会消费到相同的数据。但是再另一个群组的消费中若要是也消费分区1内的消息却又能从头开始消费，这是为什么？

同时解答以上两个问题：

具体来说：举例分区内有一个消息‘111’

- 当群组A的消费者消费了消息‘111’，并且发送ACK（确认消息已成功处理）后，群组A的偏移量会从0增加到1。
- 然而，群组B有自己独立的偏移量，即使群组A已经消费了‘111’，群组B的偏移量仍然是0。群组B在消费同样的‘111’消息并发送ACK后，它的偏移量才会从0变为1。

这正是Kafka的**群组隔离**机制的体现，不同消费群组之间的消费进度互不影响。





## 二、序列化

创建生产者对象必须指定序列化器，默认的序列化器并不能满足我们所有的场景。我们完全可以自定义序列化器。只要实现org.apache.kafka.common.serialization.Serializer接口即可。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/8e473e701c814e6e9b755f6fe44499b1.png)

### 一、自定义序列化

代码见：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/8eb6dfa47624416fb16ed4b02739ee5d.png)

代码中使用到了自定义序列化。

```java
/**
 * 类说明：实体类
 */
@Data
@AllCons...
public class User {
    private int id;
    private String name;
}
```



id的长度4个字节，字符串的长度描述4个字节， 字符串本身的长度nameSize个字节

```java
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.common.serialization.Serializer;
import java.nio.ByteBuffer;
import java.util.Map;
/**
 * 类说明：自定义序列化器
 */
public class UserSerializer implements Serializer<User> {
    public void configure(Map<String, ?> configs, boolean isKey) {
        //do nothing
    }

    public byte[] serialize(String topic, User data) {
        try {
            byte[] name;
            int nameSize;
            if(data==null){
                return null;
            }
            if(data.getName()!=null){
                name = data.getName().getBytes("UTF-8");
                //字符串的长度
                nameSize = data.getName().length();
            }else{
                name = new byte[0];
                nameSize = 0;
            }
            /*id的长度4个字节，字符串的长度描述4个字节，
            字符串本身的长度nameSize个字节*/
            ByteBuffer buffer = ByteBuffer.allocate(4 + 4 + nameSize);
            buffer.putInt(data.getId()); //4
            buffer.putInt(nameSize);	 //4
            buffer.put(name);			 //nameSize
            return buffer.array();
        } catch (Exception e) {
            throw new SerializationException("Error serialize User:"+e);
        }
    }

    public void close() {
        //do nothing
    }
}

```

返序列化

```java
import org.apache.kafka.common.errors.SerializationException;
import org.apache.kafka.common.serialization.Deserializer;
import java.nio.ByteBuffer;
import java.util.Map;
/**
 * 类说明：自定义反序列化器
 */
public class UserDeserializer implements Deserializer<User> {
    public void configure(Map<String, ?> configs, boolean isKey) {
        //do nothing
    }

    public User deserialize(String topic, byte[] data) {
        try {
            if(data == null){
                return null;
            }
            if(data.length < 8){
                throw new SerializationException("Error data size.");
            }
            ByteBuffer buffer = ByteBuffer.wrap(data);
            int id;
            String name;
            int nameSize;
            id = buffer.getInt();
            nameSize = buffer.getInt();
            byte[] nameByte = new byte[nameSize];
            buffer.get(nameByte);
            name = new String(nameByte,"UTF-8");
            return new User(id, name);
        } catch (Exception e) {
            throw new SerializationException("Error Deserializer DemoUser."+e);
        }
    }

    public void close() {
        //do nothing
    }
}

```

这样一来，对应的生产者消费者类型可以这么设置，类型可以修改为`UserSerializer`：

```java
...
// 设置String的序列化
properties.put("key.serializer", StringSerializer.class);
// 设置value的自定义序列化
properties.put("value.serializer", UserSerializer.class);

// 构建kafka生产者对象
KafkaProducer<String, User> producer  = new KafkaProducer<String, User>(properties);
try {
    ProducerRecord<String, User> record;
    try {
        // 构建消息
        record = new ProducerRecord<String, User>("msb-user", "teacher", new User(1,"lijin"));
        // 发送消息
        producer.send(record);
        System.out.println("message is sent.");
        ...
```

消费者：同上部份修改即可



**问题：自定义序列化容易导致程序的脆弱性**。举例，在我们上面的实现里，我们有多种类型的消费者，每个消费者对实体字段都有各自的需求，比如，有的将字段变更为long型，有的会增加字段，这样会出现新旧消息的兼容性问题。特别是在系统升级的时候，经常会出现一部分系统升级，其余系统被迫跟着升级的情况。

解决这个问题，可以考虑使用自带格式描述以及语言无关的序列化框架。比如Protobuf，Kafka官方推荐的Apache Avro





## 三、分区

因为在Kafka中一个topic可以有多个partition，所以当一个生产发送消息，这条消息应该发送到哪个partition，这个过程就叫做分区。

- 当然，在新建消息的时候，也可以指定partition，只要指定partition，那么分区器的策略则失效，因为这个优先级最高。（一般不这么做）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/65ed2b6059b746e2b43532ca18c0369a.png)

### 一、系统分区器

在我们的代码中可以看到，生产者参数中是可以选择分区器的。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/c088dd4584bb4285b282a306abef438b.png)

默认的分区器根据key来分区。假设传送了10条相同key的消息，那么最终都会放在同一个分区内，即使配置文件里写着启动创建4个分区。

若是key + 1

```java
...
// 设置String的序列化
properties.put("key.serializer", StringSerializer.class);
properties.put("value.serializer", StringSerializer.class);
// 默认分区器
// properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, DefaultPartitioner.class);
// 轮询分区器
// properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, RoundRobinPartitioner.class);
// properties.put("partitioner.availability.timeout.ms", "0");
// properties.put("partitioner.ignore.keys", "true");
// 统一粘性分区器
properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, UniformStickyPartitioner.class);
// 构建kafka生产者对象
KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
try {
    ProducerRecord<String, String> record;
    try {
        for (int i = 0; i < 10; i++) {  // 发10条消息
            // 构建消息
            // record = new ProducerRecord<String,String>("msb", "teacher","lijin");
            // 指定partition
            record = new ProducerRecord<String, String>("msb", 0, "teacher" + i, "lijin" + i);
            // 发送消息
            Future<RecordMetadata> future = producer.send(record);
            RecordMetadata recordMetadata = future.get();
            if (null != recordMetadata) {
                System.out.println(i + "," + "offset:" + recordMetadata.offset() + ","
                        + "partition:" + recordMetadata.partition());
            }
            ....
```

输出结果为： 

```
0,offset:0,partition:0
1,offset:11,partition:3
2,offset:12,partition:3
3,offset:0,partition:1
4,offset:1,partition:1
5,offset:1,partition:0
...
```

> 以上可以看出key不相同的时候，随机在分区内存储，存储按偏移量存放，如分区3，由于之前测试了相同key存放同一个分区的实验10条数据，所以这次从11开始。



#### 1.1 DefaultPartitioner 默认分区策略

全路径类名：org.apache.kafka.clients.producer.internals.DefaultPartitioner

* 如果消息中指定了分区，则使用它
* 如果未指定分区但存在key，则根据序列化key使用murmur2哈希算法对分区数取模。
* 如果不存在分区或key，则会使用**粘性分区策略**（1.3）

采用默认分区的方式，键的主要用途有两个：

一，用来决定消息被写往主题的哪个分区，拥有相同键的消息将被写往同一个分区。

二，还可以作为消息的附加消息。

#### 1.2 RoundRobinPartitioner 分区策略

全路径类名：org.apache.kafka.clients.producer.internals.RoundRobinPartitioner

* 如果消息中指定了分区，则使用它
* **将消息平均的分配到每个分区中。**

即key为null，那么这个时候一般也会采用RoundRobinPartitioner

#### 1.3 UniformStickyPartitioner 纯粹的粘性分区策略

全路径类名：org.apache.kafka.clients.producer.internals.UniformStickyPartitioner

他跟**DefaultPartitioner** 分区策略的唯一区别就是。

**DefaultPartitionerd 如果有key的话,那么它是按照key来决定分区的,这个时候并不会使用粘性分区**
**UniformStickyPartitioner 是不管你有没有key, 统一都用粘性分区来分配**（什么是粘性分区？）

#### 1.4 另外关于粘性分区策略

从客户端最新的版本上来看（3.3.1），有两个序列化器已经进入 弃用阶段。

这个客户端在3.1.0都还不是这样。关于粘性分区策略

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/c9e0c8d6613b4b73b87e320823f76391.png)

如果感兴趣可以看下这篇文章

[https://bbs.huaweicloud.com/blogs/348729?utm_source=oschina&amp;utm_medium=bbs-ex&amp;utm_campaign=other&amp;utm_content=content]()

### 二、自定义分区器

我们完全可以去实现Partitioner接口，去实现有一个自定义的分区器

```java
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;
/**
 * 类说明：自定义分区器，以value值进行分区
 */
public class SelfPartitioner implements Partitioner {
    public int partition(String topic, Object key, byte[] keyBytes,
                         Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitionInfos = cluster.partitionsForTopic(topic);
        int num = partitionInfos.size();
        int parId = Utils.toPositive(Utils.murmur2(valueBytes)) % num;//来自DefaultPartitioner的处理
        return parId;
    }

    public void close() {
        //do nothing
    }
    public void configure(Map<String, ?> configs) {
        //do nothing
    }
}
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/f07c49100ca44560975e7f94f00713c4.png)

## 四、生产缓冲机制

>  客户端发送消息给kafka服务器的时候、消息会先写入一个内存缓冲中，然后直到多条消息组成了一个Batch，才会一次网络通信把Batch发送过去。主要有以下参数：

- **buffer.memory** 设置生产者内存缓冲区的大小

  生产者用它缓冲要发送到服务器的消息。如果数据产生速度大于向broker发送的速度，导致生产者空间不足，producer会阻塞或者抛出异常。缺省33554432 (32M)。

  所有缓存消息的总体大小超过配置的数值后，就会触发把消息发往服务器。此时会忽略batch.size和linger.ms的限制。

  默认数值是32 MB，对于单个 Producer 来说，可以保证足够的性能。 需要注意的是，如果您在同一个JVM中启动多个 Producer，那么每个 Producer 都有可能占用 32 MB缓存空间，此时便有可能触发 OOM。

- **batch.size**

  当多个消息被发送同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。当批次内存被填满后，批次里的所有消息会被发送出去。但是生产者不一定都会等到批次被填满才发送，半满甚至只包含一个消息的批次也有可能被发送（linger.ms控制）。

  缺省16384(16k) ，**如果一条消息超过了批次的大小，会写不进去。**（然后呢？这条数据怎么处理？- 被单独发送，作为独立批次发送）

- **linger.ms** 

  指定了生产者在发送批次前等待更多消息加入批次的时间。
  
  它和batch.size以先到者为先。也就是说，一旦我们获得消息的数量够batch.size的数量了，他将会立即发送而不顾这项设置，然而如果我们获得消息字节数比batch.size设置要小的多，我们需要“linger”特定的时间以获取更多的消息。
  
  这个设置**默认为0**，即没有延迟，所以没办法成批次，只有 >0 才形成批次。
  
  设定linger.ms=5，例如，将会减少请求数目，但是同时会增加5ms的延迟，但也会提升消息的吞吐量。

### 一、问题：为何要设计缓冲机制

1. 减少IO的开销（单个 ->批次）

   这种情况基本上也只是linger.ms配置 >0 的情况下才会有，原因在上方对应配置解说。

2. 减少Kafka中Java客户端的GC。(主要目的)

   说减少GC压力其实一句话就能理解：**避免创建过多对象导致年轻代频繁触发YGC（Minor GC）**，那频繁创建的是什么对象？

   比如缓冲池默认大小是32MB（buffer.memory = 32M）。然后把32MB划分为N多个内存块，比如说一个内存块是16KB（batch.size = 16384），这样的话这个缓冲池里就会有很多的内存块。缓存池要是满了，就停止收数据，等发送完清空后有空间了再收。

   批次在发送完消息后，会被清空，然后再次被使用。说的复用其实就是**内存复用**，而不特指什么对象的复用，别理解错了。
   
   **频繁创建的是什么对象？**- 1. 每发送一条消息就要创建一个新的对象（例如，ProducerRecord 或其他消息封装类）、2. 网络传输对象：缓冲区、序列化后的消息对象、3. 批量消息发送创建的批次对象、4. 日志数据的封装的对象。主要是前两个。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/5638282571c44238b3b362e3985468d5.png)

疑问：图中的想法与最上面的图一理解的有些偏差，图一是一个生产者对应一个缓存池，并且在讲解的位置还专门提到多生产者可能产生OOM问题，但是到这里这图，看起来就像是多个生产者使用同一个缓存池。怎么理解？哪一个是正确的？

**一个生产者对应着一个缓存池。**



## 五、消费者偏移量提交

一般情况下，我们调用poll方法的时候，broker返回的是生产者写入Kafka同时kafka的消费者提交偏移量，这样可以确保消费者消息消费不丢失也不重复，所以一般情况下Kafka提供的原生的消费者是安全的，但是事情会这么完美吗？

### 一、自动提交

最简单的提交方式是让消费者自动提交偏移量。 如果enable.auto.commit被设为 true，消费者会自动把从poll()方法接收到的**最大**偏移量提交上去。提交时间间隔由auto.commit.interval.ms控制，默认值是5s。

自动提交是在轮询里进行的，消费者每次在进行轮询时会检査是否该提交偏移量了，如果是，那么就会提交从上一次轮询返回的偏移量。

不过，自动提交还会有些问题：

假设我们仍然使用默认的5s提交时间间隔, 在最近一次提交之后的3s发生了再均衡，再均衡之后,消费者从最后一次提交的偏移量位置开始读取消息。这个时候偏移量已经落后了3s，所以在这3s内到达的**消息会被重复处理**。可以通过修改提交时间间隔来更频繁地提交偏移量, 减小可能出现重复消息的时间窗, 不过**这种情况是无法完全避免的**。

**自己理解**：两个消费者a、b，有两个分区z、y，他们分别这么对应a -> z, b -> y, 按照5s提交后的第3秒，分区在均衡了下变成了a->y, b-> z ，那么对于a来说，他之前在z上消费的3s偏移量没了，又从y上重新从上次提交的偏移量开始消费，然后再均衡过来的y会重复的消费者3s内的消息（GPT说没错）。重复消费发生的原因正是因为偏移量在再均衡后无法同步到准确位置，导致消费者从稍早的偏移量重新开始消费消息。

在使用自动提交时,每次调用轮询方法都会把上一次调用返回的最大偏移量提交上去,它并不知道具体哪些消息已经被处理了,所以在再次调用之前最好确保所有当前调用返回的消息都已经处理完毕(enable.auto.comnit被设为 true时，在调用 close()方法之前也会进行自动提交)。一般情况下不会有什么问题,不过在处理异常或提前退出轮询时要格外小心。

**解决方案：**

这种情况的处理主要依赖于 **幂等性** 和 **偏移量管理**。虽然无法完全避免重复消费，但可以通过一些技术手段来**减少**重复消费的影响，或者**保证处理结果的一致性**。

**1. 幂等性处理**

幂等性是解决重复消费问题的关键。如果消费者端的处理逻辑是幂等的，即使某些消息被重复消费，也不会影响最终结果。

- **幂等性设计**：

  - 在业务逻辑中加入去重机制。例如，使用唯一的**消息 ID**或者**外部唯一标识符**来确保每条消息只处理一次。即使消费者重复消费相同的消息，也不会执行重复的操作。

  - 比如，在数据库中可以通过**唯一约束**来确保同一条消息只会插入一次，或者通过**幂等性 API**来确保同一请求只会被处理一次。

- **案例：**
  - 假设你在处理订单时使用了订单的唯一 ID。无论订单是否被重复消费，系统都只会处理一次（插入数据库时会检查订单 ID 是否已经存在，防止重复插入）。
- 额外补充知识点：“幂等性”（Idempotence）是一个数学和计算机科学中的概念，指的是“多次执行相同的操作，结果是相同的，不会发生副作用”。（详情见最底部）

**2. 提高提交偏移量的频率**

频繁提交偏移量有助于减少由于再均衡导致的偏移量丢失问题。这样消费者就能更快地从正确的地方恢复消费。

- **增加提交间隔的频率：**
  - 比如，将提交偏移量的时间间隔从 5 秒缩短为 1 秒或更短。虽然这可以减少因偏移量延迟提交而导致的消息重复消费的时间窗，但频繁提交偏移量会增加网络负担，因此需要权衡。

- **手动提交偏移量：**
  - 如果使用的是自动提交偏移量（默认情况下 Kafka 会自动提交），你可以改为手动提交偏移量，这样可以确保偏移量仅在消息成功处理后提交，从而避免提交不必要的偏移量。

**3. 使用 Kafka 的 Exactly Once 语义**

从 Kafka 0.11 版本开始，Kafka 支持 **Exactly Once 语义**（精确一次语义），确保每条消息恰好被消费一次，既不丢失，也不重复。

- **如何使用 Exactly Once：**

  - 对于生产者：Kafka 支持幂等的生产者，它会自动去重，确保每条消息只生产一次。

  - 对于消费者：消费者可以开启事务性处理来实现精确一次语义，确保消息处理的幂等性。

- **配置方式：**

  - 在生产者端，启用幂等性生产模式，通过 acks=all 和 enable.idempotence=true 配置来确保消息的唯一性。

  - 在消费者端，使用事务模式，通过设置 isolation.level=read_committed 来确保消息处理的幂等性。
  
    事务（Transactions）
  
    - 作用：将生产消息和消费消息的操作绑定为一个原子操作。
    - 场景：适用于需要读写跨多个 Topic/Partition 的原子性操作。
    - 配置：
      - 生产者：设置 `transactional.id`，并调用 `beginTransaction()`, `commitTransaction()`。
      - 消费者：设置 `isolation.level=read_committed`，只读取已提交的事务消息。

#### 1.1 消费者的配置参数

**auto.offset.reset**

- earliest
  当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
- latest
  当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据。只要group.Id不变，不管auto.offset.reset 设置成什么值，都从上一次的消费结束的地方开始消费。



### 二、手动提交

将配置`enable.auto.commit=false`，然后在处理完业务逻辑后手动提交

```java
// 构建kafka消费者对象
KafkaConsumer<String,String> consumer = new KafkaConsumer<String, String>(properties);
try {
    consumer.subscribe(Collections.singletonList("msb"));
    // 调用消费者拉取消息
    while(true){
        // 每隔1秒拉取一次消息
        ConsumerRecords<String, String> records= consumer.poll(Duration.ofSeconds(1));
        for(ConsumerRecord<String, String> record:records){
            String key = record.key();
            String value = record.value();
            System.out.println("接收到消息: key = " + key + ", value = " + value);
        }
        consumer.commitAsync();//异步提交：不阻塞我们的应用程序的线程，不会重试（有可能失败）
    }
}catch (CommitFailedException e) {
    System.out.println("Commit failed:");
    e.printStackTrace();
}finally {
    try {
        consumer.commitSync();//同步提交： 会阻塞我们的应用的线程，并且会重试（一定会成功）
    } finally {
        consumer.close();
    }
}
```

1. 异步提交

   `comsumer.commitAsync()`：建议在主流程处理业务内

2. 同步提交

   `consumer.commitSync()`







#### 幂等性

在编程和系统设计中，**幂等性**通常指的是某个操作无论执行多少次，最终的效果是一样的，也就是说，即使同一操作被重复执行，也不会改变系统的最终状态。

**举个简单的例子：**

假设你有一个**增值操作**，如向账户余额添加 100 元，假设账户余额原本是 500 元。

- **非幂等操作：** 如果你多次执行这个操作，账户余额会不断增加。比如第一次执行后账户余额变成 600 元，再执行一次变成 700 元，这显然是非幂等的，因为操作会产生副作用（余额不断增加）。

- **幂等操作：** 假设你设置账户余额为固定值（例如，账户余额最大为 500 元），无论你多少次执行**设置账户余额为 500 元**的操作，余额都保持 500 元，不会受操作次数的影响。这就是幂等的。

**在分布式系统和消息处理中，幂等性非常重要，因为系统可能会因为网络问题、崩溃、重试等原因，导致某些操作重复执行。幂等性确保即使操作被重复执行，也不会产生不一致或错误的结果。**

**幂等性的应用：**

1. **数据库操作：**

- 比如插入一条记录时，如果已经插入过这条记录，再次插入时不做任何操作，确保数据库的状态不会受到影响。可以通过唯一键（例如订单 ID）来保证这一点。

2. **API 请求：**

- 例如，向服务器发起一个“创建订单”的请求，如果请求被重复提交（例如由于网络抖动导致的重试），订单不会被重复创建，而是返回相同的结果（例如返回已经创建的订单信息）。

3. **消息队列：**

- 在消息队列系统中，如果消息被重复消费，消费者的处理操作应该是幂等的，例如更新库存时，只更新一次，而不会因为多次处理同一消息而导致库存数目增加多次。



**幂等性在实际场景中的解决方法：**

- **使用唯一标识符：** 每条请求或消息都有一个唯一的 ID，处理时检查是否已处理过，避免重复处理。

- **事务或锁机制：** 确保操作的原子性，避免多次执行相同操作。

- **数据库唯一约束：** 在数据库中，使用唯一约束或索引来防止重复插入相同数据。
