---
layout:     post
title:      "享元模式"
subtitle:   " \设计模式之享元模式\""
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---



#### 享元模式
##### 定义
1. 提供了减少对象数量二改善应用所需的对象结构的方法
2. 运用共享技术有效的支持大量粗粒度的对象。

> 用通俗的大白话来说就是减少对象的数量，提高对象的利用率，减少内存的使用，提高系统性能。 

##### 类型
创建型
##### 适用场景
1. 常常应用于系统底层的开发，一遍解决系统的性能问题。
2. 系统中有大量的相似对象、需要使用缓冲池的场景。
##### 优点
1. 减少对象的创建，降低内存中对象的数量，降低系统的内存，提高效率。
2. 减少内存之外的其他资源占用。（对象的创建是消耗其他的资源的）

##### 缺点
1. 关注内部/外部状态、关注线程安全问题。
2. 系统逻辑的复杂化。（比如我们经常做一道面试题Integer a=128；Integer b=128；a==b是true还是false，这里就有缓存的问题，后面我们再分析这个）

##### 内部状态
在享对象类的内部，不会随着环境改变而改变的状态。
##### 外部状态
记录在享元对象的外部，随着环境改变而改变。不可以共享的状态。

> 内部状态是享元对象的属性，不会随环境变化而变化的属性。外部状态我们打个比方就是在外部调用享元对象的方法时传过来一个int 这个int 为1时一种状态，为2时一种状态这就是外部状态。

我们来看看怎么实现。国际惯例假设一个应用场景，假设我们要做一个部门会议。

```java
public interface Employee {
    void report();
}
```
员工接口有个做报告方法。

```java
public class Manager implements Employee {
    @Override
    public void report() {
        System.out.println(reportContent);
    }
    private String title = "部门经理";
    private String department;
    private String reportContent;

    public void setReportContent(String reportContent) {
        this.reportContent = reportContent;
    }

    public Manager(String department) {
        this.department = department;
    }


}
```
经理实现了员工接口。有三个属性，标题，部门，报告内容。

```java
public class EmployeeFactory {
    private static final Map<String,Employee> EMPLOYEE_MAP = new HashMap<String,Employee>();

    public static Employee getManager(String department){
        Manager manager = (Manager) EMPLOYEE_MAP.get(department);

        if(manager == null){
            manager = new Manager(department);
            System.out.print("创建部门经理:"+department);
            String reportContent = department+"部门汇报:此次报告的主要内容是......";
            manager.setReportContent(reportContent);
            System.out.println(" 创建报告:"+reportContent);
            EMPLOYEE_MAP.put(department,manager);

        }
        return manager;
    }

}
```
员工工厂，这里使用了容器单例模式。

```java
public class FlyweightTest {
    private static final String departments[] = {"RD","QA","PM","BD"};

    public static void main(String[] args) {
        for(int i=0; i<10; i++){
            String department = departments[(int)(Math.random() * departments.length)];
            Manager manager = (Manager) EmployeeFactory.getManager(department);
            manager.report();
        }
    }
}
```
这里我们定义了一个部门列表，然后随机的使用其中一个部门去去执行报告操作。
```
创建部门经理:RD 创建报告:RD部门汇报:此次报告的主要内容是......
RD部门汇报:此次报告的主要内容是......
创建部门经理:PM 创建报告:PM部门汇报:此次报告的主要内容是......
PM部门汇报:此次报告的主要内容是......
RD部门汇报:此次报告的主要内容是......
创建部门经理:QA 创建报告:QA部门汇报:此次报告的主要内容是......
QA部门汇报:此次报告的主要内容是......
创建部门经理:BD 创建报告:BD部门汇报:此次报告的主要内容是......
BD部门汇报:此次报告的主要内容是......
RD部门汇报:此次报告的主要内容是......
RD部门汇报:此次报告的主要内容是......
PM部门汇报:此次报告的主要内容是......
QA部门汇报:此次报告的主要内容是......
BD部门汇报:此次报告的主要内容是......
```

