# Spring Mergeable 阅读路线
- 本节主要围绕 Mergeable 接口做展开, 分析 Mergeable 接口的作用和子类实现的细节. 
    Mergeable 接口主要提供两个方法
    1. 是否需要合并
    2. 合并
    从方法上可以看出 Mergeable 主要作用: 合并对象. 
    
- 子类实现
    - [ManagedProperties](/docs/beans/BeanMetadataElement/Spring-ManagedProperties.md)
    - [ManagedList](/docs/beans/BeanMetadataElement/Spring-ManagedList.md)
        - ManagedArray
    - HtmlUnitRequestBuilder
    - [ManagedMap](/docs/beans/BeanMetadataElement/Spring-ManagedMap.md)
    - [ManagedSet](/docs/beans/BeanMetadataElement/Spring-ManagedSet.md)
    - MockHttpServletRequestBuilder
       - MockMultipartHttpServletRequestBuilder