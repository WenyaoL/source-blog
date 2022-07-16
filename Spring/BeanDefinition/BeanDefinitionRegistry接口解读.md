## BeanDefinitionRegistry接口解读

BeanDefinitionRegistry 是一个接口，它定义了关于 BeanDefinition 的注册、移除、查询等一系列的操作。

基本所有BeanFactory接口的实现类都会实现BeanDefinitionRegistry接口，因为要对外提供BeanDefinition 的注册、移除、查询等一系列的操作。

基本所有BeanFactory接口的实现类都默认会维护一张beanDefinitionMap表，并且实现BeanDefinitionRegistry接口对这张表进行操作。

```java
public interface BeanDefinitionRegistry extends AliasRegistry {

	// 注册 BeanDefinition
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) throws BeanDefinitionStoreException;

	// 移除 BeanDefinition
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	// 获取 BeanDefinition
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	// 根据 beanName 判断容器是否存在对应的 BeanDefinition 
	boolean containsBeanDefinition(String beanName);

	// 获取所有的 BeanDefinition
	String[] getBeanDefinitionNames();

	// 获取 BeanDefinition 数量
	int getBeanDefinitionCount();

	// 判断 beanName 是否被占用
	boolean isBeanNameInUse(String beanName);
}
```

