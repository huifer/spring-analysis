# 第二十九章 Spring 配置类解析
在这一章笔者将和各位一起探讨 Spring 是如何对配置类(Spring Configuration Bean) 进行解析。





## 29.1 `parse` 方法分析

对于 Spring 配置类的解析笔者在第二十三章的时候简单提到过一个方法。

- `ConfigurationClassPostProcessor#processConfigBeanDefinitions` 中关于配置类的解析

```java
 // ..省略其他
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
 // ..省略其他   
}
```

在这段代码中我们对于 `parser.parser` 当时的了解不是很深入，现在笔者将对这个方法做出更详细的分析。下面我们先来看这个方法的代码



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



在这段 `parse` 方法中主要执行两个任务：

1. 第一个：对 Spring Configuration Bean 的解析操作。
2. 第二个：`ImportSelector` 相关解析操作。

本章主要针对第一个任务进行分析，第二个任务的分析在第二十七章中有详细分析。





## 29.2 `processConfigurationClass` 分析

跟着 `parse` 方法我们继续向下追踪源代码，不难发现它依赖了 `processConfigurationClass` 方法。

- `parse` 方法对 `processConfigurationClass` 方法的依赖

```java
protected final void parse(AnnotationMetadata metadata, String beanName) throws IOException {
   processConfigurationClass(new ConfigurationClass(metadata, beanName));
}
```



下面我们就先对 `processConfigurationClass` 方法进行分析看看在这个方法中做了那些操作。

- `processConfigurationClass` 方法详情

```java
protected void processConfigurationClass(ConfigurationClass configClass) throws IOException {
   // 条件注解解析，阶段为配置解析阶段
   if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
      return;
   }

   // 尝试从配置类缓存中获取配置类
   ConfigurationClass existingClass = this.configurationClasses.get(configClass);
   // 缓存中的配置类存在
   if (existingClass != null) {
      // 判断 配置类是否是导入的
      if (configClass.isImported()) {
         // 判断缓存中的配置类是否是导入的
         if (existingClass.isImported()) {
            // 合并配置信息
            existingClass.mergeImportedBy(configClass);
         }
         // Otherwise ignore new imported config class; existing non-imported class overrides it.
         return;
      }
      else {
         // Explicit bean definition found, probably replacing an import.
         // Let's remove the old one and go with the new one.
         // 从配置类缓存中移除当前配置类
         this.configurationClasses.remove(configClass);
         // 从已知的配置类中移除当前的配置类
         this.knownSuperclasses.values().removeIf(configClass::equals);
      }
   }

   // Recursively process the configuration class and its superclass hierarchy.
   // 将配置类转换成 SourceClass
   SourceClass sourceClass = asSourceClass(configClass);
   do {
      // 解析配置类
      sourceClass = doProcessConfigurationClass(configClass, sourceClass);
   }
   while (sourceClass != null);

   // 放入配置类缓存
   this.configurationClasses.put(configClass, configClass);
}
```

通过这段代码的阅读我们来整理整个方法的处理流程

1. 对配置类进行条件注解的条件判断

2. 从配置类缓存中获取配置类对应的缓存配置类信息 `existingClass`

3. 如果 `existingClass` 存在

   1. 如果当前需要处理的配置类是导入的

      1. 如果缓存中的配置类 `existingClass` 是导入的

         `existingClass` 将和当前正在处理的配置类进行 `import` 数据合并

   2. 如果当前需要处理的配置类不是导入的的

      1. 从配置类缓存中移除当前配置类
      2. 从已知的配置类中移除当前配置类

4. 将配置类转换成 `SourceClass` 对象

5. 解析配置类

6. 将配置类放入配置类缓存中



在这个方法中我们出现了几个变量，下面我们来看这些变量的信息





| 变量名称               | 变量类型                                      | 变量作用                                           |
| ---------------------- | --------------------------------------------- | -------------------------------------------------- |
| `conditionEvaluator`   | `ConditionEvaluator`                          | 条件注解的解析器                                   |
| `configurationClasses` | `Map<ConfigurationClass, ConfigurationClass>` | 存储配置类的缓存结构，key value 相同               |
| `knownSuperclasses`    | `Map<String, ConfigurationClass>`             | 已知的配置类<br />key：父类名称<br />value：配置类 |



下面笔者来对这三个变量进行数据展示，为了演示我们需要准备下面的代码

- 父配置类

```java
@Configuration
public class SupperConfiguration {

}
```

- 子配置类

```java
@Configuration
public class ExtendConfiguration extends SupperConfiguration {

}
```

- 注解上下文

```java
@Configuration
@Import(ExtendConfiguration.class)
public class ThreeVarBeans {
}
```

- 测试用例

```java
@Test
void testThreeVar() {
   AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ThreeVarBeans.class);
}
```

一切准备就绪下面我们来看当前容器中的数据情况

- `configurationClasses` 变量和 `knownSuperclasses` 变量数据

![image-20210209094358670](/docs/ch-29/images/image-20210209094358670.png)









## 29.3 `doProcessConfigurationClass` 方法分析

现在我们了解了解析后的一个结果，下面我们将对这个处理解析的核心方法 `doProcessConfigurationClass` 进行分析。我们先来看 `doProcessConfigurationClass` 中的源代码



- `doProcessConfigurationClass` 方法详情

