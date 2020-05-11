---

layout: post
title: Mybatis：延迟加载策略 & 缓存
tags: java  
---


> 记录Mybatis的延迟加载策略

##  目录
* 目录
{:toc}
# 1.延迟加载

问题：在一对多中，当我们有一个用户，他有100个账户。

1. 在查询该用户的时候，要不要把关联的账户查出来？
2. 在查询账户时候，要不要把关联的用户查出来？

推论：

1. 在查询用户的时候，用户下的账户信息是什么时候使用什么时候查询。
2. 在查询账户时，账户的所属用户信息应该是随着账户查询时一起查询出来。

在对应的四种表关系中：一对多，多对一，一对一，多对多：

1. 一对多，多对多：通常情况下我们都是采用延迟加载。
2. 多对一，一对一：通常情况下我们都是采用立即加载。

- 延迟加载
  - 在真正使用数据时才发起查询，不用的时候不查询，按需加载（懒加载）。
- 立即加载
  - 不管用不用，只要一调用方法，马上发起查询。

## 1.1 一对一实现延迟加载

在真正使用数据时菜发起查询，不用的时候不查询，按需加载（懒加载）

使用一对一的案例进行配置延迟加载：

主要使用的标签为：

```xml
<!--
 配置懒加载。
 一对一的关系映射，配置封装user的内容，
 select 属性指定的内容：查询用户的唯一标识
 column 属性指定的内容：用户根据id查询时，所需要的参数的值。
 表示的含义也就是，当查询到这一行的时候，使用com.diaowenjie.dao.IUserDao.findById这条语句进行查询，并且将uid得值传递进去，这样就是
 不用写复杂的sql语句，而是纯使用java去时下，可以参考前面笔记的一对一纯sql的实现方式。
-->
<association property="user" column="uid" javaType="user" select="com.diaowenjie.dao.IUserDao.findById"/>
```

1.改造IAccountDao.xml使其进行懒加载

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IAccountDao">

    <!--定义role的resultMap-->
    <resultMap id="accountUserMap" type="account">
        <id property="id" column="id"/>
        <result property="uid" column="uid"/>
        <result property="money" column="money"/>
        <!--
        配置懒加载。
        一对一的关系映射，配置封装user的内容，
        select 属性指定的内容：查询用户的唯一标识
        column 属性指定的内容：用户根据id查询时，所需要的参数的值。
        -->
        <association property="user"  javaType="user" select="com.diaowenjie.dao.IUserDao.findById" column="uid"/>
    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="accountUserMap">
         SELECT  * FROM account;
    </select>
</mapper>
```

2.调用的是IUserDao中的findByID属性

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IUserDao">

    <!--根据id查询用户 -->
    <select id="findById" parameterType="Integer" resultType="user">
        select * from user where id  = #{id};
    </select>
</mapper>
```

3.开启Mybatis懒加载设置

```xml
<!--配置参数-->
<settings>
    <!--开启Mybatis支持延迟加载-->
    <setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

对比之前一对一的xml配置文件，其映射关系更加的简洁，但是需要调用其xml中的配置。通过以上便能实现需要进行懒加载。

## 1.2 一对多延迟加载

原理也是一样，需要哪行查哪行。每显示一行都会获取uid并调用`select="com.diaowenjie.dao.IAccountDao.findAccountByUid" `语句去查询需要的用户账号信息，这样的写法其实是比sql语句更好些。

1.改造IUserDao.xml将其实现懒加载

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IUserDao">

    <!--定义user的resultMap-->
    <resultMap id="userAccountMap" type="user">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="address" column="address"/>
        <result property="sex" column="sex"/>
        <result property="birthday" column="birthday"/>
        <!--配置user对象中accounts集合的映射-->
        <collection property="accounts" ofType="account"  select="com.diaowenjie.dao.IAccountDao.findAccountByUid" column="id"/>

    </resultMap>
    <!--查询所有-->
    <select id="findAll" resultMap="userAccountMap">
        SELECT * FROM user;
    </select>
</mapper>
```

