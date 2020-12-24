# Spring AbstractApplicationContext
- 类全路径: `org.springframework.context.support.AbstractApplicationContext`





- `AbstractApplicationContext` 集成自`DefaultResourceLoader` , 有关`DefaultResourceLoader`的分析请查看[这篇文章](/docs/core/io/ResourceLoader/Spring-DefaultResourceLoader.md)

- 阅读`AbstractApplicationContext`成员变量相关代码





## 成员变量

<details>
    <summary>AbstractApplicationContext 静态变量</summary>



```java
  /**
   * Name of the MessageSource bean in the factory.
   * If none is supplied, message resolution is delegated to the parent.
* MessageSource 的名字
   * @see MessageSource
   */
  public static final String MESSAGE_SOURCE_BEAN_NAME = "messageSource";

  /**
   * Name of the LifecycleProcessor bean in the factory.
   * If none is supplied, a DefaultLifecycleProcessor is used.
*
* LifecycleProcessor 的名字
   * @see org.springframework.context.LifecycleProcessor
   * @see org.springframework.context.support.DefaultLifecycleProcessor
   */
  public static final String LIFECYCLE_PROCESSOR_BEAN_NAME = "lifecycleProcessor";

  /**
   * Name of the ApplicationEventMulticaster bean in the factory.
   * If none is supplied, a default SimpleApplicationEventMulticaster is used.
*
* ApplicationEventMulticaster 的名称
* 事件传播器
   * @see org.springframework.context.event.ApplicationEventMulticaster
   * @see org.springframework.context.event.SimpleApplicationEventMulticaster
   */
  public static final String APPLICATION_EVENT_MULTICASTER_BEAN_NAME = "applicationEventMulticaster";
```

</details>

在静态变量中定义了下面几个接口的beanName

1. `MessageSource`
2. `LifecycleProcessor`
3. `ApplicationEventMulticaster`





在了解过静态变量之后来看看剩下的一些变量





<details>
    <summary>AbstractApplicationContext 其他成员变量</summary>





```java
  /**
* BeanFactoryPostProcessors to apply on refresh.
* bean 后置处理器列表
* */
  private final List<BeanFactoryPostProcessor> beanFactoryPostProcessors = new ArrayList<>();

  /**
* Flag that indicates whether this context is currently active.
* 是否存活状态的标记
* */
  private final AtomicBoolean active = new AtomicBoolean();

  /**
* Flag that indicates whether this context has been closed already.
* 是否关闭的标记
* */
  private final AtomicBoolean closed = new AtomicBoolean();

  /**
*  Synchronization monitor for the "refresh" and "destroy".
*
* refresh 方法和 destroy 方法的锁
* */
  private final Object startupShutdownMonitor = new Object();

  /**
*  Statically specified listeners.
*
*  应用监听器列表
*  */
  private final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();

  /**
*  Unique id for this context, if any.
* 上下文id
* */
  private String id = ObjectUtils.identityToString(this);

  /**
*  Display name.
* 上下文名称
* */
  private String displayName = ObjectUtils.identityToString(this);

  /**
*  Parent context.
*  父上下文
*  */
  @Nullable
  private ApplicationContext parent;

  /**
* Environment used by this context.
* 环境配置
* */
  @Nullable
  private ConfigurableEnvironment environment;

  /**
* System time in milliseconds when this context started.
*
* 启动时间
* */
  private long startupDate;

  /**
* Reference to the JVM shutdown hook, if registered.
* 关闭的钩子线程
* */
  @Nullable
  private Thread shutdownHook;

  /**
*  ResourcePatternResolver used by this context.
*  资源解析器
*  */
  private final ResourcePatternResolver resourcePatternResolver;

  /**
* LifecycleProcessor for managing the lifecycle of beans within this context.
*
* 生命周期处理器
* */
  @Nullable
  private LifecycleProcessor lifecycleProcessor;

  /**
*  MessageSource we delegate our implementation of this interface to.
* 消息源 主要用于国际化
* */
  @Nullable
  private MessageSource messageSource;

  /**
* Helper class used in event publishing.
* 事件传播器
* */
  @Nullable
  private ApplicationEventMulticaster applicationEventMulticaster;

  /**
* Local listeners registered before refresh.
* 应用监听器列表
* */
  @Nullable
  private Set<ApplicationListener<?>> earlyApplicationListeners;

  /**
*  ApplicationEvents published before the multicaster setup.
* 需要提前发布的应用事件
* */
  @Nullable
  private Set<ApplicationEvent> earlyApplicationEvents;
```

