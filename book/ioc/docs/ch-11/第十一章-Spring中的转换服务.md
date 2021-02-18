# 第十一章 Spring 中的转换服务
在第九章中我们在 `doGetBean` 方法最后看到了下面这段代码 `getTypeConverter().convertIfNecessary(bean, requiredType);` 在这段代码中我们引出了 `TypeConverter` 这个接口, 本章将围绕 `TypeConverter` 进行分析。



## 11.1 初识 Spring 转换服务

在 Spring Core 工程中提供了很多默认的转换服务，这些转换服务基本上都是基础类型的实现，如果我们要自定义一个转换规则应该如何操作呢？本节将会和各位先来探讨自定义转换服务的编写。

这里我们就举一个简单的例子将一个字符串转换成其他对象，首先我们定义一个 `Product` 对象

- `Product`

```java
public class Product {
 private String name;

 private BigDecimal price;
}
```

在完成转换对象的定义后我们需要着手编写转换规则

- 字符串转换成 `Product` 规则

```java
public class StringToProduct implements Converter<String, Product> {
 @Override
 public Product convert(String source) {
    String[] split = source.split(";");
    return new Product(split[0],new BigDecimal(split[1]));
 }
}
```

在完成 Java 代码编写后我们需要来进行 Spring XML 配置

- `convert.xml`

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://www.springframework.org/schema/beans"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


 <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
       <list>
          <bean class="com.source.hot.ioc.book.convert.StringToProduct"/>
       </list>
    </property>
 </bean>
</beans>
```

完成所有准备工作后最后来编写测试用例

- `ConvertTest`

```java
class ConvertTest {
 @Test
 void convertTest() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("/META-INF/convert.xml");
    ConversionService bean = context.getBean(ConversionService.class);
    Product convert = bean.convert("product;10.00", Product.class);
    System.out.println();
 }

}
```

在这个例子中我们希望通过字符串分割后第一部分作为 `Product` 的名称, 第二部分作为 `Product` 的价格。从测试用例中我们可以看到下面的结果

![image-20210120131934246](./images/image-20210120131934246.png)



这样我们就完成了对 Spring 中转换服务的基本使用。下面笔者将和各位聊一聊 Spring 中对于转换服务的具体实现。







## 11.2 `ConversionServiceFactoryBean` 的实例化

从我们的配置文件上我们可以看到在配置阶段我们选择的是 `org.springframework.context.support.ConversionServiceFactoryBean` 类 ，并且设置了 `converters` 的信息。我们首先要关注的就是 `ConversionServiceFactoryBean` 的实例化。

进入 `ConversionServiceFactoryBean` 我们来看这个类的结构，在 `ConversionServiceFactoryBean` 中我们可以看到实现了 `InitializingBean` 接口，知道这个信息后我们首先需要关注的方法是 `afterPropertiesSet`，此外`ConversionServiceFactoryBean` 还实现了 `FactoryBean` 接口。



### 11.2.1 `afterPropertiesSet` 细节分析

这一节笔者将对  `org.springframework.context.support.ConversionServiceFactoryBean#afterPropertiesSet` 进行分析。我们先来看整段代码

```java
@Override
public void afterPropertiesSet() {
   this.conversionService = createConversionService();
   ConversionServiceFactory.registerConverters(this.converters, this.conversionService);
}
```

总共两行代码我们逐行分析。

1. 第一行代码：通过 `createConversionService` 方法创建了 `GenericConversionService` 对象, 实际对象是 `DefaultConversionService`

```java
protected GenericConversionService createConversionService() {
  return new DefaultConversionService();
}
```

2. 第二行代码：注册转换服务





#### 11.2.1.1 `GenericConversionService` 的创建

前文笔者提到了关于 `GenericConversionService` 的创建过程 (`new DefaultConversionService();`) 这里我们需要来了解 `DefaultConversionService` 的构造方法。

- `DefaultConversionService` 构造方法

```java
public DefaultConversionService() {
   addDefaultConverters(this);
}
```



在这个构造方法中我们可以看到继续调用方法 `addDefaultConverters`，在 `addDefaultConverters` 方法中会往容器中注册 Spring 提供的各个转换服务。这里我们对于注册了那些内容我们可以暂时忽略，主要关注注册方法和存储容器。提供注册的方法是 `ConverterRegistry` 接口。

- `ConverterRegistry` 代码详情

```java
public interface ConverterRegistry {

 void addConverter(Converter<?, ?> converter);

 <S, T> void addConverter(Class<S> sourceType, Class<T> targetType, Converter<? super S, ? extends T> converter);

 void addConverter(GenericConverter converter);

 void addConverterFactory(ConverterFactory<?, ?> factory);

 void removeConvertible(Class<?> sourceType, Class<?> targetType);

}
```

在 `ConverterRegistry` 中我们可以看到总共有四个注册方法，一个移除方法

