# Spring中的BeanFactoryPostProcessor

[TOC]

### BeanFactoryPostProcessor介绍

`BeanFactoryPostProcessor`和`BeanPostProcessor`接口类似，也是属于spring中的一种处理器。于`BeanPostProcessor`相比，`BeanFactoryPostProcessor`属于容器级别的后置处理器，它对容器做处理。



### BeanFactoryPostProcessor执行时机

`BeanFactoryPostProcessor`函数的执行时机可以在`AbstractApplicationContext`的`refresh()`中看到。

可以发现在`ApplicationContext`刷新时（初始化），在完成`BeanFactory`初始化后，会调用各种`BeanFactoryPostProcessor`

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
            //调用各种BeanFactoryPostProcessor(可以实现BeanFactoryPostProcessor接口，工厂后置处理器)
            //这些调用的BeanFactoryPostProcessor有application和在beanFactory中的
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



### 代码示例

这里有部分代码参考《Spring源码深度解析》，对`bean`元数据的一些值进行字符串解析（其实这里做了对xxx字符串的过滤而已）。

这么说呢，假如你有一个Bean User，`Class User(userName="xxx",password="123456")`，那么就会变成`Class User(userName="",password="123456")`这样的效果。

```java
/**
 * @author liangwy
 * @since 2022-5-4
 */
public class TestBeanFactoryPostProcessor implements BeanFactoryPostProcessor, Ordered {


    /**
     * 可以根据自己的逻辑去对beanFactory进行改造
     * @param beanFactory
     * @throws BeansException
     */
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        //添加各种BeanPostProcessor
        beanFactory.addBeanPostProcessor(new TestBeanPostProcessor());


        //或者对BeanDefinition做做属性值过滤
        //下面的代码参考《Spring源码深度解析》
        String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
        for (String name:beanDefinitionNames){
            BeanDefinition beanDefinition = beanFactory.getBeanDefinition(name);
            //字符串解析器
            StringValueResolver stringValueResolver = new StringValueResolver() {
                @Override
                public String resolveStringValue(String strVal) {
                    //解析字符串，这里用来过滤xxx字符串
                    if(strVal.equals("xxx")) return "";
                    return strVal;
                }
            };
            //这是个访问器，用来applying the specified value resolver to all bean metadata values.
            //也就是对bean的元数据值调用解析器去解析，只不过这里对xxx的值会解析成空字符串
            BeanDefinitionVisitor beanDefinitionVisitor = new BeanDefinitionVisitor(stringValueResolver);
            beanDefinitionVisitor.visitBeanDefinition(beanDefinition);
        }

    }

    @Override
    public int getOrder() {
        //如果想对多个BeanFactoryPostProcessor做排序处理，还可以实现改接口
        return 0;
    }
}
```

