---
layout:     post
title:      " MyBatis延迟加载 "
subtitle:   " MyBatis延迟加载 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - mybatis
---

#### mybatis 延迟加载
##### 什么是延迟加载
延迟加载又叫懒加载，也叫按需加载，也就是说先加载主信息，需要的时候，再去加载从信息。代码中有查询语句，当执行到查询语句时，并不是马上去DB中查询，而是根据设置的延迟策略将查询向后推迟。
##### 什么时候会执行延迟加载
配置之后在对关联对象进行查询时使用延迟加载。
##### 延迟加载策略
###### 直接加载
遇到代码中查询语句，马上到DB中执行select语句进行查询。（这种只能用于多表单独查询）
###### 侵入式延迟加载
将关联对象的详情（具体数据，如id、name）侵入到主加载对象，作为主加载对象的详情的一部分出现。当要访问主加载对象的详情时才会查询主表，但由于关联对象详情作为主加载对象的详情一部分出现，所以这个查询不仅会查询主表，还会查询关联表。
###### 深度延迟加载
将关联对象的详情（具体数据，如id、name）侵入到主加载对象，作为主加载对象的详情的一部分出现。当要访问主加载对象的详情时才会查询主表，但由于关联对象详情作为主加载对象的详情一部分出现，所以这个查询不仅会查询主表，还会查询关联表。
##### 使用延迟加载的目的
减轻DB服务器的压力,因为我们延迟加载只有在用到需要的数据才会执行查询操作。
##### 配置

```java
    <settings>
        <setting name ="aggressiveLazyLoading" value="false"/>
        <!--开启延迟加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
```
我们用关联查询来实现延迟加载，假设我们现在要查出用户和用户角色。

首先我们在user中添加查询userVo的方法和xml。

```xml
<!--userMapper.xml-->

....
<resultMap id="BaseResultMap" type="com.redstar.basemapper.pojo.User">
        <id column="id" jdbcType="VARCHAR" property="id"/>
        <result column="name" jdbcType="VARCHAR" property="name"/>
        <result column="age" jdbcType="INTEGER" property="age"/>
        <result column="role_id" jdbcType="INTEGER" property="roleId"/>
    </resultMap>
    <resultMap id="userRoleMapSelect" type="com.redstar.basemapper.pojo.UserVo">
        <association property="user" resultMap="BaseResultMap"/>
        <association property="role" fetchType="lazy" column="{id=role_id}"
                     select="com.redstar.basemapper.dao.RoleMapper.getRoleById"/>
    </resultMap>
    <sql id="Base_Column_List">
    id, `name`, age, role_id
  </sql>
    <select id="getUserVo" resultMap="userRoleMapSelect">
      select * from user where id=#{userId}
    </select>
...
    
    
    
    <!--roleMapper.xml-->
...    
    <resultMap id="BaseResultMap" type="com.redstar.basemapper.pojo.Role">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="role_name" jdbcType="VARCHAR" property="roleName" />
  </resultMap>
  <sql id="Base_Column_List">
    id, role_name
  </sql>
  <select id="getRoleById" resultMap="BaseResultMap">
    select * from role where id=#{id}
  </select>
...  
```
测试用例

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class BaseMapperApplicationTests {
    @Autowired
    private UserMapper userMapper;

    @Autowired
    private RoleMapper roleMapper;

    @Test
    public void getUserVo() {
        System.out.println(userMapper.getUserVo("12312232"));
//        System.out.println(userMapper.getUserById("12312232"));
//        System.out.println(roleMapper.getRoleById(1));

    }


}
```
输出结果：

```
UserVo{user=User [Hash = 1937575946, id=12312232, name=哇哈哈啊娃哈哈哇哈哈哈哈哈哈哈, age=48, roleId=1, serialVersionUID=1], role=Role [Hash = 317053574, id=1, roleName=admin, serialVersionUID=1]}
```
##### 注意
> 许多对延迟加载原理不太熟悉的朋友会经常遇到一些莫名其妙的问题：有些时候延迟加载
> 可以得到数据，有些时候延迟加载就会报错，为什么会出现这种情况呢？
> MyBatis 延迟加载是通过动态代理实现的，当调用配直为延迟加载的属性方法时， 动态代
> 理的操作会被触发，这些额外的操作就是通过 MyBatis 的 SqlSessio口去执行嵌套 SQL 的 。
> 由于在和某些框架集成时， SqlSession 的生命周期交给了框架来管理，因此当对象超出
> SqlSession 生命周期调用时，会由于链接关闭等问题而抛出异常 。 在和 Spring 集成时，要
> 确保只能在 Service 层调用延迟加载的属性 。 当结果从 Service 层返回至 Controller 层时， 如果
> 获取延迟加载的属性值，会因为 SqlSessio口已经关闭而抛出异常 。