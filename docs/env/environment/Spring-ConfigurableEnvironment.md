# Spring ConfigurableEnvironment
- 类全路径: `org.springframework.core.env.ConfigurableEnvironment`

- `ConfigurableEnvironment` 作为接口在这只对方法作用进行描述


```java
public interface ConfigurableEnvironment extends Environment, ConfigurablePropertyResolver {

	/**
	 * 设置激活的 profile
	 */
	void setActiveProfiles(String... profiles);

	/**
	 * 添加激活的 profile
	 */
	void addActiveProfile(String profile);

	/**
	 * 设置默认的 profile
	 */
	void setDefaultProfiles(String... profiles);

	/**
	 * 获取 属性信息
	 */
	MutablePropertySources getPropertySources();

	/**
	 * 获取系统属性
	 */
	Map<String, Object> getSystemProperties();

	/**
	 * 获取系统环境
	 */
	Map<String, Object> getSystemEnvironment();

	/**
	 * 合并
	 */
	void merge(ConfigurableEnvironment parent);

}
```