| 方法名称                                                     | 参数说明                                                     | 方法作用                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------ |
| `void addConverter(Converter<?, ?> converter);`              | `converter`  实现了 `Converter` 接口的对象                   | 注册转换服务                               |
| `<S, T> void addConverter(Class<S> sourceType, Class<T> targetType, Converter<? super S, ? extends T> converter);` | `sourceType`：原始对象类型<br />`targetType`：目标对象类型<br />`converter`：提供转换服务的对象 | 注册转换服务                               |
| `void addConverter(GenericConverter converter);`             | `converter`：实现了 `GenericConverter` 的对象                | 注册转换服务                               |
| `void addConverterFactory(ConverterFactory<?, ?> factory);`  | `factory`：实现了 `ConverterFactory` 的对象                  | 注册转换服务                               |
| `void removeConvertible(Class<?> sourceType, Class<?> targetType);` | `sourceType`：原始对象类型<br />`targetType`：目标对象类型<br /> | 根据原始对象类型和目标对象类型移除转换服务 |



**在 `GenericConversionService` 构造方法中最重要的就是将各个默认定义的转换服务注册**，下面我们来看 `ConverterRegistry` 的实现。



#### 11.2.1.2  `ConverterRegistry` 转换服务注册

在 `DefaultConversionService` 的构造方法中我们看到了提供注册能力的是 `this` 也就是 `DefaultConversionService` 本身，最终提供 `ConverterRegistry` 实现方法的其实是 `DefaultConversionService` 的父类 `GenericConversionService` ，现在我们找到了分析目标 。**`ConverterRegistry`  的实现提供者是 `GenericConversionService`** ，下面笔者将和各位分析四个注册方法



##### 11.2.1.2.1 转换器注册方式1 - `Converter` 直接注册

首先我们来看代码实现



```java
@Override
public void addConverter(Converter<?, ?> converter) {
   // 获取解析类型
   ResolvableType[] typeInfo = getRequiredTypeInfo(converter.getClass(), Converter.class);
   if (typeInfo == null && converter instanceof DecoratingProxy) {
      typeInfo = getRequiredTypeInfo(((DecoratingProxy) converter).getDecoratedClass(), Converter.class);
   }
   if (typeInfo == null) {
      throw new IllegalArgumentException("Unable to determine source type <S> and target type <T> for your " +
            "Converter [" + converter.getClass().getName() + "]; does the class parameterize those types?");
   }
   // 添加 converter
   addConverter(new ConverterAdapter(converter, typeInfo[0], typeInfo[1]));
}
```



在这个方法实现中我们要理解两个内容，第一个是 `ResolvableType` 类 ，第二个是 `addConverter` 方法

我们先来理解 `ResolvableType` 是什么，`ResolvableType` 是 Spring 对 `java.lang.reflect.Type` 的一个封装，我们可以简单理解成这个变量中有类的信息。在这里我们看到的 `typeInfo` 是一个数组，笔者在这里直接告诉大家这里存储的是**`Converter` 接口的两个泛型**，在我们的例子中 `Converter<String, Product>` 这里可以直接理解成数组变量存储了 `String` 类型和 `Product` 类型

- `typeInfo` 信息

  ![image-20210121101801716](./images/image-20210121101801716.png)



    `getRequiredTypeInfo` 方法细节各位可以自行展开，主要是反射技术的使用。



现在我们对 `ResolvableType[]` 有一个认识了下面我们来看第二个关注点 `addConverter`

在这里我们可以看到参数是 `ConverterAdapter` 对象，先来看看 `ConverterAdapter` 类图

- `ConverterAdapter` 类图

  ![ConverterAdapter](./images/ConverterAdapter.png)



通过类图查看我们可以看到 `ConverterAdapter` 是 `GenericConverter` 类型，那么这段方法的调用会归到第三种注册方式 `GenericConverter` 的注册。

在看第三种注册方式之前我们需要先来了解 `ConverterAdapter` 的内容。



- `ConverterAdapter` 成员变量表

| 变量名称     | 变量类型                    | 说明                         |
| ------------ | --------------------------- | ---------------------------- |
| `converter`  | `Converter<Object, Object>` | 转换接口提供者               |
| `typeInfo`   | `ConvertiblePair`           | 存储源数类型和目标类型的对象 |
| `targetType` | `ResolvableType`            | 转换目标的类型描述           |



对于第三种注册方式笔者会在下面进行分析，下面我们先来看第二种注册方式。



##### 11.2.1.2.2 转换器注册方式2 - 原始类型+目标类型+`Converter` 注册

我们先来看第二种注册方式的代码

```java
@Override
public <S, T> void addConverter(Class<S> sourceType, Class<T> targetType, Converter<? super S, ? extends T> converter) {
   // 添加 convert 的适配器对象
   addConverter(new ConverterAdapter(
         converter, ResolvableType.forClass(sourceType), ResolvableType.forClass(targetType)));
}
```

在这段代码中我们可以看到这里创建的对象依然还是 `ConverterAdapter` ，抛开 `ConverterAdapter` 以外我们还有一个关于 `ResolvableType.forClass(sourceType)` 的内容可以进行分析。在 `ResolvableType.forClass()` 方法中本质上指向的是



```java
private ResolvableType(@Nullable Class<?> clazz) {
    this.resolved = (clazz != null ? clazz : Object.class);
    this.type = this.resolved;
    this.typeProvider = null;
    this.variableResolver = null;
    this.componentType = null;
    this.hash = null;
}
```

这里我们主要关注 `resolved` 就好了，这个属性存储了类型。



下面我们来看我们在注册方式一和注册方式二中都使用过的方法了。



