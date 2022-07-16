## AbstractBeanDefinitionReader解读

#### AbstractBeanDefinitionReader简介

AbstractBeanDefinitionReader是读取BeanDefinition一个抽象类，他已经实现了部分BeanDefinitionReader接口的部分方法，并且内部维护着一个成员变量registry，这个registry变量是BeanDefinitionRegistry接口的实现类对象，BeanDefinitionRegistry接口提供了关于 BeanDefinition 的注册、移除、查询等一系列的操作。



#### AbstractBeanDefinitionReader部分源码

我们主要读AbstractBeanDefinitionReader的成员属性和构造方法

```java
public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader, EnvironmentCapable {

   /** Logger available to subclasses. */
   protected final Log logger = LogFactory.getLog(getClass());
   
   //实现BeanDefinitionRegistry接口的通常都是IOC容器
   private final BeanDefinitionRegistry registry;
   
   //一些IOC容器自身还是ResourceLoader
   @Nullable
   private ResourceLoader resourceLoader;
   
   //加载bean用的类加载器
   @Nullable
   private ClassLoader beanClassLoader;
   
    //环境
   private Environment environment;
   
    //beanName生产器，没有bean名是会用该生成器生成
   private BeanNameGenerator beanNameGenerator = DefaultBeanNameGenerator.INSTANCE;
    
    
   protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		this.registry = registry;
		
		// Determine ResourceLoader to use.
       //如果IOC容器也实现了ResourceLoader，定义
		if (this.registry instanceof ResourceLoader) {
			this.resourceLoader = (ResourceLoader) this.registry;
		}
		else {
			this.resourceLoader = new PathMatchingResourcePatternResolver();
		}

		// Inherit Environment if possible
       //如果IOC容器中有environment
		if (this.registry instanceof EnvironmentCapable) {
			this.environment = ((EnvironmentCapable) this.registry).getEnvironment();
		}
		else {
			this.environment = new StandardEnvironment();
		}
	}
    
    //..............
    
}
```



