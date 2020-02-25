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