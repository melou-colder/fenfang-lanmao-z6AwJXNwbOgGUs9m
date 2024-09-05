
# Java系列


[Java核心知识体系1：泛型机制详解](https://github.com "Java核心知识体系1：泛型机制详解")
[Java核心知识体系2：注解机制详解](https://github.com "Java核心知识体系2：注解机制详解")
[Java核心知识体系3：异常机制详解](https://github.com "Java核心知识体系3：异常机制详解")
[Java核心知识体系4：AOP原理和切面应用](https://github.com "Java核心知识体系4：AOP原理和切面应用")
[Java核心知识体系5：反射机制详解](https://github.com "Java核心知识体系5：反射机制详解"):[西部世界官网](https://tianchuang88.com)
[Java核心知识体系6：集合框架详解](https://github.com "Java核心知识体系6：集合框架详解")
[Java核心知识体系7：线程不安全分析](https://github.com "Java核心知识体系7：线程不安全分析")
[Java核心知识体系8：Java如何保证线程安全性](https://github.com "Java核心知识体系8：Java如何保证线程安全性")


# 1 先导


![image](https://img2024.cnblogs.com/blog/167509/202408/167509-20240831095812155-1979318789.png "点击查看大图")
Java线程基础主要包含如下知识点，相信我们再面试的过程中，经常会遇到类似的提问。


1. 线程有哪几种状态? 线程之间如何转变？
2. 线程有哪几种实现方式? 各优缺点？
3. 线程的基本操作（线程管理机制）有哪些?
4. 线程如何中断?
5. 线程有几种互斥同步方式? 如何选择?
6. 线程之间的协作方式（通信和协调）?


下面我们 一 一 解读。


# 2 线程的状态和流转


![image](https://img2024.cnblogs.com/blog/167509/202408/167509-20240831103045218-865612816.png "点击查看大图")


## 2\.1 新建(New)


如上图，创建完线程，但尚未启动。


## 2\.2 可运行(Runnable)


如上图，处于可运行阶段，正在运行，或者正在等待 CPU 时间片。包含了 `Running` 和 `Ready` 两种线程状态。


## 2\.3 阻塞(Blocking)


如上图，正被Lock住，等待获取一个排它锁，如果其他的线程释放了锁，该状态就会结束。


## 2\.4 无限期等待(Waiting)


如上图，处在无限期等待阶段，等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。
主要有两种方式进行释放：


* 调用方的线程执行完成
* 使用 Object.notify() / Object.notifyAll()进行显性唤醒


## 2\.5 限期等待(Timed Waiting)


如上图，因为有时间控制，所以无需等待其它线程显式地唤醒，一定时间之后，系统会自动唤醒。
所以他有三种方式进行释放：
主要有两种方式进行释放：


* 调用方的线程执行完成
* 使用 Object.notify() / Object.notifyAll()进行显性唤醒
* 时间到结束
	+ Thread.sleep()
	+ Object.wait() 方法，带Timeout参数
	+ Thread.join() 方法，带Timeout参数


## 2\.6 死亡(Terminated)


* 线程结束任务之后结束
* 产生了异常并结束


# 3 线程实现方式


在Java中，线程的实现方式主要有两种：继承`Thread`类和实现`Runnable`接口。此外，Java 5开始，引入了`java.util.concurrent`包，提供了更多的并发工具，如`Callable`接口与`Future`接口，它们主要用于任务执行。


## 3\.1 继承Thread类


通过继承`Thread`类来创建线程是最基本的方式。你需要创建一个扩展自`Thread`类的子类，并重写其`run()`方法。然后，可以创建该子类的实例来创建新的线程。



```
class MyThread extends Thread {
    public void run() {
        System.out.println("线程运行中");
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start(); // 调用start()方法来启动线程
    }
}

```

## 3\.2 实现Runnable接口


另一种方式是让你的类实现`Runnable`接口，并实现`run()`方法。然后，你可以创建`Thread`类的实例，将实现了`Runnable`接口的类的实例作为构造参数传递给它。



```
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("线程运行中");
    }
}

public class RunnableDemo {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start(); // 调用start()方法来启动线程
    }
}

```

## 3\.3 使用Callable和Future


虽然`Callable`和`Future`不是直接用于创建线程的，但它们提供了一种更灵活的方式来处理线程执行的结果。`Callable`类似于`Runnable`，但它可以返回一个结果，并且可以抛出异常。`Future`用于获取`Callable`执行的结果。



```
import java.util.concurrent.*;

class MyCallable implements Callable {
    public String call() throws Exception {
        return "任务完成";
    }
}

public class CallableDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future future = executor.submit(new MyCallable());
        System.out.println(future.get()); // 阻塞等待获取结果
        executor.shutdown();
    }
}

```

## 3\.4 优缺点解读


* **继承Thread类**：简单直观，但Java不支持多重继承，如果类已经继承了其他类，则不能再用这种方式。另外继承整个 Thread 类开销过大，太重了。
* **实现Runnable接口**：更加灵活，推荐的方式。
* **Callable和Future**：提供了更为强大的功能，例如返回执行结果和抛出异常，但通常用于与`ExecutorService`等高级并发工具一起使用。


# 4 线程管理机制


Java 中的线程管理机制非常强大，涵盖了从简单的线程创建到复杂的线程池管理等多个方面。


## 4\.1 Executor 框架


`Executor` 框架是 Java 并发包（`java.util.concurrent`）中的一个关键组件，它提供了一种更高级别的抽象来管理线程池。通过使用 `Executor`，你可以更容易地控制线程的创建、执行、调度、生命周期等。它主要有三种类型：


1. CachedThreadPool: 一个任务创建一个线程
2. FixedThreadPool: 所有任务只能使用固定大小的线程
3. SingleThreadExecutor: 单个线程，相当于大小为 1 的 FixedThreadPool。


* **优点**：提高程序性能和响应速度，通过复用线程来减少线程创建和销毁的开销，简化并发编程。
* **使用示例**：
```
ExecutorService executor = Executors.newFixedThreadPool(5);
for (int i = 0; i < 10; i++) {
    Runnable worker = new WorkerThread("" + i);
    executor.execute(worker);
}
executor.shutdown();

```


## 4\.2 守护线程（Daemon Threads）


守护线程是一种特殊的线程，它主要用于程序中“后台”任务的支持。守护线程与普通线程的区别在于，当程序中所有非守护线程结束时，JVM 会自动退出，即使还有守护线程在运行。守护线程常用于垃圾回收、JVM 内部的监控等任务。
**设置守护线程**：通过调用线程对象的 `setDaemon(true)` 方法，在启动线程之前将其设置为守护线程。



```
 Thread thread = new Thread(new MyRunnable());
 thread.setDaemon(true);

```

## 4\.3 sleep() 方法


`sleep()` 方法是 `Thread` 类的一个静态方法，用于让当前正在执行的线程暂停执行指定的时间（毫秒），以毫秒为单位。在指定的时间过去后，线程将回到可运行状态，等待CPU的调度。


* **用途**：常用于线程间的简单同步。
* **注意**：`sleep()` 方法不会释放锁（如果当前线程持有锁的话）。
* **示例**：



```
 try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

```

## 4\.4 yield() 方法


`yield()` 方法也是 `Thread` 类的一个静态方法，它告诉调度器当前线程愿意放弃当前处理器的使用，但这并不意味着线程会立即停止执行或进入等待/阻塞状态。
调度器可以忽略这个提示，继续让当前线程运行。


* **用途**：提示调度器让出CPU时间，但具体是否让出取决于调度器的实现。
* **注意**：`yield()` 方法不会使线程进入阻塞状态，也不会释放锁（如果持有的话）,类似仅建议。
* **示例**：



```
Thread.yield();

```

# 5 线程中断方式


在Java中，线程中断是一种重要的线程间通信机制，用于通知线程应该停止当前正在执行的任务。线程中断的方式主要有以下几种：


## 5\.1 使用`interrupt()`方法


`interrupt()`方法是Java推荐的线程中断方式。它并不会直接停止线程，而是设置线程的中断状态为true。线程需要定期检查这个中断状态（通过`isInterrupted()`方法），并根据需要自行决定如何响应中断请求，比如退出循环、释放资源等。


* **优点**：安全、灵活，符合Java的并发编程理念。
* **示例**：
```
Thread thread = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        // 执行任务
    }
    // 线程中断后的清理工作
});
thread.start();
// 稍后中断线程
thread.interrupt();

```


## 5\.2 使用`Executor`的中断操作


1. 调用 Executor 的 shutdown() 方法，会等待线程都执行完毕之后再关闭
2. 调用 Executor 的 shutdownNow() 方法，则相当于直接调用具体线程的 interrupt() 方法


# 6 线程互斥同步方式


Java中的线程互斥同步是并发编程中的一个重要概念，用于保证多个线程在访问共享资源时的互斥性，即同一时间只有一个线程能够访问某个资源。Java提供了多种机制来实现线程的互斥同步，主要包括以下几种方式：


## 6\.1 synchronized关键字


**1\. 基本概念**：synchronized是Java中最基本的同步机制，它可以用来修饰方法或代码块。当一个线程访问一个被synchronized修饰的方法或代码块时，其他试图访问该方法或代码块的线程将被阻塞，直到当前线程执行完毕释放锁。
**2\. 使用方法**：


* 修饰方法：直接在方法声明上加上synchronized关键字，例如`public synchronized void method() {...}`。
* 修饰代码块：将需要同步的代码放在synchronized(对象) {...}中，这里的对象就是锁对象，例如`synchronized(this) {...}`或`synchronized(某个对象) {...}`。


**3\. 特性**：


* 可见性：synchronized不仅保证了互斥性，还保证了变量的可见性。当一个线程释放锁时，会将锁变量的值刷新到主存储器中，从而使其他线程可以看到最新的变量值。
* 可重入性：synchronized支持可重入性，即同一个线程可以多次获取同一个锁，而不会导致死锁。


**4\. 示例**：



```
public class Counter {  
    private int count = 0;  
  
    // synchronized修饰方法  
    public synchronized void increment() {  
        count++;  
    }  
  
    public synchronized int getCount() {  
        return count;  
    }  
}  
  
public class TestSynchronized {  
    public static void main(String[] args) throws InterruptedException {  
        Counter counter = new Counter();  
  
        Thread t1 = new Thread(() -> {  
            for (int i = 0; i < 10; i++) {  
                counter.increment();  
            }  
        });  
  
        Thread t2 = new Thread(() -> {  
            for (int i = 0; i < 10; i++) {  
                counter.increment();  
            }  
        });  
  
        t1.start();  
        t2.start();  
  
        t1.join();  
        t2.join();  
  
        System.out.println("Final count: " + counter.getCount());  
    }  
}

```

## 6\.2 ReentrantLock类


* **基本概念**：ReentrantLock是java.util.concurrent.locks包中的一个可重入锁，它提供了比synchronized更灵活的锁定机制。
* **使用方法**：
	+ 创建锁对象：`ReentrantLock lock = new ReentrantLock();`
	+ 加锁：`lock.lock();`
	+ 释放锁：通常将释放锁的代码放在finally块中，以确保锁一定会被释放，例如`try {...} finally { lock.unlock(); }`。
* **特性**：
	+ 支持公平锁和非公平锁：通过构造器参数可以指定使用哪种锁，默认是非公平锁。
	+ 支持尝试获取锁：提供了`tryLock()`等方法，尝试获取锁，如果获取不到则不会阻塞线程。
	+ 支持中断锁定的线程：与synchronized不同，ReentrantLock的锁可以被中断。



```
import java.util.concurrent.locks.ReentrantLock;  
  
public class CounterWithLock {  
    private int count = 0;  
    private final ReentrantLock lock = new ReentrantLock(); // 创建ReentrantLock对象  
  
    public void increment() {  
        lock.lock(); // 加锁  
        try {  
            count++;  
        } finally {  
            lock.unlock(); // 释放锁，放在finally块中确保一定会被释放  
        }  
    }  
  
    public int getCount() {  
        lock.lock(); // 加锁  
        try {  
            return count;  
        } finally {  
            lock.unlock(); // 释放锁  
        }  
    }  
}  
  
public class TestReentrantLock {  
    public static void main(String[] args) throws InterruptedException {  
        CounterWithLock counter = new CounterWithLock();  
  
        Thread t1 = new Thread(() -> {  
            for (int i = 0; i < 10000; i++) {  
                counter.increment();  
            }  
        });  
  
        Thread t2 = new Thread(() -> {  
            for (int i = 0; i < 10000; i++) {  
                counter.increment();  
            }  
        });  
  
        t1.start();  
        t2.start();  
  
        t1.join();  
        t2.join();  
  
        System.out.println("Final count: " + counter.getCount());  
    }  
}

```

## 6\.3 对比


对于大多数简单场景，synchronized关键字是最直接、最简单的选择；而对于需要更灵活控制锁的场景，则可以考虑使用ReentrantLock等高级同步机制。


# 7 线程协作（通信）方案


Java中线程之间的协作主要可以通过多种机制实现，其中等待/通知机制（`wait/notify/notifyAll`）和`join`方法是两种常用的方式。下面我将分别给出这两种方式的简单代码示例。


## 7\.1 等待/通知机制（wait/notify/notifyAll）


等待/通知机制依赖于Java中的`Object`类，因为`wait()`, `notify()`, 和 `notifyAll()` 方法都定义在`Object`类中。这些方法必须在同步块或同步方法中被调用，因为它们是用来控制对某个对象的访问的。


**示例代码**：



```
public class WaitNotifyExample {
    private final Object lock = new Object();
    private boolean ready = false;

    public void doWait() {
        synchronized (lock) {
            while (!ready) {
                try {
                    lock.wait(); // 当前线程等待
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt(); // 保持中断状态
                }
            }
            // 当ready为true时，继续执行
        }
    }

    public void doNotify() {
        synchronized (lock) {
            ready = true;
            lock.notify(); // 唤醒在此对象监视器上等待的单个线程
            // 或者使用 lock.notifyAll(); 唤醒所有等待的线程
        }
    }

    public static void main(String[] args) {
        WaitNotifyExample example = new WaitNotifyExample();

        Thread t1 = new Thread(() -> {
            System.out.println("Thread 1 is waiting");
            example.doWait();
            System.out.println("Thread 1 is proceeding");
        });

        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(1000); // 假设t2需要一些时间来完成准备工作
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Thread 2 is notifying");
            example.doNotify();
        });

        t1.start();
        t2.start();
    }
}

```

在这个例子中，`t1`线程在`doWait()`方法中等待，直到`t2`线程调用`doNotify()`方法并设置`ready`为`true`。`t2`线程模拟了一些准备工作，并在之后唤醒`t1`。


## 7\.2 Join 方法


`join`方法是`Thread`类的一个方法，用于让当前线程等待另一个线程完成其执行。


**示例代码**：



```
public class JoinExample {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(1000); // 假设t1执行需要一些时间
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Thread 1 completed");
        });

        t1.start();

        try {
            t1.join(); // 当前线程（main线程）等待t1完成
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println("Thread 1 has joined, continuing main thread");
    }
}

```

在这个例子中，`main`线程启动了一个新线程`t1`，并通过调用`t1.join()`等待`t1`完成。`t1`线程在完成后会打印一条消息，而`main`线程会在`t1`完成后继续执行并打印另一条消息。


# 8 总结


总结一下，我们讲了让如下内容


1. 线程流转状态
2. 线程实现方式
3. 线程基本操作
4. 线程中断方案
5. 线程互斥同步方法
6. 线程协作（通信）方案


