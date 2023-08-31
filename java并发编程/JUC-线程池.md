# 提出背景

以往都是使用Thread类来创建一个新的线程：

```java
public static void main(String[] args) {
    Thread t = new Thread(() -> System.out.println("Hello World!"));
    t.start();
}
```

由于线程的创建和销毁也需要占用系统资源，如果程序频繁地创建线程，会降低整个程序的性能

优化方法是将已创建的线程复用，利用池化技术，就像数据库连接池一样，创建很多个线程，然后反复地使用这些线程，而不对它们进行销毁。

比如Tomcat服务器，要在同一时间接受和处理大量的请求，那么就必须要在短时间内创建大量的线程，结束后又进行销毁，这显然会导致很大的开销，因此这种情况下使用线程池显然是更好的解决方案。

由于线程池可以反复利用已有线程执行多线程操作，所以它一般是有容量限制的，当所有的线程都处于工作状态时，那么新的多线程请求会被阻塞，直到有一个线程空闲出来为止，实际上这里就会用到阻塞队列。

# ThreadPoolExecutor线程池

## 创建线程池

直接创建一个新的线程池对象，它已经提前实现好了线程的调度机制
先来看它的构造方法：
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

* corePoolSize：**核心线程池大小**。每向线程池提交一个多线程任务时，无论是否存在其他空闲线程，都会创建一个新的`核心线程`，直到到达核心线程池大小为止，之后会尝试复用线程资源。当然也可以在一开始就全部初始化好，调用` prestartAllCoreThreads()`即可。
* maximumPoolSize：**最大线程池大小**，当目前线程池中所有的线程都处于运行状态，并且等待队列已满，那么就会直接尝试继续创建新的`非核心线程`运行，但是不能超过最大线程池大小。
* keepAliveTime：**线程最大空闲时间**，当一个`非核心线程`空闲超过一定时间，会自动销毁。
* unit：**线程最大空闲时间的时间单位**
* workQueue：**线程等待队列**，当线程池中核心线程数已满时，就会将任务暂时存到等待队列中，直到有线程资源可用为止，这里可以使用阻塞队列。
* threadFactory：**线程创建工厂**，我们可以干涉线程池中线程的创建过程，进行自定义。
* handler：**拒绝策略**，当等待队列和线程池都没有空间了，真的不能再来新的任务时，来了个新的多线程任务，那么只能拒绝了，这时就会根据当前设定的拒绝策略进行处理。

## 线程池大小

最为重要的就是线程池大小的限定了，合理地分配大小会使得线程池的执行效率事半功倍：
首先分析一下，线程池执行任务的特性，是CPU 密集型还是 IO 密集型
  * **CPU密集型**：主要执行计算任务，响应时间很快，CPU一直在运行，这种任务CPU的利用率很高，那么线程数应该是根据 CPU 核心数来决定，CPU 核心数 = 最大同时执行线程数。以 i5-9400F 处理器为例，CPU 核心数为 6，那么最多就能同时执行 6 个线程。
  * **IO密集型**：主要是进行 IO 操作，因为执行 IO 操作的时间比较较长，比如从硬盘读取数据之类的，CPU就得等着IO操作，很容易出现空闲状态，导致 CPU 的利用率不高，这种情况下可以适当增加线程池的大小，让更多的线程可以一起进行IO操作，一般可以配置为CPU核心数的2倍。

***

手动创建一个新的线程池看看效果：
```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor =
            new ThreadPoolExecutor(2, 4,   //2个核心线程，最大线程数为4个
                    3, TimeUnit.SECONDS,        //最大空闲时间为3秒钟
                    new ArrayBlockingQueue<>(2));     //这里使用容量为2的ArrayBlockingQueue队列

    for (int i = 0; i < 6; i++) {   //开始6个任务
        int finalI = i;
        executor.execute(() -> {
            try {
                System.out.println(Thread.currentThread().getName()+" 开始执行！（"+ finalI);
                TimeUnit.SECONDS.sleep(1);
                System.out.println(Thread.currentThread().getName()+" 已结束！（"+finalI);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }

    TimeUnit.SECONDS.sleep(1);    //看看当前线程池中的线程数量
    System.out.println("线程池中线程数量："+executor.getPoolSize());
    TimeUnit.SECONDS.sleep(5);     //等到超过空闲时间
    System.out.println("线程池中线程数量："+executor.getPoolSize());

    executor.shutdownNow();    //使用完线程池记得关闭，不然程序不会结束，它会取消所有等待中的任务以及试图中断正在执行的任务，关闭后，无法再提交任务，一律拒绝
  	//executor.shutdown();     同样可以关闭，但是会执行完等待队列中的任务再关闭
}
```

