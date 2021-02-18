# 第二十八章 ConfigurationClassBeanDefinitionReader
本章笔者将和各位一起探讨 Spring 是如何读取被 `@Configuration` 注解标标记对象中的 Bean 数据. 





## 28.1 测试环境搭建

我们在这里需要搭建的测试环境很简单，只需要做一个 Spring 注解环境下的一个简单应用即可。

- 定义Bean

```java
public class AnnPeople {
   private String name;

   public String getName() {
      return name;
   }

   public void setName(String name) {
      this.name = name;
   }
}
```

- 定义注解模式下的上下文

```java
@Component
@Scope
public class AnnBeans {
   @Bean(name = "abc")
   public AnnPeople annPeople() {
      AnnPeople annPeople = new AnnPeople();
      annPeople.setName("people");
      return annPeople;
   }
}
```

- 编写测试用例

```java
@Test
void testComponentClasses(){
   AnnotationConfigApplicationContext context =
         new AnnotationConfigApplicationContext(AnnBeans.class);
   AnnPeople bean = context.getBean(AnnPeople.class);
   assert bean.getName().equals("people");
}
```







这一次我们需要分析的目标对象我们是在分析 `ConfigurationClassPostProcessor` 类的时候遇到的一个对象，当时笔者告诉各位说 `ConfigurationClassBeanDefinitionReader` 可以读取 Spring Configuration Bean 中的 Bean 信息将其注册到 Spring 容器中，下面我们就要来解开这个处理流程。

首先我们来回顾我们需要分析的入口

- `processConfigBeanDefinitions` 中的入口

```java
// Read the model and create bean definitions based on its content
if (this.reader == null) {
   this.reader = new ConfigurationClassBeanDefinitionReader(
         registry, this.sourceExtractor, this.resourceLoader, this.environment,
         this.importBeanNameGenerator, parser.getImportRegistry());
}
// 解析 Spring Configuration Class 主要目的是提取其中的 Spring 注解并将其转换成 Bean Definition
this.reader.loadBeanDefinitions(configClasses);
```





## 28.2 构造函数

在调用 `loadBeanDefinitions` 方法前我们先来看  `reader` 的构造函数，在构造函数中的对象我们或多或少熟悉一些下面我们来看具体说明。

- `ConfigurationClassBeanDefinitionReader` 构造函数参数列表说明



| 参数名称 | 参数类型 | 参数说明 |
| -------- | -------- | -------- |
|   `registry`     |   `BeanDefinitionRegistry`     | Bean Definition 注册器 |
|    `sourceExtractor`   |    `SourceExtractor`    | 元数据解析器 |
|    `resourceLoader`   |    `ResourceLoader`    | 资源读取器 |
|    `environment`   |    `Environment`    | 配置信息接口 |
|    `importBeanNameGenerator`   |  `BeanNameGenerator`     | Bean Name 生成接口 |
|    `importRegistry`   |     `ImportRegistry`   | import 注册器 |



## 28.3 `loadBeanDefinitions` 分析

了解完成构造函数后我们来看具体的处理方法 `loadBeanDefinitions`，先来看处理代码

- `loadBeanDefinitions` 方法详情

```java
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
   TrackedConditionEvaluator trackedConditionEvaluator = new TrackedConditionEvaluator();
   for (ConfigurationClass configClass : configurationModel) {
      loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
   }
}
```



在这里我们看到了一个新的对象 `TrackedConditionEvaluator`， 先来对这个对象做分析吧。





## 28.4 `TrackedConditionEvaluator` 分析

在 `TrackedConditionEvaluator` 有一个Map 结构存储了关于配置类的一些数据：

key：存储配置类

value：Condition 执行结果

- 成员变量

```java
private final Map<ConfigurationClass, Boolean> skipped = new HashMap<>();
```

- 处理单个配置类的数据信息方法

