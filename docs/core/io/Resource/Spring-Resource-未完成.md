# Spring Resource
- 类全路径: `org.springframework.core.io.Resource`


- `Resource` 是一个接口, 在这仅作方法作用介绍. 

## 详细代码


```java
public interface Resource extends InputStreamSource {

	/**
	 * 是否存在
	 */
	boolean exists();

	/**
	 * 是否可读
	 */
	default boolean isReadable() {
		return exists();
	}

	/**
	 * 是否打开
	 */
	default boolean isOpen() {
		return false;
	}

	/**
	 * 是否是文件类型
	 */
	default boolean isFile() {
		return false;
	}

	/**
	 * 获取 URL 对象
	 */
	URL getURL() throws IOException;

	/**
	 * 获取 URI 对象
	 */
	URI getURI() throws IOException;

	/**
	 * 获取文件
	 */
	File getFile() throws IOException;

	/**
	 * 可读字节通道
	 */
	default ReadableByteChannel readableChannel() throws IOException {
		return Channels.newChannel(getInputStream());
	}

	/**
	 * 内容长度
	 */
	long contentLength() throws IOException;

	/**
	 * 最后修改实践
	 */
	long lastModified() throws IOException;

	/**
	 * 创建资源
	 */
	Resource createRelative(String relativePath) throws IOException;

	/**
	 * 获取文件名称
	 */
	@Nullable
	String getFilename();

	/**
	 * 获取资源描述
	 */
	String getDescription();

}
```