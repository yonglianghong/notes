##### 1. 类结构图

<div style="text-align: center;"> <img src="https://raw.githubusercontent.com/yonglianghong/my-drive/md/img/Java.Collections.Framework.jpg"/> </div>
- 说明：
  - Produces：可生成
  - 虚线：
    - 箭头：实现接口
    - 框：接口/抽象类
  - 实线：
    - 箭头：继承
    - 框：类

Java容器，容器是一种保存对象的方式。

尖括号，类型参数，编译期就可以防止将错误类型的对象放置到容器中；get时，因为知道保存的是什么类型，会替你执行转型。

Collection接口概括了序列的概念，一种存放一组对象的方式。

Collection的构造器可以接受另一个Collection，用它来将自身初始化。



Collections: A collection represents a group of objects, known as its elements. 一组对象的集合，即元素
- some collections allow duplicate elements and others do not 有的允许重复，有的不允许
- some are ordered and others unordered 有的有序，有的无序
- which typically implement Collection indirectly through one of its subinterfaces. 通常通过子接口间接实现Collection, should provide two "standard" constructors: a void (no arguments) constructor, which creates an empty collection, and a constructor with a single argument of type Collection. 应该提供无参构造器和只含一个参数的构造器
- ineligible element(such as null elements)，对于集合中的元素，有的严格，有的不严格
- It is up to each collection to determine its own synchronization policy. 每一种集合实现由自己决定同步策略
- Many methods in Collections Framework interfaces are defined in terms of the equals/hashcode method.许多方法的实现会用到元素的equals方法，比如contains(Object o),return ture if contains at least one element such that o==null ? e==null : o.equals(e)，但这不应该成为绝对

- 主要方法：

  ```java
  // the number of elements in this collection
  int size();
  // if this collection contains no elements
  boolean isEmpty();
  // if this collection contains the specified element
  boolean contains(Object o);
  // 添加指定元素
  boolean add(E e);
  // 移除指定元素
  boolean remove(Object o);
  // an Iterator over the elements in this collection
  Iterator<E> iterator();
  ```

  



##### 2. 数组

数组也是一种表示一组对象的方式，不管是普通对象，或者基本类型都可以，特点如下：

- 效率：非常高，随机访问
- 类型：编译期间检查加入或者获取的元素类型
- 基本类型：支持保存基本类型

数组本质保存的是指向对象的引用，唯一的成员变量`length`，表示数组本身的长度。

数组的声明：

```java
// 普通
String[] strs = new String[10];

// 多维
int[][] a = new int[2][3]
int[][] a = {
  {1, 2, 3},
  {4, 5, 6}
}

Random rand = new Random(47);
int[][][] a = new int[rand.nextInt(7)][][]; // 只声明第一维的长度
for(int i = 0; i < a.length; i++){
    a[i] = new int[rand.nextInt(5)][];
    for(int j = 0; j < a[i].length; j++){
        a[i][j] = new int[rand.nextInt(5)];
    }
}
```

数组与泛型：

擦除会移除参数类型的信息，而数组必须知道它们所持有的确切类型，以强制保证类型安全。

`Item[] s = (Item[])new Object[Capacity]`



Arrays：

- Arrays.toString()
- Arrays.deepToString()
- Arrays.fill()：填充整个数组，如果是对象，则填充的是同一个引用



数组复制：System.arraycopy(源数组, 源偏移量, 目标数组, 目标偏移量, 复制长度)

- 注意：复制长度不能越界，即不能超出源数组本身的长度



##### 3. 基础（Infrastructure）

