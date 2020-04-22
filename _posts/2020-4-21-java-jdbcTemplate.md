---
layout: post
title: Spring:JdbcTemplate & Transaction
tags: java  
---


> 记录spring中的jdbcTemplate和事务控制学习记录

##  目录
* 目录
{:toc}
## 1.Spring JdbcTemplates

### 1.1  简单实现

spring 也提供了实现简单的数据库操作，对于小型项目可以实现很好的支持。创建过程:

bean.xml

```xml
<!--使用jdbcTemplate 进行数据操作。-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--配置数据源-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <!--注入连接数据库的必备信息-->
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/study_test"/>
    <property name="user" value="root"/>
    <property name="password" value="A123456"/>
</bean>
```

测试类，进行增删改查

```java
public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
    
        JdbcTemplate jdbcTemplate = ac.getBean("jdbcTemplate",JdbcTemplate.class);
        //保存
		//jdbcTemplate.update("insert  into account(name,money) values(?,?)","eee",333f);
        
    	//更新
		//jdbcTemplate.update("update account set name = ? ,money = ? where  id = ?","test",200f,3);
        
    	//删除
		//jdbcTemplate.update("delete from account where  id = ?",5);
        
    	//查询方法 查询多个
		//List<Account> query = jdbcTemplate.query("select * from account where money > ?", new BeanPropertyRowMapper<>(Account.class), 1000f);
		//for (Account account:query){
			//System.out.println(account);
		//}
        
    	//查询一个
		// List<Account> accounts = jdbcTemplate.query("select * from account where id = ?", new BeanPropertyRowMapper<>(Account.class), 1);
		//System.out.println(accounts.isEmpty()?"没有内容":accounts.get(0));

        //查询返回一行一列 (使用聚合函数，但是不加group by字句)
        Long count = jdbcTemplate.queryForObject("select count(*) from account where money > ?",Long.class,1000f);
        System.out.println(count);
    }
```

### 1.2 基于三层架构的实现

更改与之前的用户转账操作，使用jdbctemplates实现。

首先是AccountDaoImpl.class

```java
public class AccountDaoImpl implements IAccountDao {
	
    //注入JdbcTemplate实现数据库操作
    private JdbcTemplate jdbcTemplate;
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    //通过id查找用户
    public Account findAccountById(Integer accountId) {
       List<Account> accounts =  jdbcTemplate.query("select * from account where id = ?",new BeanPropertyRowMapper<>(Account.class),accountId);
       return accounts.isEmpty()? null : accounts.get(0);
    }

    //更新用户状态
    public void updateAccount(Account account) {
       jdbcTemplate.update("update account set name  =  ? ,money = ? where id = ?",account.getName(),account.getMoney(),account.getId());
    }

    //通过名字查找用户
    @Override
    public Account findAccountByName(String accountName) {
        List<Account> accounts =  jdbcTemplate.query("select * from account where name = ?",new BeanPropertyRowMapper<>(Account.class),accountName);
        if (accounts.isEmpty())
            return null;
        if (accounts.size() > 1)
            throw new RuntimeException("结果不唯一");
        return accounts.get(0);
    }
}
```

AccountServiceImpl.class类进行调用持久层。

```java
public class AccountServiceImpl implements IAccountService {
	
    //注入持久层
    private IAccountDao iAccountDao;
	public void setiAccountDao(IAccountDao iAccountDao) {
        this.iAccountDao = iAccountDao;
    }
    /**
     * 根据用户id查找用户
     * @param accountId
     * @return Account
     */
    public Account findAccountById(Integer accountId) {
        return findAccountById(accountId);
    }
	//根据名字查询
    @Override
    public Account findAccountByName(String name) {
        return iAccountDao.findAccountByName(name);
    }

    /**
     * 更新账号
     * @param account
     */
    public void updateAccount(Account account) {
        iAccountDao.updateAccount(account);
    }

    /**
     * 转账操作
     * @param sourceName 源账户
     * @param targetName 目的账户
     * @param money 转账金额
     */
    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        System.out.println("执行了..");
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
    }
}
```

bean.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--使用jdbcTemplate 进行数据操作。-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置业务层-->
    <bean id="accountService" class="com.diaowenjie.service.impl.AccountServiceImpl">
        <property name="iAccountDao" ref="accountDao"></property>
    </bean>

    <!--配置持久层-->
    <bean id="accountDao" class="com.diaowenjie.dao.impl.AccountDaoImpl">
       <property name="jdbcTemplate" ref="jdbcTemplate"></property>
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

以上就是只使用spring自带的jdbc包实现读数据库的操作。但是以上还没有进行事务的支持，之前是采用手动写java类，然后进行配置完成事务的绑定， 其实spring支持配置的模型完成jdbc事务的绑定。

## 2.Spring TransactionManager

spring中通过配置实现事务的控制，与上面的进行结合实现。

### 2.1 基于xml声明式事务控制配置步骤

不用自己写事务代码实现：
1. 配置事务管理器
2. 配置事务的通知：此时需要导入事务的约束,tx的名称控件和约束，同时也需要aop的  
      1. 使用tx:advice标签配置事务通知：
         1. id：给事务通知起一个唯一标识
         2. transaction-manger:给职务通知提供一个事务管理器引用
