---
layout: post
title: Spring AOP 学习
tags: java  
---


> 记录spring AOP 的知识，配以简单的例子进行说明。

## 目录
[toc]
## 1. 对于数据库的事务分析

```java
@Override
public void transfer(String sourceName, String targetName, Float money) {
    //1.根据名称查询转出账户
    Account source = iAccountDao.findAccountByName(sourceName);
    //2.根据名称查询注入账户
    Account target = iAccountDao.findAccountByName(targetName);
    //3.转出账户减钱
    source.setMoney(source.getMoney() - money);
    //4.转入账户加钱
    target.setMoney(target.getMoney() + money);
    //5.更新转出账户
    iAccountDao.updateAccount(source);
    //6.更新转入账户
    iAccountDao.updateAccount(target);
}
```

对于以上的一个转钱操作，如果是正常情况下，一步步的走是完全没有问题的，但是如果在第5步以后出现异常比如加上`int i=1/0;`，那么就会出现钱已经扣掉但是对方账户没有收到钱的情形。出现的原因就是，所有的操作是没有关系的，每个都是单独操作而不是一个整体，我们需要的就是把他们变成一个整体。

解决方案：

所以需要`TheadLocal`对象，把`Connection`和当前线程绑定，从而使得一个线程中只有一个能够控制事务的对象。

**主要就是借助java的事务管理，将多个事务整合成一个原子性操作。**

需要借助两个工具类`ConnectionUtils.class`和`TransactionManager.class`定义事务操作。

```java
/**
 * Describe: 连接的工具类，从数据源汇总获取一个连接，并且实现和线程的绑定。 使用的是jdbc事务概念，即通过事务将多个操作
 * 绑定在一起，实现原子性操作。
 *
 * @author Seven on 2020/4/20
 */
public class ConnectionUtils {
    private final ThreadLocal<Connection> t1 = new ThreadLocal<Connection>();

    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    /**
     * 获取当前线程的连接
     * @return
     */
    public Connection getThreadConnection(){
        try {
            //1.先从ThreadLocal上获取
            Connection conn = t1.get();
            //2.判断当前线程是否有连接
            if (conn == null){
                //3.从数据源中获取一个连接，并且存入到ThreadLocal中。
                conn = dataSource.getConnection();
                t1.set(conn);
                //4. 返回当前线程上的连接
            }
            return conn;
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return  null;
    }

    /**
     * 把连接和线程解绑
     */
    public void removeConnection(){
        t1.remove();
    }
}
```

```java
/**
 * Describe: 和事务管理的工具类，包含开启事务，提交事务，回滚事务和释放连接。
 *
 * @author Seven on 2020/4/20
 */
public class TransactionManager {

    private ConnectionUtils connectionUtils;
    public void setConnectionUtils(ConnectionUtils connectionUtils) {
        this.connectionUtils = connectionUtils;
    }
    /**
     * 开启事务
     */
    public void beginTransaction(){
        try {
            //关闭自动事务提交
            connectionUtils.getThreadConnection().setAutoCommit(false);
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 提交事务
     */
    public void commit(){
        try {
            connectionUtils.getThreadConnection().commit();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    /**
     * 回滚事务
     */
    public void rollback(){
        try {
            connectionUtils.getThreadConnection().rollback();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 释放连接
     */
    public void release(){
        try {
            connectionUtils.getThreadConnection().close(); //还回了连接池中
            connectionUtils.removeConnection();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
}
```

以改造交易事务为例：

```java
/**
 * Describe: 账户的业务层实现类
 *
 * @author Seven on 2020/4/14
 */
public class AccountServiceImpl implements IAccountService {
	//使用spring注入。
    private IAccountDao iAccountDao;
	public void setiAccountDao(IAccountDao iAccountDao) {
        this.iAccountDao = iAccountDao;
    }
    
    private TransactionManager txManager;
    public void setTxManager(TransactionManager txManager) {
        this.txManager = txManager;
    }

    /**
     * 转账操作
     * @param sourceName 源账户
     * @param targetName 目的账户
     * @param money 转账金额
     */
    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        try {
            //1.开启事务
            txManager.beginTransaction();
            //2.执行操作
                //1.根据名称查询转出账户
                Account source = iAccountDao.findAccountByName(sourceName);
                //2.根据名称查询注入账户
                Account target = iAccountDao.findAccountByName(targetName);
                //3.转出账户减钱
                source.setMoney(source.getMoney() - money);
                //4.转入账户加钱
                target.setMoney(target.getMoney() + money);
                //5.更新转出账户
                iAccountDao.updateAccount(source);
                int i = 1/0;
                //6.更新转入账户
                iAccountDao.updateAccount(target);
            //3.提交事务
            txManager.commit();

        }catch (Exception e){
            //4.回滚操作
            txManager.rollback();
            e.printStackTrace();
        }finally {
            //5.释放连接
            txManager.release();
        }
    }
}
```

