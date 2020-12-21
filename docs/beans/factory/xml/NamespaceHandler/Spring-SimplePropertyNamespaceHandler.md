# Spring SimplePropertyNamespaceHandler
- 类全路径: `org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler`
- `SimplePropertyNamespaceHandler`的主要作用是给 beanDefinition 的属性映射表添加属性. 主要由`decorate`方法进行. 下面开始进行方法分析


## 方法分析

### decorate
- 方法签名: `org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler.decorate`
- 方法作用: 补充  BeanDefinition 的 属性映射表信息
- 执行流程
    1. 判断Node是否是 Attr
    2. 解析出 Attr 的属性名称和属性值
    3. 放入 BeanDefinition 的 MutablePropertyValues 属性中
        放入情况分为三种
            1. BeanDefinition 中已经包含当前正在处理的属性名称, 此时抛出异常
            2. 属性名称结尾是`-ref`放入容器时会对 value进行转换, 即创建`RuntimeBeanReference`对象
            3. 直接将属性进行设置
       

- 更多信息请阅读下面代码

<details>
<summary>decorate 方法详情</summary>

```java
	@Override
	public BeanDefinitionHolder decorate(Node node, BeanDefinitionHolder definition, ParserContext parserContext) {
		// node 是否是  attr
		if (node instanceof Attr) {
			// 将 attr 转换成 属性名称 属性值
			Attr attr = (Attr) node;
			// 属性名称
			String propertyName = parserContext.getDelegate().getLocalName(attr);
			// 属性值
			String propertyValue = attr.getValue();
			// bean 定义中的属性映射表
			MutablePropertyValues pvs = definition.getBeanDefinition().getPropertyValues();
			// 如果属性映射吧已经存在当前的 属性名称 抛出异常
			if (pvs.contains(propertyName)) {
				parserContext.getReaderContext().error("Property '" + propertyName + "' is already defined using " +
						"both <property> and inline syntax. Only one approach may be used per property.", attr);
			}
			// 如果属性名称 结尾是-ref
			if (propertyName.endsWith(REF_SUFFIX)) {
				// 切分字符串得到 -ref 之前的内容
				propertyName = propertyName.substring(0, propertyName.length() - REF_SUFFIX.length());
				// 设置属性映射表中
				// key: 属性名称
				// value: RuntimeBeanReference
				pvs.add(Conventions.attributeNameToPropertyName(propertyName), new RuntimeBeanReference(propertyValue));
			}
			else {
				// 直接设置
				// key: 属性名称
				// value: 属性值
				pvs.add(Conventions.attributeNameToPropertyName(propertyName), propertyValue);
			}
		}
		return definition;
	}

```


</details>



### init
- 方法签名: `org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler.init`
- 空方法


```java
	@Override
	public void init() {
	}

```


### parse
- 方法签名: `org.springframework.beans.factory.xml.SimplePropertyNamespaceHandler.parse`
- 记录异常, 返回 null . 

```java
	@Override
	@Nullable
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		// 直接抛出异常
		parserContext.getReaderContext().error(
				"Class [" + getClass().getName() + "] does not support custom elements.", element);
		return null;
	}

```