```java
@Nullable
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
    throws IOException {

    // 配置类是否存在 Component 注解
    if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
        // Recursively process any member (nested) classes first
        processMemberClasses(configClass, sourceClass);
    }

    // 处理 PropertySource 和 PropertySource 注解
    // Process any @PropertySource annotations
    for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), PropertySources.class,
        org.springframework.context.annotation.PropertySource.class)) {
        if (this.environment instanceof ConfigurableEnvironment) {
            processPropertySource(propertySource);
        }
        else {
            logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
                        "]. Reason: Environment must implement ConfigurableEnvironment");
        }
    }

    // 处理 ComponentScans ComponentScan 注解
    // Process any @ComponentScan annotations
    Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
    if (!componentScans.isEmpty() &&
        !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
        for (AnnotationAttributes componentScan : componentScans) {
            // The config class is annotated with @ComponentScan -> perform the scan immediately
            Set<BeanDefinitionHolder> scannedBeanDefinitions =
                this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
            // Check the set of scanned definitions for any further config classes and parse recursively if needed
            for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
                BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
                if (bdCand == null) {
                    bdCand = holder.getBeanDefinition();
                }
                if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
                    parse(bdCand.getBeanClassName(), holder.getBeanName());
                }
            }
        }
    }

    // 处理 Import 注解
    // Process any @Import annotations
    processImports(configClass, sourceClass, getImports(sourceClass), true);

    // 处理 ImportResource 注解
    // Process any @ImportResource annotations
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

    // 处理 Bean 注解
    // Process individual @Bean methods
    Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
    for (MethodMetadata methodMetadata : beanMethods) {
        configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
    }

    // 处理 Bean Method
    // Process default methods on interfaces
    processInterfaces(configClass, sourceClass);

    // 父类配置处理
    // Process superclass, if any
    if (sourceClass.getMetadata().hasSuperClass()) {
        String superclass = sourceClass.getMetadata().getSuperClassName();
        if (superclass != null && !superclass.startsWith("java") &&
            !this.knownSuperclasses.containsKey(superclass)) {
            this.knownSuperclasses.put(superclass, configClass);
            // Superclass found, return its annotation metadata and recurse
            return sourceClass.getSuperClass();
        }
    }

    // No superclass -> processing is complete
    return null;
}
```

通过阅读这段代码后我们来整理处理顺序

1. 处理 `@Component` 注解
2. 处理 `@PropertySource` 和 `@PropertySources`  注解
3. 处理 `@ComponentScans` 和 `@ComponentScan` 注解
4. 处理 `@Import` 注解
5. 处理 `@ImportResource` 注解
6. 处理 `@Bean` 注解
7. 处理父类配置

现在我们了解了这七个处理顺序，接下来我们需要做的事情就是对这七个处理行为做更进一步的分析。





## 29.4 处理 `@Component` 注解

我们先来看注解 `@Component` 的处理过程，先看入口代码

- `@Component` 注解处理的入口代码

```java
// 配置类是否存在 Component 注解
if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
   // Recursively process any member (nested) classes first
   processMemberClasses(configClass, sourceClass);
}
```

这段代码我们需要继续向下追踪找到 `processMemberClasses` ，下面我们来看 `processMemberClasses` 的代码

- `processMemberClasses` 方法详情

```java
private void processMemberClasses(ConfigurationClass configClass, SourceClass sourceClass) throws IOException {
   // 找到当前配置类中存在的成员类
   Collection<SourceClass> memberClasses = sourceClass.getMemberClasses();
   // 成员类列表不为空
   if (!memberClasses.isEmpty()) {
      List<SourceClass> candidates = new ArrayList<>(memberClasses.size());
      for (SourceClass memberClass : memberClasses) {
         // 成员类是否符合配置类候选标准 Component ComponentScan Import ImportResource 注解是否存在
         // 成员类是否和配置类同名
         if (ConfigurationClassUtils.isConfigurationCandidate(memberClass.getMetadata()) &&
               !memberClass.getMetadata().getClassName().equals(configClass.getMetadata().getClassName())) {

            candidates.add(memberClass);
         }
      }
      // 排序候选类
      OrderComparator.sort(candidates);
      for (SourceClass candidate : candidates) {
         // 判断 importStack 中是否存在当前配置类
         if (this.importStack.contains(configClass)) {
            this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
         }
         else {
            this.importStack.push(configClass);
            try {
               // 解析成员类
               processConfigurationClass(candidate.asConfigClass(configClass));
            }
            finally {
               this.importStack.pop();
            }
         }
      }
   }
}
```

阅读这段代码后我们来整理这段代码的处理流程

1. **提取当前配置类中的成员类**

2. **对成员类列表进行条件过滤，将符合下面条件的类做收集**

   **条件：成员类存在 `Component`、 `ComponentScan`、 `Import`、 `ImportResource` 或者 `Bean` 注解**

3. **对收集到的内部配置类进行排序。**

4. **对符合条件的内部类进行配置类解析。**



为了更好的进行这段代码的调试和代码的理解，我们可以编写下面这样的代码来进行测试。

- 定义一个配置类，在这个配置类中在编写两个内部类，同时一个内部类用 `@Component` 注解标记

```java
@Configuration
public class BigConfiguration {


   public class SmallConfigA {

   }

   @Component
   public class SmallConfigB {


   }
}
```

注解上下文编写完成，下面我们来编写测试用例

```java
@Test
void testProcessMemberClasses(){
   AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BigConfiguration.class);

}
```

下面我们来看这个测试用例中的一些数据情况，首先我们要看的是 `memberClasses` 变量，该对象表示了当前配置类下有那些内部类

- `memberClasses` 数据信息

