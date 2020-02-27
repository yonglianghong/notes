##### 1. 一切都是对象

面向对象程序设计（Object-oriented Programming）

编程的目的是利用计算机等工具解决问题。编程的过程即将问题的求解模型映射为机器模型，面向对象的程序设计提供了一种更灵活的方式。将求解问题涉及到的元素抽象为对象，利用对象与对象之间的交互实现对问题的求解。而相同特性和行为的对象即构成了类。

在Java中，类（class）即构建对象的模板。由类构造（construct）对象的过程称为创建类的实例（instance），类实例中的数据称为域（field），操作数据的过程称为方法（method）。每个类实例的都有一组特定的域值，这些值的集合就是这个对象的当前状态（state）。

*封装*（encapsulation）即将数据和行为组合在一个对象中，并对对象的使用者隐藏细节。关键在于程序仅通过对象的方法与对象数据交互。

对象三要素：

- 状态（state）
- 行为（behavior）
- 标识（identity）



##### 2. 基本类型及关键字

Java基本类型：

| 基本类型 | 大小    | 最小值    | 最大值         | 包装类    | 默认值         |
| -------- | ------- | --------- | -------------- | --------- | -------------- |
| boolean  |         |           |                | Boolean   | false          |
| char     | 16-bit  | Unicode o | Unicode 2^16-1 | Character | '\u0000'(null) |
| byte     | 8 bits  | -128      | +127           | Byte      | (byte)0        |
| short    | 16 bits | -2^15     | +2^15-1        | Short     | (short)0       |
| int      | 32 bits | -2^31     | +2^31-1        | Integer   | 0              |
| long     | 64 bits | -2^63     | +2^63-1        | Long      | 0L             |
| float    | 32 bits | IEEE754   | IEEE754        | Float     | 0.0f           |
| double   | 64 bits | IEEE754   | IEEE754        | Float     | 0.0d           |



两个高精度类：

- BigInteger

- BigDecimal



关键字：

- this：调用方法的这个对象

- static：可修饰类、变量、方法，也可以定义静态代码块
  - 若修饰变量，只占用一份存储
  - 若修饰方法，内部不能调用非静态方法

- final



权限管理：

按照类库（library）、包（package）、类（class）、方法/属性（method/field）进行命名空间管理。一共有4中权限，其关键字分别是public、protected、默认（包）、private，均可修饰类、方法、属性。

- public：公共访问权限。
- protected：继承访问权限，其修饰方法、属性只能被类本身的方法及子类访问，即使子类在不同的包。同时提供包访问权限。
- 默认/包（default/friendly）访问权限：同一个包中可以访问
- private：私有访问权限

注：

1. 每一个编译文件只能有一个public类
2. 可以`import static abc.xyz.Hello.*`，这样在调用静态方法时可以省略类名



可变参数



##### 3. 初始化

类的成员变量会被自动初始化为默认值。

初始化顺序：

1. 静态对象
2. 构造器
3. 

##### 4. 操作符

算术操作符：

- 加`+`、减`-`、乘`*`、除`/`、取余`%`，可与`=`组合，简化写法（String重载了`+`、`+=` 这两个操作符）

- 自增`++`，自减`--`

关系操作符：大于`>`、小于`<`、大于等于`>=`、小于等于`<=`、等于`==`、不等于`!=`

> ==和!=适用于对象及基本类型的比较，比较对象的引用。equals默认比较对象的引用，可通过覆写比较对象的内容。

逻辑操作符：与`&&`、或`||`、非`!`，返回一个布尔值（true或false），存在短路现象

按位操作符：与`&`、或`|`、异或`^`、按位取反`~`，可与`=`组合（`~`除外），简化写法

移位操作符：左移`<<`、右移`>>`、无符号右移`>>>`，可与`=`组合

三元操作符：`boolean-exp ? value0 : value1`



##### 5. 语句

条件语句（if-else）：

```java
if(Boolean-expression)
  statement
else if(Boolean-expression)
  statement
else
  statement
```

循环语句：

