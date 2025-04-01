## Redis的Java常用客户端

Redis 官方推荐的Java 客户端 Jedis、lettuce 和 Redisson

### Jedis

老牌的Redis 的Java客户端，提供了比较全面的Redis命令的支持

**优点：**

API比较全面（参考：[https://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html](https://tool.oschina.net/uploads/apidocs/redis/clients/jedis/Jedis.html)）

**缺点：**

使用阻塞的 I/O（方法调用都是同步的，程序流需要等到 sockets 处理完 I/O 才能执行，不支持异步）

Jedis 客户端实例不是线程安全的（多线程使用一个Jedis连接），所以需要通过连接池来使用Jedis（每个线程使用独自的Jedis连接）

### lettuce

lettuce是基于netty实现的与redis进行同步和异步的通信。

在spring boot2之后，redis连接默认就采用了lettuce（spring-boot-starter-data-redis）

官网：[https://lettuce.io/](https://lettuce.io/)

github：[https://github.com/lettuce-io/lettuce-core](https://github.com/lettuce-io/lettuce-core)

SpringData：[https://spring.io/projects/spring-data-redis](https://spring.io/projects/spring-data-redis)

**优点：**

线程安全的 Redis 客户端，支持异步模式

lettuce 底层基于 Netty，支持高级的 Redis 特性，比如哨兵，集群，管道，自动重新连接和Redis数据模型。

**缺点：**

没人知道... API比较复杂

### Redisson

Redisson 提供了使用Redis 的最简单和最便捷的方法，还提供了许多分布式服务（分布式锁，分布式集合，延迟队列等）

**优点：**

Redisson基于Netty框架的事件驱动的通信层，其方法调用是异步的

Redisson的API是线程安全的，所以可以操作单个Redisson连接来完成各种操作

**缺点：**

Redisson 对字符串的操作支持比较差

推荐文档：[https://redisson.org/docs/](https://redisson.org/docs/)

[redisson中文API](https://github.com/redisson/redisson/wiki/8.-%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E5%92%8C%E5%90%8C%E6%AD%A5%E5%99%A8#84-%E7%BA%A2%E9%94%81redlock)

## 项目整合spring-boot-starter-data-redis

&emsp;&emsp;要整合Redis那么我们在SpringBoot项目中首页来添加对应的依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

&emsp;&emsp;然后我们需要添加对应的配置信息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1462/1647243560000/753f65df9029411a8145b2477d1e58d3.png)

测试操作Redis的数据

```java
    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Test
    public void testStringRedisTemplate(){
        // 获取操作String类型的Options对象
        ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        // 插入数据
        ops.set("name","lijin"+ UUID.randomUUID());
        // 获取存储的信息
        System.out.println("刚刚保存的值："+ops.get("name"));
    }
```

查看可以通过Redis的客户端连接查看

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1462/1647243560000/1b678ea26d684404a93217b8bb781164.png)

也可以通过工具查看（大家自行搜索去找，有收费的，有免费的）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/1462/1647243560000/33e044f827f0470bb9f3a27308f77118.png)

## 项目整合Redisson

添加对应的依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.16.1</version>
</dependency>
```

添加对应的配置类

```java
@Configuration
public class MyRedisConfig {
    @Bean
    public RedissonClient redissonClient(){
        Config config = new Config();
        // 配置连接的信息
        config.useSingleServer()
                .setAddress("redis://192.168.56.100:6379");
        RedissonClient redissonClient = Redisson.create(config);
        return  redissonClient;
    }
}
```

案例代码

POM文件 片段

```xml
<!-- spring-redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- redisson -->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.30.0</version>
</dependency>
<!-- jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.6.3</version>
</dependency>
```

test代码

```java
import org.junit.jupiter.api.Test;
import org.redisson.api.*;
import org.redisson.client.protocol.ScoredEntry;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.ZSetOperations;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Tuple;

import java.util.Collection;
import java.util.List;
import java.util.Set;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import lombok.extern.slf4j.Slf4j;

@SpringBootTest
@Slf4j
public class TestClient {
    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    StringRedisTemplate stringRedisTemplate;

    @Autowired
    private RedissonClient redisson;

    @Test
    public void JedisDemo() {

        Jedis jedis =  new Jedis("127.0.0.1",6379);

        // 设置字符串值
        jedis.del("key1");
        jedis.set("key1", "value1");
        // 获取字符串值
        String value = jedis.get("key1");
        System.out.println(value);

        // 向列表中插入元素
        jedis.del("list1");
        jedis.lpush("list1", "element1", "element2", "element3");
        // 获取列表中的所有元素
        List elements = jedis.lrange("list1", 0, -1);
        System.out.println(elements);

        // 向集合中添加元素
        jedis.del("set1");
        jedis.sadd("set1", "member1", "member2", "member3");
        // 获取集合中的所有元素
        Set members = jedis.smembers("set1");
        System.out.println(members);


        // 设置哈希字段值
        jedis.del("hash1");
        jedis.hset("hash1", "field1", "value1");
        // 获取哈希字段值
        String hashValue = jedis.hget("hash1", "field1");
        System.out.println(hashValue);


        // 向有序集合中添加元素
        jedis.del("zset1");
        jedis.zadd("zset1", 100, "player1");
        jedis.zadd("zset1", 200, "player2");
        jedis.zadd("zset1", 150, "player3");

        // 获取有序集合中的排名榜
        Set<Tuple> ranking = jedis.zrevrangeWithScores("zset1", 0, -1);

        // 展示排名榜
        int rank = 1;
        for (Tuple tuple : ranking) {
            String player = tuple.getElement();
            double score = tuple.getScore();
            System.out.println("排名：" + rank + "，玩家：" + player + "，得分：" + score);
            rank++;
        }
    }
    @Test
    public void RedisTemplateDemo() {


        // 设置字符串值
        redisTemplate.delete("key1");
        redisTemplate.opsForValue().set("key1", "value1");

        stringRedisTemplate.opsForValue().increment("lijin",1);

        // 获取字符串值
        String value = redisTemplate.opsForValue().get("key1").toString();
        System.out.println(value);

        // 向列表中插入元素
        redisTemplate.delete("list1");
        redisTemplate.opsForList().leftPushAll("list1", "element1", "element2", "element3");
        // 获取列表中的所有元素
        List<String> elements = redisTemplate.opsForList().range("list1", 0, -1);
        System.out.println(elements);

        // 向集合中添加元素
        redisTemplate.delete("set1");
        redisTemplate.opsForSet().add("set1", "member1", "member2", "member3");
        // 获取集合中的所有元素
        Set<String> members = redisTemplate.opsForSet().members("set1");
        System.out.println(members);

        // 设置哈希字段值
        redisTemplate.delete("hash1");
        redisTemplate.opsForHash().put("hash1", "field1", "value1");
        // 获取哈希字段值
        String hashValue = (String) redisTemplate.opsForHash().get("hash1", "field1");
        System.out.println(hashValue);

        // 向有序集合中添加元素
        ZSetOperations<String, Object> zSetOperations = redisTemplate.opsForZSet();


        redisTemplate.delete("zset1");
        zSetOperations.add("zset1", "player1", 100);
        zSetOperations.add("zset1", "player2", 200);
        zSetOperations.add("zset1", "player3", 150);
        // 获取有序集合中的排名榜
        Set<ZSetOperations.TypedTuple<Object>> ranking = zSetOperations.reverseRangeWithScores("zset1", 0, -1);
        // 展示排名榜
        int rank = 1;
        for (ZSetOperations.TypedTuple<Object> tuple : ranking) {
            String player = tuple.getValue().toString();
            double score = tuple.getScore();
            System.out.println("排名：" + rank + "，玩家：" + player + "，得分：" + score);
            rank++;
        }
    }

    @Test
    public void RedissonDemo() {
        // 设置字符串值
        redisson.getBucket("key1").delete();
        redisson.getBucket("key1").set("value1");
        // 获取字符串值
        String value = redisson.getBucket("key1").toString();
        System.out.println(value);

        // 向列表中插入元素
        RList<String> list = redisson.getList("list1");
        list.delete();
        list.add("element1");
        list.add("element2");
        list.add("element3");
        // 获取列表中的所有元素
        List<String> elements = list.readAll();
        System.out.println(elements);

        // 向集合中添加元素
        RSet<String> set = redisson.getSet("set1");
        set.delete();
        set.add("member1");
        set.add("member2");
        set.add("member3");
        // 获取集合中的所有元素
        Set<String> members = set.readAll();
        System.out.println(members);

        // 设置哈希字段值
        RMap<String, String> map = redisson.getMap("hash1");
        map.delete();
        map.put("field1", "value1");
        // 获取哈希字段值
        String hashValue = map.get("field1");
        System.out.println(hashValue);

        // 向有序集合中添加元素
        RScoredSortedSet<String> zset = redisson.getScoredSortedSet("zset1");
        zset.delete();
        zset.add(100, "player1");
        zset.add(200, "player2");
        zset.add(150, "player3");
        // 获取有序集合中的排名榜
        Collection<ScoredEntry<String>> ranking = zset.entryRangeReversed(0, -1);
        // 展示排名榜
        int rank = 1;
        for (ScoredEntry<String> entry : ranking) {
            String player = entry.getValue();
            double score = entry.getScore();
            System.out.println("排名：" + rank + "，玩家：" + player + "，得分：" + score);
            rank++;
        }
    }


    @Test
    public void LockExample() {
        long goods_id =13;
        String Lockkey ="goods-number"+goods_id;
        RLock lock = redisson.getLock(Lockkey);  //Lockkey 这个是标识 锁的名称
        lock.lock();//这种代码下 是会开启看门狗的（默认的10秒运行一次 进行续锁， 这种Redis的TTL30秒）
        //lock.lock(30, TimeUnit.SECONDS); //这种就不会启动看门狗了

        log.info("业务代码1");

        try {
            Thread.sleep(300000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
       }
        log.info("业务代码2");

        lock.unlock();
        log.info("业务代码3");

    }

    @Test
    public void AtomicExample() throws Exception{
        RAtomicLong atomicLong = redisson.getAtomicLong("myAtomicLong");
        atomicLong.set(3);
        atomicLong.incrementAndGet();
        log.info(String.valueOf(atomicLong.get()));

//        RAtomicDouble atomicDouble = redisson.getAtomicDouble("myAtomicDouble");
//        atomicDouble.set(2.81);
//        atomicDouble.addAndGet(4.11);
//        atomicDouble.get();
//
//        RLongAdder atomicLong2 = redisson.getLongAdder("myLongAdder");
//        for (int i=0;i<10;i++){
//            Thread thread = new Thread(()->{
//                atomicLong2.increment();
//            });
//            thread.start();
//        }
//
//        Thread.sleep(1000);
//        log.info(String.valueOf(atomicLong2.sum()));
    }

    @Test
    public void RateLimiterExample() throws Exception {
        RRateLimiter rateLimiter = redisson.getRateLimiter("myRateLimiter");
        // 初始化
        // 最大流速 = 每5秒钟产生10个令牌
        rateLimiter.trySetRate(RateType.OVERALL, 10, 5, RateIntervalUnit.SECONDS);

        try {
            Thread.sleep(3000000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        while(true){
            rateLimiter.acquire(1);
            Thread.sleep(500);
            log.info("获取令牌成功");
        }

    }

    @Test
    public void CountDownLatchExample() throws Exception{
        RCountDownLatch latch = redisson.getCountDownLatch("anyCountDownLatch");
        latch.trySetCount(1);
        latch.await();
        log.info("are you OK?");
    }

    @Test
    public void SemaphoreExample() throws Exception{
        RSemaphore lock = redisson.getSemaphore("Semaphore1");
        //lock.trySetPermits(10);//这里就是先放10个信号
        for (int i = 0; i < 10; i++) {
            double random = Math.random();
            System.out.println(random);

            //如果随机数大于0.5则获取，否则释放
            if (random > 0.5) {
                boolean b = lock.tryAcquire();//尝试获取，如果没有，就不获取
                if (b) {
                    log.info("acquire...");
                } else {
                    log.info("未获取到...");
                }

            } else {
                lock.release();
                log.info("release...");
            }
        }
    }
}
```

```java
import org.redisson.Redisson;
import org.redisson.api.RMap;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyRedissonConfig {
    /**
     * 所有对Redisson的使用都是通过RedissonClient
     */
    @Bean(destroyMethod="shutdown")
    public RedissonClient redisson(){
        //1、创建配置
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");

        //2、根据Config创建出RedissonClient实例
        RedissonClient redisson = Redisson.create(config);

        return redisson;
    }
}
```

```java
import com.msb.caffeine.service.RedisMessageListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.ChannelTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(); // 需要设置主机名，端口，密码等参数
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        // 使用Jackson2JsonRedisSerializer来序列化和反序列化对象
        GenericJackson2JsonRedisSerializer jackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        // 设置键序列化器
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        // 设置值序列化器
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }

    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory factory, RedisMessageListener listener) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(factory);
        container.addMessageListener(listener, new ChannelTopic("cacheUpdateChannel"));
        return container;
    }
}
```
