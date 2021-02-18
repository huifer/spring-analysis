# 第十六章 LifecycleProcessor
本章节笔者将和各位读者一起讨论 Spring 启动中关于容器生命周期的处理和源代码分析


## 16.1 初识 LifecycleProcessor
在开始源码分析之前我们先来看一看 LifecycleProcessor 接口在那个阶段使用，不知道各位是否还记得我们在第十章中对于 `finishRefresh` 方法的分析，我们先来回顾一下 `finishRefresh` 这个方法

- `AbstractApplicationContext#finishRefresh` 方法详情

```java
protected void finishRefresh() {
    // 清空资源缓存
    // Clear context-level resource caches (such as ASM metadata from scanning).
    clearResourceCaches();

    // 实例化生命周期处理接口
    // Initialize lifecycle processor for this context.
    initLifecycleProcessor();

    // 生命周期处理接口进行刷新操作
    // Propagate refresh to lifecycle processor first.
    getLifecycleProcessor().onRefresh();

    // 推送事件: 上下文刷新事件
    // Publish the final event.
    publishEvent(new ContextRefreshedEvent(this));

    // 注册应用上下文
    // Participate in LiveBeansView MBean, if active.
    LiveBeansView.registerApplicationContext(this);
}
```

在这段方法中我们可以看到 `getLifecycleProcessor().onRefresh()` 这个方法就是我们需要去分析的目标方法，在 Spring 中对于  `LifecycleProcessor` 有三处地方：

1. 第一处：在 `finishRefresh` 方法中执行 `onRefresh` 事件。
2. 第二处：在 `start` 方法中执行 `start` 事件。
3. 第三处：在 `stop` 方法中执行 `stop` 事件。

这三处地方就是我们下面需要进行分析的内容，在开始分析之前我们来看看 `LifecycleProcessor` 的类图

![LifecycleProcessor](images/DefaultLifecycleProcessor.png)

从这个类图上我们可以确定我们接下来的分析目标是  `DefaultLifecycleProcessor` 类。





## 16.2 测试环境搭建

在进入方法分析之前我们还需要来编写测试用例。在这我们主要从 `Lifecycle` 接口出发，来编写一个实现类。

- `HelloLifeCycle`

```java
public class HelloLifeCycle implements Lifecycle {
    private volatile boolean running = false;
   
    @Override
    public void start() {
        System.out.println("lifycycle start");
        running = true;

    }
   @Override
    public void stop() {
        System.out.println("lifycycle stop");
        running = false;
    }

    @Override
    public boolean isRunning() {
        return running;
    }
}
```

在编写 `Lifecycle` 实现类之后我们需要来编写一个 Spring xml 文件将 `HelloLifecycle` 注入到容器中

- `spring-lifecycle.xml`

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns="http://www.springframework.org/schema/beans"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

   <bean class="com.source.hot.ioc.book.lifecycle.HelloLifeCycle"/>
</beans>
```

完成 Spring XML 编写后我们将这个文件配合 Spring 进行使用

- 测试用例

```java
public class LifeCycleTest {
   @Test
   void testLifeCycle() {
      ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring-lifecycle.xml");
      context.start();
      context.stop();
      context.close();
   }
}
```

执行该测试用例我们来看输出结果

```tex
lifycycle start
lifycycle stop
```



用例准备完毕下面我们来看 `start` 中的处理

## 16.3 `start` 分析

我们所分析的对象是 `DefaultLifecycleProcessor#start` 下面我们先来看代码

```java
public void start() {
   startBeans(false);
   this.running = true;
}
```

在这段代码中我们重点关注的是 `startBeans` 方法，此时我们的分析目标是 `startBeans`。下面我们来看具体实现

- `startBeans` 方法详情

