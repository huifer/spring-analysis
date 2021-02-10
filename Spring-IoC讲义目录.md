序章

第一章 容器环境搭建及基本使用
- 1.1 容器环境搭建
- 1.1.1 工程创建
- 1.1.2 基本代码编辑
- 1.2 容器功能

第二章 IoC 中的核心类

第三章 IoC 资源读取及注册
- 3.1 XML 文档验证
- 3.1.1 认识 XML 验证模式
- 3.1.2 Spring 中 XML 的验证
- 3.2 Document 对象获取
- 3.3 BeanDefinition 注册
- 3.3.1. doRegisterBeanDefinitions 流程
- 3.3.2 parseBeanDefinitions 分析
- 3.3.3 parseDefaultElement Spring 原生标签的处理
- 3.3.3.4 import 标签解析
- 3.3.4.1 import 标签解析的环境搭建
- 3.3.4.2 import 标签的定义
- 3.3.4.3 import 标签解析第一部分: 处理 resource 属性
- 3.3.4.4 import 标签解析的第二部分和第三部分 重回 loadBeanDefinitions
- 3.3.4.5 import 标签解析的第四部分 import 事件处理
- 3.3.5 标签解析
- 3.3.5.1 alias 标签解析的环境搭建
- 3.3.5.2 alias 标签的定义及含义
- 3.3.5.3 processAliasRegistration 方法分析
- 3.3.5.4 别名注册
- 3.3.5.5 别名事件处理
- 3.3.6 bean 标签解析
- 3.3.6.1 bean 标签解析的环境搭建
- 3.3.6.2 bean 标签的定义
- 3.3.6.3 processBeanDefinition 方法解析
- 3.3.6.4 bean 定义注册
- 3.3.6.5 bean 注册事件
- 3.3.7 自定义标签解析
- 3.4 总结
  
第四章 自定义标签解析
- 4.1 创建自定义标签环境搭建
- 4.1.1 编写 XSD 文件
- 4.1.2 编写 NamespaceHandler 实现类
- 4.1.3 编写 BeanDefinitionParser 实现类
- 4.1.4 编写注册方式
- 4.1.5 测试用例的编写
- 4.1.5.1 Spring xml 文件编写
- 4.1.5.2 测试用例
- 4.2 自定义标签解析
- 4.2.1 寻找自定义标签、NamesapceHandler 和 BeanDefinitionParser 之间的关系
- 4.2.2 获取命名空间地址
- 4.2.3 NamespaceHandler 对象获取
- 4.2.4 getHandlerMappings 获取命名空间的映射关系
- 4.2.5 NamespaceHandler 的获取
- 4.2.6 NamespaceHandler 的 init 方法
- 4.2.7 NamespaceHandler 缓存的刷新
- 4.2.8 解析标签 - BeanDefinitionParser 对象准备
- 4.2.9 解析标签 - parse 方法调用
- 4.3 总结

第五章 别名注册
- 5.1 环境搭建
- 5.2 别名注册接口
- 5.3 SimpleAliasRegistry 中注册别名的实现、
- 5.4 别名换算真名
- 5.5 总结

第六章 bean 标签解析
- 6.1 创建 bean 标签环境
- 6.2 parseBeanDefinitionElement 第一部分 id 和 name 的处理
- 6.2 parseBeanDefinitionElement 第二部分 关于 Bean Name 处理
- 6.3 parseBeanDefinitionElement 处理 bean 标签
- 6.3.1 处理 class name 和 parent
- 6.3.2 创建 AbstractBeanDefinition
- 6.3.3 parseBeanDefinitionAttributes 设置 Bean Definition 的基本信息
- 6.3.4 Bean Definition 描述设置
- 6.3.5 Meta 属性设置
- 6.3.6 lookup-override 标签处理
- 6.3.7 replaced-method 标签处理
- 6.3.8 constructor-arg 标签处理
- 6.3.9 property 标签处理
- 6.3.10 qualifier 标签处理
- 6.3.11 Bean Definition 的最后两个属性
- 6.4 Bean Definition 装饰
- 6.5 Bean Definition 细节
- 6.5.1 AbstractBeanDefinition 的属性
- 6.5.2 RootBeanDefinition 的属性
- 6.5.3 ChildBeanDefinition 的属性
- 6.5.4 GenericBeanDefinition 的属性
- 6.5.5 AnnotatedGenericBeanDefinition 的属性
- 6.6 总结