对应的dao操作也需要进行改造，主要是要在sql中加上`connectionUtils.getThreadConnection()`将多个操作绑定成同一个事务，实现同步同步操作。

```java
// Describe: 账户的持久层实现类
public class AccountDaoImpl implements IAccountDao {

    private QueryRunner runner;
   	public void setRunner(QueryRunner runner) {
        this.runner = runner;
    }
    //需要使用连接工具类。
    private ConnectionUtils connectionUtils;
    public void setConnectionUtils(ConnectionUtils connectionUtils) {
        this.connectionUtils = connectionUtils;
    }
    
    public Account findAccountById(Integer accountId) {
        try {
            //必须加上connectionUtils.getThreadConnection()，将其绑定在一起
            return runner.query(connectionUtils.getThreadConnection(),"select * from account where id = ?",new BeanHandler<Account>(Account.class),accountId);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

spring配置，将需要的bean注入进去。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置service-->
    <bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl">
        <!--注入runner-->
        <property name="iAccountDao" ref="accountDao"></property>
        <!--注入事务管理器-->
        <property name="txManager" ref="txManager"/>
    </bean>

    <!--配置dao对象-->
    <bean id="accountDao" class="com.diaowenjie.dao.impl.AccountDaoImpl">
        <property name="runner" ref="runner"></property>
        <property name="connectionUtils" ref="connectionUtils"/>
    </bean>

    <!--配置connectionUtil 工具类-->
    <bean id="connectionUtils" class="com.diaowenjie.utils.ConnectionUtils">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
	<!--runner 和 DataSource 省略不表 主要就是配置数据库连接，在ioc文章中有说。-->
    <!--配置事务管理器-->
    <bean id="txManager" class="com.diaowenjie.utils.TransactionManager">
        <property name="connectionUtils" ref="connectionUtils"/>
    </bean>
</beans>
```

经过以上的配置便能够实现事务的原子性操作。

## 2.动态代理

### 2.1基于接口的动态代理实现方法

特点：字节码随用随创建，随用随加载

作用: 不修改源码的基础上对方法增强，例如增加log，其实最主要的思想就是抽取公共代码，让代码更加的简洁容易维护。

1. 分类:
   1. 基于接口的动态代理
   2. 基于子类的动态代理
2. 基于接口的动态代理：
   1. 涉及的类：proxy
   2. 提供者；JDK
3. 如何创建代理对象：使用proxy类中的`newProxyInstance`方法
4. 创建代理对象的要求：**被代理类最少要实现一个接口，如果没有则不能使用**
5. `newProxyInstance`方法的参数：
   1. `ClassLoader`:类加载器，他是用于加载代理对象字节码的，和被代理对象使用相同的类加载器，固定写
   2. Class[ ]:自己码数组，他是用于让代理对象和被代理对象有相同的方法，固定写法
   3. `InvocationHandler`:用于提供增强的代码，让是让我们写如何代理，我们一般都是写一个该接口的实现类，通常情况下都是匿名类，但是不是必须的。此接口的实现类都是谁用谁写。

基于生产者的代理实现，总共有三个类，一个生产者类进行生产产品，一个经销商接口，一个用户类。

1.Producer.class

```java
public class Producer implements IProducer{
    public void  saleProduct(float money){
        System.out.println("销售产品" + money);
    }

    public void afterService(float money){
        System.out.println("提供售后" + money);
    }
}
```

2.IProducer.class 经销商

```java
public interface IProducer {
    public void  saleProduct(float money);

    public void afterService(float money);
}
```

用户测试类。

