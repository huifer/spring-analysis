# 第二十三章 ConfigurationClassPostProcessor

在这一章节中笔者将和各位一起讨论 `ConfigurationClassPostProcessor` 。

## 23.1 初识 `ConfigurationClassPostProcessor`

首先我们先来看 `ConfigurationClassPostProcessor` 类图

![ConfigurationClassPostProcessor](//book/ch-23/images/ConfigurationClassPostProcessor.png)

在这个类图中我们重点关注的是 `BeanDefinitionRegistryPostProcessor` 和 `BeanFactoryPostProcessor` ，我们先来简单理解这两个接口的作用。

1. `BeanFactoryPostProcessor` 作用：定制 Bean Factory 。
2. `BeanDefinitionRegistryPostProcessor` 作用：注册 Bean Definition 。

简单了解过这两个接口的作用后，我们先来看 `BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry` 中的代码并对其进行分析。



## 23.2 测试用例搭建

下面我们来搭建一个测试用例，该测试用例和注解环境的测试用例相同。

- 创建一个 Bean 对象

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

- 创建一个 Component Bean

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

   class InnerClass {

   }
}
```

- 使用这些Bean

```java
public class AnnotationContextTest {
   @Test
   void testBasePackages(){
      AnnotationConfigApplicationContext context =
            new AnnotationConfigApplicationContext("com.source.hot.ioc.book.ann");
      AnnPeople bean = context.getBean(AnnPeople.class);
      assert bean.getName().equals("people");
   }
}
```



测试用例准备就绪下面我们来看看具体 `ConfigurationClassPostProcessor` 中的一些处理





## 23.3 `postProcessBeanDefinitionRegistry` 分析

我们先来找到 `postProcessBeanDefinitionRegistry` 代码进行阅读

```java
@Override
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
   int registryId = System.identityHashCode(registry);
   if (this.registriesPostProcessed.contains(registryId)) {
      throw new IllegalStateException(
            "postProcessBeanDefinitionRegistry already called on this post-processor against " + registry);
   }
   if (this.factoriesPostProcessed.contains(registryId)) {
      throw new IllegalStateException(
            "postProcessBeanFactory already called on this post-processor against " + registry);
   }
   this.registriesPostProcessed.add(registryId);

   // 注册 Configuration 相关注解的bean定义
   processConfigBeanDefinitions(registry);
}
```

在这段代码中我们需要重点关注的方法是 `processConfigBeanDefinitions`，在这个方法中有很多细节可以讨论，不过我们对 Bean Definition 的新建会作为一个重点分析目标。下面我们来看代码。



```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
   // spring configuration bean definition holder
   List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
   // 容器中已存在的 Bean Name
   String[] candidateNames = registry.getBeanDefinitionNames();

   for (String beanName : candidateNames) {
      BeanDefinition beanDef = registry.getBeanDefinition(beanName);
      // bean definition 中获取属性 CONFIGURATION_CLASS_ATTRIBUTE 是否为空
      if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
         if (logger.isDebugEnabled()) {
            logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
         }
      }
      // 判断当前的 bean Definition 是否属于候选对象
      else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
         configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
      }
   }

   // 如果存有 @Configuration 的对象为空就返回，注意此时@Compoment 类也算 @Configuration 对已处理方法是 checkConfigurationClassCandidate
   // Return immediately if no @Configuration classes were found
   if (configCandidates.isEmpty()) {
      return;
   }

   // 将找到的 Configuration Bean 进行排序
   // Sort by previously determined @Order value, if applicable
   configCandidates.sort((bd1, bd2) -> {
      int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
      int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
      return Integer.compare(i1, i2);
   });

   // 创建两个 Bean Name 生成器
   // 1. 组件扫描的 Bean Name 生成器
   // 2. 导入的 Bean Name 生成器
   // Detect any custom bean name generation strategy supplied through the enclosing application context
   SingletonBeanRegistry sbr = null;
   if (registry instanceof SingletonBeanRegistry) {
      sbr = (SingletonBeanRegistry) registry;
      if (!this.localBeanNameGeneratorSet) {
         BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
               AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
         if (generator != null) {
            this.componentScanBeanNameGenerator = generator;
            this.importBeanNameGenerator = generator;
         }
      }
   }

   // 环境信息
   if (this.environment == null) {
      this.environment = new StandardEnvironment();
   }

   // 解析阶段
   // 创建 Configuration 解析对象
   // Parse each @Configuration class
   ConfigurationClassParser parser = new ConfigurationClassParser(
         this.metadataReaderFactory, this.problemReporter, this.environment,
         this.resourceLoader, this.componentScanBeanNameGenerator, registry);

   // 候选的需要解析的 Bean Definition 容器
   Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
   // 已经完成解析得 Spring Configuration Bean
   Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
   do {
      // 解析候选 Bean Definition Holder 集合
      parser.parse(candidates);
      // 解析器中的验证
      parser.validate();

      // 配置类
      Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
      configClasses.removeAll(alreadyParsed);

      // Read the model and create bean definitions based on its content
      if (this.reader == null) {
         this.reader = new ConfigurationClassBeanDefinitionReader(
               registry, this.sourceExtractor, this.resourceLoader, this.environment,
               this.importBeanNameGenerator, parser.getImportRegistry());
      }
      // 解析 Spring Configuration Class 主要目的是提取其中的 Spring 注解并将其转换成 Bean Definition
      this.reader.loadBeanDefinitions(configClasses);
      alreadyParsed.addAll(configClasses);

      candidates.clear();
      if (registry.getBeanDefinitionCount() > candidateNames.length) {
         // 新的候选名称
         String[] newCandidateNames = registry.getBeanDefinitionNames();
         // 历史的候选名称
         Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
         // 已经完成解析的类名
         Set<String> alreadyParsedClasses = new HashSet<>();
         // 加入解析类名
         for (ConfigurationClass configurationClass : alreadyParsed) {
            alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
         }
         // 新的候选名称和对应的 Bean Definition 添加到候选容器中
         for (String candidateName : newCandidateNames) {
            if (!oldCandidateNames.contains(candidateName)) {
               BeanDefinition bd = registry.getBeanDefinition(candidateName);
               if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                     !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                  candidates.add(new BeanDefinitionHolder(bd, candidateName));
               }
            }
         }
         candidateNames = newCandidateNames;
      }
   }
   while (!candidates.isEmpty());

   // 注册 Import 相关 Bean
   // Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
   if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
      sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
   }

   if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
      // Clear cache in externally provided MetadataReaderFactory; this is a no-op
      // for a shared cache since it'll be cleared by the ApplicationContext.
      ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
   }
}
```



对于 `processConfigBeanDefinitions` 的分析我们可以分为下面六个步骤：

1. 步骤一：容器内已存在的 Bean 进行候选分类。
2. 步骤二：候选 Bean Definition Holder 的排序。
3. 步骤三：Bean Name 生成器的创建。
4. 步骤四：初始化基本环境信息。
5. 步骤五：解析候选 Bean。
6. 步骤六：注册 Import Bean 和清理数据。



在了解这六个处理步骤后我们来分别探讨每个步骤中做了那些操作。



### 23.3.1 容器内已存在的 Bean 进行候选分类

我们先来看在容器中的 Bean 是如何进行分类的。

- 分类的主题操作代码

```java
List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
// 容器中已存在的 Bean Name
String[] candidateNames = registry.getBeanDefinitionNames();

