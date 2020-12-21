# Spring Environment
- 类全路径: `org.springframework.core.env.Environment`

- `Environment` 作为接口在这只对方法作用进行描述

```java
public interface Environment extends PropertyResolver {

	/**
	 * 获取已经激活的文件列表
	 */
	String[] getActiveProfiles();

	/**
	 * 获取默认的文件列表
	 */
	String[] getDefaultProfiles();

	/**
	 * profiles 是否在 激活的 profile 中.
	 */
	@Deprecated
	boolean acceptsProfiles(String... profiles);

	/**
	 * profiles 是否激活
	 */
	boolean acceptsProfiles(Profiles profiles);

}
```


- [ConfigurableEnvironment](/doc/book/env/environment/Spring-ConfigurableEnvironment.md)
    - [ConfigurableWebEnvironment](/doc/book/env/environment/Spring-ConfigurableWebEnvironment.md)
        - [StandardServletEnvironment](/doc/book/env/environment/Spring-StandardServletEnvironment.md)
    - [AbstractEnvironment](/doc/book/env/environment/Spring-AbstractEnvironment.md)
        - [StandardEnvironment](/doc/book/env/environment/Spring-StandardEnvironment.md)
            - [StandardServletEnvironment](/doc/book/env/environment/Spring-StandardServletEnvironment.md)
        - [MockEnvironment](/doc/book/env/environment/Spring-MockEnvironment.md)
