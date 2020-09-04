---
layout: post
title: Java：Spring IoC源码阅读
tags: java  
---


> 记录spring源码阅读记录，记录Bean的生命周期，主要参考了Spring技术内幕这本书

#  目录
* 目录
{:toc}
# 参考

[Spring Analysis](https://github.com/seaswalker/spring-analysis) 《Spring技术内幕》第二版   

简单的初始化Spring例子  

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

# Spring容器

Spring容器可以简单的理解为ApplicationContext对象，即用户所有的对象申请和配置的对象的属性和依赖关系都可以通过ApplicationContext对象获取到，它是个底层实现类，Spring定义了一个工厂接口类型BeanFactory，并且定义了spring容器最基本的操作：

```java
public interface BeanFactory {

	String FACTORY_BEAN_PREFIX = "&";
    // 获取Bean 最主要的方法。
	Object getBean(String name) throws BeansException;
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	Object getBean(String name, Object... args) throws BeansException;
	<T> T getBean(Class<T> requiredType) throws BeansException;
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
	
    // 是否包含指定名称的Bean
	boolean containsBean(String name);
    // Bean是否是单例的
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    // Bean是否是多例的
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    // 查询指定名字Bean的Class类型是否是特定的Class类型
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
	// 查询指定名字Bean的Class类型
    @Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
	@Nullable
	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;
	// 查询指定名字Bean的所有别名。别名由用户定义
    String[] getAliases(String name);
}
```

从方法名字中可以看出一个最简单的Spring容器需要提供什么功能。其中最主要的方法就是getBean()方法，这个也是用户最常用的方法。BeanFactory作为生成Bean的工厂，那么生成BeanFactory的对象在Spring中被命名为FactoryBean，即通过FactoryBean对象可以获取BeanFactory。用户在使用容器的时候，可以通过转义符“&”获取得到FactoryBean对象。

Spring容器的具体实现通常使用具体的实现类进行实现，比如DefaultListableBeanFactory、XMLBeanFactory、ApplicationContext。下面开始分析Spring中Ioc的实现。整个IOC可以分为如下几个部分：

- IoC容器的初始化过程
  - BeanDefinition的Resource定位
  - BeanDefinition的载入和解析
  - BeanDefinition在Ioc容器中的注册
- 和IoC容器的依赖注入

# BeanDefinition

从上面可以看出，整个Ioc的过程是基于BeanDefinition对象实现的。在Spring中，用户可以使用的对象叫做Bean，它是由用户进行定义的，可以通过XML的方式或者是通过注解进行实现，而spring中解析过后的xml文件就成了BeanDefinition对象，即BeanDefinition记录了用户对于Bean的属性和依赖关系。

> Spring通过定义BeanDefinition来管理基于Spring应用中的各种对象以及它们之间的相互依赖关系，BeanDefinition抽象了我们对于Bean的定义，是容器起主要作用的数据类型。IoC容器是管理对象依赖关系的，对于IoC容器来说，BeanDefinition就是对依赖反转模式中管理的对象依赖关系的数据抽象，也是容器实现依赖反转功能的核心数据结构，依赖反转功能都是围绕这个BeanDefinition的处理来完成的。

通过Spring对BeanDefinition完成生产Bean，话句话说，BeanDefinition可以看成Bean的配置类。

# IoC初始化过程

简单说，IoC容器初始化是有refresh()方法来启动的，这个方法标志了Ioc容器的正式启动，主要包括BeanDefinition的Resource定位、载入和注册三个基本的过程。即先找到xml文件，然后加载到内存中，然后进行解析xml文件进行配置BeanDefinition对象，最后根据BeanDefinition对象在IoC容器中进行注册Bean。以下为AbstractApplicationContext对容器的初始化过程方法。

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // 设置BeanFactory的后置处理器
            postProcessBeanFactory(beanFactory);

            // 调用BeanFactory的后置处理器，这些处理器是在Bean定义中向容器注册的。
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册Bean的后置处理器
            registerBeanPostProcessors(beanFactory);

            // 对上下文的消息源进行初始化
            initMessageSource();

            // 初始化上下文中的时间机制
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // 检查监听Bean并且将这些Bean向容器注册
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // 发布容器事件，结束Refresh进程。
            finishRefresh();
        }

        catch (BeansException ex) {
  
            // 为防止Bean资源占用，在异常处理中，销毁已经在前面过程中生成的单间
            destroyBeans();
            // Reset 'active' flag.
            cancelRefresh(ex);
            // Propagate exception to caller.
            throw ex;
        }
        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

