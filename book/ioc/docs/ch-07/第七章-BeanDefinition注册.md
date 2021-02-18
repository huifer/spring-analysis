# 第七章 Spring Bean Definition 注册
- 本章笔者将和各位读者介绍 BeanDefinition 注册相关的源码. 

在[第六章](/docs/ch-06/第六章-bean标签解析.md)最后笔者留下了一个问题  Bean Definition 是如何进行注册的。首先我们需要先将注册方法找到



- Bean Definition 注册

```java
BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry())
```





## 7.1 Bean Definition 注册

目标方法我们已经找到，下面请各位读者先进行方法阅读



```java
public static void registerBeanDefinition(
      BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
      throws BeanDefinitionStoreException {

   // Register bean definition under primary name.
   // 获取 beanName
   String beanName = definitionHolder.getBeanName();
   // 注册bean definition
   registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

   // Register aliases for bean name, if any.
   // 别名列表
   String[] aliases = definitionHolder.getAliases();
   // 注册别名列表
   if (aliases != null) {
      for (String alias : aliases) {
         registry.registerAlias(beanName, alias);
      }
   }
}
```



通过这段方法的阅读我们可以明确在  `registerBeanDefinition` 中做了什么。

1. 第一步：Bean Name 和 Bean Definition 的关系绑定。 
2. 第二步：Bean Name 和 Alias 的关系绑定。

第二步 BeanName 和 Alias 的关系处理笔者在[第五章](/docs/ch-05/第五章-别名注册.md)中已经分析完成，各位可以往前翻一翻，本章就针对第一步进行详细分析了。



通过阅读 `registerBeanDefinition` 方法我们可以看到负责提供 Bean Definition 注册的是 `BeanDefinitionRegistry` 接口。第一个关键问题: **具体是 `BeanDefinitionRegistry` 的哪一个实现类提供了这里的方法。**

先来看 `BeanDefinitionRegistry` 的类图

![BeanDefinitionRegistry](./images/BeanDefinitionRegistry.png)



在 Spring 中用到的是 `DefaultListableBeanFactory` 作为 `registerBeanDefinition` 方法的提供者。

-  `BeanDefinitionRegistry` 对象信息

  ![image-20210108152328085](images/image-20210108152328085.png)



这样我们就找到了真正的处理方法，下面就进入对这个方法的分析

## 7.2 DefaultListableBeanFactory 中存储 Bean Definition 的容器

首先我们需要知道我们的分析目标在哪里，分析方法的全路径：`org.springframework.beans.factory.support.DefaultListableBeanFactory#registerBeanDefinition`。

下面我们来思考一个问题，应该如何存储 Bean Definition 对象，在 Java 中和存储容器相关的有 `List` 、`Map` 等，这些存储容器在存储 Bean Definition 时候会有什么问题。

那么我们先来思考 `List` 首先从存储上肯定是可以存储的，但是光存储还不够，我们还需要使用，当使用者想要通过 Bean Name 来得到 Bean Definition 时我们需要循环 `List` 在判断 Bean Definition 的名称是否和我们需要的相同，相同就返回。不过在 Spring 中 `BeanDefinition` 接口并没有提供获取 Bean Name 的方法，各位读者可以自行考虑拓展。

下面我们再来思考 `Map` 这个结构，同样的 `Map` 也可以完成存储的要求。 根据 `Map` 这种键值对存储的形式我们可以直接将 Bean Name 作为 key ，Bean Definition 作为 value 。

关于 `List` 和 `Map` 两者的选择在笔者看来从使用角度上来看时间复杂度上 `Map` 会更有优势。`List` 在本例中获取对象的时间复杂度`O(n)` ，`Map` 在本例中获取对象的时间复杂度 `O(1)`。

最终我们来看看 Spring 中对于 Bean Definition 的存储容器

- `DefaultListableBeanFactory` 中对于 Bean Definition 的存储容器。

```java
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```
    
    在这个容器中 key ： Bean Name ，value：Bean Definition 。



到现在我们找到了存储容器，存储形式，到这儿我们对于 Bean Definition 的存储了解了，下面我们深入代码分析一下内部做了什么。





## 7.3 DefaultListableBeanFactory 中的注册细节

笔者在开始分析之前将真个流程全部整理出来，在对其进行分析。

1. 第一步： Bean Definition 的验证
2. 第二步： Bean Name 在容器中的不同情况处理。
   1. 情况一： Bean Name 已经在容器中存在，并且拥有对应的 Bean Definition 对象。
   2. 情况二：Bean Name 在容器中搜索 Bean Definition 失败。
