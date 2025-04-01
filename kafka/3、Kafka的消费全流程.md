# Kafka的消费全流程

我们接着继续去理解最后这条消息是如何被消费者消费掉的。其中最核心的有以下内容。

1、多线程安全问题

2、群组协调

3、分区再均衡

# 一、多线程安全问题

## 一、生产者

KafkaProducer的实现是线程安全的。

KafkaProducer就是一个不可变类。线程安全的，可以在多个线程中共享单个KafkaProducer实例

所有字段用private final修饰，且不提供任何修改方法，这种方式可以确保多线程安全。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/c439a638df354357b3dc857f7cc83e86.png)

如何节约资源的多线程使用KafkaProducer实例

```java
package com.msb.concurrent;

import com.msb.selfserial.User;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 类说明：多线程下使用生产者
 */
public class KafkaConProducer {
    //发送消息的个数
    private static final int MSG_SIZE = 1000;
    //负责发送消息的线程池
    private static ExecutorService executorService = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    private static CountDownLatch countDownLatch  = new CountDownLatch(MSG_SIZE);

    private static User makeUser(int id) {
        User user = new User(id);
        String userName = "msb_"+id;
        user.setName(userName);
        return user;
    }

    /*发送消息的任务*/
    private static class ProduceWorker implements Runnable{
        private ProducerRecord<String,String> record;
        private KafkaProducer<String,String> producer;

        public ProduceWorker(ProducerRecord<String, String> record, KafkaProducer<String, String> producer) {
            this.record = record;
            this.producer = producer;
        }

        public void run() {
            final String id = Thread.currentThread().getId() +"-"+System.identityHashCode(producer);
            try {
                producer.send(record, new Callback() {
                    public void onCompletion(RecordMetadata metadata, Exception exception) {
                        if(null!=exception){
                            exception.printStackTrace();
                        }
                        if(null!=metadata){
                            System.out.println(id+"|" +String.format("偏移量：%s,分区：%s", metadata.offset(),
                                    metadata.partition()));
                        }
                    }
                });
                System.out.println(id+":数据["+record+"]已发送。");
                countDownLatch.countDown();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

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
            for(int i=0;i<MSG_SIZE;i++){
                User user = makeUser(i);
                ProducerRecord<String,String> record = new ProducerRecord<String,String>("concurrent-test",null,System.currentTimeMillis(), user.getId()+"", user.toString());
                executorService.submit(new ProduceWorker(record,producer));
            }
            countDownLatch.await();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.close();
            executorService.shutdown();
        }
    }
}
```

## 二、消费者

KafkaConsumer的实现**不是**线程安全的

实现消费者多线程最常见的方式： **线程封闭** ——即**为每个线程实例化一个 KafkaConsumer对象**

```java
package com.msb.concurrent;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 类说明：多线程下正确的使用消费者，需要记住，一个线程一个消费者
 */
public class KafkaConConsumer {
    public static final int CONCURRENT_PARTITIONS_COUNT = 2;
    
    private static ExecutorService executorService = Executors.newFixedThreadPool(CONCURRENT_PARTITIONS_COUNT);

    private static class ConsumerWorker implements Runnable{
        private KafkaConsumer<String,String> consumer;

        public ConsumerWorker(Map<String, Object> config, String topic) {
            Properties properties = new Properties();
            properties.putAll(config);
            this.consumer = new KafkaConsumer<String, String>(properties);
            consumer.subscribe(Collections.singletonList(topic));
        }

        public void run() {
            final String ThreadName = Thread.currentThread().getName();
            try {
                while(true){
                    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                    for(ConsumerRecord<String, String> record:records){
                        System.out.println(ThreadName+"|"+String.format(
                                "主题：%s，分区：%d，偏移量：%d，" +
                                        "key：%s，value：%s",
                                record.topic(),record.partition(),
                                record.offset(),record.key(),record.value()));
                        //do our work
                    }
                }
            } finally {
                consumer.close();
            }
        }
    }

    public static void main(String[] args) {
        /*消费配置的实例*/
        Map<String,Object> properties = new HashMap<String, Object>();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"127.0.0.1:9092");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class);
        properties.put(ConsumerConfig.GROUP_ID_CONFIG,"c_test");
        properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"earliest");

        for(int i = 0; i<CONCURRENT_PARTITIONS_COUNT; i++){
            //一个线程一个消费者
            executorService.submit(new ConsumerWorker(properties, "concurrent-test"));
        }
    }
}
```



