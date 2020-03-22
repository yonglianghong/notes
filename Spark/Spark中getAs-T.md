---
title: 'Spark中getAs[T]'
date: 2018-06-26 17:31:08
tags: [spark基础]
---

Spark中getAs[T]回忆录
<!-- more -->

### 0.问题起源
spark中，如果get的值为null，则返回？

查看源码：
```scala
/**
 * Returns the value of a given fieldName.
 * For primitive types if value is null it returns 'zero value' specific for primitive
 * ie. 0 for Int - use isNullAt to ensure that value is not null
 *
 * @throws UnsupportedOperationException when schema is not defined.
 * @throws IllegalArgumentException when fieldName do not exist.
 * @throws ClassCastException when data type does not match.
 */
def getAs[T](fieldName: String): T = getAs[T](fieldIndex(fieldName))
```
对于primitive types，如果是null，则返回zreo value。参考[Java DataTypes](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)：

| Data Type|  Default Value (for fields) |
| :--------: | :--------:|
| byte    |   0 |
| short    |   0 |
| int    |   0 |
| long    |   0L |
| float    |   0.0f |
| double    |   0.0d |
| char    |   '\u0000' |
| String (or any object)    |   null |
| boolean    |   false |

### 1.实践getAs[T]
```scala
val conf = new SparkConf()

val spark = SparkSession.builder().master("local[10]").config(conf).enableHiveSupport().getOrCreate()
import spark.sql
import spark.implicits._

val seq2df = Seq((1,"jack"),(2,"ming"),(3,null)).toDF("id","name")
val seq2df2 = Seq((1,"America")).toDF("id","region")

seq2df.createOrReplaceTempView("user")
seq2df2.createOrReplaceTempView("country")

val leftjoin = sql("select a.id,a.name,c.region from user a left join country c on a.id = c.id")
leftjoin.collect().map(x => println(x))
println("##############")
seq2df.rdd.map(row=>row.getAs[String]("name")).collect().map(x => println(x))

//输出信息
[1,jack,America]
[2,ming,null]
[3,null,null]
##############
jack
ming
null
```
由此可见，对于String，如果为null，则get到的值为null字符串，暂未测试插入mysql。

注：实践中有遇到各种情况，建议数据处理时对于null都手动特殊处理，以保证数据的准确和统一。

### 2.一般getAs[T]
```scala
val product = row.isNullAt(0) match {
  case true => ""
  case false => row.getAs[String]("product")
}

if (row.isNullAt(0)) "-" else row.getAs[String]("product")
```

### 3.通用getAs[T]
```scala
import com.alibaba.fastjson.{JSONArray, JSONObject}
import org.apache.spark.sql.Row

//将row中的数据取出，放入JSONObject
def getFromRow(row:Row): Unit ={
  val fields = row.schema.fields
  val obj = new JSONObject()
  for (field <- fields) {
    field.dataType.typeName match {
      case "integer" =>
        if (row.getAs[Int](field.name) != null) {
          obj.put(field.name, row.getAs[Int](field.name))
        }
      case "array" =>
        if (row.getSeq(row.fieldIndex(field.name)) != null) {
          val jsonArray = new JSONArray()
          val valueList = row.getSeq(row.fieldIndex(field.name)).mkString(",")
            .replace("]", "").replace("[", "").replace("\"", "").split(",").
            filter(_.indexOf("_TDERR") < 0)
          for (vv <- valueList) {
            jsonArray.add(vv)
          }
          obj.put(field.name, jsonArray)
        }
      case "double" =>
        if (row.getAs[Double](field.name) != null) {
          obj.put(field.name, row.getAs[Double](field.name))
        }
      case "long" =>
        if (row.getAs[Long](field.name) != null) {
          obj.put(field.name, row.getAs[Long](field.name))
        }
      case _ =>
        if (row.getAs[String](field.name) != null && row.getAs[String](field.name).indexOf("_TDERR") < 0) {
          if (!field.name.equalsIgnoreCase("mac")) {
            obj.put(field.name, checkValue(row.getAs[String](field.name)))
          }
        }
    }
  }
}

def checkValue(str: String): String = {
  if (str == null || str == "" || str.length <= 0 || str.replaceAll(" ", "").length <= 0 || str == "None") {
    null
  } else {
    str.replaceAll(" ", "")
  }
}
```