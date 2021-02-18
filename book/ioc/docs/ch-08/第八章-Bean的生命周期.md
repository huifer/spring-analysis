# 第八章 Bean 的生命周期
- 本章笔者将和各位读者介绍 Bean 生命周期相关的源码内容。

## 8.1 Bean 的生命周期



在聊 Spring Bean 的生命周期之前我们需要先来了解一下 Java 对象的生命周期，

### 8.1.1 Java 对象的生命周期
1. 创建阶段(Created)
2. 应用阶段(In Use)
3. 不可见阶段(Invisible)
4. 不可达阶段(Unreachable)
5. 收集阶段(Collected)
6. 终结阶段(Finalized)
7. 对象空间重分配阶段(De-allocated)

在这7个生命周期状态中我们主要关注的是前两个生命周期状态，其余五个生命周期状态一般在 Java 程序中不可控制。笔者以一个最简单的对象创建来进行描述



```java
public class JavaBeanTest {

    @Test
    void javaBeanLifeCycle(){
        PeopleBean peopleBean = new PeopleBean();
        peopleBean.setName("zhangsan");

        System.out.println(peopleBean.getName());
    }
}
```



`PeopleBean peopleBean = new PeopleBean();` 这一段代码就对应着第一步创建阶段

后面的所有步骤 `set` 也好，`get` 也罢都是在对 `peopleBean` 对象进行使用，对应第二步应用阶段。

在了解 Java 对象的生命周期之后我们来看看 Spring Bean 的生命周期。



### 8.1.2 浅看 Bean 生命周期

关于 Bean 生命周期的内容各位可以在网上很容易的搜索到，笔者在这里将 Spring 中 `BeanFactory` 接口上所编写的生命周期过程复制过来给各位做一个参考。



- Bean Factory 初始化阶段的生命周期
    1. BeanNameAware's `setBeanName`
    1. BeanClassLoaderAware's `setBeanClassLoader`
    1. BeanFactoryAware's `setBeanFactory`
    1. EnvironmentAware's `setEnvironment`
    1. EmbeddedValueResolverAware's `setEmbeddedValueResolver`
    1. ResourceLoaderAware's `setResourceLoader` (only applicable when running in an application context)
    1. ApplicationEventPublisherAware's `setApplicationEventPublisher` (only applicable when running in an application context)
    1. MessageSourceAware's `setMessageSource` (only applicable when running in an application context)
    1. ApplicationContextAware's `setApplicationContext` (only applicable when running in an application context)
    1. ServletContextAware's `setServletContext` (only applicable when running in a web application context)
    1. `postProcessBeforeInitialization` methods of BeanPostProcessors
    1. InitializingBean's `afterPropertiesSet`
    1. a custom init-method definition
    1. `postProcessAfterInitialization` methods of BeanPostProcessors



- Bean Factory 关闭阶段的生命周期
  1. `postProcessBeforeDestruction` methods of DestructionAwareBeanPostProcessors
  2. DisposableBean's `destroy`
  3. a custom destroy-method definition



在整个生命周期(包括初始化和关闭)中会涉及到下面这些方法

