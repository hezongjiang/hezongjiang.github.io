---
title: Java 多线程详解
catalog: true
date: 2019-11-13 15:40:58
subtitle:
header-img: timg.jpeg
tags: 服务端开发
---
## 一、线程与进程

**进程**是资源(CPU、内存等)分配的基本单位，它是程序执行时的实例。程序运行时系统就会创建一个进程，并为它分配资源，然后把该进程放入进程就绪队列，进程调度器选中它的时候就会为它分配 CPU 时间，程序开始真正运行。

**线程**是程序执行时的最小单位，它是进程的一个执行流，是 CPU 调度和分派的基本单位，进程由线程组成，线程间共享进程的所有资源，每个线程有自己的堆栈和局部变量。

**线程与进程区别：**

* 进程是资源分配的最小单位，线程是程序执行的最小单位。

* 进程有自己的独立地址空间；线程是共享进程中的数据的，使用相同的地址空间。因此 CPU 切换线程的花费比进程小很多，同时创建线程的开销也比进程小很多。

* 线程之间的通信更方便，同一进程下的线程共享全局变量、静态变量等数据，而进程之间的通信需要以通信的方式(IPC)进行。

* 多进程程序更健壮，一个进程死掉并不会对另外一个进程造成影响，因为进程有自己独立的地址空间。而多线程程序只要有一个线程死掉，整个进程也死掉了。

## 二、线程的状态

* 新建(New)：创建后尚未启动，如：`Thread thread = new Thread()`。

* 可运行(Runnable)：可能正在运行，也可能正在等待 CPU 时间片。包含了操作系统线程状态中的 Running 和 Ready。如：`thread.start()`。

* 阻塞(Blocked)：等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

* 无限期等待(Waiting)：等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。如调用 `thread.wait()`、`thread.join()` 方法能让线程进入无限期等待状态。

> 阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的。

* 限期等待(Timed Waiting)：无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。如 `Thread.sleep(10)`。

> **`sleep()` 和 `wait()` 的区别：**
1、sleep() 是 Thread 的方法，wait() 是 Object 的方法；
2、sleep() 可以在任意地方调用，wait() 必须在同步方法或者同步代码块中调用；
3、sleep() 只会让出 CPU，不会导致锁的改变；
4、wait() 不仅会让出 CPU，还会释放锁。


* 死亡(Terminated)：可以是线程结束任务之后自己结束，或者产生了异常而结束。死亡的线程不能再次调用 `start()` 方法。

