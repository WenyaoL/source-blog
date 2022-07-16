## DefaultBeanDefinitionDocumentReader解读

[TOC]



### DefaultBeanDefinitionDocumentReader介绍

`DefaultBeanDefinitionDocumentReader`是`BeanDefinitionDocumentReader`的为数不多的实现类，是专门提取DOM树中的bean信息，并转化为BeanDefinition的一个实现类



### DefaultBeanDefinitionDocumentReader重点源码

#### registerBeanDefinitions

实现`BeanDefinitionDocumentReader`接口方法，其实具体实现在`doRegisterBeanDefinitions(doc.getDocumentElement())`

```java
@Override
	public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
        //具体注册操作
		doRegisterBeanDefinitions(doc.getDocumentElement());
	}
```



#### doRegisterBeanDefinitions

这个方法主要看后面那几句，改方法传入一个根节点，并对改根节点的解析做以下步骤：

- preProcessXml(root)   //根节点解析前处理
- parseBeanDefinitions(root, this.delegate)     //解析
- postProcessXml(root)    //解析后处理(留给子类实现)

```java
/**
	 * Register each bean definition within the given root {@code <beans/>} element.
	 */
	@SuppressWarnings("deprecation")  // for Environment.acceptsProfiles(String...)
	protected void doRegisterBeanDefinitions(Element root) {
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					if (logger.isDebugEnabled()) {
						logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
								"] not matching: " + getReaderContext().getResource());
					}
					return;
				}
			}
		}
		//解析前处理(留给子类实现)
		preProcessXml(root);
        //解析
		parseBeanDefinitions(root, this.delegate);
        //解析后处理(留给子类实现)
		postProcessXml(root);

		this.delegate = parent;
	}
```

#### parseBeanDefinitions

这个方法是拿出根节点的子节点，并对子节点列表遍历，然后正在解析节点元素的是parseDefaultElement，如果不了解Java对DOM的操作的化，建议先学Java dom，也就是org.w3c.dom（Java自带了）这个Java包。

```java
/**
	 * Parse the elements at the root level in the document:
	 * "import", "alias", "bean".
	 * @param root the DOM root element of the document
	 */
	protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		if (delegate.isDefaultNamespace(root)) {
            //获取子节点
			NodeList nl = root.getChildNodes();
			for (int i = 0; i < nl.getLength(); i++) {
                //遍历
				Node node = nl.item(i);
				if (node instanceof Element) {
                    //转化为Element元素
					Element ele = (Element) node;
                    //解析节点元素
					if (delegate.isDefaultNamespace(ele)) {
                        //解析Element元素
						parseDefaultElement(ele, delegate);
					}
					else {
						delegate.parseCustomElement(ele);
					}
				}
			}
		}
		else {
			delegate.parseCustomElement(root);
		}
	}
```

#### parseDefaultElement

这个是分类讨论分别判断import，alias，bean，beans几个节点元素的处理

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    	//如果是<import>,就去导入新的Resource
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}//如果是<alias>,就做别名注册
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}//如果是<bean>,就将element解析为BeanDefinition
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}//如果是<beans>,就递归调用doRegisterBeanDefinitions，递归解析
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}
```



#### processBeanDefinition

processBeanDefinition是真正解析`<bean>`并注册BeanDefinition到Registry（一个可以注册的IOC容器对象）中

```java
/**
	 * Process the given bean element, parsing the bean definition
	 * and registering it with the registry.
	 */
	protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// Register the final decorated instance.
                /**
                *使用工具类去做真正注册操作，getReaderContext().getRegistry()会获得要实现了
                *BeanDefinitionRegistry接口的IOC容器
                */
				BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				getReaderContext().error("Failed to register bean definition with name '" +
						bdHolder.getBeanName() + "'", ele, ex);
			}
			// Send registration event.
            //发布注册成功事件
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```





