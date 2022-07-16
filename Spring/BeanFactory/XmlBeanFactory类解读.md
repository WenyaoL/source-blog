## XmlBeanFactory类解读

#### XmlBeanFactory介绍

XmlBeanFactory继承于DefaultListableBeanFactory，拥有BeanFactory默认的所有功能。

XmlBeanFactory只是在DefaultListableBeanFactory的基础上添加了一个XmlBeanDefinitionReader（用以专门读取XML文件中的bean），所有XmlBeanFactory是一个专门读取xml文件中的bean而创建出来的BeanFactory。

并且XmlBeanFactory属于老旧的BeanFactory，已经被弃用。



#### XmlBeanFactory源码

```java
@Deprecated
@SuppressWarnings({"serial", "all"})
public class XmlBeanFactory extends DefaultListableBeanFactory {

	private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);


	/**
	 * 通过XmlBeanDefinitionReader加载XML资源文件
	 */
	public XmlBeanFactory(Resource resource) throws BeansException {
		this(resource, null);
	}

	/**
	 * 通过XmlBeanDefinitionReader加载XML资源文件
	 */
	public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
		super(parentBeanFactory);
		this.reader.loadBeanDefinitions(resource);
	}

}
```



#### XmlBeanFactory例子

```java
@SpringBootTest
public class TestXmlFactory {
    @Test
    public void testXmlFactory(){
        XmlBeanFactory xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("application.xml"));
        TestBean testBean = (TestBean)xmlBeanFactory.getBean("testBean");
        //TestBean的toString被我重写过
        System.out.println(testBean);
    }
}
```

结果展示

```
15:58:13.435 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 1 bean definitions from class path resource [application.xml]
15:58:13.439 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanFactory - Creating shared instance of singleton bean 'testBean'
这是测试bean
```



#### 例子结果分析

> 15:58:13.435 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 1 bean definitions from class path resource [application.xml]

**说明了bean definitions是通过XmlBeanDefinitionReader读取的**

> 15:58:13.439 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanFactory - Creating shared instance of singleton bean 'testBean'

**说明了bean是由XmlBeanFactory创建的**
