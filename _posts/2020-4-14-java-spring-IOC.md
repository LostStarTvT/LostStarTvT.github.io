---
layout: post
title: Spring Ioc、DI 学习
tags: java  
---


> 记录spring Ioc 和DI的知识，配以简单的例子进行说明。


## 1.Spring的思想实现之工厂模式创建对象

耦合：程序之间的依赖，分为两种：

1. 类之间的依赖
2. 方法之间的依赖

解耦：降低程序间的依赖，在实际开发中，编译期间不依赖，运行时不依赖。  

实现思路：  

1. 使用反射来创建对象，而避免使用new关键字
2. 通过读取配置文件来获取需要创建的对象全限定类名。

使用工厂模式进行解耦操作示例：  

1.首选定义properties文件，即通过配置文件来进行创建对象。

```properties
accountService = com.diaowenjie.service.impl.AccountServiceImpl
accountDao = com.diaowenjie.dao.impl.AccountDaoImpl
```

2.创建工厂模式：

```java
/**
 * Describe: 创建bean对象的工厂
 *   Bean：在 计算机英语中，有可重用组件的含义。
 *
 *   如何使用工厂类进行新建对象？
 *   第一个：需要一个配置文件来配置我们的service和dao
 *          配置的内容，全限定类名。（key：value）
 *   第二个：通过读取配置文件的配置内容，反射创建对象
 * @author Seven on 2020/4/14
 */
public class BeanFactory {
    //定一个properties 对象
    private static Properties props;

    //这段代码在类加载的时候会进行运行，jvm的知识。
    static {
        try {
            //实例化props对象
            props = new Properties();
            //对去props文件。
            InputStream in  = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
            }
        } catch (IOException e) {
            throw  new ExceptionInInitializerError("初始化Bean失败");
        }
    }

    //根据bean的名称获取bean对象
    //但是此时的对象是多例的，因为每次调用工厂都会产生一个新的对象。
   public static Object getBean(String beanName){
       Object bean = null;
		
       //省略trycatch
       String  beanPath  = props.getProperty(beanName);
       //使用反射创建对象。 每次调用都是创建一个新的bean所以是多例的。
       bean = Class.forName(beanPath).getDeclaredConstructor().newInstance();
 
       return bean;
    }
}
```

以上代码表示在进行创建对象的时候，不使用new关键字，而是使用工厂类**BeanFactory.getBean(beanName)**方法获取对象，其中beanName需要时在properties文件中定义好的对象。但是以上方法有个问题就是，**新建的对象是多例的，即每调用一次getBean都是新建一个对象。**  

以下进行新建单例对象。即通过一个Map存储对象，将对象名与对象保存起来，每次获取对象都是从Map中获取，即使用配置文件中的对象名。也是spring的一个简单实现思想。

```java
/**
 * Describe: 创建bean对象的工厂
 *   Bean：在 计算机英语中，有可重用组件的含义。
 *
 *   如何使用工厂类进行新建对象？
 *   第一个：需要一个配置文件来配置我们的service和dao
 *          配置的内容，全限定类名。（key：value）
 *   第二个：通过读取配置文件的配置内容，反射创建对象
 * @author Seven on 2020/4/14
 */
public class BeanFactory {
    //定一个properties 对象
    private static Properties props;

    //实现单例模型的对象创建,首先使用一个map容器进行存储对象。
    private static Map<String,Object> beans;

    //这段代码在类加载的时候会进行运行，jvm的知识。
    static {
   			
        //省略trycatch
        //实例化props对象
        props = new Properties();
        //对去props文件。
        InputStream in  = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
        props.load(in);

        //实例化容器,实现单例模式。
        beans = new HashMap<String, Object>();
        //取出配置文件中的所有的key
        Enumeration<Object> keys = props.keys();
        while (keys.hasMoreElements()){
            //取出每个key
            String key = keys.nextElement().toString();
            //根据key获取value
            String beanPath = props.getProperty(key);
            //反射创建对象
            Object value = Class.forName(beanPath).getDeclaredConstructor().newInstance();
            //把key 和value 存入容器
            beans.put(key,value);
        }
       
    }

    //根据名称获取单例对象
    public static Object getBean(String beanName){
        return beans.get(beanName);
    }
}

```

