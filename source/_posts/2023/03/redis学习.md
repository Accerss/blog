---
title: redis学习
date: 2023-03-21 20:29:12
tags: [java,Redis]
categories: [笔记]
---
# Redis简介
## 什么是Redis
Redis是一款开源的内存数据库，也称为键值存储(database)、缓存(database)和消息队列(database)系统。它提供了丰富的数据结构和高效的操作方式，并且支持多种编程语言的客户端，如Java、Python、C++等，让开发人员可以轻松地使用Redis来构建高性能、可扩展的应用程序。Redis支持数据持久化、主从复制、Lua脚本等特性，并且广泛应用于Web应用、实时计算、日志分析、消息发布等场景。
## 什么是Jedis
Jedis是Redis官方推荐的Java连接Redis的客户端库，提供了丰富的API和易于使用的接口，使得Java开发人员能够轻松地与Redis进行交互。Jedis支持连接池、集群、管道和事务等特性，在高并发，高吞吐量的场景下，提供了性能优秀和稳定可靠的解决方案。Jedis在Redis社区得到了广泛的应用和支持，是Java开发人员使用Redis的首选客户端库之一。
以下是一个简单的使用Jedis连接Redis服务器，并执行一些Redis操作的例子：
```java
import redis.clients.jedis.Jedis;

public class Example {

   public static void main(String[] args) {

      // 创建一个Jedis对象，用于与Redis服务器交互
      Jedis jedis = new Jedis("localhost", 6379);

      // 执行一些Redis命令

      // 存储一个字符串值
      jedis.set("key1", "value1");

      // 获取一个字符串值
      String value = jedis.get("key1");
      System.out.println(value);

      // 存储一个哈希映射
      jedis.hset("hash1", "field1", "value1");

      // 获取一个哈希映射的所有字段和值
      System.out.println(jedis.hgetAll("hash1"));

      // 关闭Jedis对象
      jedis.close();
   }
}
```
## 什么是Spring Data Redis
Spring Data Redis是Spring Data项目家族中的一个子项目，它通过简化和抽象操作，简化了在Redis中存储和访问数据的过程。它提供了丰富的API和基于注解的配置，使得Java开发人员能够更加便捷地与Redis交互，而无需过多地关注低级别的Redis操作。Spring Data Redis支持散列、列表、集合、有序集合等丰富的数据结构，同时也支持事务操作、消息发布订阅、Lua脚本执行、持久化等特性。通过Spring Data Redis，Java开发人员能够快速地搭建和运行分布式和高并发的应用程序。
### spring-data-redis针对jedis提供了如下功能：
1. 连接池自动管理，提供了一个高度封装的“RedisTemplate”类，提供了许多方法，包括以下几种类型的方法：
- 对字符串的操作方法：如opsForValue().set、opsForValue().get等方法。
- 对哈希的操作方法：如opsForHash().put、opsForHash().get等方法。
- 对列表的操作方法：如opsForList().leftPush、opsForList().rightPop等方法。
- 对集合的操作方法：如opsForSet().add、opsForSet().members等方法。
- 对有序集合的操作方法：如opsForZSet().add、opsForZSet().range等方法。
- 事务操作方法：如multi、watch、exec等方法。
- 管道操作方法：如executePipelined()方法。
- 异步操作方法：如opsForValue().setAsync、opsForHash().getAsync等方法。
- 发布订阅操作方法：如convertAndSend()方法、subscribe()方法等。

除此之外，RedisTemplate还提供了对键、连接、序列化等方面的控制和操作的方法，如delete()、getConnectionFactory()、setKeySerializer()等方法。总之，RedisTemplate提供了丰富的方法，使得Java开发者可以轻松地使用Redis。

3. 针对jedis客户端中大量api进行了归类封装,将同一类型操作封装为operation接口：
- ValueOperations：简单K-V操作
- SetOperations：set类型数据操作
- ZSetOperations：zset类型数据操作
- HashOperations：针对map类型的数据操作
- ListOperations：针对list类型的数据操作

