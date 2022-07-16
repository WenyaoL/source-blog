## NamespaceHandler接口解析

[TOC]

**By 鸭梨的药丸哥**

### NamespaceHandler介绍

NamespaceHandler是spring中的标签处理器接口，用于扩展spring的标签解析能力，开发者可以通过实现自定义的NamespaceHandler来扩展标签解析能力。

开发者想实现自定义的`NamespaceHandler`，一般都是通过继承`NamespaceHandlerSupport`（一个实现了`NamespaceHandler`部分接口方法的抽象类）抽象类。

### NamespaceHandler接口方法解析

```java
public interface NamespaceHandler {

	//初始化函数
    //一般利用NamespaceHandlerSupport#registerBeanDefinitionParser注册解析器
	void init();

	//解析标签
	@Nullable
	BeanDefinition parse(Element element, ParserContext parserContext);

	//装饰BeanDefinitionHolder
	@Nullable
	BeanDefinitionHolder decorate(Node source, BeanDefinitionHolder definition, ParserContext parserContext);

}
```

### NamespaceHandler和BeanDefinitionParser的关系

- `NamespaceHandler`内部往往包含若干个`BeanDefinitionParser`
- `NamespaceHandler`注重某一命名空间的下标签集的解析，如：AOP命名空间下的所有标签
- `BeanDefinitionParser`注重某一标签的解析，如：AOP标签集下的aspect标签



### 参见实现类

`NamespaceHandler`有几个常见的实现类：

- `AopNamespaceHandler`：AOP命名空间下的xml标签进行处理
- `JdbcNamespaceHandler`：JDBC命名空间下的xml标签处理
- `MvcNamespaceHandler`：mvc命名空间下的xml标签处理