然后在进行实例化对象的时候，直接传递类名在properties中设置的对象名即可获得需要的对象。

```java
//使用实例。 但是要进行强转，因为工厂中实例的是通过object实现的。
IAccountDao accountDao = (IAccountDao) BeanFactory.getBean("accountDao");
```

## 2. 使用Spring创建对象

控制反转(ioc)：把创建对象的权利交给框架，是框架的最重要的特征， 并非面向对象的专业术语，它包括依赖注入(DI)和依赖查找(dependency lookup)    

ioc 作用就是**降低计算机程序的耦合**。**只能降低不能完全消除**。  

使用spring创建对象的过程，首先配置文件bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--把对象的创建交给Spring来管理-->
    <bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl"/>
    <bean id="accountDao" class="com.diaowenjie.dao.impl.AccountDaoImpl"/>
</beans>
```

获取spring容器中的对象

```java
public class Client {
    public static void main(String[] args) {

		//使用spring 进行创建对象。
        //获取核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //根据id获取bean对象
        IAccountService as = (IAccountService) ac.getBean("accountService");
        //实现的两种方式。
        IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);
		//即也就是类比于上面自定义工厂类型获取对象，不是用new关键字。
        System.out.println(as);
        System.out.println(adao);
    }
}
```

以上有点像安卓里面的context对象。代码解析。

**ApplicationContext 的三个常用实现类**  

1. ClassPathXmlApplicationContext: 它可以加载类路径下的配置文件，要求配置文件必须在类路径下，不在的话加载不了
2. FileSystemXmlApplicationContext：它可以加载磁盘任意路径下的配置文件（必须有访问权限）
3. AnnotationConfigApplicationContext：他可以读取注解创建容器。

**核心容器的两个接口引发的问题：**    

1. ApplicationContext ： 单例对象适用。一般适用这个接口它在构建核心容器时，创建对象采取的策略是采用立即加载的方式，也就是说，只要已读取完配置文件马上就创建配置文件中配置的对象。

2. BeanFactory：多例对象适用。它在创建核心容器的时候，创建对象的策略是采取延迟加载的方式，也即是说，什么时候根据id创建对象了，什么时候开始真正的创建对象。

## 3.springXML配置版注入操作

### 3.1 新建bean对象

**Spring 对象的管理细节**  

1. 创建bean的三种方式

2. bean对象的作用范围

3. bean对象的生命周期

**创建bean的三种方式：**    

1. 适用默认构造函数创建。在spring的配置文件中使用bean变迁，配置id和class属性以后，并没有其他属性和标签时，采用的就是默认构造函数创建bean对象对此时如果类中没有默认构造函数，则对象无法创建。

   ```xml
   <bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl"/>
   ```

2. 使用构造工厂进行构建

   ```xml
   <!--使用工厂方法构建bean-->
   <bean id="instanceFactory" class="com.diaowenjie.factory.InstanceFactory"/>
   <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"/>
   ```

   ```java
   //构造工厂方法
   public class InstanceFactory {
       public IAccountService getAccountService(){
           return new AccountServiceImpl();
       }
   }
   ```

3. 使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器）

   ```java
   public class StaticFactory {
       public static IAccountService getAccountService(){
           return new AccountServiceImpl();
       }
   }
   ```

   ```xml
   <!--使用静态工厂类-->
   <bean id="accountService" class="com.diaowenjie.factory.StaticFactory" factory-method="getAccountService"/>
   ```

Bean的作用范围 ：**默认都是单例的。**   

bean使用scope可以指定bean的作用范围

1. **singleton  单例的（默认值）**

2. **prototype 多例的**

3. request  作用于web应用的请求范围

4. session  作用于web应用的会划范围

5. global-session  作用于集群环境的会话范围(全局会话范围)，当不是集群环境时，它就是session（即多台服务器共享session）  

**bean对象的生命周期**  

1. 单例对象的生命周期与容器相同。

2. 多例对象：
   1. 出生；当我们使用对象是spring框架为我们创建
   2. 活着：对象只要是在使用过程中就一直活着
   3. 死亡：当对象长时间不用，而且没有别的对象引用时，由java的垃圾回收器回收。

### 3.2 注入对象

spring 的依赖注入DI   

ioc的作用：就是降低程序间的耦合(依赖关系)

依赖关系的管理：以后都交给spring来维护。

当前类需要用到其他类的对象，由spring为我们提供，我们只需要在配置文件中说明。

**依赖关系的维护，就称之为依赖注入。**

**依赖注入：有三种**

1. 基本类型和string

2. 其他bean类型（在配置文件中或者注解配置过的bean）

3. 复杂类型/集合类型

**注入的方式，有三种**

   1. 使用构造函数提供

2. 使用set方法提供

3. 使用注解提供。

**构造函数的注入：**

使用标签constructor-arg

标签出现的位置，bean标签的内部，

标签中的属性

1. type:用于指定要注入的数据的类型，该数据类型也是构造函数中某个或者某些参数的类型
2. index.用于指定要注入的数据给构造函数中指定索引位置的参数复制，索引的位置是用0开始的
3. name.用于指定给构造函数中指定名称的参数复制（**常用**）
4. value.用于提供基本类型和string类型的数据
5. ref.用于指定其他的bean类型数据，他值得就是在spring的ioc核心容器中出现过的bean对象。

优势: 

在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。

弊端：

改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据。

**使用构造器构造函数的一个例子**：

java代码：

```java
public class AccountServiceImpl implements IAccountService {
	//构造方法
    public  AccountServiceImpl(String name, Date date){
        System.out.println(name + date);
    }
    public void saveAccount() {
        System.out.println("service 中的saveAccount方法被调用了。。。");

    }
}
```

对应的注入xml方式

```xml
<bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl">
    <constructor-arg name="name" value="aaa"/>
    <constructor-arg name="date" ref="now" />