##### 11.2.1.2.3 转换器注册方式3 - `GenericConverter` 注册

先来看注册方法的实现

```java
@Override
public void addConverter(GenericConverter converter) {
   // 加入 convert 接口
   this.converters.add(converter);
   // 缓存清除
   invalidateCache();
}
```

这里有两个执行逻辑

1. 第一个执行逻辑：向对象 `Converters` 中加入 `GenericConverter` 对象
2. 第二个执行逻辑：



###### 11.2.1.2.3.1 添加 GenericConverter

在查看 `Converters#add` 方法之前我们先来认识一下 `Converters` 的成员变量

| 变量名称           | 变量类型                                  | 说明                                                         |
| ------------------ | ----------------------------------------- | ------------------------------------------------------------ |
| `globalConverters` | `Set<GenericConverter>`                   | 存储 `GenericConverter` 的容器，全局共享的转换服务，没有类型约束 |
| `converters`       | `Map<ConvertiblePair, ConvertersForPair>` | key: 存储原始类型和目标类型<br />value：存储多个 `GenericConverter` <br /> |

`ConvertersForPair` 中的成员变量



| 变量名称     | 变量类型                       | 说明                           |
| ------------ | ------------------------------ | ------------------------------ |
| `converters` | `LinkedList<GenericConverter>` | 存储 `GenericConverter` 的容器 |



现在我们对 `Converters` 中的成员变量做了一个基本了解，接下来我们就可以开始进行 `add` 方法的分析了，先看代码

- `Converters#add` 详细代码

```java
public void add(GenericConverter converter) {
   // 获取转换对象 ConvertiblePair
   Set<ConvertiblePair> convertibleTypes = converter.getConvertibleTypes();
   // 判空
   if (convertibleTypes == null) {
      Assert.state(converter instanceof ConditionalConverter,
            "Only conditional converters may return null convertible types");
      this.globalConverters.add(converter);
   }
   else {
      for (ConvertiblePair convertiblePair : convertibleTypes) {
         // 获取 ConvertersForPair对象
         ConvertersForPair convertersForPair = getMatchableConverters(convertiblePair);
         convertersForPair.add(converter);
      }
   }
}
```

在这个 `add` 方法中参数是一个接口，我们需要确认真实类型，通过前面的一些分析我们知道这个类型是 `ConverterAdapter`，下面我们来进行分析

第一行代码：`converter.getConvertibleTypes();`  这段代码是从 `ConverterAdapter` 中获取它的 `typeInfo` 信息，下面我们来看我们的测试用例在这个阶段的到的结果是什么

![image-20210121110211450](./images/image-20210121110211450.png)

当我们没有获取到 `convertibleTypes` 信息后会进行两种不同的操作。

1. 第一种：当 `convertibleTypes` 不存在的时候会将这个转换器放在全局的转换服务容器中(`globalConverters`)
2. 第二种：当 `convertibleTypes` 存在的时候会将转换对象的信息和转换器放入到 `converters` 容器中

在我们的测试用例中会进行第二种操作，我们来看看具体执行过程中的代码

- `ConvertersForPair convertersForPair = getMatchableConverters(convertiblePair);` 方法分析

```java
private ConvertersForPair getMatchableConverters(ConvertiblePair convertiblePair) {
 // 缓存中获取
 ConvertersForPair convertersForPair = this.converters.get(convertiblePair);
 if (convertersForPair == null) {
    // 创建一个空对象
    convertersForPair = new ConvertersForPair();
    this.converters.put(convertiblePair, convertersForPair);
 }
 return convertersForPair;
}
```

  在 `getMatchableConverters` 中的操作就是一个数据缓存的操作细节不多，我们来看执行完成这一步后我们的数据存储

  ![image-20210121111220501](./images/image-20210121111220501.png)

  在执行完成 `getMatchableConverters` 之后 Spring 将对应关系会放在 `converters` 容器之中，但是现在 `value` 还没有数据。下面我们来看 value 的设置

value 的设置依赖这个方法 `convertersForPair.add(converter)` 这也就是一个简单的 `add` 操作 执行后会向 `value->converters` 加入 `StringToProdut` 这个转换接口

![image-20210121111454241](./images/image-20210121111454241.png)

这样我们在前面定义的转换服务就成功注册到容器之中了，在 `converters` 中还有 Spring 的默认支持的转换服务这部分各位请自行查看。





###### 11.2.1.2.3.2 缓存处理

转换器的新增完成了，紧接着就是对于缓存的处理，先来看处理代码

```java
private void invalidateCache() {
   this.converterCache.clear();
}
```

这里就是将容器 `converterCache` 清空, 没有其他操作，我们来看容器的存储信息

- `Map<ConverterCacheKey, GenericConverter> converterCache`

  key：存储转换前对象的描述和转换目标的对象描述

  value：存储转换服务



对于第三种转换器注册笔者就分析完成了，下面我们来看最后一种转换器注册的分析。

##### 11.2.1.2.4 转换器注册方式4 - `ConverterFactory` 注册

我们来看  `ConverterFactory` 的注册的实现代码