这里创建了一个核心容量为2，最大容量为4，等待队列长度为2，空闲时间为3秒的线程池。
现在向其中执行6个任务，每个任务都会进行1秒钟休眠，那么当线程池中2个核心线程都被占用时，还有4个任务就只能进入到等待队列中了，但是等待队列中只有2个容量，这时紧接着的2个任务，线程池将直接尝试创建线程，由于不大于最大容量，因此可以成功创建。最后所有线程完成之后，在等待5秒后，超过了线程池的最大空闲时间，`非核心线程`被回收了，所以线程池中只有2个线程存在。

***
如果等待队列设定为没有容量的SynchronousQueue会发生什么？

```java
pool-1-thread-1 开始执行！（0
pool-1-thread-4 开始执行！（3
pool-1-thread-3 开始执行！（2
pool-1-thread-2 开始执行！（1
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task com.test.Main$$Lambda$1/1283928880@682a0b20 rejected from java.util.concurrent.ThreadPoolExecutor@3d075dc0[Running, pool size = 4, active threads = 4, queued tasks = 0, completed tasks = 0]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2063)
	at java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:830)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1379)
	at com.test.Main.main(Main.java:15)
```

可以看到，前4个任务都可以正常执行，但是到第五个任务时，直接抛出了异常，这其实就是因为等待队列的容量为0，相当于没有容量，那么这个时候，就只能拒绝任务了，拒绝的操作会根据拒绝策略决定。

## 拒绝策略

线程池的拒绝策略默认有以下几个：

* AbortPolicy(默认)：像上面一样，直接抛异常。
* CallerRunsPolicy：直接让提交任务的线程运行这个任务，比如在主线程向线程池提交了任务，那么就直接由主线程执行。
* DiscardOldestPolicy：丢弃队列中最近的一个任务，替换为当前任务。
* DiscardPolicy：什么也不用做。

***
**测试CallerRunsPolicy策略**
```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor =
            new ThreadPoolExecutor(2, 4,
                    3, TimeUnit.SECONDS,
                    new SynchronousQueue<>(),
                    new ThreadPoolExecutor.CallerRunsPolicy());   //使用CallerRunsPolicy策略
```

CallerRunsPolicy策略是谁提交的谁自己执行：
可以看到，当队列塞不下时，直接在主线程运行任务，运行完之后再继续向下执行

```java
pool-1-thread-1 开始执行！（0
pool-1-thread-2 开始执行！（1
main 开始执行！（4
pool-1-thread-4 开始执行！（3
pool-1-thread-3 开始执行！（2
pool-1-thread-3 已结束！（2
pool-1-thread-2 已结束！（1
pool-1-thread-1 已结束！（0
main 已结束！（4
pool-1-thread-4 已结束！（3
pool-1-thread-1 开始执行！（5
pool-1-thread-1 已结束！（5
线程池中线程数量：4
线程池中线程数量：2
```

***
**测试DiscardOldestPolicy策略**
把策略修改为DiscardOldestPolicy：

```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor =
            new ThreadPoolExecutor(2, 4,
                    3, TimeUnit.SECONDS,
                    new ArrayBlockingQueue<>(1),    //这里设置为ArrayBlockingQueue，长度为1
                    new ThreadPoolExecutor.DiscardOldestPolicy());   
```

它会移除等待队列中的最近的一个任务，所以可以看到有一个任务实际上是被抛弃了的：
```
pool-1-thread-1 开始执行！（0
pool-1-thread-4 开始执行！（4
pool-1-thread-3 开始执行！（3
pool-1-thread-2 开始执行！（1
pool-1-thread-1 已结束！（0
pool-1-thread-4 已结束！（4
pool-1-thread-1 开始执行！（5
线程池中线程数量：4
pool-1-thread-3 已结束！（3
pool-1-thread-2 已结束！（1
pool-1-thread-1 已结束！（5
线程池中线程数量：2
```