# 二、群组协调

消费者要加入群组时，会向群组协调器发送一个JoinGroup请求，第一个加入群组的消费者成为群主，群主会获得群组的成员列表，并负责给每一个消费者分配分区。

分配完毕后，群主把分配情况发送给群组协调器，协调器再把这些信息发送给所有的消费者，每个消费者只能看到自己的分配信息，只有群主知道群组里所有消费者的分配信息。**群组协调的工作会在消费者发生变化(新加入或者掉线)、主题中分区发生了变化（增加）时 发生。**

Leader消费者的选举

- **触发条件**：当消费者组**首次启动**或**发生重平衡**时，所有消费者会向协调器发送`JoinGroup`请求。
- **选举规则**：
  - 协调器从当前加入的消费者中选择一个作为**Leader消费者**，通常选择**第一个成功发送`JoinGroup`请求的消费者**（但严格来说，协调器可能根据内部逻辑选择，例如消费者ID的字典序）。
  - Leader的选举**与分区分配策略无关**，仅用于组织本次重平衡的分区分配流程。

分区分配流程

1. **JoinGroup阶段**：

   - 所有消费者发送`JoinGroup`请求，协调器收集它们的元数据（如订阅的Topic、分配策略支持情况等）。
   - 协调器选举Leader消费者，并将**组成员列表**（包括所有消费者的信息）返回给Leader。

2. **SyncGroup阶段**：

   - **Leader消费者**根据分区分配策略（如Range、RoundRobin等），为所有消费者计算分区分配方案。

   - Leader通过`SyncGroup`请求将分配方案提交给协调器。

   - 协调器将分配方案分发给**所有消费者**，每个消费者仅收到自己分配到的分区列表。

   - SyncGroup 所有交互均由客户端主动发起请求，服务端被动响应。但不同角色消费者在 SyncGroup 请求中携带的数据不同

     | 消费者角色 | SyncGroup 请求内容                     | SyncGroup 响应内容           |
     | :--------- | :------------------------------------- | :--------------------------- |
     | Leader     | 携带完整的分区分配方案（`assignment`） | 无实际数据（仅确认提交成功） |
     | Follower   | 空（不携带任何分配信息）               | 自身分配到的分区列表         |

     - 协调器只是**暂存** Leader 提交的数据，不会主动推送，必须等待 Follower 主动拉取。

示例流程

假设一个消费者组有3个消费者（C1、C2、C3）：

1. **重平衡触发**：C1率先发送`JoinGroup`请求。
2. **选举Leader**：协调器选择C1为Leader，返回组成员列表（C1、C2、C3）。
3. **分区分配**：C1根据策略（如RoundRobin）计算分配方案，提交给协调器。
4. **结果同步**：协调器通过`SyncGroup`响应将分配结果分别发给C1、C2、C3。



![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/b070d19ea7584a088c03fed8acfb0b96.png)



```mermaid
sequenceDiagram
    participant Leader as Leader消费者
    participant Follower1 as Follower消费者1
    participant Follower2 as Follower消费者2
    participant Coordinator as 组协调器

    Leader->>Coordinator: SyncGroup请求（携带全部分配方案）
    Coordinator-->>Leader: SyncGroup响应（确认提交）

    Follower1->>Coordinator: SyncGroup请求（空）
    Coordinator-->>Follower1: SyncGroup响应（分配结果）

    Follower2->>Coordinator: SyncGroup请求（空）
    Coordinator-->>Follower2: SyncGroup响应（分配结果）
```

问题澄清：

Q：如果 Follower 先于 Leader 发送 `SyncGroup` 请求会怎样？

- **协调器会阻塞该请求**，直到收到 Leader 提交的分配方案。若等待超时（`session.timeout.ms`），会触发新一轮重平衡。

Q：Leader 提交分配方案后，协调器宕机怎么办？

