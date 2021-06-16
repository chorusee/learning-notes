### Java多线程

程序：是指令和数据的有序集合，其本身没有任何运行含义，是一个静态概念。

进程：是程序的一次具体执行过程，它是一个动态的概念。是系统分配资源的单位。

线程：通常一个进程中可以包含多个线程，线程是CPU调度和执行的单位。

#### 线程创建

##### 继承`Thread`类

1. 自定义线程继承`Thread`类
2. 重写`run()`方法，编写线程具体执行体
3. 创建线程对象，调用`start()`方法启动线程

##### 实现`Runnable`接口

1. 自定义类实现`Runnable`接口
2. 实现`run()`方法，编写线程执行体
3. 创建线程对象，调用`start()`方法启动

##### 实现`Callable`接口

1. 实现`Callable`接口，需要返回值类型
2. 重写`call()`方法
3. 创建目标对象
4. 创建执行服务：`ExecutorService ser = Executors.newFixedThreadPool(1);`
5. 提交执行：`Future<Boolean> result = ser.submit(t);`
6. 获取结果：`boolean r = result.get()`
7. 关闭服务：`ser.shutDownNow()`

#### 线程停止

操作系统中线程有五个状态：

+ 创建    ->启动线程，变为就绪状态
+ 就绪    ->获得CPU资源，变为运行状态
+ 阻塞    ->解除阻塞，变为就绪状态
+ 运行    ->释放CPU资源，变为就绪状态    ->请求外部资源，变为阻塞状态    ->执行完毕，或外部干涉终止线程，变为死亡状态
+ 死亡

而在Java中定义了六种状态：

+ NEW：线程创建，还未调用`start()`方法时处于这个这个状态
+ RUNNABLE：执行中或者等待CPU资源时处于这个这个状态
+ BLOCKED：当等待监视器锁，比如说等待进入synchronized方法/代码块的线程处于阻塞状态
+ WAITING：调用`wait()`或`join()`方法的线程处于等待状态，等待另一个线程完成某个动作，另一个线程调用`notify()`、`notifyAll()`后变为RUNNABLE态
+ TIMED_WAITING：调用`sleep()`或`wait()`带有超时时间的方法后线程处于这个状态
+ TERMINATED：线程完成了执行，变为TERMINATED状态

不推荐使用JDK提供的`stop()`、`destroy`方法【已废弃】。

推荐线程自己停下来。建议使用一个标志位作为终止变量，当这个标志位为`false`时则终止线程。

#### 线程休眠

+ sleep(时间) 指定当前线程等待的毫秒数
+ sleep存在异常`InterruptedException`
+ sleep时间到达后线程进入RUNNABLE状态
+ sleep可以模拟网络延时、倒计时等
+ 每个对象都有一个锁，sleep不会释放锁

#### 线程礼让

+ 让当前正在执行的线程暂停，但不阻塞
+ 将线程从运行状态转换为就绪状态
+ 让CPU重新调度，但礼让不一定成功

#### 线程强制执行

+ join合并线程，待此线程执行完毕后，再执行其他线程，其他线程阻塞
+ 可以想象成插队

#### 线程优先级

+ Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度哪个线程来执行

+ 线程的优先级用数字表示，范围从1-10

  + `Thread.MIN_PRIORITY = 1`
  + `Thread.NORMA_PRIORITY = 5`
  + `Thread.MAX_PRIORITY = 10`

+ 使用以下方式改变或获取优先级

  `getPriority()`

  `setPriority(int xx)`

#### 守护线程

+ 线程分为**用户线程**和**守护线程**
+ 虚拟机必须确保用户线程执行完毕
+ 虚拟机不必等待守护线程执行完毕
+ 守护线程包括后台记录日志操作、监控内存、垃圾回收等
+ `thread.setDaemon(true)`设为守护线程

#### 线程同步机制

线程同步其实是一种等待机制，多个需要同时访问此对象的线程进入**对象等待池**形成队列，等待前面线程使用完毕，下一个线程再使用。

由于同一进程的多个线程共享同一块存储空间，在带来方便的同时，也带来了访问冲突问题，为保证数据在方法中被访问时的正确性，在访问时加入**锁机制 synchronized**。当一个线程获得对象的排他锁，独占资源，其他线程必须等待，使用后释放锁即可。但是存在以下问题：

+ 一个线程持有锁会导致其他所有需要此锁的线程挂起
+ 在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题
+ 如果一个优先级高的线程等待一个优先级低的线程释放锁，会导致优先级倒置，引起性能问题

#### 同步方法及同步块

`synchronized`关键字包括两种用法：

+ 同步方法

  ``` java
  public synchronized void method(int args) {}
  ```

  synchronized方法控制对“对象”的访问，每个对象对应一把锁，每个synchronized方法都必须获得调用该方法的对象的锁才能执行，否则线程会阻塞。方法一旦执行就独占该锁，直到该方法返回才释放，后面被阻塞的线程才能获得该锁，得以继续执行。

  同一个类里的多个由`synchronized`修饰的方法也要互斥执行。

  缺陷：若将一个大的方法声明为synchronized将会影响效率

