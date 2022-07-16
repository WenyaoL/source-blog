## ApplicationContext接口解读

[TOC]

#### ApplicationContext介绍

在使用Spring的时候，我们经常需要先得到一个ApplicationContext对象，然后从该上下文中获取我们配置的Bean对象。

我们知道ApplicationContext是一个接口，实现了改接口的实现类都可以看作一个IOC容器，我们可以从ApplicationContext中拿到注册的Beans。

ApplicationContext隶属于org.springframework.context，是SpringFramework中Bean的管理者，为SpringFramework的诸多功能提供支撑作用。



#### BeanFactory和ApplicationContext关系

BeanFactory是IOC容器的最基础的核心接口，而ApplicationContext接口继承了EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,MessageSource, ApplicationEventPublisher, ResourcePatternResolver。

**可以看出ApplicationContext是BeanFactory的子类，ApplicationContext是具备了BeanFactory的所有功能，是包含IOC容器的完整实现，我们一般都叫ApplicationContext上下文，上下文是IOC容器的完整实现而且还在其实现的基础上添加了许多功能，如ApplicationEventPublisher接口就提供的上下文事件发布的功能。**

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
      MessageSource, ApplicationEventPublisher, ResourcePatternResolver
```



- EnvironmentCapable：获取 Environment。
- ListableBeanFactory、HierarchicalBeanFactory：这是 BeanFactory 体系三大接口中的两个，分别提供 Bean 迭代和访问父容器的功能。
- MessageSource：支持国际化功能。
- ApplicationEventPublisher：应用事件发布器，封装事件发布功能的接口。
- ResourcePatternResolver：该接口继承至 ResourceLoader ，作用是加载多个 Resource。



#### ApplicationContext源码

ApplicationContext集合了几个核心接口外，还格外添加了几个获取上下文信息的方法，如：getId()，getApplicationName()，getParent()

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

	/**
	 * Return the unique id of this application context.
	 * @return the unique id of the context, or {@code null} if none
	 */
	@Nullable
	String getId();

	/**
	 * Return a name for the deployed application that this context belongs to.
	 * @return a name for the deployed application, or the empty String by default
	 */
	String getApplicationName();

	/**
	 * Return a friendly name for this context.
	 * @return a display name for this context (never {@code null})
	 */
	String getDisplayName();

	/**
	 * Return the timestamp when this context was first loaded.
	 * @return the timestamp (ms) when this context was first loaded
	 */
	long getStartupDate();

	/**
	 * Return the parent context, or {@code null} if there is no parent
	 * and this is the root of the context hierarchy.
	 * @return the parent context, or {@code null} if there is no parent
	 */
	@Nullable
	ApplicationContext getParent();


	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;

}
```