4. 提供了对key的“bound”(绑定)便捷化操作API，可以通过bound封装指定的key，然后进行一系列的操作而无须“显式”的再次指定Key，即:
- BoundKeyOperations：
- BoundValueOperations
- BoundSetOperations
- BoundListOperations
- BoundSetOperations、
- BoundHashOperations
5. 将事务操作封装，有容器控制。
6. 针对数据的“序列化/反序列化”，提供了多种可选择策略(RedisSerializer) 
	**JdkSerializationRedisSerializer**：POJO对象的存取场景，使用JDK本身序列化机制，将pojo类通过ObjectInputStream/ObjectOutputStream进行序列化操作，最终redis-server中将存储字节序列。是目前最常用的序列化策略。
	**StringRedisSerializer**：Key或者value为字符串的场景，根据指定的charset对数据的字节序列编码成string，是“new String(bytes, charset)”和“string.getBytes(charset)”的直接封装。是最轻量级和高效的策略。
	**JacksonJsonRedisSerializer**：jackson-json工具提供了javabean与json之间的转换能力，可以将pojo实例序列化成json格式存储在redis中，也可以将json格式的数据转换成pojo实例。因为jackson工具在序列化和反序列化时，需要明确指定Class类型，因此此策略封装起来稍微复杂。【需要jackson-mapper-asl工具支持】
# RedisTemplate中API使用
## 首要配置
**引入pom.xml依赖**
```xml
<!--Redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
**添加配置文件**
```java
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1ms
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=5000ms
```
## RedisTemplate的用法
+ 首先使用@Autowired注入RedisTemplate（后面直接使用，就不特殊说明）
```java
@Autowired
private RedisTemplate redisTemplate;
```
### KEY操作
```java
//    删除key
redisTemplate.delete(key);
//    删除多个key
redisTemplate.delete(keys);
//    指定key的失效时间
redisTemplate.expire(key,time,TimeUnit.MINUTES);
//    根据key获取过期时间
Long expire = redisTemplate.getExpire(key);
//    判断key是否存在
redisTemplate.hasKey(key);
```
### String类型相关操作
```java
/**	添加缓存	**/
//1、通过redisTemplate设置值
redisTemplate.boundValueOps("StringKey").set("StringValue");
redisTemplate.boundValueOps("StringKey").set("StringValue",1, TimeUnit.MINUTES);
//2、通过BoundValueOperations设置值
BoundValueOperations stringKey = redisTemplate.boundValueOps("StringKey");
stringKey.set("StringVaule");
stringKey.set("StringValue",1, TimeUnit.MINUTES);
//3、通过ValueOperations设置值
ValueOperations ops = redisTemplate.opsForValue();
ops.set("StringKey", "StringVaule");
ops.set("StringValue","StringVaule",1, TimeUnit.MINUTES);

/**	设置过期时间	**/
redisTemplate.boundValueOps("StringKey").expire(1,TimeUnit.MINUTES);
redisTemplate.expire("StringKey",1,TimeUnit.MINUTES);

/**	获取缓存值	**/
//1、通过redisTemplate设置值
String str1 = (String) redisTemplate.boundValueOps("StringKey").get();
//2、通过BoundValueOperations获取值
BoundValueOperations stringKey = redisTemplate.boundValueOps("StringKey");
String str2 = (String) stringKey.get();
//3、通过ValueOperations获取值
ValueOperations ops = redisTemplate.opsForValue();
String str3 = (String) ops.get("StringKey");

