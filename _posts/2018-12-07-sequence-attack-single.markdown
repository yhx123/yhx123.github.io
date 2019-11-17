---
layout:     post
title:      " 序列化和反序列化对单例破坏 "
subtitle:   " 设计模式之原型模式 "
date:       2019-01-9 12:00:00
author:     "Honson"
header-img: ""
tags:
    - Design Patterns
---


首先我们来看一下序列化和反序列化是怎么破坏单例的。看代码

```java
public class HungrySingleton implements Serializable{

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
这里我们使用之前的饿汉式的单例作为例子。在之前饿汉式的代码上做点小改动。就是让我们的单例类实现 Serializable接口。然后我们在测试类中测试一下怎么破坏。
```java
public class SingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        HungrySingleton instance = HungrySingleton.getInstance();

        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("singleton_file"));
        oos.writeObject(instance);

        File file = new File("singleton_file");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        HungrySingleton newInstance = (HungrySingleton) ois.readObject();

        System.out.println(instance == newInstance;
        
    }
    
}
```
这里首先我们使用正常的方式来获取一个对象。通过序列化将对象写入文件中，然后我们通过反序列化的到一个对象，我们再对比这个对象，输出的内存地址和布尔结果都表示这不是同一个对象。也就说我们通过使用序列化和反序列化破坏了这个单例，那我们该如何防治呢？防治起来很简单，只需要在单例类中添加一个readResolve方法，下面看代码：

```java
public class HungrySingleton implements Serializable,Cloneable{

    private final static HungrySingleton hungrySingleton;

    static{
        hungrySingleton = new HungrySingleton();
    }
    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }

    private Object readResolve(){
        return hungrySingleton;
    }
}
```
此时我们再通过测试类进行测试即可发现我们通过序列化和反序列化得到的还是同一个对象。那么为什么添加一个这个方法就可以防止呢？下我们跟进去看看为什么

首先这个readResolve方法不是object里面的方法。我们进我们的测试类中去看看这行中的HungrySingleton newInstance = (HungrySingleton) ois.readObject()中的 readObject()的实现。我们只把关键代码贴出来。

```java
    public final Object readObject()
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            Object obj = readObject0(false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
            }
            return obj;
