# Spring BeanDefinitionRegistry
- 类全路径: `org.springframework.beans.factory.support.BeanDefinitionRegistry`

- `BeanDefinitionRegistry` 类作用: 提供 BeanDefinition 处理行为

```java
public interface BeanDefinitionRegistry extends AliasRegistry {

	/**
	 *
	 * 注册beanDefinition
	 */
	void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException;

	/**
	 * 移除bean的定义
	 */
	void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 获取bean定义
	 */
	BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;

	/**
	 * 是否存在 beanDefinition
	 */
	boolean containsBeanDefinition(String beanName);

	/**
	 * 获取所有的beanDefinition的名称(beanName列表)
	 */
	String[] getBeanDefinitionNames();

	/**
	 * 获取 beanDefinition 的数量
	 */
	int getBeanDefinitionCount();

	/**
	 * beanName是否被使用
	 */
	boolean isBeanNameInUse(String beanName);

}
```