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

   - Iterable：可迭代

   - Iterator：迭代器

   - ListIterator：List的迭代器， 支持双向迭代，元素插入、替换，检索索引。

   

`Iterable<T>`：

- `Iterator<T> iterator()`
- `default forEach(Consumer<? super T> action)`和`default Spliterator<T> spliterator()`

`Iterator<E>`：

- boolean hasNext()：
- E next()：
- `default void remove() { throw new UnsupportedOperationException("remove");}`
- `default void forEachRemaining(Consumer<? super E> action)`

default表示接口的默认实现（从1.8开始，接口可添加默认实现方法）

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



`ListIterator<E> extends Iterator<E>`：

- 正序迭代：
  - boolean hasNext();
  - E next();
  - int nextIndex();
- 逆序迭代：
  - boolean hasPrevious();
  - E previous();
  - int previousIndex();
- void remove();
- void set(E e);
- void add(E e);

`ListIterator`的构造器方法`ListIterator(n)`创建一个一开始就指向索引为n的元素的迭代器。



`Foreach`语法：支持数组或其他任何Iterable。



2. Ordering

- - Comparable：实现此接口的类都提供了自然顺序（natural ordering）
  - Comparator：用来表示顺序，可以覆盖自然顺序，或者不实现Comparable接口的排序对象



`Comparable<T>`

- `int compareTo(T o)`



Comparator：

- `int compare(T o1, T o2)`：注意常用错误写法，`return i-j`，如果i是很大的正整数，j是很大的负整数，则可能溢出
- `boolean equals(Object obj)`：通常不用实现

若某类本身没有实现Comparable接口，可以自定义Comparator比较器，Arrays/Collections.sort()支持传入comparator。



3. Performance
   - RandomAccess：支持快速随机访问（generally constant time），*The primary purpose of this interface is to allow generic algorithms to alter their behavior to provide good performance when applied to either random or sequential access lists.*
   - 只是一个单独的接口，里面没有定义任何方法



##### 4. 栈和队列（Stack and Queue）

1. Stack：Last In First Out 后进先出

```java
public class Stack<T> {
  private LinkedList<T> storage = new LinkedList<T>();
  public void push(T v){ storage.addFirst(v); }
  public T peek(){ return storage.getFirst(); }
  public T pop(){ return storage.removeFirst(); }
  public boolean empty(){ return storage.isEmpty(); }
  public String toString(){ return storage.toString(); }
}
```

栈经常用来对表达式求值



2. Queue：First In First Out 先进先出

Queue接口：

- add()/offer()：插入队尾，返回boolean
- peek()/element()：返回队头，peek()在为空的时候返回null，element()抛出NoSuchElementException异常
- poll()/remove()：移除并返回队头，poll()在为空的时候返回null，remove()抛出NoSuchElementException异常



PriorityQueue类：可以确保调用peek()、poll()、remove()方法时，获取的将是队列中优先级最高的元素。基于堆实现，默认按元素自然序，可提供Comparator。



双端队列：

```java
public class Deque<T>{
    private LinkedList<T> deque = new LinkedList<T>();
    public void addFirst(T e){ deque.addFirst(e); }
    public void addLast(T e){ deque.addLast(e); }
    public T getFirst(){ return deque.getFirst(); }
    public T getLast(){ return deque.getLast(); }
    public T removeFirst(){ return deque.removeFirst(); }
    public T removeLast(){ return deque.removeLast(); }
    public int size(){ return deque.size(); }
    public String toString(){ return deque.toString(); }
}
```



##### 5. List

List：An ordered collection (also known as a *sequence*). 一种有序collection，即序列。

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

  

ArrayList：擅长快速访问（本质上是因为连续地址空间）

1. 主要方法：

- - 添加：add(E e)/add(int index, E element)
  - 删除：remove(int index)/remove(Object o)
  - 获取：get(int index)
  - 设置（替换）：set(int index, E element)

2. 允许放入null元素
3. 底层是Object[]（Java泛型是编译器提供的语法糖），get时强制转型
4. 实现Serializable、RandomAccess、Cloneable接口
5. 效率：

- - 插入末尾，如果不扩容?，O(1)，插入指定索引，插入后要移动元素，O(n)
  - 删除后要将后面的元素向前移动，O(n)
  - 查询，O(1)



LinkedList：擅长快速插入删除（用链表实现的列表）

1. 主要方法：

- - 添加：add(E e)/add(int index, E element)
  - 删除：remove(int index)/remove(Object o)
  - 获取：get(int index)
  - 设置：set(int index, E element)

2. 双向循环链表，允许元素为null?

3. 实现Serializable、Cloneable接口

4. 查找指定索引处元素，有一个加速动作，比较index与size >> 1（即size/2）来决定从first还是last开始遍历

5. 效率：

- - 插入末尾，O(1)，插入指定索引，O(n)
  - 删除，需要先查找，O(n)
  - 查找，O(n)



##### 6. Set

Set：无重复元素的Collection

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



HashSet：为快速查找而设计的Set，基于HashMap实现



LinkedHashSet：继承自HashSet，使用双链表维护插入顺序



SortedSet -> TreeSet：元素必须实现Comparable，保持自然顺序，基于TreeMap实现，底层树形结构

主要方法：

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

###### 7.1. Map整体结构

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



##### 9. Arrays和Collections、常用算法



##### 10. Stack、Vector、Enumeration、HashTable



##### 11. BitSet