如果选择没有容量的SynchronousQueue作为等待队列会爆栈：
```java
pool-1-thread-1 开始执行！（0
pool-1-thread-3 开始执行！（2
pool-1-thread-2 开始执行！（1
pool-1-thread-4 开始执行！（3
Exception in thread "main" java.lang.StackOverflowError
	at java.util.concurrent.SynchronousQueue.offer(SynchronousQueue.java:912)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1371)	
	...
pool-1-thread-1 已结束！（0
pool-1-thread-2 已结束！（1
pool-1-thread-4 已结束！（3
pool-1-thread-3 已结束！（2
```

这是为什么呢？来看看DiscardOldestPolicy拒绝策略的源码：

```java
public static class DiscardOldestPolicy implements RejectedExecutionHandler {
    public DiscardOldestPolicy() { }

    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();   //会先执行一次出队操作，但是这对于SynchronousQueue来说毫无意义
            e.execute(r);     //这里会再次调用execute方法
        }
    }
}
```

DiscardOldestPolicy策略会先对等待队列进行出队操作，但是由于SynchronousQueue压根没容量，所以这个操作毫无意义，然后就会递归执行`execute`方法，而进入之后，又发现没有容量不能插入，于是又重复上面的操作，这样就会无限的递归下去，最后就爆栈了。

***
**自定义策略**
除了使用官方提供的4种策略之外，还可以使用自定义的策略：
```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor =
            new ThreadPoolExecutor(2, 4,
                    3, TimeUnit.SECONDS,
                    new SynchronousQueue<>(),
                    (r, executor1) -> {   //比如实现一个就在当前线程执行的策略
                        System.out.println("哎呀，线程池和等待队列都满了，你自己耗子尾汁吧");
                        r.run();   //直接运行
                    });
```

## 线程创建工厂

线程创建工厂可以自定义如何创建新的线程
这里传入的Runnable对象就是提交的任务，需要返回一个Thread对象，而如何创建这个对象，以及它的一些属性，就都由我们来决定。
```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor =
            new ThreadPoolExecutor(2, 4,
                    3, TimeUnit.SECONDS,
                    new SynchronousQueue<>(),
                    new ThreadFactory() {
                        int counter = 0;
                        @Override
                        public Thread newThread(Runnable r) {
                            return new Thread(r, "我的自定义线程-"+counter++);
                        }
                    });

    for (int i = 0; i < 4; i++) {
        executor.execute(() -> System.out.println(Thread.currentThread().getName()+" 开始执行！"));
    }
}
```

如果任务在运行过程中出现异常了，那么是不是会导致线程池中的线程被销毁呢？
出现异常之后，再次提交新的任务，执行的线程是一个新的线程了
```java
public static void main(String[] args) throws InterruptedException {
    ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1,   //最大容量和核心容量锁定为1
            0, TimeUnit.MILLISECONDS, new LinkedBlockingDeque<>());
    executor.execute(() -> {
        System.out.println(Thread.currentThread().getName());
        throw new RuntimeException("我是异常！");
    });
    TimeUnit.SECONDS.sleep(1);
    executor.execute(() -> {
        System.out.println(Thread.currentThread().getName());
    });
}
```

# Executors工具类

除了自己创建线程池之外，官方也提供了很多的线程池定义，可以使用`Executors`工具类来快速创建线程池

## newFixedThreadPool

newFixedThreadPool创建一个固定容量的线程池
```java
public static void main(String[] args) throws InterruptedException {
    ExecutorService executor = Executors.newFixedThreadPool(2);   //直接创建一个固定容量的线程池
}
```

它的内部实现为：
这里直接将最大线程和核心线程数量设定为一样的，并且等待时间为0，因为压根不需要，并且采用的是一个无界的LinkedBlockingQueue作为等待队列。
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

## newSingleThreadExecutor

newSingleThreadExecutor创建只有一个线程的线程池：
```java
public static void main(String[] args) throws InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    //创建一个只有一个线程的线程池
}
```

原理如下：
这里并不是直接创建的一个ThreadPoolExecutor对象，而是套了一层FinalizableDelegatedExecutorService
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

```java
static class FinalizableDelegatedExecutorService
    extends DelegatedExecutorService {
    FinalizableDelegatedExecutorService(ExecutorService executor) {
        super(executor);
    }
    protected void finalize() {    //在GC时，会执行finalize方法，此方法中会关闭掉线程池，释放资源
        super.shutdown();
    }
}
```

