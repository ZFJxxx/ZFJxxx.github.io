---
layout: post
title:  策略模式
date:   2017-05-14 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---

## 策略模式
在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。

我们将这些算法封装成一个一个的类，任意地替换。

```
public interface Strategy {
   public int doOperation(int num1, int num2);
}
```
```
public class OperationAdd implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 + num2;
   }
}

public class OperationSubstract implements Strategy{
   @Override
   public int doOperation(int num1, int num2) {
      return num1 - num2;
   }
}
```
```
public class Context {
   private Strategy strategy;
 
   public Context(Strategy strategy){
      this.strategy = strategy;
   }
 
   public int executeStrategy(int num1, int num2){
      return strategy.doOperation(num1, num2);
   }
}
```
使用 Context 来查看当它改变策略 Strategy 时的行为变化。

```
public class StrategyPatternDemo {
   public static void main(String[] args) {
      Context context = new Context(new OperationAdd());    
      System.out.println("10 + 5 = " + context.executeStrategy(10, 5));
 
      context = new Context(new OperationSubstract());      
      System.out.println("10 - 5 = " + context.executeStrategy(10, 5));
   }
}
```

执行程序，输出结果：
```
10 + 5 = 15
10 - 5 = 5
```
