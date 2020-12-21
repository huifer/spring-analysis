# Spring ConfigurableWebEnvironment
- 类全路径: `org.springframework.web.context.ConfigurableWebEnvironment`


- `ConfigurableWebEnvironment` 作为接口在这只对方法作用进行描述



```java
public interface ConfigurableWebEnvironment extends ConfigurableEnvironment {

	/**
	 * 初始化属性
	 */
	void initPropertySources(@Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig);

}
```