</bean>
   <!--配置一个日期对象-->
<bean id="now" class="java.util.Date"/>
```

**set方法注入：更常用。**

涉及的标签，property

出现的位置：bean标签的内部

标签的属性： 

1. name：用于注入时所调用的set方法名称

2. value：用于提供基本类型和string类型的数据

3. ref：用户指定其他的bean类型属性，它指的就是在spring的ioc容器中出现过的bean对象

优势： 穿件对象时没有明确的限制，可以直接使用默认构造方法

弊端，如果 有某个成员必须有值，则获取对象是有可能set方法没有执行。

```xml
<bean id="now" class="java.util.Date"/>
<bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl">
    <property name="name" value="AA"></property>
    <property name="date" ref="now"> </property>
</bean>
```

```java
public class AccountServiceImpl implements IAccountService {

    private String name;
    private Date date;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }
```

**复杂数据的注入：**

用于给List结构集合注入的标签 List array set   

用户给Map结构集合注入的标签 map props  

其中相同类型使用相同标签赋值是一致的，所以只需要记住list 和map两个写法即可。  

```java
public class AccountServiceImpl implements IAccountService {

    private  String [] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String,String> mymMap;

    public String[] getMyStrs() {
        return myStrs;
    }

    public void setMyStrs(String[] myStrs) {
        this.myStrs = myStrs;
    }

    public List<String> getMyList() {
        return myList;
    }

    public void setMyList(List<String> myList) {
        this.myList = myList;
    }

    public Set<String> getMySet() {
        return mySet;
    }

    public void setMySet(Set<String> mySet) {
        this.mySet = mySet;
    }

    public Map<String, String> getMymMap() {
        return mymMap;
    }

    public void setMymMap(Map<String, String> mymMap) {
        this.mymMap = mymMap;
    }

    public Properties getMyProps() {
        return myProps;
    }

    public void setMyProps(Properties myProps) {
        this.myProps = myProps;
    }

    private Properties myProps;

