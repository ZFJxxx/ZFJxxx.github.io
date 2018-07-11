---
layout: post
title:  接口和抽象类
date:   2016-11-05 19:15:10
categories:  Java
tags:  Java基础
keywords: String
description: 
---

## 抽象类
* 1.抽象类必须用abstract修饰符来修饰，抽象方法也必须用abstract修饰符来修饰，抽象方法没有方法体。 
* 2.含有抽象方法的类只能被定义成抽象类。 
* 3.抽象类可以包含方法（普通方法和抽象方法都可以），成员变量，构造器，初始化块，内部类。 
* 4.抽象类不能被实例化。 
* 5.abstract关键字修饰的方法必须被其子类重写才有意义，否则这个方法永远没有方法体。所以abstract方法不能定义为private访问权限。（static也不能和abstract同时修饰某个方法，但是可以同时修饰某个内部类。）

```
public abstract class xxx{
    //抽象方法
    public abstract string aaa();
    public abstract string bbb();
    //普通方法
    public void ccc(){};
}
```
## 接口
抽象类是从多个类中抽象出来的模版，如果将这种抽象进行得更测底。则可以提炼出一种更加特殊的“抽象类”- 接口 
* 1.接口中的成员都有固定修饰符。 

常量：public static final 
普通方法：public abstract 
但是在接口中这些修饰符可以省略，在接口中下面两行代码效果完全相同
```
int a= 5;
public static final int a =5;
void out();
public abstract void out();
```
* 2.接口中只能有静态常量，抽象方法、类方法和默认方法。不能有构造器，初始化定义块。
```
interface xxx{
    public static final int a =5;
    //1.接口中的普通方法和类方法必须用public abstract修饰
    public abstract void aaa();
    //2.类方法必须是public static修饰，必须有方法体
    static void bbb(){};
    //3.默认方法必须用default修饰，必须有方法体
    default void ccc(){};
}
```
* 3.实现接口方法时，必须使用public访问控制修饰符，因为接口中的方法都是public的，子类在重写父类方法时访问权限只能更大或者相等。
* 4.接口之间可以实现多继承，因为都么有方法体，所以随便继承
```
interface A{
    void methodA();
}

interface B{
    void methodB();
}

interface C extends B,A{
    void methodC();
}

class D implements C{
    puublic void methodA(){}
    puublic void methodB(){}
    puublic void methodC(){}
}
```
## 接口与抽象类的区别
抽象类和接口很像 
* 1.接口和抽象类都不能被实例化，他们都用于被其他类继承或者实现。 
* 2.接口和抽象类都可以包含抽象方法，继承抽象类和实现接口的类都必须全部实现这些抽象方法。


抽象类和接口的区别 
* 1.设计抽象类是自下而上的过程,我子类需要,所以我定义抽象类； 设计接口是自上而下的过程,我接口规范某一行为,我某类需要这个行为,我实现某接口；
* 2.接口里不包含构造器，抽象类中可以有构造器。 
* 3.接口里不包含初始化块，抽象类中可以有初始化块。 
* 4.接口中只能定义静态常量（public static final），不能定义普通变量。而抽象类中都可以定义。 
* 5.接口能多实现，抽象类只能单继承。
