## BeanDefinitionReader接口解读

#### BeanDefinitionReader介绍

BeanDefinitionReader是一个BeanDefinition读取器的接口，实现了改接口的Reader可以从不同的资源文件中加载bean到beanFactory中。

```java
public interface BeanDefinitionReader {

	//返回BeanDefinitionRegistry(一般都是BeanFactory，因为BeanFactory的实现类基本都实现了BeanDefinitionRegistry)
	BeanDefinitionRegistry getRegistry();

	//返回资源加载器
	@Nullable
	ResourceLoader getResourceLoader();

	//返回BeanClass加载器
	@Nullable
	ClassLoader getBeanClassLoader();

	//返回BeanNameGenerator(未指定显式名称的Bean，用BeanNameGenerator生成名字)
	BeanNameGenerator getBeanNameGenerator();


	 //从指定的资源文件加载bean定义
	int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException;

	int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException;

	int loadBeanDefinitions(String location) throws BeanDefinitionStoreException;

	int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException;

}
```





#### BeanDefinitionReader的实现类

如图，BeanDefinitionReader主要的实现类有：XmlBeanDefinitionReader，GroovyBeanDefinitionReader，PropertiesBeanDefinitionReader，分别可以从xml文件，Properties文件和Groovy语言（一种基于[JVM](https://baike.baidu.com/item/JVM)（[Java虚拟机](https://baike.baidu.com/item/Java虚拟机)）的敏捷开发语言）定义的bean

![113](https://raw.githubusercontent.com/WenyaoL/blog-img/main/113.png)

