# Spring ApplicationContext
- 类全路径: `org.springframework.context.ApplicationContext`
- 应用上下文接口

- `ApplicationContext` 作为借口本文仅介绍其提供的方法定义


```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

	/**
	 * 应用上下文 id
	 */
	@Nullable
	String getId();

	/**
	 * 应用名称
	 */
	String getApplicationName();

	/**
	 * 显示名称
	 */
	String getDisplayName();

	/**
	 * 启动时间
	 */
	long getStartupDate();

	/**
	 * 父应用上下文
	 */
	@Nullable
	ApplicationContext getParent();

	/**
	 * 获取 自动注入的bean工厂
	 */
	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;

}
```