</details>



在成员变量中涉及到的接口和类如下

1. `BeanFactoryPostProcessor`
2. `ApplicationListener`
3. `ApplicationContext`
4. `ConfigurableEnvironment`
5. `ResourcePatternResolver`
6. `LifecycleProcessor`
7. `MessageSource`
8. `ApplicationEventMulticaster`
9. `ApplicationEvent`











- 下面开始进行方法分析



## 方法分析





### publishEvent

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#publishEvent(java.lang.Object, org.springframework.core.ResolvableType)`
- 方法作用: 推送事件





<details>
    <summary>publishEvent 方法详情</summary>





```java
  protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
      Assert.notNull(event, "Event must not be null");

      // Decorate event as an ApplicationEvent if necessary
// 应用事件对象
      ApplicationEvent applicationEvent;
      if (event instanceof ApplicationEvent) {
          applicationEvent = (ApplicationEvent) event;
      }
      else {
       // 应用实践对象的封装
          applicationEvent = new PayloadApplicationEvent<>(this, event);
          if (eventType == null) {
           // 事件类型
              eventType = ((PayloadApplicationEvent<?>) applicationEvent).getResolvableType();
          }
      }

      // Multicast right now if possible - or lazily once the multicaster is initialized
      if (this.earlyApplicationEvents != null) {
       // 需要传播的事件列表中加入数据
          this.earlyApplicationEvents.add(applicationEvent);
      }
      else {
       // 传播器传播事件
          getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
      }

      // 父上下存在的情况下依靠父上下文进行事件推送
      // Publish event via parent context as well...
      if (this.parent != null) {
          if (this.parent instanceof AbstractApplicationContext) {
           // 发布事件
              ((AbstractApplicationContext) this.parent).publishEvent(event, eventType);
          }
          else {
      // 发布事件
              this.parent.publishEvent(event);
          }
      }
  }
```

</details>





`publishEvent` 方法逻辑:

1. 对事件对象做类型判断后转换成下面两种事件类
   1. `ApplicationEvent`
   2. `PayloadApplicationEvent`
2. 在得到事件对象后会将事件进行发布或者加入到**需要提前处理的事件列表`earlyApplicationEvents`中**
3. 如果存在父应用上下文的情况下会通过父应用上下文进行事件发布









### refresh

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#refresh`



- `refresh` 主要处理下面几个流程
  1. **准备刷新上下文** , `prepareRefresh()`
  2. **创建或者获取 BeanFactory**  , `obtainFreshBeanFactory()`
  3. **准备 BeanFactory , 为 BeanFactory 设置属性,**  `prepareBeanFactory(beanFactory)`
  4. try
     1. **在 BeanFactory 准备好后子类可以为其定制后置操作**, `postProcessBeanFactory(beanFactory)`
     2. **`BeanFactoryPostProcessor` 接口集合的调用**, `invokeBeanFactoryPostProcessors(beanFactory)`
     3. **注册`BeanPostProcessor`接口**, `registerBeanPostProcessors(beanFactory)`
     4. **实例化 message source** , `initMessageSource`
     5. **实例化应用事件传播器,** `initApplicationEventMulticaster()`
     6. **子类的刷新操作,用于实例化特定的 bean** , `onRefresh()`
     7. **注册监听器**, `registerListeners()`
     8. 完成 BeanFactory 的实例化, 即**实例化非延迟加载的单例对象**, `finishBeanFactoryInitialization(beanFactory)`
     9. **完成刷新方法, 发布部分事件**, `finishRefresh()`
  5. catch
     1. **摧毁以及创建的bean对象**, `destroyBeans()`
     2. **取消刷新,设置 `active` 为 `false`,** `cancelRefresh(ex)`
  6. finally
     1. 重置通用缓存,`resetCommonCaches()`