第七章 Spring Bean Definition 注册
- 7.1 Bean Definition 注册
- 7.2 DefaultListableBeanFactory 中存储 Bean Definition 的容器
- 7.3 DefaultListableBeanFactory 中的注册细节
- 7.3.1 Bean Definition 的验证
- 7.3.2 容器中存在 Bean Name 对应的 Bean Definition 的处理
- 7.3.3 容器中不存在 Bean Name 对应的 Bean Definition 的处理
- 7.3.4 Bean Definition 的刷新处理
- 7.4 Bean Definition 的获取
- 7.5 总结

第八章 Bean 的生命周期
- 8.1 Bean 的生命周期
- 8.1.1 Java 对象的生命周期
- 8.1.2 浅看 Bean 生命周期
- 8.1.3 初始化 Bean
- 8.1.3.1 无构造标签
- 8.1.3.2 构造标签配合 index 模式 和 name 模式
- 8.1.3.4 Spring 中实例化策略
- 8.1.4 Bean 属性设置
- 8.1.4.1 Bean Wrapper 创建
- 8.1.4.2 Bean Wrapper 设置属性
- 8.1.4.2.1 CachedIntrospectionResults 介绍
- 8.1.4.2.2 PropertyValue 介绍
- 8.1.4.2.3 BeanPropertyHandler#setValue 最终的属性设置
- 8.1.5 Bean生命周期 Aware
- 8.1.6 BeanPostProcessor#postProcessBeforeInitialization 方法
- 8.1.7 InitializingBean 和 自定义 init-method 方法
- 8.1.7.1 invokeCustomInitMethod 分析
- 8.1.8 BeanPostProcessor#postProcessAfterInitialization 方法
- 8.1.9 Bean 的摧毁
- 8.1.9.1 DefaultSingletonBeanRegistry 中的摧毁
- 8.1.9.2 DefaultListableBeanFactory 中的摧毁
- 8.2 总结

第九章 Bean 的获取
- 9.1 Bean 获取的方式
- 9.2 测试用例准备
- 9.3 doGetBean 分析
- 9.3.1 Bean Name 转换
- 9.3.2 单例容器中尝试获取
- 9.3.3 获取 Bean 实例( FactoryBean )
- 9.3.3.1 FactoryBean 的单例创建
- 9.3.3.1.1 doGetObjectFromFactoryBean
- 9.3.3.1.2 beforeSingletonCreation
- 9.3.3.1.3 postProcessObjectFromFactoryBean
- 9.3.3.1.4 afterSingletonCreation
- 9.3.3.2 FactoryBean 的原型创建
- 9.3.4 从父容器中尝试获取
- 9.3.5 Bean Name 标记
- 9.3.6 非 FactoryBean 创建，单例模式
- 9.3.6.1 进行非 FactoryBean 创建之前的通用准备
- 9.3.6.1.1 BeanDefinition 合并
- 9.3.6.1.2 验证 Bean Definition 的合并结果
- 9.3.6.1.3 MergeBeanDefinition 依赖处理
- 9.3.6.2 doCreateBean 方法分析 第一部分-获取实例
- 9.3.6.2.1 Supplier 中创建 Bean
- 9.3.6.2.2 FactoryBean + FactoryMethod 创建 Bean
- 9.3.6.2.3 autowireConstructor 创建 Bean
- 9.3.6.2.3.1 参数列表的确认
- 9.3.6.2.3.2 构造函数的确认
- 9.3.6.2.4 instantiateBean 创建 Bean
- 9.3.6.2.4.1 非 Cglib 模式的 Bean
- 9.3.6.2.4.2 Cglib 模式的 Bean
- 9.3.6.2.5 createBeanInstance 小结
- 9.3.6.3 doCreateBean 方法分析 第二部分-后置方法调用
- 9.3.6.4 doCreateBean 方法分析 第三部分-早期单例 Bean 的暴露
- 9.3.6.5.1 提前暴露的确认
- 9.3.6.5.2 getEarlyBeanReference 分析
- 9.3.6.5.3 addSingletonFactory 分析
- 9.3.6.5 doCreateBean 方法分析 第四部分-属性设置、InitializingBean 和 init-method 调用
- 9.3.6.6 doCreateBean 方法分析 第五部分-依赖处理
- 9.3.6.7 doCreateBean 方法分析 第六部分-注册摧毁 Bean 信息
- 9.3.6.8 getSingleton 分析
- 9.3.6.8.1 beforeSingletonCreation 创建之前的验证
- 9.3.6.8.2 afterSingletonCreation 创建之后的验证
- 9.3.6.8.1 addSingleton 添加到单例容器中
- 9.3.7 非 FactoryBean 创建，原型模式
- 9.3.8 非 FactoryBean 创建 并且非单例非原型模式的创建
- 9.3.9 类型转换中获取 Bean 实例
- 9.3.10 doGetBean 小结
- 9.4 循环依赖
- 9.4.1 Java 中的循环依赖
- 9.4.2 Spring 中循环依赖处理
- 9.5 总结