    public void saveAccount() {
        System.out.println("service 中的saveAccount方法被调用了。。。");
    }
}

```

```xml

<bean id="map" class="com.diaowenjie.service.impl.AccountServiceImpl">
        <!--list注入方法-->
    	<property name="myList" >
            <set>
                <value>AAA</value>
                <value>BBB</value>
            </set>
        </property>
		<!--map的注入方法-->
        <property name="mymMap">
            <map>
                <entry key="CCC" value="BB"></entry>
                <entry key="DDD">
                    <value>EEE</value>
                </entry>
            </map>
        </property>
    </bean>
```

## 4.spring注解版操作

开启注解版最重要的操作就是开启注解扫描。

```xml
<context:component-scan base-package="com.diaowenjie"/>
```

另外因为spring中存储对象是采用Map的方式进行，所以会有key：value，对应的key就是类的名称，即首字母小写，例如，User-->user。

### 4.1 用于创建对象的注解：  

他们的作用就是和在xml配置文件中编写一个<bean.标签实现的功能是一样的。

`Component`：  

**作用：**用于把当前类对象存入Spring容器中

**属性**：`value`用于指定bean的id。当我们不写时，它的默认值是当前类名，而且小字母改小写。

`Controller`：一般用在表现层。  

`Service`：一般用在业务层  

`Repository`:一般用在持久层  

以上三个注解他们的作用和属性与`Component`是一模一样的，他们三个是spring框架为我们提供明确的三层使用的注解，使我们的三层对象更加清晰。  

对于以上三个注解，因为spring容器中的对象是以Map的形式存储，所以对于实例化的对象名来说，其key如果以上注解不进行指定，则默认为类名的首字母小写。但是也可以通过以下方式自定义对象名称。

```java
@Service("accountService")
@Service(value = "accountservice")
```

### 4.2 用于注入数据的：

他们作用就是和在xml配置文件中的bean便签中写一个\<property>便签的作用是一样的。  

`Autowired`：  

**作用：**自动按照**类型进行注入**，只要容器中有唯一的一个**bean对象类型和要注入的变量类型匹配**，就可以注入成功。如果ioc容器中没有任何bean的类型和要注入的变量类型匹配，则报错。如果多个在按照对象名称（key String）进行匹配。

**出现位置**：可以是在变量上，也可以是在方法上。  

在使用注解注入时，set方法就不是必须的的了。

实例解析。

```java
@Component("accountService")
public class AccountServiceImpl implements IAccountService {
	
    //依赖注入  对于这个注解来说，不需要添加set方法。
    @Autowired
    private IAccountDao accountDao;

    public void saveAccount() {
        System.out.println("service 中的saveAccount方法被调用了。。。");
        accountDao.saveAccount();
    }
}

//注入的是实现类，IAccountDao是一个接口。
@Component("accountDao")
public class AccountDaoImpl implements IAccountDao {

    public void saveAccount() {
        System.out.println("保存了账户。");
    }
}

//进行测试类，然后便能够直接的输出。
public class Client {
    public static void main(String[] args) {
//        使用spring 进行创建对象。
        //获取核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
        //根据id获取bean对象
        IAccountService as = (IAccountService) ac.getBean("accountService");
        //实现的两种方式。
        as.saveAccount();
    }
}
```

当只有唯一的一个**bean对象类型和要注入的变量类型匹配**时，如下所示。以上总共有两个对象在容器中。

```java
@Autowired
private IAccountDao accountDao = null;
```

|   key String   |                        value Object                         |
| :------------: | :---------------------------------------------------------: |
| accountService | class **AccountServiceImpl** implements **IAccountService** |
|   accountDao   |     class **AccountDaoImpl** implements **IAccountDao**     |

通过查以上的表格找到**IAccountDao**便能够注入成功。  

如果是该接口有两个实现类的话，如下所示：

|   key String   |                        value Object                         |
| :------------: | :---------------------------------------------------------: |
| accountService | class **AccountServiceImpl** implements **IAccountService** |
|  accountDao1   |    class **AccountDaoImpl1** implements **IAccountDao**     |
|  accountDao2   |    class **AccountDaoImpl2** implements **IAccountDao**     |

在注入的时候ioc会先匹配Object中的实现类对象，发现有两个，然后在进行配置key String中的key值能不能匹配成功，如果匹配成功则注入成功。代码如下所示

```java
// 匹配成功。此时在key String中匹配到对应的 key accountDao1。
@Autowired
private IAccountDao accountDao1 = null;