1. [`BeanNameAware.setBeanName(java.lang.String)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanNameAware.html#setBeanName-java.lang.String-)
1. [`BeanClassLoaderAware.setBeanClassLoader(java.lang.ClassLoader)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanClassLoaderAware.html#setBeanClassLoader-java.lang.ClassLoader-)
1. [`BeanFactoryAware.setBeanFactory(org.springframework.beans.factory.BeanFactory)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactoryAware.html#setBeanFactory-org.springframework.beans.factory.BeanFactory-)
1. [`ResourceLoaderAware.setResourceLoader(org.springframework.core.io.ResourceLoader)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ResourceLoaderAware.html#setResourceLoader-org.springframework.core.io.ResourceLoader-)
1. [`ApplicationEventPublisherAware.setApplicationEventPublisher(org.springframework.context.ApplicationEventPublisher)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationEventPublisherAware.html#setApplicationEventPublisher-org.springframework.context.ApplicationEventPublisher-)
1. [`MessageSourceAware.setMessageSource(org.springframework.context.MessageSource)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/MessageSourceAware.html#setMessageSource-org.springframework.context.MessageSource-)
1. [`ApplicationContextAware.setApplicationContext(org.springframework.context.ApplicationContext)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContextAware.html#setApplicationContext-org.springframework.context.ApplicationContext-)
1. [`ServletContextAware.setServletContext(javax.servlet.ServletContext)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/ServletContextAware.html#setServletContext-javax.servlet.ServletContext-)
1. [`BeanPostProcessor.postProcessBeforeInitialization(java.lang.Object, java.lang.String)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html#postProcessBeforeInitialization-java.lang.Object-java.lang.String-),
1. [`InitializingBean.afterPropertiesSet()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html#afterPropertiesSet--)
1. [`AbstractBeanDefinition.getInitMethodName()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/AbstractBeanDefinition.html#getInitMethodName--)
1. [`BeanPostProcessor.postProcessAfterInitialization(java.lang.Object, java.lang.String)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html#postProcessAfterInitialization-java.lang.Object-java.lang.String-)
1. [`DisposableBean.destroy()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/DisposableBean.html#destroy--)
1. [`AbstractBeanDefinition.getDestroyMethodName()`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/AbstractBeanDefinition.html#getDestroyMethodName--)

本章的主要分析也会围绕这些方法进行。





在开始正式分析之前我们需要看看我们的测试用例，首先我们先来编写 Bean 对象，笔者在这里定义为 `LiveBean` 



- `LiveBean` 详细代码

```java
package com.source.hot.ioc.book.live;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanClassLoaderAware;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.context.EmbeddedValueResolverAware;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.MessageSource;
import org.springframework.context.MessageSourceAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.core.env.Environment;
import org.springframework.core.io.ResourceLoader;
import org.springframework.util.StringValueResolver;

public class LiveBean implements BeanNameAware, BeanFactoryAware,
      ApplicationContextAware, InitializingBean, DisposableBean, BeanClassLoaderAware,
      EnvironmentAware, EmbeddedValueResolverAware, ResourceLoaderAware,
      ApplicationEventPublisherAware, MessageSourceAware {

  private String address;

  public LiveBean() {
      System.out.println("init LiveBean");
  }

  @Override
  public void setBeanClassLoader(ClassLoader classLoader) {
      System.out.println("run setBeanClassLoader method.");
  }

  @Override
  public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
      System.out.println("run setApplicationEventPublisher method.");

  }

  @Override
  public void setEmbeddedValueResolver(StringValueResolver resolver) {
      System.out.println("run setEmbeddedValueResolver method.");

  }

  @Override
  public void setEnvironment(Environment environment) {
      System.out.println("run Environment method.");

  }

  @Override
  public void setMessageSource(MessageSource messageSource) {
      System.out.println("run setMessageSource method.");

  }

  @Override
  public void setResourceLoader(ResourceLoader resourceLoader) {
      System.out.println("run setResourceLoader method.");

  }

  @Override
  public void setBeanName(String name) {
      System.out.println("run setBeanName method.");
  }

  @Override
  public void destroy() throws Exception {
      System.out.println("run destroy method.");
  }

  @Override
  public void afterPropertiesSet() throws Exception {
      System.out.println("run afterPropertiesSet method.");
  }

  @Override
  public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
  }

  @PostConstruct
  public void springPostConstruct() {
      System.out.println("@PostConstruct");
  }

  @PreDestroy
  public void springPreDestroy() {
      System.out.println("@PreDestroy");
  }


  public void myPostConstruct() {
      System.out.println("run myPostConstruct method.");
  }

  public void myPreDestroy() {
      System.out.println("run myPreDestroy method.");
  }


  public String getAddress() {
      return address;
  }

  public void setAddress(String address) {
     System.out.println("run setAddress method.");
     this.address = address;
  }


  @Override
  public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
      System.out.println("run setBeanFactory method.");
  }
}
```

下面我们来编写 `BeanPostProcessor` 代码

- `MyBeanPostProcessor` 详细代码

```java
package com.source.hot.ioc.book.live;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {

  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      if (bean instanceof LiveBean) {
          System.out.println("run MyBeanPostProcessor postProcessBeforeInitialization method.");
      }
      return bean;
  }

  public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      if (bean instanceof LiveBean) {
          System.out.println("run MyBeanPostProcessor postProcessAfterInitialization method.");
      }
      return bean;
  }
}
```

完成 Java 代码的准备后我们来编写 Spring xml 配置文件

- `live-bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns="http://www.springframework.org/schema/beans"
     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean class="com.source.hot.ioc.book.live.MyBeanPostProcessor"/>
  <bean id="liveBean" class="com.source.hot.ioc.book.live.LiveBean" init-method="myPostConstruct" destroy-method="myPreDestroy">
      <property name="address" value="shangHai"/>
  </bean>
</beans>
```



最后编写测试用例代码

```java
public class JavaBeanTest {


    @Test
    void testSpringBeanLive() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/live-bean.xml");
        LiveBean liveBean = context.getBean("liveBean", LiveBean.class);
        context.close();
    }
}
```





笔者在这先将执行结果全部输出

```
init LiveBean
run setAddress method.
run setBeanName method.
run setBeanClassLoader method.
run setBeanFactory method.
run Environment method.
run setEmbeddedValueResolver method.
run setResourceLoader method.
run setApplicationEventPublisher method.
run setMessageSource method.
run MyBeanPostProcessor postProcessBeforeInitialization method.
run afterPropertiesSet method.
run myPostConstruct method.
run MyBeanPostProcessor postProcessAfterInitialization method.
run destroy method.
run myPreDestroy method.

```



用例准备就绪下面就开始对 Bean 的生命周期进行分析。





### 8.1.3 初始化 Bean 

首先我们来看 Bean 对象创建。简单说就是 `new` 对象。但是在 SpringIoC 中不是 `new` 出来的，下面我们来看看 Spring 是怎么做的吧。

请各位先看下面这段 Spring xml 配置

```xml
<bean id="liveBean" class="com.source.hot.ioc.book.live.LiveBean" init-method="myPostConstruct" destroy-method="myPreDestroy">
    <property name="address" value="shangHai"/>
</bean>
```

在这段 xml 的配置中我们并没有看到关于构造函数相关的内容，在[第六章](/docs/ch-06/第六章-bean标签解析.md)中提到过关于 `<constructor-arg/>` 标签的处理，`<constructor-arg/>` 标签会和构造函数有关，在  `<constructor-arg/>` 中笔者当时提到了两中模式，一种是 `index` 进行数据绑定，另一种是 `name` 进行数据绑定，那么我们对于初始化 Bean 就有三个分析方向，第一：没有任何配置的情况下，第二：使用 `<constructor-arg/>` + `index` 模式的情况下，第三：使用 `<constructor-arg/>` + `name` 模式的情况下，本节将围绕这三种情况对 Bean 的初始化过程做一个分析。

tips: 对于 `<constructor-arg/>` 笔者在下文将其称为**构造标签**



#### 8.1.3.1 无构造标签

首先我们来看无构造标签的情况，在看 Spring 中的实现之前笔者想请各位读者思考：**已知类全路径(字符串)如何得到该对象的实例?** 只要理解这个问题对于无构造标签的底层各位就可以理解了。下面笔者来进行一下回答，这里必不可少的就是**反射**的使用，下面笔者将实现过程完整的列出

1. 第一步：定义一些变量

```java
String className = "com.source.hot.ioc.book.live.LiveBean";
ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
```

   在这一步中笔者将 `className` 和 `ClassLoader` 定义出来

2. 第二步：通过 `ClassLoader` 和 `className` 转化成 `Class` 对象

```java
Class<?> aClass = contextClassLoader.loadClass(className);
```

3. 第三步：通过 `Class` 得到对象实例，在这一步会有两种方式

   第一种：通过 `Class#newInstance` 方法直接获取对象实例

```java
Object o2 = aClass.newInstance();
```

   第二种：通过 `Class#getConstructor` 再通过 `Constructor#newInstance` 方法进行对象实例的获取

```java
Constructor<?> constructor = aClass.getConstructor();
Object o = constructor.newInstance();
```



下面是完整代码

```java
@Test
void testClass() throws Exception {
    String className = "com.source.hot.ioc.book.live.LiveBean";
    ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
    Class<?> aClass = contextClassLoader.loadClass(className);

    Object o2 = aClass.newInstance();

    Constructor<?> constructor = aClass.getConstructor();
    Object o = constructor.newInstance();
    if (o instanceof LiveBean) {
        LiveBean o1 = (LiveBean) o;
        o1.setAddress("shangHai");
        System.out.println(o1.getAddress());
    }
}
```



在聊了这么多关于 **已知类全路径(字符串)如何得到该对象的实例？** 的问题后我们该回到 Spring 中来看看 Spring 中的具体实现了。

 我们先不管 Spring 外层的各种调用我们直接找到最核心的处理逻辑。负责处理构造对象的方法是：`org.springframework.beans.BeanUtils#instantiateClass(java.lang.reflect.Constructor<T>, java.lang.Object...)`

下面我们就是来看这个方法的实现

- 方法详情

```java
// 删除注释、异常处理和部分验证
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
    
  ReflectionUtils.makeAccessible(ctor);
  if (KotlinDetector.isKotlinReflectPresent() && KotlinDetector.isKotlinType(ctor.getDeclaringClass())) {
      return KotlinDelegate.instantiateClass(ctor, args);
  }
  else {
      Class<?>[] parameterTypes = ctor.getParameterTypes();
      Assert.isTrue(args.length <= parameterTypes.length, "Can't specify more arguments than constructor parameters");
      Object[] argsWithDefaultValues = new Object[args.length];
      for (int i = 0; i < args.length; i++) {
          if (args[i] == null) {
              Class<?> parameterType = parameterTypes[i];
              argsWithDefaultValues[i] = (parameterType.isPrimitive() ? DEFAULT_TYPE_VALUES.get(parameterType) : null);
          }
          else {
              argsWithDefaultValues[i] = args[i];
          }
      }
      return ctor.newInstance(argsWithDefaultValues);
  }

}
```

  

我们可以先将关于 Kotlin 的代码忽略就关注 `else` 中的代码，从这段 `else` 的返回值上我们可以看到一个在前面讲 **已知类全路径(字符串)如何得到该对象的实例？** 问题的时候出现过的代码：`ctor.newInstance(argsWithDefaultValues)` ，这段代码和笔者在前文编写的差异是传入了构造参数一个 `Object` 数组。深究下去的话数据 Java 本身的内容了在这就不进行深入了。整个方法无非就是通过 `Constructor` + 构造参数 进行反射得到实例。 







#### 8.1.3.2 构造标签配合 `index` 模式 和 `name` 模式

在关于无构造标签解析的分析中我们找到了核心的 Bean 实例获取方式，在其中有一个关键信息 `argsWithDefaultValues` 这个信息里面存储了构造函数中需要的数据下面我们以 `PeopleBean` 举例

- `PeopleBean` 信息

```java
public class PeopleBean {
    private String name;
	public PeopleBean(String name) {
    	this.name = name;
    }
}
```

- Spring xml 配置

```xml
<bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
    <constructor-arg index="0" type="java.lang.String" value="zhangsan">

    </constructor-arg>

</bean>
```



在[第六章](/docs/ch-06/第六章-bean标签解析.md)中笔者告诉了各位读者 `<constructor-arg/>` 标签的解析结果是在 `ConstructorArgumentValues` 类中进行存储。那么这里的问题就转换成了**如何将对象 `ConstructorArgumentValues` 中存储的内容转换成构造函数对象 `Constructor` 中所需要的构造参数列表？**这个问题的答案在 `org.springframework.beans.factory.support.ConstructorResolver#autowireConstructor` 方法中有详细的说明，下面我们来看看具体的处理方式。

在 `ConstructorResolver#autowireConstructor` 方法中还包含了一个关于依赖注入的一个方式的处理：**构造函数的依赖注入**，下面笔者将列出在该方法中的处理方式。

在分析处理方法之前我们需要明确三个变量

1. `Constructor<?> constructorToUse ` ：需要被使用的构造函数
2. `ArgumentsHolder argsHolderToUse `：参数持有者
3. `Object[] argsToUse` ：需要被使用的构造函数的参数列表

这三个变量决定了对象的创建结果。

在 `org.springframework.beans.factory.support.ConstructorResolver#autowireConstructor` 中关于 `argsToUse` 的获取方式

1. 第一种：直接将方法参数作为 `argsToUse` 
2. 第二种：通过 `org.springframework.beans.factory.support.ConstructorResolver#resolvePreparedArguments` 方法得到 `argsToUse` 对象，
3. 第三种：通过候选的构造函数对象(`Constructor`)进行推论



在第三种方法中衍生出了一个推论，具体怎么推论也是值得分析的内容，首先在推论`argsToUse` 之前必须要做的一件事情：**找到构造函数对象 `Constructor` ** 首先通过反射通过方法`Class#getDeclaredConstructors` 和 `Class#getConstructors` 我们可以获取到所有的 `Constructor` 但是还缺少一个至关重要的条件。这个条件由三部分组成：第一部分：**参数长度**，第二部分：**参数类型**，第三部分：**参数名称**。通过这三个条件我们就可以确认到我们需要的构造函数 `Constructor`，在确认完成之后 `argsToUse` 的确认工作也就完成了。



事实上在处理 `name` 模式时候的行为和上述的行为是一样的，他们的差异只是下面这两个图的对比



- `name` 模式的存储

  ![image-20210112144746394](./images/image-20210112144746394.png)

- `index` 模式的存储

  ![image-20210112144823088](./images/image-20210112144823088.png)



具体的存储信息可以看下面的内容

```java
/**
 * 构造函数信息
 * key: 索引
 * value: 标签数据
 */
private final Map<Integer, ValueHolder> indexedArgumentValues = new LinkedHashMap<>();

/**
 * name 模式的存储信息
 */
private final List<ValueHolder> genericArgumentValues = new ArrayList<>();
```



最终对于 `argsToUse` 的确定其实就是对于 `ValueHolder` 的数据提取 ( 提取 `ValueHolder` 中的 `convertedValue` 对象数据)，具体的提取方法各位可以看这个方法签名对应的方法：`org.springframework.beans.factory.support.ConstructorResolver#createArgumentArray`

注意：在 `ValueHolder` 中会有依赖注入的操作



- 修改 Spring xml 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns="http://www.springframework.org/schema/beans"
     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="people" class="com.source.hot.ioc.book.pojo.PeopleBean">
      <constructor-arg index="0" type="java.lang.String" value="zhangsan">

      </constructor-arg>

  </bean>
  <bean id="people2" class="com.source.hot.ioc.book.pojo.PeopleBean">
      <constructor-arg name="name" type="java.lang.String" value="zhangsan">

      </constructor-arg>
      <constructor-arg name="pb" ref="people"/>

  </bean>

</beans>
```



在这个配置文件基础上我们先来看存储情况

![image-20210112145832012](./images/image-20210112145832012.png)

在这个例子中我们有两种模式一个是字面量，另一个是引用量，对于这两种方式的处理在 Spring 中都有相关处理具体对于这部分的处理各位可以看 `org.springframework.beans.factory.support.ConstructorResolver#resolveConstructorArguments` 

- 处理调用

```java
minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues)
```

  在这个方法调用中会修改 `resolvedValues` 的数据，在这个处理过程中就会去将数据补充完整

  ![image-20210112151748864](./images/image-20210112151748864.png)



我们可以看到现在数据工作都已经准备好了，最后就是提取成 `Object[] argsToUse ` ，当我们将二大要素准备为完成就可以准备进行构造了，构造方式就是 `Constructor#newInstance(argsToUse)`，最终还是会交给 `BeanUtils` 来进行。



至此笔者对于实例化对象的形式已经全部讲述完成，相信各位肯定有所收获。







#### 8.1.3.4 Spring 中实例化策略

在 Spring 中对于实例化方案的选择还有一个细节点，在 Java 这门语言中有一个叫做代理对象的东西，那么这里会有两种情况，第一种：没有代理的实例化方案，第二种：存在代理的实例化方案。围绕这两点 Spring 定义了 `InstantiationStrategy` 接口，这个接口的主要作用就是来进行实例化策略的执行。目前在 Spring 中关于动态代理这方面技术所使用的是 cglib 技术，在关于动态代理层面的内容时候 Spring 都会用 cglib 中的内容。下面我们来看看 `InstantiationStrategy` 类图

- `InstantiationStrategy` 类图

  ![InstantiationStrategy](./images/InstantiationStrategy.png)

在整个 Spring 中有两个创建对象的形式(可能还有其他，欢迎读者提出)。

1. 第一种：通过 `BeanUtils.instantiateClass()` 方法创建实例
2. 第二种：通过 `new CglibSubclassCreator(bd, owner).instantiate(ctor, args)` 方式进行创建

这两种方式分别在 `SimpleInstantiationStrategy` 和 `CglibSubclassingInstantiationStrategy` 中有所体现。

在了解两种实例化对象的方式后我们需要知道什么时候会进行 cglib 形式的创建，什么时候进行 `BeanUtils` 方式的创建。

在  `RootBeanDefinition` 中有 `hasMethodOverrides` 方法。具体代码看下面内容

```java
public boolean hasMethodOverrides() {
   return !this.methodOverrides.isEmpty();
}
```

- `methodOverrides` 该变量存储的是重写方法，Spring xml 环境下关于配置方法重写有两种方式，第一种：使用 `lookup-method` 标签，第二种：使用 `replaced-method` 标签。

  这个方法就是用来判断是否使用 `cglib` 方式进行创建的核心。总结一句话：**当方法有重写的时候就会调用 `cglib` 方式创建。** 













### 8.1.4 Bean 属性设置

现在我们对于初始化 Bean 已经有了一个全面的了解，下面我们将来对于属性设置进行分析。对于字段的数据设置在 Java 层面来说肯定还是通过反射来进行设置的，在 Java 中可以通过下面这种形式进行数据字段设置。

- Java 中通过反射的形式设置数据

```java
void setField() throws Exception {
    LiveBean liveBean = new LiveBean();
    Field address = liveBean.getClass().getDeclaredField("address");
    address.setAccessible(true);
    address.set(liveBean, "shangHai");
    System.out.println(liveBean.getAddress());
}
```



在了解 Java 反射层面的设置数据的形式后我们来看 Spring 中对于属性赋值的操作。

首先各位需要先看 Spring XML 配置文件内容

- 测试用 XML 配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.source.hot.ioc.book.live.MyBeanPostProcessor"/>
    <bean id="liveBean" class="com.source.hot.ioc.book.live.LiveBean" init-method="myPostConstruct" destroy-method="myPreDestroy">
        <property name="address" value="shangHai"/>
    </bean>
</beans>
```



通过 Java 反射的形式我们可以进行设置数据，那么在属性设置前我们其实已经拥有对象了，下面就可以使用类似的形式进行数据设置。在本例中我们是 `address` 属性需要设置，那么这样就可以直接设置成功，Spring 中的细节会比我所说的内容多很多，我们来看看 Spring 中的处理逻辑吧。最重要的一部是找到处理属性设置的方法，笔者在这里将方法全路径贴出：`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyPropertyValues`，下面笔者将对这个方法进行分析。

首先我们来看该方法的参数

1.  `beanName` ：Bean 名称
2. `BeanDefinition`： Bean 定义
3. `BeanWrapper`：Bean 包装类
4. `PropertyValues`：属性值接口，包含 `PropertyValue` 对象

在这四个变量中我们在属性设置的时候更多应该关注 `PropertyValues` 这个变量，在 `PropertyValues` 中存储了标签 `property` 的数据内容，关于标签到对象的分析各位可以看第六章。

对应前文提到的测试配置我们先来看 Bean Definition 的对象内容

![image-20210113151712020](./images/image-20210113151712020.png)



了解方法的参数内容后我们来看具体的设置过程。

首先明确我们可以通过属性名称获取对应的设置方法，再通过对应的方法进行数据设置。我们围绕这两个细节进行分析。

`PropertyValues` 接口是迭代器 `Iterable` 的子类，它本身可以进行迭代获取每个元素，在这里每个元素对象是 `PropertyValue`	那么 `PropertyValue` 中的存储尤为关键。来看 `PropertyValue` 中的关键字段以及含义



- `PropertyValue` 字段及其含义

| 字段名称 | 类型     | 含义                                                      |
| -------- | -------- | --------------------------------------------------------- |
| `name`   | `String` | 属性名称，对应 xml 配置中的标签 `property` 的 `name` 属性 |
| `value`  | `Object` | 属性值                                                    |



现在我们有了需要设置的字段和设置的值，我们还缺少一个被设置的对象，也就是 Bean 实例，那么 Bean 实例藏在哪里了呢？回答这个问题各位需要了解 `BeanWrapper` 接口。在了解这个接口之前我们先看看它的类图

- `BeanWrapper` 类图

  ![BeanWrapper.png](./images/BeanWrapper.png) 



这里我们需要解决一个最重要的问题 **`BeanWrapper` 怎么创建**



#### 8.1.4.1 Bean Wrapper 创建



创建 `Bean Wrapper` 的方法是： `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBeanInstance`

各位如果深入找代码可以看到下面这段内容

```java
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
   try {
      Object beanInstance;
      final BeanFactory parent = this;
      if (System.getSecurityManager() != null) {
         // 获取实例化策略来进行实例化
         beanInstance = AccessController.doPrivileged(
               (PrivilegedAction<Object>) () ->
                     getInstantiationStrategy().instantiate(mbd, beanName, parent),
               getAccessControlContext()
         );
      }
      else {
         // 获取实例化策略来进行实例化
         beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
      }

      // beanWrapper 创建
      BeanWrapper bw = new BeanWrapperImpl(beanInstance);
      initBeanWrapper(bw);
      return bw;
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
   }
}
```



这段代码中包含两大快内容

1. 第一块：获取实例化策略接口 `InstantiationStrategy` 实例化对象
2. 第二块：封装 `BeanWrapper` 对象



看到这里我们前面所缺少的一个东西就出来了，我们缺少需要被赋值的对象就是通过实例化策略接口实例化出来的对象。



下面我们看 `BeanWrapper` 的数据设置。`BeanWrapper` 中提供了 `setPropertyValues` 作为数据设置的方案。



#### 8.1.4.2 Bean Wrapper 设置属性

我们还是需要先找到设置的具体方法， `org.springframework.beans.AbstractNestablePropertyAccessor#setPropertyValue(org.springframework.beans.PropertyValue)`  方法是设置属性的入口，详细代码如下

