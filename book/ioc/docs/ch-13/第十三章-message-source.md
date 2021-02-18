# 第十三章 Message Source
在这一章中笔者将和各位读者一起讨论 Spring 中对于国际化相关的一些处理

## 13.1 基本环境搭建
本节笔者将和各位共同搭建一个用来模拟国际化的环境。首先我们来确认处理流程。

1. 第一步：编写一个或多个数据源。
2. 第二步：配置 Spring XML 
3. 第三步：使用

在了解处理步骤后我们来编写相关代码

首先我们要准备一个或者多个数据源，这里我们先来了解数据源文件名称的定义。

在 Spring 中使用 **前缀_[语言代码]_[国家/地区代码].properties** 方式进行文件命名，存放位置在 `resources` 中。

下面我们先来做第一步操作。

- `messages_en_US.properties`

```properties
home=Home
format_data={0}.abc
```

- `messages_zh_CN.properties`

```properties
home=jia
```

如果需要使用中文作为对应值请在往上搜索 **中文转 unicode 编码**的工具

```properties
home=\u5bb6
```

如果直接编写中文会出现乱码的情况。



继续我们来编写 Spring XML 配置文件

- `message-source.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

   <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
      <property name="basename">
         <value>messages</value>
      </property>
   </bean>
</beans>
```

最后我们来编写测试用例

```java
@Test
void testXml() {
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/message-source.xml");

    String usHome = context.getMessage("home", null, Locale.US);
    assert usHome.equals("Home");
    String zhHome = context.getMessage("home", null, Locale.CHINESE);
    assert zhHome.equals("jia");

    String format_data = context.getMessage("format_data", new Object[] {"abc"}, Locale.US);
    System.out.println(format_data);

}
```





## 13.2 `MessageSource` 的实例化

在第十章的时候笔者和各位介绍过 `initMessageSource` 方法，`MessageSource` Bean 的创建也就是在这里进行。我们来看这段代码

- `initMessageSource` 方法详情

```java
protected void initMessageSource() {
    // 获取 beanFactory
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    // 判断容器中是否存在 messageSource 这个beanName
    // 存在的情况
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        // 获取 messageSource 对象
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);

        // 设置 父 MessageSource
        // Make MessageSource aware of parent MessageSource.
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                // Only set parent context as parent MessageSource if no parent MessageSource
                // registered already.
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
    }
    // 不存在的情况
    else {
        //  MessageSource 实现类
        // Use empty MessageSource to be able to accept getMessage calls.
        DelegatingMessageSource dms = new DelegatingMessageSource();
        // 设置父 MessageSource
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        // 注册 MessageSource
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
    }
}
```

在这段代码中我们知道了一个关键字 `MESSAGE_SOURCE_BEAN_NAME` -> `messageSource` ，这就是我们在 Spring XML 中配置的 bean id 一致，这样就会进入 `if` 而不是进入 `else` 了。对于 `getBean` 笔者就不再这里赘述了，各位读者可以查看第九章。下面我们来看 `messageSource` 对象的数据内容。

- `messageSource` 初始化的信息

![image-20210127155108912](./images/image-20210127155108912.png)

 	这里我们需要关注的信息是 `basenameSet` ，在 `basenameSet` 存储的内容是我们在 Spring XML 中配置的 `basename` 属性。这个属性会为后续的读取配置文件提供帮助。





## 13.3 `getMessage` 分析

通过测试用例我们进入源码经过查找可以找到 `getMessage` 的具体方法：`org.springframework.context.support.AbstractMessageSource#getMessage(java.lang.String, java.lang.Object[], java.util.Locale)`

- `getMessage` 主要处理代码

```java
@Override
public final String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException {
   String msg = getMessageInternal(code, args, locale);
   if (msg != null) {
      return msg;
   }
   String fallback = getDefaultMessage(code);
   if (fallback != null) {
      return fallback;
   }
   throw new NoSuchMessageException(code, locale);
}
```



在`getMessageInternal` 方法中有我们想要知道的处理过程，下面请各位先阅读代码

