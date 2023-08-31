# 多线程

进程是程序执行的实体，每一个进程都是一个应用程序，都有自己的内存空间，CPU一个核心同时只能处理一件事情，当出现多个进程需要同时运行时，CPU一般通过`时间片轮转调度`算法，来实现多个进程的同时运行。

由于每个进程都有一个自己的内存空间，进程之间的通信就变得非常麻烦（比如要共享某些数据），而且执行不同进程会产生上下文切换，非常耗时

后来，线程横空出世，一个进程可以有多个线程，线程是程序执行中一个单一的顺序控制流程，各个线程之间共享程序的内存空间（也就是所在进程的内存空间），上下文切换速度也高于进程。

通常，初学java以来，编写的都是单线程应用程序（运行`main()`方法的内容），同时只能执行一个任务

如果希望同时执行多个任务，就需要用到Java多线程框架。

实际上一个Java程序启动后，会创建很多线程，不仅仅只运行一个主线程【涉及到JVM相关知识】：

```java
public static void main(String[] args) {
    ThreadMXBean bean = ManagementFactory.getThreadMXBean();
    long[] ids = bean.getAllThreadIds();
    ThreadInfo[] infos = bean.getThreadInfo(ids);
    for (ThreadInfo info : infos) {
        System.out.println(info.getThreadName());
    }
}
```

# 线程的创建和启动

通过创建Thread对象来创建一个新的线程，Thread构造方法中需要传入一个Runnable接口的实现（其实就是编写要在另一个线程执行的内容逻辑），这个Runnable接口只有一个未实现方法，因此可以直接使用lambda表达式：

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

***
**例一**
在主线程中创建一个新线程并启动运行：
```java
public static void main(String[] args) {
    Thread t = new Thread(() -> {
        System.out.println("我是线程："+Thread.currentThread().getName());
        System.out.println("我正在计算 0-10000 之间所有数的和...");
        int sum = 0;
        for (int i = 0; i <= 10000; i++) {
            sum += i;
        }
        System.out.println("结果："+sum);
    });
    t.start();
    System.out.println("我是主线程！");
}
```

运行结果：
上面这段代码执行结果并不是按照从上往下的顺序了，由于分别位于两个线程，它们是同时进行的！
```
我是主线程！
我是线程：Thread-0
我正在计算 0-10000 之间所有数的和...
结果：50005000
```

***
**例二**
```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 50; i++) {
            System.out.println("我是一号线程："+i);
        }
    });
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 50; i++) {
            System.out.println("我是二号线程："+i);
        }
    });
    t1.start();
    t2.start();
}
```
打印输出的结果实际上是在交替进行的，也证明了这两个线程是在同时运行！

>**注意**：除了start方法外，还有一个run方法，也能执行线程里面定义的内容，但是run是直接在当前线程执行，并不是创建一个线程执行！

# 线程的休眠和中断

一个线程处于运行状态，线程的下一个状态会出现以下情况：

- 当CPU给予的运行时间结束时，会从运行状态回到就绪（可运行）状态，等待下一次获得CPU资源。
- 当线程进入休眠 / 阻塞(如等待IO请求) / 手动调用`wait()`方法时，会使得线程处于等待状态，当等待状态结束后会回到就绪状态。
- 当线程出现异常或错误 / 被`stop()` 方法强行停止 / 所有代码执行结束时，会使得线程的运行终止。

***
**线程休眠**
`sleep`休眠线程使其进入等待状态：

```java
public static void main(String[] args) {
    Thread t = new Thread(() -> {
        try {
            System.out.println("l");
            //sleep方法是Thread的静态方法，它只作用于当前线程（它知道当前线程是哪个）
            Thread.sleep(1000);  
            System.out.println("b");    
            //调用sleep后，线程会直接进入到等待状态，直到时间结束
        } catch (InterruptedException e) { //`sleep()`方法显示声明了会抛出一个InterruptedException异常
            e.printStackTrace();
        }
    });
    t.start();
}
```

