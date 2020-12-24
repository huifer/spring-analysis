# Spring XmlReaderContext
- 类全路径: `org.springframework.beans.factory.xml.XmlReaderContext`







## 成员变量

```java
public class XmlReaderContext extends ReaderContext {
   /**
    * xml bean 定义读取器
    */
   private final XmlBeanDefinitionReader reader;

   /**
    * 命名空间处理器解析类
    */
   private final NamespaceHandlerResolver namespaceHandlerResolver;
}
```





在`XmlReaderContext`中所提供的方法全部是由两个成员变量所提供. 本身的可读性很好, 在这不对每个方法进行分析. 重点阅读下面两篇文章

1. [XmlBeanDefinitionReader](/docs/beans/factory/xml/Spring-XmlBeanDefinitionReader.md)
2. [NamespaceHandlerResolver](/docs/beans/factory/xml/NamespaceHandlerResolver/Spring-NamespaceHandlerResolver.md)







```java
public class XmlReaderContext extends ReaderContext {
   /**
    * xml bean 定义读取器
    */
   private final XmlBeanDefinitionReader reader;

   /**
    * 命名空间处理器解析类
    */
   private final NamespaceHandlerResolver namespaceHandlerResolver;


   /**
    * Construct a new {@code XmlReaderContext}.
    * @param resource the XML bean definition resource
    * @param problemReporter the problem reporter in use
    * @param eventListener the event listener in use
    * @param sourceExtractor the source extractor in use
    * @param reader the XML bean definition reader in use
    * @param namespaceHandlerResolver the XML namespace resolver
    */
   public XmlReaderContext(
         Resource resource, ProblemReporter problemReporter,
         ReaderEventListener eventListener, SourceExtractor sourceExtractor,
         XmlBeanDefinitionReader reader, NamespaceHandlerResolver namespaceHandlerResolver) {

      super(resource, problemReporter, eventListener, sourceExtractor);
      this.reader = reader;
      this.namespaceHandlerResolver = namespaceHandlerResolver;
   }


   /**
    * Return the XML bean definition reader in use.
    */
   public final XmlBeanDefinitionReader getReader() {
      return this.reader;
   }

   /**
    * Return the bean definition registry to use.
    * @see XmlBeanDefinitionReader#XmlBeanDefinitionReader(BeanDefinitionRegistry)
    */
   public final BeanDefinitionRegistry getRegistry() {
      return this.reader.getRegistry();
   }

   /**
    * Return the resource loader to use, if any.
    * <p>This will be non-null in regular scenarios,
    * also allowing access to the resource class loader.
    * @see XmlBeanDefinitionReader#setResourceLoader
    * @see ResourceLoader#getClassLoader()
    */
   @Nullable
   public final ResourceLoader getResourceLoader() {
      return this.reader.getResourceLoader();
   }

   /**
    * Return the bean class loader to use, if any.
    * <p>Note that this will be null in regular scenarios,
    * as an indication to lazily resolve bean classes.
    * @see XmlBeanDefinitionReader#setBeanClassLoader
    */
   @Nullable
   public final ClassLoader getBeanClassLoader() {
      return this.reader.getBeanClassLoader();
   }

   /**
    * Return the environment to use.
    * @see XmlBeanDefinitionReader#setEnvironment
    */
   public final Environment getEnvironment() {
      return this.reader.getEnvironment();
   }

   /**
    * Return the namespace resolver.
    * @see XmlBeanDefinitionReader#setNamespaceHandlerResolver
    */
   public final NamespaceHandlerResolver getNamespaceHandlerResolver() {
      return this.namespaceHandlerResolver;
   }


   // Convenience methods to delegate to

   /**
    * Call the bean name generator for the given bean definition.
    * @see XmlBeanDefinitionReader#getBeanNameGenerator()
    * @see org.springframework.beans.factory.support.BeanNameGenerator#generateBeanName
    */
   public String generateBeanName(BeanDefinition beanDefinition) {
      return this.reader.getBeanNameGenerator().generateBeanName(beanDefinition, getRegistry());
   }

   /**
    * Call the bean name generator for the given bean definition
    * and register the bean definition under the generated name.
    * @see XmlBeanDefinitionReader#getBeanNameGenerator()
    * @see org.springframework.beans.factory.support.BeanNameGenerator#generateBeanName
    * @see BeanDefinitionRegistry#registerBeanDefinition
    */
   public String registerWithGeneratedName(BeanDefinition beanDefinition) {
      String generatedName = generateBeanName(beanDefinition);
      getRegistry().registerBeanDefinition(generatedName, beanDefinition);
      return generatedName;
   }

   /**
    * Read an XML document from the given String.
    * @see #getReader()
    */
   public Document readDocumentFromString(String documentContent) {
      InputSource is = new InputSource(new StringReader(documentContent));
      try {
         return this.reader.doLoadDocument(is, getResource());
      }
      catch (Exception ex) {
         throw new BeanDefinitionStoreException("Failed to read XML document", ex);
      }
   }

}
```