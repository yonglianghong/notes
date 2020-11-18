###Hive Data Type With Examples

Hive的数据类型用来说明Hive表的cloumn/field的类型，可以大致分为两类：

- Primitive Data Types
- Complex Data Types

Primitive Data Types可以进一步分为四类：

- Numeric Types
- String Types
- Date/Time Types
- Miscellaneous Types

这些数据类型和占用空间大小与Java/SQL primitive相似。

##### 1. Hive数据类型

<u>**Primitive Data Types**</u>

***Numeric Data Types***

整型包括tinyint、smallint、int和bigint，等价于Java的byte、short、int和long primitive types；

浮点型包括float、double和decimal，等价于Java的float、double，SQL的decimal类型。

decimal(5,2)表示一共有5位，其中2位是小数。下面的表格是所有数值类型的范围及示例：

|   Type   |               Size                |              Range              |    Examples    |
| :------: | :-------------------------------: | :-----------------------------: | :------------: |
| tinyint  |       1 byte signed integer       |           -128 to 127           |      100       |
| smallint |       2 byte signed integer       |        -32,768 to 32,767        |    100,1000    |
|   int    |       4 byte signed integer       | -2,147,483,648 to 2,147,483,647 | 100,1000.50000 |
|  bigint  |       8 byte signed integer       |     -9.2*10^18 to 9.2*10^18     | 100,1000*10^10 |
|  float   |   4 byte single precision float   |      1.4*e^-45 to 3.4*e^38      |    1500.00     |
|  double  |   8 byte double precision float   |    4.94*e^-324 to 1.79*e^308    |   750000.00    |
| decimal  | 17 bytes precision upto 38 digits |       -10^38+1 to 10^38-1       |  decimal(5,2)  |

在Hive里面，整型数值默认当做int处理，除非超出了int值的范围。如果需要当做tinyint or smallint or bigint处理，需要在值后面分别添加后缀Y,S or L。

例如：100Y -> TINYINT, 100S -> SMALLINT, 100L -> BIGINT

**String Data Types**

从0.14版本开始，Hive支持3种字符类型，见如下表格：

