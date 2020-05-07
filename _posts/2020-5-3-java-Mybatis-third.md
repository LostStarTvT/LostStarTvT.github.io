---

layout: post
title: Mybatis：连接池与事务分析
tags: java  
---


> 记录Mybatis的连接池与事务分析

##  目录
* 目录
{:toc}
# 1.连接池&事务控制

## 1.1连接池使用及分析

1.连接池基本概念：

1. 连接池就是用于存储连接的一个容器。

2. 容器就是一个集合对象，该集合必须是线程安全的，不能两个线程拿到同一个连接。

3. 该集合还必须实现队列的特性：先进先出。

2.mybatis中的连接池：

1. mybatis连接池提供了三种方式的配置。
   1. 配置的位置：主配置文件sqlMapConfig.xml中的DataSource标签，type属性就是表示采用何种连接池方式
2. type属性取值：
   1. POOLED 采用传统的javax.sql.DataSource规范中的连接池， mybatis中有针对规范的实现
   2. UNPOOLED 采用传统的获取连接的方式，虽然也实现Javax.sql.DataSource接口，但是并没有使用池的思想。
   3. JNDI 采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到的DataSource时不一样的。注意：如果不是web或者maven的war工程，是不能使用的。我们实际开发中使用的是tomcat服务器，采用的连接池也就是dbcp连接池。

对应的配置文件：

```xml
<!-- 配置数据源(连接池)-->
<dataSource type="POOLED">
    <!-- 配置连接数据的四个基本信息-->
    <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</dataSource>
```

## 1.2事务控制分析

什么是事务？

这些基础控制需要自己去查询，李姐万岁。

事务的四大特性ACID。

不考虑隔离性会产生的3个问题

解决方法：四种隔离级别

它是通过SQLSession对象的commit方法和rollback方法实现事务的提交和回滚。

一般都是自己去控制提交，即设置自动事务提交为false，因为有些操作比如转账是多个事务组合，需要用一个事务去控制，如果全都是自动提交的话，是控制不住的，不能回滚，所以开发中总是将自动事务提交关闭，设置为手动提交。

# 2.基于xml配置的动态SQL语句

在有些业务逻辑比较复杂的时候，其sql是动态变化的，所以需要动态sql语句。例如查询条件，有些条件可能有有些就没有，即模糊查询。

## 2.1 if 标签

if标签可以进行判断是否有该条件。  

测试案例，通过传递一个user对象，进行模糊查询，其中if表示的条件为，如果有username，则进行查询，否则就不进行查询，后面的sex也是类似。其中接口不表。  

```xml
<!--根据条件查询-->
<select id="findUserByCondition" resultType="user" parameterType="user">
    select  * from user where 1=1
    <if test="username != null">
        and username = #{ username }
    </if>
    <if test="sex != null" >
        and sex = #{ sex }
    </if>
</select>
```

测试查询：表示查询一个叫老王的用户并且性别为女。

```java
//测试查询所有
@Test
public void  testFindByCondition(){
    User user = new User();
    user.setUsername("老王");
    user.setSex("女");
    List<User> users = userDao.findUserByCondition(user);
    for (User user1:users){
        System.out.println(user1);
    }
}
```

## 2.2 where 标签

where标签的使用也就是让mybatis自动的进行拼接sql语句,下面可以看出，上面if标签中的`where 1=1`没有了。

```xml
<!--根据条件查询-->
<select id="findUserByCondition" resultType="user" parameterType="user">
    select  * from user
    <where>
        <if test="username != null">
            and username = #{ username }
        </if>
        <if test="sex != null" >
            and sex = #{ sex }
        </if>
    </where>
</select>
```

## 2.3 for each标签

对应的解决的SQL语句：`select * from user where id in (1,2,3)`

\<foreach >标签用于遍历集合，属性：

1. collection:代表要遍历的集合元素，注意编写时不要写#{}
2. open:代表语句的开始部分，
3. close:代表结束部分
4. item:代表遍历集合的每个元素，生成的变量名
5. sperator:代表分隔符

```xml
<!-- 根据queryVo中的id集合实现查询用户列表-->
<select id="findUserInIds" resultType="user" parameterType="QueryVo">
    select * from user
    <where>
        <if test="ids != null and ids.size()>0">
            <foreach collection="ids" open="and id in (" close=")" item="uid" separator=",">
                #{uid}
            </foreach>
        </if>
    </where>
</select>
```

其中#`{uid}`与`item="uid"`名称必须保持一致。  

对应的测试类：

```java
//接口
/**
 *  根据queryVo中提供的id，查询用户信息
 * @param vo
 * @return
*/
List<User> findUserInIds(QueryVo vo);

//测试foreach方法
@Test
public void  testFindInIds(){
    QueryVo queryVo = new QueryVo();
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    queryVo.setIds(list);
    List<User> users = userDao.findUserInIds(queryVo);

    for (User user1:users){
        System.out.println(user1);
    }
}
```

