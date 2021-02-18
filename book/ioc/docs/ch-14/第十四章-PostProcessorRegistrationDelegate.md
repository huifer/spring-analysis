# 第十四章 PostProcessorRegistrationDelegate
在这一章节中笔者将和各位读者一起讨论 Spring 中关于操作 `BeanPostProcessor` 的委托对象 `PostProcessorRegistrationDelegate`


## 14.1 `BeanPostProcessor` 注册
在开始分析 `PostProcessorRegistrationDelegate` 的两个主要方法之前我们需要回忆一下 Spring 在什么时候使用了。在 `AbstractApplicationContext#refresh` 中有相关调用，

- 注册 `BeanPostProcessor` ：`org.springframework.context.support.AbstractApplicationContext#registerBeanPostProcessors`
- 调用 `BeanFactoryPostProcessor`：`org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors`

好下面我们开始阅读注册方法的具体实现，方法签名：`org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, org.springframework.context.support.AbstractApplicationContext)`

在注册方法中 Spring 将`BeanPostProcessor` 分为下面四种情况

1. 第一种：`priorityOrderedPostProcessors`，同时实现了 `BeanPostProcessor` 接口和 `PriorityOrdered` 接口的 `BeanPostProcessor` 实例。
2. 第二种：`internalPostProcessors`，同时实现了 `BeanPostProcessor` 接口、`MergedBeanDefinitionPostProcessor` 接口和 `PriorityOrdered` 接口的 `BeanPostProcessor` 实例。
3. 第三种：`orderedPostProcessorNames`，同时实现了 `BeanPostProcessor` 接口和 `Ordered` 接口的 `BeanPostProcessor` 实例的 Bean Name。
4. 第四种：`nonOrderedPostProcessorNames`，只实现了 `BeanPostProcessor` 接口的  `BeanPostProcessor` 实例的 Bean Name。



四种 `BeanPostProcessors` 注册顺序如下

1. 第一：`priorityOrderedPostProcessors`
2. 第二：`orderedPostProcessors`
3. 第三：`nonOrderedPostProcessors`
4. 第四：`internalPostProcessors`
5. 第五：独立的 `BeanPostProcessors` 注册，类型是 `ApplicationListenerDetector`



在注册之前除了 `nonOrderedPostProcessors` 以外都会进行排序操作，这个排序操作是由 `PriorityOrdered` 和 `Ordered` 提供，在这两个接口种有提供 `getOrder` 方法，排序就是依靠这个字段的数据内容进行排序。

- 排序 `BeanPostProcessors` 方法

```java
private static void sortPostProcessors(List<?> postProcessors, ConfigurableListableBeanFactory beanFactory) {
   Comparator<Object> comparatorToUse = null;
   if (beanFactory instanceof DefaultListableBeanFactory) {
      comparatorToUse = ((DefaultListableBeanFactory) beanFactory).getDependencyComparator();
   }
   if (comparatorToUse == null) {
      comparatorToUse = OrderComparator.INSTANCE;
   }
   postProcessors.sort(comparatorToUse);
}
```

排序完成后就进入注册阶段注册是一个统一方法，具体代码如下

```java
private static void registerBeanPostProcessors(
      ConfigurableListableBeanFactory beanFactory, List<BeanPostProcessor> postProcessors) {

   for (BeanPostProcessor postProcessor : postProcessors) {
      beanFactory.addBeanPostProcessor(postProcessor);
   }
}
```

注册方法中会将所有的 `BeanPostProcessor` 注册到 `AbstractBeanFactory#beanPostProcessors` 中







## 14.2 `BeanFactoryPostProcessor` 方法调用

下面我们来看 `BeanPostProcessor` 的调度方法，在执行 `BeanPostProcessor` 的方法时会分别执行下面这三个不同的 `BeanDefinitionRegistryPostProcessor`

1. 同时实现 `BeanDefinitionRegistryPostProcessor` 和  `PriorityOrdered`
2. 同时实现 `BeanDefinitionRegistryPostProcessor` 和 `Ordered`
3. 只实现 `BeanDefinitionRegistryPostProcessor`

