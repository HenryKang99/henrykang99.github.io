## Java多线程

### 概念

**多线程**：一个程序同时执行多个任务，通常每一个任务称为一个线程（ thread），可以同时运行一个以上线程的程序称为多线程程序（ multi-threaded ) 。

**进程**：进程是操作系统资源分配的基本单位，是程序的一次执行过程。进程间隔离性强，较安全但是进程间通信效率低；

**线程**：线程是CPU进行调度的基本单位，是进程内的一个执行单元。进程间可以共享内存，便于通信但是伴随着风险，同时线程调度也要耗费一定资源；

运行一个 Java 程序，开启一个 JVM 就是一个进程，这个进程包含 main 线程和 GC 线程等，另外还有一些守护线程，当一个Java程序中只剩下守护线程时，JVM就会退出。；

### OS线程的实现方式

操作系统实现线程的方式可以分为：用户级线程、内核级线程、混合线程。

**用户级线程(1:N)**：在用户空间实现，对外不可见，内核只能看到一个进程。由进程自己维护线程的调度，优点是避免了像内核级线程调度一样陷入内核态引起的上下文切换；缺点是实现复杂，且无法发挥多核CPU的优势，该进程只能得到一个CPU核心，并且当某一线程进行系统调用时，该进程都会被阻塞，其他能执行的线程也将无法执行。

**内核级线程(1:1)**：内核负责线程调度，将各线程映射到处理器上，优点是在多核CPU中可以并行执行，缺点是和用户级线程相比花费开销较大。

**混合线程(M:N)**：在每个内核线程（又称为轻量级进程）中又实现了用户级线程（又称为协程、纤程），如 go 的协程。

**注意：**Java 中的线程是映射到 OS 中的内核级线程。

### 线程状态

<img src="D:/OneDrive/_mine/docsify/_img/g2Qt5IlyFOWy.png" style="zoom:75%;" />

1. New 新创建：即 new Thread(r) 后还没有调用start方法时，线程还没有开始运行；

2. Runnable 可运行：线程处在随时可以运行的状态，但可能在运行也可能没有运行，取决于是否得到了时间片；

3. Blocked 被阻塞 ；

4. Waiting 等待；

5. Timed Waiting 计时等待；

6. Terminated 被终止：run方法正常结束，或因为一个没有捕获的异常导致run方法终止，则线程死亡，异常终止后会自动释放持有的锁。

当线程处于被阻塞或等待状态时，它暂时不活动 ，它不运行任何代码且消耗最少的资源，直到线程调度器重新激活它。 

当一个线程试图获得另一个线程持有的锁时，进入**阻塞**；

当线程等待另一个线程通知调度器一个**条件**时（线程通信），进入**等待**；

当调用了`Thread.sleep`、`Object.wait`、`Thread.join`、`Lock.tryLock`以及`Condition.await`并添加时间参数时进入**计时等待**；

### 线程属性

1. 优先级：Java中使用`setPriority`方法设置线程优先级在[1,10]之间越来越高，默认为5；

   - 线程优先级高度依赖于宿主机系统的实现，即Java将优先级映射到宿主机的线程实现机制上，例如 Windows 有7个优先级，Oracle为Linux提供的 JVM 中没有线程优先级；

   - 若高优先级的线程没有进入非活动状态，那么低优先级的线程可能会饿死；

2. 守护线程：在线程**启动之前**调用`thread.setDaemon(true)`声明该线程为守护线程，守护线程唯一的作用是为其他线程提供服务，例如计时线程；

### 线程的创建

一般有这几种方法：

1. 继承Thread类或实现Runnable接口，重写其run方法。
2. 关于callable接口，一般与线程池连用。

注意：启动线程需要调用 start() 方法，如果直接调用 run() 方法，那么会被当做普通方法执行。



## Java中的锁

### 几个基础的同步方法

#### synchronized

**原理**：基于“管程”(Monitor)实现，管程是有OS提供的基于信号量的高级同步原语，目的是方便编程，由OS负责加锁、释放锁。

**使用**：可以加在代码块、方法声明上；锁方法时，对于普通方法，锁是this；对于static方法，锁是该类的class对象。

**注意**：要使用同一个 Runnable 的实例 target 去创建多线程，才能使多线程锁到同一个对象上。不要锁不可变类如 String、包装类。

**锁优化**：synchronized 基于OS信号量实现，加锁解锁需要陷入内核态，属于重量级锁。JDK 1.6 时对其进行了优化，加入了锁升级的过程，自旋 --> 偏向锁 --> 轻量级锁 -->重量级锁，前三个阶段都使用到了CAS操作，相当于无锁。锁的降级条件比较苛刻，可以理解为无法降级。



#### volatile

**作用**：用于声明禁用缓存和指令重排序，只能保证有序性和可见性，无法保证原子性。

**原理**：内存屏障。



#### Thread.sleep(ms)

- 只会让出 CPU 时间片，不会释放其他任何资源，可以指定时间，时间过后进入就绪态，重新参与时间片竞争。sleep(0) 可以看成一个运行态的进程产生一个中断，由运行态直接转入就绪态。



#### Thread.yield()

- 将当前线程由运行态转换为就绪态，重新参与时间片竞争，与 sleep(0) 效果相似。



#### join(ms)

- join 方法本质是**让调用线程 wait 在当前被等待线程的实例对象上**，被等待线程退出之前会调用 notifyAll() 方法唤醒所有等待在自己身上的线程。

