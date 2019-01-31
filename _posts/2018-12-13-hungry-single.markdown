---
layout:     post
title:      " 饿汉式单例模式 "
subtitle:   " 设计模式之单例模式 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---



### 设计模式之饿汉式单例模式

懒汉式的单例比较简单，注重的是在类加载的时候就完成初始化，不用考虑类的同步问题，这个有个问题就是在类加载的时候就完成初始化，如果这个类在整个项目中都没有被使用也依旧会被初始化，这样就不能起到节省内存消耗的作用了。
这个实现起来非常简单，我们不做过多的讲解，我们看一下实现
```java
public class HungrySingleton {

    private final static HungrySingleton hungrySingleton;

    static{
        hungrySingleton = new HungrySingleton();
    }
    private HungrySingleton(){
    }
    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }
}
```
或者这么写，两种实现方式都是一样的。

```java
public class HungrySingleton {

    private final static HungrySingleton hungrySingleton= hungrySingleton = new HungrySingleton();
    private HungrySingleton(){
    }
    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }
}
```
最后我们提下，我们这里都没有对反射进行堵死，等候我们的文章我们会对堵死反射进行说明。