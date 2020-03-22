##### 1. 进程与线程

进程是系统资源分配和调度的基本单位，线程是进程的一个实体，是CPU分配和调度的基本单位。一个进程可以包含多个线程（至少一个），同一进程内的多个线程共享进程所拥有的全部资源。

一个Java程序实际上是一个JVM进程，JVM进程用一个主线程来执行main()方法，在main()方法内部，可以启动多个线程。此外，JVM还有负责垃圾回收的其他线程。

1. 创建线程的两种方式

   方法一：继承Thread类，覆写run方法

   ```java
   public class Main {
       public static void main(String[] args) {
           Thread t = new MyThread();
           t.start(); // 启动新线程
       }
   }
   
   class MyThread extends Thread {
       @Override
       public void run() {
           System.out.println("start new thread!");
       }
   }
   ```

   方法二：实现Runnable接口

   ```java
   public class Main {
       public static void main(String[] args) {
           Thread t = new Thread(new MyRunnable());
           t.start(); // 启动新线程
       }
   }
   
   class MyRunnable implements Runnable {
       @Override
       public void run() {
           System.out.println("start new thread!");
       }
   }
   ```

2. Thread

   线程休眠`Thread.sleep(n)`或`TimeUnit.SECONDS.sleep(n)`。

   - Every thread has a priority.Each thread may or may not also be marked as a daemon. When code running in some thread creates a new Thread object, the new thread has its priority initially set equal to the priority of the creating thread, and is a daemon thread if and only if the creating thread is a daemon.（每一个线程都有优先级，都可以被标记为守护进程。线程内部创建子线程，子线程初始优先级与父线程相同。守护线程创建的子线程也是守护线程。）
   - When a Java Virtual Machine starts up, there is usually a single non-daemon thread (which typically calls the method named main of some designated class). (JVM启动，通常执行main线程（非守护）)

   主要方法：
   - Thread currentThread() 返回当前线程
   - void interrupt() 中断
   - boolean isDaemon() 是否为守护进程
   - void join() 等待当前线程执行完毕
   - void sleep(long millis) 休眠，与`TimeUnit.SECONDS.sleep(n)`一样
   - void start() 开始执行
   - void yield() 让出CPU，回到就绪状态，此时CPU会调度相同或更高优先级的线程

   优先级：`setPriority(int newPriority)`，1~10，默认5。或用下面静态变量

   - `MAX_PRIORITY`
   - `MIN_PRIORITY`
   - `NORM_PRIORITY`

3. 线程的状态

   - New（新建）
   - Runnable（就绪）：Thread.start()
   - Running（运行）
   - Blocked（阻塞）
     - sleep()
     - wait()
     - 输入/出完成
     - 调用同步代码，但未获取锁
   - Dead（死亡）

4. 守护线程

   守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。因此，JVM退出时，不必关心守护线程是否已结束。

   ```java
   Thread t = new MyThread();
   t.setDaemon(true); // 必须在start前面
   t.start();
   ```

5. 中断线程

   可以向线程发出“中断请求”，当收到“中断请求”后，线程结束运行。

   ```java
   public class Main {
       public static void main(String[] args) throws InterruptedException {
           Thread t = new MyThread();
           t.start();
           Thread.sleep(1); // 暂停1毫秒
           t.interrupt(); // 中断t线程
           t.join(); // 等待t线程结束
           System.out.println("end");
       }
   }
   
   class MyThread extends Thread {
       public void run() {
           int n = 0;
           while (! isInterrupted()) { // 需要检测是否是interrupted状态
               n ++;
               System.out.println(n + " hello!");
           }
       }
   }
   ```
   
   使用场景：假设从网络下载一个100M的文件，如果网速很慢，用户等得不耐烦，就可能在下载过程中点“取消”，这时，程序就需要中断下载线程的执行。
   
   

##### 2. 线程池

