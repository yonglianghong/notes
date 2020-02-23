做数据处理时，都会经历关于String的各种转换...

一般转换方式：
```scala
def parseDouble(s: String): Option[Double] = try { Some(s.toDouble) } catch { case _ => None }
```

隐式转换方式：
```scala
/* 隐式转换类型类*/
case class ParseOp[T](op: String => T)
implicit val popDouble = ParseOp[Double](_.toDouble)
implicit val popInt = ParseOp[Int](_.toInt)
implicit val popLong = ParseOp[Long](_.toLong)
implicit val popFloat = ParseOp[Float](_.toFloat)
implicit val popTimestamp = ParseOp[Timestamp](Timestamp.valueOf(_))

def parse[T: ParseOp](s: String): Option[T] = try { Some(implicitly[ParseOp[T]].op(s)) }  catch {case _ => None}

val d1 = parse[Double]("12.26") //Some
val l1 = parse[Long]("12.26") //None
val t1 = parse[Timestamp]("2018-06-21 11:52:00") //Some
val t2 = parse[Timestamp]("0000-00-00 00:00:00") //None
```

对比以前转化时间戳时：
```java
//check时间格式是否合法
def checkTimeFormat(t:String): String ={
  var isValid = true
  var result = "1970-01-01 00:00:00"
  val format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
  try{
    format.setLenient(false)
    format.parse(t)
  }catch {
    case e:Exception =>  {
      isValid = false
    }
  }
  if(isValid)
    result = t
  result
}

//然后再用Timestamp自带的方法
Timestamp.valueOf(...)
```

参考：
- [Scala: How to convert a String to an Int (Integer)](https://alvinalexander.com/scala/how-cast-string-to-int-in-scala-string-int-conversion)
- [[Scala基础]--类型转换(String to Double 、Long、Float和Int)](https://blog.csdn.net/high2011/article/details/78271324)