### tuple & case class & 模式匹配

##### tuple的便利

- 函数可以通过tuple返回多个值
- tuple可以存储在容器类中，代替java bean
- 可以一次为多个变量赋值

```scala
val (one, two) = (1, 2)     
one //res0: Int = 1 
two //res1: Int = 2         
def sellerAndItemId(orderId: Int): (Int, Int) =
   orderId match {  
    case 0 => (1, 2)    
 }          
val (sellerId, itemId) = sellerAndItemId(0)
sellerId // sellerId: Int = 1
itemId // itemId: Int = 2       
val sellerItem = sellerAndItemId(0)
sellerItem._1 //res4: Int = 1
sellerItem._2 //res5: Int = 2
```

##### 用模式匹配增加tuple的可读性

```scala
val sampleList = List((1, 2, 3), (4, 5, 6), (7, 8, 9))
sampleList.map(x => s"${x._1}_${x._2}_${x._3}")
//res0: List[String] = List(1_2_3, 4_5_6, 7_8_9)
sampleList.map {    
  case (orderId, shopId, itemId) =>
    s"${orderId}_${shopId}_$itemId"
}   
//res1: List[String] = List(1_2_3, 4_5_6, 7_8_9)
```

> 上下两个map做了同样的事情，但下一个map为tuple中的三个值都给了名字，增加了代码的可读性。

##### match vs switch

- match是表达式，有返回值
- match不需要"break"
- 如果没有满足要求的case，match会抛出异常
- match可以匹配任何东西，但switch只能匹配数字或字符串常量

```scala
//case如果是常量,就在值相等时匹配.
//如果是变量,就匹配任何值.
def describe(x: Any) = x match {
   case 5 => "five" 
   case true => "truth" 
   case "hello" => "hi!"    
   case Nil => "the empty list"
   case somethingElse => "something else " + somethingElse  
}   
```

case class、tuple以及列表都可以在匹配的同时捕获内部的内容

```scala
case class Sample(a:String,b:String,c:String,d:String,e:String)
def showContent(x: Any) =
 x match {      
  case Sample(a,b,c,d,e) => 
  s"Sample $a.$b.$c.$d.$e"  
  case (a,b,c,d,e) =>   
  s"tuple $a,$b,$c,$d,$e"   
  case head::second::rest =>    
  s"list head:$head second:$second rest:$rest"
}
```

##### case class

- 模式匹配过程中调用了类的unapply方法
- case class还是普通的class，但是自动为你实现了apply,unapply,toString方法
- 其实tuple就是泛型的case class