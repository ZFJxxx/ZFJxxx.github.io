---
layout: post
title:  工厂模式  抽象工厂模式
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

创建一个接口: Fruti.java
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
```
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
```
使用该工厂，通过传递类型信息来获取实体类的对象:FactoryPatternDemo.java
```
public class FactoryPatternDemo {
   public static void main(String[] args) {
      FruitFactory fruitFactory = new FruitFactory();

      Fruit apple = fruitFactory.getFruit("apple");
      apple.eat();
      
      Fruit banana = fruitFactory.getFruit("banana");
      banana.eat();
   }
}
```
执行程序，输出结果：
```
i want to eat apple!
i want to eat banana!
```


## 抽象工厂模式
如果产品单一，最合适用工厂模式（比如上面只有水果），但是如果有多个业务品种、业务分类时（比如下面的水果和食物），通过抽象工厂模式产生需要的对象是一种非常好的解决方式。

再通俗深化理解下：工厂模式针对的是一个产品等级结构 ，抽象工厂模式针对的是面向多个产品等级结构的。


**创建一个接口: Fruit.java**
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
   public void eat() {
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

再创建一个接口:Food.java
```
public interface Food{
   void eat();
}
```
**创建实现接口的实体类**
noodle.java
```
public class noodle implements Food {
 
   @Override
   public void eat() {
      System.out.println("i want to eat noodle!");
   }
}
```
fish.java
```
public class fish implements Food {
 
   @Override
   public void eat() {
      System.out.println("i want to eat fish!");
   }
}
```

**为 Food 和 Fruit 对象创建抽象类来获取工厂 AbstractFactory.java**
```
public abstract class AbstractFactory {
   public abstract Fruit getFruit(String fruit);
   public abstract Shape getFood(String food) ;
}
```

创建扩展了 AbstractFactory 的工厂类，基于给定的信息生成实体类的对象。

```
public class FruitFactory extends AbstractFactory {
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
```

```
public class FoodFactory extends AbstractFactory {
   //使用 getFruit 方法获取形状类型的对象
   public Fruit getFood(String food){
      if(food == null){
         return null;
      }        
      if(food.equalsIgnoreCase("noodle")){
         return new noodle();
      } else if(fruit.equalsIgnoreCase("fish")){
         return new fish();
      } 
      return null;
   }
}
```

创建一个工厂创造器/生成器类，通过传递形状或颜色信息来获取工厂。

FactoryProducer.java
```
public class FactoryProducer {
   public static AbstractFactory getFactory(String choice){
      if(choice.equalsIgnoreCase("FRUIT")){
         return new FruitFactoryy();
      } else if(choice.equalsIgnoreCase("FOOD")){
         return new FoodFactory();
      }
      return null;
   }
}
```
```
public class AbstractFactoryPatternDemo {
   public static void main(String[] args) {
      //获取Fruit工厂
      AbstractFactory fruitFactory = FactoryProducer.getFactory("FRUIT");
      //获取apple的对象
      Fruit fruit = fruitFactory.getFruit("apple");
      fruit.eat();
 
      //获取Food工厂
      AbstractFactory foodFactory = FactoryProducer.getFactory("FOOD");
      //获取颜色为 Red 的对象
      Food food = foodFactory.getFood("noodle");
      food.eat();
   }
}
```
其实抽象工厂方法就是在工厂类上做了一层抽象，当业务中有多个业务品种时，先获取对应的工厂，再获取对象。
