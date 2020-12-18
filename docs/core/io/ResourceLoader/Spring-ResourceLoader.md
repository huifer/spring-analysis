# Spring ResourceLoader
- 类全路径: `org.springframework.core.io.ResourceLoader`

- `ResourceLoader` 资源加载器. 在本文仅作方法和内部类的介绍, 请各位阅读下方代码进行了解.


```java

public interface ResourceLoader {

	/**
	 * classPath 地址前缀
	 */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;


	/**
	 * 根据地址获取资源对象
	 */
	Resource getResource(String location);

	/**
	 * 获取类加载器
	 */
	@Nullable
	ClassLoader getClassLoader();

}


```
