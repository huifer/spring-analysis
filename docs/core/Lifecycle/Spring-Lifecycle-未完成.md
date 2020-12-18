# Spring Lifecycle
- 类全路径: `org.springframework.context.Lifecycle`
- 类图
  ![Lifecycle](./images/Lifecycle.png)

- `Lifecycle`: 生命周期接口. 提供了下面三个方法
    1. 启动
    2. 暂停
    3. 判断是否正在运行
    

```java
public interface Lifecycle {

	/**
	 * 启动方法
	 */
	void start();

	/**
	 *
	 * 暂停方法
	 */
	void stop();

	/**
	 * 判断是否正在运行
	 */
	boolean isRunning();

}
```