for (String beanName : candidateNames) {
   BeanDefinition beanDef = registry.getBeanDefinition(beanName);
   // bean definition 中获取属性 CONFIGURATION_CLASS_ATTRIBUTE 是否为空
   if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
      if (logger.isDebugEnabled()) {
         logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
      }
   }
   // 判断当前的 bean Definition 是否属于候选对象
   else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
      configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
   }
}
```

在这段代码中我们可以看到存储分类后的容器是 `configCandidates` ，分类的依据是通过 `ConfigurationClassUtils.checkConfigurationClassCandidate` 方法进行的，这样我们就确定了分类方式，下面我们来看分类的具体代码。



- 分类的依据

```java
public static boolean checkConfigurationClassCandidate(
      BeanDefinition beanDef, MetadataReaderFactory metadataReaderFactory) {

   String className = beanDef.getBeanClassName();
   if (className == null || beanDef.getFactoryMethodName() != null) {
      return false;
   }

   AnnotationMetadata metadata;
   if (beanDef instanceof AnnotatedBeanDefinition &&
         className.equals(((AnnotatedBeanDefinition) beanDef).getMetadata().getClassName())) {
      // Can reuse the pre-parsed metadata from the given BeanDefinition...
      metadata = ((AnnotatedBeanDefinition) beanDef).getMetadata();
   }
   else if (beanDef instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) beanDef).hasBeanClass()) {
      // Check already loaded Class if present...
      // since we possibly can't even load the class file for this Class.
      Class<?> beanClass = ((AbstractBeanDefinition) beanDef).getBeanClass();
      if (BeanFactoryPostProcessor.class.isAssignableFrom(beanClass) ||
            BeanPostProcessor.class.isAssignableFrom(beanClass) ||
            AopInfrastructureBean.class.isAssignableFrom(beanClass) ||
            EventListenerFactory.class.isAssignableFrom(beanClass)) {
         return false;
      }
      metadata = AnnotationMetadata.introspect(beanClass);
   }
   else {
      try {
         MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(className);
         metadata = metadataReader.getAnnotationMetadata();
      }
      catch (IOException ex) {
         if (logger.isDebugEnabled()) {
            logger.debug("Could not find class file for introspecting configuration annotations: " +
                  className, ex);
         }
         return false;
      }
   }

   Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());
   if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {
      beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);
   }
   else if (config != null || isConfigurationCandidate(metadata)) {
      beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);
   }
   else {
      return false;
   }

   // It's a full or lite configuration candidate... Let's determine the order value, if any.
   Integer order = getOrder(metadata);
   if (order != null) {
      beanDef.setAttribute(ORDER_ATTRIBUTE, order);
   }

   return true;
}
```

`checkConfigurationClassCandidate` 方法是对 Bean Definition 进行分类的核心，我们重点关注候选标准的定义，在其中规定了五种候选标准：

1. 第一种：判断 Bean Definition 中 `className` 和 `FactoryMethodName` 是否存在，只要有一个不存在就不符合候选标准。
2. 第二种：判断 Bean Definition 是否属于 `AbstractBeanDefinition` 类型，并且存在 `className`，于此同时如果 `className` 是属于 `BeanFactoryPostProcessor` 、`BeanPostProcessor`、`AopInfrastructureBean` 和 `EventListenerFactory` 中的某一个那么这个 Bean Definiton 不符合候选标准。
3. 第三种：提取注解元数据失败的 Bean Definition 不符合候选标准。
4. 第四种：提取注解源数据中有关 `Configuration` 注解的数据后判断 `proxyBeanMethods` 属性是否为 `false`，如果为 true 该 Bean Definition 不符合候选标准。
5.  第五种：判断注解元信息中是否存在 `Bean` `Component`、`ComponentScan`、 `Import` 和 `ImportResource` ，如果不存在该 Bean Definition 不符合候选标准。

在处理五种候选标准外代码还对 Bean Definition 的一些属性进行了设置，这些属性分别是 `CONFIGURATION_CLASS_ATTRIBUTE` 、`CONFIGURATION_CLASS_ATTRIBUTE`和 `ORDER_ATTRIBUTE`



在了解了候选标准后我们来看我们测试用例中的对象是否符合候选条件，在测试用例中我们定义了这样一个 Component Bean 对象

```java
@Component
@Scope
public class AnnBeans {
// 省略内部代码
}
```

可以发现这个对象符合我们的第五条候选标准，`AnnBeans` 存有 `@Component` 注解。下面我们进入代码debug来看看是否会被加入到候选容器中。

- `configCandidates` 候选容器数据

![image-20210205135355594](/docs/ch-23/images/image-20210205135355594.png)



### 23.3.2 候选 Bean Definition Holder 的排序

现在我们拥有了一些候选 Bean Definition Holder 接下来 Spring 回对这些 Bean Defiintion 进行排序从而保证 Bean 的加载顺序，我们来看排序的过程。

- 候选 Bean Definition Holder 的排序方法

```java
configCandidates.sort((bd1, bd2) -> {
   int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
   int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
   return Integer.compare(i1, i2);
});
```

这里的排序就是一个简单的 `compare` 其本质我们应该关注的点不在此，应该是 `getOrder` 方法，

- `getOrder` 方法详情

```java
public static int getOrder(BeanDefinition beanDef) {
   Integer order = (Integer) beanDef.getAttribute(ORDER_ATTRIBUTE);
   return (order != null ? order : Ordered.LOWEST_PRECEDENCE);
}
```

这段代码中 Spring 会从 Bean Definition 中获取 `ORDER_ATTRIBUTE` 属性，如果数据不存在直接返回最大序号，`ORDER_ATTRIBUTE` 各位可还记得它是从什么时候加入到 Bean Definition 的属性表中的吗？没错，它是在候选分类的是被加入的。





### 23.3.3 Bean Name 生成器的创建

当 Spring 对候选 Bean Definition Holder 这个容器进行排序之后 ，Spring 创建出了一些 Bean Name 生成器用于不同的场景。

- 不同场景的 Bean Name 生成器初始化

```java
// 创建两个 Bean Name 生成器
// 1. 组件扫描的 Bean Name 生成器
// 2. 导入的 Bean Name 生成器
// Detect any custom bean name generation strategy supplied through the enclosing application context
SingletonBeanRegistry sbr = null;
if (registry instanceof SingletonBeanRegistry) {
   sbr = (SingletonBeanRegistry) registry;
   if (!this.localBeanNameGeneratorSet) {
      BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
            AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
      if (generator != null) {
         this.componentScanBeanNameGenerator = generator;
         this.importBeanNameGenerator = generator;
      }
   }
}
```

从这段代码上我们可以看到会创建两个 Bean Name 生成器，这两个 Bean Name 生成器的本质是同一个，一个 Bean Name 是 `org.springframework.context.annotation.internalConfigurationBeanNameGenerator` 的 `BeanNameGenerator`。

如果进一步分析我们应该去找一找这个 Bean 是什么时候被创建的，笔者告诉各位对于这个 Bean 的创建并不是在普通工程中存在，而是在 WEB 工程中才会进行创建。各位读者感兴趣的话可以翻阅 `AnnotationConfigWebApplicationContext#loadBeanDefinitions`，在这个方法中我们可以看到这样一段代码。

