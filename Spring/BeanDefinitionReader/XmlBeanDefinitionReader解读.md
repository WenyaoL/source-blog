## XmlBeanDefinitionReader解读

[TOC]



### XmlBeanDefinitionReader介绍

`XmlBeanDefinitionReader`是`BeanDefinitionReader`三大读取器的其中一个，`XmlBeanDefinitionReader`继承于`AbstractBeanDefinitionReader`。`XmlBeanDefinitionReader`是为了读取Xml文件中的Bean而设计出来的。

`AbstractBeanDefinitionReader`内部有registry成员属性，理所当然可以往IOC容器里面添加BeanDefinition。`XmlBeanDefinitionReader`继承于`AbstractBeanDefinitionReader`，必定也能将BeanDefinition注册到容器里面。

**请记住这个AbstractBeanDefinitionReader中的成员属性，因为后面XmlBeanDefinitionReader会通过调用AbstractBeanDefinitionReader的getRegistry()拿到这个成员属性**

```java
//AbstractBeanDefinitionReader的私有成员属性
private final BeanDefinitionRegistry registry;

//AbstractBeanDefinitionReader中的公共方法
@Override
public final BeanDefinitionRegistry getRegistry() {
    return this.registry;
}
```



### DTD和XSD

- DTD（文档类型定义）的作用是定义 XML 文档的合法构建模块。
- XSD（XML Schema Definition）的作用是定义一份XML文档的合法组件群，就像文档类型定义（外语缩写：DTD）的作用一样。XML Schema 比 DTD 更强大。

其实两者都是用来定义和约束XML的结构信息的，如：有哪些节点，节点之间的结构关系等等。

我们平时的bean定义的xml文件里面的`xmlns:xsi , xsi:schemaLocation`就是XSD，用来约束xml文件结构。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```



### XmlBeanDefinitionReader源码

##### XmlBeanDefinitionReader一些关键成员属性

从注释就可以知道，这是xml验证模式的标志符，看看上面写的DTD和XSD大概就知道这几个变量的作用了。

```java
	/**
	 * Indicates that the validation should be disabled.
	 */
	//无检验
	public static final int VALIDATION_NONE = XmlValidationModeDetector.VALIDATION_NONE;

	/**
	 * Indicates that the validation mode should be detected automatically.
	 */
	//自动检测XML的检验模式
	public static final int VALIDATION_AUTO = XmlValidationModeDetector.VALIDATION_AUTO;

	/**
	 * Indicates that DTD validation should be used.
	 */
	//用DTD去进行校验
	public static final int VALIDATION_DTD = XmlValidationModeDetector.VALIDATION_DTD;

	/**
	 * Indicates that XSD validation should be used.
	 */
    //用XSD去进行校验
	public static final int VALIDATION_XSD = XmlValidationModeDetector.VALIDATION_XSD;
	//默认自动检测
	private int validationMode = VALIDATION_AUTO;
	//XML校验模式侦查器
	private final XmlValidationModeDetector validationModeDetector = new XmlValidationModeDetector();
```





##### loadBeanDefinitions(Resource resource)

实现BeanDefinitionReader接口的重要方法

```java
//这里是实现BeanDefinitionReader接口的重要方法
	@Override
	public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(new EncodedResource(resource));
	}

	//loadBeanDefinitions具体的实现方法
	public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
		Assert.notNull(encodedResource, "EncodedResource must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Loading XML bean definitions from " + encodedResource);
		}
		//已经加载的Resource
		Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();
		//判断是否重复加载
		if (!currentResources.add(encodedResource)) {
			throw new BeanDefinitionStoreException(
					"Detected cyclic loading of " + encodedResource + " - check your import definitions!");
		}
	
		try (InputStream inputStream = encodedResource.getResource().getInputStream()) {
            //封装成org.xml.sax.InputSource(xml的输入源)
			InputSource inputSource = new InputSource(inputStream);
			if (encodedResource.getEncoding() != null) {
				inputSource.setEncoding(encodedResource.getEncoding());
			}
            //实际加载方法
			return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(
					"IOException parsing XML document from " + encodedResource.getResource(), ex);
		}
		finally {
			currentResources.remove(encodedResource);
			if (currentResources.isEmpty()) {
				this.resourcesCurrentlyBeingLoaded.remove();
			}
		}
	}



```



##### doLoadBeanDefinitions(InputSource inputSource, Resource resource)

加载BeanDefinitions的实际方法。

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {

		try {
            //读取文件转成org.w3c.dom.Document(DOM树)
			Document doc = doLoadDocument(inputSource, resource);
            //注册BeanDefinitions
			int count = registerBeanDefinitions(doc, resource);
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + count + " bean definitions from " + resource);
			}
			return count;
		}
		//省去又臭又长的catch.....
	}
```



##### registerBeanDefinitions(Document doc, Resource resource)

虽然这里并不能直接告诉你BeanDefinitionDocumentReader怎么读取DOC中的bean，然后注册到IOC容器中的（因为做了很多层的封装，这里展开来读的话，已经脱离XmlBeanDefinitionReader的功能范围了）。最后还是会通过getRegistry()方法拿到Registry属性然后注册BeanDefinition

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
        //创建BeanDefinitionDocumentReader(专门读取DOM树解析成BeanDefinition的类)
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    
    	//getRegistry()获取IOC容器（如:BeanFactory,Application）
        //这里获取加载前数量
		int countBefore = getRegistry().getBeanDefinitionCount();
    
        //这里个方法做了很多doc的解析，最终会将BeanDefinitions加入到IOC容器中
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
        //返回加载数量
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}


//这里读一下上面的createReaderContext(resource)方法
public XmlReaderContext createReaderContext(Resource resource) {
    //这里传了个this，所以XmlReaderContext是里面是可以拿到IOC容器的
		return new XmlReaderContext(resource, this.problemReporter, this.eventListener,
				this.sourceExtractor, this, getNamespaceHandlerResolver());
	}
```

#####  最终分析

loadBeanDefinitions(Resource resource)最终会完成一下这些步骤：

- 判断Resource是否重复加载，并且获取输入流
- 从输入流中读取数据并生成DOM树（document对象）
- 通过BeanDefinitionDocumentReader将DOM树转为BeanDefinition并注入到IOC容器（通过registry这个成员属性）中