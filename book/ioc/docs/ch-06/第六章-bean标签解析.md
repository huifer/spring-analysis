# 第六章 bean 标签解析
- 本章笔者将和各位读者介绍 `bean` 标签的真正解析类: `BeanDefinitionParserDelegate`



在[第三章](/docs/ch-03/第三章-IoC资源读取及注册.md)中笔者在其中介绍了关于 `bean` 标签解析的大致流程

1. 将 `Element` 交给 `BeanDefinitionParserDelegate` 解析
2. 注册 Bean Definition
3. 发布组件注册事件

在第三章中笔者对于 `BeanDefinitionParserDelegate` 对象的解析过程没有做出详细的分析，在本章节笔者将会对其进行相对详细的分析。下面我们先来搭建测试环境。





## 6.1 创建 bean 标签环境

- 创建一个 `bean` 标签的基础环境，相信这一个步骤各位读者一定了解如何编写了，请各位耐心让笔者我在给各位写一遍。



1. 创建 Spring xml 配置文件，命名为: `bean-node.xml` 向文件中填写下面代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns="http://www.springframework.org/schema/beans"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

   <bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">

   </bean>
</beans>
```

2. 测试用例的编写

```java
class BeanNodeTest {
   @Test
   void testBean() {
       ClassPathXmlApplicationContext context
               = new ClassPathXmlApplicationContext("META-INF/bean-node.xml");

       Object people = context.getBean("people");
       context.close();
   }
}
```



这就是我们的一个基本测试用例。

在开始正式分析之前我们需要回忆一下真正的处理方法在哪儿。处理 `bean` 标签的方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#processBeanDefinition`

- `processBeanDefinition` 方法详情

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



在这段方法中 Spring 通过 `BeanDefinitionParserDelegate#parseBeanDefinitionElement` 进行解析 `Element` 即解析标签。 下面我们就开始对这个方法进行分析。



## 6.2 parseBeanDefinitionElement 第一部分 id 和 name 的处理

前文我们已经找到了入口方法下面我们找到最终处理方法: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element, org.springframework.beans.factory.config.BeanDefinition)`。我们先来看第一部分的处理。



- 先来看 `parseBeanDefinitionElement` 第一部分代码

```java
// 第一部分
// 获取 id
String id = ele.getAttribute(ID_ATTRIBUTE);
// 获取 name
String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

// 别名列表
List<String> aliases = new ArrayList<>();
// 是否有 name 属性
if (StringUtils.hasLength(nameAttr)) {
 // 获取名称列表, 根据 `,; `进行分割
 String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
 // 添加所有
 aliases.addAll(Arrays.asList(nameArr));
}
```



第一部分的处理很简单**获取 `bean` 标签中的 `id` 和 `name` 属性，对 `name` 属性做分隔符切分后将切分结果作为别名**。( `bean` 标签的属性值各位可以翻阅第三章 3.3.6.2 bean 标签的定义 的内容)

- 分隔符
  1. 逗号 `,`
  2. 分号 `;` 
  3. 空格 ` `

`BeanDefinitionParserDelegate#parseBeanDefinitionElement` 第一部分就处理完成了. 下面进入第二部分的分析





## 6.2 parseBeanDefinitionElement 第二部分 关于 Bean Name 处理

- 先看 `BeanDefinitionParserDelegate#parseBeanDefinitionElement` 第二部分的代码

```java
String beanName = id;
if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
 // 别名的第一个设置为beanName
 beanName = aliases.remove(0);
 if (logger.isTraceEnabled()) {
    logger.trace("No XML 'id' specified - using '" + beanName +
          "' as bean name and " + aliases + " as aliases");
 }
}

// bean definition 为空
if (containingBean == null) {
 // 判断 beanName 是否被使用, bean 别名是否被使用
 checkNameUniqueness(beanName, aliases, ele);
}
```



这一部分的代码还有两个小段. 

1. 关于 Bean Name 的推论。
2. 关于 Bean Name 是否被使用，别名是否被使用的验证

那么我们先来对 Bean Name 的确认过程进行分析，**在 Spring 中 Bean Name 可以是 `id` 属性 也可以是 `name` 属性根据分隔符切分后的第一个元素，注意 `id` 是最高优先级(当存在 `id` 属性时 Bean Name 直接采用 `id` 作为结果)**

下面我们来对此进行测试代码编写。

第一种 XML 配置，只配置 `id`

```xml
<bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
    
</bean>
```

根据前文分析此时的 Bean Name 应该是 `people` 。进入调试阶段笔者在这里直接截图展示当前的 Bean Name。

![image-20210107100916731](./images/image-20210107100916731.png)



第二种 XML 配置， 只配置 `name` 属性并且使用分隔符进行分割

```xml
<bean name="peopleBean,people" class="com.source.hot.ioc.book.pojo.PeopleBean">

</bean>
```

根据前文分析此时的 Bean Name 应该是 `peopleBean` 。进入调试阶段笔者在这里直接截图展示当前的 Bean Name。

![image-20210107101218124](./images/image-20210107101218124.png)



第三种 XML 配置，同时配置 `id` 和 `name` 属性

```xml
<bean id="p1" name="peopleBean,people" class="com.source.hot.ioc.book.pojo.PeopleBean">

</bean>
```

根据前文分析此时的 Bean Name 应该是 `p1` 。进入调试阶段笔者在这里直接截图展示当前的 Bean Name。

![image-20210107101434927](./images/image-20210107101434927.png)



通过这三种 XML 配置方式笔者相信各位读者对 Bean Name 的确认有了一定的了解。笔者在总结一下

1. `id` 存在的情况下都是用 `id` 作为 Bean Name
2. `id` 不存在的情况下使用 `name` 被分隔符切分后的第一个元素作为 Bean Name





下面进行 Bean Name 和 Alias 的验证阶段。这里的验证是指  Bean Name 和 Alias 是否使用。 下面我们来看看验证代码。

- `checkNameUniqueness` 完整代码

```java
protected void checkNameUniqueness(String beanName, List<String> aliases, Element beanElement) {
   // 当前寻找的name
   String foundName = null;

   // 是否有 beanName
   // 使用过的name中是否存在
   if (StringUtils.hasText(beanName) && this.usedNames.contains(beanName)) {
      foundName = beanName;
   }
   if (foundName == null) {
      // 寻找匹配的第一个
      foundName = CollectionUtils.findFirstMatch(this.usedNames, aliases);
   }
   // 抛出异常
   if (foundName != null) {
      error("Bean name '" + foundName + "' is already used in this <beans> element", beanElement);
   }

   // 加入使用队列
   this.usedNames.add(beanName);
   this.usedNames.addAll(aliases);
}
```

在 `checkNameUniqueness` 方法中需要重点关注 `usedNames` 变量，`usedNames` 变量是指已经使用过的名称包含 Bean Name 和 Alias 。整个验证过程就是判断传递的参数是否在 `usedNames` 容器中存在，一旦存在就会抛出异常。 如果不存在则将 Bean Name 和 Alias 加入 `usedNames` 容器等待后续使用。



到此对于 `BeanDefinitionParserDelegate#parseBeanDefinitionElement` 方法的第二部分分析完成，下面九江进入到第三部分，第三部分是最终处理 `bean` 标签的方法，我们一起看看吧



## 6.3 parseBeanDefinitionElement 处理 bean 标签

- 先看 `BeanDefinitionParserDelegate#parseBeanDefinitionElement` 第三部分的代码

```java
AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
```

这个方法是调用内部 `parseBeanDefinitionElement` 方法，最终我们应该看的是下面这段代码

```java
// 删除了异常处理和日志相关代码
public AbstractBeanDefinition parseBeanDefinitionElement(
  Element ele, String beanName, @Nullable BeanDefinition containingBean) {

  // 第一部分
  // 设置阶段 bean定义解析阶段
  this.parseState.push(new BeanEntry(beanName));

  String className = null;
  // 是否包含属性 class
  if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
      className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
  }
  String parent = null;
  // 是否包含属性 parent
  if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
      parent = ele.getAttribute(PARENT_ATTRIBUTE);
  }

  // 第二部分

  // 创建 bean definition
  AbstractBeanDefinition bd = createBeanDefinition(className, parent);

  // bean definition 属性设置
  parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
  // 设置描述
  bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
  // 元信息设置 meta 标签解析
  parseMetaElements(ele, bd);
  // lookup-override 标签解析
  parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
  // replaced-method 标签解析
  parseReplacedMethodSubElements(ele, bd.getMethodOverrides());

  // constructor-arg 标签解析
  parseConstructorArgElements(ele, bd);
  // property 标签解析
  parsePropertyElements(ele, bd);
  // qualifier 标签解析
  parseQualifierElements(ele, bd);
  // 资源设置
  bd.setResource(this.readerContext.getResource());
  // source 设置
  bd.setSource(extractSource(ele));

  return bd;
 
}

```



- `parseBeanDefinitionElement` 方法一眼看上去有很多处理操作，各位读者请耐心阅读。



### 6.3.1 处理 class name  和 parent

