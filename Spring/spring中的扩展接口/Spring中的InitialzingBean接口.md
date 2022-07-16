## Spring中的InitialzingBean接口

### InitialzingBean执行时机

InitialzingBean的afterPropertiesSet执行时机在InitMethod方法之前。

源码解析如下：

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

### 使用案例

实现`InitializingBean`接口的去实现自定义的初始化业务逻辑

```java
/**
 * @author liangwy
 * @since 2022-5-3
 */
public class TestInitializingBean implements InitializingBean {


    /**
     * 使用afterPropertiesSet，在调用InitMethod实现自定义的初始化业务逻辑
     * @throws Exception
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("该方法会在initMethod前调用");
        System.out.println("执行自定义业务逻辑");
    }
}
```