```java
static class DelegatedExecutorService extends AbstractExecutorService {
    private final ExecutorService e;    //被委派对象
    DelegatedExecutorService(ExecutorService executor) { e = executor; }   //实际上所以的操作都是让委派对象执行的，有点像代理
    public void execute(Runnable command) { e.execute(command); }
    public void shutdown() { e.shutdown(); }
    public List<Runnable> shutdownNow() { return e.shutdownNow(); }
```

所以，下面两种写法的区别在于：
前者实际上是被代理了，没办法直接修改前者的相关属性，显然使用前者创建只有一个线程的线程池更加专业和安全（可以防止属性被修改）一些。
```java
public static void main(String[] args) throws InterruptedException {
    ExecutorService executor1 = Executors.newSingleThreadExecutor();
    ExecutorService executor2 = Executors.newFixedThreadPool(1);
}
```

## newCachedThreadPool

newCachedThreadPool是一个会根据需要无限制创建新线程的线程池
```java
public static void main(String[] args) throws InterruptedException {
    ExecutorService executor = Executors.newCachedThreadPool();
    //它是一个会根据需要无限制创建新线程的线程池
}
```

它的实现：
核心线程数为0，也就是说所有的线程都是`非核心线程`，也就是说线程空闲时间超过1秒钟，一律销毁。
但是它的最大容量是`Integer.MAX_VALUE`，也就是说，它可以无限制地增长下去，所以这玩意一定要慎用。
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

# 执行带返回值的任务

当需要在任务执行之后得到一个结果时，就可以使用到Future，通过它来获取任务的结果以及任务当前是否完成：
```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
    ExecutorService executor = Executors.newSingleThreadExecutor();   //直接用Executors创建，方便就完事了
    Future<String> future = executor.submit(() -> "我是字符串!");     //使用submit提交任务，会返回一个Future对象，注意提交的对象可以是Runable也可以是Callable，这里使用的是Callable能够自定义返回值
    System.out.println(future.get());    //如果任务未完成，get会被阻塞，任务完成返回Callable执行结果返回值
    executor.shutdown();
}
```

结果也可以一开始就定义好，然后等待Runnable执行完之后再返回：
```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<String> future = executor.submit(() -> {
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, "我是字符串！");
    System.out.println(future.get());
    executor.shutdown();
}
```

还可以通过传入FutureTask对象的方式：
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService service = Executors.newSingleThreadExecutor();
    FutureTask<String> task = new FutureTask<>(() -> "我是字符串！");
    service.submit(task);
    System.out.println(task.get());
    executor.shutdown();
}
```

还通过Future对象获取当前任务的一些状态：
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<String> future = executor.submit(() -> "都看到这里了，不赏UP主一个一键三连吗？");
    System.out.println(future.get());
    System.out.println("任务是否执行完成："+future.isDone());
    System.out.println("任务是否被取消："+future.isCancelled());
    executor.shutdown();
}
```

在任务执行途中取消任务：
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<String> future = executor.submit(() -> {
        TimeUnit.SECONDS.sleep(10);
        return "这次一定！";
    });
    System.out.println(future.cancel(true));
    System.out.println(future.isCancelled());
    executor.shutdown();
}
```

# 执行定时任务

在之前，如果需要执行一个定时任务，就要用到Timer和TimerTask。但是它只会创建一个线程来处理定时任务，无法实现多线程调度，并且它无法处理异常情况，一旦抛出未捕获异常就会直接终止。

JDK5之后，可以使用ScheduledThreadPoolExecutor来提交定时任务，它继承自ThreadPoolExecutor，并且所有的构造方法都必须要求最大线程池容量为Integer.MAX_VALUE，并且都是采用的DelayedWorkQueue作为等待队列。

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}

public ScheduledThreadPoolExecutor(int corePoolSize,
                                   ThreadFactory threadFactory) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue(), threadFactory);
}

public ScheduledThreadPoolExecutor(int corePoolSize,
                                   RejectedExecutionHandler handler) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue(), handler);
}

public ScheduledThreadPoolExecutor(int corePoolSize,
                                   ThreadFactory threadFactory,
                                   RejectedExecutionHandler handler) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue(), threadFactory, handler);
}
```