![image-20210209102740698](/docs/ch-29/images/image-20210209102740698.png)

接下来我们看 `candidates` 变量，该对象表示通过条件过来的内部类，这些内部类会被当做 Spring Configuration Bean 处理

- `candidates` 数据信息

![image-20210209102904873](/docs/ch-29/images/image-20210209102904873.png)

最后这些符合条件的内部类会进行配置类解析。这里对于配置类的解析其实就是本文所讨论的内容。





## 29.5 处理 `@PropertySource` 和 `@PropertySources`  注解

接下来我们来看 `@PropertySource` 和 `@PropertySources`  注解的处理入口

-  `@PropertySource` 和 `@PropertySources`  注解处理入口

```java
for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), PropertySources.class,
      org.springframework.context.annotation.PropertySource.class)) {
   if (this.environment instanceof ConfigurableEnvironment) {
      processPropertySource(propertySource);
   }
   else {
      logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
            "]. Reason: Environment must implement ConfigurableEnvironment");
   }
}
```

通过注释我们了解到了这是对两个注解的处理，下面我们需要来编写这两个注解的使用案例，有了这个使用案例我们就可以更好的进行分析。

首先我们来编写一个配置文件 `data.properties`

```java
a=123
```

接下来我们编写测试类及测试方法

```java
@PropertySource(value = "classpath:data.properties")
@Configuration
public class PropertySourceTest {
   @Test
   void testPropertySource() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(PropertySourceTest.class);
      assert context.getEnvironment().getProperty("a").equals("123");
   }
}
```

这样我们就做好了一个测试类下面我们就开始源码分析吧。

首先我们来看这两个注解的定义

- `PropertySource` 注解定义

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {

   String name() default "";

   String[] value();

   boolean ignoreResourceNotFound() default false;

   String encoding() default "";

   Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;

}
```

- `PropertySources` 注解定义

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PropertySources {

   PropertySource[] value();

}
```

通过阅读注解`@PropertySources` 我们可以发现该注解内部包含的属性 `value` 是注解 `@PropertySource` 的集合，那么我们对于这段代码的理解就容易了。

```java
AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), PropertySources.class,
      org.springframework.context.annotation.PropertySource.class)
```

这段代码将两个注解的数据都进行读取得到一个 `PropertySources` 注解数据集合，我们来改造一下测试类上的注解

```java
@PropertySource(value = "classpath:data.properties")
@PropertySources(value = @PropertySource("classpath:InPropertySources"))
@Configuration
public class PropertySourceTest {
	@Test
	void testPropertySource() {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(PropertySourceTest.class);
		assert context.getEnvironment().getProperty("a").equals("123");
	}
}
```

通过这样的改造我们来看这个方法得到的数据情况

- `PropertySources` 读取后的数据

![image-20210209104743908](/docs/ch-29/images/image-20210209104743908.png)

可以看到这里所有的注解会被整合在一个集合中这个集合的数据是注解`@PropertySource`的全属性，由于笔者没有编写 `InPropertySources` 文件我们将测试类还原

```java
@PropertySource(value = "classpath:data.properties")
@Configuration
public class PropertySourceTest {
   @Test
   void testPropertySource() {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(PropertySourceTest.class);
      assert context.getEnvironment().getProperty("a").equals("123");
   }
}
```



现在我们了解了 `propertySource` 中的数据，下面我们来看 `processPropertySource` 方法的处理过程。

- `processPropertySource` 方法详情

```java
private void processPropertySource(AnnotationAttributes propertySource) throws IOException {
   // 提取注解属性中的 name 属性
   String name = propertySource.getString("name");
   if (!StringUtils.hasLength(name)) {
      name = null;
   }
   // 提取注解属性中的 encoding 属性
   String encoding = propertySource.getString("encoding");
   if (!StringUtils.hasLength(encoding)) {
      encoding = null;
   }
   // 提取注解属性中的 value 属性
   String[] locations = propertySource.getStringArray("value");
   Assert.isTrue(locations.length > 0, "At least one @PropertySource(value) location is required");

   // 提取注解属性中的 ignoreResourceNotFound 属性
   boolean ignoreResourceNotFound = propertySource.getBoolean("ignoreResourceNotFound");

   // 提取注解属性中的 factory 属性
   Class<? extends PropertySourceFactory> factoryClass = propertySource.getClass("factory");
   PropertySourceFactory factory = (factoryClass == PropertySourceFactory.class ?
         DEFAULT_PROPERTY_SOURCE_FACTORY : BeanUtils.instantiateClass(factoryClass));

   // 循环处理每个配置文件
   for (String location : locations) {
      try {
         // 解析配置文件地址
         String resolvedLocation = this.environment.resolveRequiredPlaceholders(location);
         // 资源加载其加载资源
         Resource resource = this.resourceLoader.getResource(resolvedLocation);
         // 加入配置
         addPropertySource(factory.createPropertySource(name, new EncodedResource(resource, encoding)));
      }
      catch (IllegalArgumentException | FileNotFoundException | UnknownHostException ex) {
         // Placeholders not resolvable or resource not found when trying to open it
         if (ignoreResourceNotFound) {
            if (logger.isInfoEnabled()) {
               logger.info("Properties location [" + location + "] not resolvable: " + ex.getMessage());
            }
         }
         else {
            throw ex;
         }
      }
   }
}
```

阅读源代码后我们先来对这个方法进行流程整理。

