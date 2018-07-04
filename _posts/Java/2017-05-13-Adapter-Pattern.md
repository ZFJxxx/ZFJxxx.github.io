---
layout: post
title:  适配器模式
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
        Apple apple1 = new Apple();
        apple1.getAppleColor("green");

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