- `setPropertyValue` 方法详情

```java
@Override
public void setPropertyValue(PropertyValue pv) throws BeansException {
   // 需要设置的字段名称
   PropertyTokenHolder tokens = (PropertyTokenHolder) pv.resolvedTokens;
   if (tokens == null) {
      // 属性名称
      // 获取属性名称
      String propertyName = pv.getName();
      AbstractNestablePropertyAccessor nestedPa;
      try {
         // 属性访问器
         nestedPa = getPropertyAccessorForPropertyPath(propertyName);
      }
      catch (NotReadablePropertyException ex) {
         throw new NotWritablePropertyException(getRootClass(), this.nestedPath + propertyName,
               "Nested property in path '" + propertyName + "' does not exist", ex);
      }
      tokens = getPropertyNameTokens(getFinalPath(nestedPa, propertyName));
      if (nestedPa == this) {
         pv.getOriginalPropertyValue().resolvedTokens = tokens;
      }
      // 设置属性
      nestedPa.setPropertyValue(tokens, pv);
   }
   else {
      setPropertyValue(tokens, pv);
   }
}
```



在这个方法中我们需要着重关注三个变量。第一个是 `AbstractNestablePropertyAccessor nestedPa ` ，第二个是 `PropertyTokenHolder tokens`，第三个是 `PropertyValue pv` 

