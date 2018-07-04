---
layout: post
title:  观察者模式
date:   2017-05-18 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---

## 观察者模式
当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知它的依赖对象。观察者模式属于行为型模式。

其主要思想就是在抽象类里有一个 ArrayList 存放观察者们，然后自己发生改变的时候就遍历列表通知所有的观察者。
```
//这个接口是为了提供一个统一的观察者做出相应行为的方法
public interface Observer {
    void update(Subject subject);  
}
```
```
public class Observer1 implements Observer{
    public void update(Subject subject) {
        System.out.println("观察者1观察到" + subject.getClass().getSimpleName() + "发生变化");
        System.out.println("观察者1做出响应");
    }
}
```
```
public class Observer2 implements Observer{
    public void update(Subject subject) {
        System.out.println("观察者2观察到" + subject.getClass().getSimpleName() + "发生变化");
        System.out.println("观察者2做出响应");
    }
}
```
被观察的对象 Subject
```
//下面是被观察者，它有一个观察者的列表，并且有一个通知所有观察者的方法，通知的方式就是调用观察者通用的接口行为update方法。下面我们看它的代码。
public class Subject {
    List<Observer> observers = new ArrayList<Observer>();

    public void addObserver(Observer o){
        observers.add(o);
    }

    public void changed(){
        System.out.println("我是被观察者，我已经发生变化了");
        notifyObservers();//通知观察自己的所有观察者
    }

    public void notifyObservers(){
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
}
```
```
//下面我们使用客户端调用一下，看一下客户端如何操作。
public class Client {
    public static void main(String[] args) throws Exception {
        Subject subject = new subject();
        //加到观察者列表中
        subject.addObserver(new Observer1());
        subject.addObserver(new Observer2());

        subject.changed();
    }
}
```
输出结果
```
我是被观察者，我已经发生变化了
观察者1观察到suject发生变化
观察者1做出响应
观察者1观察到suject发生变化
观察者1做出响应
```
