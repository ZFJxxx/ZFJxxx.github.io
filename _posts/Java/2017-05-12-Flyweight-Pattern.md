---
layout: post
title:  享元模式
date:   2017-05-12 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---
享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。

享元模式尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象。

其主要思想是：用 HashMap 存储这些对象

## 实例
1、JAVA 中的 String，如果有则返回，如果没有则创建一个字符串保存在字符串缓存池里面。 

2、数据库的数据池。

优点：大大减少对象的创建，降低系统的内存，使效率提高。


创建一个接口
```
public interface Shape {
   void draw();
}
```
创建实现接口的实体类。
```
public class Circle implements Shape {
   private String color;
   private int x;
   private int y;
   private int radius;
 
   public Circle(String color){
      this.color = color;     
   }
 
   public void setX(int x) {
      this.x = x;
   }
 
   public void setY(int y) {
      this.y = y;
   }
 
   public void setRadius(int radius) {
      this.radius = radius;
   }
 
   @Override
   public void draw() {
      System.out.println("Circle: Draw() [Color : " + color 
         +", x : " + x +", y :" + y +", radius :" + radius);
   }
}
```

创建一个工厂，生成基于给定信息的实体类的对象。

```
import java.util.HashMap;
 
public class ShapeFactory {
   private static final HashMap<String, Shape> circleMap = new HashMap<>();
 
   //中心思想就是用一个HashMap，如果实例存在就直接从HashMap中取，不存在就新建然后存入HashMap中
   public static Shape getCircle(String color) {
      Circle circle = (Circle)circleMap.get(color);
 
      if(circle == null) {
         circle = new Circle(color);
         circleMap.put(color, circle);
         System.out.println("Creating circle of color : " + color);
      }
      return circle;
   }
}
```