查询结果：会将id为1,2的查询出来。

```shell
User{id=1, username='测试', birthday=Wed Apr 22 00:00:00 CST 2020, sex='男', address='北京'}
User{id=2, username='小小', birthday=Wed Apr 22 00:00:00 CST 2020, sex='女', address='上海'}
```

## 2.4 sql标签

这个是抽取重复的sql语句，比如抽取`select * from user`这条语句。了解下就行。

```xml
<!--不要写分号-->
<sql id="defaultUser" >
    select * from user
</sql>

<!--查询所有-->
<select id="findAll" resultType="user">
    -- select * from user; 
    <!-- 直接插入 -->
    <include refid="defaultUser"></include>
</select>
```

# 3.多表操作

表之间的关系，一对多，多对一，一对一，多对多。这三个实现的思想都是一样的，主要就是sql语句的拼写，还是要加强学习一下。。外连接连接查询的高级操作。。  

**实例分析：用户和账户。**

1. 一个用户可以有多个账户
2. 一个账户只能属于一个用户

## 3.1一对一进行查询

需要实现的功能，将account表与user表通过id进行连接查询，实现查询出来每个账户的详细信息，即将account表与user表信息一一对应。  

实现的sql语句如下：

```sql
select u.*, a.id as aid, a.uid, a.money from account a,user u where u.id = a.uid;
```