1. 提取注解属性对象中关于 `@PropertySource` 注解的属性
2. 对属性中 `locations` 进行单个处理
   1. 将 `location` 进行路径解析
   2. 将 `location` 解析后的结果进行资源对象读取转换成 `Resource` 对象
   3. 添加属性 

对于 `location` 的解析请翻阅第十二章

对于 `location` 转换成 `Resoruce` 的解析请翻阅第十八章

最后添加属性我们还没没有对其进行分析，下面我们来看这里的处理。首先我们需要理解 `factory.createPropertySource(name, new EncodedResource(resource, encoding))` 这段代码。这里我们对于 `factory` 变量需要先做认识，在提取注解属性的时候 Spring 对这个变量的获取做出如下操作

- `PropertySourceFactory` 对象的确认

```java
PropertySourceFactory factory = (factoryClass == PropertySourceFactory.class ?
      DEFAULT_PROPERTY_SOURCE_FACTORY : BeanUtils.instantiateClass(factoryClass));
```

这里我们可以看到它是对注解配置中 `factory` 是否存在做出两种处理。

1. 第一种：如果 `factory`  属性不存在则采用默认的 `PropertySourceFactory`，具体类是 `org.springframework.core.io.support.DefaultPropertySourceFactory`
2. 第二种：如果  `factory`  属性存在通过 `BeanUtils` 反射创建该对象

我们的测试用例中符合第一种处理情况， 我们来看 `DefaultPropertySourceFactory` 中的实现

- `DefaultPropertySourceFactory` 类详情

```java
public class DefaultPropertySourceFactory implements PropertySourceFactory {

   @Override
   public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
      return (name != null ? new ResourcePropertySource(name, resource) : new ResourcePropertySource(resource));
   }

}
```

这里就是创建一个对象 `ResourcePropertySource` ，我们继续深入挖掘 `ResourcePropertySource` 的构造函数

- `name` 和 `resource` 的构造函数

```java
public ResourcePropertySource(String name, EncodedResource resource) throws IOException {
   // 设置 name + map 对象
   // map 对象是 资源信息
   super(name, PropertiesLoaderUtils.loadProperties(resource));
   // 获取 resource name
   this.resourceName = getNameForResource(resource.getResource());
}
```

- `resource` 的构造函数

```java
public ResourcePropertySource(EncodedResource resource) throws IOException {
   // 设置 key: name, resource 的 name
   // 设置 value: resource 资源信息
   super(getNameForResource(resource.getResource()), PropertiesLoaderUtils.loadProperties(resource));
   this.resourceName = null;
}
```

源码追踪到这里相信各位发现了这两段构造函数的共同点： `PropertiesLoaderUtils.loadProperties(resource)` ，该方法将资源文件解析转换成`Properties` 对象。

现在我们对于 `factory.createPropertySource(name, new EncodedResource(resource, encoding))` 方法的分析完成了，我们来看看这个方法的执行结果

- `factory.createPropertySource(name, new EncodedResource(resource, encoding))` 执行结果

![image-20210209110901603](/docs/ch-29/images/image-20210209110901603.png)

可以看到 `data.properties` 中的数据被解析出来了，并存储在 `source` 中。`PropertySource` 对象的获取我们已经完成，接下来我们需要做的事情是将这个数据放入到 Spring 中，具体方法是 `addPropertySource` 

- `addPropertySource` 方法详情

```java
private void addPropertySource(PropertySource<?> propertySource) {
   // 获取配置名称
   String name = propertySource.getName();
   // 提取环境对象这种的 MutablePropertySources 对象
   MutablePropertySources propertySources = ((ConfigurableEnvironment) this.environment).getPropertySources();

   // 当前处理的配置名称是否在 propertySourceNames 存在
   if (this.propertySourceNames.contains(name)) {
      // We've already added a version, we need to extend it
      // 从 propertySources 中获取当前配置名称对应的 PropertySource
      PropertySource<?> existing = propertySources.get(name);
      if (existing != null) {
         PropertySource<?> newSource = (propertySource instanceof ResourcePropertySource ?
               ((ResourcePropertySource) propertySource).withResourceName() : propertySource);
         if (existing instanceof CompositePropertySource) {
            ((CompositePropertySource) existing).addFirstPropertySource(newSource);
         }
         else {
            if (existing instanceof ResourcePropertySource) {
               existing = ((ResourcePropertySource) existing).withResourceName();
            }
            CompositePropertySource composite = new CompositePropertySource(name);
            composite.addPropertySource(newSource);
            composite.addPropertySource(existing);
            propertySources.replace(name, composite);
         }
         return;
      }
   }

   // 如果 propertySourceNames 为空
   if (this.propertySourceNames.isEmpty()) {
      propertySources.addLast(propertySource);
   }
   else {
      String firstProcessed = this.propertySourceNames.get(this.propertySourceNames.size() - 1);
      propertySources.addBefore(firstProcessed, propertySource);
   }
   this.propertySourceNames.add(name);
}
```

在这段代码中我们可以看到我们操作的对象有两个，一个是 `environment` 中的 `MutablePropertySources` 属性，另一个是 `propertySourceNames`，先来看这两个变量的存储。

-  `propertySourceNames` 中的存储比较简单，存储形式：`List<String>` ，存储内容是配置名称
- `MutablePropertySources` 存储多个 `PropertySource<?>`，具体结构如下

- `MutablePropertySources` 中的存储内容

```java
public class MutablePropertySources implements PropertySources {

   private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();
}
```

`addPropertySource` 的处理逻辑主要是围绕配置名称是否存在于 `propertySourceNames` 中和 配置对象的类型进行处理，下面我们来看处理流程