3. 第三步：根据 Bean Name 进行 Bean Definition 的刷新操作。





### 7.3.1 Bean Definition 的验证

先来看第一步关于 Bean Definition 的验证的具体代码

```java
if (beanDefinition instanceof AbstractBeanDefinition) {
   try {
      // bean定义验证
      ((AbstractBeanDefinition) beanDefinition).validate();
   }
   catch (BeanDefinitionValidationException ex) {
      throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
            "Validation of bean definition failed", ex);
   }
}
```

在 Spring 中通过 `processBeanDefinition` 方法得到的 Bean Definition 的类型是：`GenericBeanDefinition`。来看看 `GenericBeanDefinition` 的类图

![GenericBeanDefinition](./images/GenericBeanDefinition.png)

在类图中我们可以发现 `GenericBeanDefinition` 类型是 `AbstractBeanDefinition` 所以这段方法是可以执行的，下面我们来对验证方法 `validate` 进行了解。

在 Spring 中关于 `validate` 方法的实现提供者有两个，一个是 `AbstractBeanDefinition` 另一个是 `ChildBeanDefinition`。

先来看 `AbstractBeanDefinition#validate` 中提供的方法。

```java
public void validate() throws BeanDefinitionValidationException {
   // 是否存在重写方法, factory_method_name 是否为空
   if (hasMethodOverrides() && getFactoryMethodName() != null) {
      throw new BeanDefinitionValidationException(
            "Cannot combine factory method with container-generated method overrides: " +
                  "the factory method must create the concrete bean instance.");
   }
   //  bean class 是否等于 Class
   if (hasBeanClass()) {
      // 方法重写+验证
      prepareMethodOverrides();
   }
}
```



看完之后可能比较迷糊，主要这些方法都是分装好的不够直观，下面笔者对每个方法做一个描述

1. 第一步：`hasMethodOverrides()` ： `MethodOverrides` 是否为空

   `MethodOverrides` 各位如果忘记了可以在[第六章](/docs/ch-06/第六章-bean标签解析.md)中查看 `lookup-override` 和 `replaced-method` 的处理。

2. 第二步：`getFactoryMethodName()`: 是否存在工厂函数，这里不是 `Method` 对象，在这仅仅只是一个工厂函数的方法名称。

3. 第三步：`hasBeanClass()` : Bean Class 属性是否是 `Class` ，在[第六章-6.3.1 处理 class name  和 parent](/docs/ch-06/第六章-bean标签解析.md) 中提到过 Bean Class 的两种情况，一种是 `String` ，另一种是 `Class` 在这里就是对这个信息的验证。

4. 第四步：`prepareMethodOverrides` ：方法重写标记和验证。详细处理逻辑

   1. 判断是否存在重写的方法，这一点和第一步一样。
   2. 如果需要重写则循环存储 `MethodOverride` 将 `overloaded` 标记设置为 `false`

这些就是关于 `AbstractBeanDefinition#validate` 方法的全部内容了，下面我们来看另一个类 `ChildBeanDefinition` 中的验证逻辑， 请各位先进行 `ChildBeanDefinition#validate` 方法的阅读

- `ChildBeanDefinition#validate` 方法详情

```java
@Override
public void validate() throws BeanDefinitionValidationException {
   super.validate();
   if (this.parentName == null) {
      throw new BeanDefinitionValidationException("'parentName' must be set in ChildBeanDefinition");
   }
}
```

在 `ChildBeanDefinition#validate` 方法中首先调用的是父类 `AbstractBeanDefinition` 中的 `validate` 方法，这段方法的分析在前文提及，在处理完成父类提供的方法之后在 `ChildBeanDefinition` 中对 `parentName` 做了验证，判断 `parentName` 是否存在，不存在则抛出异常 `'parentName' must be set in ChildBeanDefinition` 





### 7.3.2 容器中存在 Bean Name 对应的 Bean Definition 的处理

首先请各位读者先来对这段处理代码进行简要阅读

- 容器中存在 Bean Name 对应的 Bean Definition 的处理

