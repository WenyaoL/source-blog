## ListableBeanFactory解读

[TOC]



#### ListableBeanFactory介绍

ListableBeanFactory是BeanFactory的子接口，除了BeanFactory的功能外，还格外定义一些能遍历和查找BeanDefinition的方法。

#### ListableBeanFactory源码

上网找了个翻译，其实也不用细看，就是一些遍历和查找BeanDefinition的方法

```java
public interface ListableBeanFactory extends BeanFactory {

    /**
     * 查看是否包含指定名字的Bean
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     */
    boolean containsBeanDefinition(String beanName);

    /**
     * 查看此BeanFactory中包含的Bean数量
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     */
    int getBeanDefinitionCount();

    /**
     * 返回此BeanFactory中所包含的所有Bean定义的名称
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     */
    String[] getBeanDefinitionNames();

    /**
     * 返回此BeanFactory中所有指定类型的Bean的名字。判断是否是指定类型的标准有两个：a Bean定义。
     * b FactoryBean的getObjectType方法。
     * 只考虑最顶层的Bean，对于嵌套的Bean，即使符合类型也不予考虑
     * 会考虑FactoryBean创建出的Bean
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     * 此签名的getBeanNamesForType方法会返回所有Scope类型的Bean，在大多数的实现中，其返回结果和
     * 其重载方法getBeanNamesForType(type, true, true)返回结果一致。
     * 返回结果中，Bean名字的顺序应该和其定义时一样
     */
    String[] getBeanNamesForType(ResolvableType type);

    /**
     * 返回此BeanFactory中所有指定类型（或指定类型的子类型）的Bean的名字。判断是否是指定类型的标准有
     * 两个：a Bean定义，b FactoryBean的getObjectType方法。
     * 只考虑最顶层的Bean，对于嵌套的Bean，即使符合类型也不予考虑
     * 会考虑FactoryBean创建出的Bean
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     * 此签名的getBeanNamesForType方法会返回所有Scope类型的Bean，在大多数的实现中，其返回结果和
     * 其重载方法getBeanNamesForType(type, true, true)返回结果一致。
     * 返回结果中，Bean名字的顺序应该和其定义时一样
     */
    String[] getBeanNamesForType(Class<?> type);

    /**
     * 返回此BeanFactory中所有指定类型（或指定类型的子类型）的Bean的名字。判断是否是指定类型的标准有
     * 两个：a Bean定义，b FactoryBean的getObjectType方法。
     * 只考虑最顶层的Bean，对于嵌套的Bean，即使符合类型也不予考虑
     * 会考虑FactoryBean创建出的Bean
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     * 此签名的getBeanNamesForType方法会返回所有Scope类型的Bean，在大多数的实现中，其返回结果和
     * 其重载方法getBeanNamesForType(type, true, true)返回结果一致。
     * 返回结果中，Bean名字的顺序应该和其定义时一样
     * 如果Bean是通过FactoryBean创建的，那么只考虑设置了allowEagerInit标志位的Bean。如果
     * 没有设置allowEagerInit标志位，则只考虑FactoryBean的类型
     */
    String[] getBeanNamesForType(Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);

    /**
     * 返回此BeanFactory中所有指定类型（或指定类型的子类型）的Bean的名字。判断是否是指定类型的标准有
     * 两个：a Bean定义，b FactoryBean的getObjectType方法。
     * 只考虑最顶层的Bean，对于嵌套的Bean，即使符合类型也不予考虑
     * 会考虑FactoryBean创建出的Bean
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     * 此签名的getBeanNamesForType方法会返回所有Scope类型的Bean，在大多数的实现中，其返回结果和
     * 其重载方法getBeanNamesForType(type, true, true)返回结果一致。
     * 返回结果中，Bean名字的顺序应该和其定义时一样
     */
    <T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException;

    /**
     * 返回此BeanFactory中所有指定类型（或指定类型的子类型）的Bean的名字。判断是否是指定类型的标准有
     * 两个：a Bean定义，b FactoryBean的getObjectType方法。
     * 只考虑最顶层的Bean，对于嵌套的Bean，即使符合类型也不予考虑
     * 会考虑FactoryBean创建出的Bean
     * 不支持向上或向下查找
     * 不支持查找非配置文件定义的单例Bean
     * 此签名的getBeanNamesForType方法会返回所有Scope类型的Bean，在大多数的实现中，其返回结果和
     * 其重载方法getBeanNamesForType(type, true, true)返回结果一致。
     * 返回结果中，Bean名字的顺序应该和其定义时一样
     * 如果Bean是通过FactoryBean创建的，那么只考虑设置了allowEagerInit标志位的Bean。如果
     * 没有设置allowEagerInit标志位，则只考虑FactoryBean的类型
     * @see BeanFactoryUtils#beansOfTypeIncludingAncestors(ListableBeanFactory, Class, boolean, boolean)
     */
    <T> Map<String, T> getBeansOfType(Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
            throws BeansException;

    /**
     * 找到所有带有指定注解类型的Bean
     */
    String[] getBeanNamesForAnnotation(Class<? extends Annotation> annotationType);

    /**
     * 找到所有带有指定注解的Bean，返回一个以Bean的name为键，其对应的Bean实例为值的Map
     */
    Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;

    /**
     * 在指定name对应的Bean上找指定的注解，如果没有找到的话，去指定Bean的父类或者父接口上查找
     */
    <A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType)
            throws NoSuchBeanDefinitionException;

}
```

