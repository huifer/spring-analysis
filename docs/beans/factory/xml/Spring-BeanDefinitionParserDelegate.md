# BeanDefinitionParserDelegate
- 类全路径: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate`
- 类作用: xml Bean定义解析的委托类.





## 成员变量

- 在`BeanDefinitionParserDelegate`中定义了多个个静态变量
  1. beans 的命名空间url： `public static final String BEANS_NAMESPACE_URI = "http://www.springframework.org/schema/beans";`
  2. 分隔符: `public static final String MULTI_VALUE_ATTRIBUTE_DELIMITERS = ",; ";`





其他静态变量就不一个个贴出来了直接放代码



```java
/**
 * Value of a T/F attribute that represents true.
 * Anything else represents false. Case seNsItive.
 */
public static final String TRUE_VALUE = "true";

public static final String FALSE_VALUE = "false";

public static final String DEFAULT_VALUE = "default";

public static final String DESCRIPTION_ELEMENT = "description";

public static final String AUTOWIRE_NO_VALUE = "no";

public static final String AUTOWIRE_BY_NAME_VALUE = "byName";

public static final String AUTOWIRE_BY_TYPE_VALUE = "byType";

public static final String AUTOWIRE_CONSTRUCTOR_VALUE = "constructor";

public static final String AUTOWIRE_AUTODETECT_VALUE = "autodetect";

public static final String NAME_ATTRIBUTE = "name";

public static final String BEAN_ELEMENT = "bean";

public static final String META_ELEMENT = "meta";

public static final String ID_ATTRIBUTE = "id";

public static final String PARENT_ATTRIBUTE = "parent";

public static final String CLASS_ATTRIBUTE = "class";

public static final String ABSTRACT_ATTRIBUTE = "abstract";

public static final String SCOPE_ATTRIBUTE = "scope";

public static final String LAZY_INIT_ATTRIBUTE = "lazy-init";

public static final String AUTOWIRE_ATTRIBUTE = "autowire";

public static final String AUTOWIRE_CANDIDATE_ATTRIBUTE = "autowire-candidate";

public static final String PRIMARY_ATTRIBUTE = "primary";

public static final String DEPENDS_ON_ATTRIBUTE = "depends-on";

public static final String INIT_METHOD_ATTRIBUTE = "init-method";

public static final String DESTROY_METHOD_ATTRIBUTE = "destroy-method";

public static final String FACTORY_METHOD_ATTRIBUTE = "factory-method";

public static final String FACTORY_BEAN_ATTRIBUTE = "factory-bean";

public static final String CONSTRUCTOR_ARG_ELEMENT = "constructor-arg";

public static final String INDEX_ATTRIBUTE = "index";

public static final String TYPE_ATTRIBUTE = "type";

public static final String VALUE_TYPE_ATTRIBUTE = "value-type";

public static final String KEY_TYPE_ATTRIBUTE = "key-type";

public static final String PROPERTY_ELEMENT = "property";

public static final String REF_ATTRIBUTE = "ref";

public static final String VALUE_ATTRIBUTE = "value";

public static final String LOOKUP_METHOD_ELEMENT = "lookup-method";

public static final String REPLACED_METHOD_ELEMENT = "replaced-method";

public static final String REPLACER_ATTRIBUTE = "replacer";

public static final String ARG_TYPE_ELEMENT = "arg-type";

public static final String ARG_TYPE_MATCH_ATTRIBUTE = "match";

public static final String REF_ELEMENT = "ref";

public static final String IDREF_ELEMENT = "idref";

public static final String BEAN_REF_ATTRIBUTE = "bean";

public static final String PARENT_REF_ATTRIBUTE = "parent";

public static final String VALUE_ELEMENT = "value";

public static final String NULL_ELEMENT = "null";

public static final String ARRAY_ELEMENT = "array";

public static final String LIST_ELEMENT = "list";

public static final String SET_ELEMENT = "set";

public static final String MAP_ELEMENT = "map";

public static final String ENTRY_ELEMENT = "entry";

public static final String KEY_ELEMENT = "key";

public static final String KEY_ATTRIBUTE = "key";

public static final String KEY_REF_ATTRIBUTE = "key-ref";

public static final String VALUE_REF_ATTRIBUTE = "value-ref";

public static final String PROPS_ELEMENT = "props";

public static final String PROP_ELEMENT = "prop";

public static final String MERGE_ATTRIBUTE = "merge";

public static final String QUALIFIER_ELEMENT = "qualifier";

public static final String QUALIFIER_ATTRIBUTE_ELEMENT = "attribute";

public static final String DEFAULT_LAZY_INIT_ATTRIBUTE = "default-lazy-init";

public static final String DEFAULT_MERGE_ATTRIBUTE = "default-merge";

public static final String DEFAULT_AUTOWIRE_ATTRIBUTE = "default-autowire";

public static final String DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE = "default-autowire-candidates";

public static final String DEFAULT_INIT_METHOD_ATTRIBUTE = "default-init-method";