```java
// 第二部分 beanName 存在 BeanDefinition 的情况
// 从 map 中根据 beanName 获取 beanDefinition
BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
if (existingDefinition != null) {
   // bean name 是否允许重复注册
   if (!isAllowBeanDefinitionOverriding()) {
      throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
   }
   // role 值比较
   else if (existingDefinition.getRole() < beanDefinition.getRole()) {
      // e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
      if (logger.isInfoEnabled()) {
         logger.info("Overriding user-defined bean definition for bean '" + beanName +
               "' with a framework-generated bean definition: replacing [" +
               existingDefinition + "] with [" + beanDefinition + "]");
      }
   }
   // map 中存储的 beanDefinition 是否和参数相同
   else if (!beanDefinition.equals(existingDefinition)) {
      if (logger.isDebugEnabled()) {
         logger.debug("Overriding bean definition for bean '" + beanName +
               "' with a different definition: replacing [" + existingDefinition +
               "] with [" + beanDefinition + "]");
      }
   }
   else {
      if (logger.isTraceEnabled()) {
         logger.trace("Overriding bean definition for bean '" + beanName +
               "' with an equivalent definition: replacing [" + existingDefinition +
               "] with [" + beanDefinition + "]");
      }
   }
   // 设置 beanName 和 beanDefinition 关系
   this.beanDefinitionMap.put(beanName, beanDefinition);
}
```

在这段代码中我们可以关注下面这些内容

1.  `existingDefinition` ：通过 Bean Name 在容器中获取的历史 Bean Definition
2. `isAllowBeanDefinitionOverriding()` 判断是否允许覆盖 Bean Definition

在整个流程之中上面两个变量会直接影响处理方式。首先 `existingDefinition` 的存在与否觉得了后续的操作，在本节主要描述的是存在的处理情况，下一节会对不存在进行分析。其次是 `isAllowBeanDefinitionOverriding` 方法，当 `isAllowBeanDefinitionOverriding` 方法说不允许覆盖的时候也就是下面这段代码成立，抛出异常 `BeanDefinitionOverrideException`

```java
if (!isAllowBeanDefinitionOverriding()) {
   throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
}
```

在处理 Bean Name 存在 Bean Definition 的情况下上面就是核心内容，那么剩下的还有一些小的 `else `if  比如 `role` 的处理，当前 Bean Definition 和 历史 Bean Definition 的对比，下面我们来看看里面具体的行为。

在对比 Role 和 两个 Bean Definition 的时候都是一个简单的日志输出。这里相信各位在阅读最开始的代码的时候已经了解。

在方法最后，如果是允许覆盖 Bean Definition 的，那么 Spring 就会将数据覆盖。即 `this.beanDefinitionMap.put(beanName, beanDefinition)`



在了解 Bean Name 存在 Bean Definition 的处理方式后我们来看看不存在 Bean Definition 的时候是怎么处理的。

### 7.3.3 容器中不存在 Bean Name 对应的 Bean Definition 的处理

首先我们先来看代码

- 容器中不存在 Bean Name 对应的 Bean Definition 的处理

```java
else {
   // 检查 bean 是否已经开始创建
   if (hasBeanCreationStarted()) {
      // Cannot modify startup-time collection elements anymore (for stable iteration)
      synchronized (this.beanDefinitionMap) {
         // 设置 beanName 和 beanDefinition 关系
         this.beanDefinitionMap.put(beanName, beanDefinition);
         // bean definition 的名称列表
         List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
         // 加入内存数据
         updatedDefinitions.addAll(this.beanDefinitionNames);
         // 加入当前的 beanName
         updatedDefinitions.add(beanName);
         // 对象替换
         this.beanDefinitionNames = updatedDefinitions;
         // 移除当前的beanName
         removeManualSingletonName(beanName);
      }
   }
   else {
      // Still in startup registration phase
      // 设置容器数据
      this.beanDefinitionMap.put(beanName, beanDefinition);
      this.beanDefinitionNames.add(beanName);
      // 移除当前的beanName
      removeManualSingletonName(beanName);
   }
   this.frozenBeanDefinitionNames = null;
}
```

在这段代码中首先强调第一个方法  `hasBeanCreationStarted` ，该方法来确定 Bean 是否开始创建

在 `hasBeanCreationStarted` 方法中可以看到具体使用的对象是 `alreadyCreated`。

- `hasBeanCreationStarted` 方法详情。

```java
protected boolean hasBeanCreationStarted() {
   return !this.alreadyCreated.isEmpty();
}
```



在这儿就是一个简单的判空操作。这里就存在一个问题是否有方法来对 `alreadyCreated` 进行赋值，在什么时候赋值。下面我们来逐一回答。

- 问：对 `alreadyCreated` 进行赋值的方法 ？

  答：在 Spring 中有一个叫做 `markBeanAsCreated` 的方法，这个方法就是用来标记 Bean 正在被创建。

- 问：在什么时候赋值？

  答：在 Get Bean 的时候会被赋值，在 Create Bean 的时候会被赋值。



