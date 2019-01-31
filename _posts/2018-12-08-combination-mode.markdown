---
layout:     post
title:      " 组合模式 "
subtitle:   " 设计模式之组合模式 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---



#### 组合模式
##### 定义
1. 将对象组合成树形结构以表示“部分-整体”的层次结构。
2. 组合模式是客户端对单个对象和组合对象保持一致的方式处理。
##### 类型
> 结构型
##### 适用场景
1. 希望客户端可以忽略组合对象与单个对象的差异是
2. 处理一个树形结构时

##### 优点
1. 清楚地定义分层次的复杂对象，表示对象的全部或部分层次。
2. 让客户端忽略了层次的差异，方便对整个层次结构进行控制。
3. 简化客户端代码

下面开始写代码，我们首先假设一个业务场景，假设我们有一个课程，这个课程里面有很多课程包括java课程python等等，我们定义一个抽象类，让这些课程类继承这个抽象类，那么我们就可以认为这些课程是一个对象。

```java
public abstract class CatalogComponent {
    public void add(CatalogComponent catalogComponent){
        throw new UnsupportedOperationException("不支持添加操作");
    }

    public void remove(CatalogComponent catalogComponent){
        throw new UnsupportedOperationException("不支持删除操作");
    }


    public String getName(CatalogComponent catalogComponent){
        throw new UnsupportedOperationException("不支持获取名称操作");
    }


    public double getPrice(CatalogComponent catalogComponent){
        throw new UnsupportedOperationException("不支持获取价格操作");
    }


    public void print(){
        throw new UnsupportedOperationException("不支持打印操作");
    }


}
```
这个就是抽象方法，至于我们为什么要在这个抽象类中抛出异常呢？这个问题我们等写完了其他的类再解答。

```java
public class Course extends CatalogComponent {
    private String name;
    private double price;

    public Course(String name, double price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public String getName(CatalogComponent catalogComponent) {
        return this.name;
    }

    @Override
    public double getPrice(CatalogComponent catalogComponent) {
        return this.price;
    }

    @Override
    public void print() {
        System.out.println("Course Name:"+name+" Price:"+price);
    }

}
```
这是一个课程类，我们继承了抽象类，有价格和名称两个属性。


```java
public class CourseCatalog extends CatalogComponent {
    private List<CatalogComponent> items = new ArrayList<CatalogComponent>();
    private String name;
    private Integer level;


    public CourseCatalog(String name,Integer level) {
        this.name = name;
        this.level = level;
    }

    @Override
    public void add(CatalogComponent catalogComponent) {
        items.add(catalogComponent);
    }

    @Override
    public String getName(CatalogComponent catalogComponent) {
        return this.name;
    }

    @Override
    public void remove(CatalogComponent catalogComponent) {
        items.remove(catalogComponent);
    }

    @Override
    public void print() {
        System.out.println(this.name);
        for(CatalogComponent catalogComponent : items){
            if(this.level != null){
                for(int  i = 0; i < this.level; i++){
                    System.out.print("  ");
                }
            }
            catalogComponent.print();
        }
    }

}

```
这个是目录类，有两个属性，层级和名称。这个是我们看到代码，就解释一下为什么在抽象类中要抛出异常，因为我们的子类有的功能我们都重写了抽象类的方法，但是如果没有重写的方法也是被继承了的。如果这时候被调用了，子类是没有这个功能我希望不被调用所以就直接抛异常。最后我们看看测试类的代码。

```java
public class CompositeTest {
    public static void main(String[] args) {
        CatalogComponent linuxCourse = new Course("Linux课程",11);
        CatalogComponent windowsCourse = new Course("Windows课程",11);

        CatalogComponent javaCourseCatalog = new CourseCatalog("Java课程目录",2);

        CatalogComponent mmallCourse1 = new Course("Java设计模式一",55);
        CatalogComponent mmallCourse2 = new Course("Java设计模式二",66);
        CatalogComponent designPattern = new Course("Java设计模式三",77);

        javaCourseCatalog.add(mmallCourse1);
        javaCourseCatalog.add(mmallCourse2);
        javaCourseCatalog.add(designPattern);

        CatalogComponent imoocMainCourseCatalog = new CourseCatalog("课程主目录",1);
        imoocMainCourseCatalog.add(linuxCourse);
        imoocMainCourseCatalog.add(windowsCourse);
        imoocMainCourseCatalog.add(javaCourseCatalog);

        imoocMainCourseCatalog.print();
    }
}

```
组合模式最大的问题在于要花代码去判断到底是那一个类，因为我们在使用的时候都是使用了父类对象。