1. **提取配置名称。**
2. **从环境配置中提取 `MutablePropertySources` 对象。**
3. **配置名称集合中存在当前需要处理的配置名称。**
   1. **从 `MutablePropertySources` 中提取当前配置名称对应的 `PropertySource`  ，命名为`existing`。**
   2. **如果 `existing` 存在。**
      1. **对当前需要处理的 `PropertySource` 进行类型判断并转换，判断是否是 `ResourcePropertySource` 类型。**
      2. **对 `existing` 进行类型判断。**
         1. **类型如果是 `CompositePropertySource`，采用头插法插入数据。**
         2. **其他类型则进行数据替换。**
4. **如果 `propertySourceNames` 为空进行尾插法插入数据。**
5. **如果`propertySourceNames` 不为空进行指定标记后往前插入。**
6. **将配置名称加入到 `propertySourceNames` 中。**



最后我们来看整个方法的处理结果

- `propertySourceNames` 数据信息

![image-20210209113549516](/docs/ch-29/images/image-20210209113549516.png)

- `environment` 数据信息

![image-20210209113612877](/docs/ch-29/images/image-20210209113612877.png)





## 29.6 处理 `@ComponentScans` 和 `@ComponentScan` 注解

接下来我们来看  `@ComponentScans` 和 `@ComponentScan`  注解的处理入口

-   `@ComponentScans` 和 `@ComponentScan`  注解的处理入口

```java
// 处理 ComponentScans ComponentScan 注解
// Process any @ComponentScan annotations
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
if (!componentScans.isEmpty() &&
      !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
   for (AnnotationAttributes componentScan : componentScans) {
      // The config class is annotated with @ComponentScan -> perform the scan immediately
      Set<BeanDefinitionHolder> scannedBeanDefinitions =
            this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
      // Check the set of scanned definitions for any further config classes and parse recursively if needed
      for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
         BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
         if (bdCand == null) {
            bdCand = holder.getBeanDefinition();
         }
         if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
            parse(bdCand.getBeanClassName(), holder.getBeanName());
         }
      }
   }
}
```



通过注释我们了解到了这是对两个注解的处理，下面我们需要来编写这两个注解的使用案例，有了这个使用案例我们就可以更好的进行分析。

首先我们来编写一个需要被扫描到的 Spring Configuration Bean 

```java
@Configuration
public class ScanBeanA {
   @Bean
   public AnnPeople annPeople() {
      AnnPeople annPeople = new AnnPeople();
      annPeople.setName("scanBeanA.people");
      return annPeople;
   }
}
```

其次我们来编写一个测试用例，在这个测试用例中我们希望 `ScanBeanA` 被扫描到

- 测试用例

```java
@ComponentScan(basePackageClasses = ScanBeanA.class)
@Configuration
public class ComponentScanTest {
   @Test
   void scan(){
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ComponentScanTest.class);
      AnnPeople bean = context.getBean(AnnPeople.class);
      assert bean.getName().equals("scanBeanA.people");
   }
}
```

完成这些编辑之后我们来看 `@ComponentScans` 注解和  `@ComponentScan` 注解的关系

- `@ComponentScans` 注解详细信息

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
public @interface ComponentScans {

   ComponentScan[] value();

}
```

- `@ComponentScan` 注解详细信息

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

   @AliasFor("basePackages")
   String[] value() default {};

   @AliasFor("value")
   String[] basePackages() default {};

   Class<?>[] basePackageClasses() default {};

   Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;
    
   Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

   ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

   String resourcePattern() default ClassPathScanningCandidateComponentProvider.DEFAULT_RESOURCE_PATTERN;

   boolean useDefaultFilters() default true;

   Filter[] includeFilters() default {};

   Filter[] excludeFilters() default {};

   boolean lazyInit() default false;

   @Retention(RetentionPolicy.RUNTIME)
   @Target({})
   @interface Filter {

      FilterType type() default FilterType.ANNOTATION;

      @AliasFor("classes")
      Class<?>[] value() default {};
       
      @AliasFor("value")
      Class<?>[] classes() default {};

      String[] pattern() default {};

   }

}
```

可以看到 `@ComponentScans` 注解中的属性是多个 `@ComponentScans` 注解，下面我们来看提取数据的这段代码

- 提取 `@ComponentScans` 和 `@ComponentScan` 数据

```java
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
      sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
```

通过前文对 `@PropertySources` 注解 和 `@PropertySource` 注解的分析，我们可以做出猜测：此时 `AnnotationAttributes` 中的数据是 `ComponentScan` 的属性表，下面我们来看其中的数据情况。

- `componentScans` 数据信息

![image-20210209131512085](/docs/ch-29/images/image-20210209131512085.png)



下面我们来梳理单个`ComponentScan` 的处理过程

1. 第一步：通过  `componentScanParser` 进行解析，解析候得到 `BeanDefinitionHolder`
2. 第二步：将解析得到的 `BeanDefinitionHolder` 根据条件筛选后进行注册

下面我们来看第一步的处理方法 `org.springframework.context.annotation.ComponentScanAnnotationParser#parse`



- `ComponentScanAnnotationParser#parse` 方法详情

