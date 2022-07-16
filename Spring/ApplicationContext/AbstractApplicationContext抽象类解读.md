# AbstractApplicationContext抽象类解读

[TOC]



### AbstractApplicationContext介绍

`AbstractApplicationContext`一个抽象应用程序上下文，主要实现了上下文初始化的功能，如：定义了初始化`BeanFactory`，初始化一些`BeanFactoryPostProcessors`，初始化`MessageSource`（国际化），还要一些事件广播器`ApplicationEventMulticaster`等等

`AbstractApplicationContext`因为是抽象类，其实其中一些初始化操作还是交个子类实现，如初始化`BeanFactory`，在`AbstractApplicationContext`只是做了简单的定义，具体实现还是交个子类。子类根据实际情况选择使用那种`BeanFactory`并进行初始化。

`ApplicationContext`是一个比BeanFactory更具丰富IOC容器接口，而`AbstractApplicationContext`是其的一个抽象实现，定义和实现了一系列的初始化流程。

### AbstractApplicationContext核心方法

#### refresh

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        //准备上下文刷新
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        //这里创建BeanFactory，具体逻辑交个子类实现（不同的上下文使用不同的BeanFactory）
        //加载逻辑还是基本通过BeanDefinitionReader去加载BeanDefinition并注册到BeanFactory中
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        //为BeanFactory在上下文中使用做准备（其实是给BeanFactory配置一些信息）
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            //调用各种BeanFactoryPostProcessor(可以实现BeanFactoryPostProcessor接口制作工厂后置处理器)
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            //注册BeanPostProcessors
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            //初始化MessageSource（国际化）
            initMessageSource();

            // Initialize event multicaster for this context.
            //初始化事件广播器
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            //空实现，交给子类实现，具体是让子类如何刷新上下文
            onRefresh();

            // Check for listener beans and register them.
            //注册监听器
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
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



#### obtainFreshBeanFactory

```java
/**
	 * Tell the subclass to refresh the internal bean factory.
	 * @return the fresh BeanFactory instance
	 * @see #refreshBeanFactory()
	 * @see #getBeanFactory()
	 */
//
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //空实现，具体交给子类实现（因为不同子类根据实际情况用不同的BeanFactory,所以具体实现交个子类）
    refreshBeanFactory();
    return getBeanFactory();
}
```



#### prepareBeanFactory

该方法主要用于配置BeanFactory，如：配置`BeanClassLoader`，配置`BeanPostProcessor`，忽略一些依赖接口，配置一些环境Bean等等

```java
	/**
	 * Configure the factory's standard context characteristics,
	 * such as the context's ClassLoader and post-processors.
	 * @param beanFactory the BeanFactory to configure
	 */
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		// Tell the internal bean factory to use the context's class loader etc.
		beanFactory.setBeanClassLoader(getClassLoader());
        //这里增加了SPEL语言支持（使得能解析#{bean.xxx}相关形式的属性值）
        //这玩意会在解析Bean时用到
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
        //忽略接口，添加BeanPostProcessor
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

		// Detect a LoadTimeWeaver and prepare for weaving, if found.
        //这里增加对AspectJ的支持
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}

		// Register default environment beans.
        //注册环境bean
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```

