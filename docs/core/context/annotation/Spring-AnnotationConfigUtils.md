# Spring AnnotationConfigUtils
- 类全路径: `org.springframework.context.annotation.AnnotationConfigUtils`





## registerAnnotationConfigProcessors

- 方法签名: `org.springframework.context.annotation.AnnotationConfigUtils#registerAnnotationConfigProcessors(org.springframework.beans.factory.support.BeanDefinitionRegistry, java.lang.Object)`

- 方法作用: 注册注解配置相关的处理器
    1. ConfigurationClassPostProcessor
    1. AutowiredAnnotationBeanPostProcessor
    1. CommonAnnotationBeanPostProcessor
    1. PersistenceAnnotationBeanPostProcessor
    1. EventListenerMethodProcessor
    1. DefaultEventListenerFactory
    


<details>
<summary>registerAnnotationConfigProcessors 方法详情</summary>

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, @Nullable Object source) {

		// 获取 BeanFactory
		DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
		if (beanFactory != null) {
			// 获取依赖比较器
			// 判断依赖比较器 是否是 AnnotationAwareOrderComparator
			if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
				beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
			}
			// 获取 自动装配解析器
			// 自动装配解析器 是否是 ContextAnnotationAutowireCandidateResolver
			if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
				beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
			}
		}

		// bean定义持有器列表
		Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

		// 注册下面这些 PostProcessor
		// ConfigurationClassPostProcessor
		// AutowiredAnnotationBeanPostProcessor
		// CommonAnnotationBeanPostProcessor
		// PersistenceAnnotationBeanPostProcessor
		// EventListenerMethodProcessor
		// DefaultEventListenerFactory
		if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
		if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
		if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition();
			try {
				def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
						AnnotationConfigUtils.class.getClassLoader()));
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
			}
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
		}

		if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
			RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
			def.setSource(source);
			beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
		}

		return beanDefs;
	}
```


</details>

## processCommonDefinitionAnnotations
- 方法签名: `org.springframework.context.annotation.AnnotationConfigUtils.processCommonDefinitionAnnotations(org.springframework.beans.factory.annotation.AnnotatedBeanDefinition, org.springframework.core.type.AnnotatedTypeMetadata)`
- 方法作用: 设置beanDefinition的通用属性
    1. Lazy
    1. Primary
    1. DependsOn
    1. Role
    1. Description


<details>
<summary>processCommonDefinitionAnnotations 方法详情</summary>

```java
	static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd, AnnotatedTypeMetadata metadata) {
		AnnotationAttributes lazy = attributesFor(metadata, Lazy.class);
		if (lazy != null) {
			abd.setLazyInit(lazy.getBoolean("value"));
		}
		else if (abd.getMetadata() != metadata) {
			lazy = attributesFor(abd.getMetadata(), Lazy.class);
			if (lazy != null) {
				abd.setLazyInit(lazy.getBoolean("value"));
			}
		}

		if (metadata.isAnnotated(Primary.class.getName())) {
			abd.setPrimary(true);
		}
		AnnotationAttributes dependsOn = attributesFor(metadata, DependsOn.class);
		if (dependsOn != null) {
			abd.setDependsOn(dependsOn.getStringArray("value"));
		}

		AnnotationAttributes role = attributesFor(metadata, Role.class);
		if (role != null) {
			abd.setRole(role.getNumber("value").intValue());
		}
		AnnotationAttributes description = attributesFor(metadata, Description.class);
		if (description != null) {
			abd.setDescription(description.getString("value"));
		}
	}

```
</details>