- 在这里笔者将抛弃 `parseState` 对象的分析，各位可以简单理解成这是一个阶段标记，直接开始对 class name 和 parent 两个变量进行分析吧。

- 对于 `className` 和 `parent` 两个变量的处理代码如下

```java
String className = null;
// 是否包含属性 class
if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
   className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
}
String parent = null;
// 是否包含属性 parent
if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
   parent = ele.getAttribute(PARENT_ATTRIBUTE);
}
```

关于 `className` 和 `parent` 的处理就很简单了, 直接从 `Element` 中直接提取即可. 



在 `className` 和 `parent` 变量处理完成之后 Spring 做了什么呢？ 创建 `AbstractBeanDefinition` 为返回值做准备。



### 6.3.2 创建 AbstractBeanDefinition 

首先请各位阅读创建 `AbstractBeanDefinition` 代码

```java
// parseBeanDefinitionElement 方法内调用
AbstractBeanDefinition bd = createBeanDefinition(className, parent);

// createBeanDefinition 详情
protected AbstractBeanDefinition createBeanDefinition(@Nullable String className, @Nullable String parentName)
    throws ClassNotFoundException {


    return BeanDefinitionReaderUtils.createBeanDefinition(
        parentName, className, this.readerContext.getBeanClassLoader());
}

// 最终的调用
public static AbstractBeanDefinition createBeanDefinition(
    @Nullable String parentName, @Nullable String className, @Nullable ClassLoader classLoader) throws ClassNotFoundException {

    GenericBeanDefinition bd = new GenericBeanDefinition();
    // 设置 父bean
    bd.setParentName(parentName);
    if (className != null) {
        if (classLoader != null) {
            // 设置 class
            // 内部是通过反射创建 class
            bd.setBeanClass(ClassUtils.forName(className, classLoader));
        }
        else {
            // 设置 class name
            bd.setBeanClassName(className);
        }
    }
    return bd;
}

```

笔者在此将最终的方法签名贴出便于各位读者进行查阅，`createBeanDefinition` 方法签名 `org.springframework.beans.factory.support.BeanDefinitionReaderUtils#createBeanDefinition`，下面我们来看看其中做了什么操作。

在 `BeanDefinitionReaderUtils#createBeanDefinition` 中创建了 `GenericBeanDefinition` 类型的 `BeanDefinition` 对象，并给成员变量赋值，赋值的成员变量有:  `parentName` 、`beanClass` 。注意: ** `beanClass` 的赋值存在两种情况，第一种类型是 `String` ，第二种类型是 `Class` **



到此准备完成了基本数据信息

1. `parent`
2. `className` 

我们来进行调试，根据下面的一个 Spring XML 配置 ，通过这个创建方法 ( `createBeanDefinition` ) 结果是什么样的。

```xml
<bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">

</bean>
```

- 执行 `createBeanDefinition` 方法后的信息

![image-20210107110414401](./images/image-20210107110414401.png)

在这个截图中我们可以看到 `beanClass` 还是一个字符串状态。





好的, `AbstractBeanDefinition` 准备完成, 接下来就是补充完整其他的数据信息。那么我们跟着源代码先来看 `parseBeanDefinitionAttributes` 方法做了什么



### 6.3.3 parseBeanDefinitionAttributes 设置 Bean Definition 的基本信息

首先请各位阅读  `parseBeanDefinitionAttributes` 代码

```java
public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName,
      @Nullable BeanDefinition containingBean, AbstractBeanDefinition bd) {

   // 是否存在 singleton 属性
   if (ele.hasAttribute(SINGLETON_ATTRIBUTE)) {
      error("Old 1.x 'singleton' attribute in use - upgrade to 'scope' declaration", ele);
   }
   // 是否存在 scope 属性
   else if (ele.hasAttribute(SCOPE_ATTRIBUTE)) {
      // 设置 scope 属性
      bd.setScope(ele.getAttribute(SCOPE_ATTRIBUTE));
   }
   // bean 定义是否为空
   else if (containingBean != null) {
      // Take default from containing bean in case of an inner bean definition.
      // 设置 bean definition 中的 scope
      bd.setScope(containingBean.getScope());
   }

   // 是否存在 abstract 属性
   if (ele.hasAttribute(ABSTRACT_ATTRIBUTE)) {
      // 设置 abstract 属性
      bd.setAbstract(TRUE_VALUE.equals(ele.getAttribute(ABSTRACT_ATTRIBUTE)));
   }

   // 获取 lazy-init 属性
   String lazyInit = ele.getAttribute(LAZY_INIT_ATTRIBUTE);
   // 是否是默认的 lazy-init 属性
   if (isDefaultValue(lazyInit)) {
      // 获取默认值
      lazyInit = this.defaults.getLazyInit();
   }
   // 设置 lazy-init 属性
   bd.setLazyInit(TRUE_VALUE.equals(lazyInit));

   // 获取注入方式
   // autowire 属性
   String autowire = ele.getAttribute(AUTOWIRE_ATTRIBUTE);
   // 设置注入方式
   bd.setAutowireMode(getAutowireMode(autowire));

   // 依赖的bean
   // depends-on 属性
   if (ele.hasAttribute(DEPENDS_ON_ATTRIBUTE)) {
      String dependsOn = ele.getAttribute(DEPENDS_ON_ATTRIBUTE);
      bd.setDependsOn(StringUtils.tokenizeToStringArray(dependsOn, MULTI_VALUE_ATTRIBUTE_DELIMITERS));
   }

   // autowire-candidate 是否自动注入判断
   String autowireCandidate = ele.getAttribute(AUTOWIRE_CANDIDATE_ATTRIBUTE);
   if (isDefaultValue(autowireCandidate)) {
      String candidatePattern = this.defaults.getAutowireCandidates();
      if (candidatePattern != null) {
         String[] patterns = StringUtils.commaDelimitedListToStringArray(candidatePattern);
         // * 匹配 设置数据
         bd.setAutowireCandidate(PatternMatchUtils.simpleMatch(patterns, beanName));
      }
   }
   else {
      bd.setAutowireCandidate(TRUE_VALUE.equals(autowireCandidate));
   }

   // 获取 primary 属性
   if (ele.hasAttribute(PRIMARY_ATTRIBUTE)) {
      bd.setPrimary(TRUE_VALUE.equals(ele.getAttribute(PRIMARY_ATTRIBUTE)));
   }

   // 获取 init-method 属性
   if (ele.hasAttribute(INIT_METHOD_ATTRIBUTE)) {
      String initMethodName = ele.getAttribute(INIT_METHOD_ATTRIBUTE);
      bd.setInitMethodName(initMethodName);
   }
   // 没有 init-method 的情况处理
   else if (this.defaults.getInitMethod() != null) {
      bd.setInitMethodName(this.defaults.getInitMethod());
      bd.setEnforceInitMethod(false);
   }

   // 获取 destroy-method 属性
   if (ele.hasAttribute(DESTROY_METHOD_ATTRIBUTE)) {
      String destroyMethodName = ele.getAttribute(DESTROY_METHOD_ATTRIBUTE);
      bd.setDestroyMethodName(destroyMethodName);
   }
   // 没有 destroy-method 的情况处理
   else if (this.defaults.getDestroyMethod() != null) {
      bd.setDestroyMethodName(this.defaults.getDestroyMethod());
      bd.setEnforceDestroyMethod(false);
   }

   // 获取 factory-method 属性
   if (ele.hasAttribute(FACTORY_METHOD_ATTRIBUTE)) {
      bd.setFactoryMethodName(ele.getAttribute(FACTORY_METHOD_ATTRIBUTE));
   }
   // 获取 factory-bean 属性
   if (ele.hasAttribute(FACTORY_BEAN_ATTRIBUTE)) {
      bd.setFactoryBeanName(ele.getAttribute(FACTORY_BEAN_ATTRIBUTE));
   }

   return bd;
}
```



这段代码的处理过程和处理 `className` 、`parent` 的过程基本相似。简单来说就是从 `bean` 标签中获取对应的属性然后赋值给 Bean Definition (Bean Definition 就是前面创建出来的。) ，值得注意的是在整个方法中还涉及到一个变量 `defaults` ，那么这个变量是什么呢？

先来看 `defaults` 的定义代码

```java
private final DocumentDefaultsDefinition defaults = new DocumentDefaultsDefinition();
```

- `DocumentDefaultsDefinition` 类呢就是 Spring 定义的默认值 , 主要针对 Bean Definition 的属性进行定义默认值

  默认值涉及的信息如下

  - lazyInit
    merge
    autowire
    autowireCandidates
    initMethod
    destroyMethod
    source

  在了解了默认值有哪些后我们需要了解默认值的具体数据是什么。笔者在这里准备了一个表各位可以看一下



| 默认值属性名称       | 默认值  |
| -------------------- | ------- |
| `lazyInit`           | `false` |
| `merge`              | `false` |
| `autowire`           | `no`    |
| `autowireCandidates` | `null`  |
| `initMethod`         | `null`  |
| `destroyMethod`      | `null`  |
| `source`             | `null`  |