***
**线程中断**
`sleep()`方法显示声明了会抛出一个InterruptedException异常
每一个Thread对象中，都有一个`interrupt()`方法，调用此方法后，会给指定线程添加一个中断标记以告知线程需要立即停止运行或是进行其他操作，由线程来响应此中断并进行相应的处理
```java
public static void main(String[] args) {
    Thread t = new Thread(() -> {
        try {
            Thread.sleep(10000);  //休眠10秒
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    t.start();
    try {
        Thread.sleep(3000);   //主线程休眠3秒，一定比线程t先醒来
        t.interrupt();   //调用t的interrupt方法
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

>`stop()`方法也可以中断线程，但它是强制终止，虽然简单粗暴，但是很有可能导致资源不能完全释放，因此不推荐用该方法终止线程。

而类似这样的发送通知来告知线程需要中断，让线程自行处理后续，会更加合理一些，也是更加推荐的做法。

`interrupt`的用法：
通过`isInterrupted()`可以判断线程是否存在中断标志，如果存在，说明外部希望当前线程立即停止，也有可能是给当前线程发送一个其他的信号。如果我们并不是希望收到中断信号就是结束程序，而是通知程序做其他事情，我们可以在收到中断信号后，复位中断标记，然后继续做我们的事情：

```java
public static void main(String[] args) {
    Thread t = new Thread(() -> {
        System.out.println("线程开始运行！");
        while (true){
            if(Thread.currentThread().isInterrupted()){   //判断是否存在中断标志
                System.out.println("发现中断信号，复位，继续运行...");
                Thread.interrupted();  //复位中断标记（返回值是当前是否有中断标记，这里不用管）
            }
        }
    });
    t.start();
    try {
        Thread.sleep(3000);   //休眠3秒，一定比线程t先醒来
        t.interrupt();   //调用t的interrupt方法
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

复位中断标记后，会立即清除中断标记。

***
**线程暂停和恢复**
如果现在希望线程暂时停下，比如等待其他线程执行完成后，再继续运行，可以用suspend()和resume()这两个方法
>虽然这样很方便地控制了线程的暂停状态，但是这两个方法依然是不推荐的，很容易导致死锁！

```java
public static void main(String[] args) {
    Thread t = new Thread(() -> {
        System.out.println("线程开始运行！");
        Thread.currentThread().suspend();   //暂停此线程
        System.out.println("线程继续运行！");
    });
    t.start();
    try {
        Thread.sleep(3000);   //休眠3秒，一定比线程t先醒来
        t.resume();   //恢复此线程
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

# 线程的优先级

Java程序采用抢占式调度方式，优先级越高的线程，优先使用CPU资源
线程的优先级一般分为以下三种：

- MIN_PRIORITY  最低优先级
- MAX_PRIORITY  最高优先级
- NOM_PRIORITY  常规优先级

>优先级越高的线程，获得CPU资源的概率会越大，并不是说一定优先级越高的线程越先执行！

```java
public static void main(String[] args) {
    Thread t = new Thread(() -> {
        System.out.println("线程开始运行！");
    });
    t.start();
    t.setPriority(Thread.MIN_PRIORITY);  //通过使用setPriority方法来设定优先级
}
```

# 线程的礼让和加入

**线程的礼让**
在当前线程的工作不重要时，可以通过`yield()`方法将当前CPU资源让位给其他同优先级线程：

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        System.out.println("线程1开始运行！");
        for (int i = 0; i < 50; i++) {
            if(i % 5 == 0) {
                System.out.println("让位！");
                Thread.yield();
            }
            System.out.println("1打印："+i);
        }
        System.out.println("线程1结束！");
    });
    Thread t2 = new Thread(() -> {
        System.out.println("线程2开始运行！");
        for (int i = 0; i < 50; i++) {
            System.out.println("2打印："+i);
        }
    });
    t1.start();
    t2.start();
}
```

***
**线程加入**
当希望一个线程等待另一个线程执行完成后再继续进行，可以使用`join()`方法来实现线程的加入：

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        System.out.println("线程1开始运行！");
        for (int i = 0; i < 50; i++) {
            System.out.println("1打印："+i);
        }
        System.out.println("线程1结束！");
    });
    Thread t2 = new Thread(() -> {
        System.out.println("线程2开始运行！");
        for (int i = 0; i < 50; i++) {
            System.out.println("2打印："+i);
            if(i == 10){
                try {
                    System.out.println("线程1加入到此线程！");
                    t1.join();    //在i==10时，让线程1加入，先完成线程1的内容，在继续当前内容
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    });
    t1.start();
    t2.start();
}
```

线程1加入后，线程2等待线程1待执行的内容全部执行完成之后，才会继续执行的线程2内容。
线程的加入只是等待另一个线程的完成，并不是将另一个线程和当前线程合并

# 线程锁和线程同步

多线程情况下Java的内存管理：

![img/多线程内存模型png.png](https://image.itbaima.net/images/253/image-20230831129418640.png)

线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的工作内存（本地内存），工作内存中存储了该线程以读/写共享变量的副本。

这种设计类似于多核心处理器高速缓存机制：
高速缓存通过保存内存中数据的副本来提供更加快速的数据访问，但是如果多个处理器的运算任务都涉及同一块内存区域，就可能导致各自的高速缓存数据不一致，在写回主内存时就会发生冲突。这就是引入高速缓存引发的新问题：**缓存一致性。**

Java的内存模型也是如此，当多线程同时去操作一个共享变量时，如果仅仅是读取还好，但是如果同时写入内容，就会出现问题

比如下面这个问题：
当两个线程同时读取value的时候，可能会同时拿到同样的值，而进行自增操作之后，也是同样的值，再写回主内存后，本来应该进行2次自增操作，实际上只执行了一次！

```java
private static int value = 0;

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 10000; i++) value++;
        System.out.println("线程1完成");
    });
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 10000; i++) value++;
        System.out.println("线程2完成");
    });
    t1.start();
    t2.start();
    Thread.sleep(1000);  //主线程停止1秒，保证两个线程执行完成
    System.out.println(value);
}
```

***

**synchronized关键字来创造线程锁**
>synchronized是一种悲观锁，随时都认为有其他线程在对数据进行修改，

synchronized代码块需要在括号中填入一个内容，必须是一个对象或是一个类

```java
private static int value = 0;

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 10000; i++) {
            synchronized (Main.class){  //使用synchronized关键字创建同步代码块
                value++;
            }
        }
        System.out.println("线程1完成");
    });
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 10000; i++) {
            synchronized (Main.class){
                value++;
            }
        }
        System.out.println("线程2完成");
    });
    t1.start();
    t2.start();
    Thread.sleep(1000);  //主线程停止1秒，保证两个线程执行完成
    System.out.println(value);
}
```

现在得到的结果就是预想的10000了。
因为在同步代码块执行过程中，拿到了我们传入的对象或类的锁【传入的是对象，就是对象锁；如果是类，就是类锁，类锁只有一个。实际上类锁也是对象锁，是Class类实例，但是Class类实例同样的类无论怎么获取都是同一个】
**注意两个线程必须使用同一把锁**！

当一个线程进入到同步代码块时，会获取到当前的锁，而这时如果其他使用同样的锁的同步代码块也想执行内容，就必须等待当前同步代码块的内容执行完毕，然后自动释放这把锁，其他的线程才能拿到这把锁并开始执行同步代码块里面的内容

synchronized关键字也可以作用于方法上，调用此方法时也会获取锁：

```java
private static int value = 0;