```java
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, final String declaringClass) {
   ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry,
         componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);

   Class<? extends BeanNameGenerator> generatorClass = componentScan.getClass("nameGenerator");
   boolean useInheritedGenerator = (BeanNameGenerator.class == generatorClass);
   scanner.setBeanNameGenerator(useInheritedGenerator ? this.beanNameGenerator :
         BeanUtils.instantiateClass(generatorClass));

   ScopedProxyMode scopedProxyMode = componentScan.getEnum("scopedProxy");
   if (scopedProxyMode != ScopedProxyMode.DEFAULT) {
      scanner.setScopedProxyMode(scopedProxyMode);
   }
   else {
      Class<? extends ScopeMetadataResolver> resolverClass = componentScan.getClass("scopeResolver");
      scanner.setScopeMetadataResolver(BeanUtils.instantiateClass(resolverClass));
   }

   scanner.setResourcePattern(componentScan.getString("resourcePattern"));

   for (AnnotationAttributes filter : componentScan.getAnnotationArray("includeFilters")) {
      for (TypeFilter typeFilter : typeFiltersFor(filter)) {
         scanner.addIncludeFilter(typeFilter);
      }
   }
   for (AnnotationAttributes filter : componentScan.getAnnotationArray("excludeFilters")) {
      for (TypeFilter typeFilter : typeFiltersFor(filter)) {
         scanner.addExcludeFilter(typeFilter);
      }
   }

   boolean lazyInit = componentScan.getBoolean("lazyInit");
   if (lazyInit) {
      scanner.getBeanDefinitionDefaults().setLazyInit(true);
   }

   Set<String> basePackages = new LinkedHashSet<>();
   String[] basePackagesArray = componentScan.getStringArray("basePackages");
   for (String pkg : basePackagesArray) {
      String[] tokenized = StringUtils.tokenizeToStringArray(this.environment.resolvePlaceholders(pkg),
            ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
      Collections.addAll(basePackages, tokenized);
   }
   for (Class<?> clazz : componentScan.getClassArray("basePackageClasses")) {
      basePackages.add(ClassUtils.getPackageName(clazz));
   }
   if (basePackages.isEmpty()) {
      basePackages.add(ClassUtils.getPackageName(declaringClass));
   }

   scanner.addExcludeFilter(new AbstractTypeHierarchyTraversingFilter(false, false) {
      @Override
      protected boolean matchClassName(String className) {
         return declaringClass.equals(className);
      }
   });
   return scanner.doScan(StringUtils.toStringArray(basePackages));
}
```

在这个方法中做了两件事，

1. 第一件：创建 `ClassPathBeanDefinitionScanner` 对象并从注解属性中提取数据赋值到对应字段中
2. 第二件：通过 `ClassPathBeanDefinitionScanner` 进行解析

对于 `ClassPathBeanDefinitionScanner` 的分析各位可以翻阅第十七章。





## 29.7 处理 `@Import` 注解

下面我们来看 `@Import` 注解的处理方法



```java
private void processImports(ConfigurationClass configClass, SourceClass currentSourceClass,
      Collection<SourceClass> importCandidates, boolean checkForCircularImports) {

   // 判断是否存在需要处理的 ImportSelector 集合, 如果不需要则不处理直接返回
   if (importCandidates.isEmpty()) {
      return;
   }

   // 判断是否需要进行import循环检查
   // 判断但钱配置类是否在 importStack 中
   if (checkForCircularImports && isChainedImportOnStack(configClass)) {
      this.problemReporter.error(new CircularImportProblem(configClass, this.importStack));
   }
   else {
      // 向 importStack 中加入当前正在处理的配置类
      this.importStack.push(configClass);
      try {
         // 循环处理参数传递过来的 importSelector
         for (SourceClass candidate : importCandidates) {

            // 判断
            if (candidate.isAssignable(ImportSelector.class)) {
               // Candidate class is an ImportSelector -> delegate to it to determine imports
               Class<?> candidateClass = candidate.loadClass();
               // 将 class 转换成对象
               ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
                     this.environment, this.resourceLoader, this.registry);
               // 如果类型是 DeferredImportSelector
               if (selector instanceof DeferredImportSelector) {
                  this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
               }
               else {
                  // 获取需要导入的类
                  String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                  // 将需要导入的类从字符串转换成 SourceClass
                  Collection<SourceClass> importSourceClasses = asSourceClasses(importClassNames);
                  // 递归处理需要导入的类
                  processImports(configClass, currentSourceClass, importSourceClasses, false);
               }
            }

            // 处理类型是 ImportBeanDefinitionRegistrar 的情况
            else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
               // Candidate class is an ImportBeanDefinitionRegistrar ->
               // delegate to it to register additional bean definitions
               Class<?> candidateClass = candidate.loadClass();
               ImportBeanDefinitionRegistrar registrar =
                     ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class,
                           this.environment, this.resourceLoader, this.registry);
               configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
            }
            // 其他情况
            else {
               // Candidate class not an ImportSelector or ImportBeanDefinitionRegistrar ->
               // process it as an @Configuration class
               this.importStack.registerImport(
                     currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
               processConfigurationClass(candidate.asConfigClass(configClass));
            }
         }
      }
      catch (BeanDefinitionStoreException ex) {
         throw ex;
      }
      catch (Throwable ex) {
         throw new BeanDefinitionStoreException(
               "Failed to process import candidates for configuration class [" +
               configClass.getMetadata().getClassName() + "]", ex);
      }
      finally {
         // 从 importStack 删除配置类
         this.importStack.pop();
      }
   }
}
```



下面我们来将这个方法处理流程整理出来

1. **判断参数 `importCandidates` 是否存在数据，如果不存在数据就不进行后续处理。**