关于 `alreadyCreated` 对象的分析结束，下面我们来看其他的代码。通过阅读源代码相信各位可以看出整个操作都是 Java 集合的操作这部分内容没有什么可以多讲的。最后有一个 `removeManualSingletonName` 方法。这个方法里面做的事情和**手动注册的单例 Bean** 有关，下面我们先来看看代码

- `removeManualSingletonName` 详细信息

```java
private void removeManualSingletonName(String beanName) {
   updateManualSingletonNames(set -> set.remove(beanName), set -> set.contains(beanName));
}
```

- `updateManualSingletonNames` 详细信息

```java
private void updateManualSingletonNames(Consumer<Set<String>> action, Predicate<Set<String>> condition) {
   if (hasBeanCreationStarted()) {
      // Cannot modify startup-time collection elements anymore (for stable iteration)
      synchronized (this.beanDefinitionMap) {
         // 输入的 beanName 是否在 manualSingletonNames 存在
         if (condition.test(this.manualSingletonNames)) {
            // 数据重写
            Set<String> updatedSingletons = new LinkedHashSet<>(this.manualSingletonNames);
            // 删除 beanName
            action.accept(updatedSingletons);
            // 数据重写
            this.manualSingletonNames = updatedSingletons;
         }
      }
   }
   else {
      // Still in startup registration phase
      // 输入的 beanName 是否在 manualSingletonNames 存在
      if (condition.test(this.manualSingletonNames)) {
         // 删除 beanName
         action.accept(this.manualSingletonNames);
      }
   }
}
```

在这段方法中我们需要关注 `action` 变量，整个 Spring IoC 中一般有下面这些可能


1. `environment`
1. `systemProperties`
1. `systemEnvironment`
1. `messageSource`
1. `applicationEventMulticaster`
1. `lifecycleProcessor`

上述的6个就是 Spring 中手工注册的 Bean Name，具体对应的 Bean 实例各位读者可以自行搜索。



在完成 `removeManualSingletonName` 方法后就剩下最后一部 Bean Definition 的刷新。下面我们一起来看看刷新操作的具体实现。




### 7.3.4 Bean Definition 的刷新处理


在开始分析 `resetBeanDefinition` 方法之前我们需要确认执行条件。在 `registerBeanDefinition` 中有这样一段代码

```java
if (existingDefinition != null || containsSingleton(beanName)) {
   // 刷新bean definition
   resetBeanDefinition(beanName);
}
```

在这段代码中可以清晰的知道在满足下面两个条件的其中一个就可以了

1. 第一个条件：通过 Bean Name 搜索 Bean Definition 搜索成功
2. 第二个条件：单例对象容器中包含当前 Bean Name 的实例对象

在了解入口条件后下面我们来看看整个 `resetBeanDefinition` 方法内部做了什么。

在整个 `resetBeanDefinition` 方法中分为下面几个步骤

1. 将合并的 Bean Definition ( `mergedBeanDefinitions` ) 中可能存在 Bean Definition 中的 `stale` 属性设置为 `true`
2. 摧毁当前 Bean Name 对应的单例实例
3. `MergedBeanDefinitionPostProcessor` 的后置方法执行
4. 处理 Bean Definition 名称列表中名称和当前 Bean Name 相同的数据

在这4个步骤中提到了关于 `mergedBeanDefinitions` 的信息那么它是从什么地方创建出来的，`mergedBeanDefinition` 又是什么呢？

1. 问题1： `mergedBeanDefinitions` 的信息那么它是从什么地方创建出来的？

   答：在 `AbstractBeanFactory` 中提供了 `getMergedBeanDefinition` 方法，该方法就是获取 `mergedBeanDefinition` 的核心。

2. 问题2：`mergedBeanDefinition` 是什么？

   答：`mergedBeanDefinition` 的本质还是 Bean Definition ，在这里主要以 `RootBeanDefinition` 类型出现，这一部分其实还没有解答 `merged` 这个行为(什么是合并？谁和谁进行了合并?) 这个问题的答案在 `AbstractBeanFactory#getMergedBeanDefinition` 中可以明确，合并的是 父 Bean Definition 对象和 当前的 Bean Definition , 细节代码如下。

```java
// 将 父 BeanDefinition 对象拷贝
mbd = new RootBeanDefinition(pbd);
// 覆盖 beanDefinition
mbd.overrideFrom(bd);
```