- 消费者组会重新选举协调器，并从 `__consumer_offsets` 主题中恢复组状态。若 Leader 已提交分配方案但未同步给所有消费者，会触发重平衡。

Q：为什么 Follower 需要主动拉取，不能由 Leader 直接分发？

- **安全性**：若 Leader 直接分发，需确保所有 Follower 可信，且网络拓扑可直达，这在分布式系统中不现实。
- **去中心化**：通过协调器中转，保证分配结果的权威性和一致性。



## 一、组协调器

**组协调器是Kafka服务端自身维护的。**

组协调器(**GroupCoordinator**)可以理解为`各个消费者协调器的一个中央处理器`, 每个消费者的所有交互都是和组协调器进行的。

选举Leader消费者客户端

**组协调器的主要职责：**

1. **消费者群组的加入与离开**

   - 当一个消费者启动并想要加入某个消费者群组时，它会向 Kafka 集群中的组协调器发送请求。组协调器会将该消费者加入群组，并决定它消费哪些分区。
   - 如果某个消费者崩溃或主动离开群组，组协调器会接收到该事件，并通过重新平衡（rebalance）将该消费者原先消费的分区分配给其他消费者。

2. **分配分区**

   - 组协调器负责将分区分配给消费者群组中的各个消费者。根据消费者数量和分区数量，协调器会决定每个消费者需要消费哪些分区。
   - 分区的分配是动态的，如果群组内的消费者数量或分区数量发生变化，组协调器会进行再平衡（rebalance）。

3. **管理消费者的偏移量**

   - 消费者会定期向组协调器提交其当前的消费偏移量（即已消费到消息的位置信息）。这样，Kafka 可以确保在消费者故障或重启时，消费者能够从正确的位置继续消费。

   - 偏移量一般会被保存在 Kafka 内置的主题 `__consumer_offsets` 中，由组协调器管理。

     - **kafka上的组协调器有很多**，有多少个 `__consumer_offset`分区, 那么就有多少个组协调器。

       默认情况下, `__consumer_ offset`有50个分区, 每个消费组都会对应其中的一个分区，对应的逻辑为 hash(`group.id`)%分区数。

4. **负载均衡（Rebalance 重平衡）**

   - 每当消费者数量发生变化（例如某个消费者加入或退出群组）时，组协调器会启动负载均衡（rebalance）过程。在这个过程中，消费者群组中的分区会被重新分配，以确保每个消费者负担合理。
   - ==负载均衡通常会导致消费者在某段时间内暂停消费消息==，直到新的分区分配完成。

5. **处理消费者的心跳**

   - **参数**：

     - `session.timeout.ms`：协调器等待心跳的超时时间（默认10秒），超时则剔除消费者。
     - `heartbeat.interval.ms`：消费者发送心跳的频率（默认3秒），需小于`session.timeout.ms`。
     - `max.poll.interval.ms`：消费者处理消息的最大间隔（默认5分钟），超时触发离开组。
     
   - **作用**：维持消费者会话，避免误判离线。
   
     

**组协调器的工作流程：**

1. **消费者启动并加入群组：**
   - 消费者启动时，会向 Kafka 集群中的组协调器发送请求，申请加入某个消费者群组。
   - 组协调器接收到加入请求后，会检查群组内当前的消费者和分区信息，并将分区分配给新的消费者。
2. **负载均衡（Rebalance 重平衡）：**
   - 如果消费者群组内的消费者数量变化（例如，新消费者加入或已有消费者退出），组协调器会启动负载均衡操作，重新分配分区。
   - 在负载均衡过程中，`消费者会被暂停消费`，直到新的分配完成。
3. **消费者提交偏移量：**
   - 消费者会定期向组协调器提交已消费消息的偏移量。组协调器将这些偏移量存储在内置的 `__consumer_offsets` 主题中，以便在消费者崩溃或重启时能够恢复消费。（提交是定期提交，但如果在这时候突然奔溃了怎么半？消费一半没提交的记录，机子重新恢复后不就又重新消费了？--答：消费者协调器自己本地也有对应的consumer_offsets保存当前的记录）
4. **消费者心跳与失效检测：**
   - 消费者会定期向组协调器发送心跳信号，确认自己仍然活跃。如果心跳超时，组协调器会认为该消费者已失效，触发再平衡，将其分配的分区分配给其他消费者。