上面我们将 `refresh`方法的流程都整理出来, 下面是`refresh` 方法的完整代码



<details>
    <summary>refresh 方法详情</summary>





```JAVA
    @Override
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            // Prepare this context for refreshing.
            // 准备刷新此上下文。
            prepareRefresh();

			// 创建出 beanFactory
            // Tell the subclass to refresh the internal bean factory.
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // Prepare the bean factory for use in this context.
			// 准备 beanFactory , 对 beanFactory 进行设置数据等
            prepareBeanFactory(beanFactory);

            try {
				// beanFactory 在子类中进行后置处理
                // Allows post-processing of the bean factory in context subclasses.
                postProcessBeanFactory(beanFactory);

                // BeanFactoryPostProcessor 方法调用
                // Invoke factory processors registered as beans in the context.
                invokeBeanFactoryPostProcessors(beanFactory);

                // 注册 beanPostProcessor
                // Register bean processors that intercept bean creation.
                registerBeanPostProcessors(beanFactory);

                // 实例化 message source 相关信息
                // Initialize message source for this context.
                initMessageSource();

                // 实例化 应用事件传播器
                // Initialize event multicaster for this context.
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses.
                onRefresh();

                // Check for listener beans and register them.
				// 注册监听器
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons.
				// 完成 beanFactory 的实例化
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
				// 完成刷新
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.
				// 摧毁bean
                destroyBeans();

                // Reset 'active' flag.
				// 取消刷新
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            }

            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
				// 重置通用缓存
                resetCommonCaches();
            }
        }
    }

```

</details>







在了解 `refresh` 的几个阶段方法后我们来对每一个方法进行分析了解其内部正真的执行事项。





### prepareRefresh

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#prepareRefresh`



<details>
    <summary>prepareRefresh 方法详情</summary>







```java
/**
 * Prepare this context for refreshing, setting its startup date and
 * active flag as well as performing any initialization of property sources.
 *
 * 准备刷新此上下文
 */
protected void prepareRefresh() {
    // Switch to active.
    // 设置开始时间
    this.startupDate = System.currentTimeMillis();
    // 设置关闭标记位 false
    this.closed.set(false);
    // 设置激活标记位 true
    this.active.set(true);

    // 日志
    if (logger.isDebugEnabled()) {
        if (logger.isTraceEnabled()) {
            logger.trace("Refreshing " + this);
        }
        else {
            logger.debug("Refreshing " + getDisplayName());
        }
    }

    // 初始化属性, 占位符资源等数据处理
    // 抽象方法, 子类实现
    // Initialize any placeholder property sources in the context environment.
    initPropertySources();

    // 进行数据必填性验证
    // Validate that all properties marked as required are resolvable:
    // see ConfigurablePropertyResolver#setRequiredProperties
    getEnvironment().validateRequiredProperties();

    // 处理早期应用监听器列表 和 应用监听器列表
    // Store pre-refresh ApplicationListeners...
    if (this.earlyApplicationListeners == null) {
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        // Reset local application listeners to pre-refresh state.
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

</details>





在`prepareRefresh`流程中处理下面这些事项

1. 设置开始时间
2. 设置关闭标记位
3. 设置激活标记位
4. 属性资源初始化
5. 数据必填性验证
6. 关于应用监听器的处理
   1. `earlyApplicationListeners`
   2. `applicationListeners`









### obtainFreshBeanFactory

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory`



- 方法中调用了两个抽象方法, 这两个抽象方法均由子类实现. 
  1. `refreshBeanFactory` 刷新beanFactory
  2. `getBeanFactory` 获取BeanFactory







