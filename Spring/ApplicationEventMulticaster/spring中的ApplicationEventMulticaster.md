# spring中的ApplicationEventMulticaster

[TOC]



### ApplicationEventMulticaster介绍

ApplicationEventMulticaster是spring中事件广播器接口，负责事件的广播发布。`AbstractApplicationContext`中使用`initApplicationEventMulticaster()`初始化事件广播器。

### ApplicationEventMulticaster接口源码

```java
public interface ApplicationEventMulticaster {

	/**
	 * Add a listener to be notified of all events.
	 * @param listener the listener to add
	 */
    //添加事件监听器
	void addApplicationListener(ApplicationListener<?> listener);

	/**
	 * Add a listener bean to be notified of all events.
	 * @param listenerBeanName the name of the listener bean to add
	 */
    //添加事件监听器，使用容器中的bean
	void addApplicationListenerBean(String listenerBeanName);

	/**
	 * Remove a listener from the notification list.
	 * @param listener the listener to remove
	 */
    //移除事件监听器
	void removeApplicationListener(ApplicationListener<?> listener);

	/**
	 * Remove a listener bean from the notification list.
	 * @param listenerBeanName the name of the listener bean to remove
	 */
	void removeApplicationListenerBean(String listenerBeanName);

	/**
	 * Remove all listeners registered with this multicaster.
	 * <p>After a remove call, the multicaster will perform no action
	 * on event notification until new listeners are registered.
	 */
    //移除所有事件监听器
	void removeAllListeners();

	/**
	 * Multicast the given application event to appropriate listeners.
	 * <p>Consider using {@link #multicastEvent(ApplicationEvent, ResolvableType)}
	 * if possible as it provides better support for generics-based events.
	 * @param event the event to multicast
	 */
    //发布事件
	void multicastEvent(ApplicationEvent event);

	/**
	 * Multicast the given application event to appropriate listeners.
	 * <p>If the {@code eventType} is {@code null}, a default type is built
	 * based on the {@code event} instance.
	 * @param event the event to multicast
	 * @param eventType the type of event (can be {@code null})
	 * @since 4.2
	 */
    //发布事件
	void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);

}
```



### 上下文初始化ApplicationEventMulticaster

`AbstractApplicationContext`中使用`initApplicationEventMulticaster()`初始化事件广播器。

- 如果容器里面有名为`applicationEventMulticaster`的bean，这将该bean设为上下文中的事件广播器。
- 如果容器里面没有`applicationEventMulticaster`的bean，默认创建`SimpleApplicationEventMulticaster`来代替。

```java
	/**
	 * Initialize the ApplicationEventMulticaster.
	 * Uses SimpleApplicationEventMulticaster if none defined in the context.
	 * @see org.springframework.context.event.SimpleApplicationEventMulticaster
	 */
protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //如果beanFactory里面有applicationEventMulticaster，这使用容器里面的这个事件广播器
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
        this.applicationEventMulticaster =
            beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
        if (logger.isTraceEnabled()) {
            logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
        }
    }
    else {
        //否则创建SimpleApplicationEventMulticaster来代替
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
                         "[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
        }
    }
}
```



### 自定义简易的ApplicationEventMulticaster

#### ApplicationEventMulticaster

自定义广播器

```java
/**
 * @author liangwy
 * @since 2022-5-16
 */
@Component
public class MyApplicationEventMulticaster implements ApplicationEventMulticaster {


    private ConfigurableBeanFactory beanFactory;

    private List<ApplicationListener<?>> listenerList;

    public MyApplicationEventMulticaster(ConfigurableBeanFactory beanFactory){
        this.beanFactory = beanFactory;
    }

    @Override
    public void addApplicationListener(ApplicationListener<?> listener) {
        listenerList.add(listener);
    }

    @Override
    public void addApplicationListenerBean(String listenerBeanName) {
        ApplicationListener bean = beanFactory.getBean(listenerBeanName, ApplicationListener.class);
        addApplicationListener(bean);
    }

    @Override
    public void removeApplicationListener(ApplicationListener<?> listener) {
        listenerList.remove(listener);
    }

    @Override
    public void removeApplicationListenerBean(String listenerBeanName) {
        ApplicationListener bean = beanFactory.getBean(listenerBeanName, ApplicationListener.class);
        removeApplicationListener(bean);
    }

    @Override
    public void removeAllListeners() {
        listenerList.clear();
    }

    @Override
    public void multicastEvent(ApplicationEvent event) {
        multicastEvent(event,ResolvableType.forInstance(event));
    }

    @Override
    public void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType) {
        //判空
        ResolvableType type = (eventType != null ? eventType : ResolvableType.forInstance(event));
        
        //如果来自自定义事件,才进行事件分发（不建议这样，这里只是演示示例）
        if (type.isAssignableFrom(MyApplicationEvent.class)){
            //对监听器遍历处理事件
            for (final ApplicationListener listener:listenerList){
                listener.onApplicationEvent(event);
            }
        }

    }
}
```

#### ApplicationListener

自定义监听器

```java
/**
 * @author liangwy 
 * @since 2022-5-16
 */
@Component
public class MyApplicationListener implements ApplicationListener {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        //事件处理
        if (event instanceof MyApplicationEvent){
            ((MyApplicationEvent) event).toDo();
        }
    }
}
```

#### ApplicationEvent

自定义事件

```java
/**
 * @author liangwy
 * @since 2022-5-16
 */

public class MyApplicationEvent extends ApplicationEvent {

    public String msg;
    /**
     * Create a new {@code ApplicationEvent}.
     *
     * @param source the object on which the event initially occurred or with
     *               which the event is associated (never {@code null})
     */
    public MyApplicationEvent(Object source) {
        super(source);
    }
	public MyApplicationEvent(Object source,String msg){
        super(source);
        this.msg = msg;
    }
    public void toDo(){
        System.out.println(msg);
    }
}
```



#### 测试

```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationMain.class);
applicationContext.getBeanFactory().registerCustomEditor(Date.class,TestPropertyEditorSupport.class);
applicationContext.publishEvent(new MyApplicationEvent("事件源","测试事件"));
```

**测试结果**

在控制台打印如下结果

> 测试事件





### SimpleApplicationEventMulticaster源码解读

`SimpleApplicationEventMulticaster`是spring中一个`ApplicationEventMulticaster`实现类。看`SimpleApplicationEventMulticaster`，看`multicastEvent`方法部分即可。

```java
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    //参试使用线程池执行
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
        if (executor != null) {
            //线程执行
            executor.execute(() -> invokeListener(listener, event));
        }
        else {
            //直接调用
            invokeListener(listener, event);
        }
    }
}
```

