# Spring FileSystemXmlApplicationContext
- 类全路径: `org.springframework.context.support.FileSystemXmlApplicationContext`


## 构造函数

```java
	public FileSystemXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}

```

- 关于 `refresh` 方法查看[这篇文章](/docs/core/context/ApplicationContext/Spring-AbstractApplicationContext.md)