- `AbstractNestablePropertyAccessor` 作用是嵌套属性访问器，说是访问器但它也是可以设置属性的。
- `PropertyTokenHolder` 存储了属性名称等内容

下面我们来看看当前的数据有哪些。

1. 参数 `pv` 的信息

   ![image-20210113162144229](./images/image-20210113162144229.png)

2. `tokens` 的信息

   ![image-20210113162206886](./images/image-20210113162206886.png)

3. 嵌套属性访问器的信息

   ![image-20210113162241124](./images/image-20210113162241124.png)



在正真处理的时候会有两种处理方式

1. 第一种：通过 `processKeyedProperty` 方法进行处理
2. 第二种：通过 `processLocalProperty` 方法进行处理



这里笔者没有测试出可以进入 `processKeyedProperty` 方法的配置，目前只有关于 `processLocalProperty` 的处理。下面我们来看 `processLocalProperty` 的细节



```java
// 删除日志，异常处理等代码
private void processLocalProperty(PropertyTokenHolder tokens, PropertyValue pv) {
    // ph 里面有对象
    PropertyHandler ph = getLocalPropertyHandler(tokens.actualName);
    Object oldValue = null;
    // 获取 PV 的属性值
    Object originalValue = pv.getValue();
    Object valueToApply = originalValue;
    // 是否需要
    if (!Boolean.FALSE.equals(pv.conversionNecessary)) {
        // 是否能够转换
        if (pv.isConverted()) {
            valueToApply = pv.getConvertedValue();
        }
        // 不能转换换
        else {
            if (isExtractOldValueForEditor() && ph.isReadable()) {
                oldValue = ph.getValue();
            }
            valueToApply = convertForProperty(tokens.canonicalName, oldValue, originalValue, ph.toTypeDescriptor());
        }
        pv.getOriginalPropertyValue().conversionNecessary = (valueToApply != originalValue);
    }
    ph.setValue(valueToApply);
}
```



在这个方法中关键对象是 `PropertyHandler` , 从代码的细节上我们可以看到设置数据也和这个对象有关，对于 `getLocalPropertyHandler()` 方法的细节和  `PropertyHandler` 的类图都有值得研究的价值。下面我们先来看 `PropertyHandler` 类图

![PropertyHandler](./images/PropertyHandler.png) 