```java
public boolean shouldSkip(ConfigurationClass configClass) {
   Boolean skip = this.skipped.get(configClass);
   if (skip == null) {
      if (configClass.isImported()) {
         boolean allSkipped = true;
         for (ConfigurationClass importedBy : configClass.getImportedBy()) {
            if (!shouldSkip(importedBy)) {
               allSkipped = false;
               break;
            }
         }
         if (allSkipped) {
            // The config classes that imported this one were all skipped, therefore we are skipped...
            skip = true;
         }
      }
      if (skip == null) {
         skip = conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN);
      }
      this.skipped.put(configClass, skip);
   }
   return skip;
}
```

在这个方法中我们需要使用条件注解相关的知识点，具体分析各位可以阅读第二十四章。







## 28.5 `loadBeanDefinitionsForConfigurationClass` 方法分析

下面我们先来看 `loadBeanDefinitionsForConfigurationClass` 代码

- `loadBeanDefinitionsForConfigurationClass` 方法详情

```java
private void loadBeanDefinitionsForConfigurationClass(
      ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {

   if (trackedConditionEvaluator.shouldSkip(configClass)) {
      // Bean Name 提取
      String beanName = configClass.getBeanName();
      // Bean Name 是否存在
      // 注册容器中是否存在当前处理的 Bean Name
      if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
         // 注册容器中移除 Bean Name 对应的数据
         this.registry.removeBeanDefinition(beanName);
      }
      // import 注册器移除当前的
      this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
      return;
   }

   // 判断当前处理的配置类是否是 import 的
   if (configClass.isImported()) {
      registerBeanDefinitionForImportedConfigurationClass(configClass);
   }
   // 配置类中获取 Bean Method 列表
   for (BeanMethod beanMethod : configClass.getBeanMethods()) {
      // 处理单个 Bean Method 
      loadBeanDefinitionsForBeanMethod(beanMethod);
   }

   // 处理 ImportedResoruce 数据
   loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
   // 处理 ImportBeanDefinitionRegistrar 
   loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
}
```

在这个方法中主要处理流程如下

1. 对当前需要处理的配置类进行条件注解验证，如果通过验证则进行下面处理

   1. 提取配置类的 Bean Name 。

   2. Bean Name 为空 并且当前容器中存在 Bean Name 。

      如果符合这个条件此时需要将容器中 Bean Name 对应的数据删除。

   3. 在 `importRegistry` 中移除当前类名

   

2. 判断当前配置类是否是通过 `Import` 导入的对象

   如果是则进入 `registerBeanDefinitionForImportedConfigurationClass` 进行处理

3. 获取配置类中的 `BeanMethod` 列表，让每个 `BeanMethod` 都经过 `loadBeanDefinitionsForBeanMethod` 方法处理

4. 处理 `ImportedResoruce`

5. 处理 `ImportBeanDefinitionRegistrar`



在这里出现的多个方法调用笔者将在本章后续一个个进行分析，请各位保持耐心。





## 28.6 `loadBeanDefinitionsForBeanMethod` 分析

现在我们来分析 `BeanMethod` 处理，首先我们需要了解 `BeanMethod` 是什么。要了解 `BeanMethod` 必然跑不开类图的阅读

- `BeanMethod` 类图

