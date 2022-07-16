## BeanFactory接口解读

[TOC]



#### BeanFactory的介绍

在Spring中，**BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory等**



#### BeanFactory接口源码

我们从源码可以得知，实现了BeanFactory接口的类，必定具备获取bean的能力，并且具备实例化bean的能力。

实现了BeanFactory接口是已经具备IOC容器的基本功能，当然你想要实现一个完整的IOC容器，单单实现BeanFactory接口还不足以实现完整的IOC容器。

```java
public interface BeanFactory {

    /**
     * factoryBean的前缀
     */
    String FACTORY_BEAN_PREFIX = "&";

    /*
     * 四个不同形式的getBean方法
     */
    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException;
	
	// 是否存在某个bean
    boolean containsBean(String name); 
    
	// 是否为单实例，bean的作用域为Singleton
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	// 是否为原型（多实例），bean的作用域为Prototype
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	// 名称、类型是否匹配
    boolean isTypeMatch(String name, Class<?> targetType)
            throws NoSuchBeanDefinitionException;
	// 通过Bean的名字获取该bean的类型
    Class<?> getType(String name) throws NoSuchBeanDefinitionException; 
	// 根据实例的名字获取实例的别名
    String[] getAliases(String name);

}
```



#### BeanFactory直属派生接口

BeanFactory有三大派生的直属接口，分别是：ListableBeanFactory，HierarchicalBeanFactory，AutowireCapableBeanFactory

> - ListableBeanFactory：提供容器中 bean 迭代的功能。如返回所有 Bean 的名字、容器中 Bean 的数量等。
> - HierarchicalBeanFactory：提供父容器的访问功能，可通过 ConfigurableBeanFactory 的 setParentBeanFactory 方法设置父容器，主要是为了实现了Bean工厂的分层。
> - AutowireCapableBeanFactory：为 Spring 容器之外的 Bean ，也就是未交由 Spring 管理的 Bean ，提供依赖注入的功能。



#### BeanFactory与BeanDefinitionRegistry

BeanFactory和BeanDefinitionRegistry一样，属于IOC容器的核心接口，BeanFactory提供了IOC容器获取bean的功能，而BeanDefinitionRegistry就提供了IOC容器的注册功能。所有要实现一个完整的IOC容器还必须实现BeanDefinitionRegistry接口。





#### BeanFactory和ApplicationContext关系

BeanFactory是IOC容器的最基础的核心接口，而ApplicationContext接口继承了EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,MessageSource, ApplicationEventPublisher, ResourcePatternResolver。

**可以看出ApplicationContext是BeanFactory的子接口，ApplicationContext是具备了BeanFactory的所有功能，是包含IOC容器的完整实现，我们一般都叫ApplicationContext上下文，上下文是IOC容器的完整实现而且还在其实现的基础上添加了许多功能，如ApplicationEventPublisher接口就提供的上下文事件发布的功能。**

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```