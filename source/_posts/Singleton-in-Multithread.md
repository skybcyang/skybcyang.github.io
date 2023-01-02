---
title: 多线程下的单例模式
date: 2019-01-23 20:46:10
tags: 
- 设计模式
- 单例模式
---

单例模式作为最为常见的设计模式，一直被广泛使用。在重构一个项目做成多线程，识别到单例模式在多线程中具有一定的风险。
<!-- more -->
### 单例模式
在某些时候我们需要确保系统中有且仅有一个实例，这时候就要用到单例模式，单例模式的要点有三个；一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。

在多线程中，单例模式有如下风险
- 这个单例模式的类的所有普通成员也成为临界资源，需要保护
- 如果单例模式使用懒汉方式构建，可能会有生成多个实例的风险

下面就来介绍一下懒汉方式和饿汉方式

### 懒汉方式和饿汉方式
懒汉模式和饿汉模式不同点在于，饿汉模式在第一次调用单例类的时候，类已经构造好了，而饿汉模式第一次被访问的时候才去构造。下面用代码来展示
- 饿汉模式
```cpp
class Singleton {
    ...
    Singleton& getInst() {
        return *pInst;
    }
    ...
    Singleton* pInst;
}
Singleton* Singleton::pInst = new Singleton();
```

- 懒汉模式
```cpp
class Singleton {
    ...
    Singleton& getInst() {
        if(pInst == NULL) {
            pInst = new Singleton();
        }
        return *pInst;
    }
    ...
    Singleton* pInst;
}
Singleton* Singleton::pInst = NULL;
```
### 双重锁DCLP
双重锁DCLP，也就是`double-checked-locking pattern`,可以使用双重锁改善懒汉模式来确保只生成一个对象。
每次getInst的时候
1. 先判空
2. 再上锁
3. 再判空

代码如下
```cpp
class Singleton {
    ...
    Singleton& getInst() {
        if(pInst == NULL) {
            Lock lock;
            if(pInst == NULL) {
                pInst = new Singleton();
            }
        }
        return *pInst;
    }
    ...
    Singleton* pInst;
}
Singleton* Singleton::pInst = NULL;
```
但是DCLP只能保证生成一个对象，还是有风险的。

因为C++的new分为如下步骤，
1. 分配内存
2. 调用构造函数
3. 把地址复制给指针

但是步骤2或者步骤3有可能被编译器优化调换了顺序，先执行步骤3再执行步骤2。

基于以上，我们来探讨一下DCLP的风险，比如说：
- 第一次访问这个对象的时候，线程A判空后加了锁，然后走到了`pInst = new Singleton();`，执行了步骤3还没有执行步骤2，现在指针pInst已经被赋值，但是那块地址还没有被构造函数初始化。
- 这时候线程B来访问对象，在第一次判空的时候直接return了指针，然后拿着还没有调用构造函数的类执行下面的操作。

所以可以得知DCLP还是有一定风险的，还不如采用饿汉模式，一方面是规避了单例模式在多线程的风险，另一方面在取单例的时候少一次判空操作。

### 后续
理想是丰满的，现实是很骨感的。现在用的平台强制对内存进行管理，通过重载基类的operator new，系统初始化之后才能使用平台分配内存的api，不能使用饿汉模式在系统完成初始化之前就构造好单例类。无奈，最终采取了DCLP来保护单例模式，并且在线程串行初始化的时候构造单例类。


### 另外
C++有`volatile`关键字，能够确保语句顺序不被编译器优化，是否加了`volatile`能够让DCLP奏效呢，下篇文章探讨。

参考文章：
[C++和双重检查锁定模式(DCLP)的风险](http://blog.jobbole.com/86392/)