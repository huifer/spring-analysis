# Spring ClassPathXmlApplicationContext
- 类全路径: `org.springframework.context.support.ClassPathXmlApplicationContext`


## 构造函数


```java
	public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		// 设置本地配置信息
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

```


- 关于 `refresh` 方法查看[这篇文章](/docs/core/context/ApplicationContext/Spring-AbstractApplicationContext.md)