public static final String DEFAULT_DESTROY_METHOD_ATTRIBUTE = "default-destroy-method";

private static final String SINGLETON_ATTRIBUTE = "singleton";
```





上面👆这些字符串常量就是在xml中的各类标签, 属性，这些静态变量会在解析xml文档时提供帮助，即规范xml文档内容预定义. 



除了这部分静态变量外还有一些解析时需要用到的成员变量





```java
/**
 * xml 阅读器上下文
 */
private final XmlReaderContext readerContext;

/**
 * 文档预设值. 
 */
private final DocumentDefaultsDefinition defaults = new DocumentDefaultsDefinition();

/**
 * 阶段容器
 */
private final ParseState parseState = new ParseState();

/**
 * Stores all used bean names so we can enforce uniqueness on a per
 * beans-element basis. Duplicate bean ids/names may not exist within the
 * same level of beans element nesting, but may be duplicated across levels.
 *
 * 已经使用过的beanName
 */
private final Set<String> usedNames = new HashSet<>();
```







下面就开始进行方法分析





## 方法分析

### populateDefaults

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#populateDefaults`
- 方法作用: 设置默认值



<details>
    <summary>populateDefaults 方法详情</summary>





```JAVA
protected void populateDefaults(DocumentDefaultsDefinition defaults, @Nullable DocumentDefaultsDefinition parentDefaults, Element root) {
   // 获取 default-lazy-init 属性值
   String lazyInit = root.getAttribute(DEFAULT_LAZY_INIT_ATTRIBUTE);
   // 判断是否是默认值
   if (isDefaultValue(lazyInit)) {
      // Potentially inherited from outer <beans> sections, otherwise falling back to false.
      lazyInit = (parentDefaults != null ? parentDefaults.getLazyInit() : FALSE_VALUE);
   }
   defaults.setLazyInit(lazyInit);

   // 获取 default-merge 属性值
   String merge = root.getAttribute(DEFAULT_MERGE_ATTRIBUTE);
   if (isDefaultValue(merge)) {
      // Potentially inherited from outer <beans> sections, otherwise falling back to false.
      merge = (parentDefaults != null ? parentDefaults.getMerge() : FALSE_VALUE);
   }
   defaults.setMerge(merge);

   // 获取 default-autowire 属性
   String autowire = root.getAttribute(DEFAULT_AUTOWIRE_ATTRIBUTE);
   if (isDefaultValue(autowire)) {
      // Potentially inherited from outer <beans> sections, otherwise falling back to 'no'.
      autowire = (parentDefaults != null ? parentDefaults.getAutowire() : AUTOWIRE_NO_VALUE);
   }
   defaults.setAutowire(autowire);

   // 是否存在 default-autowire-candidates
   if (root.hasAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE)) {
      defaults.setAutowireCandidates(root.getAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE));
   }
   else if (parentDefaults != null) {
      defaults.setAutowireCandidates(parentDefaults.getAutowireCandidates());
   }

   // 设置 default-init-method
   if (root.hasAttribute(DEFAULT_INIT_METHOD_ATTRIBUTE)) {
      defaults.setInitMethod(root.getAttribute(DEFAULT_INIT_METHOD_ATTRIBUTE));
   }
   else if (parentDefaults != null) {
      defaults.setInitMethod(parentDefaults.getInitMethod());
   }

   // 设置 default-destroy-method
   if (root.hasAttribute(DEFAULT_DESTROY_METHOD_ATTRIBUTE)) {
      defaults.setDestroyMethod(root.getAttribute(DEFAULT_DESTROY_METHOD_ATTRIBUTE));
   }
   // 设置摧毁函数
   else if (parentDefaults != null) {
      defaults.setDestroyMethod(parentDefaults.getDestroyMethod());
   }

   // 设置源
   defaults.setSource(this.readerContext.extractSource(root));
}
```

</details>



在这段方法中会设置下列这些字段的默认值

1. lazyInit: 是否懒加载
1. merge
1. autowire: 自动注入方式
1. autowireCandidates: 默认候选beanName
1. initMethod: 实例化方法
1. destroyMethod: 摧毁方法
1. source: 源对象





- 在 `populateDefaults` 了解方法后与之关联的有一个`initDefaults`方法, 它在`populateDefaults`基础上做了什么呢？下面我们来看看`initDefaults`方法



### initDefaults

- 方法分析: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#initDefaults(org.w3c.dom.Element, org.springframework.beans.factory.xml.BeanDefinitionParserDelegate)`

- 方法作用: 设置默认值+触发默认值注册事件





```java
public void initDefaults(Element root, @Nullable BeanDefinitionParserDelegate parent) {
   populateDefaults(this.defaults, (parent != null ? parent.defaults : null), root);
   // 触发默认值注册事件
   this.readerContext.fireDefaultsRegistered(this.defaults);
}
```



- 这里需要关注**注册的对象是什么**，**存储的容器是什么** 

  这里注册的对象是`DefaultsDefinition` (接口)，

  存储容器是`List`

  下面是注册的方法详情