+ 同步块

  ```java
  synchronized(Obj) {
      // 代码块
  }
  ```

  `Obj`称之为**同步监视器**，`Obj`可以是任何对象，但是推荐使用共享资源作为同步监视器。同步方法中无需指定同步监视器，因为同步方法的同步监视器就是`this`，或者是`class`。

  同步监视器的执行过程：

  1. 第一个线程访问，锁定同步监视器，执行其中代码
  2. 第二个线程访问，发现同步监视器被锁定，无法访问
  3. 第一个线程访问完毕，解锁同步监视器
  4. 第二个线程访问，发现同步监视器没有锁，然后锁定访问

`synchronized`原理：

每个对象都有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行同步方法或同步块时尝试获取该锁。该监视器锁是一个可重入锁。过程如下：

1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。
2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1。
3. 线程退出一次临界资源，则monitor进入数减1。
4. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

#### 死锁

多个线程各自占有一些共享资源，并且互相等待其他线程占有的资源才能运行，而导致两个或多个线程都在等待对方释放资源，都停止执行的情形称为死锁。

产生死锁四个必要条件：

+ 互斥条件：一个资源每次只能被一个进程使用
+ 请求保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放
+ 不剥夺条件：进程已获得的资源在未使用完之前，不能强行剥夺
+ 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系

#### Lock

从JDK 5.0开始，Java提供了更强大的线程同步机制——通过显式定义同步锁对象来实现同步。同步锁使用Lock对象充当。

`java.util.concurrent.locks.Lock`接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象。

`ReentrantLock`类实现了Lock，它拥有与synchronized相同的并发性与内存语义。在实现线程安全控制中，比较常用的是`ReentrantLock`，可以显式加锁，释放锁。

```java
private final ReentrantLock lock = new ReentrantLock();
try {
    lock.lock();
    // 业务代码
} finally {
    lock.unlock();
}
```

`synchronized`与`Lock`对比：

+ Lock是显式锁（手动开启和关闭锁），synchronized是隐式锁，出了作用域自动释放
+ Lock只有代码块锁，synchronized有代码块锁和方法锁
+ 使用Lock，JVM将花费更少的时间来调度线程，性能更好。并且具有更好的扩展性（提供更多子类）
+ 使用优先顺序：Lock > 同步代码块 > 同步方法

#### 线程通信

Java提供了几个方法解决线程之间的通信问题

+ `wait()` ：表示线程一直等待，直到其他线程通知，与`sleep`不同，会释放锁
+ `wait(long timeout)` ：指定等待的毫秒数
+ `notify()` ：唤醒一个处于等待状态的线程
+ `notifyAll()` ：唤醒同一个对象上所有调用`wait()`方法的线程，优先级别高的线程优先调度

注意：都是Object类的方法，都只能在同步方法或者同步代码块中使用，因为必须要明确是在哪个锁上的操作，否则会抛出异常`IllegalMonitorStateException`。

除此之外，还可使用以下方式实现进程间通信：

+ synchronized实现同步，本质上是“共享内存”式的通信，多个线程访问同一个共享变量，谁拿到了锁，谁就可以执行。
+ while轮询的方式，设置一个标志或变量，线程A不断改变这个标志，线程B不停地通过while语句检测这个标志，当满足条件时才执行后面的代码。这种方式会浪费CPU资源，并且设置的标志有可见性问题，需要设置成`volatile`。
+ JUC里的`Condition`类

#### 线程池

经常创建和销毁线程，比较耗费资源，对性能影响很大。

可以提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中。可以避免频繁创建销毁，实现重复利用。

好处：

+ 提高了响应速度
+ 降低了资源消耗
+ 便于线程管理

核心是`ThreadPoolExecutor`类：

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}
```

`ThreadPoolExecutor`，`AbstractExecutorService`，`ExecutorService`，`Executor`

`Executor`是一个顶层接口，只声明了一个`execute(Runnable)`方法，返回void 。

`ExecutorService`继承了`Executor`接口，并声明了一些方法：`submit`，`invokeAll`，`invokeAny`，`shutdown`等。

`AbstractExecutorService`实现了`ExecutorService`接口，基本实现了`ExecutorService`中申明的所有方法。

`ThreadPoolExecutor`继承了`AbstractExecutorService`。

在`ThreadPoolExecutor`中有几个非常重要的方法：

+ `execute(Runnable)`：通过这个方法向线程池提交一个任务，交由线程池执行
+ `submit(Callable)`：与execute方法不同，它能够返回执行的结果，用`Future`来获取任务执行结果
+ `shutdown()`：用来关闭线程池，调用此方法，线程池处于`SHUTDOWN`状态，线程池不能接受新的任务，它会等待所有的任务执行完毕
+ `shutdownNow()`：也是用来关闭线程池，调用此方法，线程池处于`STOP`状态，线程池不能接受新的任务，并且会去尝试终止正在执行的任务

线程池状态：

+ `RUNNING`，当创建线程池后线程池处于RUNNING状态
+ `SHUTDOWN`
+ `STOP`
+ `TERMINATED`，当线程池处于`SHUTDOWN`或`STOP`状态，并且所有工作线程都已销毁，任务缓存队列已经清空或执行完毕后，线程池被设置为`TERMINATED`状态

重要成员变量：

```java
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private volatile long  keepAliveTime;    //线程存活时间
private final TimeUnit timeUnit;    // 线程存活的时间单位
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
private final RejectedExecutionHandler RejectedExecutionHandler;    //当任务队列满了且线程池个数达到maximumPoolSize采取的拒绝操作

