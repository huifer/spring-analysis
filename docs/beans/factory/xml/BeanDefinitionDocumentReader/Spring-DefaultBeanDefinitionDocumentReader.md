# Spring DefaultBeanDefinitionDocumentReader
- 类全路径: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader`
- Spring 中 默认的读取bean定义的类





先来看成员变量



## 成员变量

<details>
    <summary>DefaultBeanDefinitionDocumentReader 成员变量</summary>





```java
public class DefaultBeanDefinitionDocumentReader  implements BeanDefinitionDocumentReader {

   /**
    * bean 元素标签
    */
   public static final String BEAN_ELEMENT = BeanDefinitionParserDelegate.BEAN_ELEMENT;


   public static final String NESTED_BEANS_ELEMENT = "beans";


   public static final String ALIAS_ELEMENT = "alias";


   public static final String NAME_ATTRIBUTE = "name";


   public static final String ALIAS_ATTRIBUTE = "alias";


   public static final String IMPORT_ELEMENT = "import";


   public static final String RESOURCE_ATTRIBUTE = "resource";


   public static final String PROFILE_ATTRIBUTE = "profile";

	protected final Log logger = LogFactory.getLog(getClass());

	/**
	 * xml读取器的上下文
	 */
	@Nullable
	private XmlReaderContext readerContext;

	/**
	 * bean 定义解析委托对象
	 */
	@Nullable
	private BeanDefinitionParserDelegate delegate;

}
```



</details>





在成员变量中定义了 Spring 所支持的 xml 标签名称,属性. 详细如下

1. bean
2. beans
3. alias
4. name
5. alias
6. import
7. resource
8. profile



在提供了标签名称和属性意外还提供了两个类

1. XmlReaderContext： xml读取器上下文

   [分析文章](/docs/beans/factory/xml/ReaderContext/Spring-XmlReaderContext.md)

2. BeanDefinitionParserDelegate：bean定义解析委托对象

   [分析文章](/docs/beans/factory/xml/Spring-BeanDefinitionParserDelegate.md)







在了解过成员变量的意义后我们来了解方法. 

 

## 方法分析

### registerBeanDefinitions

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#registerBeanDefinitions`



```java
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
   this.readerContext = readerContext;
   doRegisterBeanDefinitions(doc.getDocumentElement());
}
```



这个方法是 Spring Ioc 读取xml文件将bean信息存储在容器中的核心方法. 下面我们来进一步了解方法`doRegisterBeanDefinitions`(正真的将XmlDoc注册到IoC容器的方法)





### doRegisterBeanDefinitions

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions`



<details>
    <summary>doRegisterBeanDefinitions 方法详情</summary>







```java
@SuppressWarnings("deprecation")  // for Environment.acceptsProfiles(String...)
protected void doRegisterBeanDefinitions(Element root) {
   // Any nested <beans> elements will cause recursion in this method. In
   // order to propagate and preserve <beans> default-* attributes correctly,
   // keep track of the current (parent) delegate, which may be null. Create
   // the new (child) delegate with a reference to the parent for fallback purposes,
   // then ultimately reset this.delegate back to its original (parent) reference.
   // this behavior emulates a stack of delegates without actually necessitating one.


   // 父 BeanDefinitionParserDelegate 一开始为null
   BeanDefinitionParserDelegate parent = this.delegate;
   // 创建 BeanDefinitionParserDelegate
   this.delegate = createDelegate(getReaderContext(), root, parent);

   // 判断命名空间是否为默认的命名空间
   // 默认命名空间: http://www.springframework.org/schema/beans
   if (this.delegate.isDefaultNamespace(root)) {
      // 获取 profile 属性
      String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
      // 是否存在 profile
      if (StringUtils.hasText(profileSpec)) {
         // profile 切分后的数据
         String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
               profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
         // We cannot use Profiles.of(...) since profile expressions are not supported
         // in XML config. See SPR-12458 for details.
         if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
            if (logger.isDebugEnabled()) {
               logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
                     "] not matching: " + getReaderContext().getResource());
            }
            return;
         }
      }
   }

   // 前置处理
   preProcessXml(root);
   // bean definition 处理
   parseBeanDefinitions(root, this.delegate);
   // 后置 xml 处理
   postProcessXml(root);

   this.delegate = parent;
}
```

</details>



`doRegisterBeanDefinitions`的主线方法只有四个

1. `createDelegate` 创建 `BeanDefinitionParserDelegate` 
2. `preProcessXml` 前置处理
3. `parseBeanDefinitions` beanDefinition 解析
4. `postProcessXml` 后置处理





在第一步和第二步之间还有一段关于命名空间的处理. 逻辑如下

1. 获取当前解析的文档(`Element`)的namespace_url 判断是否是默认的命名空间(默认命名空间地址:  http://www.springframework.org/schema/beans)

   是的情况下

   ​	是否包含 `profile` 属性

   ​		包含

   ​			是否包含多个, 如果包含多个判断是否是激活的

   ​			`!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)`







在四个方法中 `preProcessXml` 和 `postProcessXml` 属于预留方法. 



下面我们来看 `createDelegate` 方法





### createDelegate

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#createDelegate`



```java
protected BeanDefinitionParserDelegate createDelegate(
      XmlReaderContext readerContext, Element root, @Nullable BeanDefinitionParserDelegate parentDelegate) {

   // 创建对象 BeanDefinitionParserDelegate
   BeanDefinitionParserDelegate delegate = new BeanDefinitionParserDelegate(readerContext);
   // 设置默认值数据
   delegate.initDefaults(root, parentDelegate);
   return delegate;
}
```



创建中两个方法

1. 构造函数
2. 调用`initDefaults`方法



详细分析查看[BeanDefinitionParserDelegate](/docs/beans/factory/xml/Spring-BeanDefinitionParserDelegate.md)





下面开始核心方法`parseBeanDefinitions`的分析





### parseBeanDefinitions

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseBeanDefinitions`



<details>
    <summary>parseBeanDefinitions 方法详情</summary>







```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   // 是否是默认的命名空间
   if (delegate.isDefaultNamespace(root)) {
      // 子节点列表
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (node instanceof Element) {
            Element ele = (Element) node;
            // 是否是默认的命名空间
            if (delegate.isDefaultNamespace(ele)) {
               // 处理标签的方法
               parseDefaultElement(ele, delegate);
            }
            else {
               // 处理自定义标签
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
      // 处理自定义标签
      delegate.parseCustomElement(root);
   }
}
```



</details>





`parseBeanDefinitions` 方法处理 root 节点下的所有子节点. 对于节点处理在Spring中提供了两种

1. spring 默认支持的

   ```java
   parseDefaultElement(ele, delegate)
   ```

2. 自定义的

   ```java
   delegate.parseCustomElement(ele)
   ```



自定义标签解析需要实现下面3个接口

1. `org.springframework.beans.factory.xml.NamespaceHandler`
2. `org.springframework.beans.factory.xml.NamespaceHandlerResolver`
3. `org.springframework.beans.factory.xml.BeanDefinitionParser`





### parseDefaultElement

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#parseDefaultElement`



`parseDefaultElement` 中处理四个标签

1. import

   处理方法: `importBeanDefinitionResource`

2. alias

   处理方法: `processAliasRegistration`

3. bean

   处理方法: `processBeanDefinition`

4. beans

   处理方法: `doRegisterBeanDefinitions`

   递归处理



<details>
    <summary>parseDefaultElement 方法详情</summary>





```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
   // 解析 import 标签
   if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
      importBeanDefinitionResource(ele);
   }
   // 解析 alias 标签
   else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
      processAliasRegistration(ele);
   }
   // 解析 bean 标签
   else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
      processBeanDefinition(ele, delegate);
   }
   // 解析 beans 标签
   // 嵌套的 beans
   else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
      // recurse
      doRegisterBeanDefinitions(ele);
   }
}
```





</details>





### importBeanDefinitionResource

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#importBeanDefinitionResource`

方法相对来说比较庞大, 我们先看第一部分代码



<details>
    <summary>importBeanDefinitionResource 第一部分代码</summary>





```java
// 获取 resource 属性
String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
// 是否存在地址
if (!StringUtils.hasText(location)) {
   getReaderContext().error("Resource location must not be empty", ele);
   return;
}

// Resolve system properties: e.g. "${user.dir}"
// 处理配置文件占位符
location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

// 资源集合
Set<Resource> actualResources = new LinkedHashSet<>(4);

// Discover whether the location is an absolute or relative URI
// 是不是绝对地址
boolean absoluteLocation = false;
try {
   // 1. 判断是否为 url
   // 2. 通过转换成URI判断是否是绝对地址
   absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
}
catch (URISyntaxException ex) {
   // cannot convert to an URI, considering the location relative
   // unless it is the well-known Spring prefix "classpath*:"
}
```



</details>

第一部分代码中主要处理 `resource` 属性	处理时会有关于占位符的处理,占位符处理接口`PropertyResolver`真正的处理方法: `org.springframework.util.PropertyPlaceholderHelper#parseStringValue`. 有关分析文章:**[PropertyPlaceholderHelper](/docs/env/PropertyResolver/Spring-PropertyPlaceholderHelper.md)**



在得到`location`后会判断**是否是绝对路径**

```JAVA
absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute()
```





第一部分到此结束 , 下面开始第二部分代码的分析

<details>
    <summary>importBeanDefinitionResource 第二部分</summary>







```java
// 第二部分
// Absolute or relative?
// 是不是绝对地址
if (absoluteLocation) {
   try {
      // 获取 import 的数量(bean定义的数量)
      int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
      if (logger.isTraceEnabled()) {
         logger.trace("Imported " + importCount + " bean definitions from URL location [" + location + "]");
      }
   }
   catch (BeanDefinitionStoreException ex) {
      getReaderContext().error(
            "Failed to import bean definitions from URL location [" + location + "]", ele, ex);
   }
}
```

</details>



第二部分代码是将第一部分中得到的 `resource` 属性进行解析，即将xml文件解析成 beanDefinition后返回数量

真正的调度方法: `loadBeanDefinitions`位于`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.support.EncodedResource)`

获取数量的方法在 `registerBeanDefinitions` 方法中(`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#registerBeanDefinitions`) 详细代码如下



```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   // 历史已有的bean定义数量
   int countBefore = getRegistry().getBeanDefinitionCount();
   // 注册
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   // 注册后的数量-历史数量
   return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

👆上述代码中的 `registerBeanDefinitions` 就是我们当前正在分析的方法. 请各位保持耐心继续阅读. 



第二部分代码分析结束. 开始第三部分的代码分析

<details>
    <summary>importBeanDefinitionResource 第三部分</summary>







```java
else {
   // No URL -> considering resource location as relative to the current file.
   try {
      // import 的数量
      int importCount;
      // 资源信息
      Resource relativeResource = getReaderContext().getResource().createRelative(location);
      // 资源是否存在
      if (relativeResource.exists()) {
         // 确定加载的bean定义数量
         importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
         // 加入资源集合
         actualResources.add(relativeResource);
      }
      // 资源不存在处理方案
      else {
         // 获取资源URL的数据
         String baseLocation = getReaderContext().getResource().getURL().toString();
         // 获取import数量
         importCount = getReaderContext().getReader().loadBeanDefinitions(
               StringUtils.applyRelativePath(baseLocation, location), actualResources);
      }
      if (logger.isTraceEnabled()) {
         logger.trace("Imported " + importCount + " bean definitions from relative location [" + location + "]");
      }
   }
   catch (IOException ex) {
      getReaderContext().error("Failed to resolve current resource location", ele, ex);
   }
   catch (BeanDefinitionStoreException ex) {
      getReaderContext().error(
            "Failed to import bean definitions from relative location [" + location + "]", ele, ex);
   }
}
```



</details>



第二部分是处理`location` 为绝对地址的情况, 第三步则相反, 处理非绝对地址的情况

第三部分整体流程

1. 将资源路径转换成资源对象`Resource`

2. 资源是否存在

   1. 存在的处理形式

      直接通过`Resource`获取

      通过`loadBeanDefinitions` 方法加载bean定义的数量, 其实质还是和第二部分的代码中`getReaderContext().getReader().loadBeanDefinitions`的形式一直

   2. 不存在的处理形式

      通过xml读取器的上下文获取 Resource , 接着将读取 bean 定义的数量

      





第三部分代码分析完成, 下面进入第四部分的分析



<details>
    <summary>importBeanDefinitionResource 第四部分</summary>





```java
// 第四部分
Resource[] actResArray = actualResources.toArray(new Resource[0]);
// 唤醒 import 处理事件
getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
```

</details>



第四部分是触发import事件 相关接口: `ReaderEventListener` 执行方法: `org.springframework.beans.factory.parsing.ReaderEventListener#importProcessed`



事件处理的事项: 将`ImportDefinition` 放入容器. 具体存储对象是`org.springframework.beans.testfixture.beans.CollectingReaderEventListener` 

相关分析: [ReaderEventListener](/docs/event/Spring_ReaderEventListener-未完成.md)





到这里`importBeanDefinitionResource` 方法分析完成. 下面贴出完整代码, 以便完整阅读



<details>
    <summary>importBeanDefinitionResource 完整方法</summary>





```java
/**
 * Parse an "import" element and load the bean definitions
 * from the given resource into the bean factory.
 *
 * 解析 import 标签
 */
protected void importBeanDefinitionResource(Element ele) {
   // 第一部分
   // 获取 resource 属性
   String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
   // 是否存在地址
   if (!StringUtils.hasText(location)) {
      getReaderContext().error("Resource location must not be empty", ele);
      return;
   }

   // Resolve system properties: e.g. "${user.dir}"
   // 处理配置文件占位符
   location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

   // 资源集合
   Set<Resource> actualResources = new LinkedHashSet<>(4);

   // Discover whether the location is an absolute or relative URI
   // 是不是绝对地址
   boolean absoluteLocation = false;
   try {
      // 1. 判断是否为 url
      // 2. 通过转换成URI判断是否是绝对地址
      absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
   }
   catch (URISyntaxException ex) {
      // cannot convert to an URI, considering the location relative
      // unless it is the well-known Spring prefix "classpath*:"
   }

   // 第二部分
   // Absolute or relative?
   // 是不是绝对地址
   if (absoluteLocation) {
      try {
         // 获取 import 的数量(bean定义的数量)
         int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
         if (logger.isTraceEnabled()) {
            logger.trace("Imported " + importCount + " bean definitions from URL location [" + location + "]");
         }
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error(
               "Failed to import bean definitions from URL location [" + location + "]", ele, ex);
      }
   }
   // 第三部分
   else {
      // No URL -> considering resource location as relative to the current file.
      try {
         // import 的数量
         int importCount;
         // 资源信息
         Resource relativeResource = getReaderContext().getResource().createRelative(location);
         // 资源是否存在
         if (relativeResource.exists()) {
            // 确定加载的bean定义数量
            importCount = getReaderContext().getReader().loadBeanDefinitions(relativeResource);
            // 加入资源集合
            actualResources.add(relativeResource);
         }
         // 资源不存在处理方案
         else {
            // 获取资源URL的数据
            String baseLocation = getReaderContext().getResource().getURL().toString();
            // 获取import数量
            importCount = getReaderContext().getReader().loadBeanDefinitions(
                  StringUtils.applyRelativePath(baseLocation, location), actualResources);
         }
         if (logger.isTraceEnabled()) {
            logger.trace("Imported " + importCount + " bean definitions from relative location [" + location + "]");
         }
      }
      catch (IOException ex) {
         getReaderContext().error("Failed to resolve current resource location", ele, ex);
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error(
               "Failed to import bean definitions from relative location [" + location + "]", ele, ex);
      }
   }
   // 第四部分
   Resource[] actResArray = actualResources.toArray(new Resource[0]);
   // 唤醒 import 处理事件
   getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
}
```

</details>





- 三个方法中的`importBeanDefinitionResource` 分析已经完成，下面开始`processAliasRegistration`方法分析



### processAliasRegistration

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#processAliasRegistration`



<details>
    <summary>processAliasRegistration 方法详情</summary>







```java
/**
 * Process the given alias element, registering the alias with the registry.
 * 解析 alias 标签
 */
protected void processAliasRegistration(Element ele) {
   // 获取 name 属性
   String name = ele.getAttribute(NAME_ATTRIBUTE);
   // 获取 alias 属性
   String alias = ele.getAttribute(ALIAS_ATTRIBUTE);
   boolean valid = true;
   // name 属性验证
   if (!StringUtils.hasText(name)) {
      getReaderContext().error("Name must not be empty", ele);
      valid = false;
   }
   // alias 属性验证
   if (!StringUtils.hasText(alias)) {
      getReaderContext().error("Alias must not be empty", ele);
      valid = false;
   }
   if (valid) {
      try {
         // 注册
         getReaderContext().getRegistry().registerAlias(name, alias);
      }
      catch (Exception ex) {
         getReaderContext().error("Failed to register alias '" + alias +
               "' for bean with name '" + name + "'", ele, ex);
      }
      // alias注册事件触发
      getReaderContext().fireAliasRegistered(name, alias, extractSource(ele));
   }
}
```

</details>



处理逻辑

1. 读取节点的 `name` 属性和 `alias` 属性
2. 验证 `name` ， `alias` 属性是否存在
   1. 验证通过
      1. 注册别名
      2. 触发别名事件





别名事件的流程代码👇

```java
@Override
public void aliasRegistered(AliasDefinition aliasDefinition) {
   // 获取已经注册过的beanName对应的别名
   List<AliasDefinition> aliases = this.aliasMap.get(aliasDefinition.getBeanName());
   if (aliases == null) {
      aliases = new ArrayList<>();
      // beanName 和 别名对应关系设置
      this.aliasMap.put(aliasDefinition.getBeanName(), aliases);
   }
   // 别名列表添加
   aliases.add(aliasDefinition);
}
```



`processAliasRegistration` 分析到此结束, 下面是三个方法中的最后一个方法的分析`processBeanDefinition`





### processBeanDefinition

- 方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#processBeanDefinition`



<details>
    <summary>processBeanDefinition 方法详情</summary>





```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
   // 创建 bean definition
   BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
   if (bdHolder != null) {
      // bean definition 装饰
      bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
      try {
         // Register the final decorated instance.
         // 注册beanDefinition
         BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
      }
      catch (BeanDefinitionStoreException ex) {
         getReaderContext().error("Failed to register bean definition with name '" +
               bdHolder.getBeanName() + "'", ele, ex);
      }
      // Send registration event.
      // component注册事件触发
      getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
   }
}
```





</details>

`processBeanDefinition` 操作流程

1. 通过`BeanDefinitionParserDelegate`对象解析element, 生成bean定义
2. bean定义的装饰. 
3. 注册bean定义
4. 触发组件注册事件



第一步和第二步的行为请查看[这篇文章](/docs/beans/factory/xml/Spring-BeanDefinitionParserDelegate.md)



bean定义注册核心代码



```java
public static void registerBeanDefinition(
      BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
      throws BeanDefinitionStoreException {

   // Register bean definition under primary name.
   // 获取 beanName
   String beanName = definitionHolder.getBeanName();
   // 注册bean definition
   registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

   // Register aliases for bean name, if any.
   // 别名列表
   String[] aliases = definitionHolder.getAliases();
   // 注册别名列表
   if (aliases != null) {
      for (String alias : aliases) {
         registry.registerAlias(beanName, alias);
      }
   }
}
```

- 将 bean 定义注册到`BeanDefinitionRegistry`中





组件注册事件核心代码

```java
@Override
public void componentRegistered(ComponentDefinition componentDefinition) {
   this.componentDefinitions.put(componentDefinition.getName(), componentDefinition);
}
```

