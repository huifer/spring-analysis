# Spring ProblemReporter
- 类全路径: `org.springframework.beans.factory.parsing.ProblemReporter`
- 问题报告记录器

- `ProblemReporter` 问题报告记录器主要记录下面三种级别的问题
  1. fatal: 致命问题
  2. error: 异常问题
  3. warning: 警告问题



- 详细代码如下

```java
public interface ProblemReporter {

   /**
    * Called when a fatal error is encountered during the parsing process.
    * <p>Implementations must treat the given problem as fatal,
    * i.e. they have to eventually raise an exception.
    * 致命信息记录
    * @param problem the source of the error (never {@code null})
    */
   void fatal(Problem problem);

   /**
    * Called when an error is encountered during the parsing process.
    * <p>Implementations may choose to treat errors as fatal.
    * 异常信息记录
    * @param problem the source of the error (never {@code null})
    */
   void error(Problem problem);

   /**
    * Called when a warning is raised during the parsing process.
    * <p>Warnings are <strong>never</strong> considered to be fatal.
    *
    * 警告信息记录
    * @param problem the source of the warning (never {@code null})
    */
   void warning(Problem problem);

}
```





在整个类中围绕`Problem` 进行, 我们也需要对`Problem`有一定的了解, 详细分析查看[这篇文章](/docs/beans/factory/parsing/ProblemReporter/Spring-Problem.md)