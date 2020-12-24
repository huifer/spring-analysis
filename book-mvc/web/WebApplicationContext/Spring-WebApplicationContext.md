# Spring WebApplicationContext
- 类全路径: `org.springframework.web.context.WebApplicationContext`
- web应用上下文




```java
public interface WebApplicationContext extends ApplicationContext {

	/**
	 * web 程序的应用上下文属性
	 */
	String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";

	/**
	 * 作用域: request
	 */
	String SCOPE_REQUEST = "request";

	/**
	 * 作用域: session
	 */
	String SCOPE_SESSION = "session";

	/**
	 * 作用域: application
	 */
	String SCOPE_APPLICATION = "application";

	/**
	 * servlet上下文名称 (bean name)
	 */
	String SERVLET_CONTEXT_BEAN_NAME = "servletContext";

	/**
	 * servlet 上下文参数名称(bean name).
	 */
	String CONTEXT_PARAMETERS_BEAN_NAME = "contextParameters";

	/**
	 * servlet 上下文属性名称(bean name)
	 */
	String CONTEXT_ATTRIBUTES_BEAN_NAME = "contextAttributes";


	/**
	 * 获取 servlet 上下文
	 */
	@Nullable
	ServletContext getServletContext();

}
```