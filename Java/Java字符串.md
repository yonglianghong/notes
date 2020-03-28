##### 1. 可变与不可变字符串

1. String：
   - The String class represents character strings. All string literals in Java programs, such as "abc", are implemented as instances of this class.（String类表示字符串，Java中所有字符串都是此类的实例。）

   - String是常量，被创建后不能被更改；

     ```java
     String str = "abc"; 
     // 等价于
     char data[] = {'a', 'b', 'c'};
     String str = new String(data); 
     ```

   - The class String includes methods for examining individual characters of the sequence, for comparing strings, for searching strings, for extracting substrings, and for creating a copy of a string with all characters translated to uppercase or to lowercase. Case mapping is based on the Unicode Standard version specified by the Character class.（String类包含用于检查序列中的各个字符串、比较字符串、搜索字符串、提取字符串以及创建字符串副本（uppercase or lowercase）的方法）

   - 重载`+`、`+=`，基于StringBuilder的append()方法和所有类的toString方法实现

   - 主要方法：

     ```java
     // 检查字符串
     
     // 比较字符串
     boolean equals(Object anObject)
     boolean equalsIgnoreCase(String anotherString)
     int compareTo(String anotherString) // 按字典序比较
     int compareToIgnoreCase(String str)
     boolean contains(CharSequence s)
     boolean contentEquals(CharSequence cs)
     boolean contentEquals(StringBuffer sb)
     
     // 搜索字符串
     char charAt(int index)
     int indexOf(int ch)
     int indexOf(String str)
     boolean startsWith(String prefix)
     boolean endsWith(String suffix)
       
     // 提取字符串
     String substring(int beginIndex)
     String substring(int beginIndex, int endIndex)
     String concat(String str)
     
     // 创建副本
     String toLowerCase()
     String toUpperCase()
       
     // 其他
     int length() // 返回长度
     char[] toCharArray()
     String replace(CharSequence target, CharSequence replacement)
     String trim() // 两端空白字符去掉，若没有发生改变则返回原对象
     static String valueOf()
     String[] split(String regex) // 从正则匹配的字符切开
     String[] split(String regex, int limit) // 限制切割的次数
     String intern() // 为每个字符序列仅生成一个String引用
     ```

     

2. StringBuilder

   - A mutable sequence of characters.（可变的字符序列，不保证线程安全。）

   - The principal operations on a StringBuilder are the append and insert methods, which are overloaded so as to accept data of any type.The append method always adds these characters at the end of the builder; the insert method adds the characters at a specified point.（主要操作包括append和insert，append添加在末尾，insert添加在指定位置）

     ```java
     z -> "start"
     z.append("le") => "startle"
     z.insert(4, "le") => "starlet"
     sb.append(x) <=> sb.insert(sb.length(), x)
     ```

   - Every string builder has a capacity.If the internal buffer overflows, it is automatically made larger.（每个stringbuilder都有容量，内部缓存区满了后，可以自动扩容）

3. StringBuffer
   - A thread-safe, mutable sequence of characters.（线程安全，可变字符序列）
   - The principal operations on a StringBuffer are the append and insert methods, which are overloaded so as to accept data of any type. (主要是append和insert方法)
   - Every string buffer has a capacity. If the internal buffer overflows, it is automatically made larger.（每个stringbuffer都有容量，会自动扩容）



##### 2. 格式化

Java的字符串格式化可用于`PrintStream`和`PrintWriter`，比如：

```java
System.out.println("Row 1: [" + 5 + " " + 5.332542 + "]");
System.out.format("Row 1: [%d %f]\n", 5, 5.332542);
System.out.printf("Row 1: [%d %f]\n", 5, 5.332542);

// 输出
Row 1: [5 5.332542]
```

其中，`format()`和`printf()`是等价的。

在Java中，所有的格式化功能都由`java.util.Formatter`类处理。

格式化语法：`%[argument_index][flags][width][.precision]conversion`

- argument_index：如果格式化参数超过1个，可以指定index
- flags：比如可以指定数字前面的+、-、千位分隔符`,`
- width：字符长度
- precision：精度
- conversion：格式化的类型



