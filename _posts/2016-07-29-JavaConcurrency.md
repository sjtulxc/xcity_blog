---
title: "大型网站系统与JAVA中间件读书笔记二"
tags: [JAVA,并发]
---

关于Java并发编程中的一些重要类、接口，书里整理出了以下几种：

### 线程池
线程池复用线程，在java中，我们主要使用的线程池就是`ThreadPoolExecutor`，此外还有定时的线程池`ScheduledThradPoolExecutor`。需要注意的是对于`Executors.newCachedThreadPool()`方法返回的线程池的使用，该方法返回的线程池是没有线程上限的，在使用时一定要当心，因为没有办法控制总体的线程数量，而每个线程都是消耗内存的，这可能会导致过多的内存被占用，建议尽量不要用这个方法返回线程池 ，而要使用有固定线程上限的线程池。

{% highlight java %}

public class WorkerPool {
  public static void main(String args[]) throws InterruptedException{
    //RejectedExecutionHandler implementation
    RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();

    //Get the ThreadFactory implementation to use
    ThreadFactory threadFactory = Executors.defaultThreadFactory();

    //creating the ThreadPoolExecutor
    ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);

    //start the monitoring thread
    MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
    Thread monitorThread = new Thread(monitor);
    monitorThread.start();

    //submit work to the thread pool
    for(int i=0; i<10; i++){
      executorPool.execute(new WorkerThread("cmd"+i));
    }
    Thread.sleep(30000);

    //shut down the pool
    executorPool.shutdown();

    //shut down the monitor thread
    Thread.sleep(5000);
    monitor.shutdown();
  }
}
{% endhighlight %}

### synchronized

`synchronized`关键字可以用于声明方法，也可用于声明代码块，它有有两个作用   

- 互斥
- 可见性(在一个线程中修改变量的值后，在其他线程中能够看到这个值，synchronized保证了`synchronized`块中变量的可见性)

### ReentrantLock

`ReentrantLock`是`java.util.concurrent.locks`中的一个类，是从JDK5开始加入的，`ReentrantLock`的用法类似于修饰代码段的`synchronized`，不过需要显示的进行unlock，这是容易出错的地方，假如代码出现异常而导致没有unlock，就会出现问题，为什么JDK5会加入这个类呢，原因有两个：

`ReentrackLock`提供了`tryLock`方法，`tryLock`调用的时候，如果锁被其他线程持有，那么`tryLock`会立即返回，返回的结果为false，如果没有被其他线程持有，那么当前调用的线程会持有锁，并且`tryLock`返回的结果是true
构造`RenntrantLock`对象的时候，有一个构造函数可以接受一个boolean类型的参数，那就是描述锁公平与否的函数，公平锁的好处是等待锁的线程不会饿死，但是整体效率相对低一些；非公平锁的好处是整体效率相对高一些，但是有些线程可能会饿死或者说很早就在等待锁，但要等很久才会得到锁，其中的原因是公平锁是严格按照请求锁的顺序来排队获取锁的，而非公平锁是可以抢占的，即如果在某个时刻有线程需要获取锁，而这个时候刚好锁可用，那么这个线程就会直接抢占，而这时阻塞在等待队列的线程则不会被唤醒

{% highlight java %}
lock.lock();
try{
  //don something
}finally{
  lock.unlock();
}
{% endhighlight %}

### volatile

`volatitle`是轻量级的实现变量可见性的方法，在变量前面增加`volatile`关键字就可以了，因为`volatile`只是保证了同一个变量在多线程中的可见性，所以它更多是用于修饰为开关状态的变量

{% highlight java %}
volatile int count;
HashTable<Stringj,String> hash=new HashTable(String,String)();
public void addContent(String key,String value){
  if(count < 100){
    hash.put(key,value);
    count++;
  }
}
{% endhighlight %}

上面的代码中，我们用count来计算当前进入HashTable的总数，但是这段代码是有问题的，原因是volatile虽然接解决了可见性的问题，但是不能控制并发，也就是多个线程同时执行addConent时会可能让HashTable的元素数量超过100，对于这一问题采用synchronized关键字就可以解决了，因为synchronized保证了代码块的串行执行，它有互斥的效果。

### Atomic

在JDK5中增加了`java.util.concurrent.atomic`包，这个包中是一些以Atomic开头的类，这些类主要提供一些相关的原子操作

//以`AtomicInteger`为例来看一个多线程计数器的场景，场景很简单，让多个线程都对计数器进行加1操作，我们一般可能会这样做：

{% highlight java %}
public class Conter01(){
  private int counter=0;
  public int increase(){
    synchronized(this){
      counter=counter+1;
      return counter;
    }
  }
  public int decrease(){
    synchronized(this){
      counter=counter-1;
        return counter;
    }
  }
}
{% endhighlight %}

对上面的例子采用`AtomicInteger`后，代码会变成这样

{% highlight java %}
public class Counter2{
  private AtomicInteger counter=new AtomicInteger();
  public int increase(){
    return counter.incrementAndGet();
  }
  public int decrease(){
    return counter.decrementAndGet();
  }
}
{% endhighlight %}
采用`AtomicInteger`之后代码变的简洁了，更重要的是性能得到了提升，而且是比较明显的提升，性能提升的原因主要在于`AtomicInteger`内部通过JNI的方式使用了硬件支持的CAS指令

### Future和FutureTask

`Future`是一个接口，`FutureTask`是一个具体实现类
例如：现在通过调用一个方法从远程获取一些计算结果，假设有这样一个方法：

{% highlight java %}
HahsMap getDataFromRemote();
{% endhighlight %}

如果是最传统的同步方式使用，代码大概是这样的：

{% highlight java %}
HashMap data=getDataFromRemote();
{% endhighlight %}

我们将一直等待`getDataFromRemote()`的返回，然后才能继续后面的工作，这个函数是从远程获取数据的计算结果，如果需要的时间很长，并且后面的那部分代码与这些数据没有关系的话，阻塞在这里等待结果就会比较浪费时间，那么我们有什么办法改进呢？

能够想到的办法就是调用函数后马上返回，然后继续向下执行，等需要用数据时再来取，或者说再来等待这个数据，具体事项起来有两种方式，一个是用Future,另一个用回调

{% highlight java %}
Future<HahsMap> future=getDataFromRemote2();
//do something
HashMap data=(HashMap) future.get();

//getDataFromRemote2的实现
private Future<HashMap> getDataFromRemote2(){
  return threadPool.sumit(new CallableM<HahsMap>(){
    public HahsMap call() throws Exception{
        return getDataFromRemote();
    }
  });
}
{% endhighlight %}

对于`Future`来说，除了刚才代码中的get方法外，还有一个带参数的get方法，用来设置get的等待时间，也就是进行超时设置，而不是一直等下去

### 并发容器

并发容器的思路是尽量不用锁，比较有代表性的是以`CopyOnWrite`和`Concurrent`开头的几个容器，`CopyOnWrite`的思路是在更改容器的时候把容器写一份进行修改，保证正在读的线程不受影响，这种方式用在读多写少得场景中会更好，因为实质上是在写的时候重建了一次容器，而以`Concurrent`开头的容器的具体方式则不完全相同，总体来说是尽量保证读不加锁，并且修改时不影响读，所以会达到比使用读写锁更高的并发性能