下面我们进入第二步，摧毁单例 Bean ，关于 Bean 的摧毁笔者会在 [第八章-Bean的生命周期](/docs/ch-08/第八章-Bean的生命周期.md)中进行讲解分析，在这里仅需要知道会从单例容器中将当前 Bean Name 从容器中剔除即可。



下面进入第三步，`MergedBeanDefinitionPostProcessor` 的处理，对于 `MergedBeanDefinitionPostProcessor` 大家可能不是很熟悉，其实 `MergedBeanDefinitionPostProcessor` 是 `BeanPostProcessor` 的子接口，也就是我们常说的 Bean 后置处理器。在 Spring 中实现了 `MergedBeanDefinitionPostProcessor` 只有 `AutowiredAnnotationBeanPostProcessor` 在其中的处理方式也很简单，容器中移除当前 Bean Name

- `AutowiredAnnotationBeanPostProcessor#resetBeanDefinition` 详细代码

```java
public void resetBeanDefinition(String beanName) {
   this.lookupMethodsChecked.remove(beanName);
   this.injectionMetadataCache.remove(beanName);
}
```



下面进入第四步，也是最后一步，在第四步中其实是一个递归处理处理的目标是在 Bean Definition 的名称列表中和当前正在处理的 Bean Name 相同，满足这个条件就会进入递归处理即重复 步骤一到四。



最后我们来看一下整个 `resetBeanDefinition` 方法的代码回忆一下每一个步骤

- `DefaultListableBeanFactory#resetBeanDefinition` 方法详情

```java
protected void resetBeanDefinition(String beanName) {
   // Remove the merged bean definition for the given bean, if already created.
   // 清空合并的BeanDefinition
   clearMergedBeanDefinition(beanName);

   // Remove corresponding bean from singleton cache, if any. Shouldn't usually
   // be necessary, rather just meant for overriding a context's default beans
   // (e.g. the default StaticMessageSource in a StaticApplicationContext).
   // 摧毁单例bean
   destroySingleton(beanName);

   // Notify all post-processors that the specified bean definition has been reset.
   // 后置处理器执行 resetBeanDefinition 方法
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
      if (processor instanceof MergedBeanDefinitionPostProcessor) {
         // 执行后置方法的 resetBeanDefinition
         ((MergedBeanDefinitionPostProcessor) processor).resetBeanDefinition(beanName);
      }
   }

   // Reset all bean definitions that have the given bean as parent (recursively).
   // 处理其他的 bean name
   for (String bdName : this.beanDefinitionNames) {
      if (!beanName.equals(bdName)) {
         BeanDefinition bd = this.beanDefinitionMap.get(bdName);
         // Ensure bd is non-null due to potential concurrent modification
         // of the beanDefinitionMap.
         // beanName 等于 父名称
         if (bd != null && beanName.equals(bd.getParentName())) {
            // 递归刷新 beanName 对应的 beanDefinition
            resetBeanDefinition(bdName);
         }
      }
   }
}
```





通过这一节的分析相信各位读者对于 Bean Definition 的注册流程有一个比较全面的认识了，下面我们需要讨论注册之后的事项：**使用**，注册完成之后我们应该如何使用。



## 7.4 Bean Definition 的获取

首先回顾一下存储 Bean Definition 的容器

- Bean Definition 存储容器

```java
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```



一般情况下我们可以明确 Bean Name 和 Bean Definition 的关系，在这一点的基础上我们有了上面提到的存储容器，那么在获取的时候也是同样的，根据名称获取是一个最为高效的处理方式。下面我们来看看 Spring 中提供的获取方式



- `DefaultListableBeanFactory#getBeanDefinition` 方法详情

```java
@Override
public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
   BeanDefinition bd = this.beanDefinitionMap.get(beanName);
   if (bd == null) {
      if (logger.isTraceEnabled()) {
         logger.trace("No bean named '" + beanName + "' found in " + this);
      }
      throw new NoSuchBeanDefinitionException(beanName);
   }
   return bd;
}
```

从这段代码中我们可以很明确的看到它就是通过 Bean Name 在容器中直接获取 Bean Definition ，如果容器中不存在会抛出 `NoSuchBeanDefinitionException` 异常。





## 7.5 总结

在这一章节中笔者主要围绕 Bean Definition 的注册和使用进行分析，在这一节中关键的核心是存储 Bean Definition 的容器，在整个存储过程中还有一些其他对象的使用，对于使用笔者在这里只是提到关于 Bean Name 如何变成 Bean Definition 的过程，可以知道 Bean Definition 肯定不是最终的 Bean 实例 ，那么如何才可以获取到真正的实例呢？这部分内容笔者将在第八章给各位做一个解析。