```JAVA
@Nullable
protected String getMessageInternal(@Nullable String code, @Nullable Object[] args, @Nullable Locale locale) {
    // code 不存在，返回空
    if (code == null) {
        return null;
    }
    // Locale 不存在，设置默认
    if (locale == null) {
        locale = Locale.getDefault();
    }
    // 需要替换的真实数据
    Object[] argsToUse = args;

    // 是否需要进行消息解析，真实数据是否存在
    if (!isAlwaysUseMessageFormat() && ObjectUtils.isEmpty(args)) {
        // Optimized resolution: no arguments to apply,
        // therefore no MessageFormat needs to be involved.
        // Note that the default implementation still uses MessageFormat;
        // this can be overridden in specific subclasses.

        // 解析消息
        String message = resolveCodeWithoutArguments(code, locale);
        if (message != null) {
            return message;
        }
    }

    else {
        // Resolve arguments eagerly, for the case where the message
        // is defined in a parent MessageSource but resolvable arguments
        // are defined in the child MessageSource.
        argsToUse = resolveArguments(args, locale);

        MessageFormat messageFormat = resolveCode(code, locale);
        if (messageFormat != null) {
            synchronized (messageFormat) {
                return messageFormat.format(argsToUse);
            }
        }
    }

    // Check locale-independent common messages for the given message code.
    Properties commonMessages = getCommonMessages();
    if (commonMessages != null) {
        String commonMessage = commonMessages.getProperty(code);
        if (commonMessage != null) {
            return formatMessage(commonMessage, args, locale);
        }
    }

    // Not found -> check parent, if any.
    return getMessageFromParent(code, argsToUse, locale);
}
```

在这段代码中总共会处理下面4种方式

1. 第一种：`code` 不存在
2. 第二种：不需要进行消息解析并且消息体为空
3. 第三种：需要进行消息解析并且消息体不为空
4. 第四种：其他

第一种情况直接返回 `null` 就结束了内容不多，我们主要关注在第二第三这两种处理方法上，下面我们先来看第二种处理方式





### 13.3.1 `resolveCodeWithoutArguments` 分析

方法 `resolveCodeWithoutArguments` 存在多个实现类这里我们着重关注在 Spring XML 种配置的 `ResourceBundleMessageSource` 类

- `resolveCodeWithoutArguments` 的多种实现

![image-20210127162904940](./images/image-20210127162904940.png)



下面我们来看 `ResourceBundleMessageSource` 中的代码

- `ResourceBundleMessageSource#resolveCodeWithoutArguments` 方法详情

```java
@Override
protected String resolveCodeWithoutArguments(String code, Locale locale) {
    Set<String> basenames = getBasenameSet();
    for (String basename : basenames) {
        ResourceBundle bundle = getResourceBundle(basename, locale);
        if (bundle != null) {
            String result = getStringOrNull(bundle, code);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```

在 这段代码中我们看到了一个熟悉的函数 `getBasenameSet()` ，该函数将返回 Spring XML 中配置的数据，根据配置我们可以知道现在这个值是 `messages`，通过 `getResourceBundle(basename, locale)` 方法我们可以得到 `ResourceBundle` 对象。下面我们来看 `getResourceBundle` 中式如何操作得到 `ResourceBundle` 对象的。

```java
@Nullable
protected ResourceBundle getResourceBundle(String basename, Locale locale) {
   if (getCacheMillis() >= 0) {
      // Fresh ResourceBundle.getBundle call in order to let ResourceBundle
      // do its native caching, at the expense of more extensive lookup steps.
      return doGetBundle(basename, locale);
   }
   else {
      // Cache forever: prefer locale cache over repeated getBundle calls.
      Map<Locale, ResourceBundle> localeMap = this.cachedResourceBundles.get(basename);
      if (localeMap != null) {
         ResourceBundle bundle = localeMap.get(locale);
         if (bundle != null) {
            return bundle;
         }
      }
      try {
         ResourceBundle bundle = doGetBundle(basename, locale);
         if (localeMap == null) {
            localeMap = new ConcurrentHashMap<>();
            Map<Locale, ResourceBundle> existing = this.cachedResourceBundles.putIfAbsent(basename, localeMap);
            if (existing != null) {
               localeMap = existing;
            }
         }
         localeMap.put(locale, bundle);
         return bundle;
      }
      catch (MissingResourceException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("ResourceBundle [" + basename + "] not found for MessageSource: " + ex.getMessage());
         }
         // Assume bundle not found
         // -> do NOT throw the exception to allow for checking parent message source.
         return null;
      }
   }
}
```

在这段代码中分两种获取方式分别如下

1. 第一种：从容器中获取

   缓存结构

```java
Map<String, Map<Locale, ResourceBundle>> cachedResourceBundles
```

1. 第二种：通过 `doGetBundle` 方法获取

   在 `doGetBundle` 方法中最终会调用 JDK 提供的 `ResourceBundle.getBundle` 方法 ，JDK中的代码笔者就不进行分析了。我们抛开 JDK 的细节我们可以简单思考成这样一种方式：通过字符串拼接组成（组成规则： `basename` + `lang`+ `country` + `.properties` ），接着读取该文件的信息。



