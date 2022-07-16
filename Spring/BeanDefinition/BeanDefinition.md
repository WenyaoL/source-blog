## BeanDefinition



### BeanDefinition介绍

BeanDefinition是一个描述了 Bean 实例，实例包含属性值、构造方法参数值以及更多实现信息接口。主要提供描述bean和修改bean的信息的一个接口对象。

一般情况下BeanDefinition对象会在BeanDefinitionReader读取资源文件时生成并注入到IOC容器中（BeanFactory，ApplicationContext）。如：XmlBeanDefinitionReader，就会读取Xml文件并将解析xml文件中的Bean的配置信息转换为BeanDefinition，并注册到XmlBeanDefinitionReader中注册的IOC容器。





### BeanDefinition核心源码

这里的核心代码解读来源于：[Spring（四）核心容器 - BeanDefinition 解析 - 龙四丶 - 博客园 (cnblogs.com)](https://www.cnblogs.com/loongk/p/12262101.html)

这里主要解析一下BeanDefinition主要是记录了哪些描述信息：

- bean的作用域
- bean的父类
- bean的依赖关系
- bean是否懒加载
- bean是否可以自动注入
- bean是否为候选bean
- bean的FactoryBean（生成该bean的工厂类）
- bean的FactoryBean方法（生成该bean的工厂类的方法）
- ......等等



```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

	// 单例、原型标识符
	String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
	String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;

    // 标识 Bean 的类别，分别对应 用户定义的 Bean、来源于配置文件的 Bean、Spring 内部的 Bean
	int ROLE_APPLICATION = 0;
	int ROLE_SUPPORT = 1;
	int ROLE_INFRASTRUCTURE = 2;

    // 设置、返回 Bean 的父类名称
	void setParentName(@Nullable String parentName);
	String getParentName();

    // 设置、返回 Bean 的 className
	void setBeanClassName(@Nullable String beanClassName);
	String getBeanClassName();

    // 设置、返回 Bean 的作用域
	void setScope(@Nullable String scope);
	String getScope();

    // 设置、返回 Bean 是否懒加载
	void setLazyInit(boolean lazyInit);
	boolean isLazyInit();
	
	// 设置、返回当前 Bean 所依赖的其它 Bean 名称。
	void setDependsOn(@Nullable String... dependsOn);
	String[] getDependsOn();
	
	// 设置、返回 Bean 是否可以自动注入。只对 @Autowired 注解有效
	void setAutowireCandidate(boolean autowireCandidate);
	boolean isAutowireCandidate();
	
	// 设置、返回当前 Bean 是否为主要候选 Bean 。
	// 当同一个接口有多个实现类时，通过该属性来配置某个 Bean 为主候选 Bean。
	void setPrimary(boolean primary);
	boolean isPrimary();

    // 设置、返回创建该 Bean 的工厂类。
	void setFactoryBeanName(@Nullable String factoryBeanName);
	String getFactoryBeanName();
	
	// 设置、返回创建该 Bean 的工厂方法
	void setFactoryMethodName(@Nullable String factoryMethodName);
	String getFactoryMethodName();
	
	// 返回该 Bean 构造方法参数值、所有属性
	ConstructorArgumentValues getConstructorArgumentValues();
	MutablePropertyValues getPropertyValues();

    // 返回该 Bean 是否是单例、是否是非单例、是否是抽象的
	boolean isSingleton();
	boolean isPrototype();
	boolean isAbstract();

    // 返回 Bean 的类别。类别对应上面的三个属性值。
	int getRole();

    ...
}

```