1. Iterators：

   1. Iterable：可迭代

      - Implementing this interface allows an object to be the target of the "for-each loop" statement.（数组也支持）

      - 主要方法：

        ```java
        Iterator<T> iterator();
        default forEach(Consumer<? super T> action);
        ```
   
   2. Iterator：迭代器
   
      - An iterator over a collection. Iterator takes the place of Enumeration in the Java Collections Framework. Iterators differ from enumerations in two ways:（替换Enumeration）
        - Iterators allow the caller to remove elements from the underlying collection during the iteration with well-defined semantics.（允许在迭代的时候删除）
        - Method names have been improved.（名字简化）
   
      - 主要方法：
   
        ```java
        boolean hasNext();
        E next();
        default void remove() { throw new UnsupportedOperationException("remove");}
        default void forEachRemaining(Consumer<? super E> action);
        ```
   
   3. ListIterator：List的迭代器， 支持双向迭代，元素插入、替换，检索索引
   
      - An iterator for lists that allows the programmer to traverse the list in either direction, modify the list during iteration, and obtain the iterator's current position in the list.（List的Iterator，双向迭代，在迭代的过程中修改，持有索引）
   
      - A ListIterator has no current element; its cursor position always lies between the element that would be returned by a call to previous() and the element that would be returned by a call to next().
   
        - cursor positions: ^ Element(0) ^ Element(1) ^ Element(2) ... Element(n-1)
   
      - Note that the remove() and set(Object) methods are not defined in terms of the cursor position; they are defined to operate on the last element returned by a call to next() or previous().
   
      - 主要方法：
   
        ```java
        ListIterator<E> extends Iterator<E>
        
        // 构造器，一开始就指向索引n
        ListIterator(n)
        
        // 正序迭代
        boolean hasNext();
        E next();
        int nextIndex();
        
        // 逆序迭代
        boolean hasPrevious();
        E previous();
        int previousIndex();
        
        void remove();
        void set(E e);
        void add(E e);
        ```
   
   4. 示例：
   
      ```java
      class HelloIterable implements Iterable<Integer>{
          int[
          HelloIterable(int[] a){this.a =
          @Override
          public Iterator iterator() {
              return new Iterator(){
                  private int index=
                  @Override
                  public boolean hasNext() {
                      return index < a.length;
               
                  @Override
                  public Object next() {
                      return a[index++];
                  }
              };
          }
      }
      
      class HelloComsumer implements Consumer{
          @Override
          public void accept(Object o) {
              System.out.println(o);
          }
      }
      
      int[] array = {1, 2, 3, 4, 5};
      HelloIterable helloIterable = new HelloIterable(array);
      for(Integer i: helloIterable){
          System.out.println("i = " + i);
      }
      helloIterable.forEach(new HelloComsumer());
      ```
   
      default表示接口的默认实现（从1.8开始，接口可添加默认实现方法）



2. Ordering
   1. Comparable：定义类对象的排序（自然排序）
      - This interface imposes a total ordering on the objects of each class that implements it. This ordering is referred to as the class's natural ordering, and the class's compareTo method is referred to as its natural comparison method.（实现该接口的每个类的对象进行排序。称之为类的自然排序，compareTo方法称为自然排序方法）
      
      - Lists (and arrays) of objects that implement this interface can be sorted automatically by Collections.sort (and Arrays.sort). Objects that implement this interface can be used as keys in a sorted map or as elements in a sorted set, without the need to specify a comparator.（Lists/arrays里对象实现该接口则可以利用Collections.sort/Arrays.sort进行排序。实现该接口的对象可以作为sorted map/set的键/元素）
      
      - The natural ordering for a class C is said to be consistent with equals if and only if e1.compareTo(e2) == 0 has the same boolean value as e1.equals(e2) for every e1 and e2 of class C. Note that null is not an instance of any class, and e.compareTo(null) should throw a NullPointerException even though e.equals(null) returns false.(自然序与equals保持一致是指e1.compareTo(e2)==0与e1.equals(e2)有相同的结果。注意null，e.compareTo(null)会抛出空指针异常，但是e.equals(null)返回false。)
      
      - It is strongly recommended (though not required) that natural orderings be consistent with equals.（强烈建议equals与natural orderings保持一致）
      
      - 主要方法：
      
        ```java
        int compareTo(T o)
        ```
   
   2. Comparator
      - A comparison function, which imposes a total ordering on some collection of objects. Comparators can be passed to a sort method (such as Collections.sort or Arrays.sort) to allow precise control over the sort order. Comparators can also be used to control the order of certain data structures (such as sorted sets or sorted maps), or to provide an ordering for collections of objects that don't have a natural ordering.（比较函数，可传递给排序函数、用于控制排序sets/maps、无自然排序的对象（即未实现Comparable接口，但可自定义Comparator））
   
      - The ordering imposed by a comparator c on a set of elements S is said to be consistent with equals if and only if c.compare(e1, e2)==0 has the same boolean value as e1.equals(e2) for every e1 and e2 in S.（与equals定义保持一致）
   
      - Unlike Comparable, a comparator may optionally permit comparison of null arguments, while maintaining the requirements for an equivalence relation.（允许对null灵活处理，而comparable抛出空指针异常）
   
      - 主要方法：
   
        ```java
        int compare(T o1, T o2) // 注意如果直接return i-j则可能导致溢出，当i是很大的正整数，j是很大的负整数
        boolean equals(Object obj) // 通常不用实现
        ```
   
      - 常用Comparator：
   
        ```java
        Comparator.reverseOrder();
        Comparator.naturalOrder();
        String.CASE_INSENSITIVE_ORDER;
        ```
   
   3. 示例：
   
      ```java
      class Student {
          String name;
          int age;
          Student(String name, int age){
              this.name = name;
              this.age = age;
          }
          @Override
          public String toString(){
             return "[name="+name+",age="+age+"]";
          }
      }
      
      List<Student> students = new ArrayList<>();
      students.add(new Student("a", 1));
      students.add(new Student("c", 3));
      students.add(new Student("b", 2));
      System.out.println(students);// [[name=a,age=1], [name=c,age=3], [name=b,age=2]]
      Collections.sort(students, new Comparator<Student>() {
          @Override
          public int compare(Student o1, Student o2) {
              return o1.age > o2.age? (o1.age==o2.age? 0:1):-1;
          }
      });
      System.out.println(students);// [[name=a,age=1], [name=b,age=2], [name=c,age=3]]
      ```
   
      



