之前举过的例子，如果要保证`i++`的原子性，当时唯一选择就是加锁

除了加锁之外，JUC提供了原子类，底层采用CAS算法，它是一种用法简单、性能高效、线程安全地更新变量的方式。

所有的原子类都位于`java.util.concurrent.atomic`包下。

# 原子类介绍

## 封装基本数据类型

常用基本数据类型，有对应的原子类封装：

* AtomicInteger：原子更新int
* AtomicLong：原子更新long
* AtomicBoolean：原子更新boolean


将int数值封装到AtomicInteger类中，必须调用构造方法，它不像Integer那样有装箱机制。
通过调用此类提供的方法来获取或是对封装的int值进行自增
但是它可不仅仅是简单的包装，同样是直接进行自增操作，使用原子类是可以保证自增操作原子性的

```java
public class Main {
    private static AtomicInteger i = new AtomicInteger(0);
    public static void main(String[] args) throws InterruptedException {
        Runnable r = () -> {
            for (int j = 0; j < 100000; j++)
                i.getAndIncrement();
            System.out.println("自增完成！");
        };
        new Thread(r).start();
        new Thread(r).start();
        TimeUnit.SECONDS.sleep(1);
        System.out.println(i.get());
    }
}
```

它的底层是如何实现的，直接从构造方法点进去：
它的底层就是封装了一个`volatile`类型的int值，这样能够保证可见性，在CAS操作的时候不会出现问题。

```java
private volatile int value;

public AtomicInteger(int initialValue) {
    value = initialValue;
}

public AtomicInteger() {
}
```

可以看到最上面是和AQS采用了类似的机制，因为要使用CAS算法更新value的值，所以得先计算出value字段在对象中的偏移地址，CAS直接修改对应位置的内存即可（可见Unsafe类的作用巨大，很多的底层操作都要靠它来完成）

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

接着来看自增操作是怎么在运行的：

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

可以看到这里调用了`unsafe.getAndAddInt()`，然后就是套娃

接着看看Unsafe里面写了什么：

```java
public final int getAndAddInt(Object o, long offset, int delta) {  //delta就是变化的值，++操作就是自增1
    int v;
    do {
      	//volatile版本的getInt()
      	//能够保证可见性
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));  //这里是开始cas替换int的值，每次都去拿最新的值去进行替换，如果成功则离开循环，不成功说明这个时候其他线程先修改了值，就进下一次循环再获取最新的值然后再cas一次，直到成功为止
    return v;
}
```

这个`do-while`循环在做一个什么事情呢？感觉和AQS队列中的机制差不多，也是采用自旋形式，来不断进行CAS操作，直到成功。

可见，原子类底层也是采用了CAS算法来保证的原子性，包括`getAndSet`、`getAndAdd`等方法都是这样。

原子类也直接提供了CAS操作方法，可以直接使用：

```java
public static void main(String[] args) throws InterruptedException {
    AtomicInteger integer = new AtomicInteger(10);
    System.out.println(integer.compareAndSet(30, 20));
    System.out.println(integer.compareAndSet(10, 20));
    System.out.println(integer);
}
```

如果想以普通变量的方式来设定值，那么可以使用`lazySet()`方法，这样就不采用`volatile`的立即可见机制了。

```java
AtomicInteger integer = new AtomicInteger(1);
integer.lazySet(2);
```

## 封装数组类型

除了基本类有原子类以外，基本类型的数组类型也有原子类：

* AtomicIntegerArray：原子更新int数组
* AtomicLongArray：原子更新long数组
* AtomicReferenceArray：原子更新引用数组

其实原子数组和原子类型一样的，不过我们可以对数组内的元素进行原子操作：

```java
public static void main(String[] args) throws InterruptedException {
    AtomicIntegerArray array = new AtomicIntegerArray(new int[]{0, 4, 1, 3, 5});
    Runnable r = () -> {
        for (int i = 0; i < 100000; i++)
            array.getAndAdd(0, 1);
    };
    new Thread(r).start();
    new Thread(r).start();
    TimeUnit.SECONDS.sleep(1);
    System.out.println(array.get(0));
}
```

