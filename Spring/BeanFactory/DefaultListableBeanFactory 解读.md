# DefaultListableBeanFactory 解读

#### DefaultListableBeanFactory介绍

BeanFactory是个Factory，也就是IOC容器或对象工厂，而DefaultListableBeanFactory是Bean工厂的一个默认实现，DefaultListableBeanFactory提供了原始的BeanFactory的功能，如：对外提供getbean()方法，维护一张beanDefinitionMap表



#### 继承关系

DefaultListableBeanFactory继承关系如下，可以看出DefaultListableBeanFactory还有子类XmlBeanFactory

![DefaultListableBeanFactory继承关系](https://raw.githubusercontent.com/WenyaoL/blog-img/main/111.png)

#### 接口实现

下图列出了DefaultListableBeanFactory的接口实现和继承图，DefaultListableBeanFactory实现了BeanDefinitionRegistry 接口，并且实现BeanFactory接口，提供了基本的BeanFactory功能。

- BeanDefinitionRegistry 是一个接口，它定义了关于 BeanDefinition 的注册、移除、查询等一系列的操作。
- BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。

![接口实现](https://raw.githubusercontent.com/WenyaoL/blog-img/main/112.png)



### **源码码解析**

#### **属性成员**

##### beanDefinitionMap

DefaultListableBeanFactory作为BeanFactory默认是维护这一张beanDefinition的表。

```java
/** Map of bean definition objects, keyed by bean name. */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```

#### **方法解析**

##### getbean

`DefaultListableBeanFactory`是Bean工厂的一个默认实现，`DefaultListableBeanFactory`是Bean工厂的一个默认实现。截取`DefaultListableBeanFactory`中与getBean方法。

可以发现getBean有两种方式获取bean：

- 根据名称获取Bean，这个方法继承于`AbstractBeanFactory`
- 根据类型获取Bean，这个方法实现于`DefaultListableBeanFactory`，属于扩展了getBean方式

```java
    //---------------------------------------------------------------------
	// Implementation of remaining BeanFactory methods
	//---------------------------------------------------------------------
	//这个方式属于是扩展了getBean的方式，通过类型获取Bean
	@Override
	public <T> T getBean(Class<T> requiredType) throws BeansException {
		return getBean(requiredType, (Object[]) null);
	}

	@SuppressWarnings("unchecked")
	@Override
	public <T> T getBean(Class<T> requiredType, @Nullable Object... args) throws BeansException {
		Assert.notNull(requiredType, "Required type must not be null");
        //根据类型解析bean
		Object resolved = resolveBean(ResolvableType.forRawClass(requiredType), args, false);
		if (resolved == null) {
			throw new NoSuchBeanDefinitionException(requiredType);
		}
		return (T) resolved;
	}
```



##### setAutowireCandidateResolver

这是一个普通的set方法，为什么我还要介绍它呢，因为这个set方法能启动扩展`DefaultListableBeanFactory`的功能。通过该方法给`DefaultListableBeanFactory`扩展自动装配的解析器。

可以通过该方法添加格外的自动装配解析器，如：`QualifierAnnotationAutowireCandidateResolver`提供`@Qualifier`注解解析的功能和`@Autowire`

```java
/**
	 * Set a custom autowire candidate resolver for this BeanFactory to use
	 * when deciding whether a bean definition should be considered as a
	 * candidate for autowiring.
	 */
	public void setAutowireCandidateResolver(AutowireCandidateResolver autowireCandidateResolver) {
		Assert.notNull(autowireCandidateResolver, "AutowireCandidateResolver must not be null");
		if (autowireCandidateResolver instanceof BeanFactoryAware) {
			if (System.getSecurityManager() != null) {
				AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
					((BeanFactoryAware) autowireCandidateResolver).setBeanFactory(this);
					return null;
				}, getAccessControlContext());
			}
			else {
				((BeanFactoryAware) autowireCandidateResolver).setBeanFactory(this);
			}
		}
		this.autowireCandidateResolver = autowireCandidateResolver;
	}
```