```java
@Override
public void addConverterFactory(ConverterFactory<?, ?> factory) {
   // 获取类型信息
   ResolvableType[] typeInfo = getRequiredTypeInfo(factory.getClass(), ConverterFactory.class);
   // 判断 factory 是不是DecoratingProxy
   if (typeInfo == null && factory instanceof DecoratingProxy) {
      // 其中 DecoratingProxy 可以获取 class
      typeInfo = getRequiredTypeInfo(((DecoratingProxy) factory).getDecoratedClass(), ConverterFactory.class);
   }
   if (typeInfo == null) {
      throw new IllegalArgumentException("Unable to determine source type <S> and target type <T> for your " +
            "ConverterFactory [" + factory.getClass().getName() + "]; does the class parameterize those types?");
   }
   // 添加转换器
   addConverter(new ConverterFactoryAdapter(factory,
         new ConvertiblePair(typeInfo[0].toClass(), typeInfo[1].toClass())));
}
```

在方法最后真正意义上进行注册的时候我们会看到此时所依赖的还是第三种注册方式，此时的注册是 `ConverterFactoryAdapter` 对象，我们来看这个注册对象的成员变量

| 变量名称           | 变量类型                           | 说明                         |
| ------------------ | ---------------------------------- | ---------------------------- |
| `converterFactory` | `ConverterFactory<Object, Object>` | 存储转换器工厂               |
| `typeInfo`         | `ConvertiblePair`                  | 存储源数类型和目标类型的对象 |

注册的具体实现是和第三种注册方式一摸一样，下面我们来看一个注册后的结果，这里笔者演示的是 `NumberToNumberConverterFactory` 注册过程

- `ConverterFactoryAdapter` 对象信息

![image-20210121114449150](./images/image-20210121114449150.png)



- 注册后的 `converters` 对象信息

  ![image-20210121114539110](./images/image-20210121114539110.png)







#### 11.2.1.3  `ConversionServiceFactory.registerConverters` 分析

通过这两节的分析相信各位对 `GenericConversionService` 的创建中所做的事情略知一二了，下面回到 `ConversionServiceFactoryBean` 的 `afterPropertiesSet` 中，来看第二行代码的内容。

-  `ConversionServiceFactory.registerConverters` 详细代码

```JAVA
ConversionServiceFactory.registerConverters(this.converters, this.conversionService)
```

这段代码主要处理的是非 Spring 原生提供的转换服务的注册，比如我们在配置中写的 `com.source.hot.ioc.book.convert.StringToProduct`

- `convert.xml`

```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean class="com.source.hot.ioc.book.convert.StringToProduct"/>
        </list>
    </property>
</bean>
```

在配置文件中编写的 `converters` 属性都会通过这个方法进行注册。

我们来理解这两个参数

1. 参数一：`converters` 该参数通过外部配置进行设置，可以填写 `GenericConverter` 、`Converter` 和 `ConverterFactory` 的实现类
2. 参数二：`conversionService` 该参数是提供注册方法和注册容器的核心，用来进行注册并将数据统一收集

在了解参数信息后我们来看具体的方法

```java
public static void registerConverters(@Nullable Set<?> converters, ConverterRegistry registry) {
   if (converters != null) {
      for (Object converter : converters) {
         if (converter instanceof GenericConverter) {
            registry.addConverter((GenericConverter) converter);
         }
         else if (converter instanceof Converter<?, ?>) {
            registry.addConverter((Converter<?, ?>) converter);
         }
         else if (converter instanceof ConverterFactory<?, ?>) {
            registry.addConverterFactory((ConverterFactory<?, ?>) converter);
         }
         else {
            throw new IllegalArgumentException("Each converter object must implement one of the " +
                  "Converter, ConverterFactory, or GenericConverter interfaces");
         }
      }
   }
}
```



在这个方法实现过程中我们可以熟悉的看到三种注册方式，这三种注册方式和笔者在前一节所提到的注册方法是同一个对象进行提供的，这里不在进行展开了，各位可以在看一看前面的分析回忆一下。







## 11.3 转换过程分析

通过前面的分析我们理解了关于 Spring 中对于转换服务提供者的存储和注册，下面我们要来看具体的转换过程发生了什么。

我们先来看如何使用转换的代码

```java
ConversionService bean = context.getBean(ConversionService.class);
Product convert = bean.convert("product;10.00", Product.class);
```



### 11.3.1 `ConversionService` 是谁

我们需要先解决一个疑问：在 Spring XML 配置文件中我们配置的是 `org.springframework.context.support.ConversionServiceFactoryBean` 为什么我们没有用它，而是用的 `ConversionService` 接口。在回答这个问题的时候各位读者需要对 Bean 的获取(创建)有一个了解，各位如果不了解可以参考第九章的内容。

回到 `ConversionServiceFactoryBean` 的代码本身，我们可以看到它实现了 `FactoryBean<ConversionService>` 

- `FactoryBean` 接口的关键方法

```java
@Override
@Nullable
public ConversionService getObject() {
   return this.conversionService;
}

@Override
public Class<? extends ConversionService> getObjectType() {
   return GenericConversionService.class;
}

@Override
public boolean isSingleton() {
   return true;
}
```

在这里我们可以看到这里的已经将 `conversionService` 作为`getObject` 返回了，并且 `getObjectType` 的返回值也是属于 `ConversionService` 类型的，因此我们在向 Spring IoC 容器中获取对象的时候就会调用这里的 `getObject()` 获取 `ConversionService` 的实现类。