1. ExecutorService

   线程的创建需要消耗时间和资源，可以复用线程。线程池内部维护了多个线程，没有任务的时候，这些线程都处于等待状态。如果有新任务，就分配一个空闲线程执行；如果所有线程都处于忙碌状态，新任务要么放入队列等待，要么增加一个新线程处理。

   ```java
   // 创建固定大小的线程池:
   ExecutorService executor = Executors.newFixedThreadPool(3);
   // 提交任务:
   executor.submit(task1);
   executor.submit(task2);
   executor.submit(task3);
   executor.submit(task4);
   executor.submit(task5);
   // 停止提交
   executor.shutdown();
   ```

   ExecutorService只是接口，Java标准库提供的如下常用实现类：

   - FixedThreadPool：线程数大小固定
   - CachedThreadPool：线程数根据任务动态调整
   - SingleThreadExecutor：单线程的线程池

   ExecutorService：

   - An Executor that provides methods to manage termination and methods that can produce a Future for tracking progress of one or more asynchronous tasks.（提供中止任务的方法，以及产生跟踪异步任务的Future的方法）
   - The shutdown() method will allow previously submitted tasks to execute before terminating, while the shutdownNow() method prevents waiting tasks from starting and attempts to stop currently executing tasks.（shutdown()后，不再接受新任务，之前提交的任务继续执行。shutdownNow()后，会尝试中止正在执行的任务。）
   - Method submit extends base method Executor.execute(Runnable) by creating and returning a Future that can be used to cancel execution and/or wait for completion.（submit()方法提交一个任务，返回Future，可以取消执行，或等待任务执行完成）
   - Methods invokeAny and invokeAll perform the most commonly useful forms of bulk execution, executing a collection of tasks and then waiting for at least one, or all, to complete.（invokeAll/invokeAny可以批量提交任务）
   - 主要方法：
     - submit
       - submit(Callable)
       - submit(Runnable)
       - submit(Runnable, T)
     - shutdown
       - shutdown()
       - shutdownNow()
       - awaitTermination(long, TimeUnit)
     - invokeAll
       - invokeAll(Collection)
       - invokeAny(Collection)

2. ScheduledThreadPool

   使用ScheduledThreadPool可以定期反复执行任务。

   ```java
   ScheduledExecutorService ses = Executors.newScheduledThreadPool(4);
   // 1秒后执行一次性任务:
   ses.schedule(new Task("one-time"), 1, TimeUnit.SECONDS);
   // 2秒后开始执行定时任务，每3秒执行:
   ses.scheduleAtFixedRate(new Task("fixed-rate"), 2, 3, TimeUnit.SECONDS);
   // 2秒后开始执行定时任务，以3秒为间隔执行:
   ses.scheduleWithFixedDelay(new Task("fixed-delay"), 2, 3, TimeUnit.SECONDS);
   ```

   

