# Spring ConfigurableWebApplicationContext
- 类全路径: `org.springframework.web.context.ConfigurableWebApplicationContext`
- 类作用: 设置 web 应用上下文属性



```java
public interface ConfigurableWebApplicationContext extends WebApplicationContext, ConfigurableApplicationContext {

	/**
	 * 应用上下文 id 前缀
	 */
	String APPLICATION_CONTEXT_ID_PREFIX = WebApplicationContext.class.getName() + ":";

	/**
	 * servlet 配置名称(bean name)
	 */
	String SERVLET_CONFIG_BEAN_NAME = "servletConfig";


	/**
	 * 设置 servlet 上下文
	 */
	void setServletContext(@Nullable ServletContext servletContext);

	/**
	 * 设置 servlet config
	 */
	void setServletConfig(@Nullable ServletConfig servletConfig);

	/**
	 * 获取 servlet config
	 */
	@Nullable
	ServletConfig getServletConfig();

	/**
	 * 设置命名空间
	 */
	void setNamespace(@Nullable String namespace);

	/**
	 * 获取命名空间
	 */
	@Nullable
	String getNamespace();

	/**
	 * 设置配置文件地址
	 */
	void setConfigLocation(String configLocation);

	/**
	 * 设置配置文件地址
	 */
	void setConfigLocations(String... configLocations);

	/**
	 * 获取配置文件地址
	 */
	@Nullable
	String[] getConfigLocations();

}
```