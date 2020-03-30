##### 0. 前言

在程序中，总会出现错误，Java异常即一套错误处理机制。具体就是定义了一类`class`，代表各种异常。异常可以在任何地方抛出，只需要在上层捕获，这样就实现了和方法调用分离。



##### 1. Throwable类结构
- `Throwable` 异常的基类
  - `Error`表示错误，程序一般不能直接处理
    - `OutOfMemoryError` 内存耗尽
    - `StackOverflowError` 栈溢出
  - `Exception`表示运行时的错误，可以被捕获并处理
    - `RuntimeException`
      - `NullPointException` 空指针
      - `IndexOutOfBoundsException` 数组越界
    - 非`RuntimeException`（比如`IOException`等）
      - `FileNotFoundException` 未找到文件



##### 2. 捕获异常

1. Java规定：
   - 必须捕获的异常，包括`Exception`及其子类，但不包括`RuntimeException`及其子类，这种类型的异常称为Checked Exception
   - 不需要捕获的异常，包括`Error`及其子类，`RuntimeException`及其子类

2. try...catch...finally

   ```java
   public static void main(String[] args) {
       try {
           process1();
           process2();
           process3();
       } catch (UnsupportedEncodingException e) {
           System.out.println("Bad encoding");
       } catch (IOException e) {
           System.out.println("IO error");
       } finally {
           System.out.println("END");
       }
   }
   ```

   存在多个catch的时候，catch的顺序非常重要：子类必须写在前面。

   finally语句无论有无异常都会执行，它是可选的；

   一个catch语句也可以匹配多个非继承关系的异常，比如`catch (IOException | NumberFormatException e)`。

   `printStackTrace()`可打印异常的调用调用栈，便于调试。



##### 3. 抛出异常
当某个方法抛出了异常时，如果当前方法没有捕获异常，异常就会被抛到上层调用方法，直到遇到某个try ... catch被捕获为止？

抛出异常分为两步：

1. 创建某个Exception的实例；
2. 用throw语句抛出。



##### 4. 自定义异常
可以首先定义一个`BaseException`作为“根异常”，然后，派生出各种业务类型的异常。

`BaseException`需要从一个合适的`Exception`派生，通常从`RuntimeException`派生：

```java
public class BaseException extends RuntimeException {
    public BaseException() {
        super();
    }

    public BaseException(String message, Throwable cause) {
        super(message, cause);
    }
    
    public BaseException(String message) {
        super(message);
    }
    
    public BaseException(Throwable cause) {
        super(cause);
    }

}
```

其他业务类型的异常就可以从BaseException派生：

```java
public class UserNotFoundException extends BaseException {

}

public class LoginFailedException extends BaseException {
}

...
```



##### 5. 使用断言
断言（Assertion）是一种调试程序的方式。在Java中，使用assert关键字来实现断言。

```java
public static void main(String[] args) {
    double x = Math.abs(-123.45);
    assert x >= 0;
    System.out.println(x);
}
```


语句`assert x >= 0`即为断言，断言条件`x >= 0`预期为`true`。如果计算结果为`false`，则断言失败，抛出`AssertionError`。

使用assert语句时，还可以添加一个可选的断言消息：

`assert x >= 0 : "x must >= 0";`

这样，断言失败的时候，`AssertionError`会带上消息`x must >= 0`，更加便于调试。