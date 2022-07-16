## Spring中的BeanPostProcessor

[TOC]



### BeanPostProcessor接口使用代码示例

```java
/**
 * @author liangwy 
 * @since 2022-5-2
 */
public class TestBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        //执行位置在init-Method方法前
        System.out.println("执行postProcessBeforeInitialization方法");
        //对Bean进行改造
        System.out.println("改造Bean对象");
        //返回对象
        System.out.println("返回改造后的Bean对象");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        //执行位置在init-Method方法后
        System.out.println("执行postProcessAfterInitialization方法");
        //对Bean进行改造
        System.out.println("改造Bean对象");
        //返回对象
        System.out.println("返回改造后的Bean对象");
        return bean;
    }
}
```

### BeanPostProcessor执行时机

**BeanPostProcessor的两个方法的执行时机分别是在bean的init-Method的前后**

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

### BeanPostProcessor作用

**常常在生成动态代理的时候会实现该BeanPostProcessor接口方法**

查看`AbstractAutoProxyCreator.java`中就实现了该接口方法用来实现生成动态代理类。

```java
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) {
		return bean;
	}

	/**
	 * Create a proxy with the configured interceptors if the bean is
	 * identified as one to proxy by the subclass.
	 * @see #getAdvicesAndAdvisorsForBean
	 */
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        //AbstractAutoProxyCreator就实现了BeanPostProcessor接口
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (!this.earlyProxyReferences.contains(cacheKey)) {
                //这里就是封装代理类了
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}



/**
	 * Wrap the given bean if necessary, i.e. if it is eligible for being proxied.
	 * @param bean the raw bean instance
	 * @param beanName the name of the bean
	 * @param cacheKey the cache key for metadata access
	 * @return a proxy wrapping the bean, or the raw bean instance as-is
	 * 这里注释说明了该方法要么返回代理类，要么生成原生的bean
	 */
	protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
		if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
			return bean;
		}
		if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
			return bean;
		}
		if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
			this.advisedBeans.put(cacheKey, Boolean.FALSE);
			return bean;
		}
		//如果有advice就生成代理类（Spirng AOP中的advice）
		// Create proxy if we have advice.
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
		if (specificInterceptors != DO_NOT_PROXY) {
			this.advisedBeans.put(cacheKey, Boolean.TRUE);
            //生成代理类
			Object proxy = createProxy(bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
			this.proxyTypes.put(cacheKey, proxy.getClass());
			return proxy;
		}
		
		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}
```



### 注册BeanPostProcessor

#### 配置文件方式

```xml
<!--注册BeanPostProcessor-->
<bean class="com.wenyao.springExtend.TestBeanPostProcessor"/>
```

#### 注解方式

使用`@Component`注解注入进上下文即可，因为BeanPostProcessor本质上也是一个bean。

#### 编码的方式

```java
/**
 * @author liangwy 
 * @since 2022-5-4
 */
public class ApplicationMain {
    public static void main(String[] args) {
        String resource = "springContext.xml";
        ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext(resource);
        classPathXmlApplicationContext.getBeanFactory().addBeanPostProcessor(new TestBeanPostProcessor());
    }
}
```

