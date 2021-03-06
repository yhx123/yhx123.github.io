---
layout:     post
title:      " 开闭原则 "
subtitle:   " 软件开发原则之开闭原则 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - Design Patterns
---


#### 开闭原则

##### 定义
所谓开闭原则就是一个软件实体如类、模块和函数应该对扩展开放、对修改关闭。
强调用抽象构建框架，实现实现拓展细节。

有优点是提高软件的复用性和易维护展性。是面向对象的最基本原则。


#### 依赖倒置原则
##### 定义
高层模块不应该依赖底层模块，二者都应该依赖其抽象。
抽象不应该依赖细节：细节应该依赖抽象。
针对接口编程，不要针对实现编程。

##### 优点
降低耦合提高稳定性，提高代码的可读性和易维护性。减少代码在修改时可能造成的风险。

下面我们用代码来说明啥是依赖倒置原则


假设我们想实现一个人在学习的需求，我们可以这样写
```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class Honson {
    public void studyJava() {
        System.out.println("Honson 在学习java");
    }

    public void studyFE() {
        System.out.println("Honson 在学习前端");
    }
}
```
然后有个测试类

```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class HonsonTest {
    public static void main(String[] args) {
        Honson Honson = new Honson();
        Honson.studyFE();
        Honson.studyJava();
    }
}
```
> 这时假设我们还想要学习Python那么我们则需要去修改Honson这个类。这样写法是面向实现编程，因为整个Honson类就是一个实现类。这个Honson类是要经常被修改的。也就拓展性比较差。我们这个HonsonTest类就是应用层（高层模块）的类是依赖于这个Honson实现类（底层模块）的。因为我们没有抽象。根据依赖倒置原则高层次的模块不应该去依赖低层次的模块。每次Honson拓展都要来HonsonTest进行补充。

我们下面开始码具有依赖倒置的代码，首先创建一个接口。这个接口有个学习方法。
```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
 
public interface ICourse {
    void studyCourse();
}

```
下面写几个实现类，实现这个接口
```java

public class JavaCourse implements ICourse {

    @Override
    public void studyCourse() {
        System.out.println("Honson在学习Java课程");
    }
}

```

```java
public class FECourse implements ICourse {
    @Override
    public void studyCourse() {
        System.out.println("Honson在学习FE课程");
    }

}
```
此时我们将Honson类修改为

```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class Honson {
    public void studyJava(ICourse iCourse) {
        iCourse.studyCourse();
    }
}

```
Test类修改为

```
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class HonsonTest {
    public static void main(String[] args) {
        Honson Honson = new Honson();
        Honson.studyJava(new JavaCourse());
        Honson.studyJava(new FECourse());
    }
}
```
输出结果为

```
Honson在学习Java课程
Honson在学习FE课程
```
这时如果我们还有其他大的课程想要学习我们可以通过添加ICourse的实现类的方式来添加。
Honson这个类是不会去动他的。对于想要学习什么课程我们有HonsonTest类这高层类来自己选择。
顺便提一下我们在HonsonTest类中使用接口方法的方式对ICourse接口的依赖进行注入。我们也可以使用构造器的方式对依赖进行注入。

```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class Honson {
    private ICourse iCourse;
    public Honson(ICourse iCourse) {
        this.iCourse = iCourse;
    }

    public void studyJava() {
        iCourse.studyCourse();
    }
}

```
此时我们Honson中就应该这么写

```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class HonsonTest {
    public static void main(String[] args) {
        JavaCourse javaCourse = new JavaCourse();
        FECourse feCourse = new FECourse();
        Honson Honson = new Honson(javaCourse);
        Honson.studyJava();
        Honson = new Honson(feCourse);
        Honson.studyJava();
    }
}

```
有构造器的方式同样我们也可以用set 的方式，此时我们的Honson就要这么写了

```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class Honson {
    private ICourse iCourse;

    public void setiCourse(ICourse iCourse) {
        this.iCourse = iCourse;
    }

    public void studyJava() {
        iCourse.studyCourse();
    }
}

```
对应的HonsonTest就要这么写了


```java
/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/11/30
 */
public class HonsonTest {
    public static void main(String[] args) {
        JavaCourse javaCourse = new JavaCourse();
        FECourse feCourse = new FECourse();
        Honson Honson = new Honson();
        Honson.setiCourse(feCourse);
        Honson.studyJava();
        Honson.setiCourse(javaCourse);
        Honson.studyJava();
    }
}
```

好到这里我们的依赖倒置原则就讲完了，总结一下总体原则就是面向接口编程，或者说面向抽象编程。上面例子中的接口我们也可以使用抽象类来代替。同学们可以使用抽象类模拟一下上面的过程。

