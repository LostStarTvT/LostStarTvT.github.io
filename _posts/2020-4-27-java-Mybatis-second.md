---

layout: post
title: Mybatis：增删改查
tags: java  
---


> 记录Mybatis的增删改查和结果集深入和参数深入。

##  目录
* 目录
{:toc}
# 1.Mybatis的curd

在进行curd操作之前需要结合第一篇博客mybatis的入门进行构建项目，具体不表。

## 1.1基础操作

对于IUserDao.java

```java
public interface IUserDao {

    /**
     * 查询所有
     * @return
     */
    List<User> findAll();

    /**
     * 保存用户
     * @param user
     */
    void saveUser(User user);

    /**
     * 更新用户
     * @param user
     */
    void updateUser(User user);

    /**
     * 删除用户
     * @param userId
     */
    void deleteUser(Integer userId);

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    User findById(Integer userId);

    /**
     * 根据名称模糊查询用户
     * @param username
     * @return
     */
    List<User> findByName(String username);

    /**
     *  使用聚合函数查询所有
     * @return
     */
    int findTotal();
}

```

对应的IUserDao.xml文件，mybatis自动实现接口实现类，下面的xml与上面的接口是一一对应的。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IUserDao">
    <!--查询所有-->
    <select id="findAll" resultType="com.diaowenjie.domain.User">
        select * from user;
    </select>

    <!--保存用户-->
    <insert id="saveUser" parameterType="com.diaowenjie.domain.User">
        insert  into user (username,address,sex,birthday) values (#{username},#{address},#{sex},#{birthday});
    </insert>

    <!--更新用户-->
    <update id="updateUser" parameterType="com.diaowenjie.domain.User">
        update user set username=#{username},address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id};
    </update>

    <!--删除用户 其中占位符id是可以随意写 因为只有一个-->
    <delete id="deleteUser" parameterType="Integer">
        delete from user where id = #{id};
    </delete>

    <!--根据id查询用户 -->
    <select id="findById" parameterType="Integer" resultType="com.diaowenjie.domain.User">
        select * from user where id  = #{id};
    </select>

    <!--根据名称模糊查询用户 -->
    <select id="findByName" parameterType="String" resultType="com.diaowenjie.domain.User">
--         select * from user where username like #{name};
        select * from user where username like '%${value}%'; <!--这种了解，上面的用的更多。-->
    </select>

    <!--获取用户的总记录条数-->
    <select id="findTotal" resultType="int" >
        select count(id) from user;
    </select>
</mapper>
```

对应的测试类：

```java
/**
 * Describe: mybatis 测试类。crud
 *
 * @author Seven on 2020/4/22
 */
public class MybatisTest {

    private InputStream in;
    private SqlSession sqlSession;
    private IUserDao userDao;

    //运行的初始化方法
    @Before
    public void init() throws IOException {
        //1.读取配置文件
        in = Resources.getResourceAsStream("sqlMapConfig.xml");
        //2.创建sqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产SQLSession对象
        sqlSession = factory.openSession();
        //4.使用SQLSession创建Dao接口代理对象
        userDao = sqlSession.getMapper(IUserDao.class);
    }

    //运行的结束方法
    @After
    public void destroy() throws IOException {
        //提交事务  不添加这一句数据写不进去 会回滚
        sqlSession.commit();
        //6.释放资源
        sqlSession.close();
        in.close();
    }

    @Test
    public void  testFindAll() throws IOException {

        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for (User user:users){
            System.out.println(user);
        }
    }

    @Test
    public void testSave(){
        User user = new User();
        user.setUsername("测试");
        user.setAddress("河南郑州");
        user.setSex("男");
        user.setBirthday(new Date());
        //5.执行保存用户
        userDao.saveUser(user);
    }

    //更新用户
    @Test
    public void testUpdate(){
        User user = new User();
        user.setId(5);
        user.setUsername("测试");
        user.setAddress("河南郑州");
        user.setSex("女");
        user.setBirthday(new Date());

        userDao.updateUser(user);
    }

    //删除用户
    @Test
    public void testDelete(){
        userDao.deleteUser(5);
    }

    //根据id查询用户
    @Test
    public void testFindById(){
        User user = userDao.findById(1);
        System.out.println(user);
    }
    
    //根据username查询
    @Test
    public void testFindByName(){
        //因为是模糊查询，所有需要提供%
        List<User> users = userDao.findByName("%王%");
        //List<User> users = userDao.findByName("王"); 了解
        for (User user:users){
            System.out.println(user);
        }
    }

    //测试查询总记录条数
    @Test
    public void testFindTotal(){
        Integer total = userDao.findTotal();
        System.out.println(total);
    }
}

```

## 1.2 进阶操作

**获取插入数据属性** 

因为插入的数据的时候是不需要记性设置id的，所以需要在插入时候获取到插入数据的id，可以实现下次查找的时候直接直接按照id查找，这样只需要在xml中进行更改即可。对之前的saveUser进行更改。

```xml
<!--保存用户-->
<insert id="saveUser" parameterType="com.diaowenjie.domain.User">
    <!-- 配置插入操作后，获取插入数据的id -->
    <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
        select  last_insert_id();
    </selectKey>
    insert  into user (username,address,sex,birthday) values (#{username},#{address},#{sex},#{birthday});
</insert>
```

其中order属性是在运行insert语句之前还是之后进行运行，现在设置的是之后，然后在进行插入语句测试以后

```java
@Test
public void testSave(){
    User user = new User();
    user.setUsername("测试00");
    user.setAddress("河南郑州");
    user.setSex("男");
    user.setBirthday(new Date());
    System.out.println("保存操作之前" + user);
    //5.执行保存用户
    userDao.saveUser(user);
    System.out.println("保存操作之后" + user);
}
```

```shell
保存操作之前User{id=null, username='测试00', birthday=Tue Apr 28 15:23:16 CST 2020, sex='男', address='河南郑州'}
保存操作之后User{id=6, username='测试00', birthday=Tue Apr 28 15:23:16 CST 2020, sex='男', address='河南郑州'}
```

可以看出来，保存以后，user中的id进行更新了，即获取到了对应的属性。

# 2.参数深入

## 2.1传递简单类型

Mybatis使用OGNL表达式解析对象字段的值，#{}或者${}括号中的值为pojo属性名称。

OGNL表达式：

Object Graphic Navigatio Language

对象	   图           导航  		 语言

他是通过对象的取值方法来获取数据，在在写法上把get给省略了。

比如：我们获取用户的名称：

1. 类中的写法：user.getUsername();
2. OGNL表达式写法 user.username;

Mybatis中为什么能直接写username，而不是user.呢：因为在parameterType中已经提供了属性所属的类，所以此时不需要写对象名。

## 2.2 传递pojo包装对象

开发中通过pojo传递查询条件，查询条件是综合的查询条件，不仅包括用户查询条件还包括其他的查询条件，这时候可以使用包装对象传递输入参数。  

对于查询条件来说，可以用一个对象进行封装，这样便能够更好的包装起来，实现如下：  

设置一个新的类，queryVo.class 里面分装了查询条件。  

```java
public class QueryVo {
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}

//通过queryVo对象进行查询数据
/**
  * 根据QueryVo中的条件查询用户
  * @param vo
  * @return
*/
List<User> findUserByVo(QueryVo vo);
```

此时对应的xml配置文件

```xml
<!--根据QueryVo的条件查询用户 传递过来的是一个对象，然后通过OGNL表达式获取到对应的属性-->
<select id="findUserByVo" parameterType="com.diaowenjie.domain.QueryVo" resultType="com.diaowenjie.domain.User">
    select * from user where username like #{user.username};
</select>
```

对应的测试类：

```java
//测试使用QueryVo作为查询条件
@Test
public void testFindByVo(){
    QueryVo queryVo = new QueryVo();

    User user = new User();
    user.setUsername("%测%");
    queryVo.setUser(user);
    //因为是模糊查询，所有需要提供%
    List<User> users = userDao.findUserByVo(queryVo);
    for (User u:users){
        System.out.println(u);
    }
}
```

```shell
User{id=1, username='测试', birthday=Wed Apr 22 00:00:00 CST 2020, sex='男', address='北京'}
User{id=6, username='测试00', birthday=Tue Apr 28 00:00:00 CST 2020, sex='男', address='河南郑州'}
```

以上这种实现方法，可以将查询条件进行封装，能够更好地优化程序。

# 3.结果集的深入

mysql数据库中，window下，userName与username是一样的，即不区分大小写。但是linux下是严格区分的。 当java实体类与数据库中的名称不一致时，此时从数据库中查询的结果边不能够正确的赋值到java实体类中。

## 3.1 结果集起别名

一个好的解决方案就是将查询结果中的属性起别名，使别名与实体类名称保持一致。如下所示：

java实体类：

```java
public class User implements Serializable {
    private Integer userId;
    private String userName;   
    //getter and setter
}
```

此时对应的数据库为：

```sql
CREATE TABLE `user` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`username` VARCHAR(50) NOT NULL COLLATE 'utf8_unicode_ci',
	INDEX `id` (`id`)
)
```

可以看出此时二者名称是不一致的，这样会导致查询的结果不能够很好的封装到实体类中，解决方案在查询是为结果集取别名，其中别名与java实体类名称保持一致。

```xml
<!--查询所有-->
<select id="findAll" resultType="com.diaowenjie.domain.User">
    select id as userId, username as userName from user;
</select>
```

此时便能够将数据封装进去，这种效果是最好的。

## 3.2 配置映射关系

在IUserDao.xml中配置映射关系

```xml
<!--配置 查询结果的列名和实体类的属性名的对应关系-->
<resultMap id="userMap" type="com.diaowenjie.domain.User" >
    <!--主键字段的对应 -->
    <id property="userId" column="id"/>
    <!--非主键字段的对应-->
    <result property="userName" column="username"/>
</resultMap>
```

然后将查询结果的xml使用：

```xml
<!--查询所有-->
<select id="findAll" resultMap="userMap">
    select * from user;
</select>
```

需要将resultType更改为resultMap。

此种方法开发效率更高，但是执行效率更低一点。

# 4.SqlMapConfig.xml配置文件

SqlMapConfig.xml中配置的内容和顺序如下：

1. properties(属性)
2. setting(全局配置参数)
3. typeAliases(类型别名)
4. typeHandlers(类型处理器)
5. objectFactory(对象工厂)
6. plugins(插件)
7. evironments(环境集合属性对象)
   1. environment(环境子属性对象)
      1. transctionManager(事务管理器)
      2. dataSource(数据源)
8. mappers(映射器)

## 4.1使用properties进行配置jdbc

**Properties属性**  

配置Properties可以在标签内部配置连接信息，也可以通过属性引用外部配置文件信息。其中可以使用ulr和resource两种写法。

```xml
<properties resource="jdbcConfig.properties"></properties>
<properties url="jdbcConfig.properties"></properties>
```

**resource属性：**常用的。用于指定配置文件的位置，是按照类路径的写法来写，必须存在于类路径下。  

**url属性：**是要求按照url的写法来写地址。e.g. http://localhost:8080/mybatiserver/demo  

1.首先是配置properties文件。在里面写好jdbc配置  

```properties
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/study_test
jdbc.username = root
jdbc.password = A123456
```

2.更改sqlMapConfig.xml配置文件

```xml
<!--引入上面的properties文件-->
<properties resource="jdbcConfig.properties"></properties>

<dataSource type="POOLED">
    <!-- 配置连接数据的四个基本信息 进行读取配置文件中的属性，属性名要一一对应 -->
    <property name="driver" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</dataSource>
```

然后便能正常的运行，这样就将重要的配置抽取出来。  

**typeAliases属性**  

使用typeAliases配置别名，**它只能配置domain中类的别名**。

sqlMapConfig.xml配置文件中进行配置。

```xml
<!--只能配置domain中的别名-->
<typeAliases>
    <!--用于配置别名，type属性指定的是实体类全限定名，alias属性指定别名 当制定了别名就不区分大小写-->
    <typeAlias type="com.diaowenjie.domain.User" alias="user"></typeAlias>
</typeAliases>
```

然后在IUserDao.xml中就可以使用别名:

```xml
<!--查询所有 直接使用别名。-->
<select id="findAll" resultType="user">
    select * from user;
</select>
```

但是以上的配置会出现起很多别名的情况，还有一种方式就是直接将包配置到别名上：

```xml
<typeAliases>
    <!--用户指定要配置别名的包，当指定以后，该包下的实体类都会注册别名，并且类名就是别名，不在区分大小写-->
    <package name="com.diaowenjie.domain"/>
</typeAliases>
```

这样在IUserDao.xml使用实体类的时候，直接使用类名就行，上面的查询所有也可以直接的使用，这种方法更加的简单。

**Mappers中的package属性:**  

```xml
<!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
<mappers>
    <!--<mapper resource="com/diaowenjie/dao/IUserDao.xml"/>-->
    <!-- package标签是用于指定dao接口所在的包，当指定了之后就需要在写mapper以及resource或者class了-->
    <package name="com.diaowenjie.dao"/>
</mappers>
```