- WEB 工程中的 `BeanNameGenerator` 创建

```java
BeanNameGenerator beanNameGenerator = getBeanNameGenerator();
if (beanNameGenerator != null) {
   reader.setBeanNameGenerator(beanNameGenerator);
   scanner.setBeanNameGenerator(beanNameGenerator);
   beanFactory.registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
}
```

我们测试用例中所使用的 `BeanNameGenerator` 不存在对应的数据因此我们获取不到 Bean 实例。



两类 `BeanNameGenerator` 

| 类型                             | 说明                                                         | java doc                                                     |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `componentScanBeanNameGenerator` | 处理组件扫描注册的`BeanNameGenerator`，默认采用短类名，默认实现是 `AnnotationBeanNameGenerator.INSTANCE` | `Using short class names as default bean names by default.`  |
| `importBeanNameGenerator`        | 处理导入的`BeanNameGenerator` ，默认采用全类名，默认实现是 `FullyQualifiedAnnotationBeanNameGenerator` | `Using fully qualified class names as default bean names by default.` |





### 23.3.4 初始化基本环境信息

下面我们来看创建环境信息的代码

```java
if (this.environment == null) {
   this.environment = new StandardEnvironment();
}
```

这段代码十分简单，如果`environment` 变量不存在就创建。







