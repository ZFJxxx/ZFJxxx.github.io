---
layout: post
title:  工厂模式
date:   2017-05-19 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---
## 工厂模式

工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

应用实例：
* 1、您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现。 
* 2、Hibernate 换数据库只需换方言和驱动就可以。

优点： 
* 1、一个调用者想创建一个对象，只要知道其名称就可以了。 
* 2、扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。 
* 3、屏蔽产品的具体实现，调用者只关心产品的接口。

创建一个接口: Shape.java
```
public interface Fruit {
   void eat();
}
```

创建实现接口的实体类:
apple.java
```
public class apple implements Fruit {
 
   @Override
   public void draw() {
      System.out.println("i want to eat apple!");
   }
}
```
banana.java
```
public class banana implements Fruit {
 
   @Override
   public void eat() {
      System.out.println("i want to eat banana!");
   }
}
```


创建一个工厂，生成基于给定信息的实体类的对象:FruitFactory.java
public class FruitFactory {
    
   //使用 getFruit 方法获取形状类型的对象
   public Fruit getFruit(String fruit){
      if(fruit == null){
         return null;
      }        
      if(fruit.equalsIgnoreCase("apple")){
         return new apple();
      } else if(fruit.equalsIgnoreCase("banana")){
         return new banana();
      } 
      return null;
   }
}

使用该工厂，通过传递类型信息来获取实体类的对象:FactoryPatternDemo.java
public class FactoryPatternDemo {
 
   public static void main(String[] args) {
      FruitFactory fruitFactory = new FruitFactory();

      Fruit apple = fruitFactory.getFruit("apple");
      apple.eat();
      
      Fruit banana = fruitFactory.getFruit("banana");
      banana.eat();
   }
}

执行程序，输出结果：
```
i want to eat apple!
i want to eat banana!
```
