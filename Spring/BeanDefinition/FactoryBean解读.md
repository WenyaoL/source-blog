## FactoryBean解读

[TOC]



#### FactoryBean介绍

在Spring中有两种类型的bean，一种是普通Bean，一种是工厂Bean，即FactoryBean。

FactoryBean是一个工厂Bean，创建的bean是getObject方法返回的对象。一般用于创建比较复杂的bean。

当实例化Bean过程比较复杂，按照传统的方式，需要在`<bean>`中提供大量的配置信息。配置方法的灵活性受限，这时采用编码方式可能会得到一个简单的方案。这时spring官方提供了FactoryBean来解决这个问题。用户有实现FactoryBean即可按Java的编程逻辑去实现负责bean的实例化操作。



#### FactoryBean源码

FactoryBean接口没什么特别，主要是如何实现接口

```java
public interface FactoryBean<T> {

	
	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

	//获取bean对象
	@Nullable
	T getObject() throws Exception;

	//bean对象的类型
	@Nullable
	Class<?> getObjectType();

	//生产的bean是否为单例
	default boolean isSingleton() {
		return true;
	}

}
```



#### FactoryBean使用例子

```java
public class UserFactoryBean implements FactoryBean<User> {

    @Override
    public User getObject() throws Exception {
        return new User();
    }

    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

**在application.xml中注册一个UserFactoryBean**

```xml
<bean id="userFactory" class="com.yaliyao.pojo.testBeanPojo.UserFactoryBean"/>
```

**测试**

```java
    @Test
    public void testFactoryBean(){
        XmlBeanFactory xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("application.xml"));
        User user1 = xmlBeanFactory.getBean(User.class);
        User user2 = xmlBeanFactory.getBean(User.class);
        Object user3 = xmlBeanFactory.getBean("userFactory");
        //要获取FactoryBean本身，要在前面加&
        Object userFactory = xmlBeanFactory.getBean("&userFactory");
        
        System.out.println(user1);
        System.out.println(user2);
        System.out.println(user3);
        System.out.println(userFactory);
        System.out.println(user1==user2);
    }
```

**结果**

```
User(id=0, username=null, password=null)
User(id=0, username=null, password=null)
User(id=0, username=null, password=null)
com.yaliyao.pojo.testBeanPojo.UserFactoryBean@77846d2c
false
```