### 23.3.5 解析候选 Bean

下面我们来看 Spring 是如何解析候选 Bean ，请各位读者先阅读下面这段代码。

- 解析候选 Bean 

```java
ConfigurationClassParser parser = new ConfigurationClassParser(
      this.metadataReaderFactory, this.problemReporter, this.environment,
      this.resourceLoader, this.componentScanBeanNameGenerator, registry);

// 解析后的 Bean Definition 容器
Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
// 需要解析的类
Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
do {
   // 解析候选 Bean Definition Holder 集合
   parser.parse(candidates);
   // 解析器中的验证
   parser.validate();

   // 配置类
   Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
   configClasses.removeAll(alreadyParsed);

   // Read the model and create bean definitions based on its content
   if (this.reader == null) {
      this.reader = new ConfigurationClassBeanDefinitionReader(
            registry, this.sourceExtractor, this.resourceLoader, this.environment,
            this.importBeanNameGenerator, parser.getImportRegistry());
   }
   // 解析 Spring Configuration Class 主要目的是提取其中的 Spring 注解并将其转换成 Bean Definition
   this.reader.loadBeanDefinitions(configClasses);
   alreadyParsed.addAll(configClasses);

   candidates.clear();
   if (registry.getBeanDefinitionCount() > candidateNames.length) {
      // 新的候选名称
      String[] newCandidateNames = registry.getBeanDefinitionNames();
      // 历史的候选名称
      Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
      // 已经完成解析的类名
      Set<String> alreadyParsedClasses = new HashSet<>();
      // 加入解析类名
      for (ConfigurationClass configurationClass : alreadyParsed) {
         alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
      }
      // 新的候选名称和对应的 Bean Definition 添加到候选容器中
      for (String candidateName : newCandidateNames) {
         if (!oldCandidateNames.contains(candidateName)) {
            BeanDefinition bd = registry.getBeanDefinition(candidateName);
            if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                  !alreadyParsedClasses.contains(bd.getBeanClassName())) {
               candidates.add(new BeanDefinitionHolder(bd, candidateName));
            }
         }
      }
      candidateNames = newCandidateNames;
   }
}
while (!candidates.isEmpty());
```

从这段代码中我们可以确定负责解析的是 `ConfigurationClassParser` 类，下面我们先来整理处理过程，再来对其进行分析。

再处理过程中第一步会创建 `ConfigurationClassParser` 对象，创建对象的过程没有什么特别的，继续往下我们需要使用 `ConfigurationClassParser` 来进行候选 Bean 的解析。这是我们的重点。