2. **判断参数 `checkForCircularImports` 是否为 `true`，同时验证 `importStack` 中是否存在当前参数 `configClass`。**

   **如果 `checkForCircularImports` 为 `true` 同时验证通过就会抛出异常。**

3. **处理 `importCandidates` 集合中的每个 `ImportSelector` 对象，处理方式如下：**

   1. **处理一：类型是 `ImportSelector`**

      **将 `SourceClass` 转换成 `ImportSelector` 对象，得到 `ImportSelector` 后分两种情况处理**

      1. **第一种：类型是 `DeferredImportSelector`**

         **重复 `handler` 的操作**

      2. **第二种：类型不是 `DeferredImportSelector`**

         **调用 `ImportSelector#selectImports` 方法得到需要导入的类，将类名转换成 `SourceClass` ，最后解析每个 `SourceClass` 处理方法是本方法(`processImports`) ，递归调用。**

   2. **处理二：类型是 `ImportBeanDefinitionRegistrar`**

      **将 `SourceClass` 转换成 `ImportBeanDefinitionRegistrar` 实例，将这个实例放入 `importBeanDefinitionRegistrars` 容器。**

   3. **处理三：类型既不是 `ImportSelector` 也不是 `ImportBeanDefinitionRegistrar`**



这部分流程在第二十七章中有所提及，第二十七章介绍了 `ImportSelector` 相关处理，本节为表层分析，关于 `ImportSelector` 处理细节需要详细了解需要查看第二十七章，比如 `handler` 操作









## 29.8 处理 `@ImportResource` 注解

接下来我们来看 `@ImportResource` 注解的处理

```java
// Process any @ImportResource annotations
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

这段代码的处理是将注解元数据中关于 `ImportResource` 的属性，得到属性后将其转换成 `importedResources` 中的数据

- `importedResources` 存储结构

```java
private final Map<String, Class<? extends BeanDefinitionReader>> importedResources =
      new LinkedHashMap<>();
```

存储结构说明：

key：资源文件地址。

value：Bean Definition Reader 。



有关 `@ImportResource`  后续的解析各位可以参考第二十八章，在这里的处理仅仅只是提取数据。





## 29.9 处理 `@Bean` 注解

接下来我们来看 `@Bean` 注解的处理，先来看代码

- `Bean` 注解处理代码

```java
// 处理 Bean 注解
// Process individual @Bean methods
Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
for (MethodMetadata methodMetadata : beanMethods) {
   configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
}

// 处理 Bean Method
// Process default methods on interfaces
processInterfaces(configClass, sourceClass);
```

这里我们先不进入 `retrieveBeanMethodMetadata` 方法，我们先来看 `processInterfaces` 方法中的处理。



```java
private void processInterfaces(ConfigurationClass configClass, SourceClass sourceClass) throws IOException {
   // 获取接口
   for (SourceClass ifc : sourceClass.getInterfaces()) {
      // 接口上获取方法元数据
      Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(ifc);
      for (MethodMetadata methodMetadata : beanMethods) {
         if (!methodMetadata.isAbstract()) {
            // A default method or other concrete method on a Java 8+ interface...
            configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
         }
      }
      processInterfaces(configClass, ifc);
   }
}
```

在这段代码中我们看到了一个类似的操作代码，这段操作代码就是在该方法调用前进行的一个操作

- 类似的操作代码

```java
Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
for (MethodMetadata methodMetadata : beanMethods) {
   configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
}
```

现在我们可以归纳出这两个处理方法的大致流程

1. 第一步：提取某个类的方法元信息
2. 第二步：放入到存储Bean Method 的容器中。

下面我们来看第一步的处理操作

- `retrieveBeanMethodMetadata` 方法详情

```java
private Set<MethodMetadata> retrieveBeanMethodMetadata(SourceClass sourceClass) {
   // 提取元数据
   AnnotationMetadata original = sourceClass.getMetadata();
   // 提取带有 Bean 标签的数据
   Set<MethodMetadata> beanMethods = original.getAnnotatedMethods(Bean.class.getName());

   if (beanMethods.size() > 1 && original instanceof StandardAnnotationMetadata) {
      // Try reading the class file via ASM for deterministic declaration order...
      // Unfortunately, the JVM's standard reflection returns methods in arbitrary
      // order, even between different runs of the same application on the same JVM.
      try {
         // 类的注解元数据
         AnnotationMetadata asm =
               this.metadataReaderFactory.getMetadataReader(original.getClassName()).getAnnotationMetadata();
         // 提取带有 Bean 注解的方法元数据
         Set<MethodMetadata> asmMethods = asm.getAnnotatedMethods(Bean.class.getName());
         if (asmMethods.size() >= beanMethods.size()) {

            // 选中的方法元数据
            Set<MethodMetadata> selectedMethods = new LinkedHashSet<>(asmMethods.size());
            // 循环两个方法元数据进行 methodName 比较如果相同会被加入到选中集合中
            for (MethodMetadata asmMethod : asmMethods) {
               for (MethodMetadata beanMethod : beanMethods) {
                  if (beanMethod.getMethodName().equals(asmMethod.getMethodName())) {
                     selectedMethods.add(beanMethod);
                     break;
                  }
               }
            }
            if (selectedMethods.size() == beanMethods.size()) {
               // All reflection-detected methods found in ASM method set -> proceed
               beanMethods = selectedMethods;
            }
         }
      }
      catch (IOException ex) {
         logger.debug("Failed to read class file via ASM for determining @Bean method order", ex);
         // No worries, let's continue with the reflection metadata we started with...
      }
   }
   return beanMethods;
}
```

阅读了这个方法后我们来整理其中的处理细节。

1. **步骤一：提取类当中的注解元数据。**

2. **步骤二：提取带有 `Bean` 注解的数据，这里提取的都是方法上的注解数据，变量名称为 `beanMethods`。**

3. **步骤三：返回值处理**

   1. **情况一：当步骤二得到的数据数量大于1并且类的元数据类型是 `StandardAnnotationMetadata` 进行下面的处理**

      1. **提取注解元数据。**

      2. **提取注解全数据中方法被 Bean  注解标记的方法元数据 ，变量名称为`asmMethods`。**

      3. **方法元数据筛选，通过筛选条件的加入到候选集合中。**

         **筛选条件：对比 `asmMethods` 中的数据和 `beanMethods` 中的数据方法名称是否相同，相同就是符合条件的。**

   2. **情况二：不符合情况一直接返回步骤二得到的数据。**





下面我们来编写这段代码的测试用例

首先我们需要编写一个接口编写这个接口的目的是为了将 `processInterfaces` 中的代码覆盖到

- 一个接口

```java
public interface InterA {
   @Bean
   default AnnPeople annp() {
      AnnPeople annPeople = new AnnPeople();
      annPeople.setName("InterPeop");
      return annPeople;
   }

}
```

接下来我们将这个接口的实现类做出来

```java
@Configuration
public class InterAImpl implements InterA {

