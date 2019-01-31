---
layout:     post
title:      " 简单工厂 "
subtitle:   " 设计模式之工厂模式 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---



#### 简单工厂
##### 定义
> 用一个工厂对象决定创建哪一种产品的实例。
##### 类型
> 创建型，但不属于GOF23中设计模式。
##### 使用场景
> 工厂类负责创建的对象比较少，客户端（应用层）只知道传入工厂类的参数，对于如何创建对象（逻辑）不关心
##### 优点
> 只需要传入一个正确的参数，就可以获取你所需要的对象二无需知道其创建细节。
##### 简单工厂类的缺点 
> 工厂类的职责相对过重，增加新的产品，需要修改工厂类的逻辑判断（随着业务不断增加工厂类的业务逻辑将变难以令人理解），违背开闭原则，无法形成基于继承的等级结构。

* 这些模式有些可能会存在冲突，所以我们在开发是需要根据实际产品进行衡量决定。 

下面看演示代码
> 假设有个场景我们有很多视频需要录制。我们定义个抽象类，叫vido同时有个方法produce。
```java
public abstract class Video {
    public abstract void produce();

}
```
这个抽象类有一些子类，比如javaVido，PythonVido，

```java
public class JavaVideo extends Video {
    @Override
    public void produce() {
        System.out.println("录制Java课程视频");
    }
}

```

```java
public class PythonVideo extends Video {
    @Override
    public void produce() {
        System.out.println("录制Python课程视频");
    }
}
```
假设我们要获取这两个类的对象，当然我们可以new这两个类来获取对象，我现在是介绍工程模式，自然要使用工厂模式来创建。我们现在创建一个工厂类

```java
public class VideoFactory {
  public Video getVideo(String type){
        if("java".equalsIgnoreCase(type)){
            return new JavaVideo();
        }else if("python".equalsIgnoreCase(type)){
            return new PythonVideo();
        }
        return null;
    }
}
```
现在写一个测试类看看这个工厂类怎么使用。

```java
public class FactoryTest {
    public static void main(String[] args) {
        VideoFactory videoFactory = new VideoFactory();
        Video video = videoFactory.getVideo("java");
        if(video == null){
            return;
        }
        video.produce();
    }
```
现在来说说这样写的缺点：这样写我们每次新增加类都要添加工厂类的代码，久而久之工厂就会变得非常复杂。下面我们改进一下工厂。
```java
public class VideoFactory {
    public Video getVideo(Class c){
        Video video = null;
        try {
            video = (Video) Class.forName(c.getName()).newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return video;
    }

}

```
再看测试类怎么写
```java
public class FactoryTest {
        VideoFactory videoFactory = new VideoFactory();
        Video video = videoFactory.getVideo(JavaVideo.class);
        if(video == null){
            return;
        }
        video.produce();
    }
}
```
本文最后我们说一下改进之后的好处。
我们class类获取对象的好处就是不用每次添加对象都去修改工厂类。
这些例子的uml图如下





