# Spring SimpleBeanDefinitionRegistry



- 全路径: `org.springframework.beans.factory.support.SimpleBeanDefinitionRegistry`



- 一个简单的map操作代码如下.

```java
public class SimpleBeanDefinitionRegistry extends SimpleAliasRegistry implements BeanDefinitionRegistry {

   /**
    *  Map of bean definition objects, keyed by bean name.
    * key:beanName
    * value: bean definition
    * */
   private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(64);


   @Override
   public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
         throws BeanDefinitionStoreException {

      Assert.hasText(beanName, "'beanName' must not be empty");
      Assert.notNull(beanDefinition, "BeanDefinition must not be null");
      this.beanDefinitionMap.put(beanName, beanDefinition);
   }

   @Override
   public void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
      if (this.beanDefinitionMap.remove(beanName) == null) {
         throw new NoSuchBeanDefinitionException(beanName);
      }
   }

   @Override
   public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
      BeanDefinition bd = this.beanDefinitionMap.get(beanName);
      if (bd == null) {
         throw new NoSuchBeanDefinitionException(beanName);
      }
      return bd;
   }

   @Override
   public boolean containsBeanDefinition(String beanName) {
      return this.beanDefinitionMap.containsKey(beanName);
   }

   @Override
   public String[] getBeanDefinitionNames() {
      return StringUtils.toStringArray(this.beanDefinitionMap.keySet());
   }

   @Override
   public int getBeanDefinitionCount() {
      return this.beanDefinitionMap.size();
   }

   @Override
   public boolean isBeanNameInUse(String beanName) {
      return isAlias(beanName) || containsBeanDefinition(beanName);
   }

}
```



