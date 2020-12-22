# Spring BeanDefinitionDocumentReader
- 类全路径: `org.springframework.beans.factory.xml.BeanDefinitionDocumentReader`


- `BeanDefinitionDocumentReader` 作为接口本文仅介绍其定义的方法作用. 直接看类


```java
public interface BeanDefinitionDocumentReader {

	/**
	 * Read bean definitions from the given DOM document and
	 * register them with the registry in the given reader context.
	 * 注册 bean 定义
	 * @param doc the DOM document
	 * @param readerContext the current context of the reader
	 * (includes the target registry and the resource being parsed)
	 * @throws BeanDefinitionStoreException in case of parsing errors
	 */
	void registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
			throws BeanDefinitionStoreException;

}

```


- 在方法中我们看到有两个参数
    1. Document
    2. XmlReaderContext
    其中 Document 属于 xml 标准接口, 本系列不会对其进行介绍, XmlReaderContext 是 Spring 开发的类, 在本系列中有介绍, 详细内容请看[这篇文章](/docs/beans/factory/xml/ReaderContext/Spring-XmlReaderContext.md)
       

- 实现类分析: [DefaultBeanDefinitionDocumentReader](/docs/beans/factory/xml/BeanDefinitionDocumentReader/Spring-DefaultBeanDefinitionDocumentReader.md)