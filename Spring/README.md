**该模块是spring源码阅读模块。若觉得源码分析文章不错，欢迎Star⭐哦。**



## spring版本

**spring-context:5.2.9.RELEASE**



## 模块介绍

- **Resource**模块：介绍了`spring`的资源模块是如何定义资源的，如文件系统资源使用`FileSystemResource`来定义。
- **ResourceLoader**模块：介绍了`spring`的资源加载模块是如何对资源进行加载，是如何通过给定的字符串去加载对应的资源，并将资源以Resource对象返回。
- **BeanDefinition**模块：介绍了`spring`是如何将一个`bean`的定义和描述信息封装成一个`BeanDefinition`对象。
- **BeanDefinitionReader**模块：介绍了`spring`的`BeanDefinition读取器`模块是如何从资源（文件）中提取出bean的描述信息，并封装到`BeanDefinition`中，然后将`BeanDefinition`放到一张`Map`表中保存的。
- **BeanFactory**模块：介绍了`spring`的一些基础IOC容器，是如何通过`BeanDefinitionReader`读取`BeanDefinition`，如何如何通过`BeanDefinition`生产JavaBean对象等
- **ApplicationContext**模块：介绍了spring的一些上下文`ApplicationContext`接口的实现类，是如何在`BeanFactory`的基础上添加更多的新功能，从而封装成`ApplicationContext`的。
- **ApplicationEventMulticaster**模块：介绍了spring中的事件发布器，是如何发布事件信息的。
- **NamespaceHandler**模块：介绍了spring是如何处理用户自定义的标签命名的（xml文件中的标签）。
- **MessageSource**模块：介绍了spring的国际化功能（语音转换）
- **spring中的扩展接口**：介绍了一些spring的扩展接口，如：initMethod，BeanPostProcessor，各类的AwareMethods
- **spring中的通用服务**：介绍了一些spring中的容器服务，如：Lifecycle（生命循环服务）等





### 资源目录

spring资源模块Resource解读：[Resource](https://github.com/WenyaoL/source-blog/tree/main/Spring/Resource)

spring上下文模块ApplicationContext解读：[ApplicationContext](https://github.com/WenyaoL/source-blog/tree/main/Spring/ApplicationContext)