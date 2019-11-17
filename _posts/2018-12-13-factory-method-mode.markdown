---
layout:     post
title:      " 工厂方法模式 "
subtitle:   " 设计模式之工厂模式 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - Design Patterns
---




#### 工厂方法
##### 定义
创建一个对象的接口（抽象类），但是让实现（继承）这个接口（抽象类）的类来决定实例化哪个类，工厂方法让类的实例化推迟到子类中进行。
##### 类型 
创建型
##### 适用场景
* 创建对象需要大量的重复代码
* 客户端（应用层）不依赖于产品实例如何被创建，实现等细节。
* 一个类通过其子类来指定创建哪个对象。

> 创建对象会产生大量的重复代码，工厂方法通过创建一个方法来交由子类创建对象。

##### 工厂方法-优点
* 用户只需要关心产品对应的工厂，无需关心创建细节
* 加入新的产品符合开闭原则，提高可拓展性
##### 工厂方法-缺点
* 类的个数过多，增加复杂度
* 增加了系统的抽象性和理解难度
下面我们开始coding，场景我们依然使用上节中的简单工厂场景

首先我们常见一个抽象类使用接口也是可以的。抽象类的作用是创建约束。
```java
public abstract class Video {
    public abstract void produce();

}
```

同样的我们再创建一个抽象类
```java
public abstract class VideoFactory {
    public abstract Video getVideo();
    }

```
我们的javavideo类，javaVideo类继承抽象Video
```java
public class JavaVideo extends Video {
    @Override
    public void produce() {
        System.out.println("录制Java课程视频");
    }
}

```
javaVideo的工厂类，继承工厂类。我们的javaVideo的创建就是由这个类来完成。
```java
public class JavaVideoFactory extends VideoFactory {
    @Override
    public Video getVideo() {
        return new JavaVideo();
    }
}
```
假设我们还有个python课和前端课程，我们同样的方式创建这些类
```java

public class PythonVideo extends Video {
    @Override
    public void produce() {
        System.out.println("录制Python课程视频");
    }
}
```

```java
public class PythonVideoFactory extends VideoFactory {
    @Override
    public Video getVideo() {
        return new PythonVideo();
    }
}
```

```java
public class FEVideo extends Video{
    @Override
    public void produce() {
        System.out.println("录制FE课程视频");
    }
}
```

```java
public class FEVideoFactory extends VideoFactory{
    @Override
    public Video getVideo() {
        return new FEVideo();
    }
}
```
此时我们应用层的test类就应该这么写

```java
public class MethodTest {
    public static void main(String[] args) {
        VideoFactory videoFactory = new PythonVideoFactory();
        VideoFactory videoFactory2 = new JavaVideoFactory();
        VideoFactory videoFactory3 = new FEVideoFactory();
        Video video = videoFactory.getVideo();
        video.produce();

    }

}
```
最后我们来看一下这些类的uml图
![](https://user-gold-cdn.xitu.io/2018/12/5/1677ec733e8aeb22?w=950&h=534&f=png&s=447612)
从图中我们可以清晰的看出这些类之间的依赖关系，同时工厂方法类过多的缺点也展示出来了。


下面我们一起来看一下在jdk中有些类使用工厂方法的这个设计模式吧。
比如这个Collection接口中的iterator方法就是一个工厂方法，我们使用idea按住ctrl+鼠标左键查看他的实现。
```java
.......
    /**
     * Returns an iterator over the elements in this collection.  There are no
     * guarantees concerning the order in which the elements are returned
     * (unless this collection is an instance of some class that provides a
     * guarantee).
     *
     * @return an <tt>Iterator</tt> over the elements in this collection
     */
    Iterator<E> iterator();
......
```
如图

![](https://user-gold-cdn.xitu.io/2018/12/5/1677ed6cf1287e7d?w=1366&h=582&f=png&s=103008)

我们就查看一下ArrayList吧
```java
 /**
     * Returns an iterator over the elements in this list in proper sequence.
     *
     * <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @return an iterator over the elements in this list in proper sequence
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
```
这里使用一个内部类Itr来实现这个工厂方法。
从工厂方法的角度去看这个类的话，我们可以认为ArrayList就是一个具体实现工厂，Iterator接口就是抽象产品，而具体生产出来的产品就是Itr对象了。对比我们的Collection接口就相当于我们的VideoFactory类，而ArrayList就相当我们代码中的javaFactory等具体工厂实现。Iterator就想相当于我们代码中的抽象产品Video， itr是具体产品就相当于我们代码中的javaVideo等具体产品。

工厂方法模式今天就聊到这里。


