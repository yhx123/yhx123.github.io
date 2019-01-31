---
layout:     post
title:      " 外观模式 "
subtitle:   " 设计模式之外观模式 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---


#### 外观模式
##### 定义
又叫门面模式，提供了一个统一的接口，用来访问子系统中的一群接口。定义了一个高层接口，让子系统更容易使用。
##### 类型
结构型
##### 使用场景
1. 子系统越来越复杂，增加外观模式提供的简单调用接口
2. 构建多层系统结构，利用外观对象作为每层的入口，简化层间调用。
##### 优点
1. 简化了调用过程，无需了解深入子系统，防止来风险。
2. 减少系统的依赖、松散耦合
3. 更好的划分访问层次
4. 符合迪米特法则，即最少知道原则。

##### 缺点
1. 增加子系统、扩展子系统行为容易引入风险
2. 增加子系统的时候不符合开闭原则

##### 相关设计模式
 外观模式与中介模式
   ~ 外观模式强调的是外界对子系统的交互。中介者模式强调的是子系统之间的交互。
 
 外观模式和单例模式
   ~ 通常我们会把外观模式的外观对象做成单例对象
   
 外观模式和抽象工厂
   ~ 我们可以通过抽象工厂来获取子系统的实例，这样子系统可以通过外观实例对外观屏蔽。

下面开始看代码。在写代码之前我们先假设一个应用场景，假设我们有个积分商城，这个积分商城就包含以下几个子系统。比如积分校验，物流系统，积分支付子系统等子系统。

首先我们创建一个礼物实体类。

```java
public class PointsGift {
    private String name;

    public PointsGift(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
模拟一个积分校验系统，看看用户是否满足兑换资格。

```java
public class QualifyService {
    public boolean isAvailable(PointsGift pointsGift){
        System.out.println("校验"+pointsGift.getName()+" 积分资格通过,库存通过");
        return true;
    }
}

```
积分支付系统

```java
public class PointsPaymentService {
    public boolean pay(PointsGift pointsGift){
        //扣减积分
        System.out.println("支付"+pointsGift.getName()+" 积分成功");
        return true;
    }

}
```
物流系统

```java
public class ShippingService {
    public String shipGift(PointsGift pointsGift){
        //物流系统的对接逻辑
        System.out.println(pointsGift.getName()+"进入物流系统");
        String shippingOrderNo = "666";
        return shippingOrderNo;
    }
}
```
最后就是我们本节的核心了，外观类。

```java
public class GiftExchangeService {

    //这里使用硬编码，讲道理的话应该是依赖注入。把服务注入进来
    private QualifyService qualifyService = new QualifyService();
    private PointsPaymentService pointsPaymentService = new PointsPaymentService();
    private ShippingService shippingService = new ShippingService();

    public void giftExchange(PointsGift pointsGift){
        if(qualifyService.isAvailable(pointsGift)){
            //资格校验通过
            if(pointsPaymentService.pay(pointsGift)){
                //如果支付积分成功
                String shippingOrderNo = shippingService.shipGift(pointsGift);
                System.out.println("物流系统下单成功,订单号是:"+shippingOrderNo);
            }
        }
    }

}
```
然后我们来测试一下。

```java
public class FacadeTest {
    public static void main(String[] args) {
        PointsGift pointsGift = new PointsGift("T恤");
        GiftExchangeService giftExchangeService = new GiftExchangeService();
        giftExchangeService.giftExchange(pointsGift);
    }
}
```
我们这里的测试类只和外观类交互，然后通过外观类和各个子系统交互。外观模式很简单，就讲到这里。