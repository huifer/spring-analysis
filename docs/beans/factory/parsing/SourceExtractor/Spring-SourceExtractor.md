# Spring SourceExtractor
- 类全路径: `org.springframework.beans.factory.parsing.SourceExtractor`
- 作用: 元数据提取, 主要用于提取 xml 中 element 的数据


```java
@FunctionalInterface
public interface SourceExtractor {

	/**
	 * Extract the source metadata from the candidate object supplied
	 * by the configuration parser.
	 *
	 * 从配置解析器提供的候选对象中提取源元数据。
	 *
	 * @param sourceCandidate the original source metadata (never {@code null})
	 * @param definingResource the resource that defines the given source object
	 * (may be {@code null})
	 * @return the source metadata object to store (may be {@code null})
	 */
	@Nullable
	Object extractSource(Object sourceCandidate, @Nullable Resource definingResource);

}

```

- Spring 有两种实现
    1. `NullSourceExtractor`: 返回为空
    2. `PassThroughSourceExtractor`:  将入参 sourceCandidate 直接返回 