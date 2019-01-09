#### 反射对单例模式的破坏

首先我们依旧是使用饿汉式作为测试。我们把之前写的饿汉式的代码贴上来。
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
然后我们在测试类中使用反射来对这个单例进行攻击。

```java
public class SingletonTest {
    public static void main(String[] args) throws  ClassNotFoundException, NoSuchMethodException, IllegalAccessException {
        HungrySingleton instance = HungrySingleton.getInstance();
        Class<HungrySingleton> hungrySingletonClass = HungrySingleton.class;
        Constructor<HungrySingleton> constructor = hungrySingletonClass.getConstructor();
        constructor.setAccessible(true);
        HungrySingleton newInstance = constructor.newInstance();
        System.out.println(instance == newInstance);
    }
}

```
这个输出结果可想而知false。那么我们怎么样防治这种反射攻击呢？下面我们给出一种解决方案

```java
public class HungrySingleton implements Serializable,Cloneable{
    private final static HungrySingleton hungrySingleton;

    static{
        hungrySingleton = new HungrySingleton();
    }
    private HungrySingleton(){
        if(hungrySingleton != null){
            throw new RuntimeException("单例构造器禁止反射调用");
        }
    }
    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }
}
```
我们再使用这个测试类进行测试就发现报出异常。那这是饿汉式的单例如果是懒汉式的单例呢？能否通过这种方式来实现？答案是不能。至于原因的大家想想就知道了。懒汉式一开始加载的时候成员变量是null，也就无法通过判断是否为null来阻止反射获取实例。

