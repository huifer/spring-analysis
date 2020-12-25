# Spring AbstractRefreshableApplicationContext
- 类全路径: `org.springframework.context.support.AbstractRefreshableApplicationContext`
- 可刷新的应用上下文(抽象类)


## 成员变量

```java
	/**
	 *  Synchronization monitor for the internal BeanFactory.
	 *
	 * beanFactory 锁
	 * */
	private final Object beanFactoryMonitor = new Object();

	/**
	 * 是否允许 bean定义覆盖
	 */
	@Nullable
	private Boolean allowBeanDefinitionOverriding;

	/**
	 * 是否允许循环引用
	 */
	@Nullable
	private Boolean allowCircularReferences;

	/**
	 *  Bean factory for this context.
	 *  BeanFactory
	 * */
	@Nullable
	private DefaultListableBeanFactory beanFactory;

```




## 方法分析

### refreshBeanFactory
- 方法签名: `org.springframework.context.support.AbstractRefreshableApplicationContext.refreshBeanFactory`

- 方法作用: 刷新 BeanFactory 

方法逻辑:
    1. 如果beanFactory已经存在
        1. 摧毁 Bean
        1. 清空 BeanFactory
    1. 创建 BeanFactory
    1. 定制工厂的处理
        1. 设置 allowBeanDefinitionOverriding
        1. 设置 allowCircularReferences
    1. 加载bean定义


<details>

<summary>refreshBeanFactory 方法详情</summary>

```java
	@Override
	protected final void refreshBeanFactory() throws BeansException {
		// 是否存在 beanFactory
		if (hasBeanFactory()) {
			// 如果存在 beanFactory 则清空 bean 相关信息
			// 摧毁bean
			destroyBeans();
			// 清空 beanFactory
			closeBeanFactory();
		}
		try {
			// 创建 beanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			// 设置序列化id
			beanFactory.setSerializationId(getId());
			// 定制工厂的处理
			// 设置两个属性值
			// 	1. allowBeanDefinitionOverriding
			//  2. allowCircularReferences
			customizeBeanFactory(beanFactory);
			// 加载 bean定义
			loadBeanDefinitions(beanFactory);
			// 上锁设置 beanFactory
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}

```


</details>