3. Performance
   1. RandomAccess：
      - 支持快速随机访问（generally constant time），*The primary purpose of this interface is to allow generic algorithms to alter their behavior to provide good performance when applied to either random or sequential access lists.*
      - 只是一个单独的接口，里面没有定义任何方法



##### 4. 栈和队列（Stack and Queue）

1. Stack：Last In First Out 后进先出

   - 自定义Stack

     ```java
     // 利用LinkedList
     public class Stack<T> {
       private LinkedList<T> stack = new LinkedList<T>();
       public void push(T v){ stack.addFirst(v); } // 入栈
       public T peek(){ return stack.getFirst(); } // 一瞥
       public T pop(){ return stack.removeFirst(); } //出栈
       public boolean empty(){ return stack.isEmpty(); }
       public String toString(){ return stack.toString(); }
     }
     ```

   - java.util.Stack

     - 继承自Vector

     - 主要方法：

       ```java
       E push(E item) // 入栈
       synchronized E pop() // 出栈
       synchronized E peek() // 一瞥
       boolean empty()
       synchronized int search(Object o) // 检索,discover how far it is from the top
       ```

   - 栈经常用来对表达式求值



2. Queue：First In First Out 先进先出

   1. Queue整体结构

      <div style="text-align: center;"> <img src="https://raw.githubusercontent.com/yonglianghong/my-drive/md/img/Java.Collections.Framework.Queue.jpg"  width = "50%" height = "50%"/>
      </div>

   2. Queue

      - A collection designed for holding elements prior to processing.

      - Each of these methods exists in two forms: one throws an exception if the operation fails, the other returns a special value (either null or false, depending on the operation). (两种形式，当失败时，一种抛出异常，一种返回特殊值，比如null)

        |                          | Throws NoSuchElementException | Returns special value(null) |
        | ------------------------ | ----------------------------- | --------------------------- |
        | Insert（插入队尾）       | add(e)                        | offer(e)                    |
        | Remove（移除并返回队头） | remove()                      | poll()                      |
        | Examine（查看队头）      | element()                     | peek()                      |

   3. Deque(Double ended queue)

      - A linear collection that supports element insertion and removal at both ends.（两端都支持插入和删除）

      - this interface supports capacity-restricted deques as well as those with no fixed size limit.（支持指定容量或者不限容量）

      - Summary of Deque methods

        |                 | Throws exception | Special value | Throws exception | Special value |
        | --------------- | ---------------- | ------------- | ---------------- | ------------- |
        | Insert（插入）  | addFirst(e)      | offerFirst(e) | addLast(e)       | offerLast(e)  |
        | Remove（移除）  | removeFirst()    | pollFirst()   | removeLast()     | pollLast()    |
        | Examine（查看） | getFirst()       | peekFirst()   | getLast()        | peekLast()    |

   4. ArrayDeque&LinkedList

      ```java
      Deque<T> deque = new ArrayDeque<T>();
      Deque<T> deque = new LinkedList<T>();
      ```

   5. PriorityQueue

      - An unbounded priority queue based on a priority heap. The elements of the priority queue are ordered according to their natural ordering, or by a Comparator provided at queue construction time, depending on which constructor is used.（基于堆，按自然序或者指定Comparator排序的队列）

      - The head of this queue is the least element with respect to the specified ordering.The queue retrieval operations poll, remove, peek, and element access the element at the head of the queue.（队列头是最小的元素，poll、remove、peek、element访问队头元素）
      - 时间复杂度：
      	- O(log(n)) time for the enqueuing and dequeuing methods (offer, poll, remove() and add); 
      	- linear time for the remove(Object) and contains(Object) methods; 
      	- constant time for the retrieval methods (peek, element, and size).