```java
private void startBeans(boolean autoStartupOnly) {
   // 获取 生命周期bean 容器
   Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
   // 阶段容器
   Map<Integer, LifecycleGroup> phases = new HashMap<>();
   // 循环 生命周期bean 创建阶段对象
   lifecycleBeans.forEach((beanName, bean) -> {

      // 1. autoStartupOnly 自动启动
      // 2. 类型是否是 SmartLifecycle
      // 3. SmartLifecycle 方法isAutoStartup是否 true
      if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
         // 获取bean的生命周期阶段
         int phase = getPhase(bean);
         // 通过阶段值获取生命周期组
         LifecycleGroup group = phases.get(phase);
         if (group == null) {
            // 创建 生命周期组
            group = new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly);
            phases.put(phase, group);
         }
         group.add(beanName, bean);
      }
   });
   if (!phases.isEmpty()) {
      // 获取阶段数据
      List<Integer> keys = new ArrayList<>(phases.keySet());
      // 阶段排序
      Collections.sort(keys);
      // 顺序执行 生命周期组的start方法
      for (Integer key : keys) {
         phases.get(key).start();
      }
   }
}
```

在 `startBeans` 方法中主要围绕下面这三个事项进行

1. 第一项：搜索容器中的 `Lifecycle` 类型对象，处理方式是通过 `getBeanNamesForType` 进行获取。
2. 第二项：创建阶段容器
3. 第三项：根据阶段容器是否存在进一步执行 `Lifecycle` 接口的 `start` 方法



在这三项中我们需要理解阶段的含义，阶段在这里笔者将其作为一个注释表示，其本身在 Spring 中是一个接口的形式出现，这个接口是 `Phased`，在代码中我们看到 `(bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())` 这样一段代码，在这段代码中的 `SmartLifecycle` 接口也是集成 `Phased` 接口的，但是普通的 `Lifecycle` 是不存在 `Phased` 接口的继承关系的，因此在这里我们获取阶段数值的方式会有两种

1. 第一种：默认取0

2. 第二种：从 `SmartLifecycle` 接口中获取

   这两种方式就对应 `getPhase` 方法

现在我们打上断点来看**阶段容器**中的数据

- `phases` 数据信息

![image-20210202094516515](images/image-20210202094516515.png)



这便是第二项所准备的全部内容，接下来我们来看第三项的执行，执行简单一些说就是调用方法，现在我们获取了 `Lifecycle` 实例想要执行就很简单，直接 `bean.start` 即可执行。我们来看代码中的具体操作。

- `doStart` 方法详情

```java
private void doStart(Map<String, ? extends Lifecycle> lifecycleBeans, String beanName, boolean autoStartupOnly) {
	// 从容器中删除当前处理的beanName
	// 生命周期接口
	Lifecycle bean = lifecycleBeans.remove(beanName);

	if (bean != null && bean != this) {

		// beanName 依赖的beanName列表
		String[] dependenciesForBean = getBeanFactory().getDependenciesForBean(beanName);

		// 循环处理 依赖的bean生命周期
		for (String dependency : dependenciesForBean) {
			doStart(lifecycleBeans, dependency, autoStartupOnly);
		}
		if (!bean.isRunning() &&
				(!autoStartupOnly || !(bean instanceof SmartLifecycle) || ((SmartLifecycle) bean).isAutoStartup())) {
		
			try {
				// 执行生命周期的start方法
				bean.start();
			}
			catch (Throwable ex) {
				throw new ApplicationContextException("Failed to start bean '" + beanName + "'", ex);
			}
			
		}
	}
}
```

在这段代码中我们可以看到存在递归操作，这里是为了处理依赖项上的 `Lifecycle#start` 方法，这里也同样涉及优先级，**提供依赖的对象优先级永远高于本身**，最后我们可以看到正真做执行的代码就是 `bean.start()` 。



现在我们了解了关于 `Lifecycle` 接口中关于 `start` 的相关处理，其他的几个方法处理方式都大同小异，下面我们来看 `stop` 中的一些处理





## 16.4 `stop` 分析

下面我们来看 `DefaultLifecycleProcessor#stop` 中的处理

- `DefaultLifecycleProcessor#stop` 方法详情