```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    // 刷新 beanFactory , 子类实现
    refreshBeanFactory();
    // 获取 beanFactory , 子类实现
    return getBeanFactory();
}
```









### prepareBeanFactory

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory`
- 方法作用: 设置 beanFactory 的属性

设置属性列表

1. classLoader
2. EL表达式解析器
   1. `StandardBeanExpressionResolver`
3. 属性编辑器注册工具
   1.  `ResourceEditorRegistrar`
4. bean 后置处理器
   1. `ApplicationContextAwareProcessor`
   2. `ApplicationListenerDetector`
5. 添加需要忽略的接口 
   1. `EnvironmentAware`
   2. ``EmbeddedValueResolverAware`
   3. `ResourceLoaderAware`
   4. `ApplicationEventPublisherAware`
   5. `MessageSourceAware`
   6. `ApplicationContextAware`
6. 添加依赖
   1. `BeanFactory`
   2. `ResourceLoader`
   3. `ApplicationEventPublisher`
   4. `ApplicationContext`
7. 注册单例对象
   1. `LoadTimeWeaverAwareProcessor`
   2. `environment`
   3. `systemProperties`
   4. `systemEnvironment`







<details>
    <summary>prepareBeanFactory 方法详情</summary>





```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    // 设置 classLaoder
    beanFactory.setBeanClassLoader(getClassLoader());
    // 设置 el 表达式解析器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    // 添加属性编辑器注册工具
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // Configure the bean factory with context callbacks.
    // 添加 bean 后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    // 添加忽略的接口
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // 注册依赖
    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // 添加 bean 后置处理器
    // Register early post-processor for detecting inner beans as ApplicationListeners.
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // 判断是否存在 loadTimeWeaver bean
    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        // 添加后置处理器
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // Set a temporary ClassLoader for type matching.
        // 设置临时的 classLoader
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // environment bean 注册
    // Register default environment beans.
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    // systemProperties bean 注册
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    // systemEnvironment bean 注册
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

</details>







### postProcessBeanFactory

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#postProcessBeanFactory`

- 抽象方法, 子类实现. 主要作用是处理 beanFactory 的后置方法



```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
}
```







### invokeBeanFactoryPostProcessors

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors`

- 方法作用: 执行`BeanFactoryPostProcessor#postProcessBeanFactory`方法



- 方法流程如下
  1. 执行`BeanFactoryPostProcessor#postProcessBeanFactory` 方法
  2. `LoadTimeWeaver` 对象的设置，设置条件
     1. 判断临时类加载器是否存在
     2.  是否包含 loadTimeWeaver bean



<details>
    <summary>invokeBeanFactoryPostProcessors方法详情</summary>







```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 后置处理器委托对象
    // 调用 BeanFactoryPostProcessor 方法
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

    // Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
    // (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
    // 判断临时类加载器是否存在
    // 是否包含 loadTimeWeaver bean
    if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        // 添加 bean后置处理器
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // 添加临时类加载器
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}
```

</details>











### registerBeanPostProcessors

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#registerBeanPostProcessors`
- 方法作用: 注册 beanPostProcessor





```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 后置处理器委托类进行能注册
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```





- 在 `invokeBeanFactoryPostProcessors` 方法和`registerBeanPostProcessors` 方法中都出现了`PostProcessorRegistrationDelegate`(后置处理器委托对象)，[分析文章](/docs/core/Spring-PostProcessorRegistrationDelegate-未完成.md)







### initMessageSource

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#initMessageSource`

- 方法作用: 实例化 MessageSource 对象

- 处理逻辑

  1. 判断BeanName(**messageSource**) 是否存在对应的bean实例

     1. 存在

        获取 MessageSource 实例并尝试设置父 MessageSource

     2. 不存在

        创建 MessageSource 实现类(`DelegatingMessageSource`) 设置 父 MessageSource 并注册到容器











<details>
    <summary>initMessageSource 方法详情</summary>