3. ForkJoin
   利用分治法，将大任务拆分成小任务，并行执行。

   利用Fork/Join对大数据进行并行求和：

   ```java
   import java.util.Random;
   import java.util.concurrent.*;
   public class Main {
       public static void main(String[] args) throws Exception {
           // 创建2000个随机数组成的数组:
           long[] array = new long[2000];
           long expectedSum = 0;
           for (int i = 0; i < array.length; i++) {
               array[i] = random();
               expectedSum += array[i];
           }
           System.out.println("Expected sum: " + expectedSum);
           // fork/join:
           ForkJoinTask<Long> task = new SumTask(array, 0, array.length);
           long startTime = System.currentTimeMillis();
           Long result = ForkJoinPool.commonPool().invoke(task);
           long endTime = System.currentTimeMillis();
           System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
       }
   
       static Random random = new Random(47);
   
       static long random() {
           return random.nextInt(10000);
       }
   }
   
   class SumTask extends RecursiveTask<Long> {
       static final int THRESHOLD = 500;
       long[] array;
       int start;
       int end;
   
       SumTask(long[] array, int start, int end) {
           this.array = array;
           this.start = start;
           this.end = end;
       }
   
       @Override
       protected Long compute() {
           if (end - start <= THRESHOLD) {
               // 如果任务足够小,直接计算:
               long sum = 0;
               for (int i = start; i < end; i++) {
                   sum += this.array[i];
                   // 故意放慢计算速度:
                   try {
                       Thread.sleep(1);
                   } catch (InterruptedException e) {
                   }
               }
               return sum;
           }
           // 任务太大,一分为二:
           int middle = (end + start) / 2;
           System.out.println(String.format("split %d~%d ==> %d~%d, %d~%d", start, end, start, middle, middle, end));
           SumTask subtask1 = new SumTask(this.array, start, middle);
           SumTask subtask2 = new SumTask(this.array, middle, end);
           invokeAll(subtask1, subtask2);
           Long subresult1 = subtask1.join();
           Long subresult2 = subtask2.join();
           Long result = subresult1 + subresult2;
           System.out.println("result = " + subresult1 + " + " + subresult2 + " ==> " + result);
           return result;
       }
   }
   ```

   一个大的计算任务0-2000首先分裂为两个小任务0-1000和1000-2000，这两个小任务仍然太大，继续分裂为更小的0-500，500-1000，1000-1500，1500-2000，最后，计算结果被依次合并，得到最终结果。

   核心代码`SumTask`继承自`RecursiveTask`，在`compute()`方法中，关键是如何“分裂”出子任务并且提交子任务：

   ```java
   class SumTask extends RecursiveTask<Long> {
       protected Long compute() {
           // “分裂”子任务:
           SumTask subtask1 = new SumTask(...);
           SumTask subtask2 = new SumTask(...);
           // invokeAll会并行运行两个子任务:
           invokeAll(subtask1, subtask2);
           // 获得子任务的结果:
           Long result1 = fork1.join();
           Long result2 = fork2.join();
           // 汇总结果:
           return result1 + result2;
       }
   }
   ```

   Java标准库提供的`java.util.Arrays.parallelSort(array)`可以进行并行排序，它的原理就是内部通过Fork/Join对大数组分拆进行并行排序，在多核CPU上就可以大大提高排序的速度。



##### 3. Runnable/Callable、Future/CompletableFuture

1. Runnable vs Callable

   ```java
   public interface Runnable {
   	void run()	
   }
   
   public interface Callable<V> {
   	V call()
   }
   ```

   Runnable无返回值，而Callable提供泛型参数设置返回值。都可用于`ExecutorService.submit()`方法。

2. Future

   `public interface Future<V>`

   - A Future represents the result of an asynchronous computation.（Future代表异步任务的结果）
   - Methods are provided to check if the computation is complete, to wait for its completion, and to retrieve the result of the computation.（可以检查是否完成，等待完成，取回计算结果。）
   - 主要方法：
   - `V get()` 同步等待结果
   - `V get(long, TimeUnit)`
   - `boolean cancel(boolean)` 取消task，但是有可能取消失败
   - `boolean isCancelled()` 
   - `boolean isDone()` 查看task是否已经执行完成

3. CompletableFuture

   从Java 8开始引入了`CompletableFuture`，它针对`Future`做了改进，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。

   `CompletableFuture`可以指定异步处理流程：

   - `thenAccept()`处理正常结果；
   - `exceptional()`处理异常结果；
   - `thenApplyAsync()`用于串行化另一个`CompletableFuture`；
   - `anyOf()`和`allOf()`用于并行化多个`CompletableFuture`。

   参考：https://www.liaoxuefeng.com/wiki/1252599548343744/1306581182447650



##### 4. wait和notify

`wait`和`notify`用于多线程协调运行：

- 在`synchronized`内部可以调用`wait()`使线程进入等待状态；
- 必须在已获得的锁对象上调用`wait()`方法；
- 在`synchronized`内部可以调用`notify()`或`notifyAll()`唤醒其他等待线程；
- 必须在已获得的锁对象上调用`notify()`或`notifyAll()`方法；
- 已唤醒的线程还需要重新获得锁后才能继续执行。

