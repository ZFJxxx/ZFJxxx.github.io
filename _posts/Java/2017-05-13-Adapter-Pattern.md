---
layout: post
title:  适配器模式 和  装饰器模式
date:   2017-05-13 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---
## 适配器模式
适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。

主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

JAVA JDK 1.1 提供了 Enumeration 接口，而在 1.2 中提供了 Iterator 接口，想要使用 1.2 的 JDK，则要将以前系统的 Enumeration 接口转化为 Iterator 接口，这时就需要适配器模式。
```
class Apple {
    public void getAppleColor(String str) {
        System.out.println("Apple color is: " + str);
    }
}

class Orange {
    public void getOrangeColor(String str) {
        System.out.println("Orange color is: " + str);
    }
}
```
```
class AppleAdapter extends Apple {
    private Orange orange;

    public AppleAdapter(Orange orange) {
        this.orange = orange;
    }

    @Override
    public void getAppleColor(String str) {
        orange.getOrangeColor(str);
    }
}
```
```
public class TestAdapter {
    public static void main(String[] args) {
        Apple apple = new Apple();
        apple.getAppleColor("green");

        Orange orange = new Orange();
        AppleAdapter aa = new AppleAdapter(orange);
        aa.getAppleColor("red");          //橘子也可以调用getAppleColor方法了,底层实际依然是getOrangeColor方法
    }
}
```
输出结果
```
Apple color is: green
Orange color is: red
```

## 装饰器模式
装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。
```
class Apple {
    public void getAppleColor(String str) {
        System.out.println("Apple color is: " + str);
    }
}
```
```
class AppleDecorator extends Apple {
    private Apple apple;

    public AppleAdapter(Apple apple) {
        this.apple = apple;
    }

    @Override
    public void getAppleColor(String str) {
        apple.getAppleColor(str);
        changeColor();
    }
    
    private void changeColor(){
        System.out.println("apple's color change to red");
    }
}
```
```
public class TestAdapter {
    public static void main(String[] args) {
        Apple apple = new Apple();
        AppleDecorator appleDecorator = new ApplDecorator(apple);
        appleDecorator.getAppleColor("green");          //橘子也可以调用getAppleColor方法了,底层实际依然是getOrangeColor方法
    }
}
```
输出结果
```
Apple color is: green
apple's color change to red
```

## 适配器模式和装饰器模式的区别
适配器模式和装饰器模式都有一个别名叫包装模式，他们都起到包装一个类或对象的作用。这两个模式看似很像
但是：
* 适配器模式的以以是将一个接口转变成另外一个接口，它的目的是通过改变接口来达到重复使用的目的。
* 装饰器模式是不改变被装饰的接口，只是增强原有对象的功能而已。
