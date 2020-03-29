##### 1. Object

- `protected native Object clone() throws CloneNotSupportedException;`
	- x.clone() != x，x.clone().getClass() == x.getClass()，x.clone().equals(x)
	- `Cloneable`接口，没有任何方法，实现`Cloneable`接口表示可以调用`Object.clone()`方法，否则会抛出`CloneNotSupportedException`，通常需要将clone()方法重写为public的
	- 数组都被认为实现了`Cloneable`接口，但是注意数组里面的比如引用，如果复制，是“浅拷贝”，而不是“深拷贝”
	- 要实现深拷贝，可以手动对引用clone，重新赋值，或者通过序列化与反序列化的方式
	- 参考：https://www.cnblogs.com/nickhan/p/8569329.html
- getClass() 返回对象的运行时类
- `public native int hashCode()`;
	- 同一对象上多次调用返回相同的值
	- 如果根据equals方法两个对象相等，则hashcode值相同
	- 如果根据equlas方法两个对象不相等，则hashcode值不相同
- equals() 比较的是引用
- toString() 返回`getClass().getName() + "@" + Integer.toHexString(hashCode());`
- wait&notify 都是final方法
	- wait()
	- wait(long)
	- wait(long, int)
	- notify()
	- notifyAll()



##### 2. Random
- An instance of this class is used to generate a stream of pseudorandom numbers.（生成伪随机数流）

- 使用相同种子创建的Random实例，产生的随机数序列是一样的

- `java.util.Random`是线程安全的，多线程中可以考虑使用`ThreadLocalRandom`

- `Random`值可预测，因此可以用`SecureRandom`获取加密安全的伪随机数生成器

- 使用示例：

  ```java
  // java.lang.Math.random()返回[0.0,1.0)区间内的double值
  double v = Math.random()
  
  // 不指定种子，默认使用当前系统实际的纳秒数作为种子
  Random r1 = new Random();
  // 指定种子
  Random r2 =new Random(47);
  
  r.nextInt();
  r.nextInt(10);// [0,10)
  r.nextDouble();
  r.nextLong();
  ```



##### 3. Enum

`java.lang.Enum`是所有枚举类的基类，通过`enum`定义的枚举类都是继承自`java.lang.Enum`。主要方法如下：
- name() 返回enum实例名字
- toString() 同name()
- ordinal() 定义enum实例的序号，从0开始
- valueOf(Class, String) 通过名字获取enum实例

定义`enum`示例，可添加额外描述和覆写`toString()`方法，可以使用静态导入`import static`在`switch`中可使用`enum`：

```java
// 定义enum
public enum Color {
    RED,
    GREEN,
    YELLOW;

    public static void main(String[] args) {
        for(Color c : Color.values()){
            System.out.println(c.ordinal()+ ": "+c.name()); // 序号从0开始
        }
    }
}

// 添加额外描述，覆写toString()方法
public enum Color {
    RED("stop"),
    GREEN("pass"),
    YELLOW("slow");

    public String toString(){
        return action;
    }

    private String action;
    private Color(String action){
        this.action = action;
    }
  
    public static void main(String[] args) {
        for(Color c : Color.values()){
            System.out.println(c);
        }
    }
}

// 静态导入，在switch中使用
import static xxx.Color.*;

public class EnumSwitch {

    public static void main(String[] args) {
        Color c = RED;
        switch(c){
            case RED:
                System.out.println("Red");
                break;
            case GREEN:
                System.out.println("Green");
                break;
            case YELLOW:
                System.out.println("Yellow");
                break;
            default:
                throw new RuntimeException("unknown color");
        }
    }
}
```

可以使用接口实现嵌套`enum`：

```java
public interface Food {
    enum Appetizer implements Food {
        SALAD, SOUP, SPRING_ROLLS;
    }
    
    enum MainCourse implements Food {
        LASAGNE, BURRITO, PAD_THAI,
        LENTILS, HUMMOUS, VINDALOO;
    }
    
    enum Dessert implements Food {
        TIRAMISU, GELATO, BLACK_FOREST_CAKE,
        FRUIT, CREME_CARAMEL;
    }
    
    enum Coffee implements Food {
        BLACK_COFFE, DECAF_COFFEE, ESPRESSO,
        LATTE, CAPPUCCINO, TEA, HERB_TEA;
    }
}
```

`EnumSet`所有元素都必须是枚举值，元素保持枚举定义时的顺序，内部以位的形式存储，比如调用`containsAll()`和`retainAll()`方法时，如果参数也是`EnumSet`，则执行效率非常高。

`EnumMap`的key必须是Enum 类型，而value可以是任意类型，其键值对按照枚举类定义的顺序有序。