|  Type   |                         Description                          | Examples |
| :-----: | :----------------------------------------------------------: | :------: |
| string  | sequence of characters.either single quotes(') or double quotes ('') can be used to enclose characters | 'string' |
| varchar | max length is specified in braces.similar to SQL's varchar.max length allowed is 65355 bytes |  'str'   |
|  char   | similar to SQL's char with fixed length. i.e values shorter than the specified length are padded with spaces |   's'    |

*char vs varchar*

- char是固定长度的，不足用空格补齐；
- varchar是变长的，但是需要指定最大长度（例如：name VARCHAR(64)）。若长度不足，不会用空格补齐，不占用剩余空间；
- char的最大长度是255，varchar的最大长度可到65335字节；
- varchar会进行空间/存储（space/storage）优化，但是char不会；
- 如果一个字符长度超过varchar指定的长度，会被自动截取。

**Date/Time Data Types**

Hive以传统的Unix时间戳格式提供日期和时间戳数据类型。

DATE值格式为YYYY-MM-DD，例如：DATE '2018-08-01'。Date值的范围是0000-01-01 to 9999-12-31。

TIMESTAMP 使用的格式为 yyyy-mm-dd hh:mm:ss[.f...]。

我们可以把String，TimeStamp值转换成Date类型：

|        Cast Type        |                            Result                            |
| :---------------------: | :----------------------------------------------------------: |
|   cast(date as date)    |                       same date value                        |
|  cast(date as string)   |          date is formatted in the form 'YYYY-MM-DD'          |
| cast(date as timestamp) | midnight of the year/month/day of the date value is returned as timestamp |
|  cast(string as date)   | if the string is in the form 'YYYY-MM-DD',then a date value corresponding to that is returned.if the string value does not match this format,then NULL is returned. |
| cast(timestamp as date) | the year/month/day of the timestamp is returned as a date value |

**Miscellaneous Types**

Hive还提供两种primitive data types，BOOLEAN和BINARY。和Java的Boolean相似，BOOLEAN只存储true或者false。

BINARY是字节数组，和很多关系型数据库的VARBINARY相似。BINARY存储在记录中，不想BLOB单独存储，可以在BINARY中包含任意字节序列，会原样存储，不会被解析成数字或者字符。

<u>**Complex Data Types**</u>

Hive还支持一些关系型数据库不支持的复合数据类型。

复合数据类型由primitive data types和other complex data types构成，如下：

- ARRAY - 相同类型的元素构成的序列，从0开始索引，与Java中的array类似。例如：array('siva','bala','praveen')；第二个元素是array[1]

- MAP - key-value对的集合，field由key获得(比如，['key'])。例如：'first' -> 'bala', 'last' -> 'PG'，bala = map['first']

- STRUCT - 类似于C语言的strcut，可以通过(.)获取元素值。例如：{a:Int; b:String}，可以用c.a获取值

- UNIONTYPE - 类似于C语言的unions，一个UNIONTYPE可以有指定的data types的任意一种

  例如：声明一列为Union Type

  ```sql
  CREATE TABLE test(col1 UNIONTYPE<INT, DOUBLE, ARRAY<VARCHAR>, STRUCT<a:INT,b:CHAR>>);
  ```

  从col1中获取值如下：

  ```sql
  SELECT col1 FROM test;
  
  {0:1}                       // Matching INT types
  {1:10.0}                    // Matching DOUBLE types
  {2:["hello","world"]}       // Matching ARRAY with VARCHAR type
  {3:{"a":50,"b":"Good"}}     // Matching STRUCT with INT & CHAR types   
  {2:["hadoop","tutorial"]}   // Again 2: represent it is Array type
  {3:{"a":100,"b":".info"}}  //3: indicate the fourth element type in Union
  {0:150}
  {1:100.0}
  ```

文本格式存储里，默认的集合数据类型的分隔符：

|  Delimiter  | Code |               Description               |
| :---------: | :--: | :-------------------------------------: |
|     \n      |  \n  |         Record or row delimiter         |
| ^A (Ctrl+A) | \001 |             Field delimiter             |
| ^B (Ctrl+B) | \002 | Element delimiter in ARRAYs and STRUCTs |
| ^C (Ctrl+C) | \003 |    Delimits key/value pairs in a MAP    |

##### 2. 类型转换

**Implicit Conversion Between Primitive Data Types**

- Primitive Type
  - Number
    - DOUBLE
      - FLOAT
        - BIGINT
          - INT
            - SMALLINT
              - TINYINT
      - DECIMAL (Can be converted to String, varchar only)
      - STRING (Can be converted to Varchar,Double,Decimal)
      - VARCHAR (Can be converted to String,Double,Decimal)
        - DATE (Converted to String,Varchar)
        - TIMESTAMP (same as date)
  - BOOLEAN
  - BINARY

在上面的层次图，子级类型可以被隐式转换为父级类型或者其他祖先类型。比如，TINYINT可以被任意转换为其他数值类型，但是BIGINT只能被转换为FLOAT或者DOUBLE。

BOOLEAN和BINARY不能被隐式转换为任意其他类型。

**Explicit Conversion**

可以用cast进行强制转换，比如：

CAST('500' as INT) 会将string '500' 转换成int型的500，如果转型不成功，将会返回NULL（CAST('Hello' as Int)）。

##### 3. 使用示例

所有数据类型使用示例：

```sql
CREATE TABLE user (
     name      STRING,
     id        BIGINT,
     isFTE     BOOLEAN,
     role      VARCHAR(64),
     salary    DECIMAL(8,2),
     phones    ARRAY<INT>,
     deductions MAP<CHAR, FLOAT>,
     address   STRUCT<street:STRING, city:STRING, state:STRING, zip:INT>,
     others    UNIONTYPE<FLOAT,BOOLEAN,STRING>,
     misc      BINARY
     )
ROW FORMAT DELIMITED 
    FIELDS TERMINATED BY '\001'
    COLLECTION ITEMS TERMINATED BY '\002'
    MAP KEYS TERMINATED BY '\003'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;
```

##### 4. 注意

1) String可以隐式转换为Double；

2) String vs Varchar vs Char

a. String：可以'c '或者"c "，空格有效；

b. Varchar：定长，多余截取；

c. Char 定长，不足补空；

3) TimeStamp

a. 纳秒？

b. TimeStamp的值可以是整数（秒），也可以是浮点数（纳秒，精确到小数点后9位），也可以是字符串，格式为YYYY-MM_DD hh:mm:ss.fffffffff

c. 无时区概念，可用to_utc_timestamp/from_utc_timestamp

d. table level?

4) Date Type

a. 0000-01-01 to 9999-12-31

b. Date <-> TimeStamp/String(YYYY-MM-DD)/Date

c. Interval?

5) Decimal

a. 与各种numeric类型互转？

b. 同时支持科学计数法与非科学计数法

c. 建表时如果没有指定长度，则默认为Decimal(10.0)

d. cast(18446744035BD as Decimal(38,0))

e. Decimal其实可以表示-10^380~10^380，但Hive规定最多38位

f. Decimal可以与任意primitive类型互转，比如boolean？

g. Decimal同样支持各种数学计算

6) Union Types，join/where/group by => fail

7) 分隔符是只在文本格式里有效还是所有文件格式？ 

8) 传统关系型数据库是写时模式（schema on write），即在写入时对模式进行检查，而Hive是读时模式（schema on read），是在读取数据时检查模式，并尽力修复，如果修复失败则返回null

9) 文本格式（TextFile）里，如果为null，则为/N

##### 5. 参考

翻译原文：[Hive Data Types With Examples](http://hadooptutorial.info/hive-data-types-examples/)

官方文档：[Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-AllowedImplicitConversions)