我们来看 `getBean` 时候的调用堆栈



![image-20210121134623185](./images/image-20210121134623185.png)

从这个调用堆栈中不难发现在执行 `context.getBean(ConversionService.class)` 最后会来 `getObject` 获取对象，这个对象就是 `DefaultConversionService` 。

这样我们可以下结论了，`ConversionService` 在这个阶段处理中就是 `DefaultConversionService` 。



如果我们要使用 `ConversionServiceFactoryBean` 来进行转换我们可以通过下面这样的方式。

```java
ConversionServiceFactoryBean bean1 = context.getBean(ConversionServiceFactoryBean.class);
Product convert1 = bean1.getObject().convert("product;10.00", Product.class);
```



### 11.3.2 转换方法分析

通过前面的分析我们找到了 `ConversionService` 的实际对象是 `GenericConversionService` ，那么在 `GenericConversionService` 对象中一定提供了对应的转换方法 `convert` ，确实通过搜索我们可以找到这个方法签名 `org.springframework.core.convert.support.GenericConversionService#convert(java.lang.Object, java.lang.Class<T>)` 它就是我们转换方法的核心

![image-20210121143439182](./images/image-20210121143439182.png)

从图中我们可以看到 `source` 和 `targetType` 就是那个我们传进去的参数，最终指向的方法签名 ：`org.springframework.core.convert.support.GenericConversionService#convert(java.lang.Object, org.springframework.core.convert.TypeDescriptor, org.springframework.core.convert.TypeDescriptor)`



先来看整体代码

```java
@Override
@Nullable
public Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType) {
   Assert.notNull(targetType, "Target type to convert to cannot be null");
   if (sourceType == null) {
      Assert.isTrue(source == null, "Source must be [null] if source type == [null]");
      // 处理 sourceType 为空的转换
      return handleResult(null, targetType, convertNullSource(null, targetType));
   }
   // 数据验证
   if (source != null && !sourceType.getObjectType().isInstance(source)) {
      throw new IllegalArgumentException("Source to convert from must be an instance of [" +
            sourceType + "]; instead it was a [" + source.getClass().getName() + "]");
   }
   // 获取转换器接口
   GenericConverter converter = getConverter(sourceType, targetType);
   if (converter != null) {
      // 通过工具获得转换结果
      Object result = ConversionUtils.invokeConverter(converter, source, sourceType, targetType);
      return handleResult(sourceType, targetType, result);
   }
   // 处理找不到 convert 的转换结果
   return handleConverterNotFound(source, sourceType, targetType);
}
```

这个真正负责处理转换的方法中总共存在四种逻辑

1. 第一种：**原类型描述不存在**
2. 第二种：**原对象存在但是原类型和原类型的描述对象中的类型不同源**
3. 第三种：**在转换器容器种存在对应的转换器**
4. 第四种：**在转换器容器种不存在对应的转换器**



#### 11.3.2.1 `handleResult` 分析

从代码上可以看到 `handleResult` 有两个调用地方，这个方法将对象处理后返回，我们直接来看这个方法中关于 `result` 的处理

```java
@Nullable
private Object handleResult(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType, @Nullable Object result) {
   if (result == null) {
      // 判断 target type
      assertNotPrimitiveTargetType(sourceType, targetType);
   }
   return result;
}

private void assertNotPrimitiveTargetType(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType) {
    if (targetType.isPrimitive()) {
    	throw new ConversionFailedException(sourceType, targetType, null,
    		new IllegalArgumentException("A null value cannot be assigned to a primitive type"));
    }
}
```



在 `handleResult` 处理过程中我们可以看到是有可能返回 `null` 的，但是 `targetType.isPrimitive` 条件满足的话会抛出异常。总结一下只要 `targetType.isPrimitive` 不为 `true` 就会原封不动的将 `result` 返回。



#### 11.3.2.2 `getConverter` 获取转换接口

在本章的前半部分笔者和各位讲述了关于 `Converter` 的存储，现在我们需要来获取这个缓存中的数据。



```java
@Nullable
protected GenericConverter getConverter(TypeDescriptor sourceType, TypeDescriptor targetType) {
   ConverterCacheKey key = new ConverterCacheKey(sourceType, targetType);
   GenericConverter converter = this.converterCache.get(key);
   if (converter != null) {
      return (converter != NO_MATCH ? converter : null);
   }

   // 找出 converter 对象
   converter = this.converters.find(sourceType, targetType);
   if (converter == null) {
      // 获取默认的 converter
      converter = getDefaultConverter(sourceType, targetType);
   }

   if (converter != null) {
      // 设置缓存
      this.converterCache.put(key, converter);
      return converter;
   }
   // 设置缓存
   this.converterCache.put(key, NO_MATCH);
   return null;
}
```

主要的搜索方法在 `converter = this.converters.find(sourceType, targetType)` ，`converterCache` 是使用过的缓存。下面我们就来看 `find` 方法的实现逻辑

##### 11.3.2.2.1 `converters.find `分析



- `find` 