//匹配失败，因为在key中找不到名称为accountDao的对象。
@Autowired
private IAccountDao accountDao = null;
```

以上总结为：在进行类型注入时，会先根据类名称进行匹配，如果只有唯一一个则成功，如果有两个再进行对象名称进行匹配匹配。这种情形多是针对接口实现类的情形。

**更好的解决方案：**

`Qualifier`注解：  

作用：在按照类中注入的基础上在按照名称注入，它在给类成员注入时不能单独使用，但是在给方法参数注入时可以。  

属性：value：用于指定注入的bean的id。

```java
//使用@Qualifier注解指定注入的对象。这样便不需要设置变量名称与ioc容器中的名称一致。
@Autowired
@Qualifier("accountDao1") //必须配合使用 (类成员)。
private IAccountDao accountDao = null;
```

`Resource`作用：可以直接按照bean的id注入，可以独立的使用。

属性：name：用于指定bean的id

```java
@Resource(name = "accountDao1")//单独使用。
private IAccountDao accountDao = null;
```

**以上三个注解都只能注入其他bean类型的数据，基本数据类型和String类型无法使用上述注解实现。另外集合类型的注入只能通过xml实现。**

`value`：作用：用于注入基本类型和string类型的数据。  

属性：value用于指定数据的类型，它可以使用spring中的SpEL（也就是spring的le表达式）写法：${表达式}

### 4.3 用于改变作用范围的：

他们的作用就和在bean便签上使用scope属性实现的功能是一样的。  

`scope`作用：用于指定bean的作用范围。  

**属性**：value；指定范围的取值，常用取值：singleton（默认 单列）  prototype（多例）

## 5.结合spring配置的方式进行的增删改查

### 5.1基本项目结构

首先是整体的文件，使用三层架构进行测试，其中：  

dao层负责数据持久化：  

```java
public class AccountDaoImpl implements IAccountDao {

    private QueryRunner runner;
	
