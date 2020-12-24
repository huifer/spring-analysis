# Spring Constants
- 类全路径: `org.springframework.core.Constants`
- 类作用: 将某一个类的字段转换成map 

- `Constants` 方法主要集中在 构造函数中, 其他的方法属于 类型转换


先看成员变量 
## 成员变量

- 成员变量存储两个信息
    1. 类名
    2. 类中的字段及其字段值的map容器
    

```java

public class Constants {

    /** The name of the introspected class. */
    private final String className;

    /** Map from String field name to object value. */
    private final Map<String, Object> fieldCache = new HashMap<>();

}
```


## 方法分析

### 构造函数

- 构造函数逻辑
    1. 通过反射获取字段列表
    2. 字段列表转换成 字段名称, 字段值 放入容器

```java
	public Constants(Class<?> clazz) {
		Assert.notNull(clazz, "Class must not be null");
		this.className = clazz.getName();
		Field[] fields = clazz.getFields();
		for (Field field : fields) {
			if (ReflectionUtils.isPublicStaticFinal(field)) {
				String name = field.getName();
				try {
					Object value = field.get(null);
					this.fieldCache.put(name, value);
				}
				catch (IllegalAccessException ex) {
					// just leave this field and continue
				}
			}
		}
	}

```