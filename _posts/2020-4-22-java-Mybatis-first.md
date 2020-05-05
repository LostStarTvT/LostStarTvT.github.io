---

layout: post
title: Mybatis：入门介绍和简单实现
tags: java  
---


> 记录Mybatis的入门案例和简单的实现分析。

#  目录
* 目录
{:toc}
# 1.Mybatis基础知识

[相关视频学习博客](https://blog.csdn.net/a1092882580/article/details/104086181)    [学习视频](https://www.bilibili.com/video/BV1Db411s7F5)  

三层架构：

1. 表现层：用于展示数据的
2. 业务层：是处理业务需求
3. 持久层：和数据库交互的

持久层技术解决方案：

1. JDBC技术：Connection、PreparedStatement
2. Spring的JdbcTemplate：spring对于jdbc的简单封装
3. Apache的DButils：它是和spring的JDBCTemplate很像，简单的封装。
4. 但是以上都不是框架，都是工具类，其中jdbc是规范

**mybatis概述**：它是一个优秀的基于java的持久层框架，它内部封装了jdbc，使得开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程，并且采用ORM思想解决了实体和数据库映射的问题。

**ORM：**Object Relational Mapping 对象关系映射。即把数据库表和实体类即实体类的属性对应起来，让我们可以操作实体类就可以实现操作数据表。

# 2.Mybatis入门案例

数据库配置文件：需要创建一个user表：

```mysql
-- 导出 study_test 的数据库结构
CREATE DATABASE IF NOT EXISTS `study_test`;
USE `study_test`;

-- 导出  表 study_test.user 结构
CREATE TABLE IF NOT EXISTS `user` (
  `id` int(11) NOT NULL auto_increment,
  `username` varchar(50) collate utf8_unicode_ci NOT NULL,
  `birthday` date default NULL,
  `sex` char(1) collate utf8_unicode_ci default NULL,
  `address` varchar(256) collate utf8_unicode_ci default NULL,
  KEY `id` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

-- 正在导出表  study_test.user 的数据：~3 rows (大约)
INSERT INTO `user` (`id`, `username`, `birthday`, `sex`, `address`) VALUES
	(1, '测试', '2020-04-22', '男', '北京'),
	(2, '小小', '2020-04-22', '女', '上海'),
	(3, '老王', '2019-09-22', '男', '浙江');
```

## 2.1 基于配置文件

1. 创建maven工程并且导入pom配置依赖。
2. 创建实体类和dao的接口
3. 创建mybatis的主配置文件 SqlMapConfig.xml
4. 创建映射配置文件IUserDao.xml

导入依赖：

```xml
<dependencies>
    
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.4</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.6</version>
    </dependency>
    
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

创建实体类User.class

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
	//get and set

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```

IUserDao.class接口
```java
public interface IUserDao {

    /**
     * 查询所有
     * @return
     */
    List<User> findAll();
}
```

mybatis主配置文件resource/sqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--mybatis主配置文件-->
<configuration>
    <!--配置环境-->
    <environments default="mysql">
        <!-- 配置mysql的环境-->
        <environment id="mysql">
            <!-- 配置事务的类型-->
            <transactionManager type="JDBC"></transactionManager>
            <!-- 配置数据源(连接池)-->
            <dataSource type="POOLED">
                <!-- 配置连接数据的四个基本信息-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/study_test"/>
                <property name="username" value="root"/>
                <property name="password" value="A123456"/>
            </dataSource>
        </environment>
    </environments>
    <!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
    <mappers>
        <mapper resource="com/diaowenjie/dao/IUserDao.xml"/>
    </mappers>
</configuration>
```

映射配置文件/resource/com/diaowenjie/dao/IUserDao.xml，三层文件夹目录，需要和接口文件IUserDao.class目录保持一致。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.diaowenjie.dao.IUserDao">
    <!--查询所有-->
    <select id="findAll" resultType="com.diaowenjie.domain.User">
        select * from user
    </select>
</mapper>
```

**搭建环境注意事项：**  

1. 创建IUserDao.xml和IUserDao.java是为了和之前保持一致，在Mybatis中它把持久层的操作接口和映射文件叫做Mapper，所以IUserDao和IUserMapper是一样的。
2. 在idea中创建目录时，它和包是不一样的，包输入`com.diaowenjie.dao`这是三级目录结构，但是文件夹这样输入时是一级目录，所以需要一个一个的新建文件夹。
3. mybatis的映射配置文件位置必须和dao接口的包结构相同。
4. 映射文件的`mapper`标签和`namespace`属性的取值必须是dao接口的全类型限定名。
5. 映射配置文件的操作配置(select)，id属性的取值必须是dao接口的方法名。

当我们遵循了3.4.5以后，我们在开发中就无需在写dao的实现类。  

配置一个test测试类，测试整个流程

```java
public class MybatisTest {

    public static void main(String[] args) throws IOException {
        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("sqlMapConfig.xml");
        //2.创建sqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产SQLSession对象
        SqlSession session = factory.openSession();
        //4.使用SQLSession创建Dao接口代理对象
        IUserDao userDao = session.getMapper(IUserDao.class);
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();

        for (User user:users){
            System.out.println(user);
        }
        //6.释放资源
        session.close();
        in.close();
    }
}
```

然后便能够查询出数据库中的所有数据

```shell
User{id=1, username='老张', birthday=Wed Apr 22 00:00:00 CST 2020, sex='男', address='北京'}
User{id=2, username='小小', birthday=Wed Apr 22 00:00:00 CST 2020, sex='女', address='上海'}
User{id=3, username='老王', birthday=Sun Sep 22 00:00:00 CST 2019, sex='男', address='浙江'}
```

## 2.2 基于注解方式

以上基于xml配置的进行数据查询，基于注解的也很简单，只需要更改一下IUserDao.class接口：

```java
public interface IUserDao {

    /**
     * 查询所有
     * @return
     */
    @Select("select * from user")
    List<User> findAll();
}
```

然后更改Mybatis的配置文件，将制定的xml文件更改为指定class文件：

```xml
<!--指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
<!--    <mappers>-->
<!--        <mapper resource="com/diaowenjie/dao/IUserDao.xml"/>-->
<!--    </mappers>-->

<!--如果是注解的来配置的话，此处应该使用class属性指定被注解的dao全限定类名-->
<mappers>
    <mapper class="com.diaowenjie.dao.IUserDao"/>
</mappers>
```

并且删除映射配置文件/resource/com/diaowenjie/dao/IUserDao.xml，及其文件夹，此时运行便能够直接的查询到结果。

**明确：**

我们在开发中，都是越简便越好，所以都是采用不写dao实现类的方式，不管是使用xml还是注解配置，但是mybatis是支持写dao实现类的。  mybatis寻找sql语句都是通过`namespace + id`找到的。

# 3.自定义mybatis分析

分析mybatis的实现过程，然后实现自定义一个mybatis框架，但是我没有实现。。  

测试类的调用过程也就表现了mybatis的执行过程，然后通过分析测试类中的调用关系，可以逐步的分析出如何实现。  

```java
public static void main(String[] args) throws IOException {
    //1.读取配置文件
    InputStream in = Resources.getResourceAsStream("sqlMapConfig.xml");
    //2.创建sqlSessionFactory工厂
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    
    SqlSessionFactory factory = builder.build(in); //使用的是构造者模式，builder就是构造者，
    // 优势： 把对象的创建细节隐藏，是使用者直接调用方法即可拿到对象。
    
    //3.使用工厂生产SQLSession对象
    SqlSession session = factory.openSession(); //工厂模式
    //优势：使用工厂模式解耦，(降低类之间的依赖关系)
    
    //4.使用SQLSession创建Dao接口代理对象
    IUserDao userDao = session.getMapper(IUserDao.class); //创建Dao接口实现类使用了代理模式，
    //优势：在不修改源码的基础上对已有的方法增强。即通过一个接口返回一个接口实现类。类似于aop的实现思想。
    
    //5.使用代理对象执行方法
    List<User> users = userDao.findAll();

    for (User user:users){
        System.out.println(user);
    }
    //6.释放资源
    session.close();
    in.close();
}
```

mybatis执行过程步骤，以调用执行findAll方法为例：  

1.连接数据库信息，有了以下代码便能够创建Connection对象。  

```xml
<!-- 配置数据源(连接池)-->
<dataSource type="POOLED">
    <!-- 配置连接数据的四个基本信息-->
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/study_test"/>
    <property name="username" value="root"/>
    <property name="password" value="A123456"/>
</dataSource>
```

2.配置映射信息。  

```xml
<mappers>
    <mapper class="com.diaowenjie.dao.IUserDao"/>
</mappers>
```

3.配置执行的sql语句，获取PreparedStatement。  

```xml
<mapper namespace="com.diaowenjie.dao.IUserDao">
    <!--查询所有-->
    <select id="findAll" resultType="com.diaowenjie.domain.User">
        select * from user
    </select>
</mapper>
```

4.以上xml配置文件：用到的技术就是解析xml的技术。此处用的就是dom4j解析xml技术。

5.将以上xml解析以后，便可以进行调用。

1. 根据配置文件创建Connection对象：注册驱动，获取连接

2. 获取预处理对象PreParedStatement：此时需要执行sql语句`con.preparedStatement(sql)`从步骤3中获取。

3. 执行查询`ResultSet resultSet = prepareedStatement.executeQuery();`

4. 遍历结果集用于封装

   ```java
   List<E> list = new ArrayList();
   while(resultSet.next()){
       E elemet = (E)Class.forName(配置的全限定类名(第三步xml配置)).newInstance();
        //使用反射进行封装。
        //进行封装，把每个rs的内容都添加到element中去
        //我们的实体类属性和表中的类名是一致的。
        //于是我们就可以把表的列名看成是实体类的属性名称
        //就可以使用反射的方式来根据名称获取每个属性，并把值赋进去
       list.add(element);
   }
   ```

5. 返回list. result list;

6.要想让findAll方法执行，我们需要给方法提供两个信息。

1. 连接信息(数据库连接)
2. 映射信息
   1. 执行的SQL语句
   2. 封装结果的实体类全限定类名**(为了反射增强方法做准备，提供类名和属性)**
   3. 以上两个组合成为一个对象**Mapper**

其中Mapper的存储结构:

| String                       | Mapper                                         |
| ---------------------------- | ---------------------------------------------- |
| com.dwj.dao.IUserDao.findall | Mapper对象：String domainClassPath、String sql |

7.使用sqlSession创建Dao接口代理对象(也就之前讲的ioc中的基于接口的动态代理)，返回一个增强的findAll方法，实现类。即将xml翻译成具体的实现类。

```java
//根据dao接口的字节码创建dao的代理对象
public <T> T getMapper(Class <T> daoInterfaceClass){
    //类加载器：它使用的和被代理对象是相同的类加载器
    //代理对象要实现的接口，和被代理对象实现相同的接口
    //如何代理？：它就是一个增强的方法，需要我们自己来提供
    //	此处是一个InvocationHandler的接口，我们需要写一个该接口的实现类，在实现类中调用findAll方法
    Proxy.newInstacce(类加载器，代理对象要实现的接口字节码数组，如何代理);
}
```

第7步也就解释了为什么mybatis可以实现只使用xml配置文件可以自动的生成对应的实现类，即使用基于接口的动态代理模式实现，通过增强一个接口方法，实现该方法需要的功能，从而达到用户不需要实现该接口方法，而是自动生成。