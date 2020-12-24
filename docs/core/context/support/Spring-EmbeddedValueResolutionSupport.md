# Spring EmbeddedValueResolutionSupport
- 类全路径: `org.springframework.context.support.EmbeddedValueResolutionSupport`


```java
public class EmbeddedValueResolutionSupport implements EmbeddedValueResolverAware {

	@Nullable
	private StringValueResolver embeddedValueResolver;


	@Override
	public void setEmbeddedValueResolver(StringValueResolver resolver) {
		this.embeddedValueResolver = resolver;
	}

	/**
	 * Resolve the given embedded value through this instance's {@link StringValueResolver}.
	 * @param value the value to resolve
	 * @return the resolved value, or always the original value if no resolver is available
	 * @see #setEmbeddedValueResolver
	 */
	@Nullable
	protected String resolveEmbeddedValue(String value) {
		// 通过 StringValueResolver 解析字符串, 或者直接返回
		return (this.embeddedValueResolver != null ? this.embeddedValueResolver.resolveStringValue(value) : value);
	}


}
```