## BeanDefinition的Resource定位

定位主要是找到Xml文件的位置，即找到BeanDefinition的资源定位，它是由ResourceLoader通过统一的Resource接口来完成。用户定义如下：

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
```

处理上面的bean.xml字符串，并且根据它找到xml的真正位置记性读取，一般都是使用DefaultResourceLoader进行实现定位。主要功能就是解析String类型的location标识。

```java
Resource resource = resourceLoader.getResource(loaction);
```

```java
@Override
public Resource getResource(String location) {
    Assert.notNull(location, "Location must not be null");

    for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
        Resource resource = protocolResolver.resolve(location, this);
        if (resource != null) {
            return resource;
        }
    }

    if (location.startsWith("/")) {
        return getResourceByPath(location);
    }
    //处理带有classpath标示的resource。
    else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
        return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
    }
    else {
        try {
            // Try to parse the location as a URL...
            URL url = new URL(location);
            return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
        }
        catch (MalformedURLException ex) {
            // No URL -> resolve as resource path.
            return getResourceByPath(location);
        }
    }
}
```

## BeanDefinition的载入和解析

通过以上的getSource进行找到了xml文件的具体位置，下一步就是考虑载入Xml文件并且进行解析。

> 对于Ioc容器来说，这个载入的过程，相当于把定义的BeanDefinition在在IoC容器中转化为一个Spring内部表示的数据结构的过程。这些BeanDefinition数据在IoC容器中是通过一个HashMap来保持和维护。

> 具体的读取是在XmlBeanDefinitionReader进行的实现，读取得到XML文件对象，有了这个对象以后，就可以按照Spring对Bean的定义规则来对这个XML文档进行解析了，而这个解析是交给BeanDefinitionParserDelegate对象来完成。
>
> BeanDefinition的载入分为两个部分，首先是调用XML解析器得到document对象，但是这个document对象并没有按照Spring的Bean规则进行解析，在完成通用的XML解析以后，才是按照Spring的Bean规则进行解析的地方，这个按照Spring的Bean规则进行解析的过程是在documentReader中实现的。具体的Spring BeanDefinition的解析是在BeanDefinitionParserDelegate中完成的

其中解析就是对一些xml中的标签进行解析，比如说对`    <bean id="accountDao" class="com.diaowenjie.dao.impl.AccountDaoImpl"/>`进行解析，并且根据解析结果，对这些属性值的处理会被分装到Property Value对象并设置到BeanDefinition对象中去。比如\<list>标签就会生成list对象进行存储。

其中载入的接口：

```java
// 主要调用这个接口。
public int loadBeanDefinitions(InputSource inputSource, @Nullable String resourceDescription)
    throws BeanDefinitionStoreException {
    return doLoadBeanDefinitions(inputSource, new DescriptiveResource(resourceDescription));
}

protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) 
    throws BeanDefinitionStoreException {
    try {
        Document doc = doLoadDocument(inputSource, resource);
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    }
    catch (BeanDefinitionStoreException ex) {
        throw ex;
    }
}
```

对于解析接口，不同的标签类型会有不同的解析类进行操作。主要是由BeanDefinitionParserDelegate这个类进行完成。具体不表。。

```java
@Nullable
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
    return parseBeanDefinitionElement(ele, null);
}
```

## BeanDefinition在IoC容器中的注册

在完成BeanDefinition在IoC容器中的载入与解析之后，需要在IoC中对这些BeanDefinition数据进行注册，注册也就是将得到的BeanDefinition存放在一个HashMap中，如下：

```java
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashmap<String, BeanDefinition>();
```

# IoC容器的依赖注入

依赖注入的过程是在用户第一次向IoC容器索要Bean的时候进行触发，即在调用getBean()方法的时候进行依赖注入。但是也可以通过配置lazy-init属性让IoC容器进行预初始化。

其中getBean是依赖注入的起点，之后会调用createBean()，而最后生成的Bean就是根据BeanDefinition定义的要求生成。

另外与依赖注入关系特别密切的方法有createBeanInstance()使用反射创建对象实例和populateBean() 完成依赖注入。还有一个重要的方法就是后置处理器。

# 解决循环依赖

什么叫做循环依赖？

```java
public class X{
    @Autowird
    Y y;
}

