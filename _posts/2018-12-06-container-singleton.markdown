---
layout:     post
title:      " 容器单例 "
subtitle:   " 设计模式之单例模式 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---
##### 容器单例
我们先看代码吧
```java
public class ContainerSingleton {

    private ContainerSingleton(){

    }
    private static Map<String,Object> singletonMap = new HashMap<String,Object>();

    public static void putInstance(String key,Object instance){
        if(StringUtils.isNotBlank(key) && instance != null){
            if(!singletonMap.containsKey(key)){
                singletonMap.put(key,instance);
            }
        }
    }

    public static Object getInstance(String key){
        return singletonMap.get(key);
    }
}
```
这种方式实现的单例是线程不安全的。如果需要线程安全的可以使用HashTable但是HashTable每次存取都会加上同步锁，性能损耗比较严重。或者使用ConcurrentHashMap。

##### ThreadLocal “单例“
这个单例严格意义上讲并不完全算是单例，它只能算在单个线程中的单例，也就是在同一个线程中的它是单例的。我们直接看代码吧。

```java
public class ThreadLocalInstance {
    private static final ThreadLocal<ThreadLocalInstance> threadLocalInstanceThreadLocal
             = new ThreadLocal<ThreadLocalInstance>(){
        @Override
        protected ThreadLocalInstance initialValue() {
            return new ThreadLocalInstance();
        }
    };
    private ThreadLocalInstance(){

    }
    public static ThreadLocalInstance getInstance(){
        return threadLocalInstanceThreadLocal.get();
    }

}
```
然后我们写个测试类测试一下。

我们定义一个线程T，在这个线程里面获取这个单例对象。
```java
public class T implements Runnable {
    @Override
    public void run() {
        ThreadLocalInstance instance = ThreadLocalInstance.getInstance();
        System.out.println(Thread.currentThread().getName()+"  "+instance);
    }
}
```
然后是测试类包含主函数。

```java
public class SingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        System.out.println("main thread"+ThreadLocalInstance.getInstance());
        System.out.println("main thread"+ThreadLocalInstance.getInstance());
        System.out.println("main thread"+ThreadLocalInstance.getInstance());
        System.out.println("main thread"+ThreadLocalInstance.getInstance());
        System.out.println("main thread"+ThreadLocalInstance.getInstance());
        System.out.println("main thread"+ThreadLocalInstance.getInstance());
        Thread t1 = new Thread(new T());
        Thread t2 = new Thread(new T());
        t1.start();
        t2.start();
        System.out.println("program end");
    }
```
我们再看一下运行结果。
```
main threadcom.design.pattern.creational.singleton.ThreadLocalInstance@12edcd21
main threadcom.design.pattern.creational.singleton.ThreadLocalInstance@12edcd21
main threadcom.design.pattern.creational.singleton.ThreadLocalInstance@12edcd21
main threadcom.design.pattern.creational.singleton.ThreadLocalInstance@12edcd21
main threadcom.design.pattern.creational.singleton.ThreadLocalInstance@12edcd21
main threadcom.design.pattern.creational.singleton.ThreadLocalInstance@12edcd21
program end
Thread-0  com.design.pattern.creational.singleton.ThreadLocalInstance@3ec28870
Thread-1  com.design.pattern.creational.singleton.ThreadLocalInstance@7a56389b
```
从结果我们可以看出这个单例之算是个伪单例，只能在一个线程里面实现单例。