2.调用的是IAccountDao.xml中的查询方法

```xml
<!--根据用户id查询列表-->
<select id="findAccountByUid" resultType="account">
    select *  from account where uid = #{uid};
</select>
```

然后便实现了多对多的懒加载，这种加载可以通过查看日志日志如下：

[![lazymul4mul.png](https://pic.tyzhang.top/images/2020/05/06/lazymul4mul.png)](https://pic.tyzhang.top/image/dHXK)

可以看出是在需要的时候，展示哪一行就调用该行的数据去进行执行，并且是简单的sql语句，传递的是该行的参数，uid。

# 2.缓存

- 什么是缓存？
  - 存在于内存中的临时数据。
- 为什么使用缓存？
  - 减少和数据库的交互次数，提高执行效率
- 适用缓存的
  - 经常查询并且不经常改变的。
  - 数据的正确与否对最终结果影响不大的、
- 不适应缓存的
  - 经常改变的数据
  - 数据的正确与否对最终结果影响很大的 
  - 例如：商品的库存，银行的汇率，股市的牌价。

## 2.1 一级缓存

它指的是Mybatis中SQLSession对象的缓存。当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供的一块区域中。该区域的结构是一个Map，当我们再次查询同样的数据，mybatis会先去SQLSession中查询是否有，有的话就直接拿出来用。

当SqlSession对象消失时，mybatis的一级缓存也就消失了。

同一个session中的话，其查询的用户数据是同一个。

```java
//测试一级缓存
@Test
public void testFirstLevelCache(){
    User user1 = userDao.findById(41);
    System.out.println(user1);

    User user2 = userDao.findById(41);
    System.out.println(user2);

    System.out.println(user1 == user2);
}
```

输出的结果如下：可以看出是同一个对象。

```shell
com.diaowenjie.domain.User@5c669da8
com.diaowenjie.domain.User@5c669da8
true
```

当进行关闭session重新打开时候的输出：

```java
//测试缓存。
@Test
public void testFirstLevelCache(){
    User user1 = userDao.findById(41);
    System.out.println(user1);

    sqlSession.close();
    
    //再次获取sqlSession对象
    sqlSession = factory.openSession();
    userDao = sqlSession.getMapper(IUserDao.class);

    User user2 = userDao.findById(41);
    System.out.println(user2);

    System.out.println(user1 == user2);
}
```

此时输出的结果就是两个二者并不是一致的。从日志上看，也是进行了两次的查询操作。

```verilog
[DEBUG]: Created connection 1324829744.  
[DEBUG]: Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4ef74c30]  
[DEBUG] : select * from user where id = ?;   
[DEBUG] : ==> Parameters: 41(Integer)  
[DEBUG] : <==      Total: 1  
com.diaowenjie.domain.User@7cb502c
[DEBUG]: Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4ef74c30]  
[DEBUG] : Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@4ef74c30]  
[DEBUG] : Returned connection 1324829744 to pool.  
[DEBUG] : Opening JDBC Connection  
[DEBUG] : Checked out connection 1324829744 from pool.  
[DEBUG]: Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4ef74c30]  
[DEBUG] : ==>  Preparing: select * from user where id = ?;   
[DEBUG] : ==> Parameters: 41(Integer)  
[DEBUG]: <==      Total: 1  
com.diaowenjie.domain.User@1b8a29df
false
```

也可以使用刷新缓存的方式进行：和以上的效果是一致的，

```java
@Test
public void testFirstLevelCache(){
    User user1 = userDao.findById(41);
    System.out.println(user1);

    sqlSession.clearCache(); //刷新缓存
    userDao = sqlSession.getMapper(IUserDao.class);

    User user2 = userDao.findById(41);
    System.out.println(user2);

    System.out.println(user1 == user2);
}
```

一级缓存的范围，当调用SQLSession的修改，添加，删除，commit()，close()等方法时，就会清除一级缓存。

如下所示，在两次查询中插入一个更新操作，然后日志如下并且两个对象也不一样。

```java
//测试缓存。
@Test
public void testClearCache(){
    User user1 = userDao.findById(41);
    System.out.println(user1);
	//更新用户
    user1.setUsername("cccc");
    user1.setAddress("cccc");
    userDao.updateUser(user1);

    User user2 = userDao.findById(41);
    System.out.println(user2);

    System.out.println(user1 == user2);
}
```

```verilog
[DEBUG]: Opening JDBC Connection  
[DEBUG]: Created connection 1324829744.  
[DEBUG]: Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@4ef74c30]  
[DEBUG]: ==>  Preparing: select * from user where id = ?;   
[DEBUG]: ==> Parameters: 41(Integer)  
[DEBUG]: <==      Total: 1  
com.diaowenjie.domain.User@7cb502c
[DEBUG]: ==>  Preparing: update user set username = ?, address = ? where id = ?;   
[DEBUG]: ==> Parameters: cccc(String), cccc(String), 41(Integer)  
[DEBUG]: <==    Updates: 1  
[DEBUG]: ==>  Preparing: select * from user where id = ?;   
[DEBUG]: ==> Parameters: 41(Integer)  
[DEBUG]: <==      Total: 1  
com.diaowenjie.domain.User@4fbe37eb
false
```

输出的结果为false。

## 2.1 二级缓存

它指的是mybatis中的sqlSessionFatory对象的缓存，由同一个SqlSessionFactory对象创建的SQLSession共享缓存。**一级缓存关闭SQLSession时候就会重新查询，二级缓存表示关闭SQLSession也不会重新查询而是使用缓存。**

[![secondLevelCache.png](https://pic.tyzhang.top/images/2020/05/07/secondLevelCache.png)](https://pic.tyzhang.top/image/dX2f)

二级缓存的使用步骤；

1. 让mybatis框架支持二级缓存(在sqlMapConfig.xml中配置)。

   ```xml
   <!--配置参数-->
   <settings>
       <!--开启缓存支持-->
       <setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. 让当前的映射文件支持二级缓存(在IUserDao.xml中配置)。

3. 让当前的操作支持二级缓存(在select标签中配置)。

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.diaowenjie.dao.IUserDao">
       <!--开启user支持二级缓存-->
       <cache/>
       <!--根据id查询用户 开启cache=true -->
       <select id="findById" parameterType="Integer" resultType="user" useCache="true">
           select * from user where id  = #{id};
       </select>
   </mapper>
   ```

经过以上的配置以后进行测试：

```java
//测试缓存。开启两个sqlSession进行查询数据。
@Test
public void testFirstLevelCache(){
    SqlSession sqlSession1 = factory.openSession();
    IUserDao dao1 = sqlSession1.getMapper(IUserDao.class);
    User user1 = dao1.findById(41);
    System.out.println(user1);
    sqlSession1.close();

    SqlSession sqlSession2 = factory.openSession();
    IUserDao dao2 = sqlSession2.getMapper(IUserDao.class);
    User user2 = dao2.findById(41);
    System.out.println(user2);
    sqlSession1.close();

    System.out.println(user1 == user2);
}
```

此时输出的日志文件

```verilog
[DEBUG]: Created connection 1200470358.  
[DEBUG]: Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@478db956]  
[DEBUG]: ==>  Preparing: select * from user where id = ?;   
[DEBUG]: ==> Parameters: 41(Integer)  
[DEBUG]: <==      Total: 1  
com.diaowenjie.domain.User@2b175c00
[DEBUG]: Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@478db956]  
[DEBUG]: Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@478db956]  
[DEBUG]: Returned connection 1200470358 to pool.  
[DEBUG]: Cache Hit Ratio [com.diaowenjie.dao.IUserDao]: 0.5  
com.diaowenjie.domain.User@10f7f7de
false
```

虽然对比结果为false，是因为二级缓存只存储数据不存储对象的原因。但是日志文件显示只查询了一次，并且有cache hit ratio的字样。

# 3. 注解开发

## 3.1 环境搭建

主要就是涉及到一个配置文件:其中这里的注解与配置文件与spring的不一样，其实和使用xml的差不多，无需指定开启注解扫描。  

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--mybatis主配置文件-->
<configuration>
    <!-- 读取数据库配置文件 -->
    <properties resource="jdbcConfig.properties"/>

    <!--只能配置domain中的别名-->
    <typeAliases>
        <!--用户指定要配置别名的包，当指定以后，该包下的实体类都会注册别名，并且类名就是别名，不在区分大小写-->
        <package name="com.diaowenjie.domain"/>
    </typeAliases>

    <!--配置环境-->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"/>
            <!-- 配置数据源(连接池)-->
            <dataSource type="POOLED">
                <!-- 配置连接数据的四个基本信息-->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!--指定带有注解的dao接口所在位置-->
    <mappers>
        <!-- package标签是用于指定dao接口所在的包，当指定了之后就不需要在写mapper以及resource或者class了-->
        <package name="com.diaowenjie.dao"/>
    </mappers>

</configuration>
```

user实体类

```java
public class User implements Serializable {

    private Integer id;
    private String username;
    private String address;
    private String sex;
    private Date birthday;
    //set and get to string
}
```

## 3.2 单表CRUD

主要写一下核心算法， 具体的实现见xml的crud。

对应的接口注解IUserDao.java

```java
/**
 * 在myBatis中只对crud共有四个注解
 * @Select @Insert @Update @Delete
 */
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    List<User> findAll();

    /**
     * 保存用户
     * @param user
     */
    @Insert("insert into user (username,address,sex,birthday) values (#{username},#{address},#{sex},#{birthday})")
    void saveUser(User user);

    /**
     * 更新用户
     * @param user
     */
    @Update("update user set username=#{username},address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id}")
    void updateUser(User user);

    /**
     * 删除用户
     * @param userId
     */
    @Delete("delete from user where id = #{id}")
    void deleteUser(Integer userId);

    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    @Select("select * from user where id  = #{id}")
    User findById(Integer userId);

    /**
     * 根据名称模糊查询用户
     * @param username
     * @return
     */
    @Select("select * from user where username like #{name}")
    List<User> findByName(String username);

    /**
     *  使用聚合函数查询所有
     * @return
     */
    @Select("select count(id) from user;")
    int findTotal();
}
```

对应的测试类：

```java
public class AnnotationCRUDTest {
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
        user.setId(49);
        user.setUsername("测试");
        user.setAddress("河南郑州");
        user.setSex("女");
        user.setBirthday(new Date());

        userDao.updateUser(user);
    }

    //删除用户
    @Test
    public void testDelete(){
        userDao.deleteUser(49);
    }

    //根据id查询用户
    @Test
    public void testFindById(){
        User user = userDao.findById(49);
        System.out.println(user);
    }

    //根据username查询
    @Test
    public void testFindByName(){
        //因为是模糊查询，所有需要提供%
        List<User> users = userDao.findByName("%王%");
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

**表名称与实体类属性名称不一致时，需要更改:**

```java
/**
 * 查询所有用户
 * @return
 */
@Select("select * from user")
@Results(id = "userMap",value = {
    @Result(id = true, property = "userId",column = "id"),
    @Result(property = "userName",column = "username"),
    @Result(property = "userBirthday",column = "birthday"),
    @Result(property = "userSex",column = "sex"),
    @Result(property = "userAddress",column = "address"),
})
List<User> findAll();
```

- `@Results` 注解用于定义映射结果集，相当于标签 。其中，id 属性为唯一标识。value 属性用于接收 `@Result[]` 注解类型的数组。
- `@Result` 注解用于定义映射关系，相当于标签。其中，id 属性指定主键。property 属性指定实体类的属性名，column 属性指定数据库表中对应的列。
- `@ResultMap` 注解用于引用 `@Results` 定义的映射结果集，避免了重复定义映射结果集。

此时通过id其他的方法也能直接的引用上面的对应关系

```java
/**
 * 根据id查询用户
 * @param userId
 * @return
 */
@Select("select * from user where id  = #{id}")
//@ResultMap(value = {"userMap"}) //标准写法
@ResultMap("userMap") //简略写法
User findById(Integer userId);
```

## 3.3 一对一立即查询

```java
public class Account {
    private Integer id;
    private Integer uid;
    private Double money;

    //多对一(mybatis中称之为一对一)的映射，一个账户只能属于一个用户
    private User user;
}
```

```java
public class User implements Serializable {

    private Integer userId;
    private String userName;
    private String userAddress;
    private String userSex;
    private Date userBirthday;  
    
}
```



```java
public interface IAccountDao {
    /**
     *  查询所有账户 并且获取每个账户所属的用户信息
     * @return
     */
    @Select("select * from account")
    @Results(id="accountMap", value={
            @Result(id=true, column = "id", property = "id"),
            @Result(column = "uid", property = "uid"),
            @Result(column = "money", property = "money"),
            @Result(property = "user", column = "uid",
                    one=@One(select="com.diaowenjie.dao.IUserDao.findById", //调用IUserDao方法
                            fetchType= FetchType.EAGER))
    })
    // @one 表示一对一
    List<Account> findAll();
}
```

`@One` 注解相当于标签 ，是多表查询的关键，在注解中用来指定子查询返回单一对象。其中，select 属性指定用于查询的接口方法，fetchType 属性用于指定立即加载或延迟加载，分别对应 FetchType.EAGER 和 FetchType.LAZY。

```java
public interface IUserDao {
    /**
     * 根据id查询用户
     * @param userId
     * @return
     */
    @Select("select * from user where id  = #{id}")
    @Results(id = "userMap",value = {
            @Result(id = true, property = "userId",column = "id"),
            @Result(property = "userName",column = "username"),
            @Result(property = "userBirthday",column = "birthday"),
            @Result(property = "userSex",column = "sex"),
            @Result(property = "userAddress",column = "address"),
    })
    User findById(Integer userId);
}
```

## 3.4 一对多懒加载

使用的是一个用户有多个账户测试

1.用户的实体类改造

```java
public class User implements Serializable {

    private Integer userId;
    private String userName;
    private String userAddress;
    private String userSex;
    private Date userBirthday;

    //一对多映射关系：一个用户对应多个账户
    private List<Account> accounts;
}
```

2.IUserDao.java改造，使用懒加载查询数据

```java
public interface IUserDao {
    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    @Results(id = "userMap",value = {
            @Result(id = true, property = "userId",column = "id"),
            @Result(property = "userName",column = "username"),
            @Result(property = "userBirthday",column = "birthday"),
            @Result(property = "userSex",column = "sex"),
            @Result(property = "userAddress",column = "address"),
            @Result(property = "accounts", column = "id",
                    many=@Many(select = "com.diaowenjie.dao.IAccountDao.findAccountByUid",
                            fetchType = FetchType.LAZY))
    })
    List<User> findAll();
}
```

3.因为需要查询用户账户，所以需要在账户中添加一个按照用户id查找账户的方法

```java
/**
 * 根据用户id查询账户信息
 * @param userId
 * @return
*/
@Select("select * from account where uid = #{userId}")
List<Account> findAccountByUid(Integer userId);
```

然后是测试类

```java
@Test
public void  testFindAll() throws IOException {

    //5.使用代理对象执行方法
    List<User> users = userDao.findAll();
    for (User user:users){
        System.out.println(user);
        System.out.println(user.getAccounts());
    }
}
```

此时便能够查询出相应的结果

## 3.5 开启二级缓存

1.开启配置

```xml
<settings>
    <!-- 开启缓存 -->
    <setting name="cacheEnabled" value="true"/>
</settings>
```

2.dao类上添加注释，

```java
@CacheNamespace(blocking = true)
public interface IUserDao {
	// .....
}
```

终于学完..