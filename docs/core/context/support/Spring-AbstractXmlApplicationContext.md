# Spring AbstractXmlApplicationContext

- 类全路径: `org.springframework.context.support.AbstractXmlApplicationContext`
- xml应用上下文





- 关联类
  - [XmlBeanDefinitionReader](/doc/book/bean/factory/xml/Spring-XmlBeanDefinitionReader.md): 读取 xml 中的 bean 定义的核心类



## 方法分析
### loadBeanDefinitions

- 方法签名: `org.springframework.context.support.AbstractXmlApplicationContext#loadBeanDefinitions(org.springframework.beans.factory.support.DefaultListableBeanFactory)`



- 方法作用: 创建`XmlBeanDefinitionReader` 读取beanDefintion

方法流程

1. 创建 `XmlBeanDefinitionReader`
2. 设置环境属性
3. 初始化 XmlBeanDefinitionReader
4. 进行资源加载



<details>
    <summary>loadBeanDefinitions 方法详情</summary>



```java
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
   // 创建 读取 xml bean定义的类
   // Create a new XmlBeanDefinitionReader for the given BeanFactory.
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

   // Configure the bean definition reader with this context's
   // resource loading environment.
   // 设置环境
   beanDefinitionReader.setEnvironment(this.getEnvironment());
   // 设置资源加载器
   beanDefinitionReader.setResourceLoader(this);
   // 资源解析器
   beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

   // Allow a subclass to provide custom initialization of the reader,
   // then proceed with actually loading the bean definitions.
   // 初始化 bean定义阅读器
   initBeanDefinitionReader(beanDefinitionReader);
   // 加载 bean 定义
   loadBeanDefinitions(beanDefinitionReader);
}
```

</details>