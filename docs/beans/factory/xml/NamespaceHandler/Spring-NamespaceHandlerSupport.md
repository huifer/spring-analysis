# Spring NamespaceHandlerSupport
- 类全路径: `org.springframework.beans.factory.xml.NamespaceHandlerSupport`





- 首先阅读`NamespaceHandlerSupport`中的成员变量



## 成员变量

- `NamespaceHandlerSupport` 成员变量有3个. 

  1. parsers

     存储 XML 中 Element 和 BeanDefinitionParser 的对应关系

  2. decorators

     存储 XML 中 Element 和 BeanDefinitionDecorator 的对应关系

  3. attributeDecorators

     存储 XML 中 Attr 和 BeanDefinitionDecorator 的对应关系

  



<details>
    <summary>成员变量 </summary>







```java
public abstract class NamespaceHandlerSupport implements NamespaceHandler {

   /**
    * Stores the {@link BeanDefinitionParser} implementations keyed by the
    * local name of the {@link Element Elements} they handle.
    *
    * key: xml elementName
    * value: BeanDefinitionParser
    */
   private final Map<String, BeanDefinitionParser> parsers = new HashMap<>();

   /**
    * Stores the {@link BeanDefinitionDecorator} implementations keyed by the
    * local name of the {@link Element Elements} they handle.
    *
    * key: xml elementName
    * value: BeanDefinitionDecorator
    */
   private final Map<String, BeanDefinitionDecorator> decorators = new HashMap<>();

   /**
    * Stores the {@link BeanDefinitionDecorator} implementations keyed by the local
    * name of the {@link Attr Attrs} they handle.
    *
    * 存储有关 {@link Attr} 的 BeanDefinitionDecorator 容器
    *
    * key: attr
    * value: BeanDefinitionDecorator
    */
   private final Map<String, BeanDefinitionDecorator> attributeDecorators = new HashMap<>();
}
```

</details>







这些成员变量很重要. 在Spring中自定义标签的解析就需要依赖它们. 下面举几个例子



Aop

```java
@Override
public void init() {
   // In 2.0 XSD as well as in 2.1 XSD.
   registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
   // Spring AOP 解析
   registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
   registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

   // Only in 2.0 XSD: moved to context namespace as of 2.1
   registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
}
```



Jdbc

```java
@Override
public void init() {
   registerBeanDefinitionParser("embedded-database", new EmbeddedDatabaseBeanDefinitionParser());
   registerBeanDefinitionParser("initialize-database", new InitializeDatabaseBeanDefinitionParser());
}
```





tx



```java
@Override
public void init() {
   // 切面解析
   registerBeanDefinitionParser("advice", new TxAdviceBeanDefinitionParser());
   // 注解驱动
   registerBeanDefinitionParser("annotation-driven",
         new AnnotationDrivenBeanDefinitionParser());
   registerBeanDefinitionParser("jta-transaction-manager",
         new JtaTransactionManagerBeanDefinitionParser());
}
```







`NamespaceHandlerSupport` 的子类都会将自己需要的`BeanDefinitionParser`和`BeanDefinitionDecorator`注册(添加)到对应的容器中. 



下面我们对注册(`registerBeanDefinitionParser`)等方法进行分析





## 方法分析

- 笔者先将三种注册方式讲述. 下面介绍的三种注册方法本质上就是将数据放在map容器中即前文提到的三个变量. 操作方式很简单 `map#put`

### registerBeanDefinitionParser

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#registerBeanDefinitionParser`

```JAVA
protected final void registerBeanDefinitionParser(String elementName, BeanDefinitionParser parser) {
   this.parsers.put(elementName, parser);
}
```



### registerBeanDefinitionDecorator

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#registerBeanDefinitionDecorator`

```java
protected final void registerBeanDefinitionDecorator(String elementName, BeanDefinitionDecorator dec) {
   this.decorators.put(elementName, dec);
}
```



### registerBeanDefinitionDecoratorForAttribute

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#registerBeanDefinitionDecoratorForAttribute`



```JAVA
protected final void registerBeanDefinitionDecoratorForAttribute(String attrName, BeanDefinitionDecorator dec) {
   this.attributeDecorators.put(attrName, dec);
}
```





在了解注册方法后我们来看`NamespaceHandler`的实现方法







### parse

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#parse`

- 方法流程
  1. 根据参数 Element 找到 对应的 BeanDefinitionParser 对象 进行解析



```java
@Override
@Nullable
public BeanDefinition parse(Element element, ParserContext parserContext) {
   // 搜索 element 对应的  BeanDefinitionParser
   BeanDefinitionParser parser = findParserForElement(element, parserContext);
   // 解析
   return (parser != null ? parser.parse(element, parserContext) : null);
}
```

- 在这个方法中我们需要关注寻找`BeanDefinitionParser`的方法





### findParserForElement

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#findParserForElement`



- 寻找过程

  通过 Element 的名称直接从容器中获取





```java
@Nullable
private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
   // 获取 element 的名称
   String localName = parserContext.getDelegate().getLocalName(element);
   // 从容器中获取
   BeanDefinitionParser parser = this.parsers.get(localName);
   if (parser == null) {
      parserContext.getReaderContext().fatal(
            "Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
   }
   return parser;
}
```









### decorate

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#decorate`



```java
@Override
@Nullable
public BeanDefinitionHolder decorate(
      Node node, BeanDefinitionHolder definition, ParserContext parserContext) {
   // 根据 node 获取 BeanDefinitionDecorator
   BeanDefinitionDecorator decorator = findDecoratorForNode(node, parserContext);
   // 解析
   return (decorator != null ? decorator.decorate(node, definition, parserContext) : null);
}
```





- 阅读👆代码我们来思考 `findDecoratorForNode` 的流程

对于`BeanDefinitionDecorator` 在前文已经提到存储的容器有2个

1. decorators

   存储key类型是 element 

2. attributeDecorators

   存储key类型是 Attr 

根据上述两点我们大致可以推算出搜索方法. 



### findDecoratorForNode

- 方法签名: `org.springframework.beans.factory.xml.NamespaceHandlerSupport#findDecoratorForNode`



```java
@Nullable
private BeanDefinitionDecorator findDecoratorForNode(Node node, ParserContext parserContext) {
   BeanDefinitionDecorator decorator = null;
   String localName = parserContext.getDelegate().getLocalName(node);
   if (node instanceof Element) {
      decorator = this.decorators.get(localName);
   }
   else if (node instanceof Attr) {
      decorator = this.attributeDecorators.get(localName);
   }
   else {
      parserContext.getReaderContext().fatal(
            "Cannot decorate based on Nodes of type [" + node.getClass().getName() + "]", node);
   }
   if (decorator == null) {
      parserContext.getReaderContext().fatal("Cannot locate BeanDefinitionDecorator for " +
            (node instanceof Element ? "element" : "attribute") + " [" + localName + "]", node);
   }
   return decorator;
}
```