- `ConfigurationClassParser#parse(java.util.Set<org.springframework.beans.factory.config.BeanDefinitionHolder>)` 方法详情

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
   for (BeanDefinitionHolder holder : configCandidates) {
      BeanDefinition bd = holder.getBeanDefinition();
      try {
         if (bd instanceof AnnotatedBeanDefinition) {
            parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
         }
         else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
            parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
         }
         else {
            parse(bd.getBeanClassName(), holder.getBeanName());
         }
      }
      catch (BeanDefinitionStoreException ex) {
         throw ex;
      }
      catch (Throwable ex) {
         throw new BeanDefinitionStoreException(
               "Failed to parse configuration class [" + bd.getBeanClassName() + "]", ex);
      }
   }

   this.deferredImportSelectorHandler.process();
}
```

在这个方法中我们可以看到三个处理策略

1. 策略一：Bean Definition 类型是 `AnnotatedBeanDefinition`。
2. 策略二：Bean Definition 类型是 `AbstractBeanDefinition` 并且拥有 `className` 属性。
3. 策略三：策略一和策略二以外的处理

了解三种处理策略后我们继续向下追踪代码可以找到它们都是由 `processConfigurationClass` 进行处理，我们来看它的代码。

- `processConfigurationClass` 方法详情

```java
protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {
   if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
      return;
   }

   ConfigurationClass existingClass = this.configurationClasses.get(configClass);
   if (existingClass != null) {
      if (configClass.isImported()) {
         if (existingClass.isImported()) {
            existingClass.mergeImportedBy(configClass);
         }
         // Otherwise ignore new imported config class; existing non-imported class overrides it.
         return;
      }
      else {
         // Explicit bean definition found, probably replacing an import.
         // Let's remove the old one and go with the new one.
         this.configurationClasses.remove(configClass);
         this.knownSuperclasses.values().removeIf(configClass::equals);
      }
   }

   // Recursively process the configuration class and its superclass hierarchy.
   SourceClass sourceClass = asSourceClass(configClass);
   do {
      sourceClass = doProcessConfigurationClass(configClass, sourceClass);
   }
   while (sourceClass != null);

   this.configurationClasses.put(configClass, configClass);
}
```

在 `processConfigurationClass` 处理过程中目的是为了建立 `configurationClasses` 缓存和 `importStack` 数据

- 执行前

  ![image-20210205163139577](/docs/ch-23/images/image-20210205163139577.png)

- 执行后

  ![image-20210205163211209](/docs/ch-23/images/image-20210205163211209.png)





继续往下查看 `this.deferredImportSelectorHandler.process()` 的代码调用我们可以看到他处理了有关 `ImportSelector` 和 `ImportBeanDefinitionRegistrar`，这部分处理过程笔者就不在整理展开了(在第二十七章中展开)，现在我们对于 `parser.parse(candidates)` 的分析告一段落，下面我们来看验证代码 ( `parser.validate()`)的细节.



-  `parser.validate()` 方法详情

```java
public void validate() {
   for (ConfigurationClass configClass : this.configurationClasses.keySet()) {
      configClass.validate(this.problemReporter);
   }
}

public void validate(ProblemReporter problemReporter) {
    // A configuration class may not be final (CGLIB limitation) unless it declares proxyBeanMethods=false
    Map<String, Object> attributes = this.metadata.getAnnotationAttributes(Configuration.class.getName());
    if (attributes != null && (Boolean) attributes.get("proxyBeanMethods")) {
        if (this.metadata.isFinal()) {
            problemReporter.error(new FinalConfigurationProblem());
        }
        for (BeanMethod beanMethod : this.beanMethods) {
            beanMethod.validate(problemReporter);
        }
    }
}
```

在这段代码中我们可以看大 Spring 对每个候选配置类都会进行验证，验证规则如下

1. 判断当前 Bean Definition 的注解元数据是否存在 `Configuration` 注解属性。
2. 判断注解元数据中 `proxyBeanMethods` 属性是否为 `true` ，该属性从 `Configuration` 注解中获取。
3. 判断注解元数据是否是 `final` 修饰的基本类。
4. 通过 BeanMethod 来验证当前的数据
   1. 验证是否是 `static` 修饰
   2. 是否存在 `Configuration` 注解，同时是否存在需要重写的方法

以上这四项就是单个 `ConfigurationClass` 对象的验证过程，下面我们来看其中出现的注解 `Configuration` 和 `BeanMethod` 中的验证代码。

- `Configuration` 注解信息

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {

   @AliasFor(annotation = Component.class)
   String value() default "";

   boolean proxyBeanMethods() default true;

}
```

- `BeanMethod` 中的验证方法。

```java
@Override
public void validate(ProblemReporter problemReporter) {
   if (getMetadata().isStatic()) {
      // static @Bean methods have no constraints to validate -> return immediately
      return;
   }

   if (this.configurationClass.getMetadata().isAnnotated(Configuration.class.getName())) {
      if (!getMetadata().isOverridable()) {
         // instance @Bean methods within @Configuration classes must be overridable to accommodate CGLIB
         problemReporter.error(new NonOverridableMethodError());
      }
   }
}
```

至此我们对于验证相关的内容已经了解下面我们继续向下阅读代码来看后续的处理内容。

下面我们先来看这段代码

```java
// 配置类
Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
configClasses.removeAll(alreadyParsed);

// Read the model and create bean definitions based on its content
if (this.reader == null) {
   this.reader = new ConfigurationClassBeanDefinitionReader(
         registry, this.sourceExtractor, this.resourceLoader, this.environment,
         this.importBeanNameGenerator, parser.getImportRegistry());
}
// 解析 Spring Configuration Class 主要目的是提取其中的 Spring 注解并将其转换成 Bean Definition
this.reader.loadBeanDefinitions(configClasses);
alreadyParsed.addAll(configClasses);
```