```java
/**
 * Describe: 使用接口代理类进行增强方法。
 *
 * @author Seven on 2020/4/20
 */
public class Client {
    public static void main(String[] args) {

        final Producer producer = new Producer();

        //基于接口实现的动态代理。 可以实现增强接口方法。类似于经销商半路拦截下20%的钱，
        IProducer proxyInstance = (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),
                producer.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     *  作用：被执行代理对象的任何接口方法都会讲过该方法
                     * @param proxy   代理对象的引用
                     * @param method  当前执行方法
                     * @param args    当前执行方法所需的参数
                     * @return        和被代理对象方法有相同的返回值
                     * @throws Throwable
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //提供增强的代码,即在原来的代码基础上 增强代码
                        Object returnValue = null;

                        //1.获取方法执行的参数
                        Float money = (Float) args[0]; //因为方法参数只有一个
                        //2.判断当前方法是不是销售
                        if ("saleProduct".equals(method.getName())){
                            Object[] args1;
                            returnValue = method.invoke(producer, 0.8f * money);
                        }
                        return returnValue;
                    }
                }
        );

        proxyInstance.saleProduct(1000f);
    }
}
```

运行结果：

```shell
销售产品800
```

即相当于经销商代理半路抽取了20%的手续费。即对方法实现再包装。

### 2.2 基于类的动态代理

这个就需要借助第三方的jar包来实现。总体的实现方法与基于接口的实现类似。

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

特点：字节码随用随创建，随用随加载

作用: 不修改源码的基础上对方法增强

1. 分类；
   1. 基于接口的动态代理
   2. 基于子类的动态代理
2. 基于接口的动态代理：
   1. 涉及的类：Enhancer
   2. 涉及的类：Enhancer
3. 如何创建代理对象：使用Enhancer类中的create方法
4. 创建代理对象的要求：被代理类不能是最终类。
5. Create方法的参数：
   1. Class字节码，他是用于指定被被代理对象的字节码
   2. CallBack:用于提供增强的代码。他是让是让我们写如何代理，我们一般都是写一个该接口的实现类，通常情况下都是匿名类，但是不是必须的。此接口的实现类都是谁用谁写。我们一般写的都是该接口的子接口实现类：`MethodInterceptor`

具体的例子。

```java
//此时Producer不需要实现接口。
public class Client {
    public static void main(String[] args) {

        final Producer producer = new Producer();
        //基于接口实现的动态代理。 可以实现增强接口方法。
        Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor() {
            /**
             * 执行被代理对象的任何方法都会经过该方法。
             * @param o 代理对象的引用
             * @param method
             * @param objects
             *   以上三个参数和基于接口的动态代理中invoke方法的参数是一样的
             * @param methodProxy 当前执行方法的代理对象
             * @return
             * @throws Throwable
             */
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                //提供增强的代码,即在原来的代码基础上 增强代码
                Object returnValue = null;
                //1.获取方法执行的参数
                Float money = (Float) objects[0]; //因为方法参数只有一个
                //2.判断当前方法是不是销售
                if ("saleProduct".equals(method.getName())) {
                    returnValue = method.invoke(producer, 0.8f * money);
                }
                return returnValue;
            }
        });
        cglibProducer.saleProduct(1000f);
    }
}
```

主要就是写增强方法。运行结果也是经销商抽取了20%的手续费。

## 3.基于动态代理实现事务原子性

使用动态代理技术实现标题1中的事务，即抽取事务绑定代码，然后使用动态代理为每个方法加上事务。 

`BeanFactory.class`对比于事务分析中的原子性操作，使用以下方法进行为service层中每一个方法加上事务。因为是调用类中的方法， 所以当直接给一个类进行增加事务操作时，也相当于为类中每一个方法加上事务？相当于代理商为每个数据库操作加了一个保险。

```java
public class BeanFactory {

    private IAccountService accountService;
    public final void setAccountService(IAccountService accountService) {
        this.accountService = accountService;
    }

    private TransactionManager txManager;
    public void setTxManager(TransactionManager txManager) {
        this.txManager = txManager;
    }

    /**
     *  获取service的代理类,已经增加上了事务处理
     * @return
     */
    public IAccountService getAccountService(){

        return (IAccountService) Proxy.newProxyInstance(accountService.getClass().getClassLoader(),
                accountService.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     * 添加事务的支持。
                     * @param proxy
                     * @param method
                     * @param args
                     * @return
                     * @throws Throwable
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

                        Object rtValue = null;
                        try {
                            //1.开启事务
                            txManager.beginTransaction();
                            //2.执行操作
                            rtValue = method.invoke(accountService, args);
                            //3.提交事务
                            txManager.commit();
                            //4.返回结果
                            return rtValue;
                        } catch (Exception e) {
                            //5.回滚操作
                            txManager.rollback();
                        } finally {
                            //6.释放连接
                            txManager.release();
                        }
                        return null;
                    }
                });
    }
}
```