##### 5. List

1. List：An ordered collection (also known as a *sequence*). 一种有序collection，即序列。

   - 可以指定位置插入，使用integer index (position in the list) 访问元素

   - 允许重复元素，包括null

   - ListIterator，双向（bidirectional）迭代，允许element 插入和替换（insertion and replacement），以及在指定位置开始迭代

   - 主要方法：

     ```java
     // 长度
     int size();
     // 是否为空
     boolean isEmpty();
     // 包含
     boolean contains(Object o);
     // 添加&替换
     boolean add(E e);
     void add(int index, E e);
     E set(int index, E e);
     // 获取
     E get(int index);
     // 删除
     E remove(int index);
     boolean remove(Object o);
     // 迭代器
     Iterator<E> iterator();
     ListIterator<E> listIterator();
     ListIterator<E> listIterator(int index);
     // 子序列(与原序列关联)
     List<E> subList(int fromIndex, int toIndex);
     ```

     

2. ArrayList：擅长快速访问（本质上是因为连续地址空间）

   1. 主要方法：
      - 添加：add(E e)/add(int index, E element)
      - 删除：remove(int index)/remove(Object o)
      - 获取：get(int index)
      - 设置（替换）：set(int index, E element)

   2. 允许放入null元素
   3. 底层是Object[]（Java泛型是编译器提供的语法糖），get时强制转型
   4. 实现Serializable、RandomAccess、Cloneable接口
   5. 效率：
      - 插入末尾，如果不扩容?，O(1)，插入指定索引，插入后要移动元素，O(n)
      - 删除后要将后面的元素向前移动，O(n)
      - 查询，O(1)



3. LinkedList：擅长快速插入删除（用链表实现的列表）

   1. 主要方法：
      - 添加：add(E e)/add(int index, E element)
      - 删除：remove(int index)/remove(Object o)
      - 获取：get(int index)
      - 设置：set(int index, E element)

   2. 双向循环链表，允许元素为null?
   3. 实现Serializable、Cloneable接口
   4. 查找指定索引处元素，有一个加速动作，比较index与size >> 1（即size/2）来决定从first还是last开始遍历
   5. 效率：
      - 插入末尾，O(1)，插入指定索引，O(n)
      - 删除，需要先查找，O(n)
      - 查找，O(n)



##### 6. Set

1. Set：无重复元素的Collection

   - 允许null（只包含1个）

   - 依赖于equals()方法，判断是否重复，因此对于已经加入set的可变元素需要注意

   - 主要方法：

     ```java
     // 长度
     int size();
     // 是否为空
     boolean isEmpty();
     // 包含
     boolean contains(Object o);
     // 添加
     boolean add(E e);
     // 移除
     boolean remove(Object o);
     // 迭代器
     Iterator<E> iterator();
     ```

2. HashSet：为快速查找而设计的Set，基于HashMap实现

3. LinkedHashSet：继承自HashSet，使用双链表维护插入顺序

4. SortedSet -> TreeSet：元素必须实现Comparable，保持自然顺序，基于TreeMap实现，底层树形结构

   - 主要方法：

     ```java
     Object first();
     Object last();
     // [) 前闭后开
     SortedSet subSet(E from, E to);
     // 从head到to的子集，同样前闭后开
     SortedSet headSet(E to);
     // 从from到tail的子集，同样前闭后开
     SortedSet tailSet(E from)
     ```



##### 7. Map

0. Map整体结构

<div style="text-align: center;"> <img src="https://raw.githubusercontent.com/yonglianghong/my-drive/md/img/Java.Collections.Framework.Map.jpg"  width = "50%" height = "50%"/> <br> <div style="color:orange; border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;padding: 2px;font-style:italic;font-size:80%">Map接口图</div>
</div>