第十章 XML 配置下的容器生命周期
- 10.1 基础环境搭建
- 10.2 XmlBeanFactory 分析
- 10.2.1 XmlBeanFactory 构造函数
- 10.2.1.1 ignoreDependencyInterface 添加忽略的依赖接口
- 10.2.1.2 setParentBeanFactory 设置父容器
- 10.2.2 XmlBeanFactory 总结
- 10.3 FileSystemXmlApplicationContext 分析
- 10.3.1 构造方法分析
- 10.3.1.1 父上下文的处理
- 10.3.1.2 配置地址的处理
- 10.3.1.3 刷新操作
- 10.3.1.3.1 prepareRefresh 刷新前的准备工作
- 10.3.1.3.2 obtainFreshBeanFactory 创建 Bean Factory
- 10.3.1.3.2.1 refreshBeanFactory 方法分析
- 10.3.1.3.2.2 getBeanFactory 方法分析
- 10.3.1.3.3 prepareBeanFactory 方法分析
- 10.3.1.3.4 postProcessBeanFactory 分析
- 10.3.1.3.5 invokeBeanFactoryPostProcessors 分析
- 10.3.1.3.6 registerBeanPostProcessors 分析
- 10.3.1.3.7 initMessageSource 分析
- 10.3.1.3.8 initApplicationEventMulticaster 分析
- 10.3.1.3.9 onRefresh 分析
- 10.3.1.3.10 registerListeners 分析
- 10.3.1.3.11 finishBeanFactoryInitialization 分析
- 10.3.1.3.12 finishRefresh 分析
- 10.3.1.3.13 destroyBeans 分析
- 10.3.1.3.14 cancelRefresh 分析
- 10.3.1.3.15 resetCommonCaches 分析
- 10.3.2 关闭方法分析
- 10.3.2.1 doClose 分析
- 10.4 ClassPathXmlApplicationContext 分析
- 10.5 总结
- 10.6 附表
- AbstractAutowireCapableBeanFactory 成员变量
- AbstractApplicationContext 成员变量
- DefaultListableBeanFactory 成员变量


第十一章 Spring 中的转换服务
- 11.1 初识 Spring 转换服务
- 11.2 ConversionServiceFactoryBean 的实例化
- 11.2.1 afterPropertiesSet 细节分析
- 11.2.1.1 GenericConversionService 的创建
- 11.2.1.2 ConverterRegistry 转换服务注册
- 11.2.1.2.1 转换器注册方式1 - Converter 直接注册
- 11.2.1.2.2 转换器注册方式2 - 原始类型+目标类型+ Converter 注册
- 11.2.1.2.3 转换器注册方式3 - GenericConverter 注册
- 11.2.1.2.3.1 添加 GenericConverter
- 11.2.1.2.3.2 缓存处理
- 11.2.1.2.4 转换器注册方式4 - ConverterFactory 注册
- 11.2.1.3 ConversionServiceFactory.registerConverters 分析
- 11.3 转换过程分析
- 11.3.1 ConversionService 是谁
- 11.3.2 转换方法分析
- 11.3.2.1 handleResult 分析
- 11.3.2.2 getConverter 获取转换接口
- 11.3.2.2.1 converters.find 分析
- 11.3.2.2.2 getDefaultConverter 默认的转换服务
- 11.3.2.3 ConversionUtils.invokeConverter 分析
- 11.3.2.3.1 ConverterAdapter 的转换
- 11.3.2.3.2 ConverterFactoryAdapter 的转换
- 11.3.2.4 handleConverterNotFound 分析
- 11.4 脱离 Spring 的实现
- 11.5 总结

