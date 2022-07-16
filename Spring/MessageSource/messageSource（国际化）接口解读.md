# spring中的messageSource（国际化）

[TOC]



### MessageSource介绍

messageSource是spring中的转换消息接口，提供了国际化信息的能力。`MessageSource`用于解析消息，并支持消息的参数化和国际化。 Spring 包含两个内置的`MessageSource`实现：`ResourceBundleMessageSource`和`ReloadableResourceBundleMessageSource`。



### 使用MessageSource做消息转换

#### 配置类

```java
/**
 * @author liangwy 
 * @since 2022-5-15
 */
@Configuration
public class MessageSourceConfig {

    @Bean
    public ResourceBundleMessageSource messageSource(){
        ResourceBundleMessageSource source = new ResourceBundleMessageSource();
        //设置基础名
        source.setBasenames("messages/message");
        //设置编码
        source.setDefaultEncoding("UTF-8");
        return source;
    }

}
```



#### 资源文件

`message_en.properties`

```
test=test
stringMsg=I am {0}
```

`message_zh.properties`

```
test=测试
stringMsg=我是{0}
```



#### 测试

```java
/**
 * @author liangwy
 * @since 2022-5-15
 */
@Component
@Slf4j
public class TestMessageSource {
    @Autowired
    private MessageSource messageSource;

    public void testMessageSource(){

        log.info("消息一:");
        log.info(messageSource.getMessage("test", null,Locale.CHINESE));
        log.info(messageSource.getMessage("test", null,Locale.ENGLISH));

        log.info("消息二:");
        log.info(messageSource.getMessage("stringMsg",new Object[]{"💊哥"},Locale.CHINESE));
        log.info(messageSource.getMessage("stringMsg",new Object[]{"wenyao"},Locale.CHINESE));
    }
}
```



#### 结果

```
[INFO ] 消息一:
[INFO ] 测试
[INFO ] test
[INFO ] 消息二:
[INFO ] 我是💊哥
[INFO ] 我是wenyao
```



## spring的AbstractApplicationContext中MessageSource初始化



### 初始化

在`AbstractApplicationContext`中有一个`refresh()`方法，在刷新方法里面调用一个`initMessageSource()`。`initMessageSource()`方法就是初始化上下文中的MessageSource资源国际化组件。

```java
	/**
	 * Initialize the MessageSource.
	 * Use parent's if none defined in this context.
	 */
protected void initMessageSource() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //获取容器中的MessageSource，spring中默认MessageSource bean的名字为messageSource
    //所有注入上下文中的MessageSource bean的名字必须为messageSource
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
        // Make MessageSource aware of parent MessageSource.
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                // Only set parent context as parent MessageSource if no parent MessageSource
                // registered already.
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Using MessageSource [" + this.messageSource + "]");
        }
    }
    else {
        // Use empty MessageSource to be able to accept getMessage calls.
        //如果容器中没有MessageSource，就使用DelegatingMessageSource(MessageSource的一个实现类)代替
        DelegatingMessageSource dms = new DelegatingMessageSource();
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        //注册到容器中，beanName是messageSource
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + MESSAGE_SOURCE_BEAN_NAME + "' bean, using [" + this.messageSource + "]");
        }
    }
}
```

### 使用容器中的messageSource

`AbstractApplicationContext`提供了接口`getMessage`方法来使用容器中的messagesource

```java
	//---------------------------------------------------------------------
	// Implementation of MessageSource interface
	//---------------------------------------------------------------------

	@Override
	public String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale) {
		return getMessageSource().getMessage(code, args, defaultMessage, locale);
	}

	@Override
	public String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException {
		return getMessageSource().getMessage(code, args, locale);
	}

	@Override
	public String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException {
		return getMessageSource().getMessage(resolvable, locale);
	}
```