目前我们位于 `BeanWrapperImpl` 中它对于 `getLocalPropertyHandler` 返回值处理是 `BeanPropertyHandler` ，这也合乎情理，毕竟我们目前正在处理的是关于 Bean 属性的相关操作。那我们来看看 `BeanWrapperImpl#getLocalPropertyHandler` 中的实现过程吧。

下面先请各位阅读代码

- `BeanWrapperImpl#getLocalPropertyHandler` 详细代码

```java
@Override
@Nullable
protected BeanPropertyHandler getLocalPropertyHandler(String propertyName) {
 PropertyDescriptor pd = getCachedIntrospectionResults().getPropertyDescriptor(propertyName);
 return (pd != null ? new BeanPropertyHandler(pd) : null);
}
```

在这个方法中我们需要知道 `getCachedIntrospectionResults` 所返回的变量 `CachedIntrospectionResults` 中存放的一些数据内容。



##### 8.1.4.2.1 `CachedIntrospectionResults` 介绍

对于 `CachedIntrospectionResults` 类我们需要重点关注下面这些成员变量，这些成员变量的关系在整个 Spring 对于 Bean 信息的存储还是一个关键的点



| 变量名称                                  | 变量类型                                              | 变量作用                                                     | Java Doc                                                     |
| ----------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `acceptedClassLoaders`                    | `Set<ClassLoader>`                                    | 存储类加载器                                                 | Set of ClassLoaders that this CachedIntrospectionResults class will always accept classes from, even if the classes do not qualify as cache-safe |
| `strongClassCache`                        | `ConcurrentMap<Class<?>, CachedIntrospectionResults>` | 存储线程安全的bean 信息                                      | Map keyed by Class containing CachedIntrospectionResults, strongly held. This variant is being used for cache-safe bean classes. |
| `softClassCache`                          | `ConcurrentMap<Class<?>, CachedIntrospectionResults>` | 存储线程不安全的bean 信息                                    | Map keyed by Class containing CachedIntrospectionResults, softly held. This variant is being used for non-cache-safe bean classes. |
| `shouldIntrospectorIgnoreBeaninfoClasses` | `boolean`                                             | 从配置文件中读取 `spring.beaninfo.ignore` 的信息作为变量的数据 |                                                              |
| `beanInfoFactories`                       | `List<BeanInfoFactory>`                               | Bean Factory 容器，存错多个 Bean Factory                     | Stores the BeanInfoFactory instances.                        |
| `beanInfo`                                | `BeanInfo`                                            | `BeanInfo` 接口是由 JDK 提供的一个关于 Bean 信息的接口，在这个接口中包含了关于 Bean 的基本描述，如：`PropertyDescriptor` 属性描述，`MethodDescriptor` 方法描述等 | The BeanInfo object for the introspected bean class.         |
| `propertyDescriptorCache`                 | `Map<String, PropertyDescriptor>`                     | 用于存储属性(字段)名称和属性(字段)描述的容器，key: 属性名称，value: 属性描述 | PropertyDescriptor objects keyed by property name String.    |
| `typeDescriptorCache`                     | `ConcurrentMap<PropertyDescriptor, TypeDescriptor>`   | 用于存储属性描述对象和属性类型的描述对象                     | TypeDescriptor objects keyed by PropertyDescriptor.          |



在了解了 `CachedIntrospectionResults` 对象之后我们需要使用它，在 `CachedIntrospectionResults` 中提供了一个 `forClass` 就可以进行数据结果的产生，这个方法也是在 `BeanWrapperImpl#getCachedIntrospectionResults` 中的一个调用

- `BeanWrapperImpl#getCachedIntrospectionResults` 方法详情

```java
private CachedIntrospectionResults getCachedIntrospectionResults() {
   if (this.cachedIntrospectionResults == null) {
      this.cachedIntrospectionResults = CachedIntrospectionResults.forClass(getWrappedClass());
   }
   return this.cachedIntrospectionResults;
}
```



下面我们来看一下关于我们定义的 `LiveBean` 通过计算后会变成什么样，

- `CachedIntrospectionResults` 中的信息

  ![image-20210114091830236](./images/image-20210114091830236.png)

  

  

  ![image-20210114092536394](./images/image-20210114092536394.png)

  

  上图信息不是完整的信息内容各位读者如果想要看到完整信息可以使用下面这个方法，**在使用这个方法的时候需要修改`CachedIntrospectionResults.forClass` 将其变成 `public` **否则不可调用 

  

```java
@Test
void testCachedIntrospectionResults(){
  CachedIntrospectionResults results = CachedIntrospectionResults.forClass(LiveBean.class);
  System.out.println();
}
```





在上面的图片中可以发现对于属性的描述会存在 `get` 和 `set` 方法，这两个都是 `Method` 对象存储在 `propertyDescriptorCache` 中，那么我们对于属性设置使用 `set` 方法就可以进行设置了。到这一切都拨开迷雾了，对于 `propertyDescriptorCache` 笔者前文介绍了是一个 `map` 对于 `getCachedIntrospectionResults().getPropertyDescriptor(propertyName)` 这段方法的后半段 `getPropertyDescriptor` 方法就清晰了解了，下面我们来看最终的获取结果

![image-20210114092946025](./images/image-20210114092946025.png)



现在我们拥有了属性名称对应的属性描述符对象(`PropertyDescriptor`) ，下面 Spring 就会将其包装成 `BeanPropertyHandler` 对象

上述这些内容就为了将 `BeanPropertyHandler` 获取。这样我们对于 `processLocalProperty` 方法中的第一个核心对象 `PropertyHandler` 做了一个充分的了解



在前文笔者告诉了大家 `pv` 中的信息，那么 `pv` 存在的信息我们可以简化的去思考，`pv` 中存储的不管是直接变量，还是类似 `${}` 需要做转换，还是 连接的 Bean (关联对象，依赖注入的) 都可以通过一定的方法得到。下面我们来看看 `pv` 的一些含义





##### 8.1.4.2.2 `PropertyValue` 介绍



| 变量名称              | 变量类型  | 变量作用         | Java Doc                                                     |
| --------------------- | --------- | ---------------- | ------------------------------------------------------------ |
| `name`                | `String`  | 属性名称         |                                                              |
| `value`               | `Object`  | 数据值           |                                                              |
| `conversionNecessary` | `Boolean` | 是否需要进行转换 | Package-visible field that indicates whether conversion is necessary. |
| `resolvedTokens`      | `Object`  | 需要解析的数据   | Package-visible field for caching the resolved property path tokens. |
| `optional`            | `boolean` | 是否可选         |                                                              |
| `converted`           | `boolean` | 是否已转换       |                                                              |
| `convertedValue`      | `Object`  | 转换后的结果     |                                                              |





在 `pv` 中存储数据值的变量有两个 `value` 和 `convertedValue` ，从这两个变量中我们可以获取数据，现在我们在归纳一下当前存在的信息



1. `PropertyHandler` 具体是 `BeanPropertyHandler` ， 在 `BeanPropertyHandler` 中提供的 `setValue` 方法可以进行属性设置，在  `PropertyHandler` 中还有 `set` 方法
2. `pv` 中存在的数据值和需要设置属性名称



现在我们具备了需要执行的方法，这个方法是 `Method` 指向 `setAddress` , 同时还有需要设置的属性，那么反射通过 `Method#invoke()` 就可以设置完成属性，那么我们来看最后的一个方法 `BeanPropertyHandler#setValue`





##### 8.1.4.2.3 `BeanPropertyHandler#setValue` 最终的属性设置

先看 `BeanPropertyHandler#setValue` 代码

- `BeanPropertyHandler#setValue` 详细信息