第十二章 占位符解析
- 12.1 基本环节搭建
- 12.2 XML 的解析
- 12.3 外部配置的读取
- 12.4 解析占位符
- 12.4.1 resolveStringValue 分析
- 12.4.2 resolvePlaceholders 分析
- 12.4.2.1 没有占位符的分析
- 12.4.2.2 只有一对占位符的分析
- 12.4.2.3 平级占位符的分析
- 12.4.2.4 嵌套占位符的分析
- 12.4.2.5 占位符解析小结
- 12.4.3 resolveRequiredPlaceholders 分析
- BeanDefinitionVisitor#visitBeanDefinition 分析
- 12.5 总结

第十三章 Message Source
- 13.1 基本环境搭建
- 13.2 MessageSource 的实例化
- 13.3 getMessage 分析
- 13.3.1 resolveCodeWithoutArguments 分析
- 13.3.2 resolveCode 分析
- 13.4 总结

第十四章 PostProcessorRegistrationDelegate
- 14.1 BeanPostProcessor 注册
- 14.2 BeanFactoryPostProcessor 方法调用
- 14.3 总结

第十五章 Spring 事件
- 15.1 环境搭建
- 15.2 事件处理器注册
- 15.2.1 ApplicationListenerDetector 分析
- 15.2.1.1 实例创建之后做什么
- 15.2.1.2 实例摧毁之前做什么
- 15.3 事件推送和处理
- 15.4 总结

第十六章 LifecycleProcessor
- 16.1 初识 LifecycleProcessor
- 16.2 测试环境搭建
- 16.3 start 分析
- 16.4 stop 分析
- 16.5 关键类
- 16.5.1 LifecycleGroup 成员变量
- 16.6 总结

第十七章 Spring注解模式
- 17.1 测试环境搭建
- 17.2 basePackages 模式启动
- 17.2.1 AnnotationConfigApplicationContext 无参构造器
- 17.2.2 scan 方法分析
- 17.2.2.1 doScan 方法分析
- 17.2.2.2 findCandidateComponents : 搜索包路径下的所有 Bean Definition
- 17.2.2.2.1 scanCandidateComponents 方法分析
- 17.2.2.2.2 addCandidateComponentsFromIndex 方法分析
- 17.2.2.3 单个 Bean Definition 的处理
- 17.2.2.3.1 处理1：Scope 相关处理。
- 17.2.2.3.2 处理2：Bean Name 相关处理
- 17.2.2.3.3 处理3.1：Bean Definiton 默认值处理
- 17.2.2.3.4 处理3.2：Spring 中通用注解的处理
- 17.2.2.3.5 处理4：代理相关的 Scope 设置
- 17.2.2.3.6 处理5：注册到容器
- 17.3 componentClasses 模式启动
- 17.3.1 register 方法分析
- 17.4 总结

第十八章 资源解析器
- 18.1 测试环境搭建
- 18.2 类图分析
- 18.2 PathMatchingResourcePatternResolver 构造器
- 18.2.1 DefaultResourceLoader 中的成员变量
- 18.2.2 DefaultResourceLoader 类图
- 18.3 getResource 方法分析
- 18.4 getResources 方法分析
- 18.4.1 findPathMatchingResources 方法分析
- 18.4.1.1 bundle 协议处理
- 18.4.1.2 vfs 协议处理
- 18.4.1.3 "jar" 协议处理
- 18.4.1.4 其他协议处理
- 18.4.2 findAllClassPathResources 方法分析
- 18.4.3 创建 Resoruce 数组
- 18.5 总结