在了解属性的来源和默认值后我们来看看经过 `parseBeanDefinitionAttributes` 处理, Bean Definition 变成什么样子了。

- `parseBeanDefinitionAttributes` 处理后的 Bean Definition 

![image-20210107111637003](./images/image-20210107111637003.png)





### 6.3.4 Bean Definition 描述设置

在设置完成 Bean Definition 的基本数据后在 `parseBeanDefinitionElement` 方法中紧接着做了关于 Bean Definition 的描述信息设置。

在原有的用例中笔者并没有设置描述信息的属性下面我们来进行修改, 修改后的内容如下

```xml
<bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <description>this is a bean description</description>
</bean>
```

用例准备完毕，来看看代码的处理

- 对于 Bean Definition 的描述信息处理方法如下

```java
bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
```



这个方法就很简答, 获取当前节点下的 `description` 节点中的数据值，将这个数据值赋值给 Bean Definition 对象。下面我们来通过调试看一下经过这个方法后 Bean Definition 的数据信息

- 经过设置描述信息后的 Bean Definition

![image-20210107112400725](./images/image-20210107112400725.png)





### 6.3.5 Meta 属性设置

在设置完 Bean Definition 的描述信息后 `parseBeanDefinitionElement`  方法中紧接着做了关于 Meta 信息设置。

在关于 Bean Defintion 描述的用例中笔者并未对其进行 `meta` 属性的设置, 下面笔者将其进行修改成下面的形式。

```xml
<bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <description>this is a bean description</description>
        <meta key="key" value="value"/>
</bean>
```

用例准备完毕，先看代码的处理。

```java
public void parseMetaElements(Element ele, BeanMetadataAttributeAccessor attributeAccessor) {
   // 获取下级标签
   NodeList nl = ele.getChildNodes();
   // 循环子标签
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      // 设置数据
      // 是否是 meta 标签
      if (isCandidateElement(node) && nodeNameEquals(node, META_ELEMENT)) {
         Element metaElement = (Element) node;
         // 获取 key 属性
         String key = metaElement.getAttribute(KEY_ATTRIBUTE);
         // 获取 value 属性
         String value = metaElement.getAttribute(VALUE_ATTRIBUTE);
         // 元数据对象设置
         BeanMetadataAttribute attribute = new BeanMetadataAttribute(key, value);
         // 设置 source
         attribute.setSource(extractSource(metaElement));
         // 信息添加
         attributeAccessor.addMetadataAttribute(attribute);
      }
   }
}
```



在 `parseMetaElements` 这段代码中我们可以看到 Spring 在获取 `bean` 标签的子节点列表中名称是 `meta` 的标签，并将其 `key` 和 `value` 两个属性提取出来创建成 `BeanMetadataAttribute` 后设置给 `BeanMetadataAttributeAccessor` 。

这便是处理 `meta` 标签的流程。不过参数 `BeanMetadataAttributeAccessor` 是什么呢？要回答这个问题我们需要从 `AbstractBeanDefinition` 出发，先来看看 `AbstractBeanDefinition` 的类图

- `AbstractBeanDefinition` 类图

  ![AbstractBeanDefinition](./images/AbstractBeanDefinition.png)

在类图中我们可以直观的看到 `AbstractBeanDefinition` 是 `BeanMetadataAttributeAccessor` 的子类。这就好办了，对于 `BeanMetadataAttributeAccessor` 的设置其实就是为 `AbstractBeanDefinition` 添加属性。



好的，理论性质的内容到此就结束了，下面我们来看看经过 `parseMetaElements` 方法后 Bean Definition 变了什么。



- 经过 `parseMetaElements`  方法后的 Bean Definition

![image-20210107113518568](./images/image-20210107113518568.png)





### 6.3.6 lookup-override 标签处理

在处理完成 `meta` 标签后 `parseBeanDefinitionElement`  方法中紧接着做了关于 `lookup-override` 标签的解析。

同样的我们前面的测试代码还不够支持我们对这个方法进行分析, 我们需要补充测试用例。

假设现在有一个商店在出售水果, 水果可以是苹果, 香蕉等此时我们可以通过不同的商店返回不同的售卖内容。首先定义下面几个Java对象

1. `Fruits`

```JAVA
public class Fruits {
  private String name;

  public String getName() {
     return name;
  }

  public void setName(String name) {
     this.name = name;
  }
}
```

2. `Apple`

```JAVA
public class Apple extends Fruits {
  public Apple() {
     this.setName("apple");
  }

  public void hello() {
     System.out.println("hello");
  }

}
```

3. `Shop`

```JAVA
public abstract class Shop {
  public abstract Fruits getFruits();
}
```

4. spring xml 配置，文件名称: `spring-lookup-method.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns="http://www.springframework.org/schema/beans"
     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="apple" class="com.source.hot.ioc.book.pojo.lookup.Apple">
  </bean>

  <bean id="shop" class="com.source.hot.ioc.book.pojo.lookup.Shop">
     <lookup-method name="getFruits" bean="apple"/>
  </bean>


</beans>
```

5. 测试用例

```java
@Test
void testLookupMethodBean() {
   ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-lookup-method.xml");

   Shop shop = context.getBean("shop", Shop.class);
   System.out.println(shop.getFruits().getName());
   assert context.getBean("apple").equals(shop.getFruits());
}
```

输出结果是 `apple`，测试通过



针对上面的测试用例笔者先对执行流程进行描述。在 Spring XML 中配置 `lookup-method` 的信息其中当我们调用, `name` 对应的方法时会返回 `bean` 中配置的 Bean 实例。转换成我们的测试用应当是这样一个流程，当使用者在调用 `shop` 这个 Bean 的 `getFruits` 方法得到的对象就是 `apple` 这个 Bean 实例。下面我们来看具体的标签解析过程。

提取出我们需要解析的标签信息。

```xml
<bean id="shop" class="com.source.hot.ioc.book.pojo.lookup.Shop">
   <lookup-method name="getFruits" bean="apple"/>
</bean>
```

进入到 `parseLookupOverrideSubElements` 方法。



各位读者可以先看 `parseLookupOverrideSubElements` 方法

- `parseLookupOverrideSubElements` 方法详情

```JAVA
public void parseLookupOverrideSubElements(Element beanEle, MethodOverrides overrides) {
   // 获取子标签
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      // 是否有 lookup-method 标签
      if (isCandidateElement(node) && nodeNameEquals(node, LOOKUP_METHOD_ELEMENT)) {
         Element ele = (Element) node;
         // 获取 name 属性
         String methodName = ele.getAttribute(NAME_ATTRIBUTE);
         // 获取 bean 属性
         String beanRef = ele.getAttribute(BEAN_ELEMENT);
         // 创建 覆盖依赖
         LookupOverride override = new LookupOverride(methodName, beanRef);
         // 设置 source
         override.setSource(extractSource(ele));
         overrides.addOverride(override);
      }
   }
}
```

这段处理模式和 `meta` 标签的处理模式基本相同。处理方式：获取所有子节点，如果子节点是 `lookup-method` 就提取 `name` 和 `bean` 两个属性，在属性获取后创建对应的 Java 对象: `LookupOverride` 并设置给 `MethodOverrides` ，`MethodOverrides` 是 Bean Definition 的一个成员变量。分析完毕我们来看执行后的结果。



- 经过 `parseLookupOverrideSubElements` 方法调用后 Bean Definition 的信息
 ![image-20210107131205214](./images/image-20210107131205214.png)



在这儿笔者要提出一个问题，配置完成了如何使用呢？在我们的测试用例中执行 `shop.getFruits()` 方法的时候发生了什么？这里就涉及到 Spring 的另一个重要知识点 Spring AOP 了，本文不对 AOP 做展开，在此笔者将关于 `shop.getFruits()` 的调用链路告诉各位读者，读者可以根据需要进行详细了解。

首先 `shop` 这个对象是一个代理对象

- `shop` 对象信息

![image-20210107131708569](./images/image-20210107131708569.png)

这一点区别于我们常见的如本例中 `apple` 这个 Bean . 

- `apple` 对象信息

  ![image-20210107131817195](./images/image-20210107131817195.png)



在对比这两个对象的时候有心的读者肯定发现了 在 `shop` 对象信息中有一个 `LookupOverrideMethodInterceptor` 内容，其实 `LookupOverrideMethodInterceptor` 这个就是我们寻找的真相，在 `LookupOverrideMethodInterceptor` 中会悄悄地把原本的方法给替换执行。

下面我们就来看看在 `LookupOverrideMethodInterceptor` 中做了什么

- `LookupOverrideMethodInterceptor#intercept` 替换执行方法的返回值 

