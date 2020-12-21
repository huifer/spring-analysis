# Spring NamespaceHandlerResolver
- 类全路径: `org.springframework.beans.factory.xml.NamespaceHandlerResolver`
- 类作用: 命名空间处理器解析

- `NamespaceHandlerResolver` 作为借口在本文仅了解方法作用即可. 详细实现类请看[DefaultNamespaceHandlerResolver](Spring-DefaultNamespaceHandlerResolver.md)

```java
@FunctionalInterface
public interface NamespaceHandlerResolver {

	/**
	 * Resolve the namespace URI and return the located {@link NamespaceHandler}
	 * implementation.
	 *
	 * 解析命名空间的 url 获得 命名空间处理器
	 * @param namespaceUri the relevant namespace URI
	 * @return the located {@link NamespaceHandler} (may be {@code null})
	 */
	@Nullable
	NamespaceHandler resolve(String namespaceUri);

}

```