- 可以传一个毫秒值作为参数，超时后会自动取消等待，继续往下执行。



#### wait() & notify()

**必须在同步块中调用**，在 synchronized(lock){ } 中调用 lock.wait() 会使当前线程**释放目标对象的锁并进入到 lock 的等待队列**，当调用 lock.notify() 时，在队列中随机选择一个唤醒，因此是不公平的。lock.notifyAll() 会唤醒所有等待在 lock 上的线程。



#### ReentrantLock

- 一个线程不能连续两次持有相同的锁，称为**不可重入**。

#### Semaphore

- 允许多个线程同时访问临界区资源。



#### stop()、resume()、suspend() 

这三个方法都已被废弃。

- stop() 会强制停止一个线程，会导致不一致性。
- suspend()  用于挂起一个线程，resume() 用于唤醒，如果执行顺序发生改变，那么该线程将永远是挂起状态。

#### interrupt

stop()、resume()、suspend() 被废弃，取而代之的是 interrupt()，当我们试图停止一个线程时，应该以中断的方式通知该线程，至于是停止还是继续运行，由该线程自己决定。

- `public void interrupt()`
  - Unless the current thread is interrupting itself, which is always permitted, the checkAccess method of this thread is invoked, which may cause a SecurityException to be thrown.
  - If this thread is blocked in an invocation of the wait(), wait(long), or wait(long, int) methods of the Object class, or of the join(), join(long), join(long, int), sleep(long), or sleep(long, int), methods of this class, then its interrupt status will be cleared and it will receive an InterruptedException.
  - If this thread is blocked in an I/O operation upon an InterruptibleChannel then the channel will be closed, the thread's interrupt status will be set, and the thread will receive a ClosedByInterruptException.
  - If this thread is blocked in a Selector then the thread's interrupt status will be set and it will return immediately from the selection operation, possibly with a non-zero value, just as if the selector's wakeup method were invoked.
  - If none of the previous conditions hold then this thread's interrupt status will be set.
  - Interrupting a thread that is not alive need not have any effect.
  - 重点：wait、sleep、join 中会被直接唤醒并转为就绪状态，如果能够得到运行，那么将抛出 InterruptedException，此时 interrupt 标志位仍为 false。如果被阻塞，那么将置 interrupt 为true。

- `public boolean isInterrupted()`
  - 返回**是否被中断过**的标志位，由于线程在中断时未处于活动状态而被忽略的线程中断将通过此方法返回 false 来反映。
- `public static boolean interrupted()`
  - 作用和 `isInterrupted()` 相同，区别在于每次调用**会将标志位置为 false**。

## CPU中的锁

> 原子性、有序性、可见性，保证了这三条就不会出错。

### 原子性

单条指令可以完成的操作可以视为**原子操作**，因为中断只能发生于指令间。在对临界资源进行访问时需要加锁，尤其是多核处理器中，还需要**锁住总线**，屏蔽其他线程的访存操作。

**原语**是由若干多机器指令构成的完成某种特定功能的一段程序，具有不可分割性。

### 有序性

CPU 会对指令进行重排序来提高性能，有时会产生意想不到的结果，这时就要使用到**内存屏障**。内存屏障分为读屏障、写屏障、读写屏障三类，保证内存屏障后面的对应操作不会被重排序优化到屏障之前执行。Volatile 的实现就使用到了内存屏障。

### 可见性

现代 CPU 拥有多级缓存，L1、L2 为每个 core 私有，L3 为 CPU 中所有 core 共享。为了保证 CPU 缓存的一致性，有很多的缓存一致性协议，Intel 使用的是 `MESI 协议`（`Modified Exclusive Shared Or Invalid`）。缓存一致性协议将锁内存的**总线锁细化到缓存行锁**，大大提高了性能。

#### MESI

MESI 标志着缓存行的四种状态：

1. 被修改 **Modified**：当前缓存行是脏的。
   - 当其他 core 要读主存该部分数据时，该缓存行必须写回主存，状态转为 S。
2. 独享的 **Exclusive**：当前缓存行是当前 core 独占的，且是干净的。
   - 当其他 core 要进行相应读取时，状态转为 S；
   - 当进行修改操作后，状态转为 M。
3. 共享的 **Shared**：当前缓存行同时存在于多个 core 的缓存中。
   - 当其中一个 core 进行修改时，该 core 缓存行状态转为 M，其他缓存行状态转为 I。
   - 假设两个 core 对应缓存行状态为 S，其中一个将其作废，另一个仍是 S，并不会变为 E。
4. 失效的 **Invalid**：当前缓存行是无效的。

更详细的状态转换看 wiki 吧：[MESI 协议 wiki](https://zh.wikipedia.org/wiki/MESI%E5%8D%8F%E8%AE%AE)

#### 伪共享

[伪共享（false sharing），并发编程无声的性能杀手](https://www.cnblogs.com/cyfonly/p/5800758.html)

CPU 缓存中每 64 bit 称为一个**缓存行**（cache line），是 CPU 对缓存进行读写的单位。

假设有两个 4 字节 int 类型 a、b 放在了同一缓存行，线程 A、B 同时分别对 a、b 进行读写操作，CPU 不会只读半字，而是将整个缓存行都读进来。这时就会发生**伪共享**。A、B 对 a、b 的修改操作会导致整个缓存行都失效。

可以通过 padding 来防止伪共享，但需要注意的是 CPU 缓存是很昂贵的。

