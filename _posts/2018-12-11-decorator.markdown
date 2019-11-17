---
layout:     post
title:      " 装饰者模式 "
subtitle:   " 设计模式之装饰者模式 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - Design Patterns
---


#### 装饰者模式
##### 定义
在不改变原有对象的基础之上，将功能附加到对象上。提供了比继承更有弹性的替代方案（扩展原有对象功能）
##### 类型
结构型
##### 适用场景
1. 扩展一个类的功能或者给一个类添加附加职责
2. 给一个对象动态的添加功能，或动态撤销功能。

##### 优点
1. 继承的有力补充，比继承灵活，不改变原有对象的情况下给一个对象扩展功能。（继承在扩展功能是静态的，必须在编译时就确定好，而使用装饰者可以在运行时决定，装饰者也建立在继承的基础之上的）
2. 通过使用不同装饰类以及这些类的排列组合，可以实现不同的效果。
3. 符合开闭原则

##### 缺点
1. 会出现更多的代码，更多的类，增加程序的复杂性。
2. 动态装饰时，多层装饰时会更复杂。（使用继承来拓展功能会增加类的数量，使用装饰者模式不会像继承那样增加那么多类的数量但是会增加对象的数量，当对象的数量增加到一定的级别时，无疑会大大增加我们代码调试的难度）

###### 装饰者相关的设计模式
  装饰者和代理模式
  ~ 装饰者模式关注的是对象的动态添加功能。代理模式关注的是对对象的控制访问，对它的用户隐藏对象的具体信息。
  
  装饰者模式和适配器模式
    ~ 装饰者模式和被装饰的类要实现同一个接口，或者装饰类是被装饰的类的子类。 适配器模式和被适配的类具有不同的接口。
    
    
按照我博客的国际惯例，下面开始看代码，看代码之前首先假设一个应用场景吧假设我们现在在路边摊看到一个卖煎饼果子的，现在想买煎饼果子，煎饼果子一般的话都是可以加鸡蛋加香肠什么的，好那我们就来模拟一下加在煎饼果子上加东西的操作。
首先我们看一下使用继承的方式怎么实现。

创建一个煎饼果子类
```java
public class Battercake {
    protected String getDesc(){
        return "煎饼果子";
    }
    protected int cost(){
        return 8;
    }

}
```
加鸡蛋类，我们让加鸡蛋类继承煎饼果子类

```java
public class BattercakeWithEgg extends Battercake {
    @Override
    public String getDesc() {
        return super.getDesc()+" 加一个鸡蛋";
    }

    @Override
    public int cost() {
        return super.cost()+1;
    }
}
```
加香肠类，同样的继承煎饼果子类。

```java
public class BattercakeWithEggSausage extends BattercakeWithEgg {
    @Override
    public String getDesc() {
        return super.getDesc()+ " 加一根香肠";
    }

    @Override
    public int cost() {
        return super.cost()+2;
    }
}

```
测试类

```java
public class DecoratorV1Test {
    public static void main(String[] args) {
        Battercake battercake = new Battercake();
        System.out.println(battercake.getDesc()+" 销售价格:"+battercake.cost());

        Battercake battercakeWithEgg = new BattercakeWithEgg();
        System.out.println(battercakeWithEgg.getDesc()+" 销售价格:"+battercakeWithEgg.cost());


        Battercake battercakeWithEggSausage = new BattercakeWithEggSausage();
        System.out.println(battercakeWithEggSausage.getDesc()+" 销售价格:"+battercakeWithEggSausage.cost());


    }
}

```
输出结果
```
煎饼 销售价格:8
煎饼 加一个鸡蛋 销售价格:9
煎饼 加一个鸡蛋 加一根香肠 销售价格:11
```
这样做有个问题，什么问题呢？假设我们现在要加2个鸡蛋呢？糟糕我们没写加2个鸡蛋的类，如果还有3个4个什么的那是不是就要类爆炸了。下面我们使用装饰者模式实现一下。

首先我们定义一个抽象的煎饼果子
```java
public abstract class ABattercake {
    protected abstract String getDesc();
    protected abstract int cost();

}
```
实体煎饼果子类，实体煎饼果子继承了抽象煎饼果子类。

```java
public class Battercake extends ABattercake {
    @Override
    protected String getDesc() {
        return "煎饼";
    }

    @Override
    protected int cost() {
        return 8;
    }
}
```
装饰父类，这里也是可以使用抽象类，等会儿我们再说什么时候使用抽象类什么时候使用实体类。注意构造器和这个里面的花费、描述方法的写法。这里注入一个抽象煎饼类的对象。我们的获取描述花费的操作都委托抽象煎饼类来执行，为什么要这么做可以去看看我之前的文章依赖倒置原则。
```java
public  class AbstractDecorator extends ABattercake {
    private ABattercake aBattercake;

    public AbstractDecorator(ABattercake aBattercake) {
        this.aBattercake = aBattercake;
    }
    @Override
    protected String getDesc() {
        return this.aBattercake.getDesc();
    }

    @Override
    protected int cost() {
        return this.aBattercake.cost();
    }
}

```
鸡蛋的装饰类，这里注意他的构造器，参数是父类的对象抽象煎饼类对象，这里获取描述和花费方法都是调用了父类的方法。
```java
public class EggDecorator extends AbstractDecorator {
    public EggDecorator(ABattercake aBattercake) {
        super(aBattercake);
    }

    @Override
    protected String getDesc() {
        return super.getDesc()+" 加一个鸡蛋";
    }

    @Override
    protected int cost() {
        return super.cost()+1;
    }
}
```
香肠装饰类
```java
public class SausageDecorator extends AbstractDecorator{
    public SausageDecorator(ABattercake aBattercake) {
        super(aBattercake);
    }

    @Override
    protected String getDesc() {
        return super.getDesc()+" 加一根香肠";
    }

    @Override
    protected int cost() {
        return super.cost()+2;
    }
}
```
最后是测试类，创建一个实体煎饼果子类并赋值给抽象煎饼果子类，然后将这个父类对象注入装饰类，再把得到的对象赋值给创建的抽象对象。
```java
public class DecoratorV2Test {
    public static void main(String[] args) {
        ABattercake aBattercake;
        aBattercake = new Battercake();
        aBattercake = new EggDecorator(aBattercake);
        aBattercake = new EggDecorator(aBattercake);
        aBattercake = new SausageDecorator(aBattercake);

        System.out.println(aBattercake.getDesc()+" 销售价格:"+aBattercake.cost());

    }
}
```
输入结果

```
煎饼 加一个鸡蛋 加一个鸡蛋 加一根香肠 销售价格:12
```
最后我们来说说装饰父类什么时候使用抽象类。一般当我们需要在具体的类中都需涛执行一些特定的操作时。我们一般就会使用抽象类，并定义抽象方法。

```java
public abstract class AbstractDecorator extends ABattercake {
    private ABattercake aBattercake;

    public AbstractDecorator(ABattercake aBattercake) {
        this.aBattercake = aBattercake;
    }
//定义每个抽象类的独特方法
    protected abstract void doSomething();

    @Override
    protected String getDesc() {
        return this.aBattercake.getDesc();
    }

    @Override
    protected int cost() {
        return this.aBattercake.cost();
    }
}

```