```java
private final List<DefaultsDefinition> defaults = new LinkedList<>();

	@Override
	public void defaultsRegistered(DefaultsDefinition defaultsDefinition) {
		this.defaults.add(defaultsDefinition);
	}
```









下面开始对处理标签，xml 元素的方法进行分析首先先看bean标签的解析.





### parseBeanDefinitionElement 

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element, org.springframework.beans.factory.config.BeanDefinition)`
- 方法作用: 解析 `bean` 标签 转换成 `BeanDefinitionHolder` 对象





<details>
    <summary>parseBeanDefinitionElement 第一部分</summary>





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



</details>



`parseBeanDefinitionElement`第一部分代码处理`id`和`name`两个属性处理逻辑如下

1. 从节点中获取 id 属性和 name 属性
2. 在 name 属性中可能存在多种，使用逗号`,`、分号`;`、空格` `进行分割, 这些name会被拆分放到 `aliases` 集合中, 做别名







下面阅读`parseBeanDefinitionElement`第二部分代码

<details>
    <summary>parseBeanDefinitionElement 第二部分</summary>







```java
// 第二部分
// beanName = id
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

</details>



`parseBeanDefinitionElement`第二部分代码是关于 `beanName` 的操作

**一般情况下 `beanName` 会用 `Id` 进行设置**，当id不存在以及别名列表存在的情况下会将别名列表中的第一个

元素作为 beanName



beanName 的可能性

1. 等于 bean Id 
2. 等于 name 中的第一个



在处理完 beanName 之后还需要对beanName 做验证, 验证的**前置条件是入参BeanDefinition不为空**



下面我们看真正的验证方法`checkNameUniqueness`





### checkNameUniqueness

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#checkNameUniqueness`
- 方法作用: 检查 beanName 是否被使用



<details>
    <summary>checkNameUniqueness 方法详情</summary>







```JAVA
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

</details>

检查的本质是从成员变量`usedNames`中搜索是否存在，检查 beanName 和  别名列表







`parseBeanDefinitionElement`第二部分后半段验证了beanName的可用性



`parseBeanDefinitionElement`第二部分代码小结

1. 确定 beanName
2. 验证 beanName 是否可用





下面进行`parseBeanDefinitionElement`第三部分代码阅读



<details>
    <summary>parseBeanDefinitionElement 第三部分</summary>



```java
// 第三部分
// 解析 bean 定义
AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
```







</details>





第三部分只有一行代码就做一件事: 将 xml 元素处理成 BeanDefinition 对象



### parseBeanDefinitionElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionElement(org.w3c.dom.Element, java.lang.String, org.springframework.beans.factory.config.BeanDefinition)`



<details>
    <summary>parseBeanDefinitionElement 第一部分</summary>





```java
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
```

</details>



第一行 parseState 阶段容器 相关分析查看[这篇文章](/docs/beans/factory/parsing/ProblemReporter/Spring-Problem.md)



`parseBeanDefinitionElement`第一部分处理行为如下

1. 设置阶段
2. 获取className
3. 获取 父属性



下面开始阅读`parseBeanDefinitionElement`第二部分代码



<details>
    <summary>parseBeanDefinitionElement 第二部分</summary>





```java
// 第二部分
try {
   // 创建 bean definition
   AbstractBeanDefinition bd = createBeanDefinition(className, parent);

   // bean definition 属性设置
   parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
   bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
   // 元信息设置
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
catch (ClassNotFoundException ex) {
   error("Bean class [" + className + "] not found", ele, ex);
}
catch (NoClassDefFoundError err) {
   error("Class that bean class [" + className + "] depends on not found", ele, err);
}
catch (Throwable ex) {
   error("Unexpected failure during bean definition parsing", ele, ex);
}
finally {
   this.parseState.pop();
}
```

</details>



第二部分代码围绕 beanDefinition 的各个属性进行处理

1. 设置beanDefinition属性
   1. scope
   2. abstract
   3. lazy-init
   4. autowire-mode
   5. depends-on
   6. autowire-candidate
   7. primary
   8. init-method
   9. destroy-method
   10. factory-method
   11. factory-bean
2. 设置 beanDefinition 的描述
3. 设置元数据
4. 解析 lookup-override
5. 解析 replaced-method
6. 解析 constructor-arg
7. 解析 property 
8. 解析 qualifier
9. 设置资源
10. 设置源







下面我们对这些方法逐一拆分进行分析





### parseBeanDefinitionAttributes

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseBeanDefinitionAttributes`
- 方法作用: 设置beanDefinition属性



<details>
    <summary>parseBeanDefinitionAttributes 方法详情</summary>





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





</details>

在`parseBeanDefinitionAttributes`方法中操作逻辑如下

1. 判断是否存在标签属性

2. 获取标签中对应的属性值

3. 设置属性值(xml中的数据或者默认值)

   









### parseMetaElements

- 方法签名:`org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseMetaElements`
- 方法作用: 解析 meta 标签





<details>
    <summary>parseMetaElements 方法详情</summary>





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





</details>





`parseMetaElements` 处理逻辑 

1. 循环当前节点下的所有节点, 判断是否是 `meta` 标签 构造`BeanMetadataAttribute`对象设置数据









### parseLookupOverrideSubElements 

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseLookupOverrideSubElements`

