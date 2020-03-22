### 惰性求值 & call by name

##### 惰性求值

> 这里的惰性求值是指延迟求值，一方面可以提升性能，另一方面可以构造一个无限数据类型？

```scala
import scala.io.Source.fromFile
val iter: Iterator[String] = fromFile("sampleFile").getLines()
```

文件迭代器就用到了惰性求值。用户可以完全像操作内存中的数据一样操作文件，然后文件只有一小部分在内存里。

##### 用lazy关键字指定惰性求值

```scala
lazy val firstLazy = {
  println("first lazy")
  1
}
lazy val secondLazy = {
  println("second lazy")
  2
} 
def add(a:Int,b:Int) = {
  a+b
}
```

```scala
scala> add(secondLazy,firstLazy)
second lazy
first lazy
res0: Int = 3
```

secondLazy先于firstLazy输出

##### call by name就是函数参数的惰性求值

```scala
def firstLazy = {
  println("first lazy")
  1
}
def secondLazy = {
  println("second lazy")
  2
}
def chooseOne(flag: Boolean, a: Int, b: Int) = {
  if (flag) a else b
}
// 注意a,b与上面定义不同
def chooseOneLazy(flag: Boolean, a: => Int, b: => Int) = {
  if (flag) a else b
}
```

```scala
chooseOne(flag = true, secondLazy, firstLazy)
//second lazy
//first lazy
//res0: Int = 2
chooseOneLazy(flag = true, secondLazy, firstLazy)
//second lazy
//res1: Int = 2
```

##### 一个例子，假设你要建立一个本地缓存

```scala
//需要查询mysql等,可能来自于一个第三方jar包
def itemIdToShopId: Int => Int 

var cache = Map.empty[Int, Int]
def cachedItemIdToShopId(itemId: Int):Int = {
  cache.get(itemId) match {
    case Some(shopId) => shopId
    case None =>
      val shopId = itemIdToShopId(itemId)
      cache += itemId -> shopId
      shopId
  }
}
```

- 逻辑没什么问题，但测试的时候不方便连mysql怎么办?
- 如果第三方jar包发生了改变，cachedItemIdToShopId也要发生改变。

```scala
//用你的本地mock来测试程序
def mockItemIdToSHopId: Int => Int

def cachedItemIdToShopId(itemId: Int): Int ={  
  cache.get(itemId) match { 
    case Some(shopId) => shopId
   case None => 
      val shopId = mockItemIdToSHopId(itemId)
      cache += itemId -> shopId
     shopId 
  } 
}   
```

- 在测试的时候用mock，提交前要换成线上的，反复测试的话要反复改动，非常令人沮丧
- 手工操作容易忙中出错

```scala
//将远程请求的结果作为函数的一个参数
def cachedItemIdToShopId(itemId: Int, remoteShopId: Int): Int = {   
  cache.get(itemId) match { 
    case Some(shopId) => shopId 
    case None =>    
     val shopId = remoteShopId  
     cache += itemId -> shopId  
      shopId
  } 
}
//调用这个函数
cachedItemIdToShopId(itemId,itemIdToShopId(itemId))
```

- 函数对mysql的依赖没有了
- 不需要在测试和提交时切换代码
- 貌似引入了新问题?

没错，cache根本没有起应有的作用，函数每次执行的时候都调用了itemIdToShopId从远程取数据

```scala
//改成call by name就没有这个问题啦
def cachedItemIdToShopId(itemId: Int, remoteShopId: =>Int): Int = { 
  cache.get(itemId) match { 
    case Some(shopId) => shopId 
    case None =>    
     val shopId = remoteShopId  
     cache += itemId -> shopId  
      shopId
  } 
}
//调用这个函数
cachedItemIdToShopId(itemId,itemIdToShopId(itemId))
```

- 函数对mysql的依赖没有了
- 不需要在测试和提交时切换代码
- 只在需要的时候查询远程库