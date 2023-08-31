# 计数器锁 CountDownLatch

多任务同步神器。它允许一个或多个线程，等待其他线程完成工作

比如现在有这样的一个需求：
>有20个计算任务，需要先将这些任务的结果全部计算出来，每个任务的执行时间未知
当所有任务结束之后，立即整合统计最终结果

这个需求有一个难点：任务到底什么时候执行完毕是未知的。
最直观的就是将最终统计操作延时进行，但无论延迟多久进行，要么不能保证所有任务都完成，要么可能所有任务都已完成，还在那等。

所以需要一个能够实现子任务同步的工具。
调用`await()`方法之后，实际上就是一个等待计数器衰减为0的过程，而进行自减操作则由各个子线程来完成，当子线程完成工作后，那么就将计数器-1，所有的子线程完成之后，计数器为0，结束等待。

```java
public static void main(String[] args) throws InterruptedException {
    CountDownLatch latch = new CountDownLatch(20);  //创建一个初始值为10的计数器锁
    for (int i = 0; i < 20; i++) {
        int finalI = i;
        new Thread(() -> {
            try {
                Thread.sleep((long) (2000 * new Random().nextDouble()));
                System.out.println("子任务"+ finalI +"执行完成！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            latch.countDown();   //每执行一次计数器都会-1
        }).start();
    }

    //开始等待所有的线程完成，当计数器为0时，恢复运行
    latch.await();   //这个操作可以同时被多个线程执行，一起等待，这里只演示了一个
    System.out.println("所有子任务都完成！任务完成！！！");
  
  	//注意这个计数器只能使用一次，用完只能重新创一个，没有重置的说法
}
```

***

**实现原理**
```java
public class CountDownLatch {
   	//同样是通过内部类实现AbstractQueuedSynchronizer
    private static final class Sync extends AbstractQueuedSynchronizer {
        
        Sync(int count) {   //这里直接使用AQS的state作为计数器（可见state能被玩出各种花样），也就是说一开始就加了count把共享锁，当线程调用countdown时，就解一层锁
            setState(count);
        }

        int getCount() {
            return getState();
        }

      	//采用共享锁机制，因为可以被不同的线程countdown，所以实现的tryAcquireShared和tryReleaseShared
      	//获取这把共享锁其实就是去等待state被其他线程减到0
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // 每次执行都会将state值-1，直到为0
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;   //如果已经是0了，那就false
                int nextc = c-1;
                if (compareAndSetState(c, nextc))   //CAS设置state值，失败直接下一轮循环
                    return nextc == 0;    //返回c-1之后，是不是0，如果是那就true，否则false，也就是说只有刚好减到0的时候才会返回true
            }
        }
    }

    private final Sync sync;

    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");  //count那肯定不能小于0啊
        this.sync = new Sync(count);   //构造Sync对象，将count作为state初始值
    }

   	//通过acquireSharedInterruptibly方法获取共享锁，但是如果state不为0，那么会被持续阻塞，详细原理下面讲
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    //同上，但是会超时
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

   	//countDown其实就是解锁一次
    public void countDown() {
        sync.releaseShared(1);
    }

    //获取当前的计数，也就是AQS中state的值
    public long getCount() {
        return sync.getCount();
    }

    //这个就不说了
    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```

大致了解一下CountDownLatch的基本实现思路：

* 利用共享锁实现
* 在一开始的时候就是已经上了count层锁的状态，也就是`state = count`
* `await()`就是加共享锁，但是必须`state`为`0`才能加锁成功，否则按照AQS的机制，会进入等待队列阻塞，加锁成功后结束阻塞
* `countDown()`就是解`1`层锁，也就是靠这个方法一点一点把`state`的值减到`0`

共享锁：
```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)   //上来就调用tryAcquireShared尝试以共享模式获取锁，小于0则失败，上面判断的是state==0返回1，否则-1，也就是说如果计数器不为0，那么这里会判断成功
        doAcquireShared(arg);   //计数器不为0的时候，按照它的机制，那么会阻塞，所以我们来看看doAcquireShared中是怎么进行阻塞的
}
```

