# Spring DefaultResourceLoader
- 类全路径: `org.springframework.core.io.DefaultResourceLoader`

- 首先来阅读成员变量内容. 

## 成员变量


```java
public class DefaultResourceLoader implements ResourceLoader {
    @Nullable
    private ClassLoader classLoader;

    /**
     * 协议解析器列表
     */
    private final Set<ProtocolResolver> protocolResolvers = new LinkedHashSet<>(4);

    /**
     * 资源缓存
     */
    private final Map<Class<?>, Map<Resource, ?>> resourceCaches = new ConcurrentHashMap<>(4);
}	
```

- 在了解成员变量后我们来看 `getResource` 方法

## 方法分析

### getResource
- 方法签名: `org.springframework.core.io.DefaultResourceLoader.getResource`
- 方法作用: 根据资源地址获取资源接口`Resource`



<details>
<summary>getResource 方法详情</summary>

```java
@Override
public Resource getResource(String location) {
   Assert.notNull(location, "Location must not be null");

   // 获取协议解析器列表 循环
   for (ProtocolResolver protocolResolver : getProtocolResolvers()) {
      Resource resource = protocolResolver.resolve(location, this);
      if (resource != null) {
         return resource;
      }
   }

   // 路径地址是 / 开头
   if (location.startsWith("/")) {
      return getResourceByPath(location);
   }
   // 地址路径是 classpath: 开头
   else if (location.startsWith(CLASSPATH_URL_PREFIX)) {
      return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
   }
   else {
      try {
         // Try to parse the location as a URL...
         // 尝试将 location 转换成 url 进行读取
         URL url = new URL(location);
         return (ResourceUtils.isFileURL(url) ? new FileUrlResource(url) : new UrlResource(url));
      }
      catch (MalformedURLException ex) {
         // No URL -> resolve as resource path.
         return getResourceByPath(location);
      }
   }
}
```

</details>



在`getResource` 中提供了4种处理方式， 四种处理方式有四种处理方法.

1. 协议解析器解析,对应接口

   通过`ProtocolResolver`接口方法进行解析后返回

2. 处理 `/`开头的路径

   通过`getResourceByPath`方法进行解析后返回

3. 处理 `classpath:`开头的路径

   通过 `new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader())`  将对象创建后返回

4. 尝试使用 URL 进行创建地址后解析

   既然是尝试性的就有可能失败, 面对失败的情况 Spring 将器使用`getResourceByPath`方法进行处理

   如果尝试成功则包含两种处理方式, 两种处理方式的前提是是否为url文件协议(**url文件协议是指`file`、`vfsfile`、`vfs`三种协议**).

   1. `FileUrlResource`对象创建
   2. `UrlResource`对象创建







### getResourceByPath

- 方法签名: `org.springframework.core.io.DefaultResourceLoader#getResourceByPath`



- 方法作用: 创建`ClassPathContextResource`对象.





```java
protected Resource getResourceByPath(String path) {
   return new ClassPathContextResource(path, getClassLoader());
}


/**
 * ClassPathResource that explicitly expresses a context-relative path
 * through implementing the ContextResource interface.
 */
protected static class ClassPathContextResource extends ClassPathResource implements ContextResource {

   public ClassPathContextResource(String path, @Nullable ClassLoader classLoader) {
      super(path, classLoader);
   }

   @Override
   public String getPathWithinContext() {
      return getPath();
   }

   @Override
   public Resource createRelative(String relativePath) {
      String pathToUse = StringUtils.applyRelativePath(getPath(), relativePath);
      return new ClassPathContextResource(pathToUse, getClassLoader());
   }
}
```











`getResource`法返回值涉及到`Resource`接口. 详细可以阅读[这篇文章](/doc/book/core/io/Resource/readme.md)，着重可以关注下面几个类

- ClassPathResource
- FileUrlResource
- UrlResource