下面我们来看经过 `getResourceBundle` 方法得到的 `bundle` 中的一些数据情况

- `bundle` 的数据信息

![image-20210127164946938](./images/image-20210127164946938.png)





在得到 `getStringOrNull(bundle, code)` 对象之后我们就可以进行映射关系转换了，在我们的例子中 `code` 是 `home` ，国际化文件中与之对应的是 `Home` 经过 `getStringOrNull(bundle, code)` 方法就会得到这个 `Home` 数据，以此完成调用。在上图中我们可以看到这段代码的含义就是通过 `code` 在 `bundle` 的 `lookup` 容器中找到对应的值。



现在我们对 `resolveCodeWithoutArguments` 的执行过程已经了解了，下面我们来看另一种转换方式 `resolveCode`





### 13.3.2 `resolveCode` 分析

前文笔者和各位一起探讨了关于 `code` 通过国际化文件直接取值的方式，在 Spring 中还提供了通过 `code` + 占位符取值的方式，我们来看一个测试用例。

- 测试用例

```java
String format_data = context.getMessage("format_data", new Object[] {"abc"}, Locale.US);
assert format_data.equals("abc.abc");
```



在 `AbstractMessageSource#getMessageInternal` 方法中我们可以找到下面这段代码

- `format` 消息的核心

```java
argsToUse = resolveArguments(args, locale);

MessageFormat messageFormat = resolveCode(code, locale);
if (messageFormat != null) {
   synchronized (messageFormat) {
      return messageFormat.format(argsToUse);
   }
}
```



在这段代码中我们可以看到三个处理

1. 第一个：将参数`args` 进行转换。

2. 第二个：通过 `code` 和 `locala` 在消息文件中将对应的内容转换成 `MessageFormat`
3. 第三个：`MessageFormat` 和 转换后的 `args` 解析得到最终数据

我们先来看第一个处理过程中使用的方法 `resolveArguments`

- `resolveArguments` 方法详情

```java
@Override
protected Object[] resolveArguments(@Nullable Object[] args, Locale locale) {
   if (ObjectUtils.isEmpty(args)) {
      return super.resolveArguments(args, locale);
   }
   List<Object> resolvedArgs = new ArrayList<>(args.length);
   for (Object arg : args) {
      if (arg instanceof MessageSourceResolvable) {
         resolvedArgs.add(getMessage((MessageSourceResolvable) arg, locale));
      }
      else {
         resolvedArgs.add(arg);
      }
   }
   return resolvedArgs.toArray();
}
```

通过这段代码阅读我们可以新增测试用例 

```java
String format_data2 = context.getMessage("format_data", new Object[] {new MessageSourceResolvable() {
   @Override
   public String[] getCodes() {
      return new String[] {"home"};
   }
}}, Locale.US);
```

对于 `MessageSourceResolvable` 类型的处理方式就是将每个 `code` 在消息文件中找一遍对应的值。如果不是 `MessageSourceResolvable` 类型则忽略直接返回。我们来看两个用例的解析结果

- 用例1

  ![image-20210128091038453](./images/image-20210128091038453.png)

- 用例2

  ![image-20210128091050534](./images/image-20210128091050534.png)





下面我们将第二步和第三步一起来看在 Spring 中这两步我们可以通过Java中的这个方法代替

```java
String messageFormat = MessageFormat.format("{0}.abc", "aaa");
```

- `MessageFormat.format` 在 Java 中的实现

```java
public static String format(String pattern, Object ... arguments) {
    MessageFormat temp = new MessageFormat(pattern);
    return temp.format(arguments);
}
```

在 Java 中的 `new MessageFormat(pattern);` 对应的就是 `resolveCode(code, locale)`，我们来整理一下这个处理方式

1. 通过 `basename` + `locale` 找到对应的 `message` 文件
2. 通过 `code` 在 `message` 文件中找到对应的字符串
3. 将字符串创建成  `MessageFormat`



最后第三步 `format` 的调用如果要深入的话就需要进入 Java 源码了在这里就不进行展开了。





## 13.4 总结

在这一章节中笔者和各位共同探讨了关于国际化相关的一些内容，通过这一章节的分析我们了解了在 Spring 中负责处理消息的几个核心对象

1. `MessageSource` ：消息源接口，用来获取消息
2. `MessageSourceResolvable` ：消息源解析接口

同时我们还了解了二种消息的处理

1. 第一种：直接通过 `code` 获取消息文件中对应的数据
2. 第二种：通过 `code` 加替换参数获取消息文件中对应的数据并根据占位符进行替换