```java
@Override
public void setValue(final @Nullable Object value) throws Exception {
 final Method writeMethod = (this.pd instanceof GenericTypeAwarePropertyDescriptor ?
       ((GenericTypeAwarePropertyDescriptor) this.pd).getWriteMethodForActualAccess() :
       this.pd.getWriteMethod());
 if (System.getSecurityManager() != null) {
    AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
       ReflectionUtils.makeAccessible(writeMethod);
       return null;
    });
    try {
       AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () ->
             writeMethod.invoke(getWrappedInstance(), value), acc);
    }
    catch (PrivilegedActionException ex) {
       throw ex.getException();
    }
 }
 else {
    ReflectionUtils.makeAccessible(writeMethod);
    writeMethod.invoke(getWrappedInstance(), value);
 }
}
```



在这个方法中我们来看下面三个变量

1. `writeMethod` ： 写函数，一般是 `set` 方法
2. `value`：设置的属性值
3. `getWrappedInstance()`：需要进行设置的对象

下面是我们在当前阶段需要设置的内容

![image-20210114095309170](./images/image-20210114095309170.png)





到此执行完成这个方法后属性就会设置成功。现在我们来看看 `BeanWrapper` 中的情况

- 进行数据设置后的 `BeanWrapper`

![image-20210114095551346](./images/image-20210114095551346.png)



在图中可以看到 `wrappedObject` 中填充了 `address` 的属性。



到此这两步：第一步：初始化 Bean 和 第二步：设置属性，这两步在 Java 中是在正常不过的操作，`new` 对象 ， `set` 方法设置属性，在进行后续使用。那当我们把这些事情交给 Spring 来做或者屏蔽了这些操作的时候，这就引申出了两个个名词 ，第一个：**控制反转 IoC (Inversion of Control)** ，第二个：**依赖注入 DI(Dependency Injection)**

就目前阶段 Spring 将对象创建和数据设置的过程都接管了这一部分可以说是控制反转，对于 Bean 属性设置其实也就是依赖注入，通过 [8.1.3 初始化 Bean] 和 [8.1.4 Bean 属性设置] 这两节我们可以归纳出两种依赖注入的形式：**构造器注入**和**`set` 方法注入** 。

这里笔者对 IoC 和 DI 的讨论肯定不是很完善，各位可以再进行一些资料的搜索来完善这方面的认识。

在完成这两步后 Bean 的基本实例已经完成，但 Spring 还提供了多个关于 Bean 生命周期的拓展下面我们来对这些拓展进行了解





### 8.1.5 Bean生命周期 Aware

在 `BeanWrapper` 对象中设置完成属性之后紧接着会做各类生命周期的拓展，首先我们来看这个方法的签名地址：`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)`，知道方法签名后我们来看看具体的代码

- `AbstractAutowireCapableBeanFactory#initializeBean` 方法详情

```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
   // 第一部分
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
         invokeAwareMethods(beanName, bean);
         return null;
      }, getAccessControlContext());
   }
   else {
      // aware 接口执行
      invokeAwareMethods(beanName, bean);
   }

   // 第二部分
   Object wrappedBean = bean;
   if (mbd == null || !mbd.isSynthetic()) {
      // BeanPostProcessor 前置方法执行
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   }

   // 第三部分
   try {
      // 执行实例化函数
      invokeInitMethods(beanName, wrappedBean, mbd);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex
      );
   }

   // 第四部分
   if (mbd == null || !mbd.isSynthetic()) {
      // BeanPostProcessor 后置方法执行
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   }

   return wrappedBean;
}
```



在这段方法中处理了 Spring Bean 生命周期的拓展，主要关注下面四个方法。

1. 第一个：`invokeAwareMethods`：执行 `Aware` 相关的接口方法。
2. 第二个：`applyBeanPostProcessorsBeforeInitialization`：执行 `BeanPostProcessor`中的 `postProcessBeforeInitialization` 方法。
3. 第三个：`invokeInitMethods`：执行 `InitializingBean` 接口提供的方法和自定义配置的 `init-method` 方法(Spring XML `bean` 标签的属性)。
4. 第四个：`applyBeanPostProcessorsAfterInitialization`：执行 `BeanPostProcessor` 中的 `postProcessAfterInitialization` 方法。



在这一节主要围绕第一个方法 `invokeAwareMethods` 进行分析。

- `AbstractAutowireCapableBeanFactory#invokeAwareMethods` 方法详情

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
   if (bean instanceof Aware) {
      if (bean instanceof BeanNameAware) {
         ((BeanNameAware) bean).setBeanName(beanName);
      }
      if (bean instanceof BeanClassLoaderAware) {
         ClassLoader bcl = getBeanClassLoader();
         if (bcl != null) {
            ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
         }
      }
      if (bean instanceof BeanFactoryAware) {
         ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
      }
   }
}
```

在这个方法中会执行三个和 `Aware` 相关的接口，第一个：`BeanNameAware`，第二个：`BeanClassLoaderAware`，第三个：`BeanFactoryAware` 这些接口都可以是某一个 Bean 的实现方法，当拥有这三个接口就会被执行，这里就是一个调用，细节内容不是很多。





### 8.1.6 `BeanPostProcessor#postProcessBeforeInitialization`方法

在完成 `Aware` 相关方法的处理后 Spring 会进行 `applyBeanPostProcessorsBeforeInitialization` 的处理。在这个方法中主要对 `BeanPostProcessor` 接口的实现类进行处理，且只会执行 `postProcessBeforeInitialization` 方法，我们一般称之为 Bean 前置操作。下面我们来看看代码

- `applyBeanPostProcessorsBeforeInitialization`

```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
      Object current = processor.postProcessBeforeInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```



这段代码的主要行为就是从容器中获取所有的 `BeanPostProcessor` 并调用 `postProcessBeforeInitialization` 方法，在这里我们可以发现所有的 Bean 实例都会被放进去执行，在编写 `BeanPostProcessor` 时候我们对于第一个参数 `bean` 一定要做好类型上的控制。



### 8.1.7 `InitializingBean` 和 自定义 `init-method` 方法

在完成 `applyBeanPostProcessorsBeforeInitialization`  之后就是执行 `InitializingBean` 和 自定义 `init-method` 相关方法。在我们的例子中 `LiveBean` 本身实现了 `InitializingBean` 接口，这里会直接执行 `InitializingBean` 中的 `afterPropertiesSet` 方法，这一点执行很好理解，下面还有一个关于自定义 `init-method` 的执行，首先我们需要知道 `init-method` 存储在哪里。在 `AbstractBeanDefinition` 中有一个属性 `initMethodName` 它就是存储  `bean` 标签中 `init-method` 属性的变量。下面我们来看存储信息

-  `init-method` 对应的存储

  ![image-20210114135206724](./images/image-20210114135206724.png)



在这个图中我们可以看到他是一个字符串，字符串不具备执行功能，我们还需要通过字符串来找到具体的 `Method` 对象并最终执行它。在了解这些内容后我们来看 `invokeInitMethods` 方法



```java
protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
      throws Throwable {


   // 第一部分
   // 是否是 InitializingBean
   boolean isInitializingBean = (bean instanceof InitializingBean);
   // 是否存在方法 "afterPropertiesSet"
   if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
      if (logger.isTraceEnabled()) {
         logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
      }
      if (System.getSecurityManager() != null) {
         try {
            // 执行 afterPropertiesSet
            AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
               ((InitializingBean) bean).afterPropertiesSet();
               return null;
            }, getAccessControlContext());
         }
         catch (PrivilegedActionException pae) {
            throw pae.getException();
         }
      }
      else {
         // 执行 afterPropertiesSet
         ((InitializingBean) bean).afterPropertiesSet();
      }
   }

   // 第二部分
   if (mbd != null && bean.getClass() != NullBean.class) {
      // 获取 initMethod 字符串
      String initMethodName = mbd.getInitMethodName();
      if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
         // 自定义的 init method
         invokeCustomInitMethod(beanName, bean, mbd);
      }
   }
}
```