```java
@Nullable
public GenericConverter find(TypeDescriptor sourceType, TypeDescriptor targetType) {
 // Search the full type hierarchy
 // 找到 source 类型的类关系和接口关系
 List<Class<?>> sourceCandidates = getClassHierarchy(sourceType.getType());
 // 找到 target 类型的类关系和接口关系
 List<Class<?>> targetCandidates = getClassHierarchy(targetType.getType());
 // 循环 source 的类列表 和 target 的类列表
 for (Class<?> sourceCandidate : sourceCandidates) {
    for (Class<?> targetCandidate : targetCandidates) {
       // 创建 ConvertiblePair 对象
       ConvertiblePair convertiblePair = new ConvertiblePair(sourceCandidate, targetCandidate);
       // 获取 source + target 的转换接口
       GenericConverter converter = getRegisteredConverter(sourceType, targetType, convertiblePair);
       if (converter != null) {
          return converter;
       }
    }
 }
 return null;
}
```

在这里我们先来看 `getClassHierarchy` 方法的作用，代码就不贴出来了。在 `getClassHierarchy` 中主要作用是将原始对象的所有类接口都找出来。这里我们用 `String` 举例，我们来看 `String` 类图

![String](./images/String.png)



在这个类图中还隐藏着 `Object` ，在 `getClassHierarchy` 执行之后就会将 `String` 、`Comparable` 、`Serializable` 、`CharSequence` 和 `Object` 都找出来。

`getClassHierarchy` 方法会对原始对象进行一次，会对目标对象进行一次。值得注意的是通过 `getClassHierarchy` 方法执行后的结果其顺序是**自身->继承类->实现类-> `Object`** 

在得到 `source` 和 `target` 的类关系后会做一个笛卡尔积的搜索。将 `source` 中的类和 `target` 循环一个个组装成 `ConvertiblePair` 然后在容器中寻找 `GenericConverter` 对象，如果找到了就直接返回了，找不到就返回 `null`。下面我们来看 `getRegisteredConverter` 方法的细节



- `getRegisteredConverter`

```java
@Nullable
private GenericConverter getRegisteredConverter(TypeDescriptor sourceType,
      TypeDescriptor targetType, ConvertiblePair convertiblePair) {

   // Check specifically registered converters
   // 从map中获取
   ConvertersForPair convertersForPair = this.converters.get(convertiblePair);
   if (convertersForPair != null) {
      // 获取 GenericConverter
      GenericConverter converter = convertersForPair.getConverter(sourceType, targetType);
      if (converter != null) {
         return converter;
      }
   }
   // Check ConditionalConverters for a dynamic match
   for (GenericConverter globalConverter : this.globalConverters) {
      if (((ConditionalConverter) globalConverter).matches(sourceType, targetType)) {
         return globalConverter;
      }
   }
   return null;
}
```

在 `getRegisteredConverter` 方法之中我们可以看到它的搜索顺序。首先进行 `converters` 的搜索，其次进行全局转换器（`globalConverters`）的搜索。



##### 11.3.2.2.2 `getDefaultConverter` 默认的转换服务

通过这样的一个过程 `find` 方法有可能找到了转换接口，也有可能没有找到转换接口，找到会直接返回，那么没有找到会做什么操作呢？

在 `getConverter` 之中我们可以看到下面这段代码。

```java
if (converter == null) {
   // 获取默认的 converter
   converter = getDefaultConverter(sourceType, targetType);
}
```



```java
protected GenericConverter getDefaultConverter(TypeDescriptor sourceType, TypeDescriptor targetType) {
   return (sourceType.isAssignableTo(targetType) ? NO_OP_CONVERTER : null);
}
```

这就是没有找到的处理。



在这段代码我们可以看到如果是可以转换的  `sourceType.isAssignableTo(targetType)` 判断是否可以进行转换，那么会返回一个没有任何操作的 `Converter` 对象。该对象会将 `source` 直接作为转换结果。各位可以查看下面代码做一个了解

```java
private static class NoOpConverter implements GenericConverter {

    private final String name;

    public NoOpConverter(String name) {
        this.name = name;
    }

    @Override
    public Set<ConvertiblePair> getConvertibleTypes() {
        return null;
    }

    @Override
    @Nullable
    public Object convert(@Nullable Object source, TypeDescriptor sourceType, TypeDescriptor targetType) {
        return source;
    }

    @Override
    public String toString() {
        return this.name;
    }
}
```





#### 11.3.2.3 `ConversionUtils.invokeConverter` 分析

通过签名的各种操作我们得到了一个 `GenericConverter` 对象，接下来我们的目标就是进行转换，这就涉及到 `ConversionUtils.invokeConverter` 方法的使用了，下面我们来看看这个方法做了什么。直接看代码

- `ConversionUtils#invokeConverter` 方法详情

```java
@Nullable
public static Object invokeConverter(GenericConverter converter, @Nullable Object source,
      TypeDescriptor sourceType, TypeDescriptor targetType) {

   try {
      // converter 方法调用
      return converter.convert(source, sourceType, targetType);
   }
   catch (ConversionFailedException ex) {
      throw ex;
   }
   catch (Throwable ex) {
      throw new ConversionFailedException(sourceType, targetType, source, ex);
   }
}
```