此时也需要更改xml中的配置，需要将工厂类增加到容器中去，并且为工厂类注入需要改造的正常service，工厂类返回改造后的增强proxyAccountService 

```xml
<!-- 配置beanFactory-->
<bean id="beanFactory" class="com.diaowenjie.factory.BeanFactory">
    <property name="txManager" ref="txManager"/>
    <!--注入service 需要将正常的service注入到工厂类去改造。-->
    <property name="accountService" ref="accountService"/>
</bean>

<!-- 配置工厂类service 此service为增强后的service-->
<bean id="proxyAccountService" factory-bean="beanFactory" factory-method="getAccountService"/>
<!--    因为有两个所有在测试类中就需要进行更改。-->

<!--配置service-->
<!--这个service是正常的service-->
<bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl">
    <!--注入runner-->
    <property name="iAccountDao" ref="accountDao"></property>
</bean>
<!-- 配置Dao不在赘述-->
```

此时测试方法中也需要更改对应的方法

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:bean.xml")
public class AccountServiceTest {

    @Autowired 
    @Qualifier("proxyAccountService") //因为有两个service类，所以需要指定。 指定调用增强后的service对象，
    private IAccountService accountService;

    @Test
    public void testTransFer(){
        accountService.transfer("aaa","bbb",200f);
    }
}
```

对应的，因为采用了动态代理的形式所以第一部分中的事务处理代码也要删除，不表。  

[深入理解动态代理](https://www.jianshu.com/p/cbd58642fc08)  其实也就是为类进行增强方法， 通过调用代理类，代理类为目标类进行修饰，代理类就是在调用目标类之前将目标类进行了修饰。

## 4.AOP的相关概念

AOP：Aspect oriented progarmming：即面向切面编程。  

简单的说它就是把我们程序中重复的代码抽取出来，在需要执行的时候，使用动态代理技术，在不修改源码的基础上，对我们的已有方法进行增强。  

作用：在程序运行期间，不修改源码对已有的方法进行增强。

其实aop也就是通过注解的方法实现前面所说的动态代理，来实现简化操作，即通过aop实现上一节的操作。

**基于子类的动态代理要求被代理类不能是`final`修饰的。**

### 4.1 aop相关术语

**Jointpoint(连接点)：**所谓连接点就是指那些被拦截到的点，在spring中这些点指的就是方法，因为spring只支持方法类型的连接点。所有的方法都是连接点，但不一定都是切入点。  

**Pointcut(切入点)：**所谓切入点就指我们要对哪些Jointpoint进行拦截的定义。并不是类中所有的方法都要进行增强，需要增强的方法就是切入点。  

**Advice(通知/增强)：**所谓通知就是指拦截到Jointpoint之后要做的事情就是通知。通知的类型：前置通知，后置通知，异常通知，最终通知，环绕通知。(根据切入点方法前后进行区分)。  

**Introduction(引介)：**引介是一种特殊的通知在不修改类代码的前提下，introducton可以运行期为类动态的增加一些方法或field。    

**Target(目标对象)：**代理的目标对象。    

**weaving(织入)：**是指把增强应用到目标对象来创建新的代理对象的过程。  spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入。  

**Proxy(代理)：**一个类被aop织入增强后，就产生一个结果代理类。    

**Aspect(切面):**是切入点和通知(引介)的结合。    

### 4.2 实现AOP的一个例子

该例子通过配置AOP然后实现在运行目标函数方法之前，增加上日志功能。为什么总是引用日志的功能？因为日志的功能是最容易说明的。  

引用依赖：  

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>
```

service实现类测试方法  

```java
public class AccountServiceImpl implements IAccountService {

    @Override
    public void saveAccount() {
        System.out.println("执行保存");
    }

    @Override
    public void updateAccount(int i) {
        System.out.println("执行更新" + i);
    }

    @Override
    public int deleteAccount() {
        System.out.println("执行删除");
        return 0;
    }
}
```