```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);   //向等待队列中添加一个新的共享模式结点
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {    //无限循环
            final Node p = node.predecessor();   //获取当前节点的前驱的结点
            if (p == head) {    //如果p就是头结点，那么说明当前结点就是第一个等待节点
                int r = tryAcquireShared(arg);    //会再次尝试获取共享锁
                if (r >= 0) {      //要是获取成功
                    setHeadAndPropagate(node, r);   //那么就将当前节点设定为新的头结点，并且会继续唤醒后继节点
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&   //和独占模式下一样的操作，这里不多说了
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);   //如果最后都还是没获取到，那么就cancel
    }
}
//其实感觉大体上和独占模式的获取有点像，但是它多了个传播机制，会继续唤醒后续节点
```

```java
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // 取出头结点并将当前节点设定为新的头结点
    setHead(node);
    
  	//因为一个线程成功获取到共享锁之后，有可能剩下的等待中的节点也有机会拿到共享锁
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {   //如果propagate大于0（表示共享锁还能继续获取）或是h.waitStatus < 0，这是由于在其他线程释放共享锁时，doReleaseShared会将状态设定为PROPAGATE表示可以传播唤醒，后面会讲
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared();   //继续唤醒下一个等待节点
    }
}
```

countdown过程：
```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {   //直接尝试释放锁，如果成功返回true（在CountDownLatch中只有state减到0的那一次，会返回true）
        doReleaseShared();    //这里也会调用doReleaseShared继续唤醒后面的结点
        return true;
    }
    return false;   //其他情况false
  									//不过这里countdown并没有用到这些返回值
}
```

```java
private void doReleaseShared() {
    for (;;) {   //无限循环
        Node h = head;    //获取头结点
        if (h != null && h != tail) {    //如果头结点不为空且头结点不是尾结点，那么说明等待队列中存在节点
            int ws = h.waitStatus;    //取一下头结点的等待状态
            if (ws == Node.SIGNAL) {    //如果是SIGNAL，那么就CAS将头结点的状态设定为初始值
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            //失败就开下一轮循环重来
                unparkSuccessor(h);    //和独占模式一样，当锁被释放，都会唤醒头结点的后继节点，doAcquireShared循环继续，如果成功，那么根据setHeadAndPropagate，又会继续调用当前方法，不断地传播下去，让后面的线程一个一个地获取到共享锁，直到不能再继续获取为止
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))   //如果等待状态是默认值0，那么说明后继节点已经被唤醒，直接将状态设定为PROPAGATE，它代表在后续获取资源的时候，够向后面传播
                continue;                //失败就开下一轮循环重来
        }
        if (h == head)                   // 如果头结点发生了变化，不会break，而是继续循环，否则直接break退出
            break;
    }
}
```

***
**总结**

* 共享锁是线程共享的，同一时刻能有多个线程拥有共享锁。
* 如果一个线程刚获取了共享锁，那么在其之后等待的线程也很有可能能够获取到锁，所以得传播下去继续尝试唤醒后面的结点，不像独占锁，独占的压根不需要考虑这些。
* 如果一个线程刚释放了锁，不管是独占锁还是共享锁，都需要唤醒后续等待结点的线程。

回到CountDownLatch，再结合整个AQS共享锁的实现机制，进行一次完整的推导，看明白还是比较简单的。

# 循环屏障 CyclicBarrier

**需求**
>一场游戏，必须等待房间内人数足够之后才能开始，并且游戏开始之后玩家需要同时进入游戏以保证公平性。
假如现在游戏房间内一共5人，但是游戏开始需要10人，所以必须等待剩下5人到来之后才能开始游戏，并且保证游戏开始时所有玩家都是同时进入

CyclicBarrier循环屏障就是为了解决这个问题而出现的

