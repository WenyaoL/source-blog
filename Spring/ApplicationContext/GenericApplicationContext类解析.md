## GenericApplicationContext类解析

[TOC]



#### GenericApplicationContext介绍

GenericApplicationContext通用应用程序上下文实现，该实现内部有一个 DefaultListableBeanFactory 实例。可以采用混合方式处理bean的定义，而不是采用特定的bean定义方式来创建bean。

GenericApplicationContext基本就是对DefaultListableBeanFactory 做了个简易的封装，几乎所有方法都是使用了DefaultListableBeanFactory的方法去实现。

GenericApplicationContext更多是作为一个通用的上下文（通用的IOC容器）而存在，BeanFactory本质上也就是一个IOC容器，一个用来生产和获取beans的工厂。

几乎可以把GenericApplicationContext等价于DefaultListableBeanFactory+上下文刷新等其他功能



```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
	//内部的beanFactory
	private final DefaultListableBeanFactory beanFactory;

	@Nullable
	private ResourceLoader resourceLoader;

	private boolean customClassLoader = false;

	private final AtomicBoolean refreshed = new AtomicBoolean();


	/**
	 * Create a new GenericApplicationContext.
	 * @see #registerBeanDefinition
	 * @see #refresh
	 */
	public GenericApplicationContext() {
		this.beanFactory = new DefaultListableBeanFactory();
	}
	//......
```



#### 利用BeanDefinitionReader读取不同资源到上下文中

其实就是通过BeanDefinitionReader将beanDefinition放到DefaultListableBeanFactory中维护的beanDefinitionMap表中

```java
GenericApplicationContext ctx = new GenericApplicationContext();

//其实就是往BeanFactory里面装BeanDefinition
//BeanFactory的实现类一般都维护着一张beanDefinitionMap表
XmlBeanDefinitionReader xmlReader = new XmlBeanDefinitionReader(ctx);
xmlReader.loadBeanDefinitions(new ClassPathResource("applicationContext.xml"));

PropertiesBeanDefinitionReader propReader = new PropertiesBeanDefinitionReader(ctx);
propReader.loadBeanDefinitions(new ClassPathResource("applicationContext.properties"));

ctx.refresh();
```