logger工具类 ，需要增强的方法

```java
public class Logger {
    /**
     *用于打印日志，计划让其在切入点方法执行之前执行，(切入点方法就是业务层方法)
     */
    public void printLog(){
        System.out.println("Logger类中的方法printLog开始记录日志");
    }
}
```

目标就是在通过配置xml实现在调用`AccountServiceImpl`类中的方式时，会先调用`printLog`方法。

**Spring中基于xml的配置aop说明**  

1. 把通知`bean`也交给spring来管理
2.  使用`aop:config`标签表明开始aop配置
3. 使用`aop：aspect`标签表明配置切面
   1. id属性：是给切面提供一个唯一标识
   2. ref属性:是指定通知类bean的id 

4. 在`aop:aspect`标签的内部使用对应标签来配置通知的类型
   1.  我们现在实例是让`printLog`方法在切入点方法发执行之前执行，所以是前置通知。

   2.  `aop:before `表示配置前置通知。    
   3. ` method：before` 表示配置前置通知           
   4. ` method`属性：用于指定logger类中哪个方法是前置通知           
   5. `pointcut`属性：用于指定切入点表达式，该表达式的含义指的是对业务层中哪些方法增强，

5. 切入点表达式的写法：
   1. 关键字:` execution`(表达式)

   2. 表达式:访问修饰符 `返回值 包名.包名.包名...类名.方法名(参数列表)`
   3. 标准的表达式写法：` public void com.diaowenjie.service.impl.AccountServiceImpl.saveAccount()` 
   4. 访问修饰符(public)可以省略：`void com.diaowenjie.service.impl.AccountServiceImpl.saveAccount()`
   5. 返回值可以使用通配符\*，表示任意返回值：`* com.diaowenjie.service.impl.AccountServiceImpl.saveAccount()`
   6. 包名可以使用通配符，表示任意包，但是有几级包，就需要写几个`*. ` ：`* *.*.*.*.AccountServiceImpl.saveAccount()`
   7. 包名可以使用\..表示当前包及其子包：`* *..AccountServiceImpl.saveAccount()`
   8. 类名和方法名都可以使用\*来进行通配：`* *..*.*()`但是()表示无参。
   9. 参数列表：
      1. 可以直接写数据类型：
         1. 基本类型直接写名称 : e.g.  `int`
         2. 引用类型写包名.类名的方式 ` java.lang.String`
      2. 可以使用通配符表示任意类型，但是必须有参数(*)
      3. 可以使用\..表示有无参数均可，有参数可以是任意类型。
   10. 全通配写法：`* *..*.*(..)`所有的都会被匹配。尽量不要写。
   11. 实际开发中切入点表达式的通常写法：切到业务层实现类下的所有方法，`* com.diaowenjie.service.impl.*.*(..)`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--aop的命名约束-->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 配置spring的 ioc 吧service对象配置进来。 -->
    <bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl"></bean>
	<!--配置Logger类-->
    <bean id="logger" class="com.diaowenjie.utils.Logger"></bean>

	<!--配置aop  -->
    <aop:config>
		<!--配置切面-->
        <aop:aspect id = "logAdvice" ref="logger">
			<!--配置通知的类型，并且建立通知方法和切入点方法的关联-->
            <aop:before method="printLog" pointcut="execution(public void com.diaowenjie.service.impl.AccountServiceImpl.saveAccount())">	</aop:before>
        </aop:aspect>
    </aop:config>
</beans>
```

测试方法类

```java
public static void main(String[] args) {
    ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
    IAccountService as = (IAccountService) ac.getBean("accountService");
    as.saveAccount();
    as.deleteAccount();
    as.updateAccount(6);
}
```

通过运行改方法系统便会输出：

```shell
Logger类中的方法printLog开始记录日志
执行保存
执行删除
执行更新6
```

一般建议写法为通配符的写法增强所有的方法,如下所示。

```xml
<aop:aspect id = "logAdvice" ref="logger">
    <!--配置通知的类型，并且建立通知方法和切入点方法的关联-->
    <aop:before method="printLog" pointcut="execution(* com.diaowenjie.service.impl.*.*(..))"></aop:before>