从这段代码中我们可以非常直观的看到这里只是一个转换。但是这个转换能力的提供者 `GenericConverter`  是一个接口，我们还需要明确这个接口在实际运算中的类型。在本章的前面提到过 `ConverterAdapter` 和 `ConverterFactoryAdapter` 具体章节地址：[11.2.1.2  `ConverterRegistry` 转换服务注册]。那么我们分两种情况来进行转换方法的分析



##### 11.3.2.3.1 `ConverterAdapter` 的转换

我们来看具体的转换代码



```java
@Override
@Nullable
public Object convert(@Nullable Object source, TypeDescriptor sourceType, TypeDescriptor targetType) {
   if (source == null) {
      // 空对象转换
      return convertNullSource(sourceType, targetType);
   }
   // 转换接口调用
   return this.converter.convert(source);
}
```

从这段代码中我们可以看到两种情况。

1. 第一种：需要转换的对象不存在

   当需要转换得对象不存在的时候会做一次判断是否是 `Optional` 类型，如果是则返回 `Optional.empty()` ， 不是的话直接返回 `null`

```java
@Nullable
protected Object convertNullSource(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType) {
  if (targetType.getObjectType() == Optional.class) {
     return Optional.empty();
  }
  return null;
}
```

2. 第二种：需要转换的对象存在

   执行转换接口的转换方法

```java
this.converter.convert(source)
```

   在一开始注册 `ConverterAdapter` 的时候成员变量中 `converter` 就已经存储好了转换接口，这里我们的转换接口就是 `com.source.hot.ioc.book.convert.StringToProduct` 此时调用这个方法就会来执行 `StringToProduct#convert`





##### 11.3.2.3.2 `ConverterFactoryAdapter` 的转换

对于 `ConverterAdapter` 的分析完成了，下面我们来看  `ConverterFactoryAdapter` 的转换方法，先看代码

```java
@Override
@Nullable
public Object convert(@Nullable Object source, TypeDescriptor sourceType, TypeDescriptor targetType) {
   if (source == null) {
      return convertNullSource(sourceType, targetType);
   }
   // 从工厂中获取一个 convert 接口进行转换
   return this.converterFactory.getConverter(targetType.getObjectType()).convert(source);
}
```

在 `convert` 中还是两种情况。

1. 第一种：需要转换的对象不存在

   当需要转换得对象不存在的时候会做一次判断是否是 `Optional` 类型，如果是则返回 `Optional.empty()` ， 不是的话直接返回 `null`

2. 第二种：需要转换的对象存在

   从 `converterFactory` 中找到转换接口在进行转换

   假设我们现在正在进行 `NumberToNumberConverterFactory` 的转换，在这个方法中它的调用流程如下

   1. 从 `NumberToNumberConverterFactory` 中调用 `getConverter` 方法找到 `Converter` 接口

      - `NumberToNumberConverterFactory#getConverter`

```java
@Override
public <T extends Number> Converter<Number, T> getConverter(Class<T> targetType) {
 return new NumberToNumber<>(targetType);
}
```

   2. 进行转换

      - `NumberToNumber#convert`

```java
@Override
public T convert(Number source) {
 return NumberUtils.convertNumberToTargetClass(source, this.targetType);
}
```



到此笔者对于转换的可能都已经分析完毕。下面我们将来看另一种情况 ：没有找到 `GenericConverter` 





#### 11.3.2.4 `handleConverterNotFound` 分析

下面我们来分析 `GenericConverter` 找不到的情况，在找不到的时候处理方法是 `GenericConversionService#handleConverterNotFound`，我们来看方法细节

```java
private Object handleConverterNotFound(
      @Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType) {

   if (source == null) {
      assertNotPrimitiveTargetType(sourceType, targetType);
      return null;
   }
   if ((sourceType == null || sourceType.isAssignableTo(targetType)) &&
         targetType.getObjectType().isInstance(source)) {
      return source;
   }
   throw new ConverterNotFoundException(sourceType, targetType);
}
```

这里分开多个情况进行说明

1. 情况一：`souce` 对象不存在，通过 `targetType.isPrimitive` 验证返回 `null` ， 没有通过 `targetType.isPrimitive` 抛出异常 `ConversionFailedException`
2. 情况二：`source` 对象不存在或者原类型和目标类型可以转换就返回 `source` 本身
3. 其他情况：抛出转换异常 `ConverterNotFoundException`





这样我们对于 Spring 中的转换过程也分析完成了，下面我们来看一看如果我们脱离 Spring 应该如何实现一个转换服务。



## 11.4 脱离 Spring 的实现

接下来笔者将和各位读者来一起探讨同理 Spring 我们应该如何来做一个转换服务。笔者会借鉴 Spring 中的一些处理方式也会省略一些内容，下面我们开始。

首先要做转换服务我们要做的事情和 Spring 中的一个事情是相同的：**定义一个转换接口**，那我们来定义这个接口

- 转换接口

```java
public interface CommonConvert<S, T> {
    /**
     * 转换
     * @param source 原始对象
     * @return 转换结果对象
     */
    T convert(S source);
}
```



当我们完成转换接口的定义后我们需要统一管理，这里笔者和 Spring 采用同样的方式将 `CommonConvert` 注册到一个统一的容器中，那么我们先来看容器的定义。

- 存储转换接口的容器

