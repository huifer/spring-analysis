# Spring ClassPathScanningCandidateComponentProvider
- 类全路径: `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider`







## 方法分析

### scanCandidateComponents

- 方法签名: `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#scanCandidateComponents`

处理流程

1. 将 参数包路径转换

   转换规则: `classpath*: + replace(basePackage,'.','/') + / + **/*.class`

2. 通过资源匹配器进行资源获取. 即获取 👆 上面路线下的所有class资源

3. 循环资源列表

   单个资源的处理:

   1. 是否是候选组件`isCandidateComponent`
   2. 创建`ScannedGenericBeanDefinition`设置资源对象，源对象，判断是否是组件`isCandidateComponent`,加入容器





```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
   // 候选组件列表 BeanDefinition 列表
   Set<BeanDefinition> candidates = new LinkedHashSet<>();
   try {
      // classpath*: + replace(basePackage,'.','/') + / + **/*.class
      String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
      // 转换成资源对象
      // 这里会转换成 FileSystemResource
      Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
      // 日志级别
      boolean traceEnabled = logger.isTraceEnabled();
      boolean debugEnabled = logger.isDebugEnabled();
      // 资源处理
      for (Resource resource : resources) {
         if (traceEnabled) {
            logger.trace("Scanning " + resource);
         }
         if (resource.isReadable()) {
            try {
               // 元数据读取器
               MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
               if (isCandidateComponent(metadataReader)) {
                  // bean定义扫描
                  ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                  // 设置资源对象
                  sbd.setResource(resource);
                  // 设置源对象
                  sbd.setSource(resource);
                  // 判断是否是候选值
                  if (isCandidateComponent(sbd)) {
                     if (debugEnabled) {
                        logger.debug("Identified candidate component class: " + resource);
                     }
                     // 加入容器
                     candidates.add(sbd);
                  }
                  else {
                     if (debugEnabled) {
                        logger.debug("Ignored because not a concrete top-level class: " + resource);
                     }
                  }
               }
               else {
                  if (traceEnabled) {
                     logger.trace("Ignored because not matching any filter: " + resource);
                  }
               }
            }
            catch (Throwable ex) {
               throw new BeanDefinitionStoreException(
                     "Failed to read candidate component class: " + resource, ex);
            }
         }
         else {
            if (traceEnabled) {
               logger.trace("Ignored because not readable: " + resource);
            }
         }
      }
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
   }
   return candidates;
}
```







### isCandidateComponent

- 方法签名: `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#isCandidateComponent(org.springframework.core.type.classreading.MetadataReader)`