`schedule`方法可以提交一个延时任务，在指定时间之后才会开始：
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    //直接设定核心线程数为1
    ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(1);
    //在3秒后执行
    executor.schedule(() -> System.out.println("HelloWorld!"), 3, TimeUnit.SECONDS);

    executor.shutdown();
}
```

使用`ScheduledFuture`对象接收返回值：
`schedule`方法返回了一个ScheduledFuture对象，和Future一样，它也支持返回值的获取、包括对任务的取消同时还支持获取剩余等待时间。
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(2);
    //这里使用ScheduledFuture
    ScheduledFuture<String> future = executor.schedule(() -> "????", 3, TimeUnit.SECONDS);
    System.out.println("任务剩余等待时间："+future.getDelay(TimeUnit.MILLISECONDS) / 1000.0 + "s");
    System.out.println("任务执行结果："+future.get());
    executor.shutdown();
}
```

`scheduleAtFixedRate`方法可以按照给定的频率不断执行任务
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(2);
    executor.scheduleAtFixedRate(() -> System.out.println("Hello World!"),
            3, 1, TimeUnit.SECONDS);
  	//三秒钟延迟开始，之后每隔一秒钟执行一次
}
```

Executors预置了`newScheduledThreadPool`方法用于创建线程池：
```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    ScheduledExecutorService service = Executors.newScheduledThreadPool(1);
    service.schedule(() -> System.out.println("Hello World!"), 1, TimeUnit.SECONDS);
}
```

# 线程池实现原理

前面我们了解了线程池的使用，那么接着我们来看看它的详细实现过程，结构稍微有点复杂，坐稳，发车了。

这里需要首先介绍一下ctl变量：

```java
//这个变量比较关键，用到了原子AtomicInteger，用于同时保存线程池运行状态和线程数量（使用原子类是为了保证原子性）
//它是通过拆分32个bit位来保存数据的，前3位保存状态，后29位保存工作线程数量（那要是工作线程数量29位装不下不就GG？）
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;    //29位，线程数量位
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;   //计算得出最大容量（1左移29位，最大容量为2的29次方-1）

// 所有的运行状态，注意都是只占用前3位，不会占用后29位
// 接收新任务，并等待执行队列中的任务
private static final int RUNNING    = -1 << COUNT_BITS;   //111 | 0000... (后29数量位，下同)
// 不接收新任务，但是依然等待执行队列中的任务
private static final int SHUTDOWN   =  0 << COUNT_BITS;   //000 | 数量位
// 不接收新任务，也不执行队列中的任务，并且还要中断正在执行中的任务
private static final int STOP       =  1 << COUNT_BITS;   //001 | 数量位
// 所有的任务都已结束，线程数量为0，即将完全关闭
private static final int TIDYING    =  2 << COUNT_BITS;   //010 | 数量位
// 完全关闭
private static final int TERMINATED =  3 << COUNT_BITS;   //011 | 数量位

// 封装和解析ctl变量的一些方法
private static int runStateOf(int c)     { return c & ~CAPACITY; }   //对CAPACITY取反就是后29位全部为0，前三位全部为1，接着与c进行与运算，这样就可以只得到前三位的结果了，所以这里是取运行状态
private static int workerCountOf(int c)  { return c & CAPACITY; }
//同上，这里是为了得到后29位的结果，所以这里是取线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }   
// 比如上面的RUNNING, 0，进行与运算之后：
// 111 | 0000000000000000000000000
```

![image-20230306172347411](https://s2.loli.net/2023/03/06/yoFvxQJq84G6dcH.png)

我们先从最简单的入手，看看在调用`execute`方法之后，线程池会做些什么：

```java
//这个就是我们指定的阻塞队列
private final BlockingQueue<Runnable> workQueue;

