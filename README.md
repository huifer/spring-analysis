# Spring Analysis 
## 项目介绍
- 本项目为 Spring 源码分析项目

[![Netlify Status](https://api.netlify.com/api/v1/badges/7d4d612b-0d73-47e7-a4bf-2c8e9da26bbb/deploy-status)](https://app.netlify.com/sites/huifer-spring-analysis/deploys)
- Netlify: https://huifer-spring-analysis.netlify.app
    
## 目录

### SpringBean相关分析
#### Spring propertyEditor 相关接口分析

- [propertyEditor属性接口](/docs/beans/propertyEditor/Readme.md)
    - [Spring-PropertyEditorRegistry](/docs/beans/propertyEditor/Spring-ResourceEditorRegistrar.md)
    - [Spring-PropertyEditorRegistrySupport](/docs/beans/propertyEditor/Spring-ResourceEditorRegistrar.md)
    - [Spring-PropertyEditorRegistrar](/docs/beans/propertyEditor/Spring-ResourceEditorRegistrar.md)
    - [Spring-ResourceEditorRegistrar](/docs/beans/propertyEditor/Spring-ResourceEditorRegistrar.md)
    
### BeanInfoFactory 相关接口分析
- [BeanInfoFactory](/docs/beans/BeanInfoFactory/readme.md)
    - [Spring-BeanInfoFactory](/docs/beans/BeanInfoFactory/Spring-BeanInfoFactory.md)
    - [Spring-ExtendedBeanInfo](/docs/beans/BeanInfoFactory/Spring-ExtendedBeanInfo.md)
    - [Spring-ExtendedBeanInfoFactory](/docs/beans/BeanInfoFactory/Spring-ExtendedBeanInfoFactory.md)
    
#### Spring Scope 相关接口分析

- [Scope作用域接口](/docs/beans/Scope/Readme.md)
    - [Spring-Scope](/docs/beans/Scope/Spring-Scope.md)
    - [Spring-AbstractRequestAttributesScope](/docs/beans/Scope/Spring-AbstractRequestAttributesScope.md)
    - [Spring-ServletContextScope](/docs/beans/Scope/Spring-ServletContextScope.md)
    - [Spring-SimpleMapScope](/docs/beans/Scope/Spring-SimpleMapScope.md)
    - [Spring-SimpleThreadScope](/docs/beans/Scope/Spring-SimpleThreadScope.md)
    - [Spring-SimpleTransactionScope](/docs/beans/Scope/Spring-SimpleTransactionScope.md)
    - [Spring-SimpSessionScope](/docs/beans/Scope/Spring-SimpSessionScope.md)
    
#### Spring BeanMetadataElement 相关接口分析

- [BeanMetadataElement: bean元信息对象](/docs/beans/BeanMetadataElement/Readme.md)
    - [Spring-AliasDefinition](/docs/beans/BeanMetadataElement/Spring-AliasDefinition.md)
    - [Spring-AutowireCandidateQualifier](/docs/beans/BeanMetadataElement/Spring-AutowireCandidateQualifier.md)
    - [Spring-BeanComponentDefinition](/docs/beans/BeanMetadataElement/Spring-BeanComponentDefinition.md)
    - [Spring-BeanDefinitionHolder](/docs/beans/BeanMetadataElement/Spring-BeanDefinitionHolder.md)
    - [Spring-BeanMetadataAttribute](/docs/beans/BeanMetadataElement/Spring-BeanMetadataAttribute.md)
    - [Spring-ConstructorArgumentValues.ValueHolder](/docs/beans/BeanMetadataElement/Spring-ConstructorArgumentValues.ValueHolder.md)
    - [Spring-DocumentDefaultsDefinition](/docs/beans/BeanMetadataElement/Spring-DocumentDefaultsDefinition.md)
    - [Spring-ImportDefinition](/docs/beans/BeanMetadataElement/Spring-ImportDefinition.md)
    - [Spring-ManagedList](/docs/beans/BeanMetadataElement/Spring-ManagedList.md)
    - [Spring-ManagedMap](/docs/beans/BeanMetadataElement/Spring-ManagedMap.md)
    - [Spring-ManagedProperties](/docs/beans/BeanMetadataElement/Spring-ManagedProperties.md)
    - [Spring-ManagedSet](/docs/beans/BeanMetadataElement/Spring-ManagedSet.md)
    - [Spring-MethodOverride](/docs/beans/BeanMetadataElement/Spring-MethodOverride.md)
    - [Spring-TypedStringValue](/docs/beans/BeanMetadataElement/Spring-TypedStringValue.md)
    
    - [Mergeable](/docs/beans/BeanMetadataElement/Mergeable/Readme.md)
    
    - [Spring-BeanReference](/docs/beans/BeanMetadataElement/BeanReference/Spring-BeanReference.md)
    - [Spring-RuntimeBeanNameReference](/docs/beans/BeanMetadataElement/BeanReference/Spring-RuntimeBeanNameReference.md)
    - [Spring-RuntimeBeanReference](/docs/beans/BeanMetadataElement/BeanReference/Spring-RuntimeBeanReference.md)
    - [ComponentDefinition](/docs/beans/ComponentDefinition/Readme.md)
        - [ComponentDefinition](/docs/beans/ComponentDefinition/Spring-ComponentDefinition.md)
            - [AbstractComponentDefinition](/docs/beans/ComponentDefinition/Spring-AbstractComponentDefinition.md)
                - [CompositeComponentDefinition](/docs/beans/ComponentDefinition/Spring-CompositeComponentDefinition.md)
                    - [AspectComponentDefinition](/docs/beans/ComponentDefinition/Spring-AspectComponentDefinition.md)
                - [PointcutComponentDefinition](/docs/beans/ComponentDefinition/Spring-PointcutComponentDefinition.md)
                - [AdvisorComponentDefinition](/docs/beans/ComponentDefinition/Spring-AdvisorComponentDefinition.md)
            - [BeanComponentDefinition](/docs/beans/BeanMetadataElement/Spring-BeanComponentDefinition.md)

#### Spring BeanFactory 相关接口分析

- [BeanFactory: Bean工厂,创建获取bean等](/docs/beans/factory/BeanFactory/Readme.md)
    - [HierarchicalBeanFactory](/docs/beans/factory/BeanFactory/Spring-HierarchicalBeanFactory.md)
        - [ConfigurableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ConfigurableBeanFactory.md)
            - [AbstractBeanFactory](/docs/beans/factory/BeanFactory/Spring-AbstractBeanFactory.md)
              - [AbstractAutowireCapableBeanFactory](/docs/beans/factory/BeanFactory/Spring-AbstractAutowireCapableBeanFactory.md)
                - [DefaultListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-DefaultListableBeanFactory.md)

      - [SimpleJndiBeanFactory](/docs/beans/factory/BeanFactory/impl/Spring-SimpleJndiBeanFactory.md)
    - [AutowireCapableBeanFactory](/docs/beans/factory/BeanFactory/Spring-AutowireCapableBeanFactory.md)
    - [ListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ListableBeanFactory.md)



#### Spring GenericTypeAwarePropertyDescriptor 相关接口分析

- [GenericTypeAwarePropertyDescriptor](/docs/beans/GenericTypeAwarePropertyDescriptor/Spring-GenericTypeAwarePropertyDescriptor.md)



#### Spring CachedIntrospectionResults 相关接口分析
- [CachedIntrospectionResults](/docs/beans/CachedIntrospectionResults/Spring-CachedIntrospectionResults.md)

    
    
    
#### Spring BeanWrapper 相关接口分析    
- [BeanWrapper导读](/docs/beans/BeanWrapper/Readme.md)
    - [BeanWrapper](/docs/beans/BeanWrapper/Spring-BeanWrapper.md)
    - [BeanWrapperImpl](/docs/beans/BeanWrapper/Spring-BeanWrapperImpl.md)
    
#### Spring PropertyAccessor 相关接口分析    
- [PropertyAccessor导读](/docs/beans/PropertyAccessor/Readme.md)
    - [AbstractPropertyAccessor](/docs/beans/PropertyAccessor/Spring-AbstractPropertyAccessor.md)
    - [PropertyAccessor](/docs/beans/PropertyAccessor/Spring-PropertyAccessor.md)

#### Spring ConfigurablePropertyAccessor 相关接口分析    
- [ConfigurablePropertyAccessor](/docs/beans/ConfigurablePropertyAccessor/Spring-ConfigurablePropertyAccessor.md)
    
    
    
#### Spring InstantiationStrategy 相关接口分析
- [InstantiationStrategy导读](/docs/beans/factory/BeanFactory/support/InstantiationStrategy/Readme.md)
    - [InstantiationStrategy](/docs/beans/factory/BeanFactory/support/InstantiationStrategy/Spring-InstantiationStrategy.md)
    - [Spring-SimpleInstantiationStrategy](/docs/beans/factory/BeanFactory/support/InstantiationStrategy/Spring-SimpleInstantiationStrategy.md)
    - [Spring-CglibSubclassingInstantiationStrategy](/docs/beans/factory/BeanFactory/support/InstantiationStrategy/Spring-CglibSubclassingInstantiationStrategy.md)


#### Spring AutowireCandidateResolver 相关接口分析
- [AutowireCandidateResolver](/docs/beans/factory/BeanFactory/support/AutowireCandidateResolver/Spring-AutowireCandidateResolver.md)
- [ContextAnnotationAutowireCandidateResolver](/docs/beans/factory/BeanFactory/support/AutowireCandidateResolver/Spring-ContextAnnotationAutowireCandidateResolver-未完成.md)
- [GenericTypeAwareAutowireCandidateResolver](/docs/beans/factory/BeanFactory/support/AutowireCandidateResolver/Spring-GenericTypeAwareAutowireCandidateResolver.md)
- [QualifierAnnotationAutowireCandidateResolver](/docs/beans/factory/BeanFactory/support/AutowireCandidateResolver/Spring-QualifierAnnotationAutowireCandidateResolver.md)
- [SimpleAutowireCandidateResolver](/docs/beans/factory/BeanFactory/support/AutowireCandidateResolver/Spring-SimpleAutowireCandidateResolver.md)

#### Spring NamedBean 相关接口分析
- [DependencyObjectProvider](/docs/beans/factory/BeanFactory/ObjectProvider/Spring-DependencyObjectProvider.md)
- [Jsr330Provider](/docs/beans/factory/BeanFactory/ObjectProvider/Spring-Jsr330Provider.md)
- [ObjectProvider](/docs/beans/factory/BeanFactory/ObjectProvider/Spring-ObjectProvider.md)


#### Spring ObjectProvider 相关接口分析
- [ExposeBeanNameIntroduction](/docs/beans/factory/BeanFactory/NamedBean/Spring-ExposeBeanNameIntroduction.md)
- [NamedBean](/docs/beans/factory/BeanFactory/NamedBean/Spring-NamedBean.md)
- [NamedBeanHolder](/docs/beans/factory/BeanFactory/NamedBean/Spring-NamedBeanHolder.md)


### SpringCore相关分析
#### Spring Registry 相关分析

- [DefaultSingletonBeanRegistry](/docs/core/registry/Spring-DefaultSingletonBeanRegistry.md)
- [FactoryBeanRegistrySupport](/docs/core/registry/Spring-FactoryBeanRegistrySupport.md)
- [SimpleAliasRegistry](/docs/core/registry/Spring-SimpleAliasRegistry.md)
- [SingletonBeanRegistry](/docs/core/registry/Spring-SingletonBeanRegistry.md)

#### Spring Convert 相关分析

- [Spring-ConditionalConverter](/docs/core/convert/Spring-ConditionalConverter.md)
    - [Spring-AbstractConditionalEnumConverter](/docs/core/convert/ConditionalConverter/Spring-AbstractConditionalEnumConverter.md)
    - [Spring-NumberToNumberConverterFactory](/docs/core/convert/ConditionalConverter/Spring-NumberToNumberConverterFactory.md)
- [Spring-ConfigurableConversionService](/docs/core/convert/Spring-ConfigurableConversionService.md)
- [Spring-ConversionService](/docs/core/convert/Spring-ConversionService.md)
- [Spring-Converter](/docs/core/convert/Spring-Converter.md)
- [Spring-ConverterRegistry](/docs/core/convert/Spring-ConverterRegistry.md)
- [Spring-DefaultConversionService](/docs/core/convert/Spring-DefaultConversionService.md)
- [Spring-DefaultFormattingConversionService](/docs/core/convert/Spring-DefaultFormattingConversionService.md)
- [Spring-FormatterRegistry](/docs/core/convert/Spring-FormatterRegistry.md)
- [Spring-FormattingConversionService](/docs/core/convert/Spring-FormattingConversionService.md)
- [Spring-GenericConversionService](/docs/core/convert/Spring-GenericConversionService.md)
- [Spring-GenericConverter](/docs/core/convert/Spring-GenericConverter.md)
    - [Spring-NoOpConverter](/docs/core/convert/GenericConverter/Spring-NoOpConverter.md)
    - [Spring-ParserConverter](/docs/core/convert/GenericConverter/Spring-ParserConverter)
    - [Spring-PrinterConverter](/docs/core/convert/GenericConverter/Spring-PrinterConverter)
- [Spring-TypeConverter](/docs/core/convert/Spring-TypeConverter.md)
    - [Spring-TypeConverterDelegate](/docs/core/convert/TypeConverter/Spring-TypeConverterDelegate.md)
    - [Spring-TypeConverterSupport](/docs/core/convert/TypeConverter/Spring-TypeConverterSupport.md)
- [Spring-TypeDescriptor](/docs/core/convert/Spring-TypeDescriptor.md)

#### Spring AttributeAccessor 相关分析

- [AttributeAccessor](/docs/core/AttributeAccessor/Spring-AttributeAccessor.md)
- [AttributeAccessorSupport](/docs/core/AttributeAccessor/Spring-AttributeAccessorSupport.md)


#### Spring ParameterNameDiscoverer 相关分析
- [ParameterNameDiscoverer 参数名称发现器](/docs/core/ParameterNameDiscoverer/Readme.md)
    - **[StandardReflectionParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-StandardReflectionParameterNameDiscoverer.md)**
    - [AspectJAdviceParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-AspectJAdviceParameterNameDiscoverer-未完成.md)
    - [KotlinReflectionParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-KotlinReflectionParameterNameDiscoverer.md)
    - [PrioritizedParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-PrioritizedParameterNameDiscoverer.md)
        - [DefaultParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-DefaultParameterNameDiscoverer.md)
    - [AspectJAnnotationParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-AspectJAnnotationParameterNameDiscoverer.md)
    - [LocalVariableTableParameterNameDiscoverer](/docs/core/ParameterNameDiscoverer/Spring-LocalVariableTableParameterNameDiscoverer.md)
    

### Spring环境相关分析

- [PropertyResolver](/docs/env/PropertyResolver/Readme.md)
- [Spring-AbstractPropertyResolver](/docs/env/PropertyResolver/Spring-AbstractPropertyResolver.md)
- [Spring-Environment](/docs/env/PropertyResolver/Spring-Environment.md)
- [Spring-PropertyPlaceholderHelper](/docs/env/PropertyResolver/Spring-PropertyPlaceholderHelper.md)
- [Spring-PropertyResolver](/docs/env/PropertyResolver/Spring-PropertyResolver.md)
- [Spring-PropertySources](/docs/env/PropertyResolver/Spring-PropertySources.md)
- [Spring-PropertySourcesPropertyResolver](/docs/env/PropertyResolver/Spring-PropertySourcesPropertyResolver.md)
- [Spring-AbstractEnvironment](/docs/env/environment/Spring-AbstractEnvironment.md)
- [Spring-MockEnvironment](/docs/env/environment/Spring-MockEnvironment.md)
- [Spring-StandardEnvironment](/docs/env/environment/Spring-StandardEnvironment.md)
- [Spring-StandardServletEnvironment](/docs/env/environment/Spring-StandardServletEnvironment.md)
- [PropertySource](/docs/env/PropertyResolver/PropertySource/Readme.md)
    - [Spring-CommandLinePropertySource](/docs/env/PropertyResolver/PropertySource/Spring-CommandLinePropertySource.md)
    - [Spring-ComparisonPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-ComparisonPropertySource.md)
    - [Spring-CompositePropertySource](/docs/env/PropertyResolver/PropertySource/Spring-CompositePropertySource.md)
    - [Spring-EnumerablePropertySource](/docs/env/PropertyResolver/PropertySource/Spring-EnumerablePropertySource.md)
    - [Spring-MapPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-MapPropertySource.md)
    - [Spring-MockPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-MockPropertySource.md)
    - [Spring-PropertiesPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-PropertiesPropertySource.md)
    - [Spring-ResourcePropertySource](/docs/env/PropertyResolver/PropertySource/Spring-ResourcePropertySource.md)
    - [Spring-ServletConfigPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-ServletConfigPropertySource.md)
    - [Spring-ServletContextPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-ServletContextPropertySource.md)
    - [Spring-SimpleCommandLineArgsParser](/docs/env/PropertyResolver/PropertySource/Spring-SimpleCommandLineArgsParser.md)
    - [Spring-SimpleCommandLinePropertySource](/docs/env/PropertyResolver/PropertySource/Spring-SimpleCommandLinePropertySource.md)
    - [Spring-StubPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-StubPropertySource.md)
    - [Spring-SystemEnvironmentPropertySource](/docs/env/PropertyResolver/PropertySource/Spring-SystemEnvironmentPropertySource.md)
- [PlaceholderResolver](/docs/env/PropertyResolver/PlaceholderResolver/Readme.md)
    - [Spring-PlaceholderResolver](/docs/env/PropertyResolver/PlaceholderResolver/Spring-PlaceholderResolver.md)
    - [Spring-PropertyPlaceholderConfigurerResolver](/docs/env/PropertyResolver/PlaceholderResolver/Spring-PropertyPlaceholderConfigurerResolver.md)
    - [Spring-ServletContextPlaceholderResolver](/docs/env/PropertyResolver/PlaceholderResolver/Spring-ServletContextPlaceholderResolver.md)
    - [Spring-SystemPropertyPlaceholderResolver](/docs/env/PropertyResolver/PlaceholderResolver/Spring-SystemPropertyPlaceholderResolver.md)

### Spring工具类分析

- [StringValueResolver](/docs/utils/StringValueResolver/Readme.md)
    - [Spring-EmbeddedValueResolver](/docs/utils/StringValueResolver/Spring-EmbeddedValueResolver.md)
    - [Spring-PlaceholderResolvingStringValueResolver](/docs/utils/StringValueResolver/Spring-PlaceholderResolvingStringValueResolver.md)
    - [Spring-StaticStringValueResolver](/docs/utils/StringValueResolver/Spring-StaticStringValueResolver.md)

- [ReflectionUtils](/docs/utils/Spring-ReflectionUtils.md)
- [ConversionUtils](/docs/utils/Spring-ConversionUtils.md)

### Spring 其他
- [循环依赖](/docs/other/重说循环依赖.md)