## 二、消费者协调器

**每个客户端（消费者的客户端）都会有一个消费者协调器**，他的主要作用就是向组协调器发起请求做交互，以及处理回调逻辑

1. 向组协调器发起入组请求
2. 向组协调器发起同步组请求(如果是Leader客户端，则还会计算分配策略数据放到入参传入)
3. 发起离组请求
4. 保持跟组协调器的心跳线程
5. 向组协调器发送提交已消费偏移量的请求



```mermaid
sequenceDiagram
    participant ConsumerCoordinator as 消费者协调器（客户端）
    participant GroupCoordinator as 组协调器（Broker）

    ConsumerCoordinator->>GroupCoordinator: FindCoordinator 请求
    GroupCoordinator-->>ConsumerCoordinator: 返回组协调器地址

    loop 消费者组生命周期
        ConsumerCoordinator->>GroupCoordinator: JoinGroup 请求
        GroupCoordinator-->>ConsumerCoordinator: 返回组成员信息（Leader/Follower）

        alt 是 Leader
            ConsumerCoordinator->>ConsumerCoordinator: 计算分区分配方案
            ConsumerCoordinator->>GroupCoordinator: SyncGroup 请求（携带分配方案）
        else 是 Follower
            ConsumerCoordinator->>GroupCoordinator: SyncGroup 请求（空）
        end

        GroupCoordinator-->>ConsumerCoordinator: 返回分区分配结果

        loop 心跳维持
            ConsumerCoordinator->>GroupCoordinator: Heartbeat 请求
            GroupCoordinator-->>ConsumerCoordinator: 确认存活
        end
    end
```

## 三、消费者加入分组的流程

1. 客户端启动的时候, 或者重连的时候会发起JoinGroup的请求来申请加入的组中。
2. 当前客户端都已经完成JoinGroup之后，客户端会收到JoinGroup的回调，然后客户端会再次向组协调器发起SyncGroup的请求来获取新的分配方案
   1. 当消费者客户端关机/异常 时，会触发离组LeaveGroup请求。
   2. 当然有主动的消费者协调器发起离组请求，也有组协调器一直会有针对每个客户端的心跳检测，如果监测失败，则就会将这个客户端踢出Group。
3. 客户端加入组内后，会一直保持一个心跳线程，来保持跟组协调器的一个感知。并且组协调器会针对每个加入组的客户端做一个心跳监测，如果监测到过期，则会将其踢出组内并再平衡。



**常见误解**

- **误解**：“协调器直接分配分区。”
  **纠正**：协调器仅传递分配结果，实际分配由Leader消费者计算。
- **误解**：“Leader消费者必须处理更多数据。”
  **纠正**：Leader角色仅用于协调分配，其消费的分区数量与其他消费者一致。



## 四、消费者消费的offset的存储

__consumer_offsets topic，并且默认提供了kafka_consumer_groups.sh脚本供用户查看consumer信息。
__consumer_offsets 是 kafka 自行创建的，和普通的 topic 相同。它存在的目的之一就是保存 consumer 提交的位移。

```
kafka-consumer-groups.bat --bootstrap-server :9092 --group c_test --describe
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/2589b72c9be7424bbb87a28a800d0e80.png)



那么如何使用 kafka 提供的脚本查询某消费者组的元数据信息呢？

```
Math.abs(groupID.hashCode()) % numPartitions，
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/c191a390dffc46a99dc99842960c35a1.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/6c84975c90324927ba9733b6ca6ff426.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/f13949780b95482481e81b07d8f8fde2.png)

__consumer_offsets 的每条消息格式大致如图所示

可以想象成一个 KV 格式的消息，key 就是一个三元组：`group.id+topic+分区号`，而 value 就是 offset 的值

# 三、分区再均衡

当消费者群组里的消费者发生变化，或者主题里的分区发生了变化，都会导致再均衡现象的发生。从前面的知识中，我们知道，Kafka中，存在着消费者对分区所有权的关系，

这样无论是消费者变化，比如增加了消费者，新消费者会读取原本由其他消费者读取的分区，消费者减少，原本由它负责的分区要由其他消费者来读取，增加了分区，哪个消费者来读取这个新增的分区，这些行为，都会导致分区所有权的变化，这种变化就被称为 **再均衡** 。

