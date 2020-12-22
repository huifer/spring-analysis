# Spring DocumentLoader 
- 全路径:`org.springframework.beans.factory.xml.DocumentLoader`
- 文档加载接口



- 接口说明如下

```java
/**
 * Strategy interface for loading an XML {@link Document}.
 *
 * xml 加载接口
 * @author Rob Harrop
 * @since 2.0
 * @see DefaultDocumentLoader
 */
public interface DocumentLoader {

   /**
    * Load a {@link Document document} from the supplied {@link InputSource source}.
    *
    * 从 inputSource 中读取 document 对象返回
    * @param inputSource the source of the document that is to be loaded
    * @param entityResolver the resolver that is to be used to resolve any entities
    * @param errorHandler used to report any errors during document loading
    * @param validationMode the type of validation
    * {@link org.springframework.util.xml.XmlValidationModeDetector#VALIDATION_DTD DTD}
    * or {@link org.springframework.util.xml.XmlValidationModeDetector#VALIDATION_XSD XSD})
    * @param namespaceAware {@code true} if support for XML namespaces is to be provided
    * @return the loaded {@link Document document}
    * @throws Exception if an error occurs
    */
   Document loadDocument(
         InputSource inputSource, EntityResolver entityResolver,
         ErrorHandler errorHandler, int validationMode, boolean namespaceAware)
         throws Exception;

}
```






## DefaultDocumentLoader

- 全路径: `org.springframework.beans.factory.xml.DefaultDocumentLoader`



- `loadDocument`方法的详细, 

  1. 创建出 `Document` 对象

     内部使用 javax 中 xml 相关方法的调用 

```java
@Override
public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
      ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {

   // 创建 xml document 构建工具
   DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);
   if (logger.isTraceEnabled()) {
      logger.trace("Using JAXP provider [" + factory.getClass().getName() + "]");
   }

   // documentBuilder 类创建
   DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);
   return builder.parse(inputSource);
}
```

