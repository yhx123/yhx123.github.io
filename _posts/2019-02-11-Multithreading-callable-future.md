---
layout:     post
title:      " Java多线程-Callable和Future "
subtitle:   " java  Callable和Future使用 "
date:       2019-02-11 12:00:00
author:     "Honson"
header-img: ""
tags:
 - java
 - 多线程
---

#### Callable和Future出现的原因
创建线程的2种方式，一种是直接继承Thread，另外一种就是实现Runnable接口。
这2种方式都有一个缺陷就是：在执行完任务之后无法获取执行结果。
如果需要获取执行结果，就必须通过共享变量或者使用线程通信的方式来达到效果，这样使用起来就比较麻烦。
自从Java 1.5开始，就提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。
#### Callable和Future介绍
Callable接口代表一段可以调用并返回结果的代码;Future接口表示异步任务，是还没有完成的任务给出的未来结果。所以说Callable用于产生结果，Future用于获取结果。
Callable接口使用泛型去定义它的返回类型。Executors类提供了一些有用的方法在线程池中执行Callable内的任务。由于Callable任务是并行的（并行就是整体看上去是并行的，其实在某个时间点只有一个线程在执行），我们必须等待它返回的结果。
java.util.concurrent.Future对象为我们解决了这个问题。在线程池提交Callable任务后返回了一个Future对象，使用它可以知道Callable任务的状态和得到Callable返回的执行结果。Future提供了get()方法让我们可以等待Callable结束并获取它的执行结果。
#### Callable与Runnable
java.lang.Runnable吧，它是一个接口，在它里面只声明了一个run()方法：

```java
public interface Runnable {
    public abstract void run();
}
```
　由于run()方法返回值为void类型，所以在执行完任务之后无法返回任何结果。

　Callable位于java.util.concurrent包下，它也是一个接口，在它里面也只声明了一个方法，只不过这个方法叫做call()：

```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```
这是一个泛型接口，call()函数返回的类型就是传递进来的V类型。

#### Callable的使用
一般情况下是配合ExecutorService来使用的，在ExecutorService接口中声明了若干个submit方法的重载版本。
```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```
第一个submit方法里面的参数类型就是Callable。

暂时只需要知道Callable一般是和ExecutorService配合来使用的，具体的使用方法讲在后面讲述。

一般情况下我们使用第一个submit方法和第三个submit方法，第二个submit方法很少使用。
#### Future
Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

Future类位于java.util.concurrent包下，它是一个接口：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
##### 在Future接口中声明了5个方法，下面依次解释每个方法的作用
* cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

* isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

* isDone方法表示任务是否已经完成，若任务完成，则返回true；

* get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

* get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

##### Future提供了三种功能：
1. 判断任务是否完成；
2. 能够中断任务；
3. 能够获取任务执行结果。

 因为Future只是一个接口，所以是无法直接用来创建对象使用的，因此就有了下面的FutureTask。

 #### FutureTask
 ##### FutureTask实现了RunnableFuture接口，这个接口的定义如下：

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```
可以看到这个接口实现了Runnable和Future接口，接口中的具体实现由FutureTask来实现。这个类的两个构造方法如下 ：

```java
public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        sync = new Sync(callable);
    }
    public FutureTask(Runnable runnable, V result) {
        sync = new Sync(Executors.callable(runnable, result));
    }
```
如上提供了两个构造函数，一个以Callable为参数，另外一个以Runnable为参数。这些类之间的关联对于任务建模的办法非常灵活，允许你基于FutureTask的Runnable特性（因为它实现了Runnable接口），把任务写成Callable，然后封装进一个由执行者调度并在必要时可以取消的FutureTask。

FutureTask可以由执行者调度，这一点很关键。它对外提供的方法基本上就是Future和Runnable接口的组合：get()、cancel、isDone()、isCancelled()和run()，而run()方法通常都是由执行者调用，我们基本上不需要直接调用它。

##### FutureTask的例子

```java
public class MyCallable implements Callable<String> {
    private long waitTime;
    public MyCallable(int timeInMillis){
        this.waitTime=timeInMillis;
    }
    @Override
    public String call() throws Exception {
        Thread.sleep(waitTime);
        //return the thread name executing this callable task
        return Thread.currentThread().getName();
    }

}
```

```java
public class FutureTaskExample {
     public static void main(String[] args) {
        MyCallable callable1 = new MyCallable(1000);                       // 要执行的任务
        MyCallable callable2 = new MyCallable(2000);

        FutureTask<String> futureTask1 = new FutureTask<String>(callable1);// 将Callable写的任务封装到一个由执行者调度的FutureTask对象
        FutureTask<String> futureTask2 = new FutureTask<String>(callable2);

        ExecutorService executor = Executors.newFixedThreadPool(2);        // 创建线程池并返回ExecutorService实例
        executor.execute(futureTask1);  // 执行任务
        executor.execute(futureTask2);

        while (true) {
            try {
                if(futureTask1.isDone() && futureTask2.isDone()){//  两个任务都完成
                    System.out.println("Done");
                    executor.shutdown();                          // 关闭线程池和服务
                    return;
                }

                if(!futureTask1.isDone()){ // 任务1没有完成，会等待，直到任务完成
                    System.out.println("FutureTask1 output="+futureTask1.get());
                }

                System.out.println("Waiting for FutureTask2 to complete");
                String s = futureTask2.get(200L, TimeUnit.MILLISECONDS);
                if(s !=null){
                    System.out.println("FutureTask2 output="+s);
                }
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }catch(TimeoutException e){
                //do nothing
            }
        }
    }
}
```
