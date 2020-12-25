# 加载顺序问题

- `PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors`方法的加载顺序的疑惑
- 相关 pr : https://github.com/spring-projects/spring-framework/pull/26273



> [info] spring 中`invokeBeanFactoryPostProcessors` 部分代码

```java
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
```



- 在上述代码中笔者认为`orderedPostProcessors`和`nonOrderedPostProcessors` 两个列表的容器可以提前加载 在下面代码中可以进行获取提前准备好数据. 

  ```java
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
  ```



- 对其修改成

  ```java
  List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>();
  List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>();
  
  
  for (String ppName : postProcessorNames) {
  	if (processedBeans.contains(ppName)) {
      	// skip - already processed in first phase above
  	else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
  		priorityOrderedPostProcessors.add(beanFactory.getBean(ppName,BeanFactoryPostProcessor.class));
       }
       else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
       	orderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
  	}
  	else {
  		nonOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
  	}
  }
      
      
      sortPostProcessors(orderedPostProcessors, beanFactory);
      invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);
      
      invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);
  ```



- Spring 中的人员回复说: 

  > Thanks for PR.
  >
  > Unfortunately, the proposal changes the order in which the `orderedPostProcessors` and `nonOrderedPostProcessors` are instantiated which constitutes a breaking change.
  >
  > In light of that, I am closing this PR.
  >
  > from:  https://github.com/spring-projects/spring-framework/pull/26273#issuecomment-750867140

  - 大致意思是我的改动修改了 `orderedPostProcessors` 和 `nonOrderedPostProcessors` 的加载顺序. 



对于这里的加载顺序笔者不是很理解，提前获取 bean 和 先获取 beanName 在进行获取的差异是什么. 

目前猜测: 加载顺序关于 `Ordered ` 在 `getBean` 方法中可能存在使用, 但是笔者翻阅代码后没有找到(可能有所遗漏的代码) . 





- 希望读者可以围绕这个问题进行讨论. 谢谢 