//再次提醒，这里没加锁！！该有什么意识不用我说了吧，所以说ctl才会使用原子类。
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();     //如果任务为null，那执行个寂寞，所以说直接空指针
    int c = ctl.get();      //获取ctl的值，一会要读取信息的
    if (workerCountOf(c) < corePoolSize) {   //判断工作线程数量是否小于核心线程数
        if (addWorker(command, true))    //如果是，那不管三七二十一，直接加新的线程执行，然后返回即可
            return;
        c = ctl.get();    //如果线程添加失败（有可能其他线程也在对线程池进行操作），那就更新一下c的值
    }
    if (isRunning(c) && workQueue.offer(command)) {   //继续判断，如果当前线程池是运行状态，那就尝试向阻塞队列中添加一个新的等待任务
        int recheck = ctl.get();   //再次获取ctl的值
        if (! isRunning(recheck) && remove(command))   //这里是再次确认当前线程池是否关闭，如果添加等待任务后线程池关闭了，那就把刚刚加进去任务的又拿出来
            reject(command);   //然后直接拒绝当前任务的提交（会根据我们的拒绝策略决定如何进行拒绝操作）
        else if (workerCountOf(recheck) == 0)   //如果这个时候线程池依然在运行状态，那么就检查一下当前工作线程数是否为0，如果是那就直接添加新线程执行
            addWorker(null, false);   //添加一个新的非核心线程，但是注意没添加任务
      	//其他情况就啥也不用做了
    }
    else if (!addWorker(command, false))   //这种情况要么就是线程池没有运行，要么就是队列满了，按照我们之前的规则，核心线程数已满且队列已满，那么会直接添加新的非核心线程，但是如果已经添加到最大数量，这里肯定是会失败的
        reject(command);   //确实装不下了，只能拒绝
}
```

是不是感觉思路还挺清晰的，我们接着来看`addWorker`是怎么创建和执行任务的，又是一大堆代码：

```java
private boolean addWorker(Runnable firstTask, boolean core) {
  	//这里给最外层循环打了个标签，方便一会的跳转操作
    retry:
    for (;;) {    //无限循环，老套路了，注意这里全程没加锁
        int c = ctl.get();     //获取ctl值
        int rs = runStateOf(c);    //解析当前的运行状态

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&   //判断线程池是否不是处于运行状态
            ! (rs == SHUTDOWN &&   //如果不是运行状态，判断线程是SHUTDOWN状态并、任务不为null、等待队列不为空，只要有其中一者不满足，直接返回false，添加失败
               firstTask == null &&   
               ! workQueue.isEmpty()))
            return false;

        for (;;) {   //内层又一轮无限循环，这个循环是为了将线程计数增加，然后才可以真正地添加一个新的线程
            int wc = workerCountOf(c);    //解析当前的工作线程数量
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))    //判断一下还装得下不，如果装得下，看看是核心线程还是非核心线程，如果是核心线程，不能大于核心线程数的限制，如果是非核心线程，不能大于最大线程数限制
                return false;
            if (compareAndIncrementWorkerCount(c))    //CAS自增线程计数，如果增加成功，任务完成，直接跳出继续
                break retry;    //注意这里要直接跳出最外层循环，所以用到了标签（类似于goto语句）
            c = ctl.get();  // 如果CAS失败，更新一下c的值
            if (runStateOf(c) != rs)    //如果CAS失败的原因是因为线程池状态和一开始的不一样了，那么就重新从外层循环再来一次
                continue retry;    //注意这里要直接从最外层循环继续，所以用到了标签（类似于goto语句）
            // 如果是其他原因导致的CAS失败，那只可能是其他线程同时在自增，所以重新再来一次内层循环
        }
    }

  	//好了，线程计数自增也完了，接着就是添加新的工作线程了
    boolean workerStarted = false;   //工作线程是否已启动
    boolean workerAdded = false;    //工作线程是否已添加
    Worker w = null;     //暂时理解为工作线程，别急，我们之后会解读Worker类
    try {
        w = new Worker(firstTask);     //创建新的工作线程，传入我们提交的任务
        final Thread t = w.thread;    //拿到工作线程中封装的Thread对象
        if (t != null) {      //如果线程不为null，那就可以安排干活了
            final ReentrantLock mainLock = this.mainLock;      //又是ReentrantLock加锁环节，这里开始就是只有一个线程能进入了
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());    //获取当前线程的运行状态

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {    //只有当前线程池是正在运行状态，或是SHUTDOWN状态且firstTask为空，那么就继续
                    if (t.isAlive()) // 检查一下线程是否正在运行状态
                        throw new IllegalThreadStateException();   //如果是那肯定是不能运行我们的任务的
                    workers.add(w);    //直接将新创建的Work丢进 workers 集合中
                    int s = workers.size();   //看看当前workers的大小
                    if (s > largestPoolSize)   //这里是记录线程池运行以来，历史上的最多线程数
                        largestPoolSize = s;
                    workerAdded = true;   //工作线程已添加
                }
            } finally {
                mainLock.unlock();   //解锁
            }
            if (workerAdded) {
                t.start();   //启动线程
                workerStarted = true;  //工作线程已启动
            }
        }
    } finally {
        if (! workerStarted)    //如果线程在上面的启动过程中失败了
            addWorkerFailed(w);    //将w移出workers并将计数器-1，最后如果线程池是终止状态，会尝试加速终止线程池
    }
    return workerStarted;   //返回是否成功
}
```

接着我们来看Worker类是如何实现的，它继承自AbstractQueuedSynchronizer，时隔两章，居然再次遇到AQS，那也就是说，它本身就是一把锁：

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable {
    //用来干活的线程
    final Thread thread;
    //要执行的第一个任务，构造时就确定了的
    Runnable firstTask;
    //干活数量计数器，也就是这个线程完成了多少个任务
    volatile long completedTasks;

    Worker(Runnable firstTask) {
        setState(-1); // 执行Task之前不让中断，将AQS的state设定为-1
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);   //通过预定义或是我们自定义的线程工厂创建线程
    }
  
    public void run() {
        runWorker(this);   //真正开始干活，包括当前活干完了又要等新的活来，就从这里开始，一会详细介绍
    }

   	//0就是没加锁，1就是已加锁
    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    ...
}
```