1. Map：
   - An object that maps keys to values. A map cannot contain duplicate keys; each key can map to at most one value.（keys到values的映射，map不能包含相同的keys，一个key最多只对应一个value值。）
   - This interface takes the place of the Dictionary class, which was a totally abstract class rather than an interface.（用来代替抽象类Dictionaray）
   - The Map interface provides three collection views, which allow a map's contents to be viewed as a set of keys, collection of values, or set of key-value mappings.（提供三种视图，set of keys视图，collection of values视图，set of key-value mapping视图）
   - The order of a map is defined as the order in which the iterators on the map's collection views return their elements.（map的order是指迭代的顺序）
   - great care must be exercised if mutable objects are used as map keys.（注意使用可变对象作为key，map不允许自身作为key，但是可以作为value）
   - 实现Map接口，通常得提供两个构造器，一个无参，一个提供另一个map作为参数，实现复制
   - 很多实现都基于了equals()方法（比如，containsKey(Object key){key==null ? k==null : key.equals(k)}），但这不是绝对的，比如可以先调用hashcode()方法，hashcode不一样，肯定不相同。总之，具体的实现应该完全不拘泥于equals()。



2. AbstractMap
   - This class provides a skeletal implementation of the Map interface, to minimize the effort required to implement this interface.（Map的简化实现）
   - To implement an unmodifiable map, the programmer needs only to extend this class and provide an implementation for the entrySet method, which returns a set-view of the map's mappings. (要实现不可修改的map，只需要继承这个类，并实现entrySet方法即可)
   - To implement a modifiable map, the programmer must additionally override this class's put method (which otherwise throws an UnsupportedOperationException), and the iterator returned by entrySet().iterator() must additionally implement its remove method.（要实现可修改的map，必须重写put方法，以及在entrySet().iterator()里增加remove方法）
   - The programmer should generally provide a void (no argument) and map constructor, as per the recommendation in the Map interface specification.（一个无参构造器，一个map参数构造器）



3. SortedMap

   - A Map that further provides a total ordering on its keys. The map is ordered according to the natural ordering of its keys, or by a Comparator typically provided at sorted map creation time. （自然序或者创建的时候提供的Comparator）
   - All keys inserted into a sorted map must implement the Comparable interface (or be accepted by the specified comparator). （所有key必须是可以比较的）
   - All general-purpose sorted map implementation classes should provide four "standard" constructors.（提供4个构造器）
     - A void (no arguments) constructor, which creates an empty sorted map sorted according to the natural ordering of its keys.
     - A constructor with a single argument of type Comparator, which creates an empty sorted map sorted according to the specified comparator.
     - A constructor with a single argument of type Map, which creates a new map with the same key-value mappings as its argument, sorted according to the keys' natural ordering.
     - A constructor with a single argument of type SortedMap, which creates a new sorted map with the same key-value mappings and the same ordering as the input sorted map.

   - several methods return submaps with restricted key ranges. Such ranges are half-open, that is, they include their low endpoint but not their high endpoint (where applicable).（submaps注意是前闭后开）
     - SortedMap<String, V> sub = m.subMap(low, high+"\0");（前闭后闭）
     - SortedMap<String, V> sub = m.subMap(low+"\0", high);（前开后开）



4. NavigableMap

   - A SortedMap extended with navigation methods returning the closest matches for given search targets. Methods lowerEntry, floorEntry, ceilingEntry, and higherEntry return Map.Entry objects associated with keys respectively less than, less than or equal, greater than or equal, and greater than a given key, returning null if there is no such key. Similarly, methods lowerKey, floorKey, ceilingKey, and higherKey return only the associated keys. All of these methods are designed for locating, not traversing entries.
     - 支持搜索，返回与目标最匹配的项
     - lowerEntry, floorEntry, ceilingEntry, and higherEntry 分别返回<、<=、>=、> than a given key
     - lowerKey, floorKey, ceilingKey, and higherKey 同理
     - All of these methods are designed for locating, not traversing entries.（所有的方法都不是遍历）

   - The descendingMap method returns a view of the map with the senses of all relational and directional methods inverted.（descendingMap方法返回倒序映射的视图）
   - Methods subMap, headMap, and tailMap differ from the like-named SortedMap methods in accepting additional arguments describing whether lower and upper bounds are inclusive versus exclusive.（subMap、headMap、tailMap支持设定是否包含边界）
   - firstEntry, pollFirstEntry, lastEntry, and pollLastEntry（返回或者移除the least and greatest mappings）
   - Methods subMap(K, K), headMap(K), and tailMap(K) are specified to return SortedMap to allow existing implementations of SortedMap to be compatibly retrofitted to implement NavigableMap.（subMap(K, K), headMap(K), and tailMap(k)返回SortedMap以兼容其他SortedMap实现）