    public void setRunner(QueryRunner runner) {
        this.runner = runner;
    }
    public List<Account> findAllAccount() {
        try {
            return runner.query("select * from account",new BeanListHandler<Account>(Account.class));
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

service 负责调用dao:  

```java
public class AccountServiceImpl implements IAccountService {

    private IAccountDao iAccountDao;
    public void setiAccountDao(IAccountDao iAccountDao) {
        this.iAccountDao = iAccountDao;
    }
    public List<Account> findAllAccount() {
        return iAccountDao.findAllAccount();
    }
}
```

### 5.2 纯xml配置版

spring配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置service，需要依赖Dao对象-->
    <bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl">
        <!--注入dao对象,对应类有个set方法-->
        <property name="iAccountDao" ref="accountDao"/>
    </bean>

    <!--配置Dao对象，需要依赖QueryRunner对象-->
    <bean id="accountDao" class="com.diaowenjie.dao.impl.AccountDaoImpl">
        <!--注入runner对象,对应类有个set方法-->
        <property name="runner" ref="runner"/>
    </bean>
    <!--配置QueryRunner，需要依赖数据源对象--->
    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <constructor-arg name="ds" ref="dataSource"/>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--注入连接数据库的必备信息-->
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/study_test"/>
        <property name="user" value="root"/>
        <property name="password" value="A123456"/>
    </bean>
</beans>
```

对应的测试代码：

```java
@Test
public void testFindAll() {
    //获取容器。
    ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
    //从容器中获取需要的对象
    IAccountService accountService = ac.getBean("accountService",IAccountService.class);
    //执行方法
    List<Account> allAccount = accountService.findAllAccount();

    for (Account account:allAccount){
        System.out.println(account);
    }
}
```

### 5.3 注解配置版

对于service.java

```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {
	//此时不用写setter方法。
    @Autowired
    private IAccountDao iAccountDao;

    public List<Account> findAllAccount() {
        return iAccountDao.findAllAccount();
    }
}
```

dao.java

```java
//spring 提供的三层注入结构
@Repository("accountDao")
public class AccountDaoImpl implements IAccountDao {
	
    @Autowired
    private QueryRunner runner;

    public List<Account> findAllAccount() {
        try {
            return runner.query("select * from account",new BeanListHandler<Account>(Account.class));
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

spring配置文件: 出现注解的配置文件并存的局面。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解扫描-->
    <context:component-scan base-package="com.diaowenjie"/>
    
    <!--配置QueryRunner-->
    <bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
        <constructor-arg name="ds" ref="dataSource"/>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--注入连接数据库的必备信息-->
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/study_test"/>
        <property name="user" value="root"/>
        <property name="password" value="A123456"/>
    </bean>
</beans>
```

## 6.使用java配置文件代替xml

其实xml可以说是一直简单的java文件翻版，即spring中使用xml简化java文件的书写。这样也就是说可以完全使用java文件进行配置spring项目。  

首先是配置常用的注解：  

`Configuration`：作用：指定当前类是一个配置类。  

`ComponentScan`：作用：泳衣通过注解指定spring在创建容器时要扫描的包。  

**属性：**value：他和basePackages 的作用是一样的，都是用于指定创建容器时要扫描的包，我们使用其就相当于在xml中写了：

```xml
 <context:component-scan base-package="com.diaowenjie"/>
```

`bean`：作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中。  

**属性：**name：用于指定bean的id，当不写时，默认值就是当前方法的名称。  

**细节：**当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象。查找的方式和autowired注解的作用是一样的。  

对应的代码说明：

```java
/**
 * Describe: Spring配置文件类
 *
 * @author Seven on 2020/4/15
 */
@Configuration
@ComponentScan("com.diaowenjie")
public class SpringConfiguration {

    //需要设置为多例的
    @Bean(name = "runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(DataSource dataSource){
        return new QueryRunner(dataSource);
    }

    //创造数据源对象,即往容器中添加对象。
    @Bean(name = "dataSource")
    public DataSource createDataSource(){
        ComboPooledDataSource ds = new ComboPooledDataSource();
        try {
            ds.setDriverClass("com.mysql.jdbc.Driver");
            ds.setJdbcUrl("jdbc:mysql://localhost:3306/study_test");
            ds.setUser("root");
            ds.setPassword("A123456");
        } catch (PropertyVetoException e) {
            e.printStackTrace();
        }

        return ds;
    }
}
```

从上面可以看出来，xml与java代码其实是一一对应的。然后是测试类。

```java
 @Test
public void testFindAll() {
    //获取容器。
    //ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
    //现在需要通过读取配置类的方式进行新建spring 容器。
    ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
    //从容器中获取需要的对象
    IAccountService accountService = ac.getBean("accountService",IAccountService.class);
    //执行方法
    List<Account> allAccount = accountService.findAllAccount();

    for (Account account:allAccount){
        System.out.println(account);
    }
}
```

另外对于`@Configuration`注解来说，因为一个配置文件可以分开写，所有如果不是在`AnnotationConfigApplicationContext`直接指定的配置文件，那么要加载成为配置文件必须写上`@Configuration`注解，但是`AnnotationConfigApplicationContext`必须有一个主的配置加载类，例如`new AnnotationConfigApplicationContext（SpringConfig.class）`，此时`SpringConfig.class `可以不写@Configuration注解。还有另一个  方法`new AnnotationConfigApplicationContext（SpringConfig.class,SpringConfig2.class) `这样也可以不写`@Configuration`注解。

#### 配置文件的分解问题

另外配置文件也可以分开写，类似于项目分类一样。使用如下注解进行控制:  

`Import`：作用：用于导入其他配置类。  

**属性：**value：用于指定配置类的其他字节码，当我们使用Import注解以后，带有Import注解的类就成了父配置类，导入的就是子配置类。  

使用方法：只需要在new AnnotationConfigApplicationContext（SpringConfig.class）中的SpringConfig.class导入其他配置文件即可。  

`PropertySource`:作用：用于指定properties文件的位置。  

**属性：**value：指定文件的名称和路径，关键字；classpath，表示在类路径下。  

即使用properties注解可以自动一些属性通过读取配置文件进行。  

主配置文件

```java
//主配置文件
@Configuration
@ComponentScan("com.diaowenjie")
@Import(jdbcConfiguration.class)
@PropertySource("classpath:jdbcConfig.properties" ) 
public class SpringConfiguration {
    //主配置文件
}
```

子配置文件

```java
public class jdbcConfiguration {
	
    //通过读取properties文件中的值，进行赋值，实现可以更改。
    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    //需要设置为多例的
    @Bean(name = "runner")
    @Scope("prototype")
    public QueryRunner createQueryRunner(DataSource dataSource){
        return new QueryRunner(dataSource);
    }

    //创造数据源对象,即往容器中添加对象。
    @Bean(name = "dataSource")
    public DataSource createDataSource(){
        ComboPooledDataSource ds = new ComboPooledDataSource();
        try {
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
        } catch (PropertyVetoException e) {
            e.printStackTrace();
        }

        return ds;
    }
}
```

properties文件,将连接数据库的文件不写死，通过读取进行分开。

```properties
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/study_test
jdbc.username = root
jdbc.password = A123456
```

通过以上的方式可以感受到， 纯注解和纯xml的都是很费时，建议使用xml+注解的方式。其中使用别人的包的建议使用注解的方式，比较方便。

## 7.Junit测试，使用spring 容器的方式进行测试

#### 出现问题

即以上的可以知道，在进行测试的时候，必须指定加载spring配置文件，然后获取对于bean，冗余的代码很多，我们希望在进行测试的时候能够直接使用spring容器中的对象，即使用注解进行测试。删掉冗余的代码。

1. 应用程序的入口
   1. main方法
2. Junit单元测试中，没有main方法也能够执行
   1. junit集成了一个main方法。
   2. 该方法就会判断当前测试类中哪些方法有@Test注解
   3. junit就让有Test注解的方法执行
3. junit不会管我们是否采用spring框架
   1. 在执行测试方法时，junit根本不知道我们是不是使用了spring框架
   2. 所以也就不会为我们读取配置文件、配置类创建spring核心容器
4. 由以上三点可知
   1. 当测试方法执行时，没有ioc容器，就算写了Autowired注解，也无法实现注入。

#### 解决思路

将spring整合到junit的配置：

1. 导入spring整合junit的jar(坐标)
2. 使用junit提供的一个注解把原有的main方法替换掉，替换成spring提供的@Runwith
3. 告知spring的运行器，spring和ioc创建是基于xml的还是基于注解的，并说明位置
   1. @ContextConfiguration:	
      1. locations:指定xml文件的位置，加上classpath关键字，表示在类路径下。
      2. classes：指定注解类所在位置。

当我们使用spring 5.x版本是，要求junit的jar必须是4.12及以上。

#### 具体实现

pom文件中引入spring测试依赖

```xml
<!--单元测试-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
<!--使用spring 测试。-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>
```

更改测试类

```java
@RunWith(SpringJUnit4ClassRunner.class) //指定导入的spring测试类
@ContextConfiguration(classes = SpringConfiguration.class) //指定配置文件  使用配置类的方式。
@ContextConfiguration(location = "classpath:bean.xml") //指定配置文件  使用xml配置文件的方式。
public class AccountServiceTest {
	
    //直接实现自动注入进行
    @Autowired
    private IAccountService accountService;
    @Test
    public void testFindAll() {

        List<Account> allAccount = accountService.findAllAccount();

        for (Account account:allAccount){
            System.out.println(account);
        }
    }
}
```