```
我们重点来看一下 Object obj = readObject0(false)这一行这里调用了一个readObject0方法，我们再深入看一下这个readObject0方法的实现。


```java
    /**
     * Underlying readObject implementation.
     */
    private Object readObject0(boolean unshared) throws IOException {
  
    ....  
    
    //各种判断逻辑我们暂时不管
    
    switch (tc) {
                 switch (tc) {
                case TC_NULL:
                    return readNull();

                case TC_REFERENCE:
                    return readHandle(unshared);

                case TC_CLASS:
                    return readClass(unshared);

                case TC_CLASSDESC:
                case TC_PROXYCLASSDESC:
                    return readClassDesc(unshared);

                case TC_STRING:
                case TC_LONGSTRING:
                    return checkResolve(readString(unshared));

                case TC_ARRAY:
                    return checkResolve(readArray(unshared));

                case TC_ENUM:
                    return checkResolve(readEnum(unshared));

                case TC_OBJECT:
                    return checkResolve(readOrdinaryObject(unshared));

                case TC_EXCEPTION:
                    IOException ex = readFatalException();
                    throw new WriteAbortedException("writing aborted", ex);

                case TC_BLOCKDATA:
                case TC_BLOCKDATALONG:
                 }
            ....     
    
    }
```
我们看这个 case TC_OBJECT: 也就是判断为object之后的代码，checkResolve(readOrdinaryObject(unshared))这行先是调用了readOrdinaryObject()方法，然后将方法的返回值返回给checkResolve方法，我们先查看一下readOrdinaryObject()方法。

```java
     /**
     * Reads and returns "ordinary" (i.e., not a String, Class,
     * ObjectStreamClass, array, or enum constant) object, or null if object's
     * class is unresolvable (in which case a ClassNotFoundException will be
     * associated with object's handle).  Sets passHandle to object's assigned
     * handle.
     */
    private Object readOrdinaryObject(boolean unshared){
        
        .....
        //各种判断校验
        
         Object obj;
        try {
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }

        passHandle = handles.assign(unshared ? unsharedMarker : obj);
        ClassNotFoundException resolveEx = desc.getResolveException();
        if (resolveEx != null) {
            handles.markException(passHandle, resolveEx);
        }
        
        .....
        
        
            return obj;
    }

```

我们看一下 obj = desc.isInstantiable() ? desc.newInstance() : null这一行中的obj对象是干嘛用的 我们往下翻在这个方法的最后将这个obj返回出去了。我们又回头看这个这一行obj = desc.isInstantiable() ? desc.newInstance() : null 这个进行判断如果 obj==desc.isInstantiable()就返回一个新的对象，否则返回空，代码看到这里好像有点眉目，我再看看isInstantiable这个方法的实现。


```java
 /**
     * Returns true if represented class is serializable/externalizable and can
     * be instantiated by the serialization runtime--i.e., if it is
     * externalizable and defines a public no-arg constructor, or if it is
     * non-externalizable and its first non-serializable superclass defines an
     * accessible no-arg constructor.  Otherwise, returns false.
     */
    boolean isInstantiable() {
        requireInitialized();
        return (cons != null);
    }

```
isInstantiable方法实现很简单，这里的cons是什么呢？我们继续看

```java
 /** serialization-appropriate constructor, or null if none */
    private Constructor<?> cons;
```
cons是构造器这里是通过反射获取的对象，光看着一行代码我们好像并不能看出啥东西，这时候我们看一下这一行代码的注释。 翻译过来的话就是：
> 如果表示的类是serializable/externalizable并且可以由序列化运行时实例化，则返回true  - 如果它是可外部化的并且定义了公共的无参数构造函数，或者它是不可外化的，并且它的第一个非可序列化的超类定义了可访问的无参数构造函数。否则，返回false。

externalizable这个类是serializable的一个子类用于制定序列化，比如自定义某个属性的序列化，用的比较少。
好，我们的单例实现了serializable接口所以这里返回的是true，那么回到我们之前看看到的那里，也就是这里obj = desc.isInstantiable() ? desc.newInstance() : null 此时返回的就是一个newInstance是通过反射拿到的对象，既然是反射拿到的对象自然是一个新的对象，看到这里我们算弄明白了为什么序列化获取的是一个新的对象。不过到这里还是没有得到我们想要的知道的为什么写了一个readResolve方法就可以解决反序列化得到的不是同一个对象的问题，那么我们继续往下看ObjectInputSteam这个类

```java 
 if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
```
看到这里，这里对obj进行了一次空判断，这里我们刚分析了obj不会为空，看这里desc.hasReadResolveMethod()从命名我们可以看出这个判断是判断否包含readResolve这个方法。我们再点进去看看这个的实现

```java
    /**
     * Returns true if represented class is serializable or externalizable and
     * defines a conformant readResolve method.  Otherwise, returns false.
     */
    boolean hasReadResolveMethod() {
        requireInitialized();
        return (readResolveMethod != null);
    }
```
这里依旧是看代码没啥看的，我们看看注释，符合我们的猜测，也就是说这个
```java
if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
```
判断结果为true那么我们再看看这个desc.invokeReadResolve(obj)的实现

```java
/**
     * Invokes the readResolve method of the represented serializable class and
     * returns the result.  Throws UnsupportedOperationException if this class
     * descriptor is not associated with a class, or if the class is
     * non-serializable or does not define readResolve.
     */
    Object invokeReadResolve(Object obj)
        throws IOException, UnsupportedOperationException
    {
        requireInitialized();
        if (readResolveMethod != null) {
            try {
                return readResolveMethod.invoke(obj, (Object[]) null);
            } catch (InvocationTargetException ex) {
                Throwable th = ex.getTargetException();
                if (th instanceof ObjectStreamException) {
                    throw (ObjectStreamException) th;
                } else {
                    throwMiscException(th);
                    throw new InternalError(th);  // never reached
                }
            } catch (IllegalAccessException ex) {
                // should not occur, as access checks have been suppressed
                throw new InternalError(ex);
            }
        } else {
            throw new UnsupportedOperationException();
        }
    }
```
这里我们看方法名的也能猜测这是使用了反射来调用，看这一行 return readResolveMethod.invoke(obj, (Object[]) null) 使用了反射来调用readResolveMethod方法。可是你可能会问了 也没看到用readResolveMethod这个方法啊，我对这个类进行搜索一下 readResolve

```java
/**
     * Creates local class descriptor representing given class.
     */
    private ObjectStreamClass(final Class<?> cl) {
    
    .....
    
    
    
   domains = getProtectionDomains(cons, cl);
        writeReplaceMethod = getInheritableMethod(
            cl, "writeReplace", null, Object.class);
        readResolveMethod = getInheritableMethod(
            cl, "readResolve", null, Object.class);
        return null;
    
    ....    
```

在这里可以看到是获取了readResolve这个方法。这样就算解决了我们最初的疑问了。同学们可以根据我说的源码在相应的地方打断点看看。