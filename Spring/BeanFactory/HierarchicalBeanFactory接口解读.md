## HierarchicalBeanFactory接口解读

[TOC]

#### HierarchicalBeanFactory介绍

HierarchicalBeanFactory是BeanFactory接口的子接口，也是BeanFactory三大直系接口之一，主要是为了实现了Bean工厂的分层。



#### HierarchicalBeanFactory源码

HierarchicalBeanFactory十分简单，就是在BeanFactory的基础上加了父容器的访问功能，可通过 ConfigurableBeanFactory 的 setParentBeanFactory 方法设置父容器，主要是为了实现了Bean工厂的分层。

```java
public interface HierarchicalBeanFactory extends BeanFactory {

	/**
	 * Return the parent bean factory, or {@code null} if there is none.
	 */
	@Nullable
	BeanFactory getParentBeanFactory();

	/**
	 * Return whether the local bean factory contains a bean of the given name,
	 * ignoring beans defined in ancestor contexts.
	 * <p>This is an alternative to {@code containsBean}, ignoring a bean
	 * of the given name from an ancestor bean factory.
	 * @param name the name of the bean to query
	 * @return whether a bean with the given name is defined in the local factory
	 * @see BeanFactory#containsBean
	 */
	boolean containsLocalBean(String name);

}
```