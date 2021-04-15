# FlashMapManager
本章将对 FlashMapManager 接口进行分析, FlashMapManager作用是在redirect中传递参数，默认SessionFlashMapManager通过session实现传递。在FlashMapManager接口中定义了两个方法，具体代码如下：

```java
public interface FlashMapManager {


   @Nullable
   FlashMap retrieveAndUpdate(HttpServletRequest request, HttpServletResponse response);

  
   void saveOutputFlashMap(FlashMap flashMap, HttpServletRequest request, HttpServletResponse response);

}
```

下面对方法进行说明：

1. 方法retrieveAndUpdate作用是寻找FlashMap对象。
2. 方法saveOutputFlashMap作用是将给定的FlashMap对象进行保存。





## FlashMapManager 测试环境搭建

本节将搭建一个用于FlashMapManager 源码分析和调试的测试环境，首先需要在applicationContext.xml文件中添加一个FlashMapManager 的实现类，具体代码如下：

```xml
<bean id="flashMapManager" class="org.springframework.web.servlet.support.SessionFlashMapManager"/>
```

在完成该配置后测试环境就搭建完成，下面可以请求项目中的任意一个接口，如果没有可以定义一个接口，具体定义如下：

```java
@Controller
@CrossOrigin
public class HelloController {

   @GetMapping("/demo")
   public String demo(HttpServletRequest req) {
       FlashMap flashMap = (FlashMap) req.getAttribute(DispatcherServlet.OUTPUT_FLASH_MAP_ATTRIBUTE);
       flashMap.put("name", "name");
      return "hello";
   }
}
```





## FlashMapManager 初始化

本节将对FlashMapManager对象的初始化相关内容进行分析，具体处理代码如下：

```java
private void initFlashMapManager(ApplicationContext context) {
   try {
      this.flashMapManager = context.getBean(FLASH_MAP_MANAGER_BEAN_NAME, FlashMapManager.class);
      if (logger.isTraceEnabled()) {
         logger.trace("Detected " + this.flashMapManager.getClass().getSimpleName());
      }
      else if (logger.isDebugEnabled()) {
         logger.debug("Detected " + this.flashMapManager);
      }
   }
   catch (NoSuchBeanDefinitionException ex) {
      // We need to use the default.
      this.flashMapManager = getDefaultStrategy(context, FlashMapManager.class);
      if (logger.isTraceEnabled()) {
         logger.trace("No FlashMapManager '" + FLASH_MAP_MANAGER_BEAN_NAME +
               "': using default [" + this.flashMapManager.getClass().getSimpleName() + "]");
      }
   }
}
```

在这段代码中只提供了一种方式获取FlashMapManager 对象，具体方式是通过名称+类型进行获取，这个获取方式对应了前文对于测试环境搭建中的配置信息：

```
<bean id="flashMapManager" class="org.springframework.web.servlet.support.SessionFlashMapManager"/>
```

通过调试初始化方法可以看到FlashMapManager的数据信息如下：

![image-20210415093606915](images/image-20210415093606915.png)





## SessionFlashMapManager 分析

本节将对SessionFlashMapManager类进行分析，主要关注三个方法retrieveFlashMaps、updateFlashMaps和getFlashMapsMutex。下面先对retrieveFlashMaps方法进行分析，该方法的完整代码如下：

```java
@Override
@SuppressWarnings("unchecked")
@Nullable
protected List<FlashMap> retrieveFlashMaps(HttpServletRequest request) {
   HttpSession session = request.getSession(false);
   return (session != null ? (List<FlashMap>) session.getAttribute(FLASH_MAPS_SESSION_ATTRIBUTE) : null);
}
```

在这段代码中可以明确提取FlashMap对象的方式是通过Session中的FLASH_MAPS_SESSION_ATTRIBUTE属性值进行提取。其次对updateFlashMaps方法进行分析，该方法的完整代码如下：

```java
@Override
protected void updateFlashMaps(List<FlashMap> flashMaps, HttpServletRequest request, HttpServletResponse response) {
   WebUtils.setSessionAttribute(request, FLASH_MAPS_SESSION_ATTRIBUTE, (!flashMaps.isEmpty() ? flashMaps : null));
}
```

在这段代码中可以明确它的保存操作，具体是将flashMaps对象放入到Session中的FLASH_MAPS_SESSION_ATTRIBUTE属性中。最后对getFlashMapsMutex方法进行分析，该方法的完整代码如下：

```java
@Override
protected Object getFlashMapsMutex(HttpServletRequest request) {
   return WebUtils.getSessionMutex(request.getSession());
}
```

在这段代码中主要目的是获取Session中的mutex对象。





## AbstractFlashMapManager分析

本节将对AbstractFlashMapManager类进行分析，首先对retrieveAndUpdate方法进行分析，完整代码如下：

```java
@Override
@Nullable
public final FlashMap retrieveAndUpdate(HttpServletRequest request, HttpServletResponse response) {
   List<FlashMap> allFlashMaps = retrieveFlashMaps(request);
   if (CollectionUtils.isEmpty(allFlashMaps)) {
      return null;
   }

   List<FlashMap> mapsToRemove = getExpiredFlashMaps(allFlashMaps);
   FlashMap match = getMatchingFlashMap(allFlashMaps, request);
   if (match != null) {
      mapsToRemove.add(match);
   }

   if (!mapsToRemove.isEmpty()) {
      Object mutex = getFlashMapsMutex(request);
      if (mutex != null) {
         synchronized (mutex) {
            allFlashMaps = retrieveFlashMaps(request);
            if (allFlashMaps != null) {
               allFlashMaps.removeAll(mapsToRemove);
               updateFlashMaps(allFlashMaps, request, response);
            }
         }
      }
      else {
         allFlashMaps.removeAll(mapsToRemove);
         updateFlashMaps(allFlashMaps, request, response);
      }
   }

   return match;
}
```

在该方法中主要调用逻辑如下：

1. 提取请求中的FlashMap对象集合，集合名称为allFlashMaps。
2. 获取过期FlashMap对象集合，集合名称为mapsToRemove。
3. 将第一个操作中得到的FlashMap集合和请求中的FlashMap进行匹配，匹配规则有两个：
   1. 重定向地址要和当前请求地址相同。
   2. 参数相同。
4. 将匹配结果放入到mapsToRemove集合中。
5. 将allFlashMaps中存在的mapsToRemove集合删除。
6. 将删除后的allFlashMaps更新。







## FlashMapManager 总结

本章围绕FlashMapManager 接口出发，介绍了FlashMapManager 接口的作用和两个实现类它们具体实现过程，此外对一些常见的FlashMapManager实现类做了相关测试用例作为源码调试的基础。