## JUC

线程就是一个单独的资源类，没有任何附属的操作。

### CAS

CAS（Compare and Swap）是一个硬件指令。

### Lock：锁

#### Lock接口

公平锁：公平，先来后到

非公平锁：可以插队（可重入锁的默认形式）

synchronized和Lock的区别：

1. synchronized是关键字，Lock是一个接口
2. synchronized无法获取锁的状态，Lock可以通过`trylock()`判断是否取到了锁
3. synchronized会自动释放，Lock必须手动释放，不自动释放会造成死锁
4. synchronized未获得锁会一直等待，Lock可判断是否获得锁，如果长时间不能获得锁可以转去处理其他事情
5. synchronized可重入、不可中断等待、非公平；Lock可重入、可以中断等待、默认非公平（但可通过带布尔参数的构造方法构造公平锁）
6. synchronized适合锁少量的代码，Lock适合锁大量同步代码，
7. synchronized使用对象本身的wait、notify调度机制，而Lock可以使用Condition进行线程间调度
8. synchronized独占锁，Lock的ReadWriteLock子类实现读写分离

#### Condition接口

Condition用来替代传统的Object的`wait()`和`notify()`，使用Condition的`await()`和`signal()`这种方式实现线程间协作更加安全和高效。

生成一个Condition：

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
```

调用Condition的`await()`和`signal()`方法，都必须在lock保护之内，就是说必须在`lock.lock()`和`lock.unlock`之间才可以使用。

当Consumer调用`condition.await()`方法后，线程将释放锁，并将自己阻塞，等待唤醒。

Producer获取到锁之后，开始生产，完毕后调用`condition.singalAll()`方法，唤醒Consumer，Consumer恢复执行。

#### `ReadWriteLock`接口

`ReentrantReadWriteLock`实现了`ReadWriteLock`接口，可重入。

读读可以并发，读写、写写不能并发。

### Collections：并发集合

#### `BlockingQueue`接口

 阻塞队列，顾名思义，它首先是一个队列，满足FIFO。

用队列来实现生产者消费者之间的数据共享，使用阻塞队列，当队列中的填满数据时，生产者端的线程会被自动挂起，直到队列中有空位置，线程倍自动唤醒。我们不需要关心什么时候需要阻塞线程，什么时候唤醒线程。

核心方法：

1. `boolean offer(E e)`如果可能的话，将元素放入队列，返回true，否则返回false。本方法不阻塞执行当前方法的线程。
2. `boolena offer(E e, long timeout, TimeUnit unit)`在指定时间内不能入队，则失败返回。不阻塞当前线程。
3. `void put(E e)`入队，会阻塞当前线程。
4. `E poll(long time, TimeUnit unit)` 尝试在指定时间内取走队头元素，无可用元素返回null。
5. `E take()`取走队头元素，此方法会阻塞当前线程。
6. `int drainTo(Collection c)`一次性取走所有可用元素。

#### `ArrayBlockingQueue`

数组实现，维护着两个整型变量，标识队头和队尾的位置，还有一个整型变量标识队列中元素个数。

入队和出队共用同一个锁，两者无法并行。因为数组实现数据读写已经足够轻巧，引入独立的锁性能上每太大提升。

#### `LinkedBlockingQueue`

基于链表实现，出队和入队能并行。

#### `ConcurrentLinkedQueue`

#### `CopyOnWriteArrayList`

`Copy On Write（COW）`即写时复制。

当我们往`CopyOnWriteArrayList`添加一个元素时，不直接往当前容器中添加，而是先复制其中数组地一个副本，在这个副本上进行修改。修改完成之后，再将指针指向它。

这样做的好处是我们可以对`CopyOnWriteArrayList`并发地读，而不需要加锁。`CopyOnWrite`容器是一种读写分离地思想。

它的add方法使用了`ReentrantLock`来保证不能有两个线程同时写。

`CopyOnWriteArrayList`是**弱一致的**。

当一个线程获取迭代器，同时另一个线程修改了List，这对于迭代器是不可见的。

总结：

+ `CopyOnWriteArrayList`使用Reentrant独占锁，保证同时只有一个线程对集合进行写操作。
+ 数据存储在数组中，数组长度是动态变化的，`size()`方法实时获取数组长度。
+ 在修改时，并不是直接操作数组，而是先复制一个副本，在副本上修改，最后用新数组替换旧数组。
+ 使用迭代器遍历时不加锁，不会抛出`ConcurrentModificationException`，因为使用迭代器遍历操作的是数组的副本。

#### `ConcurrentHashMap`

1.7：

采用了分段锁实现，`Segment`继承自`ReentrantLock`，多个桶共用一个锁。

put需要获取锁，get不需要获取锁。

1.8：

链表 + 红黑树实现，放弃了Segment分段锁设计，每个Node有自己的一个锁，采用了CAS + synchronized来保证并发安全。

put时，如果数组位置为空即不冲突，则直接使用CAS操作new一个Node并将Node放入这个数组的位置；如果不为空即冲突，则用synchronized关键字锁住这个桶，并插入Node。

### Atomic：原子类

其基本的特征就是再多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性。即当某个线程进入方法执行其中的指令时，不会被其他的线程打断。别的线程就像自旋锁一样，一直等到该方法执行完毕，才由JVM从等待队列中选择一个线程进入。

#### 基础类：`AtomicBoolean`、`AtomicInteger`、`AtomicLong`

#### 数组：`AtomicBooleanArray`、`AtomicIntegerArray`、`AtomicLongArray`

#### 引用：`AtomicReference`







### Executor：线程池

#### Future

#### Fork/Join框架

ForkJoin是Java7提供的原生多线程并行处理框架，其基本思想是将大任务分割成小任务，最后将小任务聚合起来得到结果。fork是分解的意思, join是收集的意思. 它非常类似于HADOOP提供的MapReduce框架，只是MapReduce的任务可以针对集群内的所有计算节点，可以充分利用集群的能力完成计算任务。ForkJoin更加类似于单机版的MapReduce。

在fork/join框架中，若某个子问题由于等待另一个子问题的完成而无法继续执行。那么处理该子问题的线程会主动寻找其他尚未运行完成的子问题来执行。这种方式减少了线程的等待时间，提高了性能。子问题中应该避免使用synchronized关键词或其他方式方式的同步。也不应该是一阻塞IO或过多的访问共享变量。在理想情况下，每个子问题的实现中都应该只进行CPU相关的计算，并且只适用每个问题的内部对象。唯一的同步应该只发生在子问题和创建它的父问题之间。