```java
public void stop() {
   stopBeans();
   this.running = false;
}

private void stopBeans() {
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new HashMap<>();
    lifecycleBeans.forEach((beanName, bean) -> {
        int shutdownPhase = getPhase(bean);
        LifecycleGroup group = phases.get(shutdownPhase);
        if (group == null) {
            group = new LifecycleGroup(shutdownPhase, this.timeoutPerShutdownPhase, lifecycleBeans, false);
            phases.put(shutdownPhase, group);
        }
        group.add(beanName, bean);
    });
    if (!phases.isEmpty()) {
        List<Integer> keys = new ArrayList<>(phases.keySet());
        keys.sort(Collections.reverseOrder());
        for (Integer key : keys) {
            phases.get(key).stop();
        }
    }
}
```

可以看到这一部分的操作和 `start` 中的操作基本相同，根据我们分析 `start` 方法是的方法我们可以直接定位到 `stop` 方法，我们先将代码找出来



```java
public void stop() {
   if (this.members.isEmpty()) {
      return;
   }
   if (logger.isDebugEnabled()) {
      logger.debug("Stopping beans in phase " + this.phase);
   }
   this.members.sort(Collections.reverseOrder());
   CountDownLatch latch = new CountDownLatch(this.smartMemberCount);
   Set<String> countDownBeanNames = Collections.synchronizedSet(new LinkedHashSet<>());
   Set<String> lifecycleBeanNames = new HashSet<>(this.lifecycleBeans.keySet());
   for (LifecycleGroupMember member : this.members) {
      if (lifecycleBeanNames.contains(member.name)) {
         doStop(this.lifecycleBeans, member.name, latch, countDownBeanNames);
      }
      else if (member.bean instanceof SmartLifecycle) {
         // Already removed: must have been a dependent bean from another phase
         latch.countDown();
      }
   }
   try {
      latch.await(this.timeout, TimeUnit.MILLISECONDS);
      if (latch.getCount() > 0 && !countDownBeanNames.isEmpty() && logger.isInfoEnabled()) {
         logger.info("Failed to shut down " + countDownBeanNames.size() + " bean" +
               (countDownBeanNames.size() > 1 ? "s" : "") + " with phase value " +
               this.phase + " within timeout of " + this.timeout + ": " + countDownBeanNames);
      }
   }
   catch (InterruptedException ex) {
      Thread.currentThread().interrupt();
   }
}
```

这段代码我们以一个简单的视角来进行理解，执行成员变量 `members` 中的所有 `Lifecycle` 都进行 `stop` 方法调用。



## 16.5 关键类

在处理 `Lifecycle` 过程中 Spring 对其进行了分组抽象，根据 `Phase` 接口的返回值进行分组。下面我们来看分组对象 `LifecycleGroup` 中出现的成员变量的含义



### 16.5.1 `LifecycleGroup` 成员变量

| 变量名称           | 变量类型                           | 说明                                                         |
| ------------------ | ---------------------------------- | ------------------------------------------------------------ |
| `phase`            | `int`                              | 从 `Phase` 接口中获取的数据                                  |
| `timeout`          | `long`                             | 超时时间                                                     |
| `lifecycleBeans`   | `Map<String, ? extends Lifecycle>` | 实现了 `Lifecycle` 的数据<br />key:Bean Name<br />value: `Lifecycle` 实例 |
| `autoStartupOnly`  | `boolean`                          | 是否自动启动                                                 |
| `members`          | `List<LifecycleGroupMember>`       | 存储 `Lifecycle` 的容器                                      |
| `smartMemberCount` | `int`                              | SmartLifecycle 实例的数量                                    |



- `LifecycleGroupMember` 成员变量
  - `name`： Bean Name
  - `bean`： Lifecycle 实例



通过 `start` 和 `stop` 两个方法的分析我们可以知道最终的处理方法就是在 `LifecycleGroup` 中。





## 16.6 总结

本章笔者和各位围绕 `LifecycleProcessor` 出发我们共同了解了 Spring 中对于 生命周期接口 (`Lifecycle`)及其处理的相关源码阅读及分析。