```java
protected void initMessageSource() {
    // 获取 beanFactory
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 判断容器中是否存在 messageSource 这个beanName
    // 存在的情况
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        // 获取 messageSource 对象
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);

        // 设置 父 MessageSource
        // Make MessageSource aware of parent MessageSource.
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                // Only set parent context as parent MessageSource if no parent MessageSource
                // registered already.
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Using MessageSource [" + this.messageSource + "]");
        }
    }
    // 不存在的情况
    else {
        // 创建空的 MessageSource 实现类
        // Use empty MessageSource to be able to accept getMessage calls.
        DelegatingMessageSource dms = new DelegatingMessageSource();
        // 设置父 MessageSource
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        // 注册 MessageSource
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + MESSAGE_SOURCE_BEAN_NAME + "' bean, using [" + this.messageSource + "]");
        }
    }
}
```

</details>







### initApplicationEventMulticaster

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#initApplicationEventMulticaster`
- 方法作用: 实例化应用事件传播器, 并设置到成员变量中



方法逻辑

1. 判断 BeanName(**`applicationEventMulticaster`**) 是否存在对应的 bean实例
   1. 存在直接从容器中获取并设置给成员变量`applicationEventMulticaster`
   2. 不存在创建`SimpleApplicationEventMulticaster` 设置到成员变量`applicationEventMulticaster`并放入容器





<details>
    <summary>initApplicationEventMulticaster 方法详情</summary>





```java
protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 存在的情况
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
        this.applicationEventMulticaster =
                beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
        if (logger.isTraceEnabled()) {
            logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
        }
    }
    // 不存在的情况
    else {
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        // 注册到容器
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
                    "[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
        }
    }
}
```

</details>







### onRefresh

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#onRefresh`
- 子类实现具体功能

```java
protected void onRefresh() throws BeansException {
    // For subclasses: do nothing by default.
}
```





### registerListeners

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#registerListeners`
- 方法作用: 注册监听器



方法逻辑

1. 获取当前类中已存在的应用监听器列表，向应用事件广播器中添加应用监听器
2. 根据类型获取应用监听器的名称列表, 向应用事件广播其中添加监听器名称
3. 早期应用事件发布



<details>
    <summary>registerListeners 方法详情</summary>





```java
protected void registerListeners() {
    // 获取 应用监听器列表
    // Register statically specified listeners first.
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        // 获取事件广播器
        // 添加应用监听器
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    // 通过类型获取 应用监听器名称列表
    // Do not initialize FactoryBeans here: We need to leave all regular beans
    // uninitialized to let post-processors apply to them!
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    // 将 应用监听器列表的名称注册到 事件广播器中
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    // 早期应用事件发布
    // Publish early application events now that we finally have a multicaster...
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (earlyEventsToProcess != null) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            // 发布事件
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```

</details>







### finishBeanFactoryInitialization

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization`

- 方法作用: 完成当前上下文的beanFactory的初始化, 加载单例bean对象





<details>
    <summary>finishBeanFactoryInitialization 方法详情</summary>





```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 判断是否存在转换服务
    //  1. 转换服务的beanName存在
    //  2. 转换服务的beanName 和 类型是否匹配
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
            beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        // 注册转换服务
        beanFactory.setConversionService(
                beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // 添加嵌套值解析器, 字符串解析其
    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // 将类型是 LoadTimeWeaverAware 的bean全部初始化
    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // 删除临时类加载器
    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // 冻结部分配置
    // Allow for caching all bean definition metadata, not expecting further changes.
    beanFactory.freezeConfiguration();

    // 非懒加载的单例对象实例化
    // Instantiate all remaining (non-lazy-init) singletons.
    beanFactory.preInstantiateSingletons();
}
```

</details>





方法具体操作

1. 转换服务设置, (`ConversionService`)
2. 嵌套式的值解析器(`StringValueResolver`)
3. 关于类型`LoadTimeWeaverAware`  的bean实例化
4. 将临时类加载器设置为null
5. 冻结配置
6. 非懒加载的单例对象实例化



在这个流程中我们需要了解

- **冻结配置是冻结什么？**. 答案: **冻结BeanDefinition名称**， 这些冻结的beanDefinitionName是我们通过xml中`<bean/>` 定义的 beanName
- **非懒加载的单例对象实例化实例化那些单例对象?** 答案: 实例化xml中通过`<bean/>`或者是`@Bean` 注解修饰的对象



关于方法`preInstantiateSingletons`的具体实现可以查看[这篇文章](/docs/beans/factory/BeanFactory/support/BeanDefinitionRegistry/Spring-DefaultListableBeanFactory.md)





### finishRefresh

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#finishRefresh`



