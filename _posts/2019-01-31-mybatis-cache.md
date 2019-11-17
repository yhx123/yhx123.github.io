---
layout:     post
title:      " MyBatis缓存 "
subtitle:   " MyBatis缓存 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - mybatis
---
#### MyBatis缓存介绍
　　正如大多数持久层框架一样，MyBatis 同样提供了一级缓存和二级缓存的支持
　　
1. 一级缓存: 基于PerpetualCache 的 HashMap本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该Session中的所有 Cache 就将清空。
2. 二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap存储，不同在于其存储作用域为          Mapper(Namespace)，并且可自定义存储源，如 Ehcache。
3. 对于缓存数据更新机制，当某一个作用域(一级缓存Session/二级缓存Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被clear。


##### mybatis的相关概念
1. SqlSession : 代表和数据库的一次会话，向用户提供了操作数据库的方法。
2. MappedStatement: 代表要发往数据库执行的指令，可以理解为是Sql的抽象表示。
3. Executor: 具体用来和数据库交互的执行器，接受MappedStatement作为参数。
4. 映射接口: 在接口中会要执行的Sql用一个方法来表示，具体的Sql写在映射文件中。
5. 映射文件: 可以理解为是Mybatis编写Sql的地方，通常来说每一张单表都会对应着一个映射文件，在该文件中会定义Sql语句入参和出参的形式。


##### 一级缓存
- MyBatis的一级查询缓存（也叫作本地缓存）是基于org.apache.ibatis.cache.impl.PerpetualCache 类的 HashMap本地缓存，其作用域是SqlSession
- 在同一个SqlSession中两次执行相同的 sql 查询语句，第一次执行完毕后，会将查询结果写入到缓存中，第二次会从缓存中直接获取数据，而不再到数据库中进行查询，这样就减少了数据库的访问，从而提高查询效率。
- 当一个 SqlSession 结束后，该 SqlSession 中的一级查询缓存也就不存在了。 
- myBatis 默认一级查询缓存是开启状态，且不能关闭。
- 增删改会清空缓存，无论是否commit
当SqlSession关闭和提交时，会清空一级缓存

 > 可能你会有疑惑，我的mybatis bean是由spring 来管理的，已经屏蔽了sqlSession这个东西了？那怎么的一次操作才算是一次sqlSession呢？
 
- spring整合mybatis后，非事务环境下，每次操作数据库都使用新的sqlSession对象。因此mybatis的一级缓存无法使用（一级缓存针对同一个sqlsession有效）

- 在开启事物的情况之下，spring使用threadLocal获取当前资源绑定同一个sqlSession，因此此时一级缓存是有效的
在开启以及缓存的时候查询得到的对象是同一个对象。
这种情况下会出现一个问题。我们先看一下代码。

```java
public void listMybatisModel() {
        List<MybatisModel> mybatisModels = mapper.listMybatisModel();
        List<MybatisModel> mybatisModelsOther = mapper.listMybatisModel();
        System.out.println(mybatisModels == mybatisModelsOther);
        System.out.println("list count: " + mybatisModels.size());
    }
```

```java
System.out.println(mybatisModels == mybatisModelsOther);
```
输出结果竟然是true，这样说来是同一个对象。 会出现这种场景，第一次查出来的对象然后修改了，第二次查出来的就是修改后的对象。

##### 一级缓存实现
对SqlSession的操作mybatis内部都是通过Executor来执行的。Executor的生命周期和SqlSession是一致的。Mybatis在Executor中创建了一级缓存，基于PerpetualCache 类的 HashMap

##### 二级缓存
- MyBatis的二级缓存是mapper范围级别的
- SqlSession关闭后才会将数据写到二级缓存区域
- 增删改操作，无论是否进行提交commit()，均会清空一级、二级缓存
- 二级缓存是默认开启的。（想开启就不必做任何配置）
- 二级缓存会使用 Least Recently Used (LRU，最近最少使用的）算法来收回。
- 根据时间表（如 no Flush Interval ，没有刷新间隔），缓存不会以任何时间顺序来刷新 。
- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的 1024 个引用。
- 缓存会被视为 read/write （可读／可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改 。
- 

用下面这张图描述一级缓存和二级缓存的关系。

![](https://user-gold-cdn.xitu.io/2019/1/17/1685beead7220c78?w=607&h=275&f=png&s=90085)


###### 配置二级缓存
1. 在保证二级缓存的全局配置开启的情况下，给某个xml开启二级缓存只需要在xml中添加<cache />即可
```xml
// mybatis-config.xml 中配置
<settings>
    默认值为 true。即二级缓存默认是开启的
    <setting name="cacheEnabled" value="true"/>
</settings>

// 具体mapper.xml 中配置
<mapper namespace="cn.itcast.mybatis.mapper.UserMapper">
	<!-- 开启本mapper的namespace下的二级缓存
	type：指定cache接口的实现类的类型，mybatis默认使用PerpetualCache
	要和ehcache整合，需要配置type为ehcache实现cache接口的类型-->

	<cache />

</mapper>
```
如果想要设置增删改操作的时候不清空二级缓存的话，可以在其insert或delete或update中添加属性flushCache=”false”，默认为 true。

```xml
<delete id="deleteStudent" flushCache="false">
    DELETE FROM user where id=#{id}
</delete>
```
通过以下一些配置可以修改一些缓存参数。

```xml
<cache
    eviction＝ "FIFO"
    flushlnterval="600000"
    size="512"
    readOnly="true" />
```
配置创建了一个FIFO缓存，并每隔 60 秒刷新一次，存储集合或对象的512个引用， 而且返回的对象被认为是只读的,因此在不同线程中的调用者之间修改它们会导致冲突。cache可以配置的属性如下。
eviction （收回策略）
> LRU （最近最少使用的:  移除最长时间不被使用的对象，这是默认值 。
 FIFO （先进先出〉 ： 按对象进入缓存的顺序来移除它们 。
 SOFT （软引用） ： 移除基于垃圾回收器状态和软引用规则的对象 。
 WEAK （弱引用） ： 更积极地移除基于垃圾收集器状态和弱引用规则的对象 
 
- flushinterval（刷新间隔)。
可以被设置为任意的正整数，而且它们代表一个合理
的毫秒形式的时间段。默认情况不设置，即没有刷新间隔，缓存仅仅在调用语句时刷新。
- size （引用数目）。 可以被设置为任意正整数，要记住缓存的对象数目和运行环境的可用内存资源数目。默认值是 1024 。
- readOnly （只读）。属性可以被设置为 true 或 false 。只读的缓存会给所有调用者
返回缓存对象的相同实例，因此这些对象不能被修改，这提供了很重要的性能优势。可读写的缓存会通过序列化返回缓存对象的拷贝，这种方式会慢一些，但是安全，因此默认是false。

2. 当只使用注解方式配置二级缓存时，如果在Mapper接口中，则需要增加如下配置 。
```java
@CacheNamespace (
eviction = FifoCache.class ,
flushinterval = 60000 ,
size = 512 ,
readWrite = true)
public interface Mapper {
    
}
```
括号内的内容是配置缓存属性。


3. Mapper 接口和对应的 XML 文件是相同的命名空间，想使用二级缓存，两者必须同时配置（如果接口不存在使用注解方式的方法，可以只在 XML 中配置〉，因此按照上面的方
式进行配置就会出错 ， 这个时候应该使用参照缓存。在 Mapper 接口中，参照缓存配置如下 。
```java
@CacheNarnespaceRef(RoleMapper.class)
public interface RoleMapper {
```
因为想让 RoleMapper 接口中的注解方法和 XML中的方法使用相同的缓存，因此使用参照缓存配置RoleMapper.class，这样就会使用命名空间为xx.xxx.xxx.xxx.RoleMapper的缓存配置，即RoleMapper.xml 中配置的缓存 。
Mapper 接口可以通过注解引用XML 映射文件或者其他接口的缓存，在 XML 中也可以配置参照缓存，如可以在 RoleMapper.xml 中进行如下修改 。
```
<cache-ref narnespace="xxx.xxx.xxx.xxx.RoleMapper"/>
```
这样配置后XML 就会引用 Mapper 接口中配置的二级缓存，同样可以避免同时配置二级缓存导致的冲突。MyBatis 中很少会同时使用 Mapper 接口注解方式和XML映射文件，所以参照缓存并不是为了解决这个问题而设计的。参照缓存除了能够通过引用其他缓存减少配置外，主要的作用是解决脏读。

MyBatis使用SerializedCache(org.apache.ibaits.cache.decorators.SerializedCache)序列化缓存来实现可读写缓存类，井通过序列化和反序列化来保证通过缓存获取数据时，得到的是一个新的实例。因此，如果配置为只读缓存，MyBatis就会使用Map来存储缓存值，这种情况下，从缓存中获取的对象就是同一个实例。因为使用可读写缓存，可以使用SerializedCache序列化缓存。这个缓存类要求所有被序列化的对象必须实现 Serializable (java.io.Serializable）接口
虽然使用序列化得到的对象都是不一样的对象修改时都是互不影响，但是还是不安全的。

###### 脏读的产生
Mybatis的二级缓存是和命名空间绑定的，所以通常情况下每一个Mapper映射文件都有自己的二级缓存，不同的mapper的二级缓存互不影响。
- 引起脏读的操作通常发生在多表关联操作中，比如在两个不同的mapper中都涉及到同一个表的增删改查操作，当其中一个mapper对这张表进行查询操作，此时另一个mapper进行了更新操作刷新缓存，然后第一个mapper又查询了一次，那么这次查询出的数据是脏数据。出现脏读的原因是他们的操作的缓存并不是同一个。


###### 脏读的避免
- mapper中的操作以单表操作为主，避免在关联操作中使用mapper
- 使用参照缓存 

##### 集成EhCache缓存

缓存数据有内存和磁盘两级，无须担心容量问题。
- 缓存数据会在虚拟机重启的过程中写入磁盘。可以通过RMI、可插入API等方式进行分布式缓存。
- 具有缓存和缓存管理器的侦昕接口。
- 支持多缓存管理器实例以及一个实例的多个缓存区域。
###### 1. 添加项目依赖

```xml
    <dependency>
	    <groupId>org.mybatis.caches</groupId>
	    <artifactId>mybatis-ehcache</artifactId>
	    <version>1.0.3</version>
	</dependency>
```
###### 2. 配置 EhCache
在 src/main/resources 目录下新增 ehcache.xml 文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="ehcache.xsd"
    updateCheck="false" monitoring="autodetect"
    dynamicConfig="true">
    
    <diskStore path="D:/cache" />
            
	<defaultCache      
		maxElementsInMemory="3000"      
		eternal="false"      
		copyOnRead="true"
		copyOnWrite="true"
		timeToIdleSeconds="3600"      
		timeToLiveSeconds="3600"      
		overflowToDisk="true"      
		diskPersistent="true"/> 
</ehcache>
```
有关EhCache的详细配置可以参考地址 http://www.ehcache.org/ehcache.xml 中的内容。
- copyOnRead 的含义是，判断从缓存中读取数据时是返回对象的引用还是复制一个对象返回。默认情况下是false，即返回数据的引用，这种情况下返回的都是相同的对象，和MyBatis默认缓存中的只读对象是相同的。如果设置为 true ，那就是可读写缓存，每次读取缓存时都会复制一个新的实例 。
- copyOnWrite 的含义是 ，判断写入缓存时是直接缓存对象的引用还是复制一个对象然后缓存，默认也是false。如果想使用可读写缓存，就需要将这两个属性配置为true，如果使用只读缓存，可以不配置这两个属性，使用默认值 false 即可 。

3. 修改Mapper.xml中的缓存配置
ehcache-cache 提供了如下 2 个可选的缓存实现。
- org.mybatis.caches.ehcache.EhcacheCache
- org.mybatis.caches.ehcache.LoggingEhcache  这个是带日志的缓存。
 
在xml中添加
```xml
 <cache type ＝"org.mybatis.caches.ehcache.EhcacheCache" />
```

只通过设置 type 属性就可 以使用 EhCache 缓存了，这时cache的其他属性都不会起到任何作用，针对缓存的配置都在ehcache.xml中进行。在ehcache.xml配置文件中，只有一个默认的缓存配置，所以配置使用EhCache缓存的Mapper映射文件都会有一个以映射文件命名空间命名的缓存。如果想针对某一个命名空间进行配置，需要在 ehcache.xml 中添加一个和映射文件命名空间一致的缓存配置，例如针对RoleMapper可以进行如下配置。

```xml
	<cache      
		name="tk.mybatis.simple.mapper.RoleMapper"
		maxElementsInMemory="3000"      
		eternal="false"      
		copyOnRead="true"
		copyOnWrite="true"
		timeToIdleSeconds="3600"      
		timeToLiveSeconds="3600"      
		overflowToDisk="true"      
		diskPersistent="true"/>
```
##### 集成Redis缓存
1. 添加依赖，目前只有bata版本。
```
        <dependency>
			<groupId>org.mybatis.caches</groupId>
			<artifactId>mybatis-redis</artifactId>
			<version>1.0.0-beta2</version>
		</dependency>
```
2. 配置Redis
使用 Redis 前，必须有一个 Redis 服务，有关Redis安装启动的相关内容，可参考如下地址中的官方文档：https://redis.io/topics/quickstart。Redis服务启动后，在src/main/resources 目录下新增 redis.properties 文件 。
```properties
host=localhost
port=6379
connectionTimeout=SOOO
soTimeout=SOOO
password=
database=O
clientName=
```
3. 修改mapper.xml中的配置。
```xml
<mapper namespace ＝ ” tk.mybat 工 s.s 工 mple.mapper.RoleMapper ” 〉
<cache type= "org.mybatis.caches.redis.RedisCache" />
〈 ！一其他自己直一 〉
</mapper>
```
配置依然很简单， RedisCache 在保存缓存数据和获取缓存数据时，使用了Java的序列化和反序列化，因此还需要保证被缓存的对象必须实现Serializable接口。改为RedisCache缓存配置后， testL2Cache 测试第一次执行时会全部成功，但是如果再次执行，就会出错。这是因为Redis作为缓存服务器，它缓存的数据和程序（或测试）的启动无关，Redis 的缓存并不会因为应用的关闭而失效。所以再次执行时没有进行一次数据库查询，所有查询都使用缓存，测试的第一部分代码中的rolel和role2都是直接从二级缓存中获取数据，因为是可读写缓存，所以不是相同的对象。当需要分布式部署应用时，如果使用MyBatis自带缓存或基础的EhCahca缓存，分布式应用会各自拥有自己的缓存，它们之间不会共享缓存 ，这种方式会消耗更多的服务器资源。如果使用类似 Redis 的缓存服务，就可以将分布式应用连接到同一个缓存服务器，实现分布式应用间的缓存共享 。