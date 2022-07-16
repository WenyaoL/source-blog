## BeanDefinitionDocumentReader

#### BeanDefinitionDocumentReader介绍

BeanDefinitionDocumentReader是一个spring中定义解析DOM树中的bean信息转换为BeanDefinition的接口

BeanDefinitionDocumentReader会解析DOM树中的bean信息转换为BeanDefinition并注册到XmlReaderContext读取器上下文中的registry中。

#### BeanDefinitionDocumentReader源码

```java
public interface BeanDefinitionDocumentReader {

	/**
	 * Read bean definitions from the given DOM document and
	 * register them with the registry in the given reader context.
	 * @param doc the DOM document
	 * @param readerContext the current context of the reader
	 * (includes the target registry and the resource being parsed)
	 * @throws BeanDefinitionStoreException in case of parsing errors
	 */
    //解析DOM树，并将bean definitions注册到readerContext中的registry
	void registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
			throws BeanDefinitionStoreException;

}
```