- 方法作用: 解析 `lookup-override` 标签



<details>
    <summary>parseLookupOverrideSubElements 方法详情</summary>





```java
public void parseLookupOverrideSubElements(Element beanEle, MethodOverrides overrides) {
   // 获取子标签
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      // 是否有 lookup-method 属性
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





</details>



`parseLookupOverrideSubElements` 处理逻辑

1. 循环当前节点下的所有节点判断是否是`lookup-method` 标签, 创建`LookupOverride`对象放入容器









### parseReplacedMethodSubElements

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseReplacedMethodSubElements`
- 方法作用: 解析 `replaced-method` 标签





<details>
    <summary>parseReplacedMethodSubElements 方法详情</summary>







```java
/**
 * Parse replaced-method sub-elements of the given bean element.
 */
public void parseReplacedMethodSubElements(Element beanEle, MethodOverrides overrides) {
   // 子节点获取
   NodeList nl = beanEle.getChildNodes();
   for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      // 是否包含 replaced-method 属性
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

</details>





`parseReplacedMethodSubElements` 处理逻辑

1. 循环当前节点下的所有节点，判断是否是`replaced-method` 标签，如果是 则继续处理 `replaced-method`的下级标签`arg-type`  创建对象 `ReplaceOverride` 放入容器









### parseConstructorArgElements

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseConstructorArgElements`
- 方法作用: 解析 `constructor-arg` 标签





<details>
    <summary>parseConstructorArgElements 方法详情</summary>





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

</details>



👆上面方法依靠`parseConstructorArgElement`方法进行一个完整的处理 重点关注`parseConstructorArgElement`方法





### parseConstructorArgElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseConstructorArgElement`



将方法分为2部分看



第一部分获取资源数据,具体如下

1. index
2. type
3. name



<details>
    <summary>parseConstructorArgElement 第一部分</summary>





```java
// 第一部分
// 获取 index 属性
String indexAttr = ele.getAttribute(INDEX_ATTRIBUTE);
// 获取 type 属性
String typeAttr = ele.getAttribute(TYPE_ATTRIBUTE);
// 获取 name 属性
String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
```



</details>

下面开始第二部分的分析

第二部分是将 xml 元素解析成 `ConstructorArgumentValues`对象，根据是否存在 index 有两种处理方式



<details>
    <summary>parseConstructorArgElement 第二部分</summary>







```java
// 第二部分 1
if (StringUtils.hasLength(indexAttr)) {
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
}
// 第二部分 2
else {
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
}
```

</details>





在上面代码中还涉及到 `property` 标签的处理