```java
@Override
public Object intercept(Object obj, Method method, Object[] args, MethodProxy mp) throws Throwable {
   // Cast is safe, as CallbackFilter filters are used selectively.
   LookupOverride lo = (LookupOverride) getBeanDefinition().getMethodOverrides().getOverride(method);
   Assert.state(lo != null, "LookupOverride not found");
   Object[] argsToUse = (args.length > 0 ? args : null);  // if no-arg, don't insist on args at all
   if (StringUtils.hasText(lo.getBeanName())) {
      return (argsToUse != null ? this.owner.getBean(lo.getBeanName(), argsToUse) :
            this.owner.getBean(lo.getBeanName()));
   }
   else {
      return (argsToUse != null ? this.owner.getBean(method.getReturnType(), argsToUse) :
            this.owner.getBean(method.getReturnType()));
   }
}
```

执行流程:

1. 通过 `method` 对象去找到 `LookupOverride` 对象
2. 通过 `BeanFactory` 来获取返回对象



在了解执行流程后我们来复盘我们的测试用例，在我们的测试用例中参数是不存在的，所以它会执行 `this.owner.getBean(lo.getBeanName())` 方法 ， 注意 `owner` 是 `BeanFactory` 可以从中获取 Bean 实例。

还记得我们存入 `LookupOverride` 中的信息吗？不记得的话可以看看下面这个截图

![image-20210107132416881](./images/image-20210107132416881.png)

哦~这下子都明白了，Spring 在这个阶段会将 `beanName` 从容器中获取对应的实例，将这个实例作为返回值。也正是通过这样的一个动态代理技术( Spring 在这里采用的 CGLIB ) 技术改变了返回值。

到此我们对于 `lookup-override` 标签的解析及调用 `lookup-override` 方法时发生的内容做了详细的分析，下面我们将进入 `replaced-method` 方法的分析



### 6.3.7 replaced-method 标签处理

在前文笔者对 `lookup-override` 做了充分的分析，各位读者在这个基础上来阅读 `replaced-method` 方法将事半功倍，它们的底层处理方式可以说是大同小异。一如既往我们先来搭建测试环境

本次测试用例的 Java 对象会延用 `lookup-override` 的对象，在此基础上我们需要编写 `MethodReplacer` 的实现类，在 Spring 中 `MethodReplacer` 时替换方法的核心接口。

- `MethodReplacerApple` 详细内容

```java
public class MethodReplacerApple implements MethodReplacer {
   @Override
   public Object reimplement(Object obj, Method method, Object[] args) throws Throwable {
      System.out.println("方法替换");
      return obj;
   }
}
```

编辑 Spring XML 配置，文件名称: `spring-replaced-method.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns="http://www.springframework.org/schema/beans"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   <bean id="apple" class="com.source.hot.ioc.book.pojo.lookup.Apple">
      <replaced-method replacer="methodReplacerApple" name="hello" >
         <arg-type>String</arg-type>
      </replaced-method>

   </bean>

   <bean id="methodReplacerApple" class="com.source.hot.ioc.book.pojo.replacer.MethodReplacerApple">
   </bean>

</beans>
```



单元测试编写

```java
@Test
void testReplacedMethod(){
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-replaced-method.xml");
    Apple apple = context.getBean("apple", Apple.class);
    apple.hello();
}
```

在没有方法替换之前会在控制台输出 `hello` ，当进行了方法替换的配置后会在控制台输出 `方法替换`。

下面我们先来看 `replaced-method` 标签的解析

首先请各位先进行 `parseReplacedMethodSubElements` 方法代码的阅读

- `parseReplacedMethodSubElements` 方法详情

```java
public void parseReplacedMethodSubElements(Element beanEle, MethodOverrides overrides) {
   // 子节点获取
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      // 是否包含 replaced-method 标签
      if (isCandidateElement(node) && nodeNameEquals(node, REPLACED_METHOD_ELEMENT)) {
         Element replacedMethodEle = (Element) node;
         // 获取 name 属性
         String name = replacedMethodEle.getAttribute(NAME_ATTRIBUTE);
         // 获取 replacer
         String callback = replacedMethodEle.getAttribute(REPLACER_ATTRIBUTE);
         // 对象组装
         ReplaceOverride replaceOverride = new ReplaceOverride(name, callback);
         // Look for arg-type match elements.
         // 子节点属性
         // 处理 arg-type 标签
         List<Element> argTypeEles = DomUtils.getChildElementsByTagName(replacedMethodEle, ARG_TYPE_ELEMENT);

         for (Element argTypeEle : argTypeEles) {
            // 获取 match 数据值
            String match = argTypeEle.getAttribute(ARG_TYPE_MATCH_ATTRIBUTE);
            // match 信息设置
            match = (StringUtils.hasText(match) ? match : DomUtils.getTextValue(argTypeEle));
            if (StringUtils.hasText(match)) {
               // 添加类型标识
               replaceOverride.addTypeIdentifier(match);
            }
         }
         // 设置 source
         replaceOverride.setSource(extractSource(replacedMethodEle));
         // 重载列表添加
         overrides.addOverride(replaceOverride);
      }
   }
}
```



这一段对于 `replaced-method` 标签的解析分为下面几个步骤

1. 提取 `name` 和  `replacer` 属性
2. 提取子标签 `arg-type` 
3. 完成对象组装并为 Bean Definition 设置数据

在了解执行流程后我们来看执行完成后的 Bean Definition 对象的信息

- 执行 `parseReplacedMethodSubElements` 方法后的 Bean Definition

  ![image-20210107134913572](./images/image-20210107134913572.png)



同样的笔者在这里提出一个问题：在执行 `apple.hello()` 时发生了什么？这个问题的查找方式笔者在 `lookup-override` 的时候有提到过一部分，只不过那是关于 `lookup-override` 的和 `replaced-method` 无关。

首先看 `apple` 这个对象是什么

![image-20210107135216085](./images/image-20210107135216085.png)

从图中可以发现 `apple` 是一个代理对象，在这个截图中我们可以看到一个叫做 `ReplaceOverrideMethodInterceptor` 的名字，这个就是我们真正需要找到的目标，在 `ReplaceOverrideMethodInterceptor` 中藏有真正的方法执行过程。

- `ReplaceOverrideMethodInterceptor#intercept` 方法详情

```java
@Override
public Object intercept(Object obj, Method method, Object[] args, MethodProxy mp) throws Throwable {
   ReplaceOverride ro = (ReplaceOverride) getBeanDefinition().getMethodOverrides().getOverride(method);
   Assert.state(ro != null, "ReplaceOverride not found");
   // TODO could cache if a singleton for minor performance optimization
   MethodReplacer mr = this.owner.getBean(ro.getMethodReplacerBeanName(), MethodReplacer.class);
   return mr.reimplement(obj, method, args);
}
```



在这个方法中 Spring 根据 `method` 进行 `ReplaceOverride` 的搜索，在这 `ReplaceOverride` 就是我们前面提到的标签解析的结果

![image-20210107135501672](./images/image-20210107135501672.png)

回过头我们来看看一开始的配置文件:  `spring-replaced-method.xml` 

在配置文件中我们配置了下面这段内容。

```xml
<bean id="methodReplacerApple" class="com.source.hot.ioc.book.pojo.replacer.MethodReplacerApple">
</bean>
```



哦~这样子我们就可以全部理解了，从容器中获取 `methodReplacerBeanName` 对应的 Bean 实例，在调用 `MethodReplacer` 实现类的相关方法，通过这样的方式就替换了原有的方法。

到这我们对 `parseReplacedMethodSubElements` 的方法就分析完成了，下面我们将进入 `constructor-arg` 标签的解析





### 6.3.8 constructor-arg 标签处理

在开始分析之前我们依然需要准备一个测试的相关代码。

首先编写一个构造函数，本例将延用  `PeopleBean` 对象进行改造

- `PeopleBean` 对象详情

```java
public class PeopleBean {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public PeopleBean() {
    }

    public PeopleBean(String name) {
        this.name = name;
    }
}
```

Java Bean 准备完成好了之后来编写 Spring XML 文件，这里采用的名称 `spring-constructor-arg.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <constructor-arg index="0" type="java.lang.String" value="zhangsan"/>
    </bean>

</beans>
```



在配置文件编写完成之后来编写单元测试相关代码

```java
@Test
void testConstructArg() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-constructor-arg.xml");
    PeopleBean people = context.getBean("people", PeopleBean.class);
    assert people.getName().equals("zhangsan");
}
```



