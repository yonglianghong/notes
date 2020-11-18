### 从语句到表达式

> 语句（statement）：一段可以执行的代码
>
> 表达式（expression）：一段可以被求值的代码

在Java中语句和表达式是有区分的，表达式必须在return或者等号右侧，而在scala中，一切都是表达式。

假设我们在公司的内网和外网要从不同的域名访问一样的机器：

```java
//Java代码
String urlString = null;
String hostName = InetAddress.getLocalHost().getHostName();
if (isInnerHost(hostName)) {
  urlString = "http://inner.host";
} else {
  urlString = "http://outter.host";
}
```

scala如此简洁：

```scala
val hostName = InetAddress.getLocalHost.getHostName
val urlString = if (isInnerHost(hostName)) "http://inner.host" else "http://outter.host"
```

> 这样的好处是啥？

1. 代码简练,符合直觉
2. urlString 是值而不是变量,有效防止 urlString 在后续的代码中被更改（编译时排错）