[![one4one.png](https://pic.tyzhang.top/images/2020/05/04/one4one.png)](https://pic.tyzhang.top/image/dwnL)

以下是实现步骤。  

1.数据库建表:

```sql
# 创建用户表
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` datetime default NULL COMMENT '生日',
  `sex` char(1) default NULL COMMENT '性别',
  `address` varchar(256) default NULL COMMENT '地址',
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
# 导入用户数据
insert  into `user`(`id`,`username`,`birthday`,`sex`,`address`) values (41,'老王','2018-02-27 17:47:08','男','北京'),(42,'小二王','2018-03-02 15:09:37','女','北京金燕龙'),(43,'小二王','2018-03-04 11:34:34','女','北京金燕龙'),(45,'传智播客','2018-03-04 12:04:06','男','北京金燕龙'),(46,'老王','2018-03-07 17:37:26','男','北京'),(48,'小马宝莉','2018-03-08 11:44:00','女','北京修正');
# 创建账户表，外键为uid，关联用户表的id
DROP TABLE IF EXISTS `account`;
CREATE TABLE `account` (
  `id` int(11) NOT NULL COMMENT '编号',
  `uid` int(11) default NULL COMMENT '用户编号',
  `money` double default NULL COMMENT '金额',
  PRIMARY KEY  (`id`),
  KEY `FK_Reference_8` (`uid`),
  CONSTRAINT `FK_Reference_8` FOREIGN KEY (`uid`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
# 导入账户数据
insert  into `account`(`id`,`uid`,`money`) values (1,46,1000),(2,45,1000),(3,46,2000);study_test
```

2.建立account和user类。**因为主要是对account表操作，所有需要对account类进行改造**，

```java
public class Account implements Serializable {

    private Integer id;
    private Integer uid;
    private Double money;

    //从表实体应该包含一个主表实体的对象引用
    private User user;
	//get and set toSting
}
```

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
	//get and set toString 
}
```

3.对应的查询接口

```java
public interface IAccountDao {

    /**
     *  返回所有账户
     * @return
     */
    List<Account> findAll();
}
```

```java
public interface IUserDao {

    /**
     * 查询所有
     * @return
     */
    List<User> findAll();

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    User findById(Integer userId);
}
```

4.xml实现类    

通过配置实现如下查询结果：  

[![one4one.png](https://pic.tyzhang.top/images/2020/05/04/one4one.png)](https://pic.tyzhang.top/image/dwnL)

IAccountDao.xml,主要就是需要定义一个resultMap对应关系，其中对应关系就是需要显示的信息，其中account类中注入user类，然后第二个就是配置注入的对应关系，对应关系就是java对象中的属性与查询表结果名的一一对应。名称映射对则可以直接将数据注入。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IAccountDao">

    <!--定义封装account和user的resultMap  主要是因为account中注入一个user类-->
    <resultMap id="accountUserMap" type="account">
        <id property="id" column="aid"/>
        <result property="uid" column="uid"/>
        <result property="money" column="money"/>
        <!--一对一的关系映射，配置封装user的内容-->
        <association property="user" column="id" javaType="user">
            <id property="id" column="id"/>
            <result column="username" property="username"/>
            <result column="address" property="address"/>
            <result column="sex" property="sex"/>
            <result column="birthday" property="birthday"/>
        </association>
    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="accountUserMap">
        select u.*, a.id as aid, a.uid, a.money from account a,user u where u.id = a.uid;
    </select>
</mapper>
```
IUserDao.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IUserDao">

    <!--查询所有-->
    <select id="findAll" resultType="user">
        select * from user;
    </select>

    <!--根据id查询用户 -->
    <select id="findById" parameterType="Integer" resultType="com.diaowenjie.domain.User">
        select * from user where id  = #{id};
    </select>
</mapper>
```

测试类；

```java
@Test
public void  testFindAllAccount() throws IOException {
    //5.使用代理对象执行方法
    List<Account> accounts = accountDao.findAll();
    for (Account account:accounts){
        System.out.println(account);
        System.out.println(account.getUser());
    }
}
```

然后便能够输出结果：

```shell
Account{id=1, uid=46, money=1000.0}
User{id=46, username='老王', birthday=Wed Mar 07 17:37:26 CST 2018, sex='男', address='北京'}
Account{id=2, uid=45, money=1000.0}
User{id=45, username='传智播客', birthday=Sun Mar 04 12:04:06 CST 2018, sex='男', address='北京金燕龙'}
Account{id=3, uid=46, money=2000.0}
User{id=46, username='老王', birthday=Wed Mar 07 17:37:26 CST 2018, sex='男', address='北京'}
```

## 3.2 一对多查询

需求：查询所有用户信息以及用户关联的账户信息 **(一个用户可以有多个账户)**，即将用户与账户表关联，查询出每个用户拥有账号的属性，查询结果如下图所示。

分析：用户信息和他的账户信息为一对多的关系，并且查询过程中如果用户没有账户信息，此时也要将用户信息查询出来，我们想到的就是左外连接查询会比较合适。

需要执行的sql语句：

```sql
SELECT u.*, a.id AS aid, a.uid, a.money FROM user u LEFT OUTER JOIN account a ON u.id = a.uid;
```

[![one4Plus.png](https://pic.tyzhang.top/images/2020/05/04/one4Plus.png)](https://pic.tyzhang.top/image/dZvs)

因为是对user进行查询，所以user为主表，首先对user进行改造，因为用户可以有多个账户，所以用List存储。  

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    //一对多关系映射， 主表实体应该包含从表实体的集合引用
    private List<Account> accounts;
	//get and set to string 
}
```

更改user的findAll操作，实体类中是集合，所以此处标签也是用\<collection>  

```xml
<resultMap id="userAccountMap" type="user">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <result property="address" column="address"/>
    <result property="sex" column="sex"/>
    <result property="birthday" column="birthday"/>
    <!--配置user对象中的accounts集合的映射 其中column表示查询结果中的属性名称，property为实体类中的属性名称。需要一一对应-->
    <collection property="accounts" ofType="account">
        <id column="aid" property="id"/>
        <result column="uid" property="uid"/>
        <result column="money" property="money"/>
    </collection>
</resultMap>
<!--查询所有-->
<select id="findAll" resultMap="userAccountMap">
    SELECT u.*, a.id AS aid, a.uid, a.money FROM user u LEFT OUTER JOIN account a ON u.id = a.uid;
</select>
```

对应测试类：

```java
@Test
public void  testFindAllUser() throws IOException {

    //5.使用代理对象执行方法
    List<User> users = userDao.findAll();
    for (User user:users){
        System.out.println("-----每个用户信息----");
        System.out.println(user);
        System.out.println(user.getAccounts());
    }
}
```

查询结果:此时与在sql中查询的结果一致。见上图。  

```shell
-----每个用户信息----
User{id=41, username='老王', birthday=Tue Feb 27 17:47:08 CST 2018, sex='男', address='北京'}
[]
-----每个用户信息----
User{id=42, username='小二王', birthday=Fri Mar 02 15:09:37 CST 2018, sex='女', address='北京金燕龙'}
[]
-----每个用户信息----
User{id=43, username='小二王', birthday=Sun Mar 04 11:34:34 CST 2018, sex='女', address='北京金燕龙'}
[]
-----每个用户信息----
User{id=45, username='传智播客', birthday=Sun Mar 04 12:04:06 CST 2018, sex='男', address='北京金燕龙'}
[Account{id=2, uid=45, money=1000.0}]
-----每个用户信息----
User{id=46, username='老王', birthday=Wed Mar 07 17:37:26 CST 2018, sex='男', address='北京'}
[Account{id=1, uid=46, money=1000.0}, Account{id=3, uid=46, money=2000.0}]  //拥有两个的会直接封装成一个list。
-----每个用户信息----
User{id=48, username='小马宝莉', birthday=Thu Mar 08 11:44:00 CST 2018, sex='女', address='北京修正'}
[]
```

## 3.2 维护多对多

用户和角色，用户可以有多个角色，一个角色也可以有多个用户，故二者是多对多。

需要实现的功能：

1. 当查询用户时，可以同时得到用户所包含的角色信息
2. 当查询角色时，可以同时得到角色的所赋予的用户信息

首先是添加数据库支持：

```sql
# 创建角色表
DROP TABLE IF EXISTS `role`;
CREATE TABLE `role` (
  `id` int(11) NOT NULL COMMENT '编号',
  `role_name` varchar(30) default NULL COMMENT '角色名称',
  `role_desc` varchar(60) default NULL COMMENT '角色描述',
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
# 添加角色数据
insert  into `role`(`ID`,`ROLE_NAME`,`ROLE_DESC`) values (1,'院长','管理整个学院'),(2,'总裁','管理整个公司'),(3,'校长','管理整个学校');
# 创建用户角色表，也就是中间表
# uid 和 rid是复合主键，同时也是外键
DROP TABLE IF EXISTS `user_role`;
CREATE TABLE `user_role` (
  `uid` int(11) NOT NULL COMMENT '用户编号',
  `rid` int(11) NOT NULL COMMENT '角色编号',study_test
  PRIMARY KEY  (`uid`,`rid`),
  KEY `FK_Reference_10` (`rid`),
  CONSTRAINT `FK_Reference_10` FOREIGN KEY (`rid`) REFERENCES `role` (`id`),
  CONSTRAINT `FK_Reference_9` FOREIGN KEY (`uid`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
# 添加用户角色数据
insert  into `user_role`(`uid`,`rid`) values (41,1),(45,1),(41,2);
```

需要实现的结果：

```sql
SELECT u.*, r.id as rid, r.role_name, r.role_desc FROM role r 
	LEFT OUTER JOIN user_role ur ON r.id = ur.rid 
	LEFT OUTER JOIN user u ON ur.uid = u.id;
```

[![multi4multi.png](https://pic.tyzhang.top/images/2020/05/05/multi4multi.png)](https://pic.tyzhang.top/image/d6D5)

需要实体类：user.class  

```java

public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
	//get and set to string
}
```

role.class

```java
public class Role implements Serializable {

    private Integer roleId;
    private String roleName;
    private String roleDesc;

    //多对多的映射关系；一个角色可以赋予多个用户
    private List<User> users;
    //get and set to string
}
```

查询所有的接口（省略不表）和实现类配置IRoleDao.xml  

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IRoleDao">

    <!--定义role的resultMap-->
    <resultMap id="roleMap" type="role">
        <id property="roleId" column="rid"/>
        <result property="roleName" column="role_name"/>
        <result property="roleDesc" column="role_desc"/>
        <!-- 配置role对象中users集合的映射 -->
        <collection property="users" ofType="user">
            <id property="id" column="id"/>
            <result property="username" column="username"/>
            <result property="sex" column="sex"/>
            <result property="birthday" column="birthday"/>
            <result property="address" column="address"/>
        </collection>
    </resultMap>
    <!--查询所有 对于sql语句换行的时候，需要多加空格防止换行的时候连接导致执行失败。-->
    <select id="findAll" resultMap="roleMap">
         SELECT u.*, r.id as rid, r.role_name, r.role_desc FROM role r
            LEFT OUTER JOIN user_role ur ON r.id = ur.rid
            LEFT OUTER JOIN user u ON ur.uid = u.id;
    </select>

</mapper>
```

测试类：

```java
//根据id查询用户
@Test
public void testFindAllRole(){
    List<Role> roles = roleDao.findAll();
    for (Role role:roles){
        System.out.println("-----每个角色信息-----");
        System.out.println(role);
        System.out.println(role.getUsers());
    }
}
```

对应测试结果：

```shell
-----每个角色信息-----
Role{roleId=1, roleName='院长', roleDesc='管理整个学院'}
[User{id=41, username='老王', birthday=Tue Feb 27 17:47:08 CST 2018, sex='男', address='北京'}, User{id=45, username='传智播客', birthday=Sun Mar 04 12:04:06 CST 2018, sex='男', address='北京金燕龙'}]
-----每个角色信息-----
Role{roleId=2, roleName='总裁', roleDesc='管理整个公司'}
[User{id=41, username='老王', birthday=Tue Feb 27 17:47:08 CST 2018, sex='男', address='北京'}]
-----每个角色信息-----
Role{roleId=3, roleName='校长', roleDesc='管理整个学校'}
[]
```

可以看出查询的结果与在sql中查询的是一致的。

# 4.JNDI

可以了解一下。。模仿window中的注册表。一种数据保存形式？  

JNDI: Java Naming and Directory Interface是SUN公司推出的一套规范，属于JavaEE技术之一，目的是模仿Windows系统中的注册表。