转换符：

| 转换符 | 说明                                          | 示例        |
| ------ | --------------------------------------------- | ----------- |
| %s     | 字符串类型                                    | ”microsoft“ |
| %c     | 字符类型                                      | 'm'         |
| %b     | 布尔类型                                      | true        |
| %d     | 整数类型（十进制）                            | 99          |
| %x     | 整数类型（十六进制）                          | FF          |
| %o     | 整数类型（八进制）                            | 77          |
| %f     | 浮点类型                                      | 99.99       |
| %a     | 十六进制浮点类型                              | FF.35AE     |
| %e     | 指数类型                                      | 9.38e+5     |
| %%     | 百分比类型                                    | %           |
| %n     | 换行符                                        |             |
| %tx    | 日期和时间类型（x代表不同的时间与时间转换符） |             |

转换符标志：

| 标志 | 说明                                                         | 示例                         | 结果             |
| ---- | ------------------------------------------------------------ | ---------------------------- | ---------------- |
| +    | 为正数或者负数添加符合                                       | ("%+d", 15)                  | +15              |
| -    | 左对齐                                                       | ("%-5d", 15)                 | \|15   \|        |
| 0    | 数字前面补0                                                  | ("%04d", 99)                 | 0099             |
| 空格 | 在整数前面添加指定数量的空格                                 | ("% 4d", 99)                 | \| 99\|          |
| ,    | 以”,“对数字分组                                              | ("%,f", 9999.99)             | 9,999.990000     |
| (    | 使用括号包含负数                                             | ("%(f", -99.99)              | (99.990000)      |
| #    | 如果是浮点数则包含小数点，如果是16进制或者8进制，则加0x或者0 | ("%#x", 99)<br />("%#o", 99) | 0x63<br />0143   |
| <    | 格式化前一个转换符所描述的参数                               | ("%f和%<3.2f", 99.45)        | 99.450000和99.45 |
| $    | 被格式化的参数索引                                           | `("%1$d,%2$s", 99, "abc")`   | 99,abc           |



##### 3. 正则匹配

正则匹配是指用正则表达式来搜索、编辑或处理文本，正则表达式定义了处理的模式。

`java.util.regex`包下主要包括`Pattern`、`Matcher`。定义一个正则表达式`String r = ""`，调用`Pattern.compile(r)`编译表达式，返回一个`Pattern`对象，再调用`Pattern.matches(content)`对内容进行正则匹配，得到`Matcher`对象。



正则表达式语法：

`\\`表示一个转义反斜线，其后的字符具有特殊意义。

`\\\\`表示一个普通的反斜线。

| 字符          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| \             | 将下一字符标记为特殊字符、文本、反向引用或八进制转义符。例如，"n"匹配字符"n"。"\n"匹配换行符。序列"\\\\"匹配"\\"，"\\("匹配"("。 |
| ^             | 匹配输入字符串开始的位置。如果设置了 **RegExp** 对象的 **Multiline** 属性，^ 还会与"\n"或"\r"之后的位置匹配。 |
| $             | 匹配输入字符串结尾的位置。如果设置了 **RegExp** 对象的 **Multiline** 属性，$ 还会与"\n"或"\r"之前的位置匹配。 |
| *             | 零次或多次匹配前面的字符或子表达式。例如，zo* 匹配"z"和"zoo"。* 等效于 {0,}。 |
| +             | 一次或多次匹配前面的字符或子表达式。例如，"zo+"与"zo"和"zoo"匹配，但与"z"不匹配。+ 等效于 {1,}。 |
| ?             | 零次或一次匹配前面的字符或子表达式。例如，"do(es)?"匹配"do"或"does"中的"do"。? 等效于 {0,1}。 |
| {*n*}         | *n* 是非负整数。正好匹配 *n* 次。例如，"o{2}"与"Bob"中的"o"不匹配，但与"food"中的两个"o"匹配。 |
| {*n*,}        | *n* 是非负整数。至少匹配 *n* 次。例如，"o{2,}"不匹配"Bob"中的"o"，而匹配"foooood"中的所有 o。"o{1,}"等效于"o+"。"o{0,}"等效于"o*"。 |
| {*n*,*m*}     | *m* 和 *n* 是非负整数，其中 *n* <= *m*。匹配至少 *n* 次，至多 *m* 次。例如，"o{1,3}"匹配"fooooood"中的头三个 o。'o{0,1}' 等效于 'o?'。注意：您不能将空格插入逗号和数字之间。 |
| ?             | 当此字符紧随任何其他限定符（*、+、?、{*n*}、{*n*,}、{*n*,*m*}）之后时，匹配模式是"非贪心的"。"非贪心的"模式匹配搜索到的、尽可能短的字符串，而默认的"贪心的"模式匹配搜索到的、尽可能长的字符串。例如，在字符串"oooo"中，"o+?"只匹配单个"o"，而"o+"匹配所有"o"。 |
| .             | 匹配除"\r\n"之外的任何单个字符。若要匹配包括"\r\n"在内的任意字符，请使用诸如"[\s\S]"之类的模式。 |
| (*pattern*)   | 匹配 *pattern* 并捕获该匹配的子表达式。可以使用 **$0…$9** 属性从结果"匹配"集合中检索捕获的匹配。若要匹配括号字符 ( )，请使用"\("或者"\)"。 |
| (?:*pattern*) | 匹配 *pattern* 但不捕获该匹配的子表达式，即它是一个非捕获匹配，不存储供以后使用的匹配。这对于用"or"字符 (\|) 组合模式部件的情况很有用。例如，'industr(?:y\|ies) 是比 'industry\|industries' 更经济的表达式。 |
| (?=*pattern*) | 执行正向预测先行搜索的子表达式，该表达式匹配处于匹配 *pattern* 的字符串的起始点的字符串。它是一个非捕获匹配，即不能捕获供以后使用的匹配。例如，'Windows (?=95\|98\|NT\|2000)' 匹配"Windows 2000"中的"Windows"，但不匹配"Windows 3.1"中的"Windows"。预测先行不占用字符，即发生匹配后，下一匹配的搜索紧随上一匹配之后，而不是在组成预测先行的字符后。 |
| (?!*pattern*) | 执行反向预测先行搜索的子表达式，该表达式匹配不处于匹配 *pattern* 的字符串的起始点的搜索字符串。它是一个非捕获匹配，即不能捕获供以后使用的匹配。例如，'Windows (?!95\|98\|NT\|2000)' 匹配"Windows 3.1"中的 "Windows"，但不匹配"Windows 2000"中的"Windows"。预测先行不占用字符，即发生匹配后，下一匹配的搜索紧随上一匹配之后，而不是在组成预测先行的字符后。 |
| *x*\|*y*      | 匹配 *x* 或 *y*。例如，'z\|food' 匹配"z"或"food"。'(z\|f)ood' 匹配"zood"或"food"。 |
| [*xyz*]       | 字符集。匹配包含的任一字符。例如，"[abc]"匹配"plain"中的"a"。 |
| [^*xyz*]      | 反向字符集。匹配未包含的任何字符。例如，"[^abc]"匹配"plain"中"p"，"l"，"i"，"n"。 |
| [*a-z*]       | 字符范围。匹配指定范围内的任何字符。例如，"[a-z]"匹配"a"到"z"范围内的任何小写字母。 |
| [^*a-z*]      | 反向范围字符。匹配不在指定的范围内的任何字符。例如，"[^a-z]"匹配任何不在"a"到"z"范围内的任何字符。 |
| \b            | 匹配一个字边界，即字与空格间的位置。例如，"er\b"匹配"never"中的"er"，但不匹配"verb"中的"er"。 |
| \B            | 非字边界匹配。"er\B"匹配"verb"中的"er"，但不匹配"never"中的"er"。 |
| \c*x*         | 匹配 *x* 指示的控制字符。例如，\cM 匹配 Control-M 或回车符。*x* 的值必须在 A-Z 或 a-z 之间。如果不是这样，则假定 c 就是"c"字符本身。 |
| \d            | 数字字符匹配。等效于 [0-9]。                                 |
| \D            | 非数字字符匹配。等效于 [^0-9]。                              |
| \f            | 换页符匹配。等效于 \x0c 和 \cL。                             |
| \n            | 换行符匹配。等效于 \x0a 和 \cJ。                             |
| \r            | 匹配一个回车符。等效于 \x0d 和 \cM。                         |
| \s            | 匹配任何空白字符，包括空格、制表符、换页符等。与 [ \f\n\r\t\v] 等效。 |
| \S            | 匹配任何非空白字符。与 [^ \f\n\r\t\v] 等效。                 |
| \t            | 制表符匹配。与 \x09 和 \cI 等效。                            |
| \v            | 垂直制表符匹配。与 \x0b 和 \cK 等效。                        |
| \w            | 匹配任何字类字符，包括下划线。与"[A-Za-z0-9_]"等效。         |
| \W            | 与任何非单词字符匹配。与"[^A-Za-z0-9_]"等效。                |
| \x*n*         | 匹配 *n*，此处的 *n* 是一个十六进制转义码。十六进制转义码必须正好是两位数长。例如，"\x41"匹配"A"。"\x041"与"\x04"&"1"等效。允许在正则表达式中使用 ASCII 代码。 |
| \num          | 匹配 *num*，此处的 *num* 是一个正整数。到捕获匹配的反向引用。例如，"(.)\1"匹配两个连续的相同字符。 |
| \n            | 标识一个八进制转义码或反向引用。如果 *n* 前面至少有 *n* 个捕获子表达式，那么 *n* 是反向引用。否则，如果 *n* 是八进制数 (0-7)，那么 *n* 是八进制转义码。 |
| \nm           | 标识一个八进制转义码或反向引用。如果 *nm* 前面至少有 *nm* 个捕获子表达式，那么 *nm* 是反向引用。如果 *nm* 前面至少有 *n* 个捕获，则 *n* 是反向引用，后面跟有字符 *m*。如果两种前面的情况都不存在，则 *nm* 匹配八进制值 *nm*，其中 *n* 和 *m* 是八进制数字 (0-7)。 |
| \nml          | 当 *n* 是八进制数 (0-3)，*m* 和 *l* 是八进制数 (0-7) 时，匹配八进制转义码 *nml*。 |
| \u*n*         | 匹配 *n*，其中 *n* 是以四位十六进制数表示的 Unicode 字符。例如，\u00A9 匹配版权符号 (©)。 |



Mattcher：

1. find()

   `find()`表示从头开始匹配，找到一个匹配的子序列后，继续匹配下一个子序列，直到不匹配为止，最后返回`false`。

   `matches()`和`lookingAt()`也是从头开始匹配，`matches()`要求整个序列都匹配，而`lookingAt()`则不要求，类似于前缀匹配。

2. start和end

   `start()和`end()`返回匹配的子序列的开始和结束索引。

3. replaceAll和replaceFirst

   `replaceAll`表示替换所有，而`replaceFirst`替换第一个匹配的子序列。

4. Group

   在表达式`((A)(B(C)))`，有四个这样的组：

   - `((A)(B(C)))`
   - `(A)`
   - `(B(c))`
   - `(C)`

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
 
public class RegexMatches
{
    public static void main( String args[] ){
 
      // 按指定模式在字符串查找
      String line = "This order was placed for QT3000! OK?";
      String pattern = "(\\D*)(\\d+)(.*)";
 
      // 创建 Pattern 对象
      Pattern r = Pattern.compile(pattern);
 
      // 现在创建 matcher 对象
      Matcher m = r.matcher(line);
      if (m.find( )) {
         System.out.println("Found value: " + m.group(0) );
         System.out.println("Found value: " + m.group(1) );
         System.out.println("Found value: " + m.group(2) );
         System.out.println("Found value: " + m.group(3) ); 
      } else {
         System.out.println("NO MATCH");
      }
   }
}

Found value: This order was placed for QT3000! OK?
Found value: This order was placed for QT
Found value: 3000
Found value: ! OK?
```

`group(0)`等价于`group()`，通常如果正则表达式没有分组，可以用`group()`返回匹配的内容。

参考：https://www.runoob.com/java/java-regular-expressions.html

































