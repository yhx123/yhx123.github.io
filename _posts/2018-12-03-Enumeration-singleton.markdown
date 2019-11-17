---
layout:     post
title:      " 枚举实现单例模式 "
subtitle:   " 设计模式之单例模式 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - Design Patterns
---



#### 枚举实现单例模式
前面我们说到序列化和反序列化以及反射对单例都是有破坏的，下面我们介绍一种更加优雅的实现，也是effective java中推荐的实现方式，枚举实现单例模式。话不多说我们直接看代码吧。

```java
public enum EnumInstance {
    INSTANCE{
        protected  void printTest(){
            System.out.println("Honson Print SingletonTest");
        }
    };
    protected abstract void printTest();
    private Object data;

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
    public static EnumInstance getInstance(){
        return INSTANCE;
    }

}
```
然后我们看看测试类

```java
public class SingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    
    EnumInstance instance = EnumInstance.getInstance();
        instance.setData(new Object());

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("singleton_file"));
        oos.writeObject(instance);

        File file = new File("singleton_file");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        EnumInstance newInstance = (EnumInstance) ois.readObject();
        
        
        
        System.out.println(instance.getData());
        System.out.println(newInstance.getData());
        System.out.println(instance.getData() == newInstance.getData());
    }
```
这里返回的结果是true，然后我们测试一下反射去获取这个对象。

```java
public class SingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        Class objectClass = EnumInstance.class;
        Constructor constructor = objectClass.getDeclaredConstructor(String.class,int.class);
        constructor.setAccessible(true);
        EnumInstance instance = (EnumInstance) constructor.newInstance("Honson",666);
}
```
运行结果报异常
> Exception in thread "main" java.lang.IllegalArgumentException: Cannot reflectively create enum objects
	at java.lang.reflect.Constructor.newInstance(Constructor.java:417)
	
我们点击进去查看一下417行的代码
```java
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor; 
```
从这里我们可以看到jdk底层就为我们对反射进行处理了。