### parsePropertyValue 

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parsePropertyValue`
- 方法作用: 解析 property 标签



在处理 `property` 标签又分为下面几种情况

1. 包含 `ref` 属性 

   创建`RuntimeBeanReference ref = new RuntimeBeanReference(refName);` 对象返回

2. 包含 `value` 属性

   创建 `TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));` 返回

3. 处理子标签,

   `parsePropertySubElement`







<details>
    <summary>parsePropertyValue 方法详情</summary>





```java
@Nullable
public Object parsePropertyValue(Element ele, BeanDefinition bd, @Nullable String propertyName) {
   String elementName = (propertyName != null ?
         "<property> element for property '" + propertyName + "'" :
         "<constructor-arg> element");

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

</details>







下面是关于property 的下级标签的解析代码




### parsePropertySubElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parsePropertySubElement(org.w3c.dom.Element, org.springframework.beans.factory.config.BeanDefinition, java.lang.String)`
- 方法作用: 解析Property的下级标签

<details>
    <summary> parsePropertySubElement 方法详情</summary>







```java
@Nullable
public Object parsePropertySubElement(Element ele, @Nullable BeanDefinition bd, @Nullable String defaultValueType) {
   if (!isDefaultNamespace(ele)) {
      // 嵌套分析
      return parseNestedCustomElement(ele, bd);
   }
   else if (nodeNameEquals(ele, BEAN_ELEMENT)) {
      // 解析 bean 标签
      BeanDefinitionHolder nestedBd = parseBeanDefinitionElement(ele, bd);
      if (nestedBd != null) {
         // 装饰 bean define
         nestedBd = decorateBeanDefinitionIfRequired(ele, nestedBd, bd);
      }
      return nestedBd;
   }
   // ref 名称判断
   else if (nodeNameEquals(ele, REF_ELEMENT)) {
      // A generic reference to any name of any bean.
      // 获取 ref 属性
      String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
      boolean toParent = false;
      if (!StringUtils.hasLength(refName)) {
         // A reference to the id of another bean in a parent context.
         // 获取 parent 属性
         refName = ele.getAttribute(PARENT_REF_ATTRIBUTE);
         toParent = true;
         if (!StringUtils.hasLength(refName)) {
            error("'bean' or 'parent' is required for <ref> element", ele);
            return null;
         }
      }
      if (!StringUtils.hasText(refName)) {
         error("<ref> element contains empty target attribute", ele);
         return null;
      }
      // bean 连接对象创建
      RuntimeBeanReference ref = new RuntimeBeanReference(refName, toParent);
      ref.setSource(extractSource(ele));
      return ref;
   }
   else if (nodeNameEquals(ele, IDREF_ELEMENT)) {
      // id-ref 解析
      return parseIdRefElement(ele);
   }
   else if (nodeNameEquals(ele, VALUE_ELEMENT)) {
      // value 解析
      return parseValueElement(ele, defaultValueType);
   }
   else if (nodeNameEquals(ele, NULL_ELEMENT)) {
      // It's a distinguished null value. Let's wrap it in a TypedStringValue
      // object in order to preserve the source location.
      TypedStringValue nullHolder = new TypedStringValue(null);
      nullHolder.setSource(extractSource(ele));
      return nullHolder;
   }
   else if (nodeNameEquals(ele, ARRAY_ELEMENT)) {
      // array 解析
      return parseArrayElement(ele, bd);
   }
   else if (nodeNameEquals(ele, LIST_ELEMENT)) {
      // list 解析
      return parseListElement(ele, bd);
   }
   else if (nodeNameEquals(ele, SET_ELEMENT)) {
      // set 解析
      return parseSetElement(ele, bd);
   }
   else if (nodeNameEquals(ele, MAP_ELEMENT)) {
      // map 解析
      return parseMapElement(ele, bd);
   }
   else if (nodeNameEquals(ele, PROPS_ELEMENT)) {
      // props 解析
      return parsePropsElement(ele);
   }
   else {
      error("Unknown property sub-element: [" + ele.getNodeName() + "]", ele);
      return null;
   }
}
```

</details>







在 property 的下级标签解析中又引出其他的标签解析， 具体如下

1. ref

   ```java
   // ref 名称判断
   else if (nodeNameEquals(ele, REF_ELEMENT)) {
      // A generic reference to any name of any bean.
      // 获取 ref 属性
      String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
      boolean toParent = false;
      if (!StringUtils.hasLength(refName)) {
         // A reference to the id of another bean in a parent context.
         // 获取 parent 属性
         refName = ele.getAttribute(PARENT_REF_ATTRIBUTE);
         toParent = true;
         if (!StringUtils.hasLength(refName)) {
            error("'bean' or 'parent' is required for <ref> element", ele);
            return null;
         }
      }
      if (!StringUtils.hasText(refName)) {
         error("<ref> element contains empty target attribute", ele);
         return null;
      }
      // bean 连接对象创建
      RuntimeBeanReference ref = new RuntimeBeanReference(refName, toParent);
      ref.setSource(extractSource(ele));
      return ref;
   }
   ```

2. id-ref

   处理方法: `parseIdRefElement`

3. value

   处理方法: `parseValueElement`

4. array

   处理方法: `parseArrayElement`

5. list

   处理方法: `parseListElement`

6. set

   处理方法: `parseSetElement`

7. map

   处理方法: `parseMapElement`

8. props

   处理方法: `parsePropsElement`





这类处理方法有很高的一致性. 即 从 xml 中标签中读取指定的属性, 转换成具体的对象 下面就将上面几个处理方法全部贴出来. 这部分代码就请各位自行阅读注释了. 笔者不将其每个都拆分出来进行一个讲解







### parseIdRefElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseIdRefElement`



<details>
    <summary>parseIdRefElement 方法详情</summary>

```java
@Nullable
public Object parseIdRefElement(Element ele) {
   // A generic reference to any name of any bean.
   // 获取 bean 属性
   String refName = ele.getAttribute(BEAN_REF_ATTRIBUTE);
   if (!StringUtils.hasLength(refName)) {
      error("'bean' is required for <idref> element", ele);
      return null;
   }
   if (!StringUtils.hasText(refName)) {
      error("<idref> element contains empty target attribute", ele);
      return null;
   }
   // 设置 bean 链接对象
   RuntimeBeanNameReference ref = new RuntimeBeanNameReference(refName);
   // 设置原
   ref.setSource(extractSource(ele));
   return ref;
}
```

</details>








### parseValueElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseValueElement`



<details>
    <summary>parseValueElement 方法详情</summary>



```java
public Object parseValueElement(Element ele, @Nullable String defaultTypeName) {
   // It's a literal value.
   // 获取 xml 中的文本变量
   String value = DomUtils.getTextValue(ele);
   // 获取 type 属性
   String specifiedTypeName = ele.getAttribute(TYPE_ATTRIBUTE);
   // 类型
   String typeName = specifiedTypeName;
   if (!StringUtils.hasText(typeName)) {
      typeName = defaultTypeName;
   }
   try {
      // 创建类型值
      TypedStringValue typedValue = buildTypedStringValue(value, typeName);
      typedValue.setSource(extractSource(ele));
      typedValue.setSpecifiedTypeName(specifiedTypeName);
      return typedValue;
   }
   catch (ClassNotFoundException ex) {
      error("Type class [" + typeName + "] not found for <value> element", ele, ex);
      return value;
   }
}
```



</details>







### parseArrayElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseArrayElement`



<details>
    <summary>parseArrayElement 方法详情</summary>

```java
public Object parseArrayElement(Element arrayEle, @Nullable BeanDefinition bd) {
   // 获取 value-type 属性
   String elementType = arrayEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
   // 子节点
   NodeList nl = arrayEle.getChildNodes();
   // 合并 array 的类
   ManagedArray target = new ManagedArray(elementType, nl.getLength());
   target.setSource(extractSource(arrayEle));
   target.setElementTypeName(elementType);
   target.setMergeEnabled(parseMergeAttribute(arrayEle));
   // 处理 collection 节点
   parseCollectionElements(nl, target, bd, elementType);
   return target;
}
```



</details>

### parseListElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseListElement`

<details>
    <summary>parseListElement 方法详情</summary>

```java
public List<Object> parseListElement(Element collectionEle, @Nullable BeanDefinition bd) {
   String defaultElementType = collectionEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
   NodeList nl = collectionEle.getChildNodes();
   ManagedList<Object> target = new ManagedList<>(nl.getLength());
   target.setSource(extractSource(collectionEle));
   target.setElementTypeName(defaultElementType);
   target.setMergeEnabled(parseMergeAttribute(collectionEle));
   parseCollectionElements(nl, target, bd, defaultElementType);
   return target;
}
```







### parseSetElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseSetElement`



<details>
    <summary>parseSetElement 方法详情</summary>

```java
public Set<Object> parseSetElement(Element collectionEle, @Nullable BeanDefinition bd) {
   String defaultElementType = collectionEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
   NodeList nl = collectionEle.getChildNodes();
   ManagedSet<Object> target = new ManagedSet<>(nl.getLength());
   target.setSource(extractSource(collectionEle));
   target.setElementTypeName(defaultElementType);
   target.setMergeEnabled(parseMergeAttribute(collectionEle));
   parseCollectionElements(nl, target, bd, defaultElementType);
   return target;
}
```

</details>



### parseMapElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseMapElement`

<details>
    <summary>parseMapElement 方法详情</summary>



```java
public Map<Object, Object> parseMapElement(Element mapEle, @Nullable BeanDefinition bd) {
   // key-type 属性获取
   String defaultKeyType = mapEle.getAttribute(KEY_TYPE_ATTRIBUTE);
   // value-type 属性互殴去
   String defaultValueType = mapEle.getAttribute(VALUE_TYPE_ATTRIBUTE);

   // entry 标签获取
   List<Element> entryEles = DomUtils.getChildElementsByTagName(mapEle, ENTRY_ELEMENT);
   // 合并 map 对象
   ManagedMap<Object, Object> map = new ManagedMap<>(entryEles.size());
   map.setSource(extractSource(mapEle));
   map.setKeyTypeName(defaultKeyType);
   map.setValueTypeName(defaultValueType);
   map.setMergeEnabled(parseMergeAttribute(mapEle));

   // 循环 entry 节点
   for (Element entryEle : entryEles) {
      // Should only have one value child element: ref, value, list, etc.
      // Optionally, there might be a key child element.
      NodeList entrySubNodes = entryEle.getChildNodes();
      Element keyEle = null;
      Element valueEle = null;
      for (int j = 0; j < entrySubNodes.getLength(); j++) {
         Node node = entrySubNodes.item(j);
         if (node instanceof Element) {
            Element candidateEle = (Element) node;
            // 节点名称是否为 key
            if (nodeNameEquals(candidateEle, KEY_ELEMENT)) {
               if (keyEle != null) {
                  error("<entry> element is only allowed to contain one <key> sub-element", entryEle);
               }
               else {
                  keyEle = candidateEle;
               }
            }
            else {
               // Child element is what we're looking for.
               if (nodeNameEquals(candidateEle, DESCRIPTION_ELEMENT)) {
                  // the element is a <description> -> ignore it
               }
               else if (valueEle != null) {
                  error("<entry> element must not contain more than one value sub-element", entryEle);
               }
               else {
                  valueEle = candidateEle;
               }
            }
         }
      }

      // Extract key from attribute or sub-element.
      Object key = null;
      // key 属性
      boolean hasKeyAttribute = entryEle.hasAttribute(KEY_ATTRIBUTE);
      // key-ref 属性
      boolean hasKeyRefAttribute = entryEle.hasAttribute(KEY_REF_ATTRIBUTE);
      if ((hasKeyAttribute && hasKeyRefAttribute) ||
            (hasKeyAttribute || hasKeyRefAttribute) && keyEle != null) {
         error("<entry> element is only allowed to contain either " +
               "a 'key' attribute OR a 'key-ref' attribute OR a <key> sub-element", entryEle);
      }
      if (hasKeyAttribute) {
         // TypedStringValue 构建
         key = buildTypedStringValueForMap(entryEle.getAttribute(KEY_ATTRIBUTE), defaultKeyType, entryEle);
      }
      else if (hasKeyRefAttribute) {
         // key-ref 属性获取
         String refName = entryEle.getAttribute(KEY_REF_ATTRIBUTE);
         if (!StringUtils.hasText(refName)) {
            error("<entry> element contains empty 'key-ref' attribute", entryEle);
         }
         // 创建 bean 连接对象
         RuntimeBeanReference ref = new RuntimeBeanReference(refName);
         ref.setSource(extractSource(entryEle));
         key = ref;
      }
      else if (keyEle != null) {
         // 获取 key 数据
         key = parseKeyElement(keyEle, bd, defaultKeyType);
      }
      else {
         error("<entry> element must specify a key", entryEle);
      }

      // Extract value from attribute or sub-element.
      Object value = null;
      // value 属性是否存在
      boolean hasValueAttribute = entryEle.hasAttribute(VALUE_ATTRIBUTE);
      // value-ref 属性是否存在
      boolean hasValueRefAttribute = entryEle.hasAttribute(VALUE_REF_ATTRIBUTE);
      // 是否存在 value-type 属性
      boolean hasValueTypeAttribute = entryEle.hasAttribute(VALUE_TYPE_ATTRIBUTE);
      if ((hasValueAttribute && hasValueRefAttribute) ||
            (hasValueAttribute || hasValueRefAttribute) && valueEle != null) {
         error("<entry> element is only allowed to contain either " +
               "'value' attribute OR 'value-ref' attribute OR <value> sub-element", entryEle);
      }
      if ((hasValueTypeAttribute && hasValueRefAttribute) ||
            (hasValueTypeAttribute && !hasValueAttribute) ||
            (hasValueTypeAttribute && valueEle != null)) {
         error("<entry> element is only allowed to contain a 'value-type' " +
               "attribute when it has a 'value' attribute", entryEle);
      }
      if (hasValueAttribute) {
         // 获取 value-type 属性
         String valueType = entryEle.getAttribute(VALUE_TYPE_ATTRIBUTE);
         if (!StringUtils.hasText(valueType)) {
            // 设置默认value-type
            valueType = defaultValueType;
         }
         // 创建 TypedStringValue
         value = buildTypedStringValueForMap(entryEle.getAttribute(VALUE_ATTRIBUTE), valueType, entryEle);
      }
      else if (hasValueRefAttribute) {
         // 获取 value-ref 属性
         String refName = entryEle.getAttribute(VALUE_REF_ATTRIBUTE);
         if (!StringUtils.hasText(refName)) {
            error("<entry> element contains empty 'value-ref' attribute", entryEle);
         }
         // 创建 bean 链接对象
         RuntimeBeanReference ref = new RuntimeBeanReference(refName);
         ref.setSource(extractSource(entryEle));
         value = ref;
      }
      else if (valueEle != null) {
         value = parsePropertySubElement(valueEle, bd, defaultValueType);
      }
      else {
         error("<entry> element must specify a value", entryEle);
      }

      // Add final key and value to the Map.
      map.put(key, value);
   }

   return map;
}
```

</details>

### parsePropsElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parsePropsElement`

<details>
    <summary>parsePropsElement 方法详情</summary>



```java
public Properties parsePropsElement(Element propsEle) {
   // 合并对象
   ManagedProperties props = new ManagedProperties();
   props.setSource(extractSource(propsEle));
   props.setMergeEnabled(parseMergeAttribute(propsEle));

   List<Element> propEles = DomUtils.getChildElementsByTagName(propsEle, PROP_ELEMENT);
   for (Element propEle : propEles) {
      String key = propEle.getAttribute(KEY_ATTRIBUTE);
      // Trim the text value to avoid unwanted whitespace
      // caused by typical XML formatting.
      String value = DomUtils.getTextValue(propEle).trim();
      TypedStringValue keyHolder = new TypedStringValue(key);
      keyHolder.setSource(extractSource(propEle));
      TypedStringValue valueHolder = new TypedStringValue(value);
      valueHolder.setSource(extractSource(propEle));
      props.put(keyHolder, valueHolder);
   }

   return props;
}
```

</details>







下面回到`parseBeanDefinitionElement` 方法的第二部分中关于 `qualifier` 标签解析的方法中





### parseQualifierElements

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseQualifierElements`



<details>
    <summary>parseQualifierElements 方法详情</summary>



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

</details>






### parseQualifierElement

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#parseQualifierElement`
- 方法作用: 处理 qualifier 元素





在处理 `qualifier` 标签是和 Java对象有关联的是`AutowireCandidateQualifier`



<details>
    <summary>parseQualifierElement 方法详情</summary>







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



</details>





在`parseBeanDefinitionElement`第二部分代码中一直围绕一开始通过`AbstractBeanDefinition bd = createBeanDefinition(className, parent);`创建的对象，下面我们来了解一下`createBeanDefinition`方法



### createBeanDefinition

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#createBeanDefinition`



<details>
    <summary>createBeanDefinition 方法详情</summary>







```java
	protected AbstractBeanDefinition createBeanDefinition(@Nullable String className, @Nullable String parentName)
			throws ClassNotFoundException {


		return BeanDefinitionReaderUtils.createBeanDefinition(
				parentName, className, this.readerContext.getBeanClassLoader());
	}


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

</details>





这个创建就很简单直接创建`GenericBeanDefinition`设置 className 和 parentName 







到这里我们就将`parseBeanDefinitionElement` 方法分析完成了. 



下面回到 `parseBeanDefinitionElement` 第四部分代码



<details>
    <summary>parseBeanDefinitionElement 第四部分</summary>





```java
// 第四部分
if (beanDefinition != null) {
   if (!StringUtils.hasText(beanName)) {
      try {
         if (containingBean != null) {
            // bean name 生成
            beanName = BeanDefinitionReaderUtils.generateBeanName(
                  beanDefinition, this.readerContext.getRegistry(), true);
         }
         else {
            // beanName 生成
            beanName = this.readerContext.generateBeanName(beanDefinition);
            // Register an alias for the plain bean class name, if still possible,
            // if the generator returned the class name plus a suffix.
            // This is expected for Spring 1.2/2.0 backwards compatibility.
            // 获取 beanClass
            String beanClassName = beanDefinition.getBeanClassName();
            if (beanClassName != null &&
                  beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
                  !this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
               aliases.add(beanClassName);
            }
         }
         if (logger.isTraceEnabled()) {
            logger.trace("Neither XML 'id' nor 'name' specified - " +
                  "using generated bean name [" + beanName + "]");
         }
      }
      catch (Exception ex) {
         error(ex.getMessage(), ele);
         return null;
      }
   }
   // 别名列表
   String[] aliasesArray = StringUtils.toStringArray(aliases);
   // 返回 bean 定义
   return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
}
```

</details>





第四部分对 beanName 做了一个定义

1. 通过下面代码进行生成

   ```java
   BeanDefinitionReaderUtils.generateBeanName(
         beanDefinition, this.readerContext.getRegistry(), true)
   ```

   生成结果

   1. parentName + `$child` +`#` + 16 进制的一个字符串
   2. factoryBeanName + `$created` + `#` + 16 进制的一个字符串
   3. beanName + `# ` +  序号

   <details>
       <summary>generateBeanName 方法详情</summary>

   

   

   ```java
   public static String generateBeanName(
         BeanDefinition definition, BeanDefinitionRegistry registry, boolean isInnerBean)
         throws BeanDefinitionStoreException {
   
      // 获取 bean class 的名称
      // Class.getName()
      String generatedBeanName = definition.getBeanClassName();
      if (generatedBeanName == null) {
         // 父类名称是否存在
         if (definition.getParentName() != null) {
            generatedBeanName = definition.getParentName() + "$child";
         }
         // 工厂 beanName 是否为空
         else if (definition.getFactoryBeanName() != null) {
            generatedBeanName = definition.getFactoryBeanName() + "$created";
         }
      }
      if (!StringUtils.hasText(generatedBeanName)) {
         throw new BeanDefinitionStoreException("Unnamed bean definition specifies neither " +
               "'class' nor 'parent' nor 'factory-bean' - can't generate bean name");
      }
   
      String id = generatedBeanName;
      if (isInnerBean) {
         // Inner bean: generate identity hashcode suffix.
         // 组装名称
         // 生成名称 + # + 16 进制的一个字符串
         id = generatedBeanName + GENERATED_BEAN_NAME_SEPARATOR + ObjectUtils.getIdentityHexString(definition);
      }
      else {
         // Top-level bean: use plain class name with unique suffix if necessary.
         // 唯一beanName设置
         // // beanName + # + 序号
         return uniqueBeanName(generatedBeanName, registry);
      }
      return id;
   }
   ```

   </details>



2. beanName 等价于 类名



- 在得到 beaname 后最后就是一个对象创建返回

```java
// 别名列表
String[] aliasesArray = StringUtils.toStringArray(aliases);
// 返回 bean 定义
return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
```







- 到此关于 beanDefinition 的创建过程分析完成. 

- 下面来看看另一组方法 装饰 beanDefinition





### decorateBeanDefinitionIfRequired

- 方法签名: `org.springframework.beans.factory.xml.BeanDefinitionParserDelegate#decorateBeanDefinitionIfRequired(org.w3c.dom.Element, org.springframework.beans.factory.config.BeanDefinitionHolder, org.springframework.beans.factory.config.BeanDefinition)`





<details>
    <summary>decorateBeanDefinitionIfRequired 方法详情</summary>





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

</details>

- 装饰依靠 `namespaceHandler` 接口的 `decorate`方法进行装饰，有关 namespaceHandler 的分析请查看[这篇文章](/docs/beans/factory/xml/NamespaceHandler/readme.md)