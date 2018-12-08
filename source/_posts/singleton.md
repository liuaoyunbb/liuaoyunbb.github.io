---
title: 深入理解单例模式
date: 2017-09-03 18:34:42
tags: 
    - 设计模式
---

&#160; &#160; &#160; &#160; 设计模式是在某种特定场合对某一特定问题给出的解决方案，所以我们学习设计模式，则可以把它应用到特定问题中，用于解决相似的问题。设计模式中的单例模式是最为基础的，也是面试中最经常考查的。
单例模式：确保一个类只有一个实例，并提供一个全局访问点。
&#160; &#160; &#160; &#160; 单例模式的写法大致可以分为四类，下面介绍它们的基本写法、改进及特点。
<!--more-->
### 基础懒汉式
先不初始化单例，等第一次使用的时候再初始化，即“懒加载”。
```Java
// 线程不安全
public class SingletonLazy1 {
    private static SingletonLazy1 instance = null;

    private SingletonLazy1(){}

    public static SingletonLazy1 getInstance(){
        if (instance == null){
            instance = new SingletonLazy1();
        }
        return instance;
    }
}
```
懒汉式的核心是懒加载，直到实例被第一次访问时才需要初始化实例，节约资源，启动速度快；缺点是多线程环境下线程不安全，if语句存在竞态条件。

### 懒汉式-改进1
最简单的处理方式是用synchronized关键字修饰getInstance()方法，这样能达到绝对的线程安全。

```Java
public class SingletonLazy2 {
    private static SingletonLazy2 instance = null;

    private SingletonLazy2(){}

    public static synchronized SingletonLazy2 getInstance(){
        if (instance == null){
            instance = new SingletonLazy2();
        }
        return instance;
    }
}
```
用synchronized关键字修饰getIntance()方法，用最简单的方式解决了线程安全的问题，但是并发性极差，单例只需要初始化一次，但就算初

### 懒汉式-DCL
双重检查锁(Double Check Lock,DCL)，DCL在改进1的外层又套了一层check，加上synchronized内层的check，即所谓双重检查锁。
```java
public class SingletonLazy3 {
    private static volatile SingletonLazy3 instance = null;

    private SingletonLazy3(){};

    public static SingletonLazy3 getInstance(){
        if (instance == null){      // 第一层判断
            synchronized (SingletonLazy3.class){
                if (instance == null){
                    instance = new SingletonLazy3();
                }
            }
        }
        return instance;
    }
}
```

### 饿汉式
相较于懒汉式，饿汉式在类加载时初始化实例，以后访问时返回即可。
```java
// 线程安全
public class SingletonHungry {

    private static SingletonHungry instance = new SingletonHungry();

    private SingletonHungry(){}

    public static SingletonHungry getInstance(){
        return instance;
    }
}
```
饿汉式写起来特别简单，而且线程安全，使用时没有延迟；缺点是有可能造成资源浪费(例如类加载后就一直未使用单例的话)。
需要注意的是，单线程环境下，饿汉式和懒汉式在新能上没有什么差别；但多线程环境下，由于懒汉式需要加锁，饿汉式性能反而更优。

### Holder模式
Holder模式：核心仍然是静态变量，足够方便和线程安全，通过静态Holder内部类持有真正的实例，间接实现了懒加载。
Holder模式是比较优秀的实现方式，建议考虑。
- 为什么静态内部类可以实现延迟加载？  
实际上是外部类被加载时内部类并不需要立即加载内部类，内部类不被加载则不需要进行类初始化，因此单例对象在外部类被加载了以后不占用内存。
```java 
public class SingletonInnerClass {

    private SingletonInnerClass() {
        if (SingletonHolder.instance != null){  //防止暴力反射
            throw new IllegalStateException();
        }
    }
    private static class SingletonHolder {
        private static SingletonInnerClass instance = new SingletonInnerClass();
    }
    public static SingletonInnerClass getIntance(){
        return SingletonHolder.instance;
    }
}
```
### 总结
对上面提及的单例模式进行总结：

| 实现方式   | 关键点              | 资源浪费 | 线程安全 | 是否优化多线程环境的性能 |
| ------ | ---------------- | ---- | ---- | ------------ |
| 基础懒汉   | 懒加载              | 否    | 否    | -            |
| 懒汉-改进1 | 懒加载、同步           | 否    | 是    | 否，串行操作       |
| 懒汉-DCL | 懒加载、DCL、volatile | 否    | 是    | 是            |
| 饿汉     | 静态变量初始化          | 是    | 是    | 是            |
| Holder | 静态变量初始化、Holder   | 否    | 是    |              |











