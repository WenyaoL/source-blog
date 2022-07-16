## spring中的AwareMethods

[TOC]



### 常用的AwareMethods

spring常用的AwareMethods如下：

- `BeanFactoryAware`：通过该Aware可以获取该bean关联的BeanFactor
- `BeanClassLoaderAware`：通过该Aware可以获取Bean的类加载器ClassLoader
- `BeanNameAware`：通过该Aware可以获取该Bean的名字
- `ApplicationContextAware`：通过该Aware可以获取该bean关联的ApplicationContext



### 代码示例

现在演示该三个接口Aware的使用

```java
/**
 * @author liangwy
 * @since 2022-5-2
 */
public class TestAwareMethods implements BeanFactoryAware, BeanClassLoaderAware, BeanNameAware, ApplicationContextAware {

    private BeanFactory beanFactory;
    private ClassLoader classLoader;
    private String beanName;
    private ApplicationContext applicationContext;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        this.classLoader = classLoader;
    }

    @Override
    public void setBeanName(String name) {
        this.beanName = name;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```



### spring中的激活Aware方法的源码

在`AbstractAutowireCapableBeanFactory.java`中有一个方法专门针对调用AwareMethods的方法，spring中有许多Aware接口，大多的大同小异。

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware) {
				((BeanNameAware) bean).setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware) {
				((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
			}
			if (bean instanceof BeanFactoryAware) {
				((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
```

### Aware方法的执行位置

**Aware方法在bean属性填充后，在调用InitMethods方法前**，也就是在Bean执行初始化时执行。

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
	protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged(new PrivilegedAction<Object>() {
				@Override
				public Object run() {
                    //调用AwareMethod
					invokeAwareMethods(beanName, bean);
					return null;
				}
			}, getAccessControlContext());
		}
		else {
            //调用AwareMethod
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

