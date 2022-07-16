## spring中的Lifecycle

### Lifecycle介绍

`Lifecycle`是spring中的生命循环接口，实现该接口的类将会可以启动，关闭。在`AbstractApplicationContext`通过`ConfigurableApplicationContext`接口中就实现了`Lifecycle`，当上下文调用`start()`函数时，就会去BeanFactory中找实现了`Lifecycle`接口的Bean，并调用该bean的`start()`函数。



### AbstractApplicationContext源码分析

#### 初始化LifecycleProcessor

`AbstractApplicationContext`中有方法`initLifecycleProcessor()`，用来初始化生命周期执行器，用以执行容器中所有实现了`Lifecycle`接口的bean。

```java
/**
	 * Initialize the LifecycleProcessor.
	 * Uses DefaultLifecycleProcessor if none defined in the context.
	 * @see org.springframework.context.support.DefaultLifecycleProcessor
	 */
protected void initLifecycleProcessor() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //参试在容器中获取LifecycleProcessor
    if (beanFactory.containsLocalBean(LIFECYCLE_PROCESSOR_BEAN_NAME)) {
        this.lifecycleProcessor =
            beanFactory.getBean(LIFECYCLE_PROCESSOR_BEAN_NAME, LifecycleProcessor.class);
        if (logger.isTraceEnabled()) {
            logger.trace("Using LifecycleProcessor [" + this.lifecycleProcessor + "]");
        }
    }
    else {
        //容器没有则使用DefaultLifecycleProcessor
        DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();
        //设置BeanFactory，因为要对BeanFactory里面的Lifecycle bean调用start,stop
        defaultProcessor.setBeanFactory(beanFactory);
        this.lifecycleProcessor = defaultProcessor;
        //注册到容器中
        beanFactory.registerSingleton(LIFECYCLE_PROCESSOR_BEAN_NAME, this.lifecycleProcessor);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + LIFECYCLE_PROCESSOR_BEAN_NAME + "' bean, using " +
                         "[" + this.lifecycleProcessor.getClass().getSimpleName() + "]");
        }
    }
}
```



#### start()和stop()

`AbstractApplicationContext`中的`start()和stop()`方法用来启动生命周期。通过`LifecycleProcessor`来去启动`BeanFactory`中的实现了`Lifecycle`接口的bean。

```java
//---------------------------------------------------------------------
// Implementation of Lifecycle interface
//---------------------------------------------------------------------

@Override
public void start() {
    getLifecycleProcessor().start();
    publishEvent(new ContextStartedEvent(this));
}

@Override
public void stop() {
    getLifecycleProcessor().stop();
    publishEvent(new ContextStoppedEvent(this));
}
```



### DefaultLifecycleProcessor源码分析

#### start()和startBeans()

`DefaultLifecycleProcessor`通过从`BeanFactory`中获取`LifecycleBean`，并调用它们的`start()`方法

```java
@Override
public void start() {
    startBeans(false);
    this.running = true;
}

// Internal helpers

private void startBeans(boolean autoStartupOnly) {
    //这里是从BeanFactory中获取所有LifecycleBean
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new HashMap<>();
    lifecycleBeans.forEach((beanName, bean) -> {
        if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
            int phase = getPhase(bean);
            LifecycleGroup group = phases.get(phase);
            if (group == null) {
                group = new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly);
                phases.put(phase, group);
            }
            group.add(beanName, bean);
        }
    });
    if (!phases.isEmpty()) {
        List<Integer> keys = new ArrayList<>(phases.keySet());
        Collections.sort(keys);
        for (Integer key : keys) {
            //最后会调用该bean的start()方法
            phases.get(key).start();
        }
    }
}
```



### Lifecycle Bean使用示例

#### MyLifecycleBean

```java
/**
 * @author liangwy 
 * @since 2022-5-17
 */
public class MyLifecycleBean implements Lifecycle {
    
    private Boolean flag = false;
    
    @Override
    public void start() {
        this.flag = true;
        System.out.println("启动MyLifecycleBean");
    }

    @Override
    public void stop() {
        this.flag = false;
        System.out.println("关闭MyLifecycleBean");
    }

    @Override
    public boolean isRunning() {
        return flag;
    }
}
```

#### 测试

```java
public static void main(String[] args) {

    AnnotationConfigApplicationContext applicationContext = runAnnotationConfigApplicationContext();
    applicationContext.start();
    MyLifecycleBean myLifecycleBean = applicationContext.getBean("myLifecycleBean", MyLifecycleBean.class);
    System.out.println(myLifecycleBean.isRunning());
    applicationContext.stop();
    System.out.println(myLifecycleBean.isRunning());
}

/**
     * 注解配置方式创建上下文
     * @return
     */
public static AnnotationConfigApplicationContext runAnnotationConfigApplicationContext(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationMain.class);
    applicationContext.getBeanFactory().registerCustomEditor(Date.class,TestPropertyEditorSupport.class);
    return applicationContext;
}
```

**结果打印**

> 启动MyLifecycleBean
> true
> 关闭MyLifecycleBean
> false