测试用例准备完毕下面我们找到真正的处理方法，处理方法的方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseConstructorArgElements`



- `parseConstructorArgElements` 方法详情

```java
public void parseConstructorArgElements(Element beanEle, BeanDefinition bd) {
   // 获取
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      if (isCandidateElement(node) && nodeNameEquals(node, CONSTRUCTOR_ARG_ELEMENT)) {
         // 解析 constructor-arg 下级标签
         parseConstructorArgElement((Element) node, bd);
      }
   }
}
```



其实最终解析的方法实在 `parseConstructorArgElement` 中，先来看第一部分的代码，

```java
// 获取 index 属性
String indexAttr = ele.getAttribute(INDEX_ATTRIBUTE);
// 获取 type 属性
String typeAttr = ele.getAttribute(TYPE_ATTRIBUTE);
// 获取 name 属性
String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
```



在 `parseConstructorArgElement` 方法得第一部分中很好理解，获取 `constructor-arg` 标签的属性。

当获取完成属性后 Spring 会根据 `index` 是否存在做出两种不同的处理，在本例中我们的 `index` 属性是存在的，下面我们对于存在的情况进行分析。

先看代码



```java
try {
   // 构造参数的所以未知
   int index = Integer.parseInt(indexAttr);
   if (index < 0) {
      error("'index' cannot be lower than 0", ele);
   }
   else {
      try {
         // 设置 阶段 构造函数处理阶段
         this.parseState.push(new ConstructorArgumentEntry(index));
         // 解析 property 标签
         Object value = parsePropertyValue(ele, bd, null);
         // 创建 构造函数的 属性控制类
         ConstructorArgumentValues.ValueHolder valueHolder = new ConstructorArgumentValues.ValueHolder(value);
         if (StringUtils.hasLength(typeAttr)) {
            // 类型设置
            valueHolder.setType(typeAttr);
         }
         if (StringUtils.hasLength(nameAttr)) {
            // 名称设置
            valueHolder.setName(nameAttr);
         }
         // 源设置
         valueHolder.setSource(extractSource(ele));
         if (bd.getConstructorArgumentValues().hasIndexedArgumentValue(index)) {
            error("Ambiguous constructor-arg entries for index " + index, ele);
         }
         else {
            // 添加 构造函数信息
            bd.getConstructorArgumentValues().addIndexedArgumentValue(index, valueHolder);
         }
      }
      finally {
         // 移除当前阶段
         this.parseState.pop();
      }
   }
}
catch (NumberFormatException ex) {
   error("Attribute 'index' of tag 'constructor-arg' must be an integer", ele);
}
```

在这段代码中至关重要的对象是 `ConstructorArgumentValues`。注意一点在解析 `constructor-arg` 标签的时候会有关于 `property`的解析这部分解析在下一节进行 (`parsePropertyValue` 方法的分析暂时跳过) 。

标签中 `index` 、`value` 和 `type` 最终被解析到那个对象了？这三个值分别被解析到 `ConstructorArgumentValues.ValueHolder` 对象和 `ConstructorArgumentValues` 中。在知道存储对象后我们来看看进行解析后的结果

- `type` 和 `value` 的存储

  ![image-20210107145210644](./images/image-20210107145210644.png)

- `index` 、`type` 和 `value` 的存储

  ![image-20210107145257472](./images/image-20210107145257472.png)



现在我们对于使用 `index` 方式的源码分析已经完成，下面我们一起来看看不使用 `index` 的情况下 Spring 是如何处理的。

进行非 `index` 模式的分析之前我们还是需要编写一个测试用例。

- 修改 `spring-constructor-arg.xml` 中的内容

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="people2" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <constructor-arg name="name" type="java.lang.String" value="zhangsan">
        </constructor-arg>
    </bean>

</beans>
```

编写单元测试

```java
@Test
void testConstructArgForName() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-constructor-arg.xml");
    PeopleBean people = context.getBean("people2", PeopleBean.class);
    assert people.getName().equals("zhangsan");
}
```



笔者将这种模式称之为根据参数名称绑定数据。

分析之前先来阅读处理这种情况的代码

```java
try {
   // 设置 阶段 构造函数处理阶段
   this.parseState.push(new ConstructorArgumentEntry());
   // 解析 property 标签
   Object value = parsePropertyValue(ele, bd, null);
   // 创建 构造函数的 属性控制类
   ConstructorArgumentValues.ValueHolder valueHolder = new ConstructorArgumentValues.ValueHolder(value);
   if (StringUtils.hasLength(typeAttr)) {
      // 类型设置
      valueHolder.setType(typeAttr);
   }
   if (StringUtils.hasLength(nameAttr)) {
      // 名称设置
      valueHolder.setName(nameAttr);
   }
   // 源设置
   valueHolder.setSource(extractSource(ele));
   // 添加 构造函数信息
   bd.getConstructorArgumentValues().addGenericArgumentValue(valueHolder);
}
finally {
   // 移除当前阶段
   this.parseState.pop();
}
```



各位可以将这段代码和 `index` 处理的代码进行比较，两者的差异就是 `index` 的控制 其他的对象信息还是一样的 ，关键对象: `ConstructorArgumentValues.ValueHolder` ，下面我们来看看经过这段方法后的 `valueHolder`

- 非 `index` 模式下的 `valueHolder` 对象

  ![image-20210107160926730](./images/image-20210107160926730.png)



在这个方法最后我们来看看 Bean Definition 的对象信息

![image-20210107161014020](./images/image-20210107161014020.png)





信息全部准备完成了，笔者此时存在一个问题：构造函数的信息都准备完成完毕如何使用？笔者在这儿仅做一个方法的标记，各位读者可以根据需要展开阅读。

处理方法是 ： `org.springframework.beans.BeanUtils#instantiateClass(java.lang.reflect.Constructor<T>, java.lang.Object...)` 这是最底层的一个方法，再往外可以看到跟多的细节，下图是一个调用堆栈![image-20210107161512489](./images/image-20210107161512489.png)





到这儿两种 `constructor-arg` 方式的处理都分析完成，下面我们就进入 `property` 标签的分析了





### 6.3.9 property 标签处理

开始分析之前我们需要进行基本用例的编写。

首先编写一个 Spring XML 配置文件，该文件笔者将其命名为: `spring-property.xml` ，往该文件中添加下面代码

- `spring-property.xml` 详细信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <property name="name" value="zhangsan"/>
    </bean>

</beans>
```

完成 Spring XML 编写后我们需要进行单元测试代码的编写



```java
@Test
void testProperty(){
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-property.xml");
    PeopleBean people = context.getBean("people", PeopleBean.class);
    assert people.getName().equals("zhangsan");
}
```



测试用例准备完毕, 下面我们真正的处理方法 `parsePropertyElements`，方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parsePropertyElements`

方法找到了请各位读者先进行方法阅读

```java
public void parsePropertyElements(Element beanEle, BeanDefinition bd) {
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      // 是否存在 property 标签
      if (isCandidateElement(node) && nodeNameEquals(node, PROPERTY_ELEMENT)) {
         // 解析单个标签
         parsePropertyElement((Element) node, bd);
      }
   }
}
```

在上面这段代码中看不出什么细节，核心代码还是得在 `parsePropertyElement` 方法中找到。请各位读者阅读下面代码。

- `parsePropertyElement` 方法详情

```java
public void parsePropertyElement(Element ele, BeanDefinition bd) {
   String propertyName = ele.getAttribute(NAME_ATTRIBUTE);
   if (!StringUtils.hasLength(propertyName)) {
      error("Tag 'property' must have a 'name' attribute", ele);
      return;
   }
   this.parseState.push(new PropertyEntry(propertyName));
   try {
      if (bd.getPropertyValues().contains(propertyName)) {
         error("Multiple 'property' definitions for property '" + propertyName + "'", ele);
         return;
      }
      // 解析 property 标签
      Object val = parsePropertyValue(ele, bd, propertyName);
      // 构造 PropertyValue 对象
      PropertyValue pv = new PropertyValue(propertyName, val);
      // 解析元信息
      parseMetaElements(ele, pv);
      pv.setSource(extractSource(ele));
      // 添加 pv 结构
      bd.getPropertyValues().addPropertyValue(pv);
   }
   finally {
      this.parseState.pop();
   }
}
```



在 `parsePropertyElement` 方法中分为下面四个步骤进行。

1. 第一步：提取 `property` 标签的 `name` 属性
2. 第二步：提取 `property` 标签的 `value` 属性
3. 第三步：解析可能存在的 `meta` 标签
4. 第四步：赋值 Bean Definition 的属性

- 在整个流程中有一个对象很关键，这个对象存储了关于 `property` 标签的数据信息，这个对象是 `PropertyValue `

在这四步操作中，第一步和第三步各位读者应该有一个很好的认识了。下面着重对第二步和第四步进行分析，先进行第二步的分析。

第二步是一个 `parsePropertyValue` 方法的调用，在 `parsePropertyValue` 方法中处理围绕 `property` 进行，那么我们首先就需要了解  `property` 标签的定义。

- 在 `spring-beans.dtd` 中对于 `property` 标签的定义如下

```xml-dtd
<!ELEMENT property (
	description?, meta*,
	(bean | ref | idref | value | null | list | set | map | props)?
)>

<!--
	The property name attribute is the name of the JavaBean property.
	This follows JavaBean conventions: a name of "age" would correspond
	to setAge()/optional getAge() methods.
-->
<!ATTLIST property name CDATA #REQUIRED>

<!--
	A short-cut alternative to a child element "ref bean=".
-->
<!ATTLIST property ref CDATA #IMPLIED>

<!--
	A short-cut alternative to a child element "value".
-->
<!ATTLIST property value CDATA #IMPLIED>
```

- 在 `spring-beans.xsd` 中对于 `property` 的定义如下