第十九章 注解元数据读取器工厂
- 19.1 认识 MetadataReaderFactory
- 19.2 SimpleMetadataReaderFactory 分析
- 19.2.1 getMetadataReader 法分析
- 19.3 CachingMetadataReaderFactory 分析
- 19.4 总结 
  
第二十章 注解元数据读取器
- 20.1 初识 MetadataReader
- 20.2 SimpleAnnotationMetadataReadingVisitor 成员变量
- 20.3 SimpleAnnotationMetadata 成员变量
- 20.4 SimpleMethodMetadata 成员变量
- 20.5 MergedAnnotationsCollection 成员变量
- 20.6 TypeMappedAnnotation 成员变量
- 20.7 AnnotationTypeMappings 成员变量
- 20.8 ClassMetadataReadingVisitor 成员变量
- 20.9 AnnotationMetadataReadingVisitor 成员变量

第二十一章 Scope元数据解析
- 21.1 ScopeMetadataResolver 分析
- 21.1.1 ScopeMetadata 分析
- 21.1.2 AnnotationScopeMetadataResolver 中的解析
- 21.1 总结

第二十二章 BeanName 生成策略
- 22.1 AnnotationBeanNameGenerator 分析
- 22.1.1 AnnotatedBeanDefinition 类型的 Bean Name 处理
- 22.1.2 非 AnnotatedBeanDefinition 类型的 Bean Name 处理
- 22.2 FullyQualifiedAnnotationBeanNameGenerator 分析
- 22.3 DefaultBeanNameGenerator 分析
- 22.4 总结

第二十三章 ConfigurationClassPostProcessor
- 23.1 初识 ConfigurationClassPostProcessor
- 23.2 测试用例搭建
- 23.3 postProcessBeanDefinitionRegistry 分析
- 23.3.1 容器内已存在的 Bean 进行候选分类
- 23.3.2 候选 Bean Definition Holder 的排序
- 23.3.3 Bean Name 生成器的创建
- 23.3.4 初始化基本环境信息
- 23.3.5 解析候选 Bean
- 23.3.6 注册 Import Bean 和清理数据
- 23.4 postProcessBeanFactory 分析
- 23.5 总结

第二十四章 条件注解
- 24.1 测试环境搭建
- 24.2 条件注解分析
- 24.3 总结

第二十五章 类元数据
- 25.1 接口说明
- 25.2 StandardClassMetadata 分析

第二十六章 注解元数据
- 26.1 基础认识
- 26.2 AnnotatedTypeMetadata 接口说明
- 26.3 AnnotationMetadata 接口说明
- 26.4 StandardAnnotationMetadata 分析
- 26.5 java 中注解数据获取

第二十七章 DeferredImportSelectorHandler
- 27.1 初识 DeferredImportSelectorHandler
- 27.1.1 DeferredImportSelectorHandler 成员变量
- 27.2 测试环境搭建
- 27.3 handle 方法分析
- 27.4 DeferredImportSelectorGroupingHandler
- 27.5 processImports 方法分析
- 27.6 总结

第二十八章 ConfigurationClassBeanDefinitionReader
- 28.1 测试环境搭建
- 28.2 构造函数
- 28.3 loadBeanDefinitions 分析
- 28.4 TrackedConditionEvaluator 分析
- 28.5 loadBeanDefinitionsForConfigurationClass 方法分析
- 28.6 loadBeanDefinitionsForBeanMethod 分析
- 28.9 loadBeanDefinitionsForBeanMethod 方法分析
- 28.10 registerBeanDefinitionForImportedConfigurationClass 方法分析
- 28.10.1 ImportedConfigurationClass 测试用例编写
- 28.11 loadBeanDefinitionsFromImportedResources 方法分析
- 28.11.1 测试用例编写
- 28.12 loadBeanDefinitionsFromRegistrars 方法分析
- 28.12.1 测试用例编写
- 28.13 总结

第二十九章 Spring 配置类解析
- 29.1 parse 方法分析
- 29.2 processConfigurationClass 分析

第三十章 Spring 排序注解
- 30.1 测试环境搭建
- 30.2 OrderComparator.sort 分析