![BeanMethod](//book/ch-28/images/BeanMethod.png)



在这里我们可以看到 BeanMethod 继承了 ConfigurationMethod 类，在源代码中 `BeanMethod` 没有提供成员变量，其成员变量都是由 `ConfigurationMethod` 提供，下面我们来看 `ConfigurationMethod` 中的成员变量



| 变量名称             | 变量类型             | 变量说明   |
| -------------------- | -------------------- | ---------- |
| `metadata`           | `MethodMetadata`     | 方法元数据 |
| `configurationClass` | `ConfigurationClass` | 配置类对象 |



在我们的测试用例中配置类是 `com.source.hot.ioc.book.ann.AnnBeans` 我们来看它的 `BeanMethod` 列表存在什么数据。

- `BeanMethod` 信息

![image-20210208112334225](/docs/ch-28/images/image-20210208112334225.png)

现在我们可以通过这个截图了解到其中的数据信息也可以大概猜测出一些数据的含义，各位可能回对数据是如何产生的有疑问，这部分内容会在二十九章中和各位一起探讨。









## 28.9 `loadBeanDefinitionsForBeanMethod` 方法分析

看到了这些数据以后我们就可以进入 `loadBeanDefinitionsForBeanMethod` 方法来看看这些数据是如何转换成 Bean Definition 。下面先请各位阅读源码

- `loadBeanDefinitionsForBeanMethod` 方法详情

```java
@SuppressWarnings("deprecation")  // for RequiredAnnotationBeanPostProcessor.SKIP_REQUIRED_CHECK_ATTRIBUTE
private void loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod) {

   // 提取配置类
   ConfigurationClass configClass = beanMethod.getConfigurationClass();
   // 提取方法元数据
   MethodMetadata metadata = beanMethod.getMetadata();
   // 提取方法名称
   String methodName = metadata.getMethodName();

   // 进行 Condition 注解处理
   // Do we need to mark the bean as skipped by its condition?
   if (this.conditionEvaluator.shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN)) {
      configClass.skippedBeanMethods.add(methodName);
      return;
   }
   if (configClass.skippedBeanMethods.contains(methodName)) {
      return;
   }

   // 从注解元数据中提取 Bean 注解相关的注解属性对象
   AnnotationAttributes bean = AnnotationConfigUtils.attributesFor(metadata, Bean.class);
   Assert.state(bean != null, "No @Bean annotation attributes");

   // 提取 Bean 属性中的 name 值
   // Consider name and any aliases
   List<String> names = new ArrayList<>(Arrays.asList(bean.getStringArray("name")));
   String beanName = (!names.isEmpty() ? names.remove(0) : methodName);

   // 别名处理
   // 真名=方法名称
   // 别名等于 @Bean 中的name属性
   // Register aliases even when overridden
   for (String alias : names) {
      this.registry.registerAlias(beanName, alias);
   }

   // 检查是否存在覆盖的定义
   // Has this effectively been overridden before (e.g. via XML)?
   if (isOverriddenByExistingDefinition(beanMethod, beanName)) {
      if (beanName.equals(beanMethod.getConfigurationClass().getBeanName())) {
         throw new BeanDefinitionStoreException(beanMethod.getConfigurationClass().getResource().getDescription(),
               beanName, "Bean name derived from @Bean method '" + beanMethod.getMetadata().getMethodName() +
               "' clashes with bean name for containing configuration class; please make those names unique!");
      }
      return;
   }

   // Bean Definition 的创建和属性设置
   ConfigurationClassBeanDefinition beanDef = new ConfigurationClassBeanDefinition(configClass, metadata);
   beanDef.setResource(configClass.getResource());
   beanDef.setSource(this.sourceExtractor.extractSource(metadata, configClass.getResource()));

   if (metadata.isStatic()) {
      // static @Bean method
      if (configClass.getMetadata() instanceof StandardAnnotationMetadata) {
         beanDef.setBeanClass(((StandardAnnotationMetadata) configClass.getMetadata()).getIntrospectedClass());
      }
      else {
         beanDef.setBeanClassName(configClass.getMetadata().getClassName());
      }
      beanDef.setUniqueFactoryMethodName(methodName);
   }
   else {
      // instance @Bean method
      beanDef.setFactoryBeanName(configClass.getBeanName());
      beanDef.setUniqueFactoryMethodName(methodName);
   }

   if (metadata instanceof StandardMethodMetadata) {
      beanDef.setResolvedFactoryMethod(((StandardMethodMetadata) metadata).getIntrospectedMethod());
   }

   beanDef.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR);
   beanDef.setAttribute(org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor.
         SKIP_REQUIRED_CHECK_ATTRIBUTE, Boolean.TRUE);

   // Lazy Primary DependsOn Role Description 注解数据设置
   AnnotationConfigUtils.processCommonDefinitionAnnotations(beanDef, metadata);

   Autowire autowire = bean.getEnum("autowire");
   if (autowire.isAutowire()) {
      beanDef.setAutowireMode(autowire.value());
   }

   boolean autowireCandidate = bean.getBoolean("autowireCandidate");
   if (!autowireCandidate) {
      beanDef.setAutowireCandidate(false);
   }

   String initMethodName = bean.getString("initMethod");
   if (StringUtils.hasText(initMethodName)) {
      beanDef.setInitMethodName(initMethodName);
   }

   String destroyMethodName = bean.getString("destroyMethod");
   beanDef.setDestroyMethodName(destroyMethodName);

   // scope 处理
   // Consider scoping
   ScopedProxyMode proxyMode = ScopedProxyMode.NO;
   AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(metadata, Scope.class);
   if (attributes != null) {
      beanDef.setScope(attributes.getString("value"));
      proxyMode = attributes.getEnum("proxyMode");
      if (proxyMode == ScopedProxyMode.DEFAULT) {
         proxyMode = ScopedProxyMode.NO;
      }
   }

   // Replace the original bean definition with the target one, if necessary
   BeanDefinition beanDefToRegister = beanDef;
   if (proxyMode != ScopedProxyMode.NO) {
      BeanDefinitionHolder proxyDef = ScopedProxyCreator.createScopedProxy(
            new BeanDefinitionHolder(beanDef, beanName), this.registry,
            proxyMode == ScopedProxyMode.TARGET_CLASS);
      beanDefToRegister = new ConfigurationClassBeanDefinition(
            (RootBeanDefinition) proxyDef.getBeanDefinition(), configClass, metadata);
   }

   if (logger.isTraceEnabled()) {
      logger.trace(String.format("Registering bean definition for @Bean method %s.%s()",
            configClass.getMetadata().getClassName(), beanName));
   }
   // 注册 Bean Definition
   this.registry.registerBeanDefinition(beanName, beanDefToRegister);
}
```



在这个方法中整体处理过程就是从 BeanMethod 对象中提取各种属性然后将其赋值给 Bean Definition 对象，在这用到的具体对象是 `ConfigurationClassBeanDefinition`。接下来我们对整个方法的处理过程整理出来。

1. **从 Bean Method 中提取配置。**
2. **从 Bean Method提取方法元数据。**
3. **从方法元数据中提取方法名称。**
4. **方法元数据中进行条件注解处理。**
5. **配置类中进行条件注解处理。**
6. **从方法元数据中提取 Bean 注解的数据。**
7. **注解属性中 `name` 属性列表。**
8. **确定 Bean Name，如果注解属性中的 `name` 列表不为空则取第一个作为 Bean Name ，否则将使用方法名称作为 Bean Name。**
9. **别名注册，真名是上一步得到的 Bean Name ，别名是 `name` 属性列表中的其他名称。**
10. **检查是否存在覆盖的 Bean Definition。如果存在就会抛出异常 `BeanDefinitionStoreException`。**
11. **创建 Bean Definition ，实际对象是 `ConfigurationClassBeanDefinition`。**
12. **解析和设置 Bean Definition 中的属性。**
13. **Bean Definition 的注册。**

这十三步就是 `loadBeanDefinitionsForBeanMethod` 方法的各项处理的总结，具体想要了解每一个的处理过程各位可以在重新阅读一下源码。









## 28.10 `registerBeanDefinitionForImportedConfigurationClass` 方法分析

接下来笔者将和各位一起来分析 `registerBeanDefinitionForImportedConfigurationClass` 方法的处理过程，在此之前我们需要先来编写测试用例



### 28.10.1 `ImportedConfigurationClass` 测试用例编写

在编写用力之前我们需要知道什么样的类会进入这个方法，回顾方法处理的前后代码

```java
if (configClass.isImported()) {
   // 导入
   registerBeanDefinitionForImportedConfigurationClass(configClass);
}
```

这里我们可以看到一个导入的 Bean 就会进入该方法进行处理，下面我们就来编写一个符合这个条件的测试用例。

首先我们来编写一个 Spring 配置类

```java
public class ConfigurationA {
   @Bean(name = "ConfigurationA.annPeople")
   public AnnPeople annPeople() {
      return new AnnPeople();
   }
}
```

其次我们来编写测试用例的配置上下文类

```java
@Configuration
@Import(ConfigurationA.class)
public class ImportedBeans {
}
```

最后我们来编写测试方法

```java
@Test
void testAnnImportConfiguration() {
   AnnotationConfigApplicationContext context
         = new AnnotationConfigApplicationContext(ImportedBeans.class);
   AnnPeople bean = context.getBean("ConfigurationA.annPeople", AnnPeople.class);
   assert bean != null;
}
```



各位读者需要调试源代码的话可以将断点放在 `registerBeanDefinitionForImportedConfigurationClass` 方法上就可以看到 `configClass` 中会表示为 `com.source.hot.ioc.book.ann.imported.ConfigurationA` 类



![image-20210208142936127](/docs/ch-28/images/image-20210208142936127.png)





现在我们拥有了测试用例并且成功的进入了调试，下面我们来看看这个方法中的处理过程吧。

- `registerBeanDefinitionForImportedConfigurationClass` 方法详情

```java
private void registerBeanDefinitionForImportedConfigurationClass(ConfigurationClass configClass) {
   // 提取注解元数据
   AnnotationMetadata metadata = configClass.getMetadata();
   // 将注解元数据转换成 Bean Definition
   AnnotatedGenericBeanDefinition configBeanDef = new AnnotatedGenericBeanDefinition(metadata);

   // 解析 scope 元数据
   ScopeMetadata scopeMetadata = scopeMetadataResolver.resolveScopeMetadata(configBeanDef);
   configBeanDef.setScope(scopeMetadata.getScopeName());
   // 处理 Bean Name
   String configBeanName = this.importBeanNameGenerator.generateBeanName(configBeanDef, this.registry);
   // 处理常规注解
   AnnotationConfigUtils.processCommonDefinitionAnnotations(configBeanDef, metadata);

   // Bean Definition 转换成 Bean Definition Holder
   BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(configBeanDef, configBeanName);
   // 应用 scope 代理
   definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
   // 注册 Bean Definition
   this.registry.registerBeanDefinition(definitionHolder.getBeanName(), definitionHolder.getBeanDefinition());
   configClass.setBeanName(configBeanName);

   if (logger.isTraceEnabled()) {
      logger.trace("Registered bean definition for imported class '" + configBeanName + "'");
   }
}
```

阅读方法后我们来整理这个方法的处理过程

1. **从配置类中提取注解元数据。**
2. **创建配置类对应的 Bean Definition 对象 ，具体类型是 `AnnotatedGenericBeanDefinition` 。**
3. **解析 Scope 元数据并且设置 scope 属性。**
4. **对当前配置类进行 Bean Name 的初始化。**
5. **处理常规注解 `Lazy` 、`Primary`、`DependsOn` 、`Role` 和 `Description` 并将其数据赋值给 Bean Definition。**
6. **将 Bean Definition 转换为 Bean Definition Holder 对象。**
7. **处理 scope proxy 相关内容。**
8. **Bean Definition 注册。**

处理完成这八个操作后对于 `registerBeanDefinitionForImportedConfigurationClass` 方法的分析也就告一段落了。







## 28.11 `loadBeanDefinitionsFromImportedResources` 方法分析

接下来笔者将和各位一起来分析 `loadBeanDefinitionsFromImportedResources` 方法的处理过程，在此之前我们需要先来编写测试用例。



### 28.11.1 测试用例编写

我们先来看处理方法的调用代码 `loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());`

在这段代码中我们可以知道它是从配置类中提取了 `importedResources` 属性进行解析，那么我们需要往上找一找这个属性是从哪里来的，在这个类中搜索可以找到 `addImportedResource` ，这个方法就是往 `importedResources` 容器中插入数据的，继续往上我们来看是谁调用了这个方法，经过搜索我们可以找到 `ConfigurationClassParser#doProcessConfigurationClass` 中存在相关操作，具体代码如下

- `ImportResource` 注解数据初始化

```java
AnnotationAttributes importResource =
      AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
if (importResource != null) {
   String[] resources = importResource.getStringArray("locations");
   Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
   for (String resource : resources) {
      String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
      configClass.addImportedResource(resolvedResource, readerClass);
   }
}
```

OK 现在我们找到了我们需要编写测试用应该使用的一个注解 `@ImportResource`，下面我们就可以开始编写测试用例了

```java
@ImportResource(locations = {
		"classpath:/META-INF/first-ioc.xml"
})
@Configuration
public class ImportResourceBeansTest {
	@Test
	void testImportResource(){
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ImportResourceBeansTest.class);
		PeopleBean bean = context.getBean(PeopleBean.class);
		assert bean != null;
	}
}

```



- `META-INF/first-ioc.xml` 文件信息

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



测试用例准备完毕，下面我们先来看 `importedResources` 容器中的数据信息

![image-20210208150615297](/docs/ch-28/images/image-20210208150615297.png)



在这个对象中相信各位可以看出 `value` 是一个 Bean Definition Reader ，它可以用来读取 `key` 当中指向的一个路径，即 `value` 可以将 `key` 所指向文件中的 Spring Bean 解析出来。

下面我们来看 `loadBeanDefinitionsFromImportedResources` 方法

- `loadBeanDefinitionsFromImportedResources` 方法详情

```java
private void loadBeanDefinitionsFromImportedResources(
      Map<String, Class<? extends BeanDefinitionReader>> importedResources) {

   Map<Class<?>, BeanDefinitionReader> readerInstanceCache = new HashMap<>();

   // 循环处理 importedResources
   importedResources.forEach((resource, readerClass) -> {
      // Default reader selection necessary?
      // 判断 Bean Defintion Reader 是否是 BeanDefinitionReader
      if (BeanDefinitionReader.class == readerClass) {
         // 判断文件地址是否是 .groovy 结尾
         if (StringUtils.endsWithIgnoreCase(resource, ".groovy")) {
            // When clearly asking for Groovy, that's what they'll get...
            readerClass = GroovyBeanDefinitionReader.class;
         }
         // 其他情况都当作 xml 处理
         else {
            // Primarily ".xml" files but for any other extension as well
            readerClass = XmlBeanDefinitionReader.class;
         }
      }

      // 尝试从缓存中获取 BeanDefinitionReader
      BeanDefinitionReader reader = readerInstanceCache.get(readerClass);
      // 如果不存在则创建一个新的 BeanDefinitionReader
      if (reader == null) {
         try {
            // Instantiate the specified BeanDefinitionReader
            reader = readerClass.getConstructor(BeanDefinitionRegistry.class).newInstance(this.registry);
            // Delegate the current ResourceLoader to it if possible
            if (reader instanceof AbstractBeanDefinitionReader) {
               AbstractBeanDefinitionReader abdr = ((AbstractBeanDefinitionReader) reader);
               abdr.setResourceLoader(this.resourceLoader);
               abdr.setEnvironment(this.environment);
            }
            readerInstanceCache.put(readerClass, reader);
         }
         catch (Throwable ex) {
            throw new IllegalStateException(
                  "Could not instantiate BeanDefinitionReader class [" + readerClass.getName() + "]");
         }
      }

      // TODO SPR-6310: qualify relative path locations as done in AbstractContextLoader.modifyLocations
      // Bean Defintion Reader 进行解析
      reader.loadBeanDefinitions(resource);
   });

}
```

下面我们来看这个方法的处理过程。

1. **创建一个用来存储 `Class` 和 `BeanDefinitionReader` 关系的容器。**

   **key：Bean Definition Reader 类型**

   **value：Bean Definition Reader 实例**

2. **处理 `importedResources`**

   1. **判断读取器是否是 `BeanDefinitionReader` 类型**

      1. **如果是 `BeanDefinitionReader` 类型进一步处理**

         **资源文件是以 `.groovy` 结尾将读取器类型定义成 `GroovyBeanDefinitionReader`**

         **其他情况都将读取器类型定义为 `XmlBeanDefinitionReader`**

   2. **从容器中获取当前读取器类型对应的读取器实例，如果不存在就创建一个实例（创建方式是反射创建），并将这个实例放入到绑定关系中。**

   3. **解析 `resource` ，这里的解析就是对 Spring xml 模式下配置文件的解析**。具体解析过程笔者在第六章的时候有相关介绍，如果忘记可以向前翻阅。





## 28.12 `loadBeanDefinitionsFromRegistrars` 方法分析

接下来笔者将和各位一起来分析 `loadBeanDefinitionsFromRegistrars` 方法的处理过程，在此之前我们需要先来编写测试用例。

### 28.12.1 测试用例编写

我们先来看处理方法的调用代码 `loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());`

在这段代码中我们可以知道它是从配置类中提取了 `importBeanDefinitionRegistrars` 属性进行解析，那么我们需要往上找一找这个属性是从哪里来的，在这个类中搜索可以找到 `addImportBeanDefinitionRegistrar` ，这个方法就是往 `importBeanDefinitionRegistrars` 容器中插入数据的，继续往上我们来看是谁调用了这个方法，经过搜索我们可以找到 `ConfigurationClassParser#processImports` 中存在相关操作，具体代码如下

- `ConfigurationClassParser#processImports` 中关于`addImportBeanDefinitionRegistrar` 方法的调度

```java
else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
   // Candidate class is an ImportBeanDefinitionRegistrar ->
   // delegate to it to register additional bean definitions
   Class<?> candidateClass = candidate.loadClass();
   ImportBeanDefinitionRegistrar registrar =
         ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
               this.environment, this.resourceLoader, this.registry);
   configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
}
```

通过这样的寻找我们可以确定我们需要实现的接口是 `ImportBeanDefinitionRegistrar` ，同时在前文第二十七章中介绍过如何导入一个Bean，现在我们需要将这些知识整合起来做出一个测试用例来测试 `loadBeanDefinitionsFromRegistrars`  这段代码



首先编写 `ImportBeanDefinitionRegistrar` 的实现类

- `MyBeanRegistrar` 代码详情

```java
public class MyBeanRegistrar implements ImportBeanDefinitionRegistrar {

   @Override
   public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
         BeanDefinitionRegistry registry) {
      GenericBeanDefinition gbd = new GenericBeanDefinition();
      gbd.setBeanClass(AnnPeople.class);
      gbd.getPropertyValues().addPropertyValue("name", "name");
      registry.registerBeanDefinition("abn", gbd);
   }
}
```



- Spring 注解模式下的配置类

```java
@Configuration
@Import(MyBeanRegistrar.class)
public class ImportBeanDefinitionRegistrarBeans {
}
```



最后来编写测试方法

```java
@Test
void testImportBeanDefinitionRegistrar() {
   AnnotationConfigApplicationContext context
         = new AnnotationConfigApplicationContext(ImportBeanDefinitionRegistrarBeans.class);

   AnnPeople bean = context.getBean("abn", AnnPeople.class);
   assert bean != null;
}
```



测试用例准备完毕，下面我们来看调试及源码分析，先来看当前容器中  `importBeanDefinitionRegistrars` 的数据。

- `importBeanDefinitionRegistrars` 数据信息

![image-20210208163141104](/docs/ch-28/images/image-20210208163141104.png)



下面我们进入方法来看其中做了什么

- `loadBeanDefinitionsFromRegistrars` 方法详情

```java
private void loadBeanDefinitionsFromRegistrars(Map<ImportBeanDefinitionRegistrar, AnnotationMetadata> registrars) {
   registrars.forEach((registrar, metadata) ->
         registrar.registerBeanDefinitions(metadata, this.registry, this.importBeanNameGenerator));
}
```

这里我们可以看到操作十分简单，循环执行每个 `ImportBeanDefinitionRegistrar` 中的 `registerBeanDefinitions` 方法。









## 28.13 总结

本章笔者和各位一起探讨了 Spring 中对配置类的解析，我们通过源码了解了四种处理

1. **第一种：`@Import` 中 `value` 填写的是一个 Spring Configuration Bean 对其进行解析，处理方法 `registerBeanDefinitionForImportedConfigurationClass`。**
2. **第二种：当前配置类中 Bean Method 的解析 ，处理方法 `loadBeanDefinitionsForBeanMethod`。**
3. **第三种：处理 `@ImportedResoruce` 注解相关内容，该处理操作是读取 XML 或者 groovy 类型的 Spring 配置文件，将其中定义的 Bean 注册到 Spring 中，处理方法：`loadBeanDefinitionsFromImportedResources`**
4. **第四种：`@Import` 中 `value` 天喜的是一个 `ImportBeanDefinitionRegistrar` 实现类，处理方法 `loadBeanDefinitionsFromRegistrars` 。**