在JDK8之后，新增了`DoubleAdder`和`LongAdder`，在高并发情况下，`LongAdder`的性能比`AtomicLong`的性能更好，主要体现在自增上，它的大致原理如下：
在低并发情况下，和`AtomicLong`是一样的，对value值进行CAS操作，但是出现高并发的情况时，`AtomicLong`会进行大量的循环操作来保证同步，而`LongAdder`会将对value值的CAS操作分散为对数组`cells`中多个元素的CAS操作（内部维护一个Cell[] as数组，每个Cell里面有一个初始值为0的long型变量，在高并发时会进行分散CAS，就是不同的线程可以对数组中不同的元素进行CAS自增，这样就避免了所有线程都对同一个值进行CAS），只需要最后再将结果加起来即可。

使用如下：

```java
public static void main(String[] args) throws InterruptedException {
    LongAdder adder = new LongAdder();
    Runnable r = () -> {
        for (int i = 0; i < 100000; i++)
            adder.add(1);
    };
    for (int i = 0; i < 100; i++)
        new Thread(r).start();   //100个线程
    TimeUnit.SECONDS.sleep(1);
    System.out.println(adder.sum());   //最后求和即可
}
```

由于底层源码比较复杂，这里就不做讲解了。两者的性能对比（这里用到了CountDownLatch，建议学完之后再来看）：

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("使用AtomicLong的时间消耗："+test2()+"ms");
        System.out.println("使用LongAdder的时间消耗："+test1()+"ms");
    }

    private static long test1() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(100);
        LongAdder adder = new LongAdder();
        long timeStart = System.currentTimeMillis();
        Runnable r = () -> {
            for (int i = 0; i < 100000; i++)
                adder.add(1);
            latch.countDown();
        };
        for (int i = 0; i < 100; i++)
            new Thread(r).start();
        latch.await();
        return System.currentTimeMillis() - timeStart;
    }

    private static long test2() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(100);
        AtomicLong atomicLong = new AtomicLong();
        long timeStart = System.currentTimeMillis();
        Runnable r = () -> {
            for (int i = 0; i < 100000; i++)
                atomicLong.incrementAndGet();
            latch.countDown();
        };
        for (int i = 0; i < 100; i++)
            new Thread(r).start();
        latch.await();
        return System.currentTimeMillis() - timeStart;
    }
}
```

## 封装引用类型

除了对基本数据类型支持原子操作外，对于引用类型，也是可以实现原子操作的：

```java
public static void main(String[] args) throws InterruptedException {
    String a = "Hello";
    String b = "World";
    AtomicReference<String> reference = new AtomicReference<>(a);
    reference.compareAndSet(a, b);
    System.out.println(reference.get());
}
```

JUC还提供了字段原子更新器，可以对类中的某个指定字段进行原子操作（注意字段必须添加volatile关键字）：

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Student student = new Student();
        AtomicIntegerFieldUpdater<Student> fieldUpdater =
                AtomicIntegerFieldUpdater.newUpdater(Student.class, "age");
        System.out.println(fieldUpdater.incrementAndGet(student));
    }

    public static class Student{
        volatile int age;
    }
}
```

# ABA问题及解决方案

想象一下这种场景：
线程1和线程2同时开始对`a`的值进行CAS修改，但是线程1的速度比较快，将a的值修改为2之后紧接着又修改回1，这时线程2才开始进行判断，发现a的值是1，所以CAS操作成功。
很明显，这里的1已经不是一开始的那个1了，而是被重新赋值的1，这也是CAS操作存在的问题（无锁虽好，但是问题多多），它只会机械地比较当前值是不是预期值，但是并不会关心当前值是否被修改过，这种问题称之为`ABA`问题。

JUC提供了带版本号的引用类型，只要每次操作都记录一下版本号，并且版本号不会重复，那么就可以解决ABA问题了：

```java
public static void main(String[] args) throws InterruptedException {
    String a = "Hello";
    String b = "World";
    AtomicStampedReference<String> reference = new AtomicStampedReference<>(a, 1);  //在构造时需要指定初始值和对应的版本号
    reference.attemptStamp(a, 2);   //可以中途对版本号进行修改，注意要填写当前的引用对象
    System.out.println(reference.compareAndSet(a, b, 2, 3));   //CAS操作时不仅需要提供预期值和修改值，还要提供预期版本号和新的版本号
}
```