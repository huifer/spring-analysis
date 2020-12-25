# Spring BeanFactory 阅读指南
- **BeanFactory** 主要用来访问 Spring Bean 容器, 它作为访问容器的根接口. 有很多拓展的接口. 
    - `org.springframework.beans.factory.HierarchicalBeanFactory`
    - `org.springframework.beans.factory.ListableBeanFactory`
    - `org.springframework.beans.factory.config.ConfigurableBeanFactory`
    - `org.springframework.context.ApplicationContext`
    -`org.springframework.jndi.support.SimpleJndiBeanFactory`
    - ...
    
- 本节将围绕 `BeanFactory` 接口展开. 介绍 BeanFactory 系列接口. 



## 接口及实现类分析



- 在 Spring 中 BeanFactory 是一个相当大的一个接口, 其子类,子接口相当多. 需要一步一步了解. 相关子类分析可查看下面这些文档



- [HierarchicalBeanFactory](/docs/beans/factory/BeanFactory/Spring-HierarchicalBeanFactory.md)
    - [ConfigurableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ConfigurableBeanFactory.md)
        - [AbstractBeanFactory](/docs/beans/factory/BeanFactory/Spring-AbstractBeanFactory.md)
            - [AbstractAutowireCapableBeanFactory](/docs/beans/factory/BeanFactory/Spring-AbstractAutowireCapableBeanFactory.md)
                - [DefaultListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-DefaultListableBeanFactory.md)
                    - [XmlBeanFactory](/docs/beans/factory/xml/Spring-XmlBeanFactory.md)
        - [ConfigurableListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ConfigurableListableBeanFactory.md)
            - [DefaultListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-DefaultListableBeanFactory.md)
                - [XmlBeanFactory](/docs/beans/factory/xml/Spring-XmlBeanFactory.md)
    - [ApplicationContext](/docs/core/context/ApplicationContext/Spring-ApplicationContext.md)
        - [ConfigurableApplicationContext](/docs/core/context/ApplicationContext/Spring-ConfigurableApplicationContext.md)
            - [AbstractApplicationContext](/docs/core/context/ApplicationContext/Spring-AbstractApplicationContext.md)
                - [AbstractRefreshableApplicationContext](/docs/core/context/support/Spring-AbstractRefreshableApplicationContext.md)
                    - [AbstractRefreshableConfigApplicationContext](/docs/core/context/support/Spring-AbstractRefreshableConfigApplicationContext.md)
                        - [AbstractXmlApplicationContext](/docs/core/context/support/Spring-AbstractXmlApplicationContext.md)
                            - [FileSystemXmlApplicationContext](/docs/core/context/support/Spring-FileSystemXmlApplicationContext.md)
                            - [ClassPathXmlApplicationContext](/docs/core/context/support/Spring-ClassPathXmlApplicationContext.md)
                        - AbstractRefreshableWebApplicationContext
                            - XmlWebApplicationContext
                            - GroovyWebApplicationContext
                            - AnnotationConfigWebApplicationContext
                        
            - ConfigurableWebApplicationContext
                - GenericWebApplicationContext
                - StaticWebApplicationContext
                - AbstractRefreshableWebApplicationContext
                    - XmlWebApplicationContext
                    - GroovyWebApplicationContext
                    - AnnotationConfigWebApplicationContext
        - WebApplicationContext
            - StubWebApplicationContext
            - ConfigurableWebApplicationContext
                - GenericWebApplicationContext
                - StaticWebApplicationContext
                - AbstractRefreshableWebApplicationContext
                    - XmlWebApplicationContext
                    - GroovyWebApplicationContext
                    - AnnotationConfigWebApplicationContext
- [SimpleJndiBeanFactory](/docs/beans/factory/BeanFactory/impl/Spring-SimpleJndiBeanFactory.md)
- [AutowireCapableBeanFactory](/docs/beans/factory/BeanFactory/Spring-AutowireCapableBeanFactory.md)
    - [ConfigurableListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ConfigurableListableBeanFactory.md)
        - [DefaultListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-DefaultListableBeanFactory.md)
            - [XmlBeanFactory](/docs/beans/factory/xml/Spring-XmlBeanFactory.md)
    - [AbstractAutowireCapableBeanFactory](/docs/beans/factory/BeanFactory/Spring-AbstractAutowireCapableBeanFactory.md)
        - [DefaultListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-DefaultListableBeanFactory.md)
            - [XmlBeanFactory](/docs/beans/factory/xml/Spring-XmlBeanFactory.md)
    - StubBeanFactory     
- [ListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ListableBeanFactory.md)
    - [ApplicationContext](/docs/core/context/ApplicationContext/Spring-ApplicationContext.md)
    - StaticListableBeanFactory
    - [ConfigurableListableBeanFactory](/docs/beans/factory/BeanFactory/Spring-ConfigurableListableBeanFactory.md)
        - [AbstractApplicationContext](/docs/core/context/ApplicationContext/Spring-AbstractApplicationContext.md)
            - [AbstractRefreshableApplicationContext](/docs/core/context/support/Spring-AbstractRefreshableApplicationContext.md)
                - [AbstractRefreshableConfigApplicationContext](/docs/core/context/support/Spring-AbstractRefreshableConfigApplicationContext.md)
                    - [AbstractXmlApplicationContext](/docs/core/context/support/Spring-AbstractXmlApplicationContext.md)
                        - [FileSystemXmlApplicationContext](/docs/core/context/support/Spring-FileSystemXmlApplicationContext.md)
                        - [ClassPathXmlApplicationContext](/docs/core/context/support/Spring-ClassPathXmlApplicationContext.md)
                    - AbstractRefreshableWebApplicationContext
                        - XmlWebApplicationContext
                        - GroovyWebApplicationContext
                        - AnnotationConfigWebApplicationContext
                    
        - ConfigurableWebApplicationContext
            - GenericWebApplicationContext
            - StaticWebApplicationContext
            - AbstractRefreshableWebApplicationContext
                - XmlWebApplicationContext
                - GroovyWebApplicationContext
                - AnnotationConfigWebApplicationContext

    - WebApplicationContext
        - StubWebApplicationContext
        - ConfigurableWebApplicationContext
            - GenericWebApplicationContext
            - StaticWebApplicationContext
            - AbstractRefreshableWebApplicationContext
                - XmlWebApplicationContext
                - GroovyWebApplicationContext
                - AnnotationConfigWebApplicationContext
    - StaticListableBeanFactory
        - StubBeanFactory
    - ConfigurableListableBeanFactory
    
    