循环屏障会不断阻挡线程，直到被阻挡的线程足够多时，才能一起冲破屏障，并且在冲破屏障时，可以做一些其他的任务
```java
public static void main(String[] args) {
    CyclicBarrier barrier = new CyclicBarrier(10,   //创建一个初始值为10的循环屏障
                () -> System.out.println("飞机马上就要起飞了，各位特种兵请准备！"));   //人等够之后执行的任务
    for (int i = 0; i < 10; i++) {
        int finalI = i;
        new Thread(() -> {
            try {
                Thread.sleep((long) (2000 * new Random().nextDouble()));
                System.out.println("玩家 "+ finalI +" 进入房间进行等待... ("+barrier.getNumberWaiting()+"/10)");

                barrier.await();    //调用await方法进行等待，直到等待的线程足够多为止

                //开始游戏，所有玩家一起进入游戏
                System.out.println("玩家 "+ finalI +" 进入游戏！");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

屏障由于是可循环的，所以它在被冲破后，会重新开始计数，继续阻挡后续的线程。
下面这段代码，通过使用循环屏障，可以对线程进行一波一波地放行，每一波都放行5个线程

```java
public static void main(String[] args) {
    CyclicBarrier barrier = new CyclicBarrier(5);  //创建一个初始值为5的循环屏障

    for (int i = 0; i < 10; i++) {   //创建5个线程
        int finalI = i;
        new Thread(() -> {
            try {
                Thread.sleep((long) (2000 * new Random().nextDouble()));
                System.out.println("玩家 "+ finalI +" 进入房间进行等待... ("+barrier.getNumberWaiting()+"/5)");

                barrier.await();    //调用await方法进行等待，直到等待线程到达5才会一起继续执行

                //人数到齐之后，可以开始游戏了
                System.out.println("玩家 "+ finalI +" 进入游戏！");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

除了自动重置，也可以调用`reset()`方法来手动进行重置操作，重新计数：
在调用`reset()`之后，处于等待状态下的线程，全部被中断并且抛出BrokenBarrierException异常，循环屏障等待线程数归零。

```java
public static void main(String[] args) throws InterruptedException {
    CyclicBarrier barrier = new CyclicBarrier(5);  //创建一个初始值为10的计数器锁

    for (int i = 0; i < 3; i++)
        new Thread(() -> {
            try {
                barrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }).start();

    Thread.sleep(500);   //等一下上面的线程开始运行
    System.out.println("当前屏障前的等待线程数："+barrier.getNumberWaiting());

    barrier.reset();
    System.out.println("重置后屏障前的等待线程数："+barrier.getNumberWaiting());
}
```

那么要是处于等待状态下的线程被中断了呢？屏障的线程等待数量会不会自动减少？
可以看到，当`await()`状态下的线程被中断，那么屏障会直接变成损坏状态，一旦屏障损坏，那么这一轮就无法再做任何等待操作了。也就是说，本来大家计划一起合力冲破屏障，结果有一个人摆烂中途退出了，那么所有人的努力都前功尽弃，这一轮的屏障也不可能再被冲破了，只能进行`reset()`重置操作进行重置才能恢复正常。

```java
public static void main(String[] args) throws InterruptedException {
    CyclicBarrier barrier = new CyclicBarrier(10);
    Runnable r = () -> {
        try {
            barrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    };
    Thread t = new Thread(r);
    t.start();
    t.interrupt();
    new Thread(r).start();
}
```

感觉和CountDownLatch有点像，这里区分一下：

* CountDownLatch：
  1. 它只能使用一次，是一个一次性的工具
  2. 它是一个或多个线程用于等待其他线程完成的同步工具
* CyclicBarrier
  1. 它可以反复使用，允许自动或手动重置计数
  2. 它是让一定数量的线程在同一时间开始运行的同步工具

***
**实现细节**：

```java
public class CyclicBarrier {
    //内部类，存放broken标记，表示屏障是否损坏，损坏的屏障是无法正常工作的
    private static class Generation {
        boolean broken = false;
    }

    /** 内部维护一个可重入锁 */
    private final ReentrantLock lock = new ReentrantLock();
    /** 再维护一个Condition */
    private final Condition trip = lock.newCondition();
    /** 这个就是屏障的最大阻挡容量，就是构造方法传入的初始值 */
    private final int parties;
    /* 在屏障破裂时做的事情 */
    private final Runnable barrierCommand;
    /** 当前这一轮的Generation对象，每一轮都有一个新的，用于保存broken标记 */
    private Generation generation = new Generation();

    //默认为最大阻挡容量，每来一个线程-1，和CountDownLatch挺像，当屏障破裂或是被重置时，都会将其重置为最大阻挡容量
    private int count;

  	//构造方法
  	public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }
  
    public CyclicBarrier(int parties) {
        this(parties, null);
    }
  
    //开启下一轮屏障，一般屏障被冲破之后，就自动重置了，进入到下一轮
    private void nextGeneration() {
        // 唤醒所有等待状态的线程
        trip.signalAll();
        // 重置count的值
        count = parties;
      	//创建新的Generation对象
        generation = new Generation();
    }

    //破坏当前屏障，变为损坏状态，之后就不能再使用了，除非重置
    private void breakBarrier() {
        generation.broken = true;
        count = parties;
        trip.signalAll();
    }
  
  	//开始等待
  	public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // 因为这里没有使用定时机制，不可能发生异常，如果发生怕是出了错误
        }
    }
    
  	//可超时的等待
    public int await(long timeout, TimeUnit unit)
        throws InterruptedException,
               BrokenBarrierException,
               TimeoutException {
        return dowait(true, unit.toNanos(timeout));
    }

    //这里就是真正的等待流程了，让我们细细道来
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        final ReentrantLock lock = this.lock;
        lock.lock();   //加锁，注意，因为多个线程都会调用await方法，因此只有一个线程能进，其他都被卡着了
        try {
            final Generation g = generation;   //获取当前这一轮屏障的Generation对象

            if (g.broken)
                throw new BrokenBarrierException();   //如果这一轮屏障已经损坏，那就没办法使用了

            if (Thread.interrupted()) {   //如果当前等待状态的线程被中断，那么会直接破坏掉屏障，并抛出中断异常（破坏屏障的第1种情况）
                breakBarrier();
                throw new InterruptedException();
            }

            int index = --count;     //如果上面都没有出现不正常，那么就走正常流程，首先count自减并赋值给index，index表示当前是等待的第几个线程
            if (index == 0) {  // 如果自减之后就是0了，那么说明来的线程已经足够，可以冲破屏障了
                boolean ranAction = false;
                try {
                    final Runnable command = barrierCommand;
                    if (command != null)
                        command.run();   //执行冲破屏障后的任务，如果这里抛异常了，那么会进finally
                    ranAction = true;
                    nextGeneration();   //一切正常，开启下一轮屏障（方法进入之后会唤醒所有等待的线程，这样所有的线程都可以同时继续运行了）然后返回0，注意最下面finally中会解锁，不然其他线程唤醒了也拿不到锁啊
                    return 0;
                } finally {
                    if (!ranAction)   //如果是上面出现异常进来的，那么也会直接破坏屏障（破坏屏障的第2种情况）
                        breakBarrier();
                }
            }

            // 能走到这里，那么说明当前等待的线程数还不够多，不足以冲破屏障
            for (;;) {   //无限循环，一直等，等到能冲破屏障或是出现异常为止
                try {
                    if (!timed)
                        trip.await();    //如果不是定时的，那么就直接永久等待
                    else if (nanos > 0L)
                        nanos = trip.awaitNanos(nanos);   //否则最多等一段时间
                } catch (InterruptedException ie) {    //等的时候会判断是否被中断（依然是破坏屏障的第1种情况）
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();   //如果线程被唤醒之后发现屏障已经被破坏，那么直接抛异常

                if (g != generation)   //成功冲破屏障开启下一轮，那么直接返回当前是第几个等待的线程。
                    return index;

                if (timed && nanos <= 0L) {   //线程等待超时，也会破坏屏障（破坏屏障的第3种情况）然后抛异常
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            lock.unlock();    //最后别忘了解锁，不然其他线程拿不到锁
        }
    }

  	//不多说了
    public int getParties() {
        return parties;
    }

  	//判断是否被破坏，也是加锁访问，因为有可能这时有其他线程正在执行dowait
    public boolean isBroken() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return generation.broken;
        } finally {
            lock.unlock();
        }
    }

  	//重置操作，也要加锁
    public void reset() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            breakBarrier();   // 先破坏这一轮的线程，注意这个方法会先破坏再唤醒所有等待的线程，那么所有等待的线程会直接抛BrokenBarrierException异常（详情请看上方dowait倒数第13行）
            nextGeneration(); // 开启下一轮
        } finally {
            lock.unlock();
        }
    }
	
  	//获取等待线程数量，也要加锁
    public int getNumberWaiting() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return parties - count;   //最大容量 - 当前剩余容量 = 正在等待线程数
        } finally {
            lock.unlock();
        }
    }
}
```

# 信号量 Semaphore

《操作系统》中学习的信号量机制在解决进程之间的同步问题中起着非常大的作用。

> 信号量(Semaphore)，有时被称为信号灯，是在多线程环境下使用的一种设施，是可以用来保证两个或多个关键代码段不被并发调用。在进入一个关键代码段之前，线程必须获取一个信号量；一旦该关键代码段完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量。

通过使用信号量，可以决定某个资源同一时间能够被访问的最大线程数，相当于对某个资源的访问进行了流量控制。
简单来说，它就是一个可以被N个线程占用的排它锁（因此也支持公平和非公平模式），可以在最开始设定Semaphore的许可证数量，每个线程都可以获得1个或n个许可证，当许可证耗尽或不足以供其他线程获取时，其他线程将被阻塞。

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    //每一个Semaphore都会在一开始获得指定的许可证数数量，也就是许可证配额
    Semaphore semaphore = new Semaphore(2);   //许可证配额设定为2

    for (int i = 0; i < 3; i++) {
        new Thread(() -> {
            try {
                semaphore.acquire();   //申请一个许可证
                System.out.println("许可证申请成功！");
                semaphore.release();   //归还一个许可证
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    //每一个Semaphore都会在一开始获得指定的许可证数数量，也就是许可证配额
    Semaphore semaphore = new Semaphore(3);   //许可证配额设定为3

    for (int i = 0; i < 2; i++)
        new Thread(() -> {
            try {
                semaphore.acquire(2);    //一次性申请两个许可证
                System.out.println("许可证申请成功！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    
}
```

通过Semaphore获取一些常规信息：
```java
public static void main(String[] args) throws InterruptedException {
    Semaphore semaphore = new Semaphore(3);   //只配置一个许可证，5个线程进行争抢，不内卷还想要许可证？
    for (int i = 0; i < 5; i++)
        new Thread(semaphore::acquireUninterruptibly).start();   //可以以不响应中断（主要是能简写一行，方便）
    Thread.sleep(500);
    System.out.println("剩余许可证数量："+semaphore.availablePermits());
    System.out.println("是否存在线程等待许可证："+(semaphore.hasQueuedThreads() ? "是" : "否"));
    System.out.println("等待许可证线程数量："+semaphore.getQueueLength());
}
```

手动回收掉所有的许可证：
```java
public static void main(String[] args) throws InterruptedException {
    Semaphore semaphore = new Semaphore(3);
    new Thread(semaphore::acquireUninterruptibly).start();
    Thread.sleep(500);
    System.out.println("收回剩余许可数量："+semaphore.drainPermits());   //直接回收掉剩余的许可证
}
```

比如现在有10个线程同时进行任务，任务要求是执行某个方法，但是这个方法最多同时只能由5个线程执行，这里我们使用信号量就非常合适。

# 数据交换 Exchanger

使用Exchanger，它能够实现线程之间的数据交换：
```java
public static void main(String[] args) throws InterruptedException {
    Exchanger<String> exchanger = new Exchanger<>();
    new Thread(() -> {
        try {
            System.out.println("收到主线程传递的交换数据："+exchanger.exchange("AAAA"));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    System.out.println("收到子线程传递的交换数据："+exchanger.exchange("BBBB"));
}
```

在调用`exchange`方法后，当前线程会等待其他线程调用同一个exchanger对象的`exchange`方法，当另一个线程也调用之后，方法会返回对方线程传入的参数。

# Fork/Join框架

在JDK7时，出现了一个新的框架用于并行执行任务，它的目的是为了把大型任务拆分为多个小任务，最后汇总多个小任务的结果，得到整大任务的结果，并且这些小任务都是同时在进行，大大提高运算效率。Fork就是拆分，Join就是合并。

比如一个算式：18x7+36x8+9x77+8x53，可以拆分为四个小任务：18x7、36x8、9x77、8x53，最后只需要将这四个任务的结果加起来即可

它不仅仅只是拆分任务并使用多线程，而且还可以利用工作窃取算法，提高线程的利用率。

> **工作窃取算法**：是指某个线程从其他队列里窃取任务来执行。一个大任务分割为若干个互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务待处理。干完活的线程与其等着，不如帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。

![image-20230306172457006](https://s2.loli.net/2023/03/06/DP7yj6pBZFGLoQb.png)

***
例子：**计算1-1000的和**

可以将其拆分为8个小段的数相加，比如1-125、126-250... ，最后再汇总，它也是依靠线程池来实现的：
```java
public class Main {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ForkJoinPool pool = new ForkJoinPool();
        System.out.println(pool.submit(new SubTask(1, 1000)).get());
    }


  	//继承RecursiveTask，这样才可以作为一个任务，泛型就是计算结果类型
    private static class SubTask extends RecursiveTask<Integer> {
        private final int start;   //比如我们要计算一个范围内所有数的和，那么就需要限定一下范围，这里用了两个int存放
        private final int end;

        public SubTask(int start, int end) {
            this.start = start;
            this.end = end;
        }

        @Override
        protected Integer compute() {
            if(end - start > 125) {    //每个任务最多计算125个数的和，如果大于继续拆分，小于就可以开始算了
                SubTask subTask1 = new SubTask(start, (end + start) / 2);
                subTask1.fork();    //会继续划分子任务执行
                SubTask subTask2 = new SubTask((end + start) / 2 + 1, end);
                subTask2.fork();   //会继续划分子任务执行
                return subTask1.join() + subTask2.join();   //越玩越有递归那味了
            } else {
                System.out.println(Thread.currentThread().getName()+" 开始计算 "+start+"-"+end+" 的值!");
                int res = 0;
                for (int i = start; i <= end; i++) {
                    res += i;
                }
                return res;   //返回的结果会作为join的结果
            }
        }
    }
}
```

```
ForkJoinPool-1-worker-2 开始计算 1-125 的值!
ForkJoinPool-1-worker-2 开始计算 126-250 的值!
ForkJoinPool-1-worker-0 开始计算 376-500 的值!
ForkJoinPool-1-worker-6 开始计算 751-875 的值!
ForkJoinPool-1-worker-3 开始计算 626-750 的值!
ForkJoinPool-1-worker-5 开始计算 501-625 的值!
ForkJoinPool-1-worker-4 开始计算 251-375 的值!
ForkJoinPool-1-worker-7 开始计算 876-1000 的值!
500500
```

可以看到，结果非常正确，整个计算任务实际上是拆分为了8个子任务同时完成的，原本的单线程任务，在多线程的加持下速度成倍提升。

***

Arrays工具类提供的并行排序也是利用了ForkJoinPool来实现：

```java
public static void parallelSort(byte[] a) {
    int n = a.length, p, g;
    if (n <= MIN_ARRAY_SORT_GRAN ||
        (p = ForkJoinPool.getCommonPoolParallelism()) == 1)
        DualPivotQuicksort.sort(a, 0, n - 1);
    else
        new ArraysParallelSortHelpers.FJByte.Sorter
            (null, a, new byte[n], 0, n, 0,
             ((g = n / (p << 2)) <= MIN_ARRAY_SORT_GRAN) ?
             MIN_ARRAY_SORT_GRAN : g).invoke();
}
```

并行排序的性能在多核心CPU环境下，肯定是优于普通排序的，并且排序规模越大优势越显著。