再均衡对Kafka很重要，这是消费者群组带来高可用性和伸缩性的关键所在。**不过一般情况下，尽量减少再均衡，因为再均衡期间，消费者是无法读取消息的，会造成整个群组一小段时间的不可用。**

消费者通过向称为群组协调器的broker（不同的群组有不同的协调器）发送心跳来维持它和群组的从属关系以及对分区的所有权关系。如果消费者长时间不发送心跳，群组协调器认为它已经死亡，就会触发一次再均衡。

心跳由单独的线程负责，相关的控制参数为max.poll.interval.ms。

## 消费者提交偏移量导致的问题

当我们调用poll方法的时候，broker返回的是生产者写入Kafka但是还没有被消费者读取过的记录，消费者可以使用Kafka来追踪消息在分区里的位置，我们称之为 **偏移量** 。消费者更新自己读取到哪个消息的操作，我们称之为 **提交** 。

消费者是如何提交偏移量的呢？消费者会往一个叫做_consumer_offset的特殊主题发送一个消息，里面会包括每个分区的偏移量。发生了再均衡之后，消费者可能会被分配新的分区，为了能够继续工作，消费者者需要读取每个分区最后一次提交的偏移量，然后从指定的地方，继续做处理。

**分区再均衡的例子：**

某软件公司，有一个项目，有两块的工作，有两个码农，一个小王、一个小李，一个负责一块（分区消费），干得好好的。突然一天，小王桌子一拍不干了，老子中了5百万了，不跟你们玩了，立马收拾完电脑就走了。这个时候小李就必须承担两块工作，这个时候就是发生了分区再均衡。

过了几天，你入职，一个萝卜一个坑，你就入坑了，你承担了原来小王的工作。这个时候又会发生了分区再均衡。

1）如果提交的偏移量`小于`消费者实际处理的最后一个消息的偏移量，处于两个偏移量之间的消息会被重复处理，

2）如果提交的偏移量`大于`客户端处理的最后一个消息的偏移量,那么处于两个偏移量之间的消息将会丢失

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/1f787d4a41484ab28e617decfe094b38.png)

### 再均衡监听器实战

我们创建一个分区数是3的主题rebalance

```
kafka-topics.bat --bootstrap-server localhost:9092  --create --topic rebalance --replication-factor 1 --partitions 3
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/7af6e805d1d14056bdfd5ae0634d71a7.png)

在为消费者分配新分区或移除旧分区时,可以通过消费者API执行一些应用程序代码，在调用 subscribe()方法时传进去一个 ConsumerRebalancelistener实例就可以了。

ConsumerRebalancelistener有两个需要实现的方法。

1) public void onPartitionsRevoked( Collection< TopicPartition> partitions)方法会在再均衡开始之前和消费者停止读取消息之后被调用。如果在这里提交偏移量，下一个接管分区的消费者就知道该从哪里开始读取了

2) public void onPartitionsAssigned( Collection< TopicPartition> partitions)方法会在重新分配分区之后和消费者开始读取消息之前被调用。

具体使用，我们先创建一个3分区的主题，然后实验一下，

在再均衡开始之前会触发onPartitionsRevoked 方法

在再均衡开始之后会触发onPartitionsAssigned 方法



对应代码在 rebalance





**彻底解决问题**需结合：

- 合理的手动提交策略（同步+异步）。

  ```java
  props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false"); // 关闭自动提交
  
  consumer.subscribe(topics, new ConsumerRebalanceListener() {
      @Override
      public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
          // 同步提交偏移量，确保提交成功
          consumer.commitSync();
      }
  
      @Override
      public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
          // 从外部存储加载偏移量（可选）
          partitions.forEach(partition -> 
              consumer.seek(partition, getOffsetFromDB(partition)));
      }
  });
  
  while (true) {
      ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
      for (ConsumerRecord<String, String> record : records) {
          processRecord(record);
      }
      consumer.commitAsync(); // 异步提交（或同步提交）
  }
  ```

- 处理逻辑的幂等性。

- 可能的**外部偏移量存储**（如数据库）
