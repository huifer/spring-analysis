# Spring ClassPathBeanDefinitionScanner
- 类全路径: `org.springframework.context.annotation.ClassPathBeanDefinitionScanner`

- 类作用: 扫描输入的 包路径进行组件注册





## 成员变量

<details>
    <summary>成员变量</summary>





```java
public class ClassPathBeanDefinitionScanner extends ClassPathScanningCandidateComponentProvider {

    /**
     * bean定义注册器
     */
    private final BeanDefinitionRegistry registry;

    /**
     * bean 定义的默认值
     */
    private BeanDefinitionDefaults beanDefinitionDefaults = new BeanDefinitionDefaults();

    /**
     * 候选对象名称列表(名称匹配模式)
     */
    @Nullable
    private String[] autowireCandidatePatterns;

    /**
     * BeanName 生成器
     */
    private BeanNameGenerator beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;

    /**
     * 作用域解析器(解析作用域元数据)
     */
    private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();

    /**
     * 是否包含注解的配置信息
     */
    private boolean includeAnnotationConfig = true;
}
```

</details>



1. BeanDefinitionRegistry: bean定义注册器,[分析文章](/docs/beans/factory/BeanFactory/support/BeanDefinitionRegistry/readme.md)
2. BeanDefinitionDefaults: bean定义默认值
3. BeanNameGenerator: beanName 生成器,[分析文章](/docs/beans/factory/BeanFactory/support/Spring-BeanNameGenerator.md)
4. ScopeMetadataResolver: scope 元数据解析接口





## 方法分析



### scan

- 方法签名: `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#scan`
- 方法作用: 扫描指定的包路径, 注册组件，得到注册组件的数量





```java
public int scan(String... basePackages) {
   // 在执行扫描方法前beanDefinition的数量
   int beanCountAtScanStart = this.registry.getBeanDefinitionCount();

   // 真正的扫描方法
   doScan(basePackages);

   // 是否需要注册 注解的配置处理器
   // Register annotation config processors, if necessary.
   if (this.includeAnnotationConfig) {
      // 注册注解后置处理器
      AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
   }

   //  当前 BeanDefinition 数量 - 历史 B
   return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);
}
```

- 方法的核心在`doScan` 方法中





### doScan

- 方法签名: `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan`

- 处理逻辑
  1. 搜索可能的组件(`findCandidateComponents`), 搜索方法在[ClassPathScanningCandidateComponentProvider](/docs/core/context/annotation/Spring-ClassPathScanningCandidateComponentProvider-未完成.md)中
  2. 候选bean定义的处理
     1. 设置 scope 属性`setScope` [分析文章](/docs/core/context/annotation/ScopeMetadataResolver/readme.md)
     2. 类型是 AbstractBeanDefinition 的处理`postProcessBeanDefinition`
     3. 类型是 AnnotatedBeanDefinition 的处理`AnnotationConfigUtils.processCommonDefinitionAnnotations`，[AnnotationConfigUtils](/docs/core/context/annotation/Spring-AnnotationConfigUtils.md)
     4. 候选检测`checkCandidate`
        1. 成功后放入容器`registerBeanDefinition`







<details>
    <summary>doScan详细方法</summary>





```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
   Assert.notEmpty(basePackages, "At least one base package must be specified");
   // bean 定义持有器列表
   Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
   // 循环包路径进行扫描
   for (String basePackage : basePackages) {
      // 搜索可能的组件. 得到 组件的BeanDefinition
      Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
      // 循环候选bean定义
      for (BeanDefinition candidate : candidates) {
         // 获取 作用域元数据
         ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
         // 设置作用域
         candidate.setScope(scopeMetadata.getScopeName());
         // beanName 生成
         String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
         // 类型判断 AbstractBeanDefinition
         if (candidate instanceof AbstractBeanDefinition) {
            // bean 定义的后置处理
            postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
         }
         // 类型判断 AnnotatedBeanDefinition
         if (candidate instanceof AnnotatedBeanDefinition) {
            // 通用注解的处理
            AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
         }
         // 候选检测
         if (checkCandidate(beanName, candidate)) {
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            // 作用于属性应用
            definitionHolder =
                  AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            beanDefinitions.add(definitionHolder);
            // 注册 bean定义
            registerBeanDefinition(definitionHolder, this.registry);
         }
      }
   }
   return beanDefinitions;
}
```

</details>





### postProcessBeanDefinition

- 方法签名: `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#postProcessBeanDefinition`

- 方法作用: 设置默认值信息



```java
	protected void postProcessBeanDefinition(AbstractBeanDefinition beanDefinition, String beanName) {
		// 应用默认值
		beanDefinition.applyDefaults(this.beanDefinitionDefaults);
		if (this.autowireCandidatePatterns != null) {
			// 设置 autowireCandidate 模式
			// AUTOWIRE_BY_TYPE 或者 AUTOWIRE_BY_NAME 
			beanDefinition.setAutowireCandidate(PatternMatchUtils.simpleMatch(this.autowireCandidatePatterns, beanName));
		}
	}

```







### checkCandidate

- 方法签名: `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#checkCandidate`



主线流程如下

1. 判断当前容器中是否含有beanName对应的实例
2. `existingDef` 或者 `originatingDef` 是否对参数 `beanDefinition` 兼容
   1. 兼容的判断		
      1.  是否是 ScannedGenericBeanDefinition 类型
      2.  source 是否相同
      3.  参数是否相同



```java
protected boolean checkCandidate(String beanName, BeanDefinition beanDefinition) throws IllegalStateException {
   // 当前注册器中是否包含 beanName
   if (!this.registry.containsBeanDefinition(beanName)) {
      return true;
   }
   // 注册器中的 beanName 的 beanInstance
   BeanDefinition existingDef = this.registry.getBeanDefinition(beanName);
   BeanDefinition originatingDef = existingDef.getOriginatingBeanDefinition();
   if (originatingDef != null) {
      existingDef = originatingDef;
   }
   // 两个对象是否兼容
   if (isCompatible(beanDefinition, existingDef)) {
      return false;
   }
   throw new ConflictingBeanDefinitionException("Annotation-specified bean name '" + beanName +
         "' for bean class [" + beanDefinition.getBeanClassName() + "] conflicts with existing, " +
         "non-compatible bean definition of same name and class [" + existingDef.getBeanClassName() + "]");
}
```

### isCompatible

- 方法签名: `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#isCompatible`

```java
protected boolean isCompatible(BeanDefinition newDefinition, BeanDefinition existingDefinition) {
   // 1. 是否是 ScannedGenericBeanDefinition 类型
   // 2. source 是否相同
   // 3. 参数是否相同
   return (!(existingDefinition instanceof ScannedGenericBeanDefinition) ||  // explicitly registered overriding bean
         (newDefinition.getSource() != null && newDefinition.getSource().equals(existingDefinition.getSource())) ||  // scanned same file twice
         newDefinition.equals(existingDefinition));  // scanned equivalent class twice
}
```