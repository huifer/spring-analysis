# Spring ConfigurableListableBeanFactory
- 类全路径: `org.springframework.beans.factory.config.ConfigurableListableBeanFactory`



- `ConfigurableListableBeanFactory` 作为接口本文仅作方法作用介绍


```java

public interface ConfigurableListableBeanFactory
		extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {

	/**
	 * 设置忽略的依赖类型
	 */
	void ignoreDependencyType(Class<?> type);

	/**
	 * 设置忽略的依赖接口
	 */
	void ignoreDependencyInterface(Class<?> ifc);

	/**
	 * 注册依赖
	 */
	void registerResolvableDependency(Class<?> dependencyType, @Nullable Object autowiredValue);

	/**
	 * 是否是依赖候选对象
	 */
	boolean isAutowireCandidate(String beanName, DependencyDescriptor descriptor)
			throws NoSuchBeanDefinitionException;

	/**
	 * 获取bean定义
	 */
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 获取 beanName 迭代器
	 *
	 */
	Iterator<String> getBeanNamesIterator();

	/**
	 * 清空元数据缓存
	 */
	void clearMetadataCache();

	/**
	 * 冻结 配置
	 */
	void freezeConfiguration();

	/**
	 * 配置是否冻结
	 */
	boolean isConfigurationFrozen();

	/**
	 * 实例化单例对象
	 */
	void preInstantiateSingletons() throws BeansException;

}
```