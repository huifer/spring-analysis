# Spring SimpleConstructorNamespaceHandler
- 类全路径: `org.springframework.beans.factory.xml.SimpleConstructorNamespaceHandler`


- `SimpleConstructorNamespaceHandler` 中的 `decorate` 方法主要围绕 `BeanDefinition` 中 类型是 `ConstructorArgumentValues` 的属性值







## 方法分析

### decorate

- 方法签名: `org.springframework.beans.factory.xml.SimpleConstructorNamespaceHandler#decorate`

- 方法作用: 补充 BeanDefinition 中 `ConstructorArgumentValues` 的数据信息

- 执行流程
  1. 判断 Node 是否是 Attr 类型
  2. 通过`ParserContext` 解析 属性名称
  3. 属性值直接通过 Attr 接口获取
  4. 设置 `ConstructorArgumentValues` 中的数据 

		1. 设置`valueHolder`数据源
		2. 将 `valueHolder` 放入容器







```java
@Override
public BeanDefinitionHolder decorate(Node node, BeanDefinitionHolder definition, ParserContext parserContext) {
   // 是否是  Attr 类型
   if (node instanceof Attr) {
      Attr attr = (Attr) node;
      // 参数名称
      String argName = StringUtils.trimWhitespace(parserContext.getDelegate().getLocalName(attr));
      // 参数值
      String argValue = StringUtils.trimWhitespace(attr.getValue());

      // 构造函数的参数指
      ConstructorArgumentValues cvs = definition.getBeanDefinition().getConstructorArgumentValues();
      boolean ref = false;

      // handle -ref arguments
      if (argName.endsWith(REF_SUFFIX)) {
         ref = true;
         argName = argName.substring(0, argName.length() - REF_SUFFIX.length());
      }

      // 属性值持有对象
      ValueHolder valueHolder = new ValueHolder(ref ? new RuntimeBeanReference(argValue) : argValue);
      // 设置源数据
      valueHolder.setSource(parserContext.getReaderContext().extractSource(attr));

      // 是否是 - 开头
      // handle "escaped"/"_" arguments
      if (argName.startsWith(DELIMITER_PREFIX)) {
         // 去掉 - 留下真实字符串
         String arg = argName.substring(1).trim();

         // fast default check
         if (!StringUtils.hasText(arg)) {
            // 设置构造函数的数据值
            cvs.addGenericArgumentValue(valueHolder);
         }
         // assume an index otherwise
         else {
            int index = -1;
            try {
               // 解析 index 数据
               index = Integer.parseInt(arg);
            }
            catch (NumberFormatException ex) {
               parserContext.getReaderContext().error(
                     "Constructor argument '" + argName + "' specifies an invalid integer", attr);
            }
            if (index < 0) {
               parserContext.getReaderContext().error(
                     "Constructor argument '" + argName + "' specifies a negative index", attr);
            }

            if (cvs.hasIndexedArgumentValue(index)) {
               parserContext.getReaderContext().error(
                     "Constructor argument '" + argName + "' with index " + index + " already defined using <constructor-arg>." +
                           " Only one approach may be used per argument.", attr);
            }

            // 设置到容器map中
            cvs.addIndexedArgumentValue(index, valueHolder);
         }
      }
      // no escaping -> ctr name
      else {
         String name = Conventions.attributeNameToPropertyName(argName);
         if (containsArgWithName(name, cvs)) {
            parserContext.getReaderContext().error(
                  "Constructor argument '" + argName + "' already defined using <constructor-arg>." +
                        " Only one approach may be used per argument.", attr);
         }
         // 设置属性值持有者的名称
         valueHolder.setName(Conventions.attributeNameToPropertyName(argName));
         cvs.addGenericArgumentValue(valueHolder);
      }
   }
   return definition;
}
```