- 方法作用: 完成刷新



方法流程:

1. 清空资源缓存

   将资源容器`resourceCaches` 清空

2. 实例化生命周期处理接口

3. 生命周期处理接口执行刷新操作

   生命周期处理接口分析:[`LifecycleProcessor`](/docs/core/Lifecycle/LifecycleProcessor/Spring-DefaultLifecycleProcessor.md)

4. 发布上下文刷新事件

5. 注册应用上下文



<details>
    <summary>finishRefresh 方法详情</summary>







```java
protected void finishRefresh() {
    // 清空资源缓存
    // Clear context-level resource caches (such as ASM metadata from scanning).
    clearResourceCaches();

    // 实例化生命周期处理接口
    // Initialize lifecycle processor for this context.
    initLifecycleProcessor();

    // 生命周期处理接口进行刷新操作
    // Propagate refresh to lifecycle processor first.
    getLifecycleProcessor().onRefresh();

    // 推送事件: 上下文刷新事件
    // Publish the final event.
    publishEvent(new ContextRefreshedEvent(this));

    // 注册应用上下文
    // Participate in LiveBeansView MBean, if active.
    LiveBeansView.registerApplicationContext(this);
}
```

</details>







### initLifecycleProcessor

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#initLifecycleProcessor`

- 方法作用: 实例化生命周期接口

方法流程:

1. 判断容器中是否存在beanName(`lifecycleProcessor`)的bean实例
   1. 存在: 从容器中获取并设置给成员变量(`lifecycleProcessor`)
   2. 不存在: 创建默认的生命周期处理接口实现类(`DefaultLifecycleProcessor`),设置给成员变量(`lifecycleProcessor`)并注册到容器中



<details>
    <summary>initLifecycleProcessor 方法详情</summary>





```java
protected void initLifecycleProcessor() {
    // 获取 beanFactory
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 判断 lifecycleProcessor beanName 是否有对应的 bean 实例
    if (beanFactory.containsLocalBean(LIFECYCLE_PROCESSOR_BEAN_NAME)) {
        // 设置 lifecycleProcessor
        this.lifecycleProcessor =
                beanFactory.getBean(LIFECYCLE_PROCESSOR_BEAN_NAME, LifecycleProcessor.class);
        if (logger.isTraceEnabled()) {
            logger.trace("Using LifecycleProcessor [" + this.lifecycleProcessor + "]");
        }
    }
    else {
        // 创建默认的 生命周期处理接口的实现都西昂
        DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();
        // 设置 beanFactory
        defaultProcessor.setBeanFactory(beanFactory);
        // 设置成员变量
        this.lifecycleProcessor = defaultProcessor;
        // 注册
        beanFactory.registerSingleton(LIFECYCLE_PROCESSOR_BEAN_NAME, this.lifecycleProcessor);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + LIFECYCLE_PROCESSOR_BEAN_NAME + "' bean, using " +
                    "[" + this.lifecycleProcessor.getClass().getSimpleName() + "]");
        }
    }
}
```

</details>









### destroyBeans

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#destroyBeans`
- 方法作用: 摧毁bean





```java
protected void destroyBeans() {
    getBeanFactory().destroySingletons();
}
```