在 `invokeInitMethods` 方法中可以很明显的看到两个处理，这两个处理和这一节开始提到的一样，`InitializingBean#afterPropertiesSet` 执行和 `invokeCustomInitMethod` 自定义 `init-method` 执行。下面我们主要来看 `invokeCustomInitMethod` 的细节



#### 8.1.7.1 `invokeCustomInitMethod` 分析

```java
protected void invokeCustomInitMethod(String beanName, final Object bean, RootBeanDefinition mbd) throws Throwable {
    // 获取 initMethod 名称
    String initMethodName = mbd.getInitMethodName();
    Assert.state(initMethodName != null, "No init method set");
    // 反射获取方法
    Method initMethod = (mbd.isNonPublicAccessAllowed() ? BeanUtils.findMethod(bean.getClass(), initMethodName) : ClassUtils.getMethodIfAvailable(bean.getClass(), initMethodName));
    // 方法是否存在判断
    if(initMethod == null) {
        if(mbd.isEnforceInitMethod()) {
            throw new BeanDefinitionValidationException("Could not find an init method named '" + initMethodName + "' on bean with name '" + beanName + "'");
        } else {
            return;
        }
    }
    // 尝试获取接口方法
    Method methodToInvoke = ClassUtils.getInterfaceMethodIfPossible(initMethod);
    if(System.getSecurityManager() != null) {
        AccessController.doPrivileged((PrivilegedAction < Object > )() - > {
            ReflectionUtils.makeAccessible(methodToInvoke);
            return null;
        });
        // 反射调用
        AccessController.doPrivileged((PrivilegedExceptionAction < Object > )() - > methodToInvoke.invoke(bean), getAccessControlContext());
    } else {
        // 反射调用
        // setAccessible true
        ReflectionUtils.makeAccessible(methodToInvoke);
        methodToInvoke.invoke(bean);
    }
}
```



在这段方法中我们的核心目标是找到方法，在这里提供了三种找方法的形式。

1. 第一种：`BeanUtils.findMethod(bean.getClass(), initMethodName)`
2. 第二种：`ClassUtils.getMethodIfAvailable(bean.getClass(), initMethodName)`
3. 第三种：`ClassUtils.getInterfaceMethodIfPossible(initMethod)`



当我们通过这三种方式找到 `Method` 之后就可以直接执行了。





### 8.1.8 `BeanPostProcessor#postProcessAfterInitialization` 方法

在完成 `InitializingBean` 方法调用后会紧接着做 Bean 后置处理方法的调用 ，这段方法的调用和前置方法的调用形似，只是在执行 `BBeanPostProcessor` 的方法时做了更换，换成了 `postProcessAfterInitialization` 方法。

- `applyBeanPostProcessorsAfterInitialization`

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
      // 执行 spring 容器中 BeanPostProcessor
      Object current = processor.postProcessAfterInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```







当我们做完 `applyBeanPostProcessorsAfterInitialization` 方法之后此时这个 Bean 实例就彻底完成构建了。就可以投入到使用之中，到目前相信各位对 Bean 生命周期中 创建部分的细节有了一定的认识，下面我们来看看摧毁的时候发生了什么





### 8.1.9 Bean 的摧毁

首先我们需要找到进行摧毁操作唤醒的行为是从那个方法开始的，`org.springframework.context.support.AbstractApplicationContext#destroyBeans` 是摧毁 Bean 的主要入口。先来看下面这段代码

- `AbstractApplicationContext#destroyBeans`

```java
protected void destroyBeans() {
    getBeanFactory().destroySingletons();
}
```

这段代码可能看着会莫名其妙，在Spring 中对于摧毁 Bean 是由 `DefaultListableBeanFactory` 和 `DefaultSingletonBeanRegistry`提供的。



#### 8.1.9.1 `DefaultSingletonBeanRegistry` 中的摧毁

我们先来看 `destroySingletons` 方法代码，再来进行分析

- `DefaultSingletonBeanRegistry#destroySingletons` 方法详情

```java
public void destroySingletons() {
  if (logger.isTraceEnabled()) {
      logger.trace("Destroying singletons in " + this);
  }
  synchronized (this.singletonObjects) {
      this.singletonsCurrentlyInDestruction = true;
  }

  String[] disposableBeanNames;
  synchronized (this.disposableBeans) {
      disposableBeanNames = StringUtils.toStringArray(this.disposableBeans.keySet());
  }
  for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
      destroySingleton(disposableBeanNames[i]);
  }

  this.containedBeanMap.clear();
  this.dependentBeanMap.clear();
  this.dependenciesForBeanMap.clear();

  clearSingletonCache();
}
```





在这段方法中我们可以看到有多个变量的清理和标记操作，笔者下面对在这个阶段中用到的各个变量做一个解释



| 变量名称                           | 变量类型                        | 作用                                                         | Java Doc                                                     |
| ---------------------------------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `singletonsCurrentlyInDestruction` | `boolean`                       | 用来标记是否正在进行清除（摧毁）状态                         | Flag that indicates whether we're currently within destroySingletons |
| `disposableBeanNames`              | `String[]`                      | 记录需要进行摧毁的 Bean Name                                 |                                                              |
| `containedBeanMap`                 | `Map<String, Set<String>>`      | Bean 包含关系容器<br />key：Bean Name<br />value：包含 key 的 Bean Name 列表 | Map between containing bean names: bean name to Set of bean names that the bean contains. |
| `dependentBeanMap`                 | `Map<String, Set<String>>`      | Bean 依赖关系容器<br />key：Bean Name<br />value：依赖 key 的 Bean Name 列表 | Map between dependent bean names: bean name to Set of dependent bean. |
| `dependenciesForBeanMap`           | `Map<String, Set<String>>`      | Bean 依赖关系容器                                            | Map between depending bean names: bean name to Set of bean names for the bean's dependencies. |
| `singletonObjects`                 | `Map<String, Object>`           | 单例对象容器<br />key：Bean Name<br />value：Bean 实例       | Cache of singleton objects: bean name to bean instance       |
| `singletonFactories`               | `Map<String, ObjectFactory<?>>` | ObjectFactory 容器<br />key：Bean Name<br />value：ObjectFactory | Cache of singleton factories: bean name to ObjectFactory     |
| `earlySingletonObjects`            | `Map<String, Object>`           | 早期暴露的 Bean 容器<br />key：Bean Name<br />value：Bean 实例 | Cache of early singleton objects: bean name to bean instance |
| `registeredSingletons`             | `Set<String>`                   | 已经注册的单例Bean的名称容器                                 | Set of registered singletons, containing the bean names in registration order |
| `disposableBeans`                  | `Map<String, Object>`           | 会被摧毁的 Bean 容器                                         | Disposable bean instances: bean name to disposable instance. |
|                                    |                                 |                                                              |                                                              |
|                                    |                                 |                                                              |                                                              |
|                                    |                                 |                                                              |                                                              |





在了解这些变量后我们来看，`destroySingleton` 方法，在 `destroySingleton` 方法中会有 `DisposableBean` 接口和 `destroy-method` 调用，摧毁方法在 `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#destroyBean` 中有详细说明，我们来对这个方法进行细节分析。



首先我们来看参数

1. `String beanName`：需要摧毁的 Bean Name
2. `@Nullable DisposableBean bean`：`DisposableBean` 实现类