	@Bean
	public AnnPeople annPeople() {
		return new AnnPeople();
	}

	@Bean
	public AnnPeople annPeople2() {
		AnnPeople annPeople = new AnnPeople();
		annPeople.setName("InterPeop2");
		return annPeople;
	}

}

```

在这个实现类中我们再定义一个 Bean 这里是为了覆盖外层的解析，写出两个 Bean 是为了测试 `beanMethods.size() > 1 && original instanceof StandardAnnotationMetadata` 条件



最后我们编写测试方法

```java
@Test
void testInterface(){
   AnnotationConfigApplicationContext context
         = new AnnotationConfigApplicationContext(InterAImpl.class);

   InterA bean = context.getBean(InterA.class);

}
```

下面我们来看调试阶段的一些数据信息

- `beanMethods` 的数据，（非`processInterfaces` 处理)

![image-20210209151622022](/docs/ch-29/images/image-20210209151622022.png)

这这里我们可以看到三个，这里最后一个是从接口 `default` 中继承过来的，如果不需要请将代码修改成下面的样子



```java
public interface InterA {
  	@Bean
    AnnPeople annp();
}
```

但是这样编写就不会得到这个 Bean 它不符合 `processInterfaces` 中 `!methodMetadata.isAbstract()` 条件，使用 `default` 符合。为了做到一个完整的测试将代码进行修改变成如下内容

- `InterA` 详细信息

```java
public interface InterA {
	@Bean
	default AnnPeople annpc() {
		AnnPeople annPeople = new AnnPeople();
		annPeople.setName("interface bean 1");
		return annPeople;
	}


	@Bean
	abstract AnnPeople annp();
}
```

- `InterAImpl` 详细信息

```java
@Configuration
public class InterAImpl implements InterA {

	@Bean
	public AnnPeople annPeople() {
		AnnPeople annPeople = new AnnPeople();
		annPeople.setName("bean1");
		return annPeople;
	}

	@Bean
	public AnnPeople annPeople2() {
		AnnPeople annPeople = new AnnPeople();
		annPeople.setName("bean2");
		return annPeople;
	}

	public AnnPeople annp() {
		AnnPeople annPeople = new AnnPeople();
		annPeople.setName("interface implements bean");
		return annPeople;
	}
}
```



最后我们来看解析后的 BeanMethod 

- Bean Methods 数据信息

![image-20210209153352278](/docs/ch-29/images/image-20210209153352278.png)



最后我们来看看容器中的数据，通过 `context.getBeansOfType(AnnPeople.class)` 进行查询

![image-20210209153027228](/docs/ch-29/images/image-20210209153027228.png)









## 29.10 处理父类配置

最后我们来看父类配置的处理，先来看代码

- 处理父类配置部分的代码详情

```java
if (sourceClass.getMetadata().hasSuperClass()) {
   String superclass = sourceClass.getMetadata().getSuperClassName();
   if (superclass != null && !superclass.startsWith("java") &&
         !this.knownSuperclasses.containsKey(superclass)) {
      this.knownSuperclasses.put(superclass, configClass);
      // Superclass found, return its annotation metadata and recurse
      return sourceClass.getSuperClass();
   }
}
```

在这段代码中主要目的是向 `knownSuperclasses` 插入数据，插入数据的前提是**当前正在处理的类存在父类，同时父类名称是java开头同时`knownSuperclasses`不存在**

下面我们来编写这部分代码的测试用例

- 父类配置

```java
@Configuration
public class SupperConfiguration {

}
```

- 子类配置

```java
@Configuration
public class ExtendConfiguration extends SupperConfiguration {

}
```

- 应用上下文

```java
@Configuration
@Import(ExtendConfiguration.class)
public class ThreeVarBeans {
}
```

- 测试方法

```java
@Test
void testThreeVar() {
   AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ThreeVarBeans.class);
}
```





- `knownSuperclasses` 数据信息

![image-20210209154454141](/docs/ch-29/images/image-20210209154454141.png)









## 29.11 总结

在本章笔者和各位围绕 Spring 配置类解析做了一个详细的分析，在这一章节中我们知道了一些常规注解在配置类中的解析处理形式。