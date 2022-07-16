## ConfigurableBeanFactory解读

#### ConfigurableBeanFactory介绍

ConfigurableBeanFactory是一个接口，提供配置BeanFactory的各种方法，继承HierarchicalBeanFactory, SingletonBeanRegistry这两个接口。

提供了配置父容器，注册bean，添加BeanPostProcessor，设置Bean表达式分解器等功能

```java
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry
```



#### ConfigurableBeanFactory源码

ConfigurableBeanFactory源码翻译来自：[Spring源码分析——BeanFactory体系之接口详细分析 - Chandler Qian - 博客园 (cnblogs.com)](https://www.cnblogs.com/zrtqsk/p/4028453.html)

```java
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {

    String SCOPE_SINGLETON = "singleton";  //  单例

    String SCOPE_PROTOTYPE = "prototype";  //  原型

    /*
     * 搭配HierarchicalBeanFactory接口的getParentBeanFactory方法
     */
    void setParentBeanFactory(BeanFactory parentBeanFactory) throws IllegalStateException;

    /*
     * 设置、返回工厂的类加载器
     */
    void setBeanClassLoader(ClassLoader beanClassLoader);

    ClassLoader getBeanClassLoader();

    /*
     * 设置、返回一个临时的类加载器
     */
    void setTempClassLoader(ClassLoader tempClassLoader);

    ClassLoader getTempClassLoader();

    /*
     * 设置、是否缓存元数据，如果false，那么每次请求实例，都会从类加载器重新加载（热加载）

     */
    void setCacheBeanMetadata(boolean cacheBeanMetadata);
    
    boolean isCacheBeanMetadata();//是否缓存元数据

    /*
     * Bean表达式分解器
     */
    void setBeanExpressionResolver(BeanExpressionResolver resolver);
    
    BeanExpressionResolver getBeanExpressionResolver();

    /*
     * 设置、返回一个转换服务
     */
    void setConversionService(ConversionService conversionService);

    ConversionService getConversionService();

    /*
     * 设置属性编辑登记员...
     */
    void addPropertyEditorRegistrar(PropertyEditorRegistrar registrar);

    /*
     * 注册常用属性编辑器
     */
    void registerCustomEditor(Class<?> requiredType, Class<? extends PropertyEditor> propertyEditorClass);

    /*
     * 用工厂中注册的通用的编辑器初始化指定的属性编辑注册器
     */
    void copyRegisteredEditorsTo(PropertyEditorRegistry registry);

    /*
     * 设置、得到一个类型转换器
     */
    void setTypeConverter(TypeConverter typeConverter);

    TypeConverter getTypeConverter();

    /*
     * 增加一个嵌入式的StringValueResolver
     */
    void addEmbeddedValueResolver(StringValueResolver valueResolver);

    String resolveEmbeddedValue(String value);//分解指定的嵌入式的值

    void addBeanPostProcessor(BeanPostProcessor beanPostProcessor);//设置一个Bean后处理器

    int getBeanPostProcessorCount();//返回Bean后处理器的数量

    void registerScope(String scopeName, Scope scope);//注册范围

    String[] getRegisteredScopeNames();//返回注册的范围名

    Scope getRegisteredScope(String scopeName);//返回指定的范围

    AccessControlContext getAccessControlContext();//返回本工厂的一个安全访问上下文

    void copyConfigurationFrom(ConfigurableBeanFactory otherFactory);//从其他的工厂复制相关的所有配置

    /*
     * 给指定的Bean注册别名
     */
    void registerAlias(String beanName, String alias) throws BeanDefinitionStoreException;

    void resolveAliases(StringValueResolver valueResolver);//根据指定的StringValueResolver移除所有的别名

    /*
     * 返回指定Bean合并后的Bean定义
     */
    BeanDefinition getMergedBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

    boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException;//判断指定Bean是否为一个工厂Bean

    void setCurrentlyInCreation(String beanName, boolean inCreation);//设置一个Bean是否正在创建

    boolean isCurrentlyInCreation(String beanName);//返回指定Bean是否已经成功创建

    void registerDependentBean(String beanName, String dependentBeanName);//注册一个依赖于指定bean的Bean

    String[] getDependentBeans(String beanName);//返回依赖于指定Bean的所欲Bean名

    String[] getDependenciesForBean(String beanName);//返回指定Bean依赖的所有Bean名

    void destroyBean(String beanName, Object beanInstance);//销毁指定的Bean

    void destroyScopedBean(String beanName);//销毁指定的范围Bean

    void destroySingletons();  //销毁所有的单例类

}
```