5. HashMap：基于数组（bucket）+链表实现（+红黑树）
   1. 继承AbstractMap类，实现了Map、Serializable和Cloneable接口

   2. 初始容量16（initial capacity），加载因子0.75（load factor），创建map时可指定合适的初始容量，以减少rehash（树化因子8，树转链表因子6）

   3. 单向链表

   4. hash算法：先取hashcode，1.7，（1.8：高16位与低16位异或，保证32位中只要有1位发生改变，整个hash值就会改变）

   5. index算法：hash&(length - 1)

   6. 判断key相同：先判断hash值，再判断==，再equals

   7. key为null的值永远放在table[0]的链表中

   8. 插入后再判断是否树化，是否扩容，扩容都是扩容两倍

   9. 容量始终为2的n次幂：

      ![](https://raw.githubusercontent.com/yonglianghong/my-drive/md/img/Java.Collections.Framework.Map.HashMap.Resize.png)

      10. 1.7多线程死循环：https://coolshell.cn/articles/9606.html
      11. 1.7与1.8区别：
          - 1.7头插（效率高），1.8尾插
          - hash算法
          - 单链表 vs 单链表+红黑树
          - resize后，如果新表的数组索引位相同，1.7链表元素会倒置，1.8不会倒置

      12. vs HashTable
          - HashTable不支持null（HashMap允许null键和值）
          - HashTable部分方法是线程安全的（synchronized，HashMap所有方法都是非同步的）
          - HashTable的初始容量是11，扩容是2*length+1
          - hash算法不同，index算法也不同，HashTable是对length取余



6. LinkedHashMap
   - Hash table and linked list implementation of the Map interface, with predictable iteration order. This implementation differs from HashMap in that it maintains a doubly-linked list running through all of its entries. This linked list defines the iteration ordering, which is normally the order in which keys were inserted into the map (insertion-order). Note that insertion order is not affected if a key is re-inserted into the map.（Hash表+双链表，保存插入序，注意，重复插入时，原来的顺序不改变）
   
   - A special constructor is provided to create a linked hash map whose order of iteration is the order in which its entries were last accessed, from least-recently accessed to most-recently (access-order). This kind of map is well-suited to building LRU caches.（提供一个构造器，指定按access顺序链接entry，由前至后，从最少访问的到最常访问的）
   
   - The removeEldestEntry(Map.Entry) method may be overridden to impose a policy for removing stale mappings automatically when new mappings are added to the map.（可覆写removeEldestEntry(Map.Entry)方法，默认返回false，改为返回true，当新的mapping加入时，会自动删除最少访问的entry）
   
   - LinkedHashMap的迭代通常比HashMap更高效，正比与mapd的size（HashMap的迭代正比于容量?）
   
   - 双链表及自定义LRU cache：
   
     - HashMap的Entry基础上增加了brfore、after指针
   
       ```java
       transient LinkedHashMap.Entry<K,V> head; // 内部维护双链表的头结点
       transient LinkedHashMap.Entry<K,V> tail; // 内部维护双链表的尾结点
       final boolean accessOrder; // 是否按照访问顺序来排序（默认false，按照插入顺序）
       ```
   
     - 访问元素时，比如调用get(k)方法后，会再调用afterNodeAccess(e)，即将节点移动到维护的双向链表的尾部
   
     - 自定义LRU cache：
   
       ```java
       LinkedHashMap<String, String> map = new LinkedHashMap<String, String>(5, 0.75F, true) {
           @Override
           protected boolean removeEldestEntry(Map.Entry<String, String> eldest) {
               //当LinkHashMap的容量大于等于5的时候,再插入就移除旧的元素
               return this.size() >= 5;
           }
       };
       ```



7. TreeMap
   - A Red-Black tree based NavigableMap implementation. The map is sorted according to the natural ordering of its keys, or by a Comparator provided at map creation time, depending on which constructor is used.（基于红黑树的NavigableMap实现，默认按自然序）
   - This implementation provides guaranteed log(n) time cost for the containsKey, get, put and remove operations. Algorithms are adaptations of those in Cormen, Leiserson, and Rivest's Introduction to Algorithms.（插入、删除、是否存在时间复杂度为log(n)）



##### 8. hashcode()和equals

1. equals() vs ==
   - Main difference between .equals() method and == operator is that one is method and other is operator.（一个是方法，一个是操作符）
   - We can use == operators for reference comparison (address comparison) and .equals() method for content comparison. In simple words, == checks if both objects point to the same memory location whereas .equals() evaluates to the comparison of values in the objects.（我们可以用==比较内存地址，equals比较内容）

2. ==
   - We can apply equality operators for every primitive types including boolean type.（适用于基本类型）
   - we can also apply equality operators for object types.there should be compatibility between arguments types (either child to parent or parent to child or same type)（适用于Object类型，且参数类型之间应该兼容）

3. equals
   - 自反性、对称性、传递性、一致性
   - string equals() method compares the two given strings based on the data/content of the string.（String的equals方法比较内容）
   - Object的equals方法，实际上是用==实现的
   - x.equals(null) => false,instanceof null => false

4. hashcode
   - 关注生成速度（速度优于唯一性）
   - 均匀
   - 任何时候，同一对象的hashcode值一致
   - hashcode生成工具：参考jakarta.apache.org/commons的lang下



##### 9. Arrays、Collections和实现算法

- 算法（Algorithms）：The Collections class contains these useful static methods.
    - Sort(List)：利用归并算法实现的排序，why not quicksort? 性能比较平均，并且是stability的算法（vs quicksort）
    - binarySearch(List, Object)：二分查找
    - reverse(List)：逆序
    - shuffle(List)：洗牌，随机打乱顺序（from the last element up to the second, repeatedly swapping a randomly selected element into the "current position"）从后向前，交换当前位置与前面的随机一个位置上的元素
    - fill(List, Object)：用指定值覆盖原来的所有元素
    - copy(List dest, List src)：复制
    - min(Collection)
    - max(Collection)
    - rotate(List list, int distance)：将列表中的所有元素旋转指定距离
    - replaceAll(List list, Object oldVal, Object newVal)：替换指定值
    - indexOfSubList(List source, List target)：返回第一个匹配的子序列index
    - lastIndexOfSubList(List source, List target)：返回最后一个匹配的子序列索引
    - swap(List, int, int)：交换元素
    - frequency(Collection, Object)：统计指定值出现的次数
    - disjoint(Collection, Collection)：判断两个集合是否相交
    - addAll(Collection<? super T>, T...)：添加所有值

- Arrays
    - Contains static methods to sort, search, compare, hash, copy, resize, convert to String, and fill arrays of primitives and objects.
    - 排序、搜索、比较、hash、复制、扩容、转成string，适用于基本类型或者object的数组



##### 10. Stack、Vector、Enumeration、HashTable

- Vector：The Vector class implements a growable array of objects.（可自动扩容的对象数组，注意很多方法使用synchronized实现同步）
- Stack继承自Vector，实现栈的特点
- Enumeration：迭代的初始设计，可用Iteration替代，参考适配器设计模式兼容历史代码
- HashTable：类似HashMap，同样很多方法使用synchronized实现同步



##### 11. BitSet

1. 基于`Array[Long]`实现，可以理解为`Array[Long][Bit]`，即一个long是64位。初始化：`Array[Long] word = new Array[Long](initSize)`
2. 添加元素：
    - 先计算一维：`val idx = elem >> LogWL //LogWL=6`,因为long是64位整型
    - 利用或运算更新：`updateWord(idx, word(idx) | (1L << elem))`
    - 注：如果elem大于64，则`1L << elem `等价于 `1L << (elem / 64)`



##### 12. fail-fast:

- fail-fast，是Java集合的一种错误检测机制。当多个线程对集合进行结构上的改变的操作时，就可能触发，抛出ConcurrentModificationException异常。
-  迭代器的快速失败行为无法得到保证，即不能保证一定会出现该错误，但是快速失败操作会尽最大努力抛出ConcurrentModificationException异常
- 利用modCount记录structurally modified次数，如果这个值更改，则iterator/listIterator会抛出ConcurrentModificationException异常