最后我们来看看一个Worker到底是怎么在进行任务的：

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();   //获取当前线程
    Runnable task = w.firstTask;    //取出要执行的任务
    w.firstTask = null;   //然后把Worker中的任务设定为null
    w.unlock(); // 因为一开始为-1，这里是通过unlock操作将其修改回0，只有state大于等于0才能响应中断
    boolean completedAbruptly = true;
    try {
      	//只要任务不为null，或是任务为空但是可以从等待队列中取出任务不为空，那么就开始执行这个任务，注意这里是无限循环，也就是说如果当前没有任务了，那么会在getTask方法中卡住，因为要从阻塞队列中等着取任务
        while (task != null || (task = getTask()) != null) {
            w.lock();    //对当前Worker加锁，这里其实并不是防其他线程，而是在shutdown时保护此任务的运行
            
          //由于线程池在STOP状态及以上会禁止新线程加入并且中断正在进行的线程
            if ((runStateAtLeast(ctl.get(), STOP) ||   //只要线程池是STOP及以上的状态，那肯定是不能开始新任务的
                 (Thread.interrupted() &&    					 //线程是否已经被打上中断标记并且线程一定是STOP及以上
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())   //再次确保线程被没有打上中断标记
                wt.interrupt();     //打中断标记
            try {
                beforeExecute(wt, task);  //开始之前的准备工作，这里暂时没有实现
                Throwable thrown = null;
                try {
                    task.run();    //OK，开始执行任务
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);    //执行之后的工作，也没实现
                }
            } finally {
                task = null;    //任务已完成，不需要了
                w.completedTasks++;   //任务完成数++
                w.unlock();    //解锁
            }
        }
        completedAbruptly = false;
    } finally {
      	//如果能走到这一步，那说明上面的循环肯定是跳出了，也就是说这个Worker可以丢弃了
      	//所以这里会直接将 Worker 从 workers 里删除掉
        processWorkerExit(w, completedAbruptly);
    }
}
```

那么它是怎么从阻塞队列里面获取任务的呢：

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {    //无限循环获取
        int c = ctl.get();   //获取ctl 
        int rs = runStateOf(c);      //解析线程池运行状态

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {      //判断是不是没有必要再执行等待队列中的任务了，也就是处于关闭线程池的状态了
            decrementWorkerCount();     //直接减少一个工作线程数量
            return null;    //返回null，这样上面的runWorker就直接结束了，下同
        }

        int wc = workerCountOf(c);   //如果线程池运行正常，那就获取当前的工作线程数量

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;   //如果线程数大于核心线程数或是允许核心线程等待超时，那么就标记为可超时的

      	//超时或maximumPoolSize在运行期间被修改了，并且线程数大于1或等待队列为空，那也是不能获取到任务的
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))   //如果CAS减少工作线程成功
                return null;    //返回null
            continue;   //否则开下一轮循环
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :   //如果可超时，那么最多等到超时时间
                workQueue.take();    //如果不可超时，那就一直等着拿任务
            if (r != null)    //如果成功拿到任务，ok，返回
                return r;
            timedOut = true;   //否则就是超时了，下一轮循环将直接返回null
        } catch (InterruptedException retry) {
            timedOut = false;
        }
      	//开下一轮循环吧
    }
}
```