```java
import java.util.*;
public class Main {
    public static void main(String[] args) throws InterruptedException {
        var q = new TaskQueue();
        var ts = new ArrayList<Thread>();
        for (int i=0; i<5; i++) {
            var t = new Thread() {
                public void run() {
                    // 执行task:
                    while (true) {
                        try {
                            String s = q.getTask();
                            System.out.println("execute task: " + s);
                        } catch (InterruptedException e) {
                            return;
                        }
                    }
                }
            };
            t.start();
            ts.add(t);
        }
        var add = new Thread(() -> {
            for (int i=0; i<10; i++) {
                // 放入task:
                String s = "t-" + Math.random();
                System.out.println("add task: " + s);
                q.addTask(s);
                try { Thread.sleep(100); } catch(InterruptedException e) {}
            }
        });
        add.start();
        add.join();
        Thread.sleep(100);
        for (var t : ts) {
            t.interrupt();
        }
    }
}

class TaskQueue {
    Queue<String> queue = new LinkedList<>();

    public synchronized void addTask(String s) {
        this.queue.add(s);
        this.notifyAll(); // 唤醒在this锁等待的线程
    }

    public synchronized String getTask() throws InterruptedException {
        while (queue.isEmpty()) {
          	// 释放this锁:
            this.wait();
          	// 重新获取this锁
        }
        return queue.remove();
    }
}
```



##### 5. 同步

1. 原子操作

   原子操作是指操作一旦开始，就一定不会被中断，会在“上下文切换”之间执行完毕。java原子操作如下：

   - 基本类型（除long、double之外，64位分离成两个32位操作来执行）上的**赋值操作**。注意，自增/减操作是非原子的。
   - 所有引用reference的赋值操作
   - java.concurrent.Atomic.* 包中所有类的一切操作

2. 原子类

   `java.util.concurrent.atomic`包下提供了组原子操作类，以`AtomicInteger`为例，它提供的主要操作有：

   - 增加值并返回新值：`int addAndGet(int delta)`
   - 加1后返回新值：`int incrementAndGet()`
   - 获取当前值：`int get()`
   - 用CAS方式设置：`int compareAndSet(int expect, int update)`

3. volatile/synchronized

   `volatile`关键字的目的是告诉虚拟机：

   - 每次访问变量时，总是获取主内存的最新值；
   - 每次修改变量后，立刻回写到主内存。

   `synchronized`同步的本质就是给指定对象加锁，加锁后才能继续执行后续代码，注意加锁对象必须是同一个实例。

   如何使用`synchronized`：

   1. 找出修改共享变量的线程代码块；
   2. 选择一个共享实例作为锁；
   3. 使用`synchronized(lockObject) { ... }`。

   volatile vs synchronized

   1. volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
   2. volatile仅能用在变量级别；synchronized则可以用在变量、代码块、方法和类级别
   3. volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性
   4. volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞
   5. volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化

4. ReentrantLock

   可以使用`ReentrantLock`代替`synchronized`加锁：

   ```java
   // synchronized
   public class Counter {
       private int count;
   
       public void add(int n) {
           synchronized(this) {
               count += n;
           }
       }
   
   }
   
   // ReentrantLock
   public class Counter {
       private final Lock lock = new ReentrantLock();
       private int count;
   
       public void add(int n) {
           lock.lock();
           try {
               count += n;
           } finally {
               lock.unlock();
           }
       }
   
   }
   ```

   与`synchronized`不同的是，`ReentrantLock`可以尝试获取锁

   ```java
   // 尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，tryLock()返回false
   if (lock.tryLock(1, TimeUnit.SECONDS)) { 
       try {
           ...
       } finally {
           lock.unlock();
       }
   }
   ```

   可以使用`Condition`替代wait和notify。`Condition`提供的`await()`、`signal()`、`signalAll()`原理和`synchronized`锁对象的`wait()`、`notify()`、`notifyAll()`是一致的，并且其行为也是一样的：

   - `await()`会释放当前锁，进入等待状态；
   - `signal()`会唤醒某个等待线程；
   - `signalAll()`会唤醒所有等待线程；
   - 唤醒线程从`await()`返回后需要重新获得锁。

   此外，和`tryLock()`类似，`await()`可以在等待指定时间后，如果还没有被其他线程通过`signal()`或`signalAll()`唤醒，可以自己醒来：

   ```java
   if (condition.await(1, TimeUnit.SECOND)) {
       // 被其他线程唤醒
   } else {
       // 指定时间内没有被其他线程唤醒
   }
   ```