</aop:aspect>
```

```shell
Logger类中的方法printLog开始记录日志
执行保存
Logger类中的方法printLog开始记录日志
执行删除
Logger类中的方法printLog开始记录日志
执行更新6
```

### 4.3 AOP的其他配置

配置前置、后置、异常、最终通知。首先在logger中写出对应的方法

```java
public class Logger {

    /**
     *前置通知
     */
    public void BeforePrintLog(){
        System.out.println("前置通知Logger类中的方法printLog开始记录日志");
    }
    /**
     *后置通知
     */
    public void AfterReturnPrintLog(){
        System.out.println("后置通知Logger类中的方法printLog开始记录日志");
    }

    /**
     *异常通知
     */
    public void AfterThrowingPrintLog(){
        System.out.println("异常通知Logger类中的方法printLog开始记录日志");
    }

    /**
     *最终通知
     */
    public void AfterPrintLog(){
        System.out.println("最终通知Logger类中的方法printLog开始记录日志");
    }
}
```

对应的xml中的配置。主要是aop中的变化。其他部分不变。

```xml
<!--配置aop  -->
<aop:config>
    <!--配置切面-->
    <aop:aspect id = "logAdvice" ref="logger">
        <!--前置通知： -->
        <aop:before method="BeforePrintLog" pointcut="execution(* com.diaowenjie.service.impl.*.*(..))"></aop:before>
        <!--后置通知-->
        <aop:after-returning method="AfterReturnPrintLog" pointcut="execution(* com.diaowenjie.service.impl.*.*(..))"></aop:after-returning>
        <!--异常通知-->
        <aop:after-throwing method="AfterThrowingPrintLog" pointcut="execution(* com.diaowenjie.service.impl.*.*(..))"></aop:after-throwing>
        <!--最终通知-->
        <aop:after method="AfterPrintLog" pointcut="execution(* com.diaowenjie.service.impl.*.*(..))"></aop:after>
    </aop:aspect>
</aop:config>
```

调用测试类方法以后运行结果；

```shell
前置通知Logger类中的方法printLog开始记录日志
执行保存
后置通知Logger类中的方法printLog开始记录日志
最终通知Logger类中的方法printLog开始记录日志
```

如果有异常，则会调用异常通知，并且后置通知不会执行，因为程序异常退出。但是最终通知会进行执行。  

对于以上的配置可以发现，其表达式是一样的，因此可以使用`<aop:pointcut>`标签进行替换。

```xml
<aop:aspect id = "logAdvice" ref="logger">
    <!--前置通知-->
    <aop:before method="BeforePrintLog" pointcut-ref="pt1"></aop:before>
    <!--后置通知-->
    <aop:after-returning method="AfterReturnPrintLog" pointcut-ref="pt1"></aop:after-returning>
    <!--异常通知-->
    <aop:after-throwing method="AfterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>
    <!--最终通知-->
    <aop:after method="AfterPrintLog" pointcut-ref="pt1"></aop:after>

    <aop:pointcut id="pt1" expression="execution(* com.diaowenjie.service.impl.*.*(..))"/>
</aop:aspect>
```

但是对于配置在`</aop:aspect>`标签内部的只能本标签中使用，要想全局使用，需要将其配置在`<aop:config>`标签下。

```xml
<!--配置aop  -->
<aop:config>
     <!--需要配置在前面才能正常使用。-->
    <aop:pointcut id="pt1" expression="execution(* com.diaowenjie.service.impl.*.*(..))"/>
    <!--配置切面-->
    <aop:aspect id = "logAdvice" ref="logger">
        <!--前置通知-->
        <aop:before method="BeforePrintLog" pointcut-ref="pt1"></aop:before>
        <!--后置通知-->
        <aop:after-returning method="AfterReturnPrintLog" pointcut-ref="pt1"></aop:after-returning>
        <!--异常通知-->
        <aop:after-throwing method="AfterThrowingPrintLog" pointcut-ref="pt1"></aop:after-throwing>
        <!--最终通知-->
        <aop:after method="AfterPrintLog" pointcut-ref="pt1"></aop:after>
    </aop:aspect>