```xml
<xsd:element name="property" type="propertyType">
 <xsd:annotation>
    <xsd:documentation><![CDATA[
Bean definitions can have zero or more properties.
Property elements correspond to JavaBean setter methods exposed
by the bean classes. Spring supports primitives, references to other
beans in the same or related factories, lists, maps and properties.
    ]]></xsd:documentation>
 </xsd:annotation>
</xsd:element>


<xsd:complexType name="propertyType">
  <xsd:sequence>
      <xsd:element ref="description" minOccurs="0"/>
      <xsd:choice minOccurs="0" maxOccurs="1">
          <xsd:element ref="meta"/>
          <xsd:element ref="bean"/>
          <xsd:element ref="ref"/>
          <xsd:element ref="idref"/>
          <xsd:element ref="value"/>
          <xsd:element ref="null"/>
          <xsd:element ref="array"/>
          <xsd:element ref="list"/>
          <xsd:element ref="set"/>
          <xsd:element ref="map"/>
          <xsd:element ref="props"/>
          <xsd:any namespace="##other" processContents="strict"/>
      </xsd:choice>
  </xsd:sequence>
  <xsd:attribute name="name" type="xsd:string" use="required">
      <xsd:annotation>
          <xsd:documentation><![CDATA[
The name of the property, following JavaBean naming conventions.
  ]]></xsd:documentation>
      </xsd:annotation>
  </xsd:attribute>
  <xsd:attribute name="ref" type="xsd:string">
      <xsd:annotation>
          <xsd:documentation><![CDATA[
A short-cut alternative to a nested "<ref bean='...'/>".
  ]]></xsd:documentation>
      </xsd:annotation>
  </xsd:attribute>
  <xsd:attribute name="value" type="xsd:string">
      <xsd:annotation>
          <xsd:documentation><![CDATA[
A short-cut alternative to a nested "<value>...</value>" element.
  ]]></xsd:documentation>
      </xsd:annotation>
  </xsd:attribute>
</xsd:complexType>

```



在了解完成 `property` 标签的定义后我们来看 `parsePropertyValue` 方法的代码

- 首先请各位阅读 `parsePropertyValue` 完整代码

```java
@Nullable
public Object parsePropertyValue(Element ele, BeanDefinition bd, @Nullable String propertyName) {
   String elementName = (propertyName != null ?
         "<property> element for property '" + propertyName + "'" :
         "<constructor-arg> element");

   // 计算子节点
   // Should only have one child element: ref, value, list, etc.
   NodeList nl = ele.getChildNodes();
   Element subElement = null;
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      if (node instanceof Element && !nodeNameEquals(node, DESCRIPTION_ELEMENT) &&
            !nodeNameEquals(node, META_ELEMENT)) {
         // Child element is what we're looking for.
         if (subElement != null) {
            error(elementName + " must not contain more than one sub-element", ele);
         }
         else {
            subElement = (Element) node;
         }
      }
   }

   // ref 属性是否存在
   boolean hasRefAttribute = ele.hasAttribute(REF_ATTRIBUTE);
   // value 属性是否存在
   boolean hasValueAttribute = ele.hasAttribute(VALUE_ATTRIBUTE);
   if ((hasRefAttribute && hasValueAttribute) ||
         ((hasRefAttribute || hasValueAttribute) && subElement != null)) {
      error(elementName +
            " is only allowed to contain either 'ref' attribute OR 'value' attribute OR sub-element", ele);
   }

   if (hasRefAttribute) {
      // 获取 ref 属性值
      String refName = ele.getAttribute(REF_ATTRIBUTE);
      if (!StringUtils.hasText(refName)) {
         error(elementName + " contains empty 'ref' attribute", ele);
      }
      // 创建 连接对象
      RuntimeBeanReference ref = new RuntimeBeanReference(refName);

      ref.setSource(extractSource(ele));
      return ref;
   }
   else if (hasValueAttribute) {
      // 获取 value
      TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
      valueHolder.setSource(extractSource(ele));
      return valueHolder;
   }
   else if (subElement != null) {
      return parsePropertySubElement(subElement, bd);
   }
   else {
      // Neither child element nor "ref" or "value" attribute found.
      error(elementName + " must specify a ref or value", ele);
      return null;
   }
}
```



在 `parsePropertyValue` 代码中我们可以分为这些步骤来进行阅读。

1. 第一步：提取 `property` 下的子节点标签中非 `description` 和 `meta` 标签。
2. 第二步：提取 `property` 中的 `ref` 属性 ，存在的情况下创建 `RuntimeBeanReference` 对象并返回。
3. 第三步：提取 `property` 中的 `value` 属性，存在的情况下创建 `TypedStringValue` 对象并返回。
4. 第四步：解析第一步中得到的子标签信息

了解一下 `property` 的子标签有那些：`meta` 、`bean`、`ref`、`idref`、`value`、`null`、`array`、`list`、`set`、`map`、`props`。在这些标签中排除 `meta` 和 `description` 就是第一步会得到的标签(节点)。

第二步获取 `ref` 属性，获取属性的动作很简单，直接用 `getAttribute` 方法就可以了，创建 `RuntimeBeanReference` 对象也是一个 `new ` 的简单过程。

第三步和第二步的实际操作逻辑是一样的，获取标签中的属性名称，创建对象。

第四步处理子标签在前文笔者告诉了各位可能存在的标签有哪些，那么在这个步骤中就是要对这些标签做解析然后转换成 Java 对象了。第四步的方法是 `parsePropertySubElement` 

笔者在这简单描述一下关于 `list`、`map` 这两个的解析

首先我们对 Java Bean 进行改造

```java
public class PeopleBean {
    private String name;

    private List<String> list;

    private Map<String, String> map;
    // 省略构造函数,getter,setter
}
```

修改完成 Java Bean 后编辑 Spring xml 配置文件，修改后的信息如下

```xml
<bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
    <property name="name" value="zhangsan"/>
    <property name="list">
        <list value-type="java.lang.String" merge="default">
            <value>a</value>
            <value>b</value>
        </list>

    </property>

    <property name="map">
        <map key-type="java.lang.String" value-type="java.lang.String">
            <entry key="a" value="1"/>
        </map>
    </property>
</bean>

```



找到 `list` 标签的解析方法 `parseListElement` 这里需要提一点对于 `list` 、`array` 、`set` 的处理最终都是交给 `parseCollectionElements` 来获取最终数据的，在这里就是用来解析 `value` 标签中的字符串量, 下面我们来看经过 `parseListElement` 方法处理后的结果![image-20210108094305469](./images/image-20210108094305469.png)



下面我们来看关于 `map` 标签的处理，在 xml 文件中我们编写了 `map` 标签的 `key-type` 、`value-type` 和 子标签 `entry` 并 `entry` 中填写了 `key` 和 `value` 这些属性。在本例中我们就仅作字符串这样的直接字面量作为解析，不展开做 `ref` 相关的解析，读者有兴趣可以进行拓展阅读 `parseMapElement` 方法的完整细节。

在处理 `list` 标签的时候我们采取的是字面量得到的对象类型是 `TypedStringValue` ，在 `map` 标签处理中用到的也是这个类型，只不过是从 `list` 存储转换成 `map` 存储。下面我们来看处理后的结果

![image-20210108100042152](./images/image-20210108100042152.png)

关于各类子标签的细节处理笔者没有在这里展开描述每个方法及方法中的细节，各位读者需要自行阅读。







好的，到现在笔者将 `property` 标签在 Spring 中的处理以及分析完毕，下面我们将进入 `qualifier` 标签的处理。





### 6.3.10 qualifier 标签处理

开始分析之前我们需要进行基本用例的编写。

因为 `qualifier` 标签一般情况下是用来指定注入类的名称的所以需要创建一个新的对象

- `PeopleBeanTwo` 对象详情

```java
public class PeopleBeanTwo {
    @Autowired
    @Qualifier("p1")
    private PeopleBean peopleBean;
	// 省略 getter 和 setter 
}
```

在 `PeopleBeanTwo` 中所采用的 `PeopleBean` 对象就是之前使用的对象不在此进行贴代码了。下面编写 Spring xml 配置文件。

- `spring-qualifier.xml` 详细信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean id="p2" class="com.source.hot.ioc.book.pojo.PeopleBeanTwo">

    </bean>

    <bean id="peopleBean" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <property name="name" value="zhangsan"/>
        <qualifier value="p1"/>
    </bean>
