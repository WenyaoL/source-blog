## GenericBeanDefinition及其子类解析

[TOC]



### GenericBeanDefinition介绍

GenericBeanDefinition是继承于抽象类AbstractBeanDefinition，相比AbstractBeanDefinition多了一个成员属性parentName。

GenericBeanDefinition通用的bean实现，自2.5以后新加入的bean文件配置属性定义类。



### GenericBeanDefinition子类介绍

GenericBeanDefinition有三个派生子类：

- ScannedGenericBeanDefinition
- ConfigurationPropertiesValueObjectBeanDefinition
- AnnotatedGenericBeanDefinition



![](https://raw.githubusercontent.com/WenyaoL/blog-img/main/115.png)

这三个派生子类的作用如下：

ScannedGenericBeanDefinition：存储@Component、@Service、@Controller等注解注释的类

ConfigurationPropertiesValueObjectBeanDefinition：存储@ConfigurationProperties等注解注释的类

AnnotatedGenericBeanDefinition：存储@Configuration注解注释的类



### GenericBeanDefinition源码

GenericBeanDefinition几乎所有实现都是依靠AbstractBeanDefinition，也就多了个成员属性parentName

```java
@SuppressWarnings("serial")
public class GenericBeanDefinition extends AbstractBeanDefinition {

	@Nullable
	private String parentName;


	/**
	 * Create a new GenericBeanDefinition, to be configured through its bean
	 * properties and configuration methods.
	 * @see #setBeanClass
	 * @see #setScope
	 * @see #setConstructorArgumentValues
	 * @see #setPropertyValues
	 */
	public GenericBeanDefinition() {
		super();
	}

	/**
	 * Create a new GenericBeanDefinition as deep copy of the given
	 * bean definition.
	 * @param original the original bean definition to copy from
	 */
	public GenericBeanDefinition(BeanDefinition original) {
		super(original);
	}


	@Override
	public void setParentName(@Nullable String parentName) {
		this.parentName = parentName;
	}

	@Override
	@Nullable
	public String getParentName() {
		return this.parentName;
	}


	@Override
	public AbstractBeanDefinition cloneBeanDefinition() {
		return new GenericBeanDefinition(this);
	}

	@Override
	public boolean equals(@Nullable Object other) {
		if (this == other) {
			return true;
		}
		if (!(other instanceof GenericBeanDefinition)) {
			return false;
		}
		GenericBeanDefinition that = (GenericBeanDefinition) other;
		return (ObjectUtils.nullSafeEquals(this.parentName, that.parentName) && super.equals(other));
	}

	@Override
	public String toString() {
		if (this.parentName != null) {
			return "Generic bean with parent '" + this.parentName + "': " + super.toString();
		}
		return "Generic bean: " + super.toString();
	}

}
```



### 获取GenericBeanDefinition

我们可以用XmlBeanFactory作为IOC容器，通过类路径中的application.xml文件去获取bean加载到IOC容器中，通过getBeanDefinition()方法去获取GenericBeanDefinition，并打印一下它有什么信息。

```java
	@Test
    public void testGenericBeanDefinition(){
        //创建IOC容器
        XmlBeanFactory xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("application.xml"));
        //获取bean
        ParentBean parentBean = (ParentBean)xmlBeanFactory.getBean("parentBean");
        ChildBean childBean = (ChildBean)xmlBeanFactory.getBean("childBean");
        System.out.println(parentBean);
        System.out.println(childBean);
		//获取BeanDefinition
        BeanDefinition parentbeanDefinition = xmlBeanFactory.getBeanDefinition("parentBean");
        BeanDefinition childbeanDefinition = xmlBeanFactory.getBeanDefinition("childBean");
        System.out.println(parentbeanDefinition);
        System.out.println(childbeanDefinition);
    }
```

结果

> 17:28:00.114 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 3 bean definitions from class path resource [application.xml]
> 17:28:00.118 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanFactory - Creating shared instance of singleton bean 'parentBean'
> 17:28:00.132 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanFactory - Creating shared instance of singleton bean 'childBean'
> ParentBean()
> ChildBean()
> Generic bean: class [com.yaliyao.pojo.testBeanPojo.ParentBean]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [application.xml]
> Generic bean with parent 'parentBean': class [com.yaliyao.pojo.testBeanPojo.ChildBean]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [application.xml]