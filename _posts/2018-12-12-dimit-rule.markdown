---
layout:     post
title:      " 迪米特法则 "
subtitle:   " 软件开发原则之迪米特法则 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - Design Patterns
---



#### 迪米特法则
##### 定义
> 一个对象应该对其他对象保持最少的了解。又叫最少知道原则。
尽量降低类之间的耦合。

主要强调降低耦合，优点也是降低耦合。

强调朋友交流，不和陌生人讲话。

所谓朋友：
出现在成员变量、方法的输入、输出参数中的类成为成员朋友类，而在方法体内部的类不属于朋友类。下面看代码

假设我们要实现一个需求校长下令给老师想知道班级有多少课。

新建一个校长类，有一个下命令方法。
```java
import java.util.ArrayList;
import java.util.List;

/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/12/4
 */
public class Principal {
    public void commandCheckNumber(Teacher teacher) {
        List<Course> courseList = new ArrayList();
        for (int i = 0; i < 20; i++) {
            courseList.add(new Course());
        }
        teacher.checkNumberOfCourses(courseList);
    }
}
```
新建一个老师类，数课程数量的方法。

```
import java.util.List;

/**
 * @author 杨红星
 * @version 1.0.0
 * @date 2018/12/4
 */
public class Teacher {
    public void checkNumberOfCourses(List<Course> courseList){
        System.out.println("总共课程数为"+courseList.size());
    }
}

```
再写一个测试类

```
public class DemeterTest {
    public static void main(String[] args) {
        Teacher teacher = new Teacher();
        Principal principal = new Principal();
        principal.commandCheckNumber(teacher);
    }
}
```
输出结果
> 总共课程数为20

