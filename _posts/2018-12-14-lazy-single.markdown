### 设计模式之单例模式

#### 定义
> 保证一个类仅有一个实例，并提供一个全局访问点。
#### 类型
> 创建型
#### 适用场景
> 想确保任何情况下都绝对只有一个实例。
#### 优点

* 在内存中只有一个实例，减少内存开销。特别是一个对象在使用时需要频繁创建和销毁同时创建和销毁性能无法优化时。
* 可以避免对资源的多重占用。比如我在对一个文件进行写操作，使用单例可以避免同时对这个文件进行写操作。
* 设置全局访问点，严格控制访问。
#### 缺点
> 没有接口，拓展困难。如果想要拓展就必须要修改代码。

下面开始看代码。我们首先实现以下懒汉式。懒汉式的单例模式注重的是延时加载，只有在引用的时候才会加载。


```java
public class LazySingleton {
    private static LazySingleton lazySingleton = null;
 
    public  static LazySingleton getInstance(){
        if(lazySingleton == null){
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }
}    
```
这个方式写的在单线程下是没有什么问题的。
但是在多线程的环境就会出现问题，假设其中一个线程到  lazySingleton = new LazySingleton();这一行时还没有return出去。
此时又有一个线程进入这个方法。
因为此时还没有return出去。
所以在进行判断时lazySingleton依旧为null。
我们说 单例模式的优点就是：
> 在内存中只有一个实例，减少内存开销。特别是一个对象在使用时需要频繁创建和销毁同时创建和销毁性能无法优化时。

本来我们希望这个对象之创建一次。这样的话这个对象2次进入都创建了对象，这样就不能达到减小内存开销的目的。
小伙伴们可以用这段代码debug走一下看看（使用idea在多线程环境下debug记得要切换一下，不会的小伙伴可以百度一下）。

我们看一下这个代码该怎么优化。那位很简单嘛，不就是多线程的问题吗，加个锁不就行了。好我们加上锁了。


```java
public class LazySingleton {
    private static LazySingleton lazySingleton = null;
 
    public synchronized static LazySingleton getInstance(){
        if(lazySingleton == null){
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }
}  

```
现在是我们在方法上添加了synchronized关键字。这个方法是个静态方法也是说这个所相当于加载类上面。同样的我们还可以这样写，效果是一样的。

```java

public class LazySingleton {
    private static LazySingleton lazySingleton = null;
 
    public  static LazySingleton getInstance(){
    synchronized(LazySingleton.class){
         if(lazySingleton == null){
            lazySingleton = new LazySingleton();
        }
    }
        return lazySingleton;
    }
} 

```
这做确实解决了多线程的问题，但是我们都知道，synchronized是比较耗费资源的加锁方式，而且在使用static方法时锁的是这个class。锁的范围也是非常的大，
性能消耗也是非常明显。下面我们看看有没有在性能和安全性上能够取得平衡的方案。

我们可以使用doubleCheck双重检查的方式来上实现懒汉式单例。这种方式是兼顾性能和安全的一种实现方式。

```java
public class LazyDoubleCheckSingleton {
    private volatile static LazyDoubleCheckSingleton lazyDoubleCheckSingleton = null;
    private LazyDoubleCheckSingleton(){

    }
    public static LazyDoubleCheckSingleton getInstance(){
        if(lazyDoubleCheckSingleton == null){
            synchronized (LazyDoubleCheckSingleton.class){
                if(lazyDoubleCheckSingleton == null){
                    lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();
                }
            }
        }
        return lazyDoubleCheckSingleton;
    }
}

```
双重锁现在不再是对方法进行加锁，我们现在对代码块进行加锁。此时我们是锁定了这个类，这个if方法还是可以进来，这种方式大大的缩短了synchronized锁定的范围，从而减少性能消耗。
至于这里为什么要加两个if判断，同学们对比上面的第二种在类上面加锁的代码，就很明显可以看出了。
然后这种实现方式还有个坑。我们来解决一下。
我们来看这个代码， lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();看起来是一行代码，其实这里经历了三个过程：
1. 分配内存给这个对象
2. 初始化对象
3. 设置lazyDoubleCheckSingleton 指向刚分配的内存地址

这里的坑就是2和3操作可能会被重排序，2和3的操作顺序可能会被颠倒。此时执行流程为：
1. 分配内存给这个对象

.
3. 设置lazyDoubleCheckSingleton 指向刚分配的内存地址

.
2. 初始化对象

此时在执行到3操作是，其实我们的实例还没有完成实例化的操作，但是在进行空判断的时候，因为已经被分配了内存空间，判断时并不为空，然后直接return一个空对象。如果我们拿到这个对象进行操作就会报空指针异常了。那为什么会进行重排序呢，在单线程中java语言规范中允许2和3进行重排序，因为重排序可以提高程序的执行效率。

为了方便大家理解，大家可以看下面的图，这是一个单线程的执行过程。
![](https://user-gold-cdn.xitu.io/2018/12/18/167c1eb88634771f?w=914&h=704&f=png&s=153491)

对于在多线程中的执行就变成下面的方式了。

![](https://user-gold-cdn.xitu.io/2018/12/18/167c1eab52f4d6d7?w=1506&h=736&f=png&s=232347)

如图，对于线程0步骤2和步骤3进行重排序并不会影响最后的结果。但是对于线程1就不一样了。当线程0执行步骤3之后线程1进入对象是否为空的判断，此时对象并没有完成初始化，但是对象已经被设置了内存空间，所以判断是否为空时不为空，这时候线程1就会执行return操作也就是步骤4，然而拿到的是一个没有初始化的完成的对象。

那么我们该如何解决这个问题呢？
我可以不允许线程0进行2和3的重排序，或者不让线程1看到2和3的重排序。

我们使用volatile这个关键字限制线程0进行重排序。完整的代码如下:
```java
public class LazyDoubleCheckSingleton {
    private volatile static LazyDoubleCheckSingleton lazyDoubleCheckSingleton = null;
    private LazyDoubleCheckSingleton(){

    }
    public static LazyDoubleCheckSingleton getInstance(){
        if(lazyDoubleCheckSingleton == null){
            synchronized (LazyDoubleCheckSingleton.class){
                if(lazyDoubleCheckSingleton == null){
                    lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();

                }
            }
        }
        return lazyDoubleCheckSingleton;
    }
}
```
下面我们来实现一下不让线程看到操作2和操作3的重排序的方案。
使用静态内部类的方式。

```java
public class StaticInnerClassSingleton {
    private static class InnerClass{
        private static StaticInnerClassSingleton staticInnerClassSingleton = new StaticInnerClassSingleton();
    }
    public static StaticInnerClassSingleton getInstance(){
        return InnerClass.staticInnerClassSingleton;
    }
     private StaticInnerClassSingleton(){
    }
}
```
我们来说一下为什么这样做可以让操作2和操作3对线程1不可见。

类在初始化的阶段：也就是类在加载并且在线程使用之前，jvm在类的初始化阶段会获取一个类初始化的一个锁，这个锁会同步多个线程对一个类的初始化。示意图如下，当线程0在执行时，对于线程1是不会看到的。

![](https://user-gold-cdn.xitu.io/2018/12/19/167c6a48ec175894?w=1716&h=636&f=png&s=190006)
上面的方式我们称之为基于类初始化的延迟加载的单例模式。 根据java语言规范主要以下几种种情况，发生这个类将被立即初始化。
1. 有个实例被创建
2. 类中的静态方法被调用
3. 类中的静态成员被赋值
4. 类中的静态成员被使用

懒汉式的单例模式我们就降到这里