3. 配置AOP汇总的通用切入点表达式
4. 建立事务通知和切入点标识的对应关系
5. 配置事务的属性：是在事务的通知tx;advice标签内部
           1. isolation=""  用于指定事务的隔离级别 默认default 标识使用数据库的默认隔离级别
        2. propagation=" " 用于指定事务的传播行为，默认是required，表示一定会有事务，增删改的选择，查询方法选择support
        3. read-only="" 用于指定事务是否只读，只有查询方法才能设置为true，默认是false，表示读写
        4. timeout="" 用于指定事务的超时时间，默认值是-1，表示永不超时，如果指定了数值，以秒为单位
        5. rollback-for="" 用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚，没有默认值，表示任何异常都回滚
        6. no-rollback-for="" 用于指定一个异常，当产生改异常时，事务不回滚，产生其他异常时，事务回滚，没有默认值，表示任何异常都回滚。


对应xml配置代码: 与上面的配置文件结合成为完整版。

```xml
<!--配置事务管理器-->
<bean id="transactionManger" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!--配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManger">
    <tx:attributes>
        <!--*表示所有的方法都进行-->
        <tx:method name="*"  propagation="REQUIRED" read-only="false" />
        <!-- 表示以find开头的方法匹配。 这样就要求命名要规范-->
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"></tx:method>
    </tx:attributes>
</tx:advice>

<!--配置aop-->
<aop:config>
    <!--配置切入点表达式-->
    <aop:pointcut id="pt1" expression="execution(* com.diaowenjie.service.impl.*.*(..))"/>
    <!--键入且锂电标识和事务通知的对应关系-->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"></aop:advisor>
</aop:config>
```

通过以上配置，此时的边实现了之前aop中手动写代码实现的事务管理功能，即将转账业务合成了一整体，也即是说之前说的都是可以通过spring的配置实现的。

### 2.2基于注解的方式

首先xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx
        https://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启扫描-->
    <context:component-scan base-package="com.diaowenjie"/>
    <!--使用jdbcTemplate 进行数据操作。-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!--注入连接数据库的必备信息-->
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/study_test"/>
        <property name="user" value="root"/>
        <property name="password" value="A123456"/>
    </bean>

<!--    spring中基于注解的声明式事务控制配置步骤
          1.配置事务管理器
          2.开启spring对注解事务的支持
          3.在需要事务支持的地方使用@Transaction注解
-->
    <!--配置事务管理器-->
    <bean id="transactionManger" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

	<!--开始spring对于注解事务的支持-->
    <tx:annotation-driven transaction-manager="transactionManger"/>
</beans>
```

然后按照ioc的方式注入，但是在需要加事务的类中加上

```java
@Service("accountService")
@Transactional  //表示事务
public class AccountServiceImpl implements IAccountService {

    @Autowired
    private IAccountDao iAccountDao;

    /**
     * 根据用户id查找用户
     * @param accountId
     * @return Account
     */
    public Account findAccountById(Integer accountId) {
        return findAccountById(accountId);
    }


    @Override
    public Account findAccountByName(String name) {
        return iAccountDao.findAccountByName(name);
    }

    /**
     * 更新账号
     * @param account
     */
    public void updateAccount(Account account) {
        iAccountDao.updateAccount(account);
    }

    /**
     * 转账操作
     * @param sourceName 源账户
     * @param targetName 目的账户
     * @param money 转账金额
     */
    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        System.out.println("执行了..");
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
    }
}
```

然后就测试成功了，但是基于注解的方式会出现不智能的情况，就是有的方法需要设置不同的事务，还是基于xml的更好用，可以为每个方法指定。

## 3.基于纯注解的方式

纯注解的方式就是需要使用config java类来代替xml配置文件，总共有三个类

主配置类SpringConfiguration.class

```java
@Configuration
@ComponentScan("com.diaowenjie")
@Import({JdbcConfig.class,TransactionConfig.class})
@PropertySource("classpath:jdbcConfig.properties" )
@EnableTransactionManagement //开启事务注解
public class SpringConfiguration {

}
```

数据库连接类jdbcConfig.class

```java
@Configuration
public class JdbcConfig {

    //通过读取properties文件中的值，进行赋值，实现可以更改。
    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;
    /**
     * 创建jdbcTemplate
     * @param dataSource
     * @return
     */
    @Bean(name = "jdbcTemplate")
    public JdbcTemplate createJdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    /**
     * 创建数据源
     * @return
     */
    @Bean(name = "dataSource")
    public DataSource createDataSource(){
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}
```

事务控制器配置类TransactionConfig.class

```java
@Configuration
public class TransactionConfig {
	//新建一个事务管理类
    @Bean(name = "transactionManager")
    public PlatformTransactionManager createTransactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```

还有一个数据库配置文件jdbcConfig.propertie文件,将连接数据库的文件不写死，通过读取进行分开。该文件在主配置类上进行设置。

```properties
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/study_test
jdbc.username = root
jdbc.password = A123456
```

然后就是测试类，业务控制逻辑代码与上面一致，以上主要是在替换xml文件

```java
public static void main(String[] args) {
    //读取配置类。
    ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
    IAccountService as = (IAccountService) ac.getBean("accountService");
    //通过测试转账有没有bug来实现整体的部署测试。
    as.transfer("aaa","bbb",100f);
}
```

OVER 注意多复习。