/**	删除key	**/
Boolean result = redisTemplate.delete("StringKey");
/**	顺序递增	**/
redisTemplate.boundValueOps("StringKey").increment(3L);
/**	顺序递减	**/
redisTemplate.boundValueOps("StringKey").increment(-3L);
```
### Hash类型相关操作
```java
/**	添加缓存	**/
//1、通过redisTemplate设置值
redisTemplate.boundHashOps("HashKey").put("SmallKey", "HashVaue");
//2、通过BoundValueOperations设置值
BoundHashOperations hashKey = redisTemplate.boundHashOps("HashKey");
hashKey.put("SmallKey", "HashVaue");
//3、通过HashOperations设置值
HashOperations hashOps = redisTemplate.opsForHash();
hashOps.put("HashKey", "SmallKey", "HashVaue");
/**	设置过期时间	**/
redisTemplate.boundValueOps("HashKey").expire(1,TimeUnit.MINUTES);
redisTemplate.expire("HashKey",1,TimeUnit.MINUTES);
/**	添加一个Map集合	**/
HashMap<String, String> hashMap = new HashMap<>();
redisTemplate.boundHashOps("HashKey").putAll(hashMap );
/**	提取所有的field	**/
//1、通过redisTemplate获取值
Set keys1 = redisTemplate.boundHashOps("HashKey").keys();
//2、通过BoundValueOperations获取值
BoundHashOperations hashKey = redisTemplate.boundHashOps("HashKey");
Set keys2 = hashKey.keys();
//3、通过ValueOperations获取值
HashOperations hashOps = redisTemplate.opsForHash();
Set keys3 = hashOps.keys("HashKey");
/**	提取所有的value值	**/
//1、通过redisTemplate获取值
List values1 = redisTemplate.boundHashOps("HashKey").values();
//2、通过BoundValueOperations获取值
BoundHashOperations hashKey = redisTemplate.boundHashOps("HashKey");
List values2 = hashKey.values();
//3、通过ValueOperations获取值
HashOperations hashOps = redisTemplate.opsForHash();
List values3 = hashOps.values("HashKey");
/**	根据key提取value值	**/
//1、通过redisTemplate获取
String value1 = (String) redisTemplate.boundHashOps("HashKey").get("SmallKey");
//2、通过BoundValueOperations获取值
BoundHashOperations hashKey = redisTemplate.boundHashOps("HashKey");
String value2 = (String) hashKey.get("SmallKey");
//3、通过ValueOperations获取值
HashOperations hashOps = redisTemplate.opsForHash();
String value3 = (String) hashOps.get("HashKey", "SmallKey");
/**	获取所有的键值对集合	**/
//1、通过redisTemplate获取
Map entries = redisTemplate.boundHashOps("HashKey").entries();
//2、通过BoundValueOperations获取值
BoundHashOperations hashKey = redisTemplate.boundHashOps("HashKey");
Map entries1 = hashKey.entries();
//3、通过ValueOperations获取值
HashOperations hashOps = redisTemplate.opsForHash();
Map entries2 = hashOps.entries("HashKey");
/**	删除	**/
//删除小key
redisTemplate.boundHashOps("HashKey").delete("SmallKey");
//删除大key
redisTemplate.delete("HashKey");
/**	判断Hash中是否含有该值	**/
Boolean isEmpty = redisTemplate.boundHashOps("HashKey").hasKey("SmallKey");
```
### Set类型相关操作
```java
/**	添加Set缓存	**/
//1、通过redisTemplate设置值
redisTemplate.boundSetOps("setKey").add("setValue1", "setValue2", "setValue3");
//2、通过BoundValueOperations设置值
BoundSetOperations setKey = redisTemplate.boundSetOps("setKey");
setKey.add("setValue1", "setValue2", "setValue3");
//3、通过ValueOperations设置值
SetOperations setOps = redisTemplate.opsForSet();
setOps.add("setKey", "SetValue1", "setValue2", "setValue3");
/**	设置过期时间	**/
redisTemplate.boundValueOps("setKey").expire(1,TimeUnit.MINUTES);
redisTemplate.expire("setKey",1,TimeUnit.MINUTES);
/**	根据key获取Set中的所有值	**/
//1、通过redisTemplate获取值
Set set1 = redisTemplate.boundSetOps("setKey").members();
//2、通过BoundValueOperations获取值
BoundSetOperations setKey = redisTemplate.boundSetOps("setKey");
Set set2 = setKey.members();
//3、通过ValueOperations获取值
SetOperations setOps = redisTemplate.opsForSet();
Set set3 = setOps.members("setKey");
/**	根据value从一个set中查询,是否存在	**/
//1、通过redisTemplate获取值
Boolean isEmpty = redisTemplate.boundSetOps("setKey").isMember("setValue2");
/**	获取Set缓存的长度	**/
Long size = redisTemplate.boundSetOps("setKey").size();
/**	移除指定的元素	**/
Long result1 = redisTemplate.boundSetOps("setKey").remove("setValue1");
/**	移除指定的key	**/
Boolean result2 = redisTemplate.delete("setKey");
###	LIST类型相关操作
/**	添加缓存	**/
//1、通过redisTemplate设置值
redisTemplate.boundListOps("listKey").leftPush("listLeftValue1");
redisTemplate.boundListOps("listKey").rightPush("listRightValue2");
//2、通过BoundValueOperations设置值
BoundListOperations listKey = redisTemplate.boundListOps("listKey");
listKey.leftPush("listLeftValue3");
listKey.rightPush("listRightValue4");
//3、通过ValueOperations设置值
ListOperations opsList = redisTemplate.opsForList();
opsList.leftPush("listKey", "listLeftValue5");
opsList.rightPush("listKey", "listRightValue6");
/**	将List放入缓存	**/
ArrayList<String> list = new ArrayList<>();
redisTemplate.boundListOps("listKey").rightPushAll(list);
redisTemplate.boundListOps("listKey").leftPushAll(list);
/**	设置过期时间(单独设置)	**/
redisTemplate.boundValueOps("listKey").expire(1,TimeUnit.MINUTES);
redisTemplate.expire("listKey",1,TimeUnit.MINUTES);
/**	获取List缓存全部内容（起始索引，结束索引）	**/
List listKey1 = redisTemplate.boundListOps("listKey").range(0, 10); 
/**	从左或从右弹出一个元素	**/
String listKey2 = (String) redisTemplate.boundListOps("listKey").leftPop();  //从左侧弹出一个元素
String listKey3 = (String) redisTemplate.boundListOps("listKey").rightPop(); //从右侧弹出一个元素
/**	根据索引查询元素	**/
String listKey4 = (String) redisTemplate.boundListOps("listKey").index(1);
/**	获取List缓存的长度	**/
Long size = redisTemplate.boundListOps("listKey").size();
/**	根据索引修改List中的某条数据(key，索引，值)	**/
redisTemplate.boundListOps("listKey").set(3L,"listLeftValue3");
/**	移除N个值为value(key,移除个数，值)	**/
redisTemplate.boundListOps("listKey").remove(3L,"value");
```
### Zset类型的相关操作
```java
/**	向集合中插入元素，并设置分数	**/
//1、通过redisTemplate设置值
redisTemplate.boundZSetOps("zSetKey").add("zSetVaule", 100D);
//2、通过BoundValueOperations设置值
BoundZSetOperations zSetKey = redisTemplate.boundZSetOps("zSetKey");
zSetKey.add("zSetVaule", 100D);
//3、通过ValueOperations设置值
ZSetOperations zSetOps = redisTemplate.opsForZSet();
zSetOps.add("zSetKey", "zSetVaule", 100D);
/**	向集合中插入多个元素,并设置分数	**/
DefaultTypedTuple<String> p1 = new DefaultTypedTuple<>("zSetVaule1", 2.1D);
DefaultTypedTuple<String> p2 = new DefaultTypedTuple<>("zSetVaule2", 3.3D);
redisTemplate.boundZSetOps("zSetKey").add(new HashSet<>(Arrays.asList(p1,p2)));
/**	按照排名先后(从小到大)打印指定区间内的元素, -1为打印全部	**/
Set<String> range = redisTemplate.boundZSetOps("zSetKey").range(0, -1);
/**	获得指定元素的分数	**/
Double score = redisTemplate.boundZSetOps("zSetKey").score("zSetVaule");
/**	返回集合内的成员个数	**/
Long size = redisTemplate.boundZSetOps("zSetKey").size();
/**	返回集合内指定分数范围的成员个数（Double类型）	**/
Long COUNT = redisTemplate.boundZSetOps("zSetKey").count(0D, 2.2D);
/**	返回集合内元素在指定分数范围内的排名（从小到大）	**/
Set byScore = redisTemplate.boundZSetOps("zSetKey").rangeByScore(0D, 2.2D);
/**	带偏移量和个数，(key，起始分数，最大分数，偏移量，个数)	**/
Set<String> ranking2 = redisTemplate.opsForZSet().rangeByScore("zSetKey", 0D, 2.2D 1, 3);
/**	返回集合内元素的排名，以及分数（从小到大）	**/
Set<TypedTuple<String>> tuples = redisTemplate.boundZSetOps("zSetKey").rangeWithScores(0L, 3L);
  for (TypedTuple<String> tuple : tuples) {
      System.out.println(tuple.getValue() + " : " + tuple.getScore());
  }ss
/**	返回指定成员的排名	**/
//从小到大
Long startRank = redisTemplate.boundZSetOps("zSetKey").rank("zSetVaule");
//从大到小
Long endRank = redisTemplate.boundZSetOps("zSetKey").reverseRank("zSetVaule");
/**	从集合中删除指定元素	**/
redisTemplate.boundZSetOps("zSetKey").remove("zSetVaule");
/**	删除指定索引范围的元素（Long类型）	**/
redisTemplate.boundZSetOps("zSetKey").removeRange(0L,3L);
/**	删除指定分数范围内的元素（Double类型）	**/
redisTemplate.boundZSetOps("zSetKey").removeRangeByScorssse(0D,2.2D);
/**	为指定元素加分（Double类型）	**/
Double score = redisTemplate.boundZSetOps("zSetKey").incrementScore("zSetVaule",1.1D);
```