</aop:config>
```

**spring中的环绕通知**  
**问题：** 当我们配置了环绕通知以后，切入点方法没有执行，而通知方法执行了  
**分析：** 通过对比动态代理中的环绕通知代码，发现动态代理的环绕通知有明确的切入点方法调用，而我们的代码没有。  
**解决：** Spring框架为我们提供了一个接口，`ProceedingJoinPoint` 该接口有一个方法`proceed()`此方法就相当于明确调用切入点。该接口可以作为环绕通知的方法参数，当程序执行时，spring框架会为我们提供该接口的实现类供我们使用。  
spring中的环绕通知：它是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式。  

首先是xml中的配置:

```xml
<aop:config>
    <aop:pointcut id="pt1" expression="execution(* com.diaowenjie.service.impl.*.*(..))"/>
    <!--配置切面-->
    <aop:aspect id = "logAdvice" ref="logger">
        <!--配置环绕配置 -->
        <aop:around method="aroundPrintLog" pointcut-ref="pt1"/>
    </aop:aspect>
</aop:config>
```

```java
public Object aroundPrintLog(ProceedingJoinPoint pjp){
    Object[] args = pjp.getArgs();
    Object rtValue = null;
    try {
        System.out.println("Logger类中的方法printLog开始记录日志...前置");
        
        rtValue= pjp.proceed(args); //目标调用方法。
        
        System.out.println("Logger类中的方法printLog开始记录日志...后置");
        return rtValue;
    } catch (Throwable throwable) {
        System.out.println("Logger类中的方法printLog开始记录日志...异常");
        throwable.printStackTrace();
    }finally {
        System.out.println("Logger类中的方法printLog开始记录日志...最终");
        return null;
    }
}
```

通过一个环绕配置，可以实现之前所有的配置。相当于代码基本的实现。也即是可以通过代码进行直接的控制，而不是通过配置版进行控制。

## 5.基于注解版的AOP

配置spring创建时扫描的包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置spring创建时要扫描的包-->
    <context:component-scan base-package="com.diaowenjie"></context:component-scan>
    <!--配置Spring开启注解AOP的支持-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

在Service类中加上注解：

```java
@Service("accountService")
public class AccountServiceImpl implements IAccountService {

    @Override
    public void saveAccount() {
        System.out.println("执行保存");
    }

    @Override
    public void updateAccount(int i) {
        System.out.println("执行更新" + i);
    }

    @Override
    public int deleteAccount() {
        System.out.println("执行删除");
        return 0;
    }
}
```

Logger类上加上注解

```java
@Component("logger")
@Aspect //表示当前类是个切面
public class Logger {

    @Pointcut("execution(* com.diaowenjie.service.impl.*.*(..))")
    private void pt1(){}
    /**
     *前置通知
     */
    @Before("pt1()")
    public void BeforePrintLog(){
        System.out.println("前置通知Logger类中的方法printLog开始记录日志");
    }
    /**
     *后置通知
     */
    @AfterReturning("pt1()")
    public void AfterReturnPrintLog(){
        System.out.println("后置通知Logger类中的方法printLog开始记录日志");
    }

    /**
     *异常通知
     */
    @AfterThrowing("pt1()")
    public void AfterThrowingPrintLog(){
        System.out.println("异常通知Logger类中的方法printLog开始记录日志");
    }

    /**
     *最终通知
     */
    @After("pt1()")
    public void AfterPrintLog(){
        System.out.println("最终通知Logger类中的方法printLog开始记录日志");
    }

    /**
     * 环绕通知
     */
//    @Around("pt1()")  //因为不能同时存在。
    public Object aroundPrintLog(ProceedingJoinPoint pjp){
        Object[] args = pjp.getArgs();
        Object rtValue = null;
        try {
            System.out.println("Logger类中的方法printLog开始记录日志...前置");
            rtValue= pjp.proceed(args);
            System.out.println("Logger类中的方法printLog开始记录日志...后置");
            return rtValue;
        } catch (Throwable throwable) {
            System.out.println("Logger类中的方法printLog开始记录日志...异常");
            throwable.printStackTrace();
        }finally {
            System.out.println("Logger类中的方法printLog开始记录日志...最终");
            return null;
        }
    }
}
```

一个小问题，对于使用注解版的aop调用的方法上有点小问题：

```shell
前置通知Logger类中的方法printLog开始记录日志
执行保存
最终通知Logger类中的方法printLog开始记录日志
后置通知Logger类中的方法printLog开始记录日志
```

由以上可以看出，最终与后置的执行顺序是错的，这种是更改不了的，如果要想按照正常的顺序，建议使用环绕注解。基于代码的控制可以完整地控制其切面。