在这段代码中第一个操作就是将 `ConfigurationClassParser` 的数据提取出来，该数据是 Spring Configuration Bean，这段代码中的后续操作 `this.reader.loadBeanDefinitions` 就是为了从中读取 Bean Definition ，在这里我们可以看到核心能力提供者是 `ConfigurationClassBeanDefinitionReader` 笔者在这里不做全面展开，这里我们只需要知道它可以将 Spring Configuration Bean 中的 Bean Definition 读取并加入到容器中即可，具体分析在第二十八章展开。下面我们来看执行 `loadBeanDefinitions` 前后 Spring 容器中的数据变化。

- 执行前

![image-20210207093319724](/docs/ch-23/images/image-20210207093319724.png)

- 执行后

![image-20210207093333020](/docs/ch-23/images/image-20210207093333020.png)



可以看到当我们执行完成 `this.reader.loadBeanDefinitions(configClasses)` 代码后 Spring 中的 Bean Definition 容器中增加了 `abc` 这个 Bean Definition 数据，该数据就是我们在测试用例中使用 `@Bean` 标记的内容。

解析完成这一批 `configClasses` 后会将这批 `configClasses` 对象放入到 `alreadyParsed`，同时将 `candidates` 清空，清空的目的是为了存放后续需要处理的数据，下面我们来看这部分操作代码



```java
    candidates.clear();
   if (registry.getBeanDefinitionCount() > candidateNames.length) {
      // 新的候选名称
      String[] newCandidateNames = registry.getBeanDefinitionNames();
      // 历史的候选名称
      Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
      // 已经完成解析的类名
      Set<String> alreadyParsedClasses = new HashSet<>();
      // 加入解析类名
      for (ConfigurationClass configurationClass : alreadyParsed) {
         alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
      }
      // 新的候选名称和对应的 Bean Definition 添加到候选容器中
      for (String candidateName : newCandidateNames) {
         if (!oldCandidateNames.contains(candidateName)) {
            BeanDefinition bd = registry.getBeanDefinition(candidateName);
            if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                  !alreadyParsedClasses.contains(bd.getBeanClassName())) {
               candidates.add(new BeanDefinitionHolder(bd, candidateName));
            }
         }
      }
      candidateNames = newCandidateNames;
   }
}
```

在这段代码中我们首先需要明确是什么情况下能够进行处理，进入的条件是**当前注册其中 BeanDefinition 的数量大于方法最开始入口的候选BeanDefinition名称列表的数量**

- `candidateNames` 数据信息

![image-20210207094633833](/docs/ch-23/images/image-20210207094633833.png)

- 执行到判读语句时容器中的 Bean Definition Name 情况

![image-20210207094731229](/docs/ch-23/images/image-20210207094731229.png)



通过这两张图我们可以很好的理解代码中的两个变量 `newCandidateNames` 和 `oldCandidateNames`，它们产生**数据差异的起源是 `reader.loadBeanDefinitions(configClasses)` 方法**，在这个方法中通过解析 Spring Configuration Bean 将其中的 Bean Defintion 注册到容器中因此产生了数据差异，**`oldCandidateNames` 可以理解成方法执行前容器中存在的 Bean Definiton Name 列表**， **`newCandidateNames` 可以理解成方法执行后容器中存在的 Bean Definiton Name 列表**。

理解完成这两个变量后我们来看 `alreadyParsedClasses` 是做什么的。`alreadyParsedClasses` 容器中的数据是从 `alreadyParsed` 中得到的，那么我们的问题转变成 `alreadyParsed` 中存储的是什么呢？我们在上一个步骤中知道了 **`alreadyParsed` 存储的是已经解析完成的 `ConfigurationClass`** ，下面我们来看该对象的数据信息。

- `alreadyParsed` 数据信息

![image-20210207100113664](/docs/ch-23/images/image-20210207100113664.png)

继续往下通过循环将 `configurationClass` 的数据进一步提取，提取类全路径，即解析过的类全类名提取加入到 `alreadyParsedClasses` 中。

- `alreadyParsedClasses` 数据信息

![image-20210207100243589](/docs/ch-23/images/image-20210207100243589.png)



现在我们了解了 `newCandidateNames` 、`oldCandidateNames` 和 `alreadyParsedClasses` 存储的数据，下面我们来看最后的一段操作，加入Bean Definition 候选容器。这里我们需要思考一个问题什么类型的 Bean Definition 需要放入到候选容器中？这个问题的答案是在 `reader.loadBeanDefinitions(configClasses)` 调用后容器中的 Bean Definition 需要放入候选容器的。下面我们来详细的看这里的处理。

1. `newCandidateNames` 中的 Bean Name 是否在 `oldCandidateNames` 存在，存在不会加入到容器，不存在进入下一步判断。
2. 提取 New Bean Name 对应的 Bean Definition 进行两层验证
   1. 第一层：验证当前的 Bean Definition 是否符合候选类。(这里的处理方法是`checkConfigurationClassCandidate` ，前文有所介绍各位请自行翻阅)
   2. 第二层：`alreadyParsedClasses` 中是否存在当前 Bean Definition 的类名，不存在会被加入到容器。

