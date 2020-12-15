# Spring DependencyObjectProvider
- 类全路径: `org.springframework.beans.factory.support.DefaultListableBeanFactory.DependencyObjectProvider`


## 成员变量
- 先来阅读 `DependencyObjectProvider` 中的成员变量的相关信息


<details>
<summary>成员变量</summary>


```java
	private class DependencyObjectProvider implements BeanObjectProvider<Object> {

    /**
     * 依赖描述
     */
    private final DependencyDescriptor descriptor;

    /**
     * 是否是 Optional 类型
     */
    private final boolean optional;

    @Nullable
    private final String beanName;
}
```

</details>

在成员变量中我们对 `optional` 变量多一点描述. 
  在构造函数中有下面这段代码
    `this.descriptor.getDependencyType() == Optional.class` 这里是做类型判断. 





## 方法分析
- 在对成员变量了解后,下面开始对方法进行逐个分析. 


### getObject
- 方法签名: `org.springframework.beans.factory.support.DefaultListableBeanFactory.DependencyObjectProvider.getObject()`

<details>
<summary>getObject 完整代码如下</summary>


```java
		@Override
		public Object getObject() throws BeansException {
			// 获取bean实例
			// 是 optional 的情况下创建
			if (this.optional) {
				return createOptionalDependency(this.descriptor, this.beanName);
			}
			else {
				// 解析依赖进行创建对象
				Object result = doResolveDependency(this.descriptor, this.beanName, null, null);
				if (result == null) {
					throw new NoSuchBeanDefinitionException(this.descriptor.getResolvableType());
				}
				return result;
			}
		}

```


</details>


在 getObject 中有两者创建(获取)bean实例的方式.
    1. 根据依赖类型是否是 Optional 做两种区分
        1. true: `createOptionalDependency`
        2. false: `doResolveDependency`
        
关于上述两个方法的分析请查看: [DefaultListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-DefaultListableBeanFactory.md)
    






### getObject
- 方法签名: `org.springframework.beans.factory.support.DefaultListableBeanFactory.DependencyObjectProvider.getObject(java.lang.Object...)`






<details>
<summary>getObject 完整代码如下</summary>


```java
    @Override
    public Object getObject(final Object... args) throws BeansException {
        // 获取bean实例
        // 是 optional 的情况下创建
        if (this.optional) {
            return createOptionalDependency(this.descriptor, this.beanName, args);
        }
        else {
            // 创建 依赖描述对象， 重写方法 resolveCandidate , 从容器(BeanFactory)中获取
            DependencyDescriptor descriptorToUse = new DependencyDescriptor(this.descriptor) {
                @Override
                public Object resolveCandidate(String beanName, Class<?> requiredType, BeanFactory beanFactory) {
                    return beanFactory.getBean(beanName, args);
                }
            };

            // 解析依赖进行创建对象
            Object result = doResolveDependency(descriptorToUse, this.beanName, null, null);
            if (result == null) {
                throw new NoSuchBeanDefinitionException(this.descriptor.getResolvableType());
            }
            return result;
        }
    }

```


</details>



- 把这个方法和上一个方法对比, 可以发现两者的差异点是 `DependencyDescriptor` 类的`resolveCandidate` 被重写了, 



- 剩下的 `getIfAvailable`、`getIfUnique` 、 `getValue` 的处理方式也和上述方式相同, 重写了`DependencyDescriptor` 中的个别方法. 下面贴出代码

<details>
<summary>getIfAvailable、getIfUnique 、 getValue 详细代码</summary>


```java
@Override
		@Nullable
		public Object getIfAvailable() throws BeansException {
			if (this.optional) {
				return createOptionalDependency(this.descriptor, this.beanName);
			}
			else {
				DependencyDescriptor descriptorToUse = new DependencyDescriptor(this.descriptor) {
					@Override
					public boolean isRequired() {
						return false;
					}
				};
				return doResolveDependency(descriptorToUse, this.beanName, null, null);
			}
		}

		@Override
		@Nullable
		public Object getIfUnique() throws BeansException {
			DependencyDescriptor descriptorToUse = new DependencyDescriptor(this.descriptor) {
				@Override
				public boolean isRequired() {
					return false;
				}

				@Override
				@Nullable
				public Object resolveNotUnique(ResolvableType type, Map<String, Object> matchingBeans) {
					return null;
				}
			};
			if (this.optional) {
				return createOptionalDependency(descriptorToUse, this.beanName);
			}
			else {
				return doResolveDependency(descriptorToUse, this.beanName, null, null);
			}
		}

		@Nullable
		protected Object getValue() throws BeansException {
			if (this.optional) {
				return createOptionalDependency(this.descriptor, this.beanName);
			}
			else {
				return doResolveDependency(this.descriptor, this.beanName, null, null);
			}
		}

```
  
</details>



下面来阅读 `stream` 和 `orderStream` 两个方法


```java
		@Override
		public Stream<Object> stream() {
			return resolveStream(false);
		}

		@Override
		public Stream<Object> orderedStream() {
			return resolveStream(true);
		}
```

- 上述两个方法最终还是依靠下面的方法`resolveStream`

```java
		@SuppressWarnings("unchecked")
		private Stream<Object> resolveStream(boolean ordered) {
			DependencyDescriptor descriptorToUse = new StreamDependencyDescriptor(this.descriptor, ordered);
			Object result = doResolveDependency(descriptorToUse, this.beanName, null, null);
			return (result instanceof Stream ? (Stream<Object>) result : Stream.of(result));
		}

```

除开 `doResolveDependency` 方法外仅有类型判断和 `stream.of` 两个作为返回值的控制