```java
// while
while(Boolean-expression)
  statement

// do-while
do
  statement
while(Boolean-expression)

// for
for(initialization; Boolean-expression; step)
  statement
```

迭代语句：

```java
for(class-name declare : Iterable)
  statement
```

选择语句：

```java
// selector支持整型、Enum、String
switch(selector){
  case value1 : statement;break;
  case value2 : statement;break;
  ...
  default : statment;
}
```

关于循环：

- 无限循环：`while(true)`,`for(;;)`
- break：中断并跳出循环
- continue：中断并继续下一次循环
- return：返回值，跳出方法

label 标签：

```java
label: outer-iteration{
  inner-iteration{
    //...
    break; // 中断内部迭代，回到外部迭代
    //...
    continue; // 执行点回到内部循环的起始处
    //...
    break label; // 中断所有迭代
    //...
    continue label; // 执行点回到外部循环的起始处
  }
}
```



 

java/bin、java/lib

常用命令：

java

javac

javadoc



2. 继承、多态、接口





##### 5. 编译与执行

*The `java` command starts a Java application. It does this by starting the Java Runtime Environment (JRE), loading the specified class, and calling that class's `main()` method.* 

*The JRE searches for the startup class (and other classes used by the application) in three sets of locations: the bootstrap class path, the installed extensions, and the user's class path.*[1](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html)

*Java™ Platform, Standard Edition Development Kit (JDK™). The JDK is a development environment for building applications, applets, and components using the Java programming language.*

*The JDK includes tools useful for developing and testing programs written in the Java programming language and running on the Java platform.*[2](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html)

`java/javac [<option>...] <classname> [<argument>...]`

- option is a command line option (starting with a "-" character)

- class-name is a fully qualified Java class name 全限定名

`class path`：

- The class path is the path that the Java Runtime Environment (JRE) searches for classes and other resource files.
- class search path of directories and zip/jar files.（搜寻用户class类的路径/jar/zip）

使用示例：

```shell
# 编译 
# ➜ algorithms1 git:(develop) ✗
javac -cp /Users/ylh/Code/Projects/programming/lib/algs4.jar Test.java

# 解释
# 指定package com.ylh.coursera.algorithms1;分别在programming项目java和~目录执行
java -cp .:/Users/ylh/Code/Projects/programming/lib/algs4.jar com.ylh.coursera.algorithms1.Test
java -cp  /Users/ylh/Code/Projects/programming/src/main/java:/Users/ylh/Code/Projects/programming/lib/algs4.jar com.ylh.coursera.algorithms1.Test

# 不指定package，在programming项目algorithm1目录执行
java -cp .:/Users/ylh/Code/Projects/programming/lib/algs4.jar Test
```

`.`代表当前目录，如果未指定package，表示为默认包，因此在解释执行时需要将`.`加入到`CLASSPATH`中。并且不管是否指定package，解释执行时都应该使用类的*全限定名*。

编译时，经测试是否指定package不影响，只要能够找到java文件即可。

`import`语句也会从`CLASSPATH`中去寻找导入的类文件。

注：

1. mac安装jdk后，没有设置`PATH`和`CLASSPATH`也可以执行`java -version`。[3](https://docs.oracle.com/javase/tutorial/essential/environment/paths.html)
2. 参考IDEA执行java程序时实际使用的java命令，将`$JAVA_HOME/jre/lib`、`$JAVA_HOME/jre/lib/ext`、`$JAVA_HOME/lib`下的jar文件加入到了`CLASSPATH`中。
3. jar包实际相当于zip包，将class文件集中放在一起，按`package`组织目录层级。要执行jar包里面的类，直接将jar包放进`CLASSPATH`中即可，比如`java -cp hello.jar abc.xyz.Hello`。
4. jar包可以包含`/META-INF/MANIFEST.MF`文件，指定`Main-Class`和其它信息。这样就不必在命令行指定启动的类名，比如`java -jar hello.jar`。
5. jdk有默认的`CLASSPATH`目录`.`吗？上述示例需要加`.`才能正常执行。