</beans>
```

最后就是单元测试的编写

```java
@Test
void testQualifier() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-qualifier.xml");
    PeopleBeanTwo peopleTwo = context.getBean("p2", PeopleBeanTwo.class);
    assert peopleTwo.getPeopleBean().equals(context.getBean("peopleBean", PeopleBean.class));
}
```



现在我们拥有了基本的测试用例下面我们就开始 `parseQualifierElements` 方法的分析。

先来阅读 `parseQualifierElements` 方法详情

```java
public void parseQualifierElements(Element beanEle, AbstractBeanDefinition bd) {
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      if (isCandidateElement(node) && nodeNameEquals(node, QUALIFIER_ELEMENT)) {
         // 单个解析
         parseQualifierElement((Element) node, bd);
      }
   }
}
```

由于 `qualifier` 标签是可以多个同时存在的，在 Spring 中做处理会将单个处理过程交给 `parseQualifierElement` 方法进行，下面我们来对这个方法进行分析。

首先来看 `parseQualifierElement` 的完整内容。

- `parseQualifierElement` 方法详情

```java
public void parseQualifierElement(Element ele, AbstractBeanDefinition bd) {
   // 获取 type 属性
   String typeName = ele.getAttribute(TYPE_ATTRIBUTE);
   if (!StringUtils.hasLength(typeName)) {
      error("Tag 'qualifier' must have a 'type' attribute", ele);
      return;
   }
   // 设置阶段 处理 qualifier 阶段
   this.parseState.push(new QualifierEntry(typeName));
   try {
      // 自动注入对象创建
      AutowireCandidateQualifier qualifier = new AutowireCandidateQualifier(typeName);
      // 设置源
      qualifier.setSource(extractSource(ele));
      // 获取 value 属性
      String value = ele.getAttribute(VALUE_ATTRIBUTE);
      if (StringUtils.hasLength(value)) {
         // 设置 属性 value , value
         qualifier.setAttribute(AutowireCandidateQualifier.VALUE_KEY, value);
      }
      NodeList nl = ele.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (isCandidateElement(node) && nodeNameEquals(node, QUALIFIER_ATTRIBUTE_ELEMENT)) {
            Element attributeEle = (Element) node;
            // 获取 key 属性
            String attributeName = attributeEle.getAttribute(KEY_ATTRIBUTE);
            // 获取 value 属性
            String attributeValue = attributeEle.getAttribute(VALUE_ATTRIBUTE);
            if (StringUtils.hasLength(attributeName) && StringUtils.hasLength(attributeValue)) {
               // key value 属性映射
               BeanMetadataAttribute attribute = new BeanMetadataAttribute(attributeName, attributeValue);
               attribute.setSource(extractSource(attributeEle));
               // 添加 qualifier 属性值
               qualifier.addMetadataAttribute(attribute);
            }
            else {
               error("Qualifier 'attribute' tag must have a 'name' and 'value'", attributeEle);
               return;
            }
         }
      }
      // 添加 qualifier
      bd.addQualifier(qualifier);
   }
   finally {
      // 移除阶段
      this.parseState.pop();
   }
}
```



这个解析过程就是对 `qualifier` 标签提取各个属性值然后转换成 Java 对象，对应 `qualifier` 标签的 Java 对象是 `AutowireCandidateQualifier` 下面我们来看看解析的结果是什么样子的。

![image-20210108105829769](./images/image-20210108105829769.png)

上图便是经过计算后 `qualifier` 标签解析后的 Java 对象。

至此 Spring 中关于 `bean` 标签的所有可能情况分析已经都做完了，最后还剩下两个属性没有设置给 Bean Definition 我们来看看最后的细节吧。



### 6.3.11 Bean Definition 的最后两个属性

首先我们需要知道最后两个属性是什么，第一个属性是 `Resource` 资源、第二个属性是 `source` ，对于这两个属性的分析我们就不用单独编写 Spring xml 配置文件了，本文提到的任何一个都可以用来作为测试。提出一个问题：**什么是资源呢？** 在 Java 工程中有一个 `resource` 文件夹，一般的笔者认为这个就是资源，**在 Spring 中资源的定义是指 Spring xml 配置文件。**

- `resource` 信息

  ![image-20210108111206299](./images/image-20210108111206299.png)

理解了 `Resource` 对象在 Spring 中的含义下面我们来对 `source` 对象进行了解。在 Spring 中对于 `source` 提供了专门的解析工具 (解析类 `SourceExtractor` ) ， 下面我们先来看看 Spring 中提供的解析工具有哪些实现类，实现类中分别做了什么。

- `SourceExtractor` 类图

  ![SourceExtractor](./images/SourceExtractor.png)

Spring 中提供了两个关于 `SourceExtractor` 的实现方式。

1. 第一种：`NullSourceExtractor` 实现类直接返回 `null` 作为解析结果。
2. 第二种：`PassThroughSourceExtractor` 实现类将对象本身作为解析结果。

在 `bean` 标签解析的过程中 `SourceExtractor` 的具体实现类是 `NullSourceExtractor` 所以在设置 Bean Definition 的 `source` 属性是设置的数据是 `null`

![image-20210108111748982](./images/image-20210108111748982.png)



到此笔者对于 `bean` 标签的分析方法已经彻底完成。当 Spring 解析完成 `bean` 标签转换成 Bean Definition 之后 Spring 做了什么呢？







## 6.4 Bean Definition 装饰

在 `DefaultBeanDefinitionDocumentReader#processBeanDefinition` 方法中我们可以了解，在获取 Bean Definition 对象后做的事情是装饰(数据补充) 

- `DefaultBeanDefinitionDocumentReader#processBeanDefinition` 方法详情

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



下面我们就来开始对 `decorateBeanDefinitionIfRequired` 这个方法进行分析。首先请各位阅读 `BeanDefinitionParserDelegate#decorateBeanDefinitionIfRequired` 方法

- `BeanDefinitionParserDelegate#decorateBeanDefinitionIfRequired` 方法详情

```java
public BeanDefinitionHolder decorateBeanDefinitionIfRequired(
      Element ele, BeanDefinitionHolder originalDef, @Nullable BeanDefinition containingBd) {

   BeanDefinitionHolder finalDefinition = originalDef;

   // Decorate based on custom attributes first.
   NamedNodeMap attributes = ele.getAttributes();
   for (int i = 0; i < attributes.getLength(); i++) {
      Node node = attributes.item(i);
      finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
   }

   // Decorate based on custom nested elements.
   NodeList children = ele.getChildNodes();
   for (int i = 0; i < children.getLength(); i++) {
      Node node = children.item(i);
      if (node.getNodeType() == Node.ELEMENT_NODE) {
         finalDefinition = decorateIfRequired(node, finalDefinition, containingBd);
      }
   }
   return finalDefinition;
}
```



在看完这段代码后我们需要知道那些情况下会做 Bean Definition 对象的装饰行为。这里的装饰行为就是赋值数据。

会做 Bean Definition 装饰的可能情况:

1. 第一种：当 `bean` 标签存在属性时
2. 第二种：当 `bean` 标签存在下级标签时



从 Spring 的源代码上我们可以发现处理的方式都是通过 `decorateIfRequired` 方法进行。那么我们的主要目标就是 `decorateIfRequired` 方法。各位读者请先看代码

- `decorateIfRequired` 方法详情

```java
public BeanDefinitionHolder decorateIfRequired(
      Node node, BeanDefinitionHolder originalDef, @Nullable BeanDefinition containingBd) {

   // 命名空间 url
   String namespaceUri = getNamespaceURI(node);
   if (namespaceUri != null && !isDefaultNamespace(namespaceUri)) {
      NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
      if (handler != null) {
         // 命名空间进行装饰
         BeanDefinitionHolder decorated =
               handler.decorate(node, originalDef, new ParserContext(this.readerContext, this, containingBd));
         if (decorated != null) {
            return decorated;
         }
      }
      else if (namespaceUri.startsWith("http://www.springframework.org/schema/")) {
         error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", node);
      }
      else {
         // A custom namespace, not to be handled by Spring - maybe "xml:...".
         if (logger.isDebugEnabled()) {
            logger.debug("No Spring NamespaceHandler found for XML schema namespace [" + namespaceUri + "]");
         }
      }
   }
   return originalDef;
}
```

在这个正在处理装饰的过程中各位可以发现这里真正用到的对象又是 `NamespaceHandler` ，在[第四章](/docs/ch-04/第四章-自定义标签解析.md) 中笔者对 `NamespaceHandler` 做了关于自定义标签解析的内容，下面我们将第四章的用例延续。

首先修改 `UserXsdNamespaceHandler` 对象补充 `decorate` 方法的实现类

```java
public class UserXsdNamespaceHandler extends NamespaceHandlerSupport {

    @Override
    public void init() {
        registerBeanDefinitionParser("user_xsd", new UserXsdParser());
    }

    @Override
    public BeanDefinitionHolder decorate(Node node, BeanDefinitionHolder definition, ParserContext parserContext) {
        BeanDefinition beanDefinition = definition.getBeanDefinition();
        beanDefinition.getPropertyValues().addPropertyValue("namespace", "namespace");
        return definition;
    }
}

```

修改第四章中的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:myname="http://www.huifer.com/schema/user"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.huifer.com/schema/user http://www.huifer.com/schema/user.xsd
">
    <bean id="p1" class="com.source.hot.ioc.book.pojo.PeopleBean">
        <myname:user_xsd id="testUserBean" name="huifer" idCard="123"/>
    </bean>