private static synchronized void add(){
    value++;
}

public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(() -> {
        for (int i = 0; i < 10000; i++) add();
        System.out.println("线程1完成");
    });
    Thread t2 = new Thread(() -> {
        for (int i = 0; i < 10000; i++) add();
        System.out.println("线程2完成");
    });
    t1.start();
    t2.start();
    Thread.sleep(1000);  //主线程停止1秒，保证两个线程执行完成
    System.out.println(value);
}
```

效果是相同的，只不过这个锁不用自己去给。如果是静态方法，就是使用的类锁；如果是普通成员方法，就是使用的对象锁。

# 死锁

死锁的概念无需多言了，简单来说，就是两个线程相互持有对方需要的锁，但是又迟迟不释放，导致程序卡住

比如：
```java
public static void main(String[] args) throws InterruptedException {
    Object o1 = new Object();
    Object o2 = new Object();
    Thread t1 = new Thread(() -> {
        synchronized (o1){
            try {
                Thread.sleep(1000);
                synchronized (o2){
                    System.out.println("线程1");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
    Thread t2 = new Thread(() -> {
        synchronized (o2){
            try {
                Thread.sleep(1000);
                synchronized (o1){
                    System.out.println("线程2");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
    t1.start();
    t2.start();
}
```

利用jstack命令来检测死锁，首先利用jps找到我们的java进程：

```shell
nagocoler@NagodeMacBook-Pro ~ % jps
51592 Launcher
51690 Jps
14955 
51693 Main
nagocoler@NagodeMacBook-Pro ~ % jstack 51693
...
Java stack information for the threads listed above:
===================================================
"Thread-1":
	at com.test.Main.lambda$main$1(Main.java:46)
	- waiting to lock <0x000000076ad27fc0> (a java.lang.Object)
	- locked <0x000000076ad27fd0> (a java.lang.Object)
	at com.test.Main$$Lambda$2/1867750575.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
"Thread-0":
	at com.test.Main.lambda$main$0(Main.java:34)
	- waiting to lock <0x000000076ad27fd0> (a java.lang.Object)
	- locked <0x000000076ad27fc0> (a java.lang.Object)
	at com.test.Main$$Lambda$1/396873410.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

jstack自动帮助我们找到了一个死锁，并打印出了相关线程的栈追踪信息.同样的，使用`jconsole`也可以进行监测。

>`suspend()`在使线程暂停的同时，并不会去释放任何锁资源。其他线程都无法访问被它占用的锁。直到对应的线程执行`resume()`方法后，被挂起的线程才能继续，执行完后，其它被阻塞在这个锁的线程才可以继续执行。但是，如果`resume()`操作出现在`suspend()`之前执行，那么线程将一直处于挂起状态，同时一直占用锁，这就产生了死锁。

# wait和notify方法

`wait()`、`notify()`以及`notifyAll()`需要配合synchronized来使用，只有在同步代码块中才能使用这些方法，不然会报错

wait和sleep的区别：
* wait来自object类, sleep来自线程类
* wait会释放锁, sleep不会释放锁
* wait必须在同步代码块中，sleep可以在任何地方睡眠
* wait是不需要捕获异常，sleep必须要捕获异常

例子
```java
public static void main(String[] args) throws InterruptedException {
    Object o1 = new Object();
    Thread t1 = new Thread(() -> {
        synchronized (o1){
            try {
                System.out.println("开始等待");
                o1.wait();     //进入等待状态并释放锁
                System.out.println("等待结束！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
    Thread t2 = new Thread(() -> {
        synchronized (o1){
            System.out.println("开始唤醒！");
            o1.notify();     //唤醒处于等待状态的线程
          	for (int i = 0; i < 50; i++) {
               	System.out.println(i);   
            }
          	//唤醒后依然需要等待这里的锁释放之前等待的线程才能继续
        }
    });
    t1.start();
    Thread.sleep(1000);
    t2.start();
}
```

对象的`wait()`方法会暂时使得此线程进入等待状态，同时会释放当前代码块持有的锁，这时其他线程可以获取到此对象的锁。当其他线程调用对象的`notify()`方法后，会唤醒刚才变成等待状态的线程（这时并没有立即释放锁）。

notifyAll其实和notify一样，也是用于唤醒。前者是唤醒所有调用`wait()`后处于等待的线程，而后者是看运气随机选择一个。

# ThreadLocal的使用

每个线程都有一个自己的工作内存，ThreadLocal类可以创建工作内存中的变量，它将变量值存储在对应线程内部（只能存储一个变量），不同的线程访问到ThreadLocal对象时，都只能获取到当前线程所属的变量。

也就是说，不同线程向ThreadLocal存放数据，只会存放在线程自己的工作空间中，而不会直接存放到主内存中，因此各个线程直接存放的内容互不干扰。

以下用三个例子具体说明

***
开启两个线程分别去访问ThreadLocal对象
```java
public static void main(String[] args) throws InterruptedException {
    ThreadLocal<String> local = new ThreadLocal<>();  //注意这是一个泛型类，存储类型为我们要存放的变量类型
    Thread t1 = new Thread(() -> {
        local.set("lbwnb");   //将变量的值给予ThreadLocal
        System.out.println("变量值已设定！");
        System.out.println(local.get());   //尝试获取ThreadLocal中存放的变量
    });
    Thread t2 = new Thread(() -> {
        System.out.println(local.get());   //尝试获取ThreadLocal中存放的变量
    });
    t1.start();
    Thread.sleep(3000);    //间隔三秒
    t2.start();
}
```

结果：
第一个线程存放的内容，第一个线程可以获取，但是第二个线程无法获取
```
变量值已设定！
lbwnb
null
```

***

第一个线程存入后，第二个线程也存放
```java
public static void main(String[] args) throws InterruptedException {
    ThreadLocal<String> local = new ThreadLocal<>();  //注意这是一个泛型类，存储类型为我们要存放的变量类型
    Thread t1 = new Thread(() -> {
        local.set("lbwnb");   //将变量的值给予ThreadLocal
        System.out.println("线程1变量值已设定！");
        try {
            Thread.sleep(2000);    //间隔2秒
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("线程1读取变量值：");
        System.out.println(local.get());   //尝试获取ThreadLocal中存放的变量
    });
    Thread t2 = new Thread(() -> {
        local.set("yyds");   //将变量的值给予ThreadLocal
        System.out.println("线程2变量值已设定！");
    });
    t1.start();
    Thread.sleep(1000);    //间隔1秒
    t2.start();
}
```

结果：
即使线程2重新设定了值，也没有影响到线程1存放的值。
所以说，不同线程向ThreadLocal存放数据，只会存放在线程自己的工作空间中，而不会直接存放到主内存中，因此各个线程直接存放的内容互不干扰。
```
线程1变量值已设定！
线程2变量值已设定！
线程1读取变量值：lbwnb
```

***
在线程中创建的子线程，无法获得父线程工作内存中的变量：

```java
public static void main(String[] args) {
    ThreadLocal<String> local = new ThreadLocal<>();
    Thread t = new Thread(() -> {
       local.set("lbwnb");
        new Thread(() -> {
            System.out.println(local.get());
        }).start();
    });
    t.start();
}
```

可以使用InheritableThreadLocal来解决：
在InheritableThreadLocal存放的内容，会自动向子线程传递。
```java
public static void main(String[] args) {
    ThreadLocal<String> local = new InheritableThreadLocal<>();
    Thread t = new Thread(() -> {
       local.set("lbwnb");
        new Thread(() -> {
            System.out.println(local.get());
        }).start();
    });
    t.start();
}
```

# 定时器

Java提供了一套自己的框架用于处理定时任务：
通过`Timer`对象可以创建任意类型的定时任务，包括延时任务、循环定时任务等。
```java
public static void main(String[] args) {
    Timer timer = new Timer();    //创建定时器对象
    timer.schedule(new TimerTask() {   //注意这个是一个抽象类，不是接口，无法使用lambda表达式简化，只能使用匿名内部类
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());    //打印当前线程名称
        }
    }, 1000);    //执行一个延时任务
}
```

虽然任务执行完成了，但是程序并没有停止，这是因为Timer内存维护了一个任务队列和一个工作线程：

```java
public class Timer {
    /**
     * The timer task queue.  This data structure is shared with the timer
     * thread.  The timer produces tasks, via its various schedule calls,
     * and the timer thread consumes, executing timer tasks as appropriate,
     * and removing them from the queue when they're obsolete.
     */
    private final TaskQueue queue = new TaskQueue();

    /**
     * The timer thread.
     */
    private final TimerThread thread = new TimerThread(queue);
  
		...
}
```

TimerThread继承自Thread，是一个新创建的线程，在构造时自动启动：

```java
public Timer(String name) {
    thread.setName(name);
    thread.start();
}
```

而它的run方法会循环地读取队列中是否还有任务，如果有任务依次执行，没有的话就暂时处于休眠状态：

```java
public void run() {
    try {
        mainLoop();
    } finally {
        // Someone killed this Thread, behave as if Timer cancelled
        synchronized(queue) {
            newTasksMayBeScheduled = false;
            queue.clear();  // Eliminate obsolete references
        }
    }
}

/**
 * The main timer loop.  (See class comment.)
 */
private void mainLoop() {
  try {
       TimerTask task;
       boolean taskFired;
       synchronized(queue) {
         	// Wait for queue to become non-empty
          while (queue.isEmpty() && newTasksMayBeScheduled)   //当队列为空同时没有被关闭时，会调用wait()方法暂时处于等待状态，当有新的任务时，会被唤醒。
                queue.wait();
          if (queue.isEmpty())
             break;    //当被唤醒后都没有任务时，就会结束循环，也就是结束工作线程
                      ...
}
```

`newTasksMayBeScheduled`实际上就是标记当前定时器是否关闭，当它为false时，表示已经不会再有新的任务到来，也就是关闭，我们可以通过调用`cancel()`方法来关闭它的工作线程：

```java
public void cancel() {
    synchronized(queue) {
        thread.newTasksMayBeScheduled = false;
        queue.clear();
        queue.notify();  //唤醒wait使得工作线程结束
    }
}
```

因此，在使用完成后，调用Timer的`cancel()`方法来正常退出程序：

```java
public static void main(String[] args) {
    Timer timer = new Timer();
    timer.schedule(new TimerTask() {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName());
            timer.cancel();  //结束
        }
    }, 1000);
}
```

# 守护线程

>不要把操作系统重的守护进程和守护线程相提并论！

守护进程在后台运行运行，不需要和用户交互，本质和普通进程类似。
而守护线程就不一样了，当其他所有的非守护线程结束之后，守护线程自动结束。也就是说，Java中所有的线程都执行完毕后，守护线程自动结束，因此**守护线程不适合进行IO操作，只适合打打杂**：

```java
public static void main(String[] args) throws InterruptedException{
    Thread t = new Thread(() -> {
        while (true){
            try {
                System.out.println("程序正常运行中...");
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
    t.setDaemon(true);   //设置为守护线程（必须在开始之前，中途是不允许转换的）
    t.start();
    for (int i = 0; i < 5; i++) {
        Thread.sleep(1000);
    }
}
```

在守护线程中产生的新线程也是守护的：

```java
public static void main(String[] args) throws InterruptedException{
    Thread t = new Thread(() -> {
        Thread it = new Thread(() -> {
            while (true){
                try {
                    System.out.println("程序正常运行中...");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        it.start();
    });
    t.setDaemon(true);   //设置为守护线程（必须在开始之前，中途是不允许转换的）
    t.start();
    for (int i = 0; i < 5; i++) {
        Thread.sleep(1000);
    }
}
```

# 再谈集合类

集合类中有一个东西是Java8新增的Spliterator接口，翻译过来就是：可拆分迭代器（Splitable Iterator）

Spliterator也用于遍历数据源中的元素，但它是为了并行执行而设计的。

Java 8已经为集合框架中包含的所有数据结构提供了一个默认的Spliterator实现。在集合跟接口Collection中提供了一个`spliterator()`方法用于获取可拆分迭代器。

集合类的根接口有这样一个方法：
并行流，其实就是一个多线程执行的流，它通过默认的ForkJoinPool实现，提高多线程任务的速度。

```java
//parallelStream就是利用了可拆分迭代器进行多线程操作
default Stream<E> parallelStream() {
    return StreamSupport.stream(spliterator(), true); 
}
```

以下的这段代码，从结果看出，forEach操作的顺序，并不是实际List中的顺序，而且每次打印也是不同的线程在执行！

```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<>(Arrays.asList(1, 4, 5, 2, 9, 3, 6, 0));
    list
            .parallelStream()    //获得并行流
            .forEach(i -> System.out.println(Thread.currentThread().getName()+" -> "+i));
}
```

通过调用`forEachOrdered()`方法来使用单线程维持原本的顺序：

```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<>(Arrays.asList(1, 4, 5, 2, 9, 3, 6, 0));
    list
            .parallelStream()    //获得并行流
            .forEachOrdered(System.out::println);
}
```

***
在Arrays数组工具类中，也包含大量的并行方法：

```java
public static void main(String[] args) {
    int[] arr = new int[]{1, 4, 5, 2, 9, 3, 6, 0};
    Arrays.parallelSort(arr);   //使用多线程进行并行排序，效率更高
    System.out.println(Arrays.toString(arr));
}
```

使用并行方法，可以更加充分地发挥现代计算机多核心的优势，但是同时需要注意多线程产生的异步问题！

```java
public static void main(String[] args) {
    int[] arr = new int[]{1, 4, 5, 2, 9, 3, 6, 0};
    Arrays.parallelSetAll(arr, i -> {
        System.out.println(Thread.currentThread().getName());
        return arr[i];
    });
    System.out.println(Arrays.toString(arr));
}
```

***

因为多线程的加入，之前认识的集合类都废掉了：

```java
public static void main(String[] args) throws InterruptedException {
    List<Integer> list = new ArrayList<>();
    new Thread(() -> {
        for (int i = 0; i < 1000; i++) {
            list.add(i);   //两个线程同时操作集合类进行插入操作
        }
    }).start();
    new Thread(() -> {
        for (int i = 1000; i < 2000; i++) {
            list.add(i);
        }
    }).start();
    Thread.sleep(2000);
    System.out.println(list.size());
}
```

有些时候运气不好，得到的结果并不是2000个元素，而是会抛出数组越界的异常
因为之前的集合类，并没有考虑到多线程运行的情况，如果两个线程同时执行，那么有可能两个线程同一时间都执行同一个方法，这种情况下就很容易出问题：

```java
public boolean add(E e) {
    /* 当数组容量刚好还差一个满的时候，这个时候两个线程同时走到了这里，
    因为都判断为没满，所以说没有进行扩容，但是实际上两个线程都要插入一个元素进来
    */
    ensureCapacityInternal(size + 1);  
    //当两个线程同时在这里插入元素，直接导致越界访问
    elementData[size++] = e;   
    return true;
}
```

当然，在Java早期的时候，还有一些老的集合类，这些集合类都是线程安全的：
因为这些集合类中的每一个方法都加了锁，所以说不会出现多线程问题，但是这些老的集合类现在已经不再使用了

```java
public static void main(String[] args) throws InterruptedException {
    Vector<Integer> list = new Vector<>();   //我们可以使用Vector代替List使用
  	//Hashtable<Integer, String>   也可以使用Hashtable来代替Map
    new Thread(() -> {
        for (int i = 0; i < 1000; i++) {
            list.add(i);
        }
    }).start();
    new Thread(() -> {
        for (int i = 1000; i < 2000; i++) {
            list.add(i);
        }
    }).start();

    Thread.sleep(1000);
    System.out.println(list.size());
}
```

>JUC并发编程有自己专门的集合类