##### 1. 一切都是对象

面向对象程序设计（Object-oriented Programming）

编程的目的是利用计算机等工具解决问题。编程的过程即将问题的求解模型映射为机器模型，面向对象的程序设计提供了一种更灵活的方式。将求解问题涉及到的元素抽象为对象，利用对象与对象之间的交互实现对问题的求解。而相同特性和行为的对象即构成了类。

在Java中，类（class）即构建对象的模板。由类构造（construct）对象的过程称为创建类的实例（instance），类实例中的数据称为域（field），操作数据的过程称为方法（method）。每个类实例的都有一组特定的域值，这些值的集合就是这个对象的当前状态（state）。

*封装*（encapsulation）即将数据和行为组合在一个对象中，并对对象的使用者隐藏细节。关键在于程序仅通过对象的方法与对象数据交互。

对象三要素：

- 状态（state）
- 行为（behavior）
- 标识（identity）



##### 2. 基本概念

1. Java基本类型：

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



2. 两个高精度类：
   - BigInteger
   - BigDecimal



3. 关键字：

   - this：调用方法的这个对象

   - static：可修饰类、变量、方法，也可以定义`静态代码块`

     - 若修饰变量，只占用一份存储空间，非static变量则是每个对象都有一份存储空间
     - 若修饰方法，方法内部不能调用非静态方法
     - 声明为static时，意味着该变量或方法不会与包含它的那个类的任何对象实例关联在一起

   - final：可以修饰类?、变量、方法，表示不能改变，声明对象时使用final，只是表示引用本身不变（关联的对象本身可以改变）

     常用用法：

     ```java
     // 静态不可变变量
     public static final String "abc"；
     
     // 静态方法
     public static void test(){}
     
     // 导入静态方法
     import static abc.xyz.Hello.*
     ```

     

4. 权限管理：

   按照类库（library）、包（package）、类（class）、方法/属性（method/field）进行命名空间管理。一共有4种权限，其关键字分别是public、protected、默认（包）、private，均可修饰类、方法、属性。

   - public：公共访问权限。

   - protected：继承访问权限，其修饰方法、属性只能被类本身的方法及子类访问，即使子类在不同的包。同时提供包访问权限。

   - 默认/包（default/friendly）：同一个包中可以访问。

   - private：私有访问权限。

     注：

     - 每一个编译文件只能有一个public类
     - 可以`import static abc.xyz.Hello.*`，这样在调用静态方法时可以省略类名

5. 方法与参数：

   方法名与参数一起构成了方法`签名`，方法（构造或普通）可以`重载`。

   可变参数列表：`f(String... args)`



##### 3. 初始化

类的成员变量会被自动初始化为默认值，而局部变量则不会被自动初始化。

对象在class文件加载完毕，以及为各成员在方法区开辟好内存空间（*初始化二进制的零*）之后，就开始所谓“初始化”的步骤：

1. 基类静态代码块，基类静态成员字段 （并列优先级，按代码中出现先后顺序执行）（只有第一次加载类时执行）
2. 派生类静态代码块，派生类静态成员字段 （并列优先级，按代码中出现先后顺序执行）（只有第一次加载类时执行）
3. 基类普通代码块，基类普通成员字段 （并列优先级，按代码中出现先后顺序执行）
4. 基类构造函数
5. 派生类普通代码块，派生类普通成员字段 （并列优先级，按代码中出现先后顺序执行）
6. 派生类构造函数

注意，对于的静态过程，只在这个类第一次被加载的时候才运行。如果创建两个对象，第二次创建就只执行3，4，5，6步。[1](https://www.zhihu.com/question/49196023/answer/114734606)



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

迭代语句：适用于任何`Iterable`对象

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



##### 6. 继承、抽象、接口

1. 继承

   `extends`关键字，描述的是类的一种复用，所有的类都自动继承自`Object`类。

   在子类中，调用父类相关的方法（或构造方法），用`super`关键字。父类的含参构造器，子类必须显式的调用。

   向上或向下转型：父类与子类支持相互转换，子类支持自动向上转型为父类，父类可以强制转型为子类。

2. 多态

   继承允许将对象视为自身或者父类的类型来使用，这提供了一种便利，就是将定义与实现进行了分离。

   绑定：方法与实际对象关联起来称为绑定。

   - 前期绑定：编译时就能确定
   - 后期绑定/运行时绑定/动态绑定：即在真正运行才能确定，除了static、final方法（private属于final方法）外，java所有的方法都是后期绑定。

   多态只针对普通方法，不包括私有方法、静态方法、属性。

3. 构造器：检查对象是否被正确地构造

   只有基类的构造器才能够保证自己的成员被正确初始化，所以基类的构造器总是在派生类构造过程中调用，而且按照继承层次逐渐向上链接，以使每个基类的构造器都能得到调用。

4. 协变：返回类型支持协变

   ```java
   class Hello(){
     Object f(){}
   }
   
   class World extends Path{
     Sting f(){}
   }
   ```

5. 类之间的关系

   - is-a：纯继承
   - is-like-a：扩展

6. 抽象类：将类的定义部分抽象

   含有抽象方法的类叫抽象类，用关键字`abstract`定义。

7. 接口：将类的定义完全抽象

   关键字`interface`，接口中的方法默认是public的，属性默认是static和final的。

   接口之间可以相互继承（本身就可以理解为类）

   一个类可以实现多个接口，继承一个类（普通或者抽象类），可以灵活向上转型为多个基类型。

8. 内部类

   内部类拥有对其外围类所有成员的访问权。

   - 外部类.this：内部类使用外部类
   - 外部类对象.new：在其他类构建外部类里面的内部类对象，必须借助外部类对象

   匿名内部类：

   - 声明：`new ClassName(){}`
   - 含参构造：`new ClassName(x){}`
   - 使用外部参数：类里面使用比如外部方法的参数，则参数必须是final的
   - 可以继承或实现接口（只能实现一个），两者二选一：`new InterfaceName(){}`

9. 嵌套类：static类



##### 7. 编译与执行

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



