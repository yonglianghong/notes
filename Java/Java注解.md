##### 1. 注解

注解（Annotation）相当于是一种特殊“注释”，Java有三类注解：

- 编译器使用，这类注解不会被编译进`.class`文件，编译后就被扔掉了
    - `@Override` 编译器检查该方法是否正确地实现覆写
    - `@Deprecated `编译器发出警告这是废弃的方法
    - `@SuppressWarnings `告诉编译器忽略此处代码产生的警告

- 由工具处理`.class`文件时使用的注解，比如加载class文件时会做动态修改，实现特殊功能。这类注解会被编译进class文件，但加载结束后并不会存在于内存中。
- 程序运行期间能够读取的注解，会被加载进JVM中（比如，一个配置了`@PostConstruct`的方法会在调用构造方法后自动被调用（这是Java代码读取该注解实现的功能，JVM并不会识别该注解）。）



##### 2. 定义注解

1. 定义注解时配置参数包括：

   - 基本类型
   - String
   - 枚举类型

   配置参数可以有默认值，如果参数名称是value，并且只有一个参数，那么可以省略参数名称

   使用`@interface`语法来定义注解（`Annotation`）

   ```java
   public @interface Report {
       int type() default 0;
       String level() default "info";
       String value() default "";
   }
   ```

   最常用的参数应当命名为`value`。



2. 元注解：修饰其他注解的注解就称为元注解（meta annotation）

   - @Target，定义`Annotation`被应用于源码的位置
     - 类或接口：ElementType.TYPE
     - 字段：ElementType.FIELD
     - 方法：ElementType.METHOD
     - 构造方法：ElementType.CONSTRUCTOR
     - 方法参数：ElementType.PARAMETER
   - @Retention，定义`Annotation`的生命周期，默认为CLASS
     - 仅编译期：RetentionPolicy.SOURCE
     - 仅class文件：RetentionPolicy.CLASS
     - 运行期：RetentionPolicy.RUNTIME

   - @Repeatable，定义`Annotation`是否可重复
   - @Inherited，定义子类是否可继承父类定义的`Annotation`

   ```java
   // @Target({
   //     ElementType.METHOD,
   //     ElementType.FIELD
   // })
   @Target(ElementType.TYPE)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface Report {
       int type() default 0;
       String level() default "info";
       String value() default "";
   }
   ```



##### 3. 处理注解

注解定义后也是一种`class`，所有的注解都继承自`java.lang.annotation.Annotation`。定义了注解，本身对程序逻辑没有任何影响。我们必须自己编写代码来使用注解。

使用反射API读取Annotation的方法包括：

- 判断某个注解是否存在于Class、Field、Method或Constructor：
    - Class.isAnnotationPresent(Class)
    - Field.isAnnotationPresent(Class)
    - Method.isAnnotationPresent(Class)
    - Constructor.isAnnotationPresent(Class)

- 使用反射API读取Annotation：
    - Class.getAnnotation(Class)
    - Field.getAnnotation(Class)
    - Method.getAnnotation(Class)
    - Constructor.getAnnotation(Class)

```java
// 判断@Report是否存在于Person类:
Person.class.isAnnotationPresent(Report.class);
// 获取Person定义的@Report注解:
Report report = Person.class.getAnnotation(Report.class);
int type = report.type();
String level = report.level();
```



##### 4. 注解示例

```java
// 定义一个Range注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Range {
    int min() default 0;
    int max() default 255;
}

// JavaBean中使用注解
public class Person {
    @Range(min=1, max=20)
    public String name;

    @Range(max=10)
    public String city;

}

// 处理注解，检查Person实例的String字段长度是否满足@Range的定义
void check(Person person) throws IllegalArgumentException, ReflectiveOperationException {
    // 遍历所有Field:
    for (Field field : person.getClass().getFields()) {
        // 获取Field定义的@Range:
        Range range = field.getAnnotation(Range.class);
        // 如果@Range存在:
        if (range != null) {
            // 获取Field的值:
            Object value = field.get(person);
            // 如果值是String:
            if (value instanceof String) {
                String s = (String) value;
                // 判断值是否满足@Range的min/max:
                if (s.length() < range.min() || s.length() > range.max()) {
                    throw new IllegalArgumentException("Invalid field: " + field.getName());
                }
            }
        }
    }
}
```