</beans>
```

查看单元测试代码

```java
@Test
void testXmlCustom() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/custom-xml.xml");
    UserXsd testUserBean = context.getBean("testUserBean", UserXsd.class);
    assert testUserBean.getName().equals("huifer");
    assert testUserBean.getIdCard().equals("123");
    context.close();
}
```



在第四章中已经讲述过从 `namespaceUri` 转换成 `NamespaceHandler` 的过程，那么下面这段代码相信各位肯定有所理解

```java
BeanDefinitionHolder decorated =
      handler.decorate(node, originalDef, new ParserContext(this.readerContext, this, containingBd));
```

这段方法最终会调用我们所编辑的 `UserXsdNamespaceHandler#decorate` 方法。下面我们来看一下经过装饰后的 Bean Definition ![image-20210108140520348](./images/image-20210108140520348.png)





到此这部分关于 Bean Definition 装饰的代码分析完成。

在了解 Bean Definition 的装饰过程后我们有一个值得思考和回味的问题，Bean Definition 里面到底是什么？



## 6.5 Bean Definition 细节

在这一节中笔者会将 Bean Definition 中的各个属性进行列举。下面我们先来看 Bean Definition 的类图

![BeanDefinition](./images/BeanDefinition.png)



根据这张类图我们先从 `AbstractBeanDefinition` 出发了解一下在 `AbstractBeanDefinition` 中定义了 Bean 的那些属性



### 6.5.1 AbstractBeanDefinition 的属性

| 属性名称                       | 属性类型                    | 属性含义                                                     |
| ------------------------------ | --------------------------- | ------------------------------------------------------------ |
| `beanClass`                    | `Object`                    | 存储 Bean 类型                                               |
| `scope`                        | `String`                    | 作用域，默认`""`，常见作用域有 `singleton`、`prototype`、`request`、`session` 和 `global session` |
| `lazyInit`                     | `Boolean`                   | 是否懒加载                                                   |
| `abstractFlag`                 | `boolean`                   | 是否是 `abstract` 修饰的                                     |
| `autowireMode`                 | `int`                       | 自动注入方式，常见的注入方式有 `no`、`byName` 、`byType` 和 `constructor` |
| `dependencyCheck`              | `int`                       | 以来检查级别，常见的依赖检查级别有 `DEPENDENCY_CHECK_NONE` 、`DEPENDENCY_CHECK_OBJECTS` 、`DEPENDENCY_CHECK_SIMPLE` 和 `DEPENDENCY_CHECK_ALL` |
| `dependsOn`                    | `String[]`                  | 依赖的 Bean Name 列表                                        |
| `autowireCandidate`            | `boolean`                   | 是否自动注入，默认值：`true`                                 |
| `primary`                      | `boolean`                   | 是否是主要的，通常在同类型多个备案的情况下使用               |
| `instanceSupplier`             | `Supplier`                  | Bean 实例提供器                                              |
| `nonPublicAccessAllowed`       | `boolean`                   | 是否禁止公开访问                                             |
| `lenientConstructorResolution` | `String`                    |                                                              |
| `factoryBeanName`              | `String`                    | 工厂 Bean 名称                                               |
| `factoryMethodName`            | `String`                    | 工厂函数名称                                                 |
| `constructorArgumentValues`    | `ConstructorArgumentValues` | 构造函数对象，可以是 XML 中 `constructor-arg` 标签的解析结果，也可以是 Java 中构造函数的解析结果 |
| `propertyValues`               | `MutablePropertyValues`     | 属性列表                                                     |
| `methodOverrides`              | `MethodOverrides`           | 重写的函数列表                                               |
| `initMethodName`               | `String`                    | Bean 初始化的函数名称                                        |
| `destroyMethodName`            | `String`                    | Bean 摧毁的函数名称                                          |
| `enforceInitMethod`            | `boolean`                   | 是否强制执行 `initMethodName`对应的 Java 方法                |
| `enforceDestroyMethod`         | `boolean`                   | 是否强制执行 `destroyMethodName` 对应的 Java 方法            |
| `synthetic`                    | `boolean`                   | 合成标记                                                     |
| `role`                         | `int`                       | Spring 中的角色，一般有 `ROLE_APPLICATION` 、`ROLE_SUPPORT` 和 `ROLE_INFRASTRUCTURE` |
| `description`                  | `String`                    | Bean 的描述信息                                              |
| `resource`                     | `Resource`                  | 资源对象                                                     |



### 6.5.2 RootBeanDefinition 的属性



| 属性名称                           | 属性类型               | 属性含义                               |
| ---------------------------------- | ---------------------- | -------------------------------------- |
| `constructorArgumentLock`            | `Object`                 | 构造阶段的锁                           |
| `postProcessingLock`                 | `Object`                 | 后置处理阶段的锁                       |
| `stale`                              | `boolean`                | 是否需要重新合并定义                   |
| `allowCaching`                       | `boolean`                | 是否缓存                               |
| `isFactoryMethodUnique`              | `boolean`                | 工厂方法是否唯一                       |
| `targetType`                         | `ResolvableType`         | 目标类型                               |
| `resolvedTargetType`·                | `Class`                  | 目标类型, Bean 的类型                  |
| `isFactoryBean`                      | `Boolean`                | 是否是工厂 Bean                        |
| `factoryMethodReturnType`            | `ResolvableType`         | 工厂方法返回值                         |
| `factoryMethodToIntrospect`          | `Method`                 |                                        |
| `resolvedConstructorOrFactoryMethod` | `Executable`             | 执行器，可能是构造函数也可能是工厂方法 |
| `constructorArgumentsResolved`       | `boolean`                | 构造函数的参数是否需要解析             |
| `resolvedConstructorArguments`       | `Object[]`               | 解析过的构造参数列表，这里是具体的实例 |
| `preparedConstructorArguments`       | `Object[]`               | 未解析的构造参数列表                   |
| `postProcessed`                      | `boolean`              | 是否需要进行后置处理                   |
| `beforeInstantiationResolved`        | `Boolean`              | 是否需要进行前置处理                   |
| `decoratedDefinition`              | `BeanDefinitionHolder` | Bean Definition 持有者                 |
| `qualifiedElement`                 | `AnnotatedElement`       | qualified 注解信息                     |
| `externallyManagedConfigMembers`   | `Set<Member>`          | 外部配置的成员                         |
| `externallyManagedInitMethods`     | `Set<String>`          | 外部的初始化方法列表                   |
| `externallyManagedDestroyMethods`  | `Set<String>`          | 外部的摧毁方法列表                     |





### 6.5.3 ChildBeanDefinition 的属性

| 属性名称     | 属性类型 | 属性含义                  |
| ------------ | -------- | ------------------------- |
| `parentName` | `String` | 父 Bean Definition 的名称 |





### 6.5.4 GenericBeanDefinition 的属性

| 属性名称     | 属性类型 | 属性含义                  |
| ------------ | -------- | ------------------------- |
| `parentName` | `String` | 父 Bean Definition 的名称 |



### 6.5.5 AnnotatedGenericBeanDefinition 的属性

| 属性名称                | 属性类型             | 属性含义         |
| ----------------------- | -------------------- | ---------------- |
| `metadata`              | `AnnotationMetadata` | 注解元信息       |
| `factoryMethodMetadata` | `MethodMetadata`     | 工厂函数的元信息 |



到此整个关于 Bean Definition 的分析全部完成，下面进行总结。



## 6.6 总结

- 在本章节中笔者主要对 `parseBeanDefinitionElement` 方法进行分析 (完整方法签名：`org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element, java.lang.String, org.springframework.beans.factory.config.BeanDefinition)` )，下面笔者对整个处理过程进行总结。

1. 第一步：处理 `className` 和 `parent` 属性
2. 第二步：创建基本的 Bean Definition 对象 , 具体类: `AbstractBeanDefinition` 、`GenericBeanDefinition`
3. 第三步：读取 `bean` 标签的属性，为 Bean Definition 对象进行赋值
4. 第四步：处理描述标签 `description`
5. 第五步：处理 `meta` 标签
6. 第六步：处理 `lookup-override` 标签
7. 第七步：处理 `replaced-method` 标签
8. 第八步：处理 `constructor-arg` 标签
9. 第九步：处理 `property` 标签
10. 第十步：处理 `qualifier` 标签
11. 第十一步：设置资源对象
12. 第十二步：设置 `source` 属性

上述这十二步就是整个 Spring 中对于 `bean` 标签的处理细节。在这之后 Spring 会进行 Bean Definition 的装饰行为，装饰行为的操作如下。

1. 第一步：读取标签所对应的 `namespaceUri` 
2. 第二步：根据 `namespaceUri` 在 `NamespaceHandler` 容器中寻找对应的  `NamespaceHandler`
3. 第三步：调用 `NamespaceHandler` 所提供的 `decorate` 方法。



在完成了上述两个大步骤后我们就需要进入 Bean Definition 的注册相关内容，这部分笔者在[第七章](/docs/ch-07/第七章-BeanDefinition注册.md)中做分析。





最后笔者将 `bean` 标签的解析方法贴出各位可以再进入源码中进行阅读。

- `bean` 标签解析的入喉方法 `processBeanDefinition`

  方法签名: `org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#processBeanDefinition`

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



