<div style='display: none'>

Copyright 2020 [HuiFer](http://github.com/huifer) All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

</div>


# 第二章 IoC 中的核心类
- 本章笔者将带领各位读者了解 SpringIoC 中的几个核心对象. 可能存在不完整的情况希望读者进行补充. 

在[第一章](/book/ch-01/第一章-容器环境搭建及基本使用.md)中笔者引出了三个类来作为 SpringIoC 的容器对象. 回顾一下三个对象分别是: `ClassPathXmlApplicationContext` 、 `XmlBeanFactory` 和 `FileSystemXmlApplicationContext`，这三个类也可以算作我们的核心类. 毕竟容器本身也很重要. 下面笔者将它们三个类的类图全部贴出来, 通过类图我们来找到同样重要的类



- 类图描述笔者就不在这里叙述了, 请各位读者自行阅读下面的图片内容. 

首先来看**`ClassPathXmlApplicationContext`** 类图

![ClassPathXmlApplicationContext](./images/ClassPathXmlApplicationContext.png)

其次来看**`FileSystemXmlApplicationContext`** 类图

![FileSystemXmlApplicationContext](./images/FileSystemXmlApplicationContext.png)

最终来看 **`XmlBeanFactory`** 类图

![XmlBeanFactory](./images/XmlBeanFactory.png)





类图看完了. 下面笔者来对其中的各个接口做一个简单的说明. 

- `Resource`: Spring 中资源的定义
- `ResourceLoader`: 提供了资源加载方法
- `BeanDefinitionReader`: 提供了读取资源对象到Bean定义的方法
- `DocumentLoader`: 提供了将资源文件转换成 `Document` 对象的方法
- `BeanDefinitionDocumentReader`: 提供了将 `Document` 对象读取并注册到容器的方法
- `EnvironmentCapable`: 提供了获取环境配置的方法
- `AliasRegistry`: 提供了关于 alias 的增删改查方法
- `SingletonBeanRegistry`: 提供了关于 单例 Bean 的增删改查方法
- `BeanDefinitionRegistry`: 提供了关于 Bean 定义的增删改查方法
- `BeanFactory`：提供了获取 Bean 实例的方法
- `HierarchicalBeanFactory`: 在 `BeanFactory`基础上提供了关于父 BeanFactory 的支持
- `ConfigurableBeanFactory`: 提供了对 BeanFactory 的设置方法在其中可以设置关于类加载器、转换服务等配置信息
- `AutowireCapableBeanFactory`: 提供了创建 Bean、注入 Bean、应用后置处理器(`BeanPostProcessor`)、摧毁 Bean 等方法

- `ApplicationContext`: 应用上下文核心接口, 各类上下文实现类都是它的实现类
- `ConfigurableApplicationContext`: 提供了配置应用上下文属性的方法
- `ListableBeanFactory`: 提供了搜索 Bean 的方法, 从容器中获取关于 Bean 的配置
- `Lifecycle`: 提供了关于容器的生命周期方法. 开始和停止





这些接口笔者对其进行了简单的分类. (各位读者如果有新的分类也可以进行补充)

-  第一类是关于资源处理, 比如 `Resource` 、`ResourceLoader`
-  第二类是关于注册形式, 在 Spring 中关于注册的几个核心就是 alias(`AliasRegistry`) 、Bean 定义注册(`BeanDefinitionRegistry`) 、单例Bean注册(`SingletonBeanRegistry`)
-  第三类是关于生命周期的, 生命周期又可以分为两类, 
   - 第一类是容器的生命周期, 容器生命周期的核心接口: `Lifecycle`
   - 第二类是Bean的生命周期, Bean 生命周期的接口有: `InitializingBean`、`DisposableBean`
-  第四类是关于Bean拓展的, 如: `BeanPostProcessor`、`Aware` 系列接口
-  第五类是关于上下文的接口主要以: `ApplicationContext` 作为主导接口
-  第六类是读取接口主要用来读取信息，如: `BeanDefinitionReader` . 这也可以归到资源处理中, 笔者在这里还是将其提取出来做一个单独的大类



- 第二章的内容到此也完成了, 本章将一些核心类, 在后续的章节中会进行详细介绍