5. ThreadLocal

   `ThreadLocal`表示线程的“局部变量”，它确保每个线程的`ThreadLocal`变量都是各自独立的；

   `ThreadLocal`适合在一个线程的处理流程中保持上下文（避免了同一参数在所有方法中传递）；

   使用`ThreadLocal`要用`try ... finally`结构，并在`finally`中清除。

   `ThreadLocal`实例通常总是以静态字段初始化如下：

   ```java
   static ThreadLocal<String> threadLocalUser = new ThreadLocal<>();
   ```

   它的典型使用方式如下：

   ```java
   void processUser(user) {
       try {
           threadLocalUser.set(user);
           step1();
           step2();
       } finally {
           threadLocalUser.remove();
       }
   }
   ```

   通过设置一个`User`实例关联到`ThreadLocal`中，在移除之前，所有方法都可以随时获取到该`User`实例：

   ```java
   void step1() {
       User u = threadLocalUser.get();
       log();
       printUser();
   }
   
   void log() {
       User u = threadLocalUser.get();
       println(u.name);
   }
   
   void step2() {
       User u = threadLocalUser.get();
       checkUser(u.id);
   }
   ```

   `step1()`、`step2()`以及`log()`方法内，`threadLocalUser.get()`获取的`User`对象是同一个实例。

6. 死锁

   死锁是指多个线程已经持有锁，并且阻塞等待对方线程释放锁。比如线程 “A”获得了刀，而线程“B”获得了叉。线程“A”就会进入阻塞状态来等待获得叉，而线程“B”则阻塞来等待“A”所拥有的刀。

   ```java
   public class DeadLockTest {
   
        public static void main(String[] args) {
            
            Thread t1 = new Thread(new DeadLock(true), "线程1");
            Thread t2 = new Thread(new DeadLock(false), "线程2");
   
            t1.start();
            t2.start();
       }
   }
   
   /**
    * 一个简单的死锁类
    * t1先运行，这个时候flag==true,先锁定obj1,然后睡眠1秒钟
    * 而t1在睡眠的时候，另一个线程t2启动，flag==false,先锁定obj2,然后也睡眠1秒钟
    * t1睡眠结束后需要锁定obj2才能继续执行，而此时obj2已被t2锁定
    * t2睡眠结束后需要锁定obj1才能继续执行，而此时obj1已被t1锁定
    * t1、t2相互等待，都需要得到对方锁定的资源才能继续执行，从而死锁。 
    */
   class DeadLock implements Runnable{
       
       private static Object obj1 = new Object();
       private static Object obj2 = new Object();
       private boolean flag;
       
       public DeadLock(boolean flag){
           this.flag = flag;
       }
       
       @Override
       public void run(){
           System.out.println(Thread.currentThread().getName() + "运行");
           
           if(flag){
               synchronized(obj1){
                   System.out.println(Thread.currentThread().getName() + "已经锁住obj1");
                   try {  
                       Thread.sleep(1000);  
                   } catch (InterruptedException e) {  
                       e.printStackTrace();  
                   }  
                   synchronized(obj2){
                       // 执行不到这里
                       System.out.println("1秒钟后，"+Thread.currentThread().getName()
                                   + "锁住obj2");
                   }
               }
           }else{
               synchronized(obj2){
                   System.out.println(Thread.currentThread().getName() + "已经锁住obj2");
                   try {  
                       Thread.sleep(1000);  
                   } catch (InterruptedException e) {  
                       e.printStackTrace();  
                   }  
                   synchronized(obj1){
                       // 执行不到这里
                       System.out.println("1秒钟后，"+Thread.currentThread().getName()
                                   + "锁住obj1");
                   }
               }
           }
       }
   
   }
   ```

