- 最终实现摧毁方法调用的类
  1. `DefaultListableBeanFactory`:[分析文章](/docs/beans/factory/BeanFactory/support/BeanDefinitionRegistry/Spring-DefaultListableBeanFactory.md)
  2. `DefaultSingletonBeanRegistry`: [分析文章](/docs/core/registry/Spring-DefaultSingletonBeanRegistry.md)









### cancelRefresh

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#cancelRefresh`



- 方法作用: 设置激活状态为 false



```java
protected void cancelRefresh(BeansException ex) {
    this.active.set(false);
}
```







### resetCommonCaches 

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#resetCommonCaches`
- 方法作用: 重置通用缓存





```java
protected void resetCommonCaches() {
    // 反射缓存
    ReflectionUtils.clearCache();
    // 注解缓存
    AnnotationUtils.clearCache();
    // 类型解析器缓存
    ResolvableType.clearCache();
    CachedIntrospectionResults.clearClassLoader(getClassLoader());
}
```





- 至此我们将`refresh`中的调用方法都分析完成. 下面是其他方法的分析



### registerShutdownHook

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#registerShutdownHook`

- 方法作用: 
  1. 定义关闭的线程
  2. 注册关闭线程到Java容器中





<details>
    <summary>registerShutdownHook 方法详情</summary>



```java
@Override
public void registerShutdownHook() {
    if (this.shutdownHook == null) {
        // No shutdown hook registered yet.
        this.shutdownHook = new Thread(SHUTDOWN_HOOK_THREAD_NAME) {
            @Override
            public void run() {
                synchronized (startupShutdownMonitor) {
                    // 真正的关闭方法
                    doClose();
                }
            }
        };
        // jdk 提供的关闭应用时触发的钩子线程
        Runtime.getRuntime().addShutdownHook(this.shutdownHook);
    }
}
```

</details>





### close

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#close`

<details>
    <summary>close 方法详情</summary>





```
@Override
public void close() {
    synchronized (this.startupShutdownMonitor) {
        // 执行关闭
        doClose();
        // If we registered a JVM shutdown hook, we don't need it anymore now:
        // We've already explicitly closed the context.
        if (this.shutdownHook != null) {
            try {
                // 移除 关闭线程
                Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
            }
            catch (IllegalStateException ex) {
                // ignore - VM is already shutting down
            }
        }
    }
}
```

</details>







### doClose

- 方法签名: `org.springframework.context.support.AbstractApplicationContext#doClose`



方法流程:

1. 在容器中移除应用上下文
2. 发布关闭上下文事件
3. 生命周期处理接口执行关闭方法
4. 摧毁bean
5. 摧毁beanFactory
6. 设置激活状态为false



<details>
    <summary>doClose 方法详情</summary>





```java
protected void doClose() {
    // Check whether an actual close attempt is necessary...
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
        if (logger.isDebugEnabled()) {
            logger.debug("Closing " + this);
        }

        // 在容器中移除当前上下文
        LiveBeansView.unregisterApplicationContext(this);

        try {
            // 发布关闭上下文事件
            // Publish shutdown event.
            publishEvent(new ContextClosedEvent(this));
        }
        catch (Throwable ex) {
            logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
        }

        // Stop all Lifecycle beans, to avoid delays during individual destruction.
        if (this.lifecycleProcessor != null) {
            try {
                // 生命周期处理器执行关闭函数
                this.lifecycleProcessor.onClose();
            }
            catch (Throwable ex) {
                logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
            }
        }

        // 摧毁bean
        // Destroy all cached singletons in the context's BeanFactory.
        destroyBeans();

        // 关闭 beanFactory
        // Close the state of this context itself.
        closeBeanFactory();

        // Let subclasses do some final clean-up if they wish...
        // 子类拓展关闭相关方法
        onClose();

        // Reset local application listeners to pre-refresh state.
        if (this.earlyApplicationListeners != null) {
            this.applicationListeners.clear();
            this.applicationListeners.addAll(this.earlyApplicationListeners);
        }

        // 设置激活状态为 false
        // Switch to inactive.
        this.active.set(false);
    }
}
```

</details>