在找到上述三种会的执行顺序还是会先排序在调用，排序方法和前文提到的排序方式是一致的，但是在执行时会有差异，根据顺序执行`BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry`（顺序定义：`PriorityOrdered` > `Ordered` > 未实现） ，再执行 `BeanFactoryPostProcessor#postProcessBeanFactory` 方法。

在这个方法中还有单纯的 `BeanFactoryPostProcessor` 实现，前面笔者提到的 `BeanDefinitionRegistryPostProcessor` 是 `BeanFactoryPostProcessor` 的子接口。关于 `BeanFactoryPostProcessor` 的分类如下



1. 同时实现 `BeanFactoryPostProcessor` 和  `PriorityOrdered`
2. 同时实现 `BeanFactoryPostProcessor` 和 `Ordered`
3. 只实现 `BeanFactoryPostProcessor`

这三个 `BeanFactoryPostProcessor` 还是会先排序再执行



下面我们来看具体的代码

- `invokeBeanFactoryPostProcessors` 方法详情

```java
public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
		// 需要处理的 BeanDefinitionRegistryPostProcessors 名称
		Set<String> processedBeans = new HashSet<>();

		// 关于 BeanFactoryPostProcessor 的处理
		// beanFactory 类型是 BeanDefinitionRegistry
		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			// BeanFactoryPostProcessor 接口列表
			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
			// BeanDefinitionRegistryPostProcessor 接口列表
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

			// 数据分类
			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
					// 将 beanDefinition 进行注册 Configuration 注解标注的对象.
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					registryProcessors.add(registryProcessor);
				}
				else {
					regularPostProcessors.add(postProcessor);
				}
			}

			// Do not initialize FactoryBeans here: We need to leave all regular beans
			// uninitialized to let the bean factory post-processors apply to them!
			// Separate between BeanDefinitionRegistryPostProcessors that implement
			// PriorityOrdered, Ordered, and the rest.
			// bean定义后置处理器
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

			// First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
			// 处理 BeanDefinitionRegistryPostProcessor + PriorityOrdered
			// 获取 BeanDefinitionRegistryPostProcessor 的 beanName
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			// 处理 BeanDefinitionRegistryPostProcessor beanName
			for (String ppName : postProcessorNames) {
				// 类型是 PriorityOrdered
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
					// 添加到容器 currentRegistryProcessors
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}

			// 排序 BeanDefinitionRegistryPostProcessor 对象
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			// 执行 BeanDefinitionRegistryPostProcessor的方法
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			// 清理数据
			currentRegistryProcessors.clear();

			// 处理 BeanDefinitionRegistryPostProcessor + Ordered
			// Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);

			for (String ppName : postProcessorNames) {
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			// 处理剩下的 BeanDefinitionRegistryPostProcessor
			// Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
				currentRegistryProcessors.clear();
			}

			// Now, invoke the postProcessBeanFactory callback of all processors handled so far.
			// BeanDefinitionRegistryPostProcessor 处理
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			// BeanFactoryPostProcessor 处理
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		// beanFactory 其他类型的处理
		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}


		// 处理 BeanFactoryPostProcessor 类型的bean
		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		// 分成两组进行处理
		// 1. BeanFactoryPostProcessor + PriorityOrdered -> priorityOrderedPostProcessors
		// 2. BeanFactoryPostProcessor + Ordered -> orderedPostProcessorNames
		// 3. BeanFactoryPostProcessor -> nonOrderedPostProcessorNames
		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		// 根据不同实现类进行分组
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// 处理 BeanFactoryPostProcessors + PriorityOrdered 的类型
		// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		// 处理 BeanFactoryPostProcessors + Ordered 的类型
		// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		// 处理 普通的 BeanFactoryPostProcessors 类型
		// Finally, invoke all other BeanFactoryPostProcessors.
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
	}
```







## 14.3 总结

在这一章节中笔者和各位共同探讨了关于 `BeanFactoryPostProcessor` 注册和调用两个方法，在这两个方法中我们间接了解了 Spring 中关于 Bean 排序的操作。整体上看这里的调用和注册相对前几章提到的一些内容都简单了很多。