当符合这两重验证后会在重复进行 Bean Defintion 解析流程，即重复 `do` 代码块。



通过这样的处理我们完成了候选 Bean 的解析操作。



### 23.3.6 注册 Import Bean 和清理数据

下面我们来看注册 Import Bean 和清理数据相关的处理操作。

```java
// 注册 Import 相关 Bean
// Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
   sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
}

if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
   // Clear cache in externally provided MetadataReaderFactory; this is a no-op
   // for a shared cache since it'll be cleared by the ApplicationContext.
   ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
}
```

这里总共执行两个事项

1. 第一事项：注册 Import Bean

   将 Import Bean 注册到 `ConfigurationClassPostProcessor.class.getName() + ".importRegistry"` 名称对应的数据中

2. 第二事项：清理数据

   清理数据是将 `metadataReaderCache` 缓存清空



至此笔者对于 `ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry` 方法的分析完成。









## 23.4 `postProcessBeanFactory` 分析

接下来笔者将和各位一起探讨在 `ConfigurationClassPostProcessor` 类中的另一个核心方法 `postProcessBeanFactory`

- `ConfigurationClassPostProcessor#postProcessBeanFactory` 方法详情

```java
@Override
public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
   int factoryId = System.identityHashCode(beanFactory);
   if (this.factoriesPostProcessed.contains(factoryId)) {
      throw new IllegalStateException(
            "postProcessBeanFactory already called on this post-processor against " + beanFactory);
   }
   this.factoriesPostProcessed.add(factoryId);
   if (!this.registriesPostProcessed.contains(factoryId)) {
      // BeanDefinitionRegistryPostProcessor hook apparently not supported...
      // Simply call processConfigurationClasses lazily at this point then.
      processConfigBeanDefinitions((BeanDefinitionRegistry) beanFactory);
   }

   enhanceConfigurationClasses(beanFactory);
   beanFactory.addBeanPostProcessor(new ImportAwareBeanPostProcessor(beanFactory));
}
```



在这段代码中我们可以看到一个熟悉的方法 `processConfigBeanDefinitions` 该方法是 `postProcessBeanDefinitionRegistry` 处理的核心，前文已有分析笔者不在此处重复分析，下面我们在这段代码中只剩下一个不明确的方法 `enhanceConfigurationClasses` 该方法就是我们下面的讨论重点。

进行方法分析之前笔者先提出一个问题，这两个方法的执行顺序谁先谁后。第一执行顺序是 `BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry` 第二执行顺序是 `BeanFactoryPostProcessor#postProcessBeanFactory`，这个问题的结论从 `PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanFactoryPostProcessor>)` 中可以得到答案，重点关注 `invokeBeanFactoryPostProcessors` 和 `invokeBeanFactoryPostProcessors` 的调用过程。



下面我们来看 `enhanceConfigurationClasses` 方法，该方法时用来增强 Spring Configuration Bean ，下面我们来看具体代码处理。

- `enhanceConfigurationClasses`  方法详情

