# Spring ConfigurableApplicationContext
- 类全路径: `org.springframework.context.ConfigurableApplicationContext`
- 类作用: 配置应用上下文

- `ConfigurableApplicationContext` 作为接口在本文仅对方法作用做一个说明


```java
public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {

	/**
	 * 配置文件分隔符
	 */
	String CONFIG_LOCATION_DELIMITERS = ",; \t\n";

	/**
	 * 转换服务名称
	 */
	String CONVERSION_SERVICE_BEAN_NAME = "conversionService";

	/**
	 * LoadTimeWeaver 名称(bean name)
	 */
	String LOAD_TIME_WEAVER_BEAN_NAME = "loadTimeWeaver";

	/**
	 * 环境名称(bean name)
	 */
	String ENVIRONMENT_BEAN_NAME = "environment";

	/**
	 * 系统配置名称(bean name)
	 */
	String SYSTEM_PROPERTIES_BEAN_NAME = "systemProperties";

	/**
	 * 系统环境名称(bean name)
	 */
	String SYSTEM_ENVIRONMENT_BEAN_NAME = "systemEnvironment";

	/**
	 * 钩子线程名称
	 */
	String SHUTDOWN_HOOK_THREAD_NAME = "SpringContextShutdownHook";


	/**
	 * 设置应用id
	 */
	void setId(String id);

	/**
	 * 设置 父上下文
	 */
	void setParent(@Nullable ApplicationContext parent);

	/**
	 * 获取环境配置
	 */
	@Override
	ConfigurableEnvironment getEnvironment();

	/**
	 * 设置环境配置
	 */
	void setEnvironment(ConfigurableEnvironment environment);

	/**
	 * 添加 bean工厂后置处理器 {@link BeanFactoryPostProcessor}
	 */
	void addBeanFactoryPostProcessor(BeanFactoryPostProcessor postProcessor);

	/**
	 * 添加应用监听对象
	 */
	void addApplicationListener(ApplicationListener<?> listener);

	/**
	 * 添加协议解析器
	 */
	void addProtocolResolver(ProtocolResolver resolver);

	/**
	 * 刷新
	 */
	void refresh() throws BeansException, IllegalStateException;

	/**
	 * 注册钩子
	 */
	void registerShutdownHook();

	/**
	 * 关闭应用
	 */
	@Override
	void close();

	/**
	 * 是否存活
	 */
	boolean isActive();

	/**
	 * 获取 bean 工厂
	 */
	ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;

}
```