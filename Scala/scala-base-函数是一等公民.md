### 函数是一等公民

##### 一个需求

- 假设我们需要检查许多的数字是否符合某一范围
- 范围存储在外部系统中,并且可能随时更改
- 数字范围像这样存储着">= 3,< 7”

##### 一个Java版本

```java
List<String> params = new LinkedList<>();
List<Integer> nums = new LinkedList<>();
List<String> marks = new LinkedList<>();

public JavaRangeMatcher(List<String> params) {
    this.params = params;
    for (String param : params) {
        String[] markNum = param.split(" ");
        marks.add(markNum[0]);
        nums.add(Integer.parseInt(markNum[1]));
    }
}

public boolean check(int input) {
    for (int i = 0; i < marks.size(); i++) {
        int num = nums.get(i);
        String mark = marks.get(i);
        if (mark.equals(">") && input <= num) return false;
        if (mark.equals(">=") && input < num) return false;
        if (mark.equals("<") && input >= num) return false;
        if (mark.equals("<=") && input > num) return false;
    }
    return true;
}

List<String> paramsList = new LinkedList<String>() {{
    add(“>= 3”);
    add(“< 7”);
}};
JavaRangeMatcher matcher = new JavaRangeMatcher(paramsList);
int[] inputs = new int[]{1, 3, 5, 7, 9};
for (int input : inputs) {
    System.out.println(matcher.check(input));
}
//给自己有限的时间,想想又没有性能优化的余地
//我们一起来跑跑看
```

##### 一个scala版本

```scala
def exprToInt(expr: String): Int => Boolean = {
  val Array(mark, num, _*) = expr.split(" ")
  val numInt = num.toInt
  mark match {
    case "<" => numInt.>
    case ">" => numInt.<
    case ">=" => numInt.<=
    case "<=" => numInt.>=
  } //返回函数的函数
}

case class RangeMatcher(range: Seq[String]) {
  val rangeFunc: Seq[(Int) => Boolean] = range.map(exprToInt)

  def check(input: Int) = rangeFunc.forall(_(input))
}

def main(args: Array[String]) {
  val requirements = Seq(">= 3", "< 7")
  val rangeMatcher = RangeMatcher(requirements)
  val results = Seq(1, 3, 5, 7, 9).map(rangeMatcher.check)
  println(results.mkString(","))
  //false,true,true,false,false
}
```

##### 性能测试

```scala
//java版本
public static void main(String[] args) {
    List<String> paramsList = new LinkedList<String>() {{
        add(">= 3");
        add("< 7");
    }};
    JavaRangeMatcher matcher = new JavaRangeMatcher(paramsList);
    Random random = new Random();
    long timeBegin = System.currentTimeMillis();
    for (int i = 0; i < 100000000; i++) {
        int input = random.nextInt() % 10;
        matcher.check(input);
    }
    long timeEnd = System.currentTimeMillis();
    System.out.println("java 消耗时间: " + (timeEnd - timeBegin) + " 毫秒");
    //java 消耗时间: 3263 毫秒
}
```

```scala
//scala版本
def main(args: Array[String]) {
  val requirements = Seq(">= 3", "< 7")
  val rangeMatcher = RangeMatcher(requirements)
  val timeBegin = System.currentTimeMillis()
  0 until 100000000 foreach {
    case _ =>
      rangeMatcher.check(Random.nextInt(10))
  }
  val timeEnd = System.currentTimeMillis()
  println(s"scala 消耗时间 ${timeEnd - timeBegin} 毫秒")
  //scala 消耗时间 2617 毫秒
}
```



