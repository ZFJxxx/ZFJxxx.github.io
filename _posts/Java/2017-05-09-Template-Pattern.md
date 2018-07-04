---
layout: post
title:  模板模式
date:   2017-05-09 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---
## 模板模式

在模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。

* 优点： 1、封装不变部分，扩展可变部分。 2、提取公共代码，便于维护。 3、行为由父类控制，子类实现。

* 缺点：每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

* 使用场景： 1、有多个子类共有的方法，且逻辑相同。 2、重要的、复杂的方法，可以考虑作为模板方法。


创建一个抽象类，它的模板方法被设置为 final。  Game.java
```
public abstract class Game {
   abstract void initialize();
   abstract void startPlay();
   abstract void endPlay();
 
   //模板
   public final void play(){
 
      //初始化游戏
      initialize();
 
      //开始游戏
      startPlay();
 
      //结束游戏
      endPlay();
   }
}
```

Football.java   创建扩展了上述类的实体类。
```
public class Football extends Game {
 
   @Override
   void endPlay() {
      System.out.println("Football Game Finished!");
   }
 
   @Override
   void initialize() {
      System.out.println("Football Game Initialized! Start playing.");
   }
 
   @Override
   void startPlay() {
      System.out.println("Football Game Started. Enjoy the game!");
   }
}
```

TemplatePatternDemo.java    使用 Game 的模板方法 play() 来演示游戏的定义方式。
```
public class TemplatePatternDemo {
   public static void main(String[] args) {
      Game game = new Football();
      game.play();      
   }
}
```

输出：
```
Football Game Initialized! Start playing.
Football Game Started. Enjoy the game!
Football Game Finished!
```