```java
static Map<ConvertSourceAndTarget, CommonConvert> convertMap
      = new ConcurrentHashMap<>(256);
```

在这个容器中笔者将 KEY 定义为  `ConvertSourceAndTarget` 这个对象的主要目的是存储 `ConvertSourceAndTarget` 的泛型信息，来看具体代码

- `ConvertSourceAndTarget` 详情

```java
public class ConvertSourceAndTarget {

    /**
     * {@link CommonConvert} 中的 S
     */
    private Class<?> sourceTypeClass;

    /**
     * {@link CommonConvert} 中的 T
     */
    private Class<?> targetTypeClass;
}
```



容器准备完成，下面我们就是来定义注册方法了，这里笔者定义了两个注册方式。

1. 第一种注册方式：通过实例进行注册，`void register(CommonConvert<?, ?> commonConvert);`
2. 第二种注册方式：通过class 进行注册，`void register(Class<? extends CommonConvert> convert) throws IllegalAccessException, InstantiationException;`



下面我们来看两个注册方式的详细实现

```JAVA
@Override
public void register(Class<? extends CommonConvert> convert) throws IllegalAccessException, InstantiationException {
    if (convert == null) {
        log.warn("当前传入的convert对象为空");
        return;
    }
    CommonConvert cv = convert.newInstance();

    if (cv != null) {
        handler(cv, convert);
    }

}

@Override
public void register(CommonConvert commonConvert ) {

    if (commonConvert == null) {
        log.warn("当前传入的convert对象为空");
        return;
    }


    Class<? extends CommonConvert> convertClass = commonConvert.getClass();

    handler(commonConvert, convertClass);

}
```

- 注册方法的核心

```java
private void handler(CommonConvert commonConvert, Class<? extends CommonConvert> convertClass) {
  Type[] genericInterfaces = convertClass.getGenericInterfaces();

  for (Type genericInterface : genericInterfaces) {
      ParameterizedType pType = (ParameterizedType) genericInterface;
      boolean equals = pType.getRawType().equals(CommonConvert.class);
      if (equals) {
          Type[] actualTypeArguments = pType.getActualTypeArguments();

          if (actualTypeArguments.length == 2) {
              Type a1 = actualTypeArguments[0];
              Type a2 = actualTypeArguments[1];

              try {

                  Class<?> sourceClass = Class.forName(a1.getTypeName());
                  Class<?> targetClass = Class.forName(a2.getTypeName());

                  ConvertSourceAndTarget convertSourceAndTarget =
                      new ConvertSourceAndTarget(sourceClass,
                                                 targetClass);
                  // 如果类型相同 覆盖
                  convertMap.put(convertSourceAndTarget, commonConvert);
              }
              catch (Exception e) {
                  log.error("a1=[{}]", a1);
                  log.error("a2=[{}]", a2);
                  log.error("从泛型中转换成class异常", e);
              }
          }
      }
  }
}

```

  在这个正在处理的注册方法的代码中笔者传递了两个参数。

  1. `commonConvert` 转换器实例
  2. `convertClass` 转换器类型

  在处理方法中主要实现逻辑：

  1. 提取转换器上的泛型将其转换成容器中的 KEY ，在将转换器实例作为 VALUE 进行存储



现在我们的存储容器已经准备完成，注册方法也提供完成，我们还缺少一个关于统一转换的入口，我们来定义这个入口和实现方式。

- 统一转换入口

```java
public static <T> T convert(Object source, Class<T> target) {
    if (log.isInfoEnabled()) {
        log.info("convert,source = {}, target = {}", source, target);
    }

    if (source == null || target == null) {
        throw new IllegalArgumentException("参数异常请重新检查");
    }


    ConvertSourceAndTarget  convertSourceAndTarget = new ConvertSourceAndTarget(source.getClass(), target);

    CommonConvert commonConvert = DefaultConvertRegister.getConvertMap(convertSourceAndTarget);
    if (commonConvert != null) {

        return (T) commonConvert.convert(source);
    }
    return null;
}
```



在转换的时候我们需要从转换器容器中获取对应的转换器在进行转换。处理方式就是将需要转换的对下个获取`class` 和转换目标类型组装成 `ConvertSourceAndTarget` ，拿着这个对象去转换器容器中获取转换器，在进行转换即可。

这样我们就做出了一个简易的转换服务，和 Spring 对比笔者这个转换服务提供的注册方式比较单一，对比 Spring 中对于转换接口的泛型处理笔者所选择的方式比较简单没有考虑其他细节内容，同时在依赖泛型的处理中没有考虑类的关系图（笔者这个方案只适合没有继承的对象转换），笔者在这里提出的转换方式是对 Spring 中转换方式的一个简单实现，各位读者还可以进行更多的加工处理。

笔者所编写的转换服务仓库地址：https://gitee.com/pychfarm_admin/convert 



## 11.5 总结

在本章中笔者和各位一起讨论了 Spring 中的转换服务，笔者向各位介绍了如何定义一个转换服务并使用这个转换服务，从这个基础用例开始笔者进一步和各位分析了关于 Spring 中实现和调用转换服务的各个细节，在理解 Spring 的转换服务实现后笔者自己也实现了一个简单的转换服务以此来加深对转换服务的理解。


