# Spring Problem
- 类全路径: `org.springframework.beans.factory.parsing.Problem`
- `Problem`: 问题对象. 



对于`Problem` 对象我们先看成员变量. 类本身没有什么复杂





```java
public class Problem {

   /**
    * 问题信息
    */
   private final String message;

   /**
    * 问题发生的地方
    */
   private final Location location;

   /**
    * 状态, 存储堆栈信息. 
    */
   @Nullable
   private final ParseState parseState;

   /**
    * 异常
    */
   @Nullable
   private final Throwable rootCause;
}
```





- 在成员变量中我们对两个类型不是很了解
  1. `ParseState`
  2. `Location`



我们先对 `ParseState` 进行说明



## ParseState
- 类全路径: `org.springframework.beans.factory.parsing.ParseState`



- `ParseState` 主要用来记录每个阶段的 Entry 对象 , 当出现问题的时候使用 toString 进行输出



```java
public final class ParseState {

   /**
    * Tab character used when rendering the tree-style representation.
    */
   private static final char TAB = '\t';

   /**
    * Internal {@link LinkedList} storage.
    * 存储每个阶段的 Entry 对象
    */
   private final LinkedList<Entry> state;
    
    
    public String toString() {
		StringBuilder sb = new StringBuilder();
		for (int x = 0; x < this.state.size(); x++) {
			if (x > 0) {
				sb.append('\n');
				for (int y = 0; y < x; y++) {
					sb.append(TAB);
				}
				sb.append("-> ");
			}
			sb.append(this.state.get(x));
		}
		return sb.toString();
	}
}
```



- 在知道存储的是`Entry`接口后我们需要了解在Spring中有那些阶段状态, 即`Entry`的实现类



## Entry

- 标识spring的各类阶段, 详细阶段信息如下
  - AspectEntry: aspect 阶段
  - QualifierEntry: 自动装配候选阶段
  - PropertyEntry: 属性设置阶段
  - ConstructorArgumentEntry: 构造函数阶段
  - AdviceEntry: 通知阶段 advice
  - AdvisorEntry: advisor阶段
  - PointcutEntry: 切面阶段
  - BeanEntry: bean定义解析阶段









## Location





```java
public class Location {
   /**
    * 资源对象
    */
   private final Resource resource;

   /**
    * 源数据
    */
   @Nullable
   private final Object source;
}
```