```java
public void enhanceConfigurationClasses(ConfigurableListableBeanFactory beanFactory) {
	// 存储 Spring Configuration Bean Definiton
	Map<String, AbstractBeanDefinition> configBeanDefs = new LinkedHashMap<>();
	// 容器中将符合条件的对象放入到 配置Bean 容器中
	for (String beanName : beanFactory.getBeanDefinitionNames()) {
		BeanDefinition beanDef = beanFactory.getBeanDefinition(beanName);
		Object configClassAttr = beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE);
		MethodMetadata methodMetadata = null;
		if (beanDef instanceof AnnotatedBeanDefinition) {
			methodMetadata = ((AnnotatedBeanDefinition) beanDef).getFactoryMethodMetadata();
		}
		if ((configClassAttr != null || methodMetadata != null) && beanDef instanceof AbstractBeanDefinition) {
			// Configuration class (full or lite) or a configuration-derived @Bean method
			// -> resolve bean class at this point...
			AbstractBeanDefinition abd = (AbstractBeanDefinition) beanDef;
			if (!abd.hasBeanClass()) {
				try {
					abd.resolveBeanClass(this.beanClassLoader);
				}
				catch (Throwable ex) {
					throw new IllegalStateException(
							"Cannot load configuration class: " + beanDef.getBeanClassName(), ex);
				}
			}
		}
		if (ConfigurationClassUtils.CONFIGURATION_CLASS_FULL.equals(configClassAttr)) {
			if (!(beanDef instanceof AbstractBeanDefinition)) {
				throw new BeanDefinitionStoreException("Cannot enhance @Configuration bean definition '" +
						beanName + "' since it is not stored in an AbstractBeanDefinition subclass");
			}
			else if (logger.isInfoEnabled() && beanFactory.containsSingleton(beanName)) {
				logger.info("Cannot enhance @Configuration bean definition '" + beanName +
						"' since its singleton instance has been created too early. The typical cause " +
						"is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor " +
						"return type: Consider declaring such methods as 'static'.");
			}
			configBeanDefs.put(beanName, (AbstractBeanDefinition) beanDef);
		}
	}
	// Spring Configuration Bean Definition 不存在
	if (configBeanDefs.isEmpty()) {
		// nothing to enhance -> return immediately
		return;
	}

	// 对 Spring Configuration Bean Definition 进行增强处理
	ConfigurationClassEnhancer enhancer = new ConfigurationClassEnhancer();
	for (Map.Entry<String, AbstractBeanDefinition> entry : configBeanDefs.entrySet()) {
		AbstractBeanDefinition beanDef = entry.getValue();
		// If a @Configuration class gets proxied, always proxy the target class
		beanDef.setAttribute(AutoProxyUtils.PRESERVE_TARGET_CLASS_ATTRIBUTE, Boolean.TRUE);
		// Set enhanced subclass of the user-specified bean class
		Class<?> configClass = beanDef.getBeanClass();
		Class<?> enhancedClass = enhancer.enhance(configClass, this.beanClassLoader);
		if (configClass != enhancedClass) {
			if (logger.isTraceEnabled()) {
				logger.trace(String.format("Replacing bean definition '%s' existing class '%s' with " +
						"enhanced class '%s'", entry.getKey(), configClass.getName(), enhancedClass.getName()));
			}
			beanDef.setBeanClass(enhancedClass);
		}
	}
}
```



这段代码做了两件事情：

1. 第一件：将容器中的 Bean Definition 过滤得到 Spring Configuration Bean 。
2. 第二件：对得到的 Spring Configuration Bean 进行增强处理。

下面我先来进行第一件事情的源码分析，先整理代码中出现的各个操作流程：

1. 第一步：提权三个属性
   1. 根据 Bean Name 在容器中找到对应的 Bean Defintion (`beanDef`)。
   2. 从 Bean Defintion 中提取属性 `ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE` (`configClassAttr`)。
   3. 如果 Bean Defintion 是 `AnnotatedBeanDefinition` 类型提取 `MethodMetadata` (`methodMetadata`) 。
2. 第二步：Bean Definition 处理 Bean Class  (`abd.resolveBeanClass(this.beanClassLoader)`)。
3. 第三步：判断是否可以加入到 `configBeanDefs` 中，判断依据如下
   1. 第一层：当前 Bean Defintion 中的 `ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE` 属性是 `full`
   2. 第二层：当前 Bean Defintion 不是 `AbstractBeanDefinition` 抛出异常



了解了第一件事的处理逻辑后欧美来看处理后的数据存储情况

![image-20210207112616586](/docs/ch-23/images/image-20210207112616586.png) 

目前我们的 Bean Definition 是不满足条件的，目前不满足条件的原因是 `ConfigurationClassUtils.CONFIGURATION_CLASS_FULL.equals(configClassAttr)` ，我们来看看这个值的设置，设置  `CONFIGURATION_CLASS_ATTRIBUTE` 属性在 `checkConfigurationClassCandidate` 中有相关处理，这部分处理操作的入口是  `postProcessBeanDefinitionRegistry` ，这里我们对于方法的执行顺序又做了一次强调。

- `CONFIGURATION_CLASS_ATTRIBUTE` 属性设置

```java
Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());
if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {
   beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);
}
else if (config != null || isConfigurationCandidate(metadata)) {
   beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);
}
```

- 测试用例中`AnnBeans` 的属性 

![image-20210207113053806](/docs/ch-23/images/image-20210207113053806.png)



最后我们来看配置类增强的操作逻辑，增强类的处理操作其本质是将原有的 `Class` 替换成增强后的 `Class` ，在这里将是将 Bean Class 从原始的替换成  `ConfigurationClassEnhancer`

操作流程如下：

1. 第一步：设置 `PRESERVE_TARGET_CLASS_ATTRIBUTE` 属性为 `true` ，表示这是一个增强类。
2. 第二步：提取 Bean Defintion 中原始的 `Class`。
3. 第三步：通过 `ConfigurationClassEnhancer` 对原始的 `Class` 进行增强。
4. 第四步：将增强后的 `Class` 填写到 Bean Definition 的 `beanClass` 中。







## 23.5 总结

在这一章节中笔者和各位一起探讨了关于 `ConfigurationClassPostProcessor` 中的源码处理，围绕两个接口（一个是 `BeanDefinitionRegistryPostProcessor` ，另一个是 `BeanFactoryPostProcessor` ）分析了它们所提供方法在 `ConfigurationClassPostProcessor` 中的实现过程，并在实现过程中又一次了解了这两个接口的执行顺序。