![线程状态转换](https://tva1.sinaimg.cn/large/006y8mN6gy1g8wgj3pjb4j30qj0e3myx.jpg)

## 三、线程的创建
有三种使用线程的方法：
1. 继承 Thread 类。
2. 实现 Runnable 接口；
3. 实现 Callable 接口；

**1、继承 Thread 类**

需要实现 `run()` 方法，因为 Thread 类也实现了 Runable 接口。

当调用 `start()` 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 `run()` 方法。

```
public static void main(String[] args) {
      Thread thread = new Thread(){
          @Override
          public void run() {
              System.out.println("do something");
          }
      };
      thread.start();
}
```

**2、实现 Runnable 接口**

实现 Runnable 接口中的 `run()` 方法，通过 Thread 调用 `start()` 方法来启动线程。

```
public class MyRunnable implements Runnable {
      public void run() {
          System.out.println("do something");
      }
}
```
```
public static void main(String[] args) {
      MyRunnable instance = new MyRunnable();
      Thread thread = new Thread(instance);
      thread.start();
}
```
**3、实现 Callable 接口**

与 Runnable 相比，**Callable 可以有返回值**，返回值通过 FutureTask 进行封装。

```
public class MyCallable implements Callable<Integer> {
      public Integer call() {
          return 123;
      }
}
```

```
public static void main(String[] args) throws ExecutionException, InterruptedException {
      MyCallable mc = new MyCallable();
      FutureTask<Integer> ft = new FutureTask<>(mc);
      Thread thread = new Thread(ft);
      thread.start();
      System.out.println(ft.get()); // 获取返回值
}
```

## 四、线程池

多线程的异步执行方式，虽然能够最大限度发挥多核计算机的计算能力，但是如果不加控制，反而会对系统造成负担。线程本身也要占用内存空间，大量的线程会占用内存资源并且可能会导致 Out of Memory。即便没有这样的情况，大量的线程回收也会给 GC 带来很大的压力。

**为了避免重复的创建线程，线程池可以让线程进行复用。**通俗点讲，当有工作来，就会向线程池拿一个线程，当工作完成后，并不是直接关闭线程，而是将这个线程归还给线程池供其他任务使用。

先看一下 ThreadPoolExecutor，它是线程池中最核心的类，这里着重说一下这个类的各个构造参数：

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

* **`corePoolSize`**
 核心线程数，线程池创建之后，线程数为 0，当有任务时，会创建一个线程去执行，直到线程数达到 corePoolSize 之后，就会被到达的任务放在队列中。换句更精炼的话：corePoolSize 表示允许线程池中允许同时运行的最大线程数。

* **`maximumPoolSize`**
线程池允许的最大线程数，表示最大能创建多少个线程。maximumPoolSize >= corePoolSize。

* **`keepAliveTime`**
表示线程没有任务时最多保持 keepAliveTime 后停止。默认情况下，只有线程池中线程数大于 corePoolSize 时，keepAliveTime 才会起作用。当线程池中的线程数大于 corePoolSize，并且一个线程空闲时间达到了 keepAliveTime，那么该线程就会 shutdown。

* **`Unit`**
keepAliveTime 的单位，通常是 `TimeUnit.SECONDS`。

* **`workQueue`**
一个阻塞队列，用来存储等待执行的任务，当线程池中的线程数超过 corePoolSize 的时候，线程会进入阻塞队列进行阻塞等待。总之，来不及处理的任务的队列，是一个 BlockingQueue。

* **`threadFactory`**
线程工厂，用来生产一组相同任务的线程。

* **`handler`**
表示拒绝处理任务时的策略。如果当前线程池中的线程数目超过 maximumPoolSize，则会采取任务拒绝策略进行处理。
策略默认 *AbortPolicy*，表无法处理新任务并抛出 RejectedExecutionException 异常。
另外还有 *CallerRunsPolicy*：该策略直接在调用者线程中，运行当前被丢弃的任务。但线程的性能极有可能会急剧下降。
*DiscardOldestPolicy*：丢弃队列中最老的一个请求，即将被执行的一个任务，并尝试再次提交当前任务。
*DiscardPolicy*：丢弃任务，不做任何处理。

![线程池任务处理策略](https://tva1.sinaimg.cn/large/006y8mN6gy1g8wgjk1pfwj30jl09paa1.jpg)

ThreadPoolExecutor 的使用：
```
public class PollTest {

    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = Math.max(4, Math.min(CPU_COUNT - 1, 5));
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 2;
    private static final BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(10);

    public static void main(String[] args) throws Exception {

        System.out.println("核心线程数=" + CORE_POOL_SIZE);
        System.out.println("最大线程数=" + MAXIMUM_POOL_SIZE);

        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, 10, TimeUnit.SECONDS, workQueue, new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setDaemon(false);
                return thread;
            }
        });

        for (int i = 0; i < 20; i++) {
            Runnable runnable = () -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("执行完毕" + Thread.currentThread().getName());
            };
            threadPoolExecutor.execute(runnable);
        }
    }
}
```

## 五、几种常见的线程池

**1、newFixedThreadPool (固定数量线程池)**
```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```
corePoolSize = maximumPoolSize = 初始化的参数

workQueue：使用无界队列 LinkedBlockingQueue 链表阻塞队列。

keepAliveTime = 0，由于使用无界队列 LinkedBlockingQueue 作为缓存队列，所以当 corePoolSize 满后，后面添加的线程任务都会添加到 LinkedBlockingQueue 中去，所以 maximumPoolSize 就失去了意义，这样也就没有必要设置空闲时间。

```
public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread());
                }
            });
        }
        executorService.shutdown();
}
```

**2、newSingleThreadExecutor (单例线程池)**

```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}
```

corePoolSize = maximumPoolSize =1  由于是单例线程池，所以线程池中是有一个重用的线程

workQueue：使用无界队列LinkedBlockingQueue链表阻塞队列

**3、newCachedThreadPool (缓存线程池)**

```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

corePoolSize = 0 表示线程池中没有核心线程，都是非核心线程

maximumPoolSize = Integer.最大值

keepAliveTime：60秒，线程池中创建的线程都是非核心线程，所以设置空闲时间 60 秒，当非核心线程60秒后没有被重用，将会被销毁，如果没有线程提交给该线程池，超过空闲时间，该线程池就没有非空闲线程，那么该线程池也就不会消耗过多的资源。

workQueue = SynchronousQueue 是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。

**4、newScheduledThreadPool (定时线程池)**
```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());
}
```
定时线程池，该线程池可用于周期性地去执行任务，通常用于周期性的同步数据。

scheduleAtFixedRate:是以固定的频率去执行任务，周期是指每次执行任务成功执行之间的间隔。

schedultWithFixedDelay:是以固定的延时去执行任务，延时是指上一次执行成功之后和下一次开始执行的之前的时间。