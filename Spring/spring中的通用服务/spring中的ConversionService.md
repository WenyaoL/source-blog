# spring中的ConversionService

[TOC]



### ConversionService介绍

ConversionService是spring中的一个转换服务，可以通过转换服务进行数据类型的转换，如：String转换为Date。

### 示例

#### ConversionService

```java
/**
 * @author liangwy 
 * @since 2022-5-16
 */
@Configuration
public class ConverterServiceConfig {

    /**
     * 注入工厂bean
     * @return
     */
    @Bean
    public ConversionServiceFactoryBean conversionService(){
        ConversionServiceFactoryBean conversionServiceFactoryBean = new ConversionServiceFactoryBean();
        HashSet<Object> objects = new HashSet<>();
        objects.add(new MyConverter());
        conversionServiceFactoryBean.setConverters(objects);
        return conversionServiceFactoryBean;
    }
}
```



#### Converter

```java
/**
 * @author liangwy 
 * @since 2022-5-16
 */
@Component
public class MyConverter implements Converter<String, Date> {
    /**
     * 格式化日期字符串
     * @param source
     * @return
     */
    @Override
    public Date convert(String source) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        try {
            return simpleDateFormat.parse(source);
        } catch (ParseException e) {
            return null;
        }
    }
}
```



#### 测试

```java
public static void main(String[] args) {

    AnnotationConfigApplicationContext applicationContext = runAnnotationConfigApplicationContext();
    //方式一
    ConversionService conversionService = applicationContext.getBeanFactory().getConversionService();
    //方式二
    //DefaultConversionService conversionService = applicationContext.getBean("conversionService", DefaultConversionService.class);
    Date convert = conversionService.convert("2022-05-16 15:58:00", Date.class);
    System.out.println(convert);

}
```

**结果打印**

> Mon May 16 15:58:00 CST 2022





### 源码分析

在applicationContext的BeanFactory初始完整时，会调用`finishBeanFactoryInitialization()`方法，该方法中有一步操作会将beanFactory中的`ConversionService bean`放到成员属性中。

```java
/**
	 * Finish the initialization of this context's bean factory,
	 * initializing all remaining singleton beans.
	 */
	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// Initialize conversion service for this context.
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
            //设置ConversionService
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// Register a default embedded value resolver if no bean post-processor
		// (such as a PropertyPlaceholderConfigurer bean) registered any before:
		// at this point, primarily for resolution in annotation attribute values.
		if (!beanFactory.hasEmbeddedValueResolver()) {
			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			getBean(weaverAwareName);
		}

		// Stop using the temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(null);

		// Allow for caching all bean definition metadata, not expecting further changes.
		beanFactory.freezeConfiguration();

		// Instantiate all remaining (non-lazy-init) singletons.
		beanFactory.preInstantiateSingletons();
	}
```

