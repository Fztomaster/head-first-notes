##  redis
<!-- TOC -->

- [redis](#redis)
    - [数据类型](#数据类型)
    - [操作数据类型](#操作数据类型)
        - [string类型(重点)](#string类型重点)
        - [hash类型(map)](#hash类型map)
        - [list类型(LinkedList)](#list类型linkedlist)
        - [set类型](#set类型)
        - [通用key操作](#通用key操作)
    - [Jedis(重点)](#jedis重点)
        - [Jedis概述](#jedis概述)
        - [Jedis操作](#jedis操作)
            - [构造方法](#构造方法)
            - [常用方法](#常用方法)
            - [基本使用](#基本使用)
        - [Jedis连接池操作](#jedis连接池操作)
            - [构造方法](#构造方法-1)
            - [使用自定义配置的连接池](#使用自定义配置的连接池)
            - [基本使用](#基本使用-1)
        - [JedisUtils工具类](#jedisutils工具类)
            - [jedis.properties配置文件](#jedisproperties配置文件)
            - [JedisUtils工具类](#jedisutils工具类-1)

<!-- /TOC -->
### 数据类型

key-value形式保存输出，key为String，value有5种数据类型：
* String：字符串
* hash：哈希类型，类似于map
* list：列表
* set：无序集合
* zset：有序集合
### 操作数据类型
#### string类型(重点)
* 设置数据：set key value
* 获取数据：get key
* 删除数据：del key
* 查看所有：keys *
#### hash类型(map)
* 设置一个hash：hset key fied value
* 获取一个hash里某个field：hget key field
* 删除一个hash里某个field：hdel key field
* 查看一个hash里所有数据：hgetall key
#### list类型(LinkedList)
* 添加数据：lpush/rpush
	* lpush key value1 value2 ....
	* rpush key value1 value2 ...
* 查看全部数据：lrange key 0 -1
* 获取数据：lpop/rpop
	* lpop key
	* rpop key
#### set类型
* 添加成员：sadd key member1 member2 ...
* 获取所有成员：smembers key
* 随机获取一个成员：srandmember key
* 删除成员：srem key member1 member2 ...
* 计算交集：sinter key1 key2 ...
* 计算并集：sunion key1 key2 ...
* 计算差集：sdiff key1 key2 ...
#### 通用key操作
* 查询key：keys pattern，pattern中可以出现"*"和"?"
* 删除key：del key
* 判断key是否存在：exists key
* 查看key的类型：type key
### Jedis(重点)
#### Jedis概述
Java Redis = Jedis，java操作redis需要引入commons.pool2-2.3.jar和jedis-2.7.0.jar两个包
#### Jedis操作
##### 构造方法
```java
	Jedis(String ip, int port);
```
##### 常用方法
```java
	jedis.set(String key, String value);
	jedis.get(String key);
	jedis.del(String key);
	jedis.lpush(String key, String value);
	jedis.close();
```
##### 基本使用
```java
	// 获取连接
	Jedis jedis = new Jedis(String ip, int port);
	// 设置数据
	jedis.set("name", "jack");
	// 获取数据
	jedis.get("name");
	// 释放资源
	jedis.close();
```
#### Jedis连接池操作
连接池类：JedisPool
##### 构造方法
```java
	JedisPool(String host, int port);
```
##### 使用自定义配置的连接池
```java
	JedisConfigPool config = new JedisConfigPool(); // 连接池的配置信息对象
	config.setMaxTotal(int maxTotal); // 设置最大的连接数
	config.setMaxIdle(int maxIdle); // 设置空闲时最大的连接数
	JedisPool(JedisConfigPool config, String host, int port); // 创建连接池对象
	Jedis jedis = pool.getResource(); // 从连接池中获取连接
```
##### 基本使用
```java
	// 设置连接池的配置信息
	JedisConfigPool config = new JedisConfigPool();
	config.setMaxTotal(30);
	config.setMaxIdle(20);
	// 创建JedisPool连接池对象
	JedisPool pool = new JedisPool(config, "192.168.111.128", 6379);
	// 从连接池中获取连接
	Jedis jedis = pool.getResource();
	// 操作redis数据库
	jedis.set("username", "jack");
	jedis.set("password", "123456");
	String username = jedis.get("username");
	String password = jedis.get("password");
	// 释放资源
	jedis.close();
```
#### JedisUtils工具类
##### jedis.properties配置文件
```properties
	host=192.168.111.128
	port=6379
	maxTotal=30
	maxIdle=20
```
##### JedisUtils工具类
```java
	public class JedisUtils {
        private static JedisPool pool;
        static {
            // 加载配置文件
            ResourceBundle bundle = ResourceBundle.getBundle("jedis");
            // 获取配置信息
            String host = bundle.getString("host");
            int port = Integer.parseInt(bundle.getString("port"));
            int maxTotal = Integer.parseInt(bundle.getString("maxTotal"));
            int maxIdle = Integer.parseInt(bundle.getString("maxIdle"));
            // 创建JedisConfigPool对象
            JedisConfigPool config = new JedisConfigPool();
            config.setMaxTotal(maxTotal);
            config.setMaxIdle(maxIdle);
            // 创建连接池对象
            pool = new JedisPool(config, host, port);
        }
        
       // 获取jedis
        public static Jedis getJedis() {
            return pool.getResource();
        }
        // 释放资源
        public static void close(Jedis jedis) {
            if (jedis != null) {
                jedis.close();
            }
        }
	}
```