public class Y{
    @Autowird
    X x;
}
```

上面两个类就是互相依赖，即互相引用，形成循环依赖，默认情况下Spring是支持循环依赖的。

## 循环引用创建过程

## Spring中的三级缓存

- 一级缓存 singletonObjects 单例池： 已经走完bean声明周期的对象，最后可以用的对象

- 二级缓存 singletonfactories 工厂： 生成bean对象的工厂。

- 三级缓存 earlySingletonObjects：  存储的是尚未走完bean周期的对象。

以上三种都是Map类型的数据。即使用HashMap进行缓存的实现。

Spring解决循环引用的问题主要由下面这个getSingleton()函数进行实现，即默认情况下Spring是支持循环依赖的。

```java
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 从一级缓存中拿出
    Object singletonObject = this.singletonObjects.get(beanName);
    // isSingletonCurrentlyInCreation 这个就是判断是否该对象正在被创建
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
           	// 三级缓存中拿出
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                // 二级缓存中拿出数据
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    // 添加到三级缓存中
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    // 从二级工厂缓存中删除。
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

对于循环引用 (x类引用y，y也引用x)来说，在创建第一个对象x的的时候，发现引用的对象y没有被创建，即getBean为null，则会去直接创建需要引用的对象y，但是在创建y的时候发现需要引用x，这时候也去创建x，但是在创建对象的时候，会首先检测有没有已经在创建，此时便会从isSingletonCurrentlyInCreation中检测到已经在创建，此时便尝试从三级缓存中拿出来，如果没有便从二级缓存（工厂缓存）拿出来进行注入，然后返回到第一层，杜绝了一直循环创造对象。

**为什么会检测到正在被创建？**  

 因为是spring在创建bean之前会将其放在一个map中表示正在被创建。 所以说也是在之前进行标记好的，就是因为有这个问题，所以才会有这套机制。isSingletonCurrentlyInCreation() 这个方法判断是否是在创建当中，上面的x就是正在创建过程中，可以说是半成品bean，一个完整的bean要走完bean的生命周期才算。此时会检测到x不是第一次创建，这时候便从去二级缓存中拿出来对象。

上面的代码表示从二级缓存中get一个对象，然后放进三级缓存中，最后二级缓存中的对象remove掉 。

**为什么需要三级缓存**？   

主要是因为二级缓存中的工厂方法非常的复杂，需要一个for循环进行bean后置处理器循环，而三级缓存的话就是直接get就可以获取到，就是为了性能的提高，其实，回到缓存这个概念，它就是为了提高效率才出现的，所以说这里需要缓存也是为了性能的提高。

**那么为什么要从二级缓存中获取到对应的半成品对象** ？  

 主要就是因为aop。因为对于一个对象来说，当进行aop增强的时候，返回的对象就已经不是最初的那个对象，即地址已经改变了。 如果直接存储到三级缓存中，那么就是直接存储的是一个没被增强的对象的地址，（这里所说的对象就是存储的对象的地址，而aop会更改对象的地址，即重新生成了一个对象），而调用工厂的方法就会返回aop之后的对象的地址，即还在生产中的对象。这样在调用二级缓存的get方法时，就会使用工厂方法将增强的对象构造出现，如此即使是循环依赖中有aop也没事，获取到的是最后aop增强的对象。当时如果没有aop的话，即不使用二级缓存即工厂生产对象即可。直接将new出来的对象地址存储到hash中就行。

这样其实也解释了一个问题，为什么可以用使用hashmap的时候定义对象为object，因为存储的value就是一个地址，而object是所有对象的父类，所有可以使用object保存地址，而使用的时候直接变成目标对象即可，即操作对象的时候只是通过引用找到对象，所有可以用object作为value进行存储。

# 容器其他相关特性

## BeanPostProcessor实现

post即为后置，所以这个是为Bean的后置处理器，他是一个监听器，可以监听容器中触发的时间，在进行依赖注入的时候会多次调用Bean的后置处理器。如果需要在Bean的声明周期种进行对Bean进行相关的操作，则需要实现一个继承类并且实现两个接口注入到容器中。可以看出这个后置处理器直接获取的是创建中的半成品bean，即这时候用户可以直接的操作bean对象进行自己的个性化操作。

```java
public interface BeanPostProcessor {
	
    // 在Bean初始化之前进行调用接口
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
	
    // 在初始化之后调用接口。
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```

