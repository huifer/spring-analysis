# Spring ReaderContext
- 类全路径: `org.springframework.beans.factory.parsing.ReaderContext`
- bean 定义读取的上下文
- 在 `ReaderContext` 没有直接的操作方法, 更多的是声名流程, 封装通用代码. 例如对异常报告的封装, 读取bean定义的事件封装. 在这个类中我们需要对成员变量有一个清晰的认识, 下面是成员变量的分析
  1. Resource: [分析文章](/docs/core/io/Resource/Spring-Resource-未完成.md)
  2. ProblemReporter: [分析文章](/doc/book/bean/factory/parsing/ProblemReporter/Spring-ProblemReporter.md)
  3. ReaderEventListener: [分析文章](/doc/book/event/Spring_ReaderEventListener-未完成.md)
  4. SourceExtractor: [分析文章](/doc/book/bean/factory/parsing/SourceExtractor/Spring-SourceExtractor.md)







## 完整代码



<details>
<summary>完整代码</summary>

```java
public class ReaderContext {

   /**
    * 资源对象
    */
   private final Resource resource;

   /**
    * 问题报告接口
    */
   private final ProblemReporter problemReporter;

   /**
    * 读取的事件监听器
    */
   private final ReaderEventListener eventListener;

   /**
    * 元数据提取接口
    */
   private final SourceExtractor sourceExtractor;


   /**
    * Construct a new {@code ReaderContext}.
    *
    * @param resource        the XML bean definition resource
    * @param problemReporter the problem reporter in use
    * @param eventListener   the event listener in use
    * @param sourceExtractor the source extractor in use
    */
   public ReaderContext(Resource resource, ProblemReporter problemReporter,
         ReaderEventListener eventListener, SourceExtractor sourceExtractor) {

      this.resource = resource;
      this.problemReporter = problemReporter;
      this.eventListener = eventListener;
      this.sourceExtractor = sourceExtractor;
   }

   public final Resource getResource() {
      return this.resource;
   }


   // Errors and warnings

   /**
    * Raise a fatal error.
    */
   public void fatal(String message, @Nullable Object source) {
      fatal(message, source, null, null);
   }

   /**
    * Raise a fatal error.
    */
   public void fatal(String message, @Nullable Object source, @Nullable Throwable cause) {
      fatal(message, source, null, cause);
   }

   /**
    * Raise a fatal error.
    */
   public void fatal(String message, @Nullable Object source, @Nullable ParseState parseState) {
      fatal(message, source, parseState, null);
   }

   /**
    * Raise a fatal error.
    */
   public void fatal(String message, @Nullable Object source, @Nullable ParseState parseState, @Nullable Throwable cause) {
      Location location = new Location(getResource(), source);
      this.problemReporter.fatal(new Problem(message, location, parseState, cause));
   }

   /**
    * Raise a regular error.
    */
   public void error(String message, @Nullable Object source) {
      error(message, source, null, null);
   }

   /**
    * Raise a regular error.
    */
   public void error(String message, @Nullable Object source, @Nullable Throwable cause) {
      error(message, source, null, cause);
   }

   /**
    * Raise a regular error.
    */
   public void error(String message, @Nullable Object source, @Nullable ParseState parseState) {
      error(message, source, parseState, null);
   }

   /**
    * Raise a regular error.
    */
   public void error(String message, @Nullable Object source, @Nullable ParseState parseState, @Nullable Throwable cause) {
      Location location = new Location(getResource(), source);
      this.problemReporter.error(new Problem(message, location, parseState, cause));
   }

   /**
    * Raise a non-critical warning.
    */
   public void warning(String message, @Nullable Object source) {
      warning(message, source, null, null);
   }

   /**
    * Raise a non-critical warning.
    */
   public void warning(String message, @Nullable Object source, @Nullable Throwable cause) {
      warning(message, source, null, cause);
   }

   /**
    * Raise a non-critical warning.
    */
   public void warning(String message, @Nullable Object source, @Nullable ParseState parseState) {
      warning(message, source, parseState, null);
   }

   /**
    * Raise a non-critical warning.
    */
   public void warning(String message, @Nullable Object source, @Nullable ParseState parseState, @Nullable Throwable cause) {
      Location location = new Location(getResource(), source);
      this.problemReporter.warning(new Problem(message, location, parseState, cause));
   }


   // Explicit parse events

   /**
    * Fire a defaults-registered event.
    */
   public void fireDefaultsRegistered(DefaultsDefinition defaultsDefinition) {
      this.eventListener.defaultsRegistered(defaultsDefinition);
   }

   /**
    * Fire a component-registered event.
    */
   public void fireComponentRegistered(ComponentDefinition componentDefinition) {
      this.eventListener.componentRegistered(componentDefinition);
   }

   /**
    * Fire an alias-registered event.
    * 执行 别名注册事件
    * @param beanName beanName
    * @param alias  bean 别名
    * @param source
    */
   public void fireAliasRegistered(String beanName, String alias, @Nullable Object source) {
      this.eventListener.aliasRegistered(new AliasDefinition(beanName, alias, source));
   }

   /**
    * Fire an import-processed event. 触发import 事件
    */
   public void fireImportProcessed(String importedResource, @Nullable Object source) {
      /**
       * 事件监听器importProcessed通知处理结果
       */
      this.eventListener.importProcessed(new ImportDefinition(importedResource, source));
   }

   /**
    * Fire an import-processed event.
    *
    * 执行 import 事件
    * @param importedResource 导入的文件
    * @param actualResources 资源路径
    * @param source source
    */
   public void fireImportProcessed(String importedResource, Resource[] actualResources, @Nullable Object source) {
      this.eventListener.importProcessed(new ImportDefinition(importedResource, actualResources, source));
   }


   // Source extraction

   /**
    * Return the source extractor in use.
    */
   public SourceExtractor getSourceExtractor() {
      return this.sourceExtractor;
   }

   /**
    * Call the source extractor for the given source object.
    *
    * @param sourceCandidate the original source object
    *
    * @return the source object to store, or {@code null} for none.
    *
    * @see #getSourceExtractor()
    * @see SourceExtractor#extractSource
    */
   @Nullable
   public Object extractSource(Object sourceCandidate) {
      return this.sourceExtractor.extractSource(sourceCandidate, this.resource);
   }

}
```

</details>

- 对于 `ReaderContext` 的分析就到这里结束了, 在 Spring 中有 `XmlReaderContext` 子类继承 `ReaderContext` 提供了更多的方法. 有关分析请查看[这篇文章](Spring-XmlReaderContext.md)