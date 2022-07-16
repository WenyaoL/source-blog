## spring中的bean的InitMethod方法

[TOC]



### InitMethod的执行时机

执行时机如下：

1. 属性填充
2. 各种Aware方法
3. BeanPostProcessor的postProcessBeforeInitialization方法
4. InitialzingBean的afterPropertiesSet方法
5. InitMethod方法
6. BeanPostProcessor的postProcessAfterInitialization方法

也就是说是夹在postProcessBeforeInitialization方法和postProcessAfterInitialization方法之间

这么说可能有点模糊，看源码，因为bean的整个初始化过程比较庞大，所以我只截取initMethod附近的代码解析。

打开`AbstractAutowireCapableBeanFactory.java`文件，找到方法`doCreateBean`方法，找到里面的一段代码。代码如下：

```java
// Initialize the bean instance.
Object exposedObject = bean;
try {
    //属性填充
    populateBean(beanName, mbd, instanceWrapper);
    if (exposedObject != null) {
        //执行bean初始化步骤（这里可以知道initMethod在属性填充后面）
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
}
```

点开`initializeBean(beanName, exposedObject, mbd)`代码，代码如下：

```java
/**
	 * Initialize the given bean instance, applying factory callbacks
	 * as well as init methods and bean post processors.
	 * <p>Called from {@link #createBean} for traditionally defined beans,
	 * and from {@link #initializeBean} for existing bean instances.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @return the initialized bean instance (potentially wrapped)
	 * @see BeanNameAware
	 * @see BeanClassLoaderAware
	 * @see BeanFactoryAware
	 * @see #applyBeanPostProcessorsBeforeInitialization
	 * @see #invokeInitMethods
	 * @see #applyBeanPostProcessorsAfterInitialization
	 */
//这里是bean的初始化步骤了
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged(new PrivilegedAction<Object>() {
            @Override
            public Object run() {
                //调用AwareMethod（如：BeanFactoryAware等）
                invokeAwareMethods(beanName, bean);
                return null;
            }
        }, getAccessControlContext());
    }
    else {
        //调用AwareMethod（如：BeanFactoryAware等）
        invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        //调用BeanPostProcessor的BeforeInitialization
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        //调用用户配置的init-Method方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
    }

    if (mbd == null || !mbd.isSynthetic()) {
        //调用BeanPostProcessor的AfterInitialization
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```

点开`invokeInitMethods`方法

```java
/**
	 * Give a bean a chance to react now all its properties are set,
	 * and a chance to know about its owning bean factory (this object).
	 * This means checking whether the bean implements InitializingBean or defines
	 * a custom init method, and invoking the necessary callback(s) if it does.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the merged bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @throws Throwable if thrown by init methods or by the invocation process
	 * @see #invokeCustomInitMethod
	 */
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
    throws Throwable {
	//这里会判断bean是否实现了InitializingBean
    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (logger.isTraceEnabled()) {
            logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
        }
        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                    //调用afterPropertiesSet
                    ((InitializingBean) bean).afterPropertiesSet();
                    return null;
                }, getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {
                throw pae.getException();
            }
        }
        else {
            //调用afterPropertiesSet
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }

    if (mbd != null && bean.getClass() != NullBean.class) {
        //获取initMethod方法名
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
            //这里才是真正的去调用initMethod
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```



### InitMethod的使用

#### 注解方式

##### 通过@PostConstruct，@PreDestroy

```java
/**
 * @author liangwy 
 * @since 2022-5-2
 */
public class TestInitMethod{
    
    //顺序：实例化-->initMethod-->destoryMethod
    public TestInitMethod(){
        System.out.println("执行类的实例化方法");
    }
    //该注解代表该方法是InitMethod
    @PostConstruct
    public void init(){
        System.out.println("执行bean的initMethod方法");
    }

    //该注解代码该方法是destoryMethod
    @PreDestroy
    public void destory(){
        System.out.println("执行bean的destoryMethod方法");
    }
}
```

##### 通过@Bean

```java
@Configuration
public class TestConfig {
    //也可以使用@bean中的属性指定方法
    @Bean(initMethod = "init",destroyMethod = "destory")
    public TestInitMethod testInitMethod(){
        return new TestInitMethod();
    }
}
```

#### 配置文件方式

```xml
<bean id="testInitMethod" class="com.wenyao.TestInitMethod" init-method="init"></bean>
```