其中postProcessBeforeInitialization是在populateBean完成之后被调用，

## Bean对IoC容器的感知

一般来说Bean不需要知道容器的状态或者是直接使用容器，但是在一些场景下还是需要的，这时候就需要一个感知类，即带有aware的类。这个感知是在Bean的声明周期中进行设置的属性。

- BeanNameAware :使用Bean得到它在IoC容器中的Bean实例名称
- BeanFactoryAware 可以在Bean中得到Bean所在IoC容器，从而直接在Bean中使用应用上下文
- ApplicaitonContextAware ：可以在Bean中得到Bean所在的应用上下文，从而直接在Bean中使用应用上下文
- MessageSourceAware：在Bean中得到信息源
- ApplicaitonEventPublisherAware
- ResourceLoaderAware

## Bean的生命周期

Bean的声明周期可以简单的理解，首先Spring中都是使用工厂方式进行创建Bean，所以首先需要创建BeanFactory，为了提供扩展性，所以在创建BeanFactory的时候，也会调用后置处理器BeanFactoryPostProcesser()方法，用来扩展或者是操作BeanFactory，在创建好工厂以后，便开始用工厂创建Bean。在这期间还需要设置各种Aware接口方法，同样为了进行扩展，也会在创建Bean的时候调用BeanPostProcesser()进行扩展Bean，其中AOP也是在调用BeanPostProcesser()的时候进行的，在这期间也会进行对象的生命周期，即初始化函数，进行赋值。在完成之后便完成了Bean的创建，之后便是Bean的销毁与回收。

以下为比较详细的Bean的声明周期。

- 通过构造方法函数或工厂方法重新创建Bean实例
- 设置属性值和对其他Bean的引用(这里其实也会调用构造方法的前置和后置处理器)
- 调用所有的*Aware接口中定义的setter方法
- 将Bean实例传递给每个Bean后置处理器postProcessBeforeInitialization()
- 初始化回调方法
- 将Bean实例传递给每个Bean的后调处理器postProcessAfterInitialization()
- 这个Bean已经本可以被使用
- 当容器关闭时调用销毁方法。

从上面的周明周期可以看出，整个Bean的生命周期就是伴随着一个简单的对象创建的过程加上各种的后置处理器，比如在调用构造函数的时候加上后置处理器，在进行属性注入的时候加上前后置处理器，并且设置Aware属性，即能够找到这个Bean是被谁创建生成的。另外所有的Bean都是使用工厂方法进行生成。

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 BeanFactory是最开始的简单的IoC容器。
对于ApplicationContext容器，**当容器启动结束后，便实例化所有的bean。**这个是扩展了的BeanFactory对象，也可以叫做应用上下文。

### BeanFactory和ApplicationContext的异同点

**相同点：**

两者都是通过xml配置文件加载bean,ApplicationContext和BeanFacotry相比,提供了更多的扩展功能。  

不同点：  

BeanFactory是延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；而ApplicationContext则在初始化自身是检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用ApplicationContext。  

### 图形化生命周期

以下为springBean的声明周期，其实主要就是涉及到BeanFactory、Bean的生成和前后置处理器，另外还有就是生成Bean以后Aware方法的设置。

![BeanlifeCyc1.png](https://pic.tyzhang.top/images/2020/07/02/BeanlifeCyc1.png)

![beanLifeCyc.png](https://pic.tyzhang.top/images/2020/07/02/beanLifeCyc.png)

Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：

1. Bean自身的方法：这个包括了Bean本身调用的方法和通过配置文件中\<bean>的init-method和destroy-method指定的方法

2. Bean级生命周期接口方法：这个包括了BeanNameAware、BeanFactoryAware、InitializingBean和DiposableBean这些接口的方法

3. 容器级生命周期接口方法：这个包括了InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。

4. 工厂后处理器接口方法：这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

[参考](https://www.cnblogs.com/zrtqsk/p/3735273.html)

一个完整的声明周期流图，从这上main可以看出来，会调用两个后置处理器，即BeanFactory的后置处理器，BeanPostProcessor的后置处理器。还有就是设置感知器Aware，其中后置处理器总共调用了六次。

[![beanLifiCycAll.png](https://pic.tyzhang.top/images/2020/07/02/beanLifiCycAll.png)](https://pic.tyzhang.top/image/Bktc)