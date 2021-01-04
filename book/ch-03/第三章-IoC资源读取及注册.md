<div style='display: none'>

Copyright 2020 [HuiFer](http://github.com/huifer) All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

</div>

# 第三章 IoC 资源读取及注册
- 本章笔者将带领各位读者了解在 XML 模式下 Spring 是如何将其进行解析成 BeanDefinition 对象的. 本文围绕三点进行分析, 第一点是关于 **XML 文档的验证**, 第二点是关于 **Document 对象的获取**, 第三点是关于 **XML 解析成 BeanDefinition 并注册**. 



## 3.1 XML 文档验证



### 3.1.1 认识 XML 验证模式

首先不得不承认现今的各类编辑器的智能提示功能都很强大，它们可以帮我们来减少编写 XML 时出错的可能, 但从程序员的角度来说我们还是需要去对 XML 进行数据验证. 

- 对于 XML 文档的验证在 XML 提出之时就有一个验证方式, 各类语言的验证也基本上是围绕这个验证规则进行开发的. 一般常用的验证方式是**DTD(Document_Type_Definition)** 验证

> DTD 的定义:
>
> - A **document type definition** (**DTD**) is a set of *markup declarations* that define a *document type* for an [SGML](https://en.wikipedia.org/wiki/SGML)-family [markup language](https://en.wikipedia.org/wiki/Markup_language) ([GML](https://en.wikipedia.org/wiki/IBM_Generalized_Markup_Language), [SGML](https://en.wikipedia.org/wiki/SGML), [XML](https://en.wikipedia.org/wiki/XML), [HTML](https://en.wikipedia.org/wiki/HTML)).
> - from: [wiki](https://en.wikipedia.org/wiki/Document_type_definition)



- 除了 **DTD** 以外我们还有另外一种 **XSD(XML_Schema_Definition)** 验证方式. 

> XSD 的定义:
>
> - **XSD** (**XML Schema Definition**), a recommendation of the World Wide Web Consortium ([W3C](https://en.wikipedia.org/wiki/W3C)), specifies how to formally describe the elements in an Extensible Markup Language ([XML](https://en.wikipedia.org/wiki/XML)) document. It can be used by programmers to verify each piece of item content in a document. They can check if it adheres to the description of the element it is placed in.[[1\]](https://en.wikipedia.org/wiki/XML_Schema_(W3C)#cite_note-1)
> - from: [wiki](https://en.wikipedia.org/wiki/XML_Schema_(W3C))



- 现在我们了解两种 XML 的验证方式: **DTD**、**XSD** 这两者的验证都是根据文档本身出发, 即需要事先编辑好两种类型的文件 (拓展名: `*.dtd` 、`*.XSD` ) , 在这两种类型文件中存储了关于 Spring 所支持的标签, 那么它们具体存储在那个地方呢? 这些预定义的文件放在 **spring-beans** 的资源文件中

  ![image-20210104133815644](images/image-20210104133815644.png)



### 3.1.2 Spring 中 XML 的验证

在 [3.1.1 认识 XML 验证模式] 中我们了解了关于 XML 的验证方式, 下面我来看看在 Spring 中是如何处理 XML 验证的. 

在前文我们已经知道了两种验证模式, 那么对于 Spring 来说它需要确定具体的一个验证模式. 

目标： **找到 Spring 推测出 XML 验证方式的代码**

- 这段代码是由 `XmlBeanDefinitionReader#getValidationModeForResource` 所提供的 (完整方法签名: `org.springframework.beans.factory.xml.XmlBeanDefinitionReader#getValidationModeForResource`) .



Spring 推测验证方式的代码如下 

```java
protected int getValidationModeForResource(Resource resource) {
   // 获取 xml 验证方式
   int validationModeToUse = getValidationMode();
   if (validationModeToUse != VALIDATION_AUTO) {
      return validationModeToUse;
   }
   int detectedMode = detectValidationMode(resource);
   if (detectedMode != VALIDATION_AUTO) {
      return detectedMode;
   }
   // Hmm, we didn't get a clear indication... Let's assume XSD,
   // since apparently no DTD declaration has been found up until
   // detection stopped (before finding the document's root tag).
   return VALIDATION_XSD;
}
```



具体的获取方式:

1. 从成员变量中获取(成员变量: `validationMode`)
2. 交给`XmlValidationModeDetector` 类进行处理, 具体处理方法签名: `org.springframework.util.xml.XmlValidationModeDetector#detectValidationMode`



我们来看方法 `detectValidationMode` 具体的推测方式

```JAVA
public int detectValidationMode(InputStream inputStream) throws IOException {
   // Peek into the file to look for DOCTYPE.
   BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
   try {
      boolean isDtdValidated = false;
      String content;
      while ((content = reader.readLine()) != null) {
         content = consumeCommentTokens(content);
         if (this.inComment || !StringUtils.hasText(content)) {
            continue;
         }
         if (hasDoctype(content)) {
            isDtdValidated = true;
            break;
         }
         if (hasOpeningTag(content)) {
            // End of meaningful data...
            break;
         }
      }
      return (isDtdValidated ? VALIDATION_DTD : VALIDATION_XSD);
   }
   catch (CharConversionException ex) {
      // Choked on some character encoding...
      // Leave the decision up to the caller.
      return VALIDATION_AUTO;
   }
   finally {
      reader.close();
   }
}
```

别看这段代码很长, 其中最关键的代码是

```java
if (hasDoctype(content)) {
   isDtdValidated = true;
   break;
}
```

在这段代码👆中 `hasDoctype` 很关键 这段就是做字符串判断: 字符串是否包含 `DOCTYPE` 字符串



此时我们可以下定论: **Spring 中 `XmlValidationModeDetector` 对验证模式的确认是循环 xml 整个文件的每一行判断是否有 `DOCTYPE` 字符串, 包含就是 DTD 验证模式, 不包含就是 XSD 模式**







关于 XML 的验证方式笔者到这儿就算是分析完成了. 下面我们将进入 `Document` 对象获取的分析中



## 3.2 Document 对象获取

首先我们需要知道 `Document` 对象从谁那里获得, 不必多说各位肯定可以想到这是从 XML 文件中获取. 那么 Spring 中谁负责这个功能呢? **Spring 中将读取输入流转换成 `Document` 对象的重任交给了 `DocumentLoader` 接口**. 

下面我们来看一下 `DocumentLoader` 的定义



```java
public interface DocumentLoader {

	Document loadDocument(
    	  	InputSource inputSource, EntityResolver entityResolver,
      		ErrorHandler errorHandler, int validationMode, boolean namespaceAware)
        throws Exception;
}
```



从这个接口定义上来看笔者这里会有一个疑问给各位: 参数是`InputSource` 但是我们在使用的时候都传递的是一个字符串(Spring xml 配置文件的文件地址) , 那这个 `InputSource` 的处理过程是在哪儿呢? 

- 处理方法在: `org.springframework.beans.factory.xml.XmlBeanDefinitionReader#loadBeanDefinitions(org.springframework.core.io.support.EncodedResource)` 中

  在这个方法中有下面这段代码. 这便是 XML 文件转换成 `InputSource` 的方式. 

  ```java
  // 省略了前候代码
  InputStream inputStream = encodedResource.getResource().getInputStream();
  try {
     InputSource inputSource = new InputSource(inputStream);
     if (encodedResource.getEncoding() != null) {
        inputSource.setEncoding(encodedResource.getEncoding());
     }
     return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
  ```



在了解了 `InputSource` 来源之后我们就可以去关注 `DocumentLoader` 的实现类了. Spring 中 `DocumentLoader` 有且仅有一个实现类 `DefaultDocumentLoader` 下面我们就来看看这个实现类的一些细节吧. 





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



在做 `InputSource` 转换 `Document` 的方法中主要使用到的是属于 `javax.xml` 和 `org.w3c` 包下的类或者接口, 这部分内容就不具体展开, 各位读者可以根据自己的需求. 





## 3.3 BeanDefinition 注册



通过前面的学习我们得到了 `Document` 对象, 下面我们就需要去看 BeanDefinition 的注册了. 这一段完整的流程代码在`org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions` 方法中有体现, 下面笔者将贴出核心代码. 



```java
// 去掉了异常处理和日志
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
      throws BeanDefinitionStoreException {
    // 将 输入流转换成 Document
    Document doc = doLoadDocument(inputSource, resource);
    // 注册bean定义,并获取数量
    int count = registerBeanDefinitions(doc, resource);
    return count;
}
```



这一章节中我们需要重点关注的方法是 `registerBeanDefinitions` 继续寻找我们的目标方法



```JAVA
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
   // 获取 基于 Document 的Bean定义读取器
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   // 历史已有的bean定义数量
   int countBefore = getRegistry().getBeanDefinitionCount();
   // 注册bean定义
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   // 注册后的数量-历史数量
   return getRegistry().getBeanDefinitionCount() - countBefore;
}
```



在我们找到上面方法并进行阅读后我们可以找到最重要的类 (接口) 已经浮现出来了 ， `BeanDefinitionDocumentReader` 重点对象,  `registerBeanDefinitions` 重点方法. 



目标: **了解 `BeanDefinitionDocumentReader#registerBeanDefinitions` 做了什么**



在开始方法分析(实现类分析) 之前我们先来对 `BeanDefinitionDocumentReader` 接口做一个了解, 主要了解接口的作用. 

`BeanDefinitionDocumentReader` 的作用就是进行 BeanDefinition 的注册

```java
public interface BeanDefinitionDocumentReader {

   /**
    * 注册 bean 定义
    */
   void registerBeanDefinitions(Document doc, XmlReaderContext readerContext)
         throws BeanDefinitionStoreException;

}
```





找到 `BeanDefinitionDocumentReader` 的实现类 `DefaultBeanDefinitionDocumentReader` 直接奔 `registerBeanDefinitions` 方法去. 

我们可以看到下面这样一段代码. 

```java
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
   this.readerContext = readerContext;
   doRegisterBeanDefinitions(doc.getDocumentElement());
}
```

在这段代码中 `doRegisterBeanDefinitions` 就是 Spring 进行 `Document` 对象解析, 并将解析结果包装成 `BeanDefinition` 进行注册的核心方法, 它的方法签名是:  `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions` , 这个方法就是我们需要重点关注的方法( 处理 XML 模式下 Bean定义注册的核心). 



下面正式开始 `doRegisterBeanDefinitions` 的分析





### 3.3.1 doRegisterBeanDefinitions 流程

- 首先我们将 `doRegisterBeanDefinitions` 的代码全部贴出来, 来说一说这个方法里面的流程. 下面请各位阅读这段代码



```java
// 删除了注释和日志
protected void doRegisterBeanDefinitions(Element root) {
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
         if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
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



前文说到我们目标是了解这个方法的整体流程, 下面各位读者可以一点点列一列, 笔者这里给出一个流程

1. 设置父`BeanDefinitionParserDelegate` 对象, 值得注意的是这个设置父对象一般情况下是不存在的即 `this.delegate = null `
2. 创建 `BeanDefinitionParserDelegate` 对象 , `BeanDefinitionParserDelegate` 对象是作为解析的重要方法. 
3. 对于 `profile` 属性的处理
4. XML 解析的前置处理
5. XML 的解析处理
6. XML 解析的后置处理
7. 设置成员变量



这里提一个拓展点 `profile` 这个属性在 Spring 中一般用来做环境区分, 在 SpringBoot 中有一个类似的配置`spring.profiles`  . 在 Spring XML 模式中 `profile` 是属于 `<beans/>` 的一个属性, 各位读者如果感兴趣可以自行搜索相关资料, 笔者这里不展开介绍. 





 在 Spring 中 `preProcessXml` 和 `postProcessXml` 方法目前属于空方法状态, 没有任何实现代码，因此我们的分析目标是: **`parseBeanDefinitions`** 方法





### 3.3.2  parseBeanDefinitions 分析

- 分析之前我们还是将代码直接贴出来, 先看整体流程在追求细节



```JAVA
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



`parseBeanDefinitions` 方法主要是对一个 `Element` 的每个节点进行处理, 节点本身又存在多样性, 

-  节点的多样性(xml标签在这里彼此称之为节点, 可能会和大部分人的说法相冲突, 请各位谅解)
	1. Spring 提供的标签: 即 DTD 或者 XSD 中定义的标签
	2. 自定义标签 
	
	根据节点多样性 Spring 提供了两个方法进行处理 `parseDefaultElement` 和 `delegate.parseCustomElement(ele)` 这两个方法也将是我们下面分析的重点

值得注意的是 `Element` 也有可能是自定义的. 







### 3.3.3 parseDefaultElement Spring 原生标签的处理



在前文我们已经了解到了两种标签的处理, 我们先展开 Spring 原生标签的处理, 那么 Spring 的原生标签有那些呢？ 

- Spring 的原生标签
  1. alias 标签
  2. bean 标签
  3. beans 标签
  4. import 标签



在开始分析标签解析之前我们需要先认识一下标签的结构, 这个其实就要回到笔者在本章(第三章) 

这里对于层级结构仅仅只是对 `alias` 、`import` 、`bean` 和 `beans` 四个标签做一个说明.

笔者这里将以 `spring-beans.dtd` 文件作为基础进行描述 标签层级结构

```xml-dtd
<!ELEMENT beans (
   description?,
   (import | alias | bean)*
)>
```

从这个结构体来看包含关系: **`beans` 下包含 `import`、 `alias` 和 `bean` 三个标签**



下面我们来看 `parseDefaultElement` 的代码内容

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



- 根据前文所说的标签包含关系再来看这段代码. 我们可以将最后一个 `else if` 给忽略. 就把这个当作是处理 `import`、 `alias` 、 `bean` 标签的方法即可. 



现在我们有了三个目标(问题)

1.  **`import` 标签是如何解析的？**
2.  **`alias` 标签是如何解析的？**
3.  **`bean` 标签是如何解析的？**

- 下面笔者就将围绕这三个目标(问题)进行分析



### 3.3.4 import 标签解析

- 首先我们找到 `import` 标签解析的方法: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#importBeanDefinitionResource`



由于 `importBeanDefinitionResource` 方法是一个比较大的方法 笔者这里将其分为几部分进行分别叙述





#### 3.3.4.1 import 标签解析的环境搭建

在开始前我们先做基本用例的搭建. 注意这里我们会沿用第一章中搭建的基本工程作为 import 的数据. 下面就开始编写代码吧. 

1. 首先我们创建一个 Spring XML 配置文件, 名称为`import-beans.xml` .  向文件中填充代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="first-ioc.xml"/>
</beans>
```

2. 编写测试用例

```java
class ImportNodeTest {

    @Test
    void testImportNode() {
        ClassPathXmlApplicationContext context
                = new ClassPathXmlApplicationContext("META-INF/import-beans.xml");
        context.close();
    }

}
```



下面我们看一下文件结构



![image-20210104160826946](images/image-20210104160826946.png)







这些准备工作完成之后我们就可以开始进行真正的分析了. 



#### 3.3.4.2 import 标签解析第一部分: 处理 resource 属性

在 `importBeanDefinitionResource` 的第一部分代码中是将 `import` 中的 `resource` 属性获取出来, 转换成资源对象 `Resource` 集合





- 第一部分的代码如下

```java
// 获取 resource 属性
String location = ele.getAttribute(RESOURCE_ATTRIBUTE);
// 是否存在地址
if (!StringUtils.hasText(location)) {
   getReaderContext().error("Resource location must not be empty", ele);
   return;
}

// 处理配置文件占位符
location = getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);

// 资源集合
Set<Resource> actualResources = new LinkedHashSet<>(4);

// 是不是绝对地址
boolean absoluteLocation = false;
try {
   // 1. 判断是否为 url
   // 2. 通过转换成URI判断是否是绝对地址
   absoluteLocation = ResourcePatternUtils.isUrl(location) || ResourceUtils.toURI(location).isAbsolute();
}
catch (URISyntaxException ex) {
}
```



这里我们是解析 `import` 标签, 那么我们将配置文件中的标签内容对比着看

```xml
<import resource="first-ioc.xml"/>
```



第一句必然是将 `import` 标签的 `resource`  属性获取, 即 `location = first-ioc.xml`

配合测试用例进行 debug 可以看到下面这样的信息

![image-20210104161345583](images/image-20210104161345583.png)



继续往下走 `location` 会背进行二次处理, 处理什么呢？ Spring 在这里对其进行占位符的处理, 占位符可能是 `${}` 在这里会将其转换成一个具体的地址

这里各位如果感兴趣可以考虑阅读: `PropertyResolver` 接口的实现, 



对于 `location` 的二次处理后 Spring 紧接着做了一次是否是绝对路径的判断. 

1. 判断是否为 url
2. 通过转换成URI判断是否是绝对地址



这里对于 `absoluteLocation` 的模拟可能比较麻烦, 笔者这里不做展开.  下面我们来看第二第三部分的代码



#### 3.3.4.3 import 标签解析的第二部分和第三部分 重回 loadBeanDefinitions

首先将代码贴出请各位读者进行基础阅读. 



```java
// 删除了异常处理和日志
// 第二部分
// Absolute or relative?
// 是不是绝对地址
if (absoluteLocation) {
      // 获取 import 的数量(bean定义的数量)
    int importCount = getReaderContext().getReader().loadBeanDefinitions(location, actualResources);
}
// 第三部分
else {
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
}
```



虽然我们对于 `absoluteLocation` 变量的模拟比较麻烦但是我们通过观察, 这里主要做的事情就是在做 `loadBeanDefinitions` ，这个方法不知道读者是否熟悉, 看着和 `doLoadBeanDefinitions` 有点相似. 事实上这就是一个同源方法. 在第二部分第三部分代码中这段代码 `getReaderContext().getReader().loadBeanDefinitions` 就是核心. 那这个核心又是在做 `beans` 标签解析了. 这里就是一个嵌套处理. 那么我们需要理清楚这个嵌套关系, 

前文我们聊了关于 `beans`、 `bean` 、`alias` 和 `import` 的关系. 现在我们发现 `import` 里面存放的是一个 `beans`   . 这个关系就是下面这个图



```mermaid
graph TD
beans --包含--> bean
beans --包含--> alias
beans --包含--> import 
import --包含--> beans
```



根据这样一个包含关系图我们可以将更多的重点放在 `bean` 和 `alias` 标签的解析中. 



#### 3.3.4.3 import 标签解析的第四部分 import 事件处理

- 最后我们来看 `import` 标签解析的第四部分代码

```java
// 第四部分
Resource[] actResArray = actualResources.toArray(new Resource[0]);
// 唤醒 import 处理事件
getReaderContext().fireImportProcessed(location, actResArray, extractSource(ele));
```



`import` 标签解析的第四部分就是做事件发布. 

我们来看看事件发布到底做了什么. 在当前事件发布的核心处理是依靠 `ReaderEventListener` 对象的

在这里对于 import 事件处理`fireImportProcessed` 本质上是`org.springframework.beans.testfixture.beans.CollectingReaderEventListener#importProcessed` 方法

这里就是存储了一个 `ImportDefinition` 对象

下面我们来看看详细代码

- 事件的处理核心

```java
@Override
public void importProcessed(ImportDefinition importDefinition) {
   this.imports.add(importDefinition);
}
```

- 存储容器 `imports` 

  ```java
  private final List<ImportDefinition> imports = new LinkedList<>();
  ```



值得注意的是在 Spring 中 事件监听器 `eventListener` 是 `EmptyReaderEventListener` 实现, 笔者在上文所说的是 `CollectingReaderEventListener` 实现, 



- `EmptyReaderEventListener` 这个实现类中什么都没有做. 

```java
public class EmptyReaderEventListener implements ReaderEventListener {

   @Override
   public void defaultsRegistered(DefaultsDefinition defaultsDefinition) {
      // no-op
   }

   @Override
   public void componentRegistered(ComponentDefinition componentDefinition) {
      // no-op
   }

   @Override
   public void aliasRegistered(AliasDefinition aliasDefinition) {
      // no-op
   }

   @Override
   public void importProcessed(ImportDefinition importDefinition) {
      // no-op
   }

}
```



这里的结论是在 debug 中发现的. 



![image-20210104164157642](images/image-20210104164157642.png)





好的. 到这里 `import` 标签的解析全部完成. 下面将和各位读者分享 `alias` 标签的解析



### 3.3.5 alias 标签解析



### 3.3.6 bean 标签解析