private volatile int   poolSize;       //线程池中当前的线程数
```



四种线程池：

+ `CachedThreadPool`

  ```java
  corePoolSize = 0;
  maximumPoolSize = Integer.MAX_VALUE;
  keepAliveTime = 60L;
  unit = TimeUnit.SECONDS;
  workQueue = SynchronousQueue; // (同步队列)
  ```

  当有新任务来时，则插入到`ThreadPoolExecutor`中，会在线程池中寻找可用线程来执行，如果有可用线程则执行，若没有可用线程则创建一个来执行该任务。若池中线程空闲时间超过指定时间，则该线程会被销毁。

  适用于执行很多短期的异步任务。

+ `FixedThreadPool`

  ```java
  corePoolSize = n; // 接受指定参数设定为线程数
  maximumPoolSize = n;
  keepAliveTime = 0L;
  unit = TimeUnit.MILLISECONDS;
  workQueue = LinkedBlockingQueue<Runnable>; // 无界阻塞队列
  ```
  创建可容纳固定数量线程的线程池，每个线程存活时间不限，当线程池满了就不再添加线程。
  
  适用执行于长期任务。
  
+ `SingleThreadExecutor`

  ```java
  corePoolSize = 1;
  maximumPoolSize = 1;
  keepAliveTime = 0L;
  unit = TimeUnit.MILLISECONDS;
  workQueue = LinkedBlockingQueue<Runnable>; // 无界阻塞队列
  ```
  创建只有一个线程的线程池。
  
  适用于按顺序执行的场景。
  
+ `ScheduledThreadPool`

  ```java
  corePoolSize = n;  // 传递来的参数
  maximumPoolSize = Integer.MAX_VALUE;
  keepAliveTime = 0L;
  unit = TimeUnit.NANOSECONDS;
  workQueue = DelayedWorkQueue; // 按时间升序排列的队列
  ```

  适用于执行周期性任务。

建议不使用Executors创建线程池，直接通过`ThreadPoolExecutor`的方式创建，因为：

1. `FixedThreadPool`和`SingleThreadPool`：允许的请求队列长度为`Integer.MAX_VALUE`，可能会堆积大量的请求从而导致OOM；

2. `CachedThreadPool`：允许创建线程数量为`Integer.MAX_VALUE`,可能会创建大量的线程，从而导致OOM。

线程池工作过程：

1. 通过`execute`方法提交任务时，当线程池中的线程数小于`corePoolSize`时，新提交的任务将创建一个新线程来执行，即使线程池中存在空闲线程。
2. 通过`execute`方法提交任务时，当线程池中的线程数达到`corePoolSize`时，新提交的任务将被放入`workQueue`中，等待线程池中线程调度执行。
3. 通过`execute`方法提交任务时，当`workQueue`已满，且当前线程数量小于`maximumPoolSize`时，新提交的任务将创建新线程执行。
4. 当线程池中的线程执行完任务空闲时，会尝试从`workQueue`中取头结点任务执行。
5. 通过`execute`方法提交任务，当线程池中数量达到`maximumPoolSize`并且`workQueue`也满时，新提交的任务有`RejectedExecutionHandler`执行拒绝操作。
6. 当线程池中线程数超过`corePoolSize`，并且配置`allowCoreThreadTimeOut = true`时，空闲时间超过`keepAliveTime`的线程将被销毁，保持线程池中线程数为`corePoolSize`。
7. 当设置`allowCoreThreadTimeOut = false`时，任何空闲时间超过`keepAliveTime`的线程都会被销毁。

#### Spring中异步执行

1. 需要在主类上开启异步支持`@EnableAsync`
2. 在需要异步执行的方法上加上`@Async`注解。

原理还是动态代理技术，Spring帮助我们创建了一个代理类来实现异步调用。

注意：

1. 被调用的异步方法不能和调用的异步方法处于同一个类中，这个`@Transaction`注解的原因一样，由Spring的代理机制造成的。
2. `@Async`中可以指定一个线程池名，由我们自己创建一个线程池。若未指定则默认使用`SimpleAsyncTaskExecutor`执行，这不是一个线程池，未实现线程的复用。
