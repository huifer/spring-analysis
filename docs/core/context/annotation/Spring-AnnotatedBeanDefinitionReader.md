# Spring AnnotatedBeanDefinitionReader
- 类全路径: `org.springframework.context.annotation.AnnotatedBeanDefinitionReader`

- 类作用: 注解版本的beanDefinition阅读器





## 成员变量



```java
public class AnnotatedBeanDefinitionReader {
   /**
    * beanDefinition 注册器
    */
   private final BeanDefinitionRegistry registry;

   /**
    * beanName 生成器
    */
   private BeanNameGenerator beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;

   /**
    * bean 作用域解析器
    */
   private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();

   /**
    * 条件解析器 , 相关接口: {@link Conditional}
    */
   private ConditionEvaluator conditionEvaluator;
}
```







## 方法分析

### 构造函数



```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
   Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
   Assert.notNull(environment, "Environment must not be null");
   this.registry = registry;
   this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
   // 注册注解的后置处理器
   AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

- 构造函数处理成员变量赋值外还做了关于后置处理器的注册(倾向注解方向)

  
    - ConfigurationClassPostProcessor
    - AutowiredAnnotationBeanPostProcessor
    - CommonAnnotationBeanPostProcessor
    - PersistenceAnnotationBeanPostProcessor
    - EventListenerMethodProcessor
    - DefaultEventListenerFactory



### doRegisterBean

- 方法签名: `org.springframework.context.annotation.AnnotatedBeanDefinitionReader#doRegisterBean`
- 方法作用: 注册 bean



- 执行流程
	- 创建 AnnotatedGenericBeanDefinition
	- 条件表达式处理
	- 设置 Scope 属性
	- beanName 处理
		- 生成
		- 获取
	- 通用注解的处理
		- 处理了那些注解
			- Lazy
			- Primary
			- DependsOn
			- Role
			- Description
		- 处理了通用租界后会设置到 BeanDefinition 中
	- qualifiers 处理
	- BeanDefinitionHolder注册



```java
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
      @Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
      @Nullable BeanDefinitionCustomizer[] customizers) {

   // 泛型bean定义
   AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
   // 和条件注解相关的函数
   if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
      return;
   }

   // 设置实例提供者
   abd.setInstanceSupplier(supplier);
   // 解析 注解的 beanDefinition 的作用域元数据
   ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
   // 设置 作用域元数据
   abd.setScope(scopeMetadata.getScopeName());
   // beanName 处理
   String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

   // 通用注解的处理
   AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
   if (qualifiers != null) {
      for (Class<? extends Annotation> qualifier : qualifiers) {
         if (Primary.class == qualifier) {
            abd.setPrimary(true);
         }
         else if (Lazy.class == qualifier) {
            abd.setLazyInit(true);
         }
         else {
            abd.addQualifier(new AutowireCandidateQualifier(qualifier));
         }
      }
   }
   if (customizers != null) {
      for (BeanDefinitionCustomizer customizer : customizers) {
         customizer.customize(abd);
      }
   }

   // 创建 beanDefinition Holder 后进行注册
   BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
   // 应用作用域代理
   definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
   BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```