# Spring LifecycleProcessor 
- 类全路径: `org.springframework.context.LifecycleProcessor`

- `LifecycleProcessor`: 生命周期处理接口, 即在生命周期的某些部分进行一些处理, 某些部分是指 刷新和关闭. 


```java
public interface LifecycleProcessor extends Lifecycle {

	/**
	 * Notification of context refresh, e.g. for auto-starting components.
	 */
	void onRefresh();

	/**
	 * Notification of context close phase, e.g. for auto-stopping components.
	 */
	void onClose();

}
```


- 在 Spring 中 `LifecycleProcessor` 仅有 `DefaultLifecycleProcessor` 实现