由于 `DisposableBean` 是一个接口，在参数传递的时候它具体是一个什么类型很重要，从 `disposableBean = (DisposableBean) this.disposableBeans.remove(beanName)` 这段内容中我们可以发现数据是从 `disposableBeans` 中获取的，但是 `disposableBeans` 的 `value` 类型是 `Object`，各位可以不必惊慌，由笔者来告诉各位这里的具体对象类型是什么。

在 `DefaultSingletonBeanRegistry` 中提供了 `registerDisposableBean` 方法，这个注册方法 `registerDisposableBean` 的调用链上我们就可以看到具体的值类型是什么了。下面我们来找 `registerDisposableBean` 的调用。



![image-20210115085052708](./images/image-20210115085052708.png)



通过搜索我们可以发现对于 `registerDisposableBean` 而言我们放入的 `DisposableBean` 是 `DisposableBeanAdapter` 对象。

对于 `DefaultSingletonBeanRegistry#destroyBean` 的参数认识我们已经完成，下面我们来看整个代码

- `DefaultSingletonBeanRegistry#destroyBean` 详细信息

```java
// 删除异常处理和日志相关代码
protected void destroyBean(String beanName, @Nullable DisposableBean bean) {
  // Trigger destruction of dependent beans first...
  Set<String> dependencies;
  synchronized (this.dependentBeanMap) {
      // Within full synchronization in order to guarantee a disconnected Set
      // 移除依赖bean
      dependencies = this.dependentBeanMap.remove(beanName);
  }
  if (dependencies != null) {
      // 依赖列表里面的也删除
      for (String dependentBeanName : dependencies) {
          destroySingleton(dependentBeanName);
      }
  }

  // Actually destroy the bean now...
  if (bean != null) {
      bean.destroy();

  }

  // Trigger destruction of contained beans...
  Set<String> containedBeans;
  synchronized (this.containedBeanMap) {
      // Within full synchronization in order to guarantee a disconnected Set
      // 别名列表
      containedBeans = this.containedBeanMap.remove(beanName);
  }
  if (containedBeans != null) {
      // 删除 别名列表中的beanName
      for (String containedBeanName : containedBeans) {
          destroySingleton(containedBeanName);
      }
  }

  // Remove destroyed bean from other beans' dependencies.
  synchronized (this.dependentBeanMap) {
      // 依赖beanMap中删除beanName
      for (Iterator<Map.Entry<String, Set<String>>> it = this.dependentBeanMap.entrySet().iterator(); it.hasNext(); ) {
          Map.Entry<String, Set<String>> entry = it.next();
          Set<String> dependenciesToClean = entry.getValue();
          dependenciesToClean.remove(beanName);
          if (dependenciesToClean.isEmpty()) {
              it.remove();
          }
      }
  }

  // Remove destroyed bean's prepared dependency information.
  this.dependenciesForBeanMap.remove(beanName);
}
```

  

在这个方法中我们需要关注处理顺序。在前文笔者提到了很多容器，在这里会进行使用。

- 摧毁顺序
  1. 第一个：`dependentBeanMap` ，**Bean 依赖关系容器中先删除依赖项**
  2. 第二个：**调用传入参数 Bean 的摧毁方法，具体实现类 `DisposableBeanAdapter`**
  3. 第三个：`containedBeanMap` ，**Bean 包含关系容器中删除需要当前 Bean 依赖的对象**
  4. 第四个：`dependentBeanMap` 二次处理
  5. 第五个：`dependenciesForBeanMap` 中删除当前 Bean 的依赖信息



在这五个摧毁操作中最后其实都会回到单个对象的摧毁操作上，那么我们的核心目标就是 `DisposableBeanAdapter` 中的摧毁方法是如何实现的。下面我们就来看这个方法的细节



- `DisposableBeanAdapter#destroy`

```java
public void destroy() {
  if(!CollectionUtils.isEmpty(this.beanPostProcessors)) {
      // 找到 DestructionAwareBeanPostProcessor
      for(DestructionAwareBeanPostProcessor processor: this.beanPostProcessors) {
          processor.postProcessBeforeDestruction(this.bean, this.beanName);
      }
  }
  if(this.invokeDisposableBean) {
      // 如果 bean 本身实现了 DisposableBean 接口会进行调用
      if(System.getSecurityManager() != null) {
          AccessController.doPrivileged((PrivilegedExceptionAction < Object > )() - > {
              ((DisposableBean) this.bean).destroy();
              return null;
          }, this.acc);
      } else {
          ((DisposableBean) this.bean).destroy();
      }
  }
  // 自定义摧毁方法的调用
  if(this.destroyMethod != null) {
      invokeCustomDestroyMethod(this.destroyMethod);
  } else if(this.destroyMethodName != null) {
      Method methodToInvoke = determineDestroyMethod(this.destroyMethodName);
      if(methodToInvoke != null) {
          invokeCustomDestroyMethod(ClassUtils.getInterfaceMethodIfPossible(methodToInvoke));
      }
  }
}
```

在这个方法上面我们可以看到很清晰的三层处理逻辑：

1. 第一层：处理容器中 `DestructionAwareBeanPostProcessor` 的实现类 
2. 第二层：如果当前正在被摧毁的 Bean 是 `DisposableBean` 的实现类则调用 `destroy` 方法
3. 第三层：执行 `bean` 标签中 `destroy-method` 所对应的方法



通过这样的一个分析我们对 `DefaultSingletonBeanRegistry#destroyBean` 方法应该有一个比较全面的认识了，下面我们来看`DefaultListableBeanFactory`  中的摧毁实现



#### 8.1.9.2 `DefaultListableBeanFactory`  中的摧毁

首先 ``DefaultListableBeanFactory`  是 `DefaultSingletonBeanRegistry` 的子类，我们来看 `DefaultListableBeanFactory#destroySingleton`  的完整代码

- `DefaultListableBeanFactory#destroySingleton` 

```java
@Override
public void destroySingleton(String beanName) {
  // 摧毁方法的调用
  super.destroySingleton(beanName);
  // 删除 manualSingletonNames 中的BeanName
  removeManualSingletonName(beanName);
  //allBeanNamesByType clean
  //singletonBeanNamesByType clean
  clearByTypeCache();
}
```

这段代码中分了多个处理操作，

1. 第一步：调用父类的摧毁方法，分析内容可以看前一节的内容做一个了解
2. 第二步：将容器 `manualSingletonNames` 中剔除当前的 Bean Name
3. 第三步：清除类型缓存

这里我们主要理解类型缓存是什么，类型缓存的清理指向两个变量 `allBeanNamesByType` 和 `singletonBeanNamesByType`

这两个变量的存储结构相同都是 `Map<Class<?>, String[]>` 但是存储的数量存在差异，`allBeanNamesByType` 存储了所有，但是 `singletonBeanNamesByType` 只存储单例的 Bean。





**注意：当 Bean 的 `scope` 属性是 `prototype` 时候是不会执行 `DisposableBean#destroy` 和 `destroy-method`的！**    





## 8.2 总结

到此笔者对于 Bean 生命周期的各项细节都已经完成了分析，在本文所提到的 Bean 生命周期种关于创建相关的是在 `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean` 方法中产生的，关于摧毁的是在 `org.springframework.context.support.AbstractApplicationContext#doClose`  方法中产生。

从 Bean Factory 的注释上我们对 Bean 生命周期有一个基本的概念后我们做了一个测试用例，从这个测试用例出发我们围绕 Bean 生命周期的每个阶段进行了流程追踪和细节代码的分析以及思考了我们如果自己做会怎么去实现，并且引出了关于 Java Bean 这方面的内容（主要由 `java.beans` 下的内容组成，这也是 Spring 中操作 Bean 的核心），这部分笔者会单独开一章作为额外的只是补充。