虽然我们的源码解读越来越深，但是只要各位的思路不断，依然是可以继续往下看的。到此，有关`execute()`方法的源码解读，就先到这里。

接着我们来看当线程池关闭时会做什么事情：

```java
//普通的shutdown会继续将等待队列中的线程执行完成后再关闭线程池
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
      	//判断是否有权限终止
        checkShutdownAccess();
      	//CAS将线程池运行状态改为SHUTDOWN状态，还算比较温柔，详细过程看下面
        advanceRunState(SHUTDOWN);
       	//让闲着的线程（比如正在等新的任务）中断，但是并不会影响正在运行的线程，详细过程请看下面
        interruptIdleWorkers();
        onShutdown(); //给ScheduledThreadPoolExecutor提供的钩子方法，就是等ScheduledThreadPoolExecutor去实现的，当前类没有实现
    } finally {
        mainLock.unlock();
    }
    tryTerminate();   //最后尝试终止线程池
}
```

```java
private void advanceRunState(int targetState) {
    for (;;) {
        int c = ctl.get();    //获取ctl
        if (runStateAtLeast(c, targetState) ||    //是否大于等于指定的状态
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))   //CAS设置ctl的值
            break;   //任意一个条件OK就可以结束了
    }
}
```

```java
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;    //拿到Worker中的线程
            if (!t.isInterrupted() && w.tryLock()) {   //先判断一下线程是不是没有被中断然后尝试加锁，但是通过前面的runWorker()源代码我们得知，开始之后是让Worker加了锁的，所以如果线程还在执行任务，那么这里肯定会false
                try {
                    t.interrupt();    //如果走到这里，那么说明线程肯定是一个闲着的线程，直接给中断吧
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();    //解锁
                }
            }
            if (onlyOne)   //如果只针对一个Worker，那么就结束循环
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```

而`shutdownNow()`方法也差不多，但是这里会更直接一些：

```java
//shutdownNow开始后，不仅不允许新的任务到来，也不会再执行等待队列的线程，而且会终止正在执行的线程
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
      	//这里就是直接设定为STOP状态了，不再像shutdown那么温柔
        advanceRunState(STOP);
      	//直接中断所有工作线程，详细过程看下面
        interruptWorkers();
      	//取出仍处于阻塞队列中的线程
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;   //最后返回还没开始的任务
}
```

```java
private void interruptWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers)   //遍历所有Worker
            w.interruptIfStarted();   //无差别对待，一律加中断标记
    } finally {
        mainLock.unlock();
    }
}
```

最后的最后，我们再来看看`tryTerminate()`是怎么完完全全终止掉一个线程池的：

```java
final void tryTerminate() {
    for (;;) {     //无限循环
        int c = ctl.get();    //上来先获取一下ctl值
      	//只要是正在运行 或是 线程池基本上关闭了 或是 处于SHUTDOWN状态且工作队列不为空，那么这时还不能关闭线程池，返回
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
      
      	//走到这里，要么处于SHUTDOWN状态且等待队列为空或是STOP状态
        if (workerCountOf(c) != 0) { // 如果工作线程数不是0，这里也会中断空闲状态下的线程
            interruptIdleWorkers(ONLY_ONE);   //这里最多只中断一个空闲线程，然后返回
            return;
        }

      	//走到这里，工作线程也为空了，可以终止线程池了
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {   //先CAS将状态设定为TIDYING表示基本终止，正在做最后的操作
                try {
                    terminated();   //终止，暂时没有实现
                } finally {
                    ctl.set(ctlOf(TERMINATED, 0));   //最后将状态设定为TERMINATED，线程池结束了它年轻的生命
                    termination.signalAll();    //如果有线程调用了awaitTermination方法，会等待当前线程池终止，到这里差不多就可以唤醒了
                }
                return;   //结束
            }
          	//注意如果CAS失败会直接进下一轮循环重新判断
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

OK，有关线程池的实现原理，我们就暂时先介绍到这里，关于更高级的定时任务线程池，这里就不做讲解了。
