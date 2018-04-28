---
layout: post
title:  深入String、StringBuffer、StringBuilder 
date:   2016-10-10 19:15:10
categories:  Java
tags:  Java
keywords: String
description: 
---
----------------------------------

## String 类

String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，不仅效率低下，而且大量浪费有限的内存空间。 
```
String a = "a";   //假设a指向地址0x0001 
a = "b";          //重新赋值后a指向地址0x0002，但0x0001地址中保存的"a"依旧存在，但已经不再是a所指向的,a 已经指向了其它地址。 
```
因此String的操作都是改变赋值地址而不是改变值操作。 

练习：
1.思考下面两个的输出
```
String s1 = "abc";
String s2 = "abc";
System.out.println(s1 == s2);               //输出true
System.out.println(s1.equals(s2));          //输出true
```
发现两个输出的都为true， equals方法是比较器字符序列，肯定输出true
但是 == 号比较的是地址值，为什么为true呢？

![enter image description here](http://p7lixluhf.bkt.clouddn.com/String1.jpg)

原来是因为s1 s2 都指向**常量池**中的同一个abc，所以他们地址值相等
==号返回true。
**常量池中没有这个字符串对象，就创建一个。如果有就直接用。**

2.问下面这句话在内存中创建了几个对象
```
String s1 = new String("abc");
```
两个。 如图

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/String3.jpg)

先在常量池中创建了abc，因为new会进堆内存，所以在堆内存中创建一个对象，将常量池中的abc拷贝进去，然后堆内存中的对象将自己的地址值赋给s1，s1就可以通过地址找到堆内存中的对象了。

3.判断下面代码的输出
```
String s1 = new String("abc");
String s2 = "abc";
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```
因为s1记录的是堆内存中的地址值，s2记录的是常量池中的地址值。所以s1 == s2 输出false . equals方法输出的肯定是true。

4.判断定义为String的s1 和 s2是否相等

```
String s1 = "a" + "b" + "c";
String s2 = "abc";
System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```
equals方法输出的肯定是true，因为字符串内容相同。
==方法输出的也是true，java有常量优化机制。在编译时，“a”+"b"+"c" 就是 “abc”赋给s1

5.判断定义为String的s1 和 s2是否相等
```
String s1 = "ab";
String s2 = "abc";
String s3 = s1 + "c";
System.out.println(s3 == s2);         //输出false
System.out.println(s3.equals(s2));    //输出true
```

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/String2.jpg)

Java语言提供对字符串串联符号（“+”）以及其他对象转换成字符串的特殊支持。字符串串联式通过StringBuilder类以及其append方法实现的。用StringBuilder使字符串串联生成一个StringBuilder对象，再通过toString方法将StringBuilder转换成字符串。如上图。

下面S3和S4的效果相同.
```
String s3 = s1+ "c";
String s4 = new StringBuilder().append(s1).append("c").toString();
```

所以==方法输出的是false 。 equals方法输出true。

## 连接符号 "+" 本质

下面S3和S4的效果相同.
```
String s3 = s1+ "c";
String s4 = new StringBuilder().append(s1).append("c").toString();
```
不过在for循环中，千万不要使用 "+" 进行字符串拼接。

```
public class test {
    public static void main(String[] args) {
        run1();
        run2();
    }   

    public static void run1() {
        long start = System.currentTimeMillis();
        String result = "";
        for (int i = 0; i < 10000; i++) {
            result += i;
        }
        System.out.println(System.currentTimeMillis() - start);
    }

    public static void run2() {
        long start = System.currentTimeMillis();
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            builder.append(i);
        }
        System.out.println(System.currentTimeMillis() - start);
    }
}

```

在for循环中使用 "+" 和StringBuilder进行1万次字符串拼接，耗时情况如下：
1、使用 "+" 拼接，平均耗时 150ms；
2、使用StringBuilder拼接，平均耗时 1ms；

for循环中使用 "+" 拼接为什么这么慢？下面是run1方法的字节码指令

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/String4.jpg)

5 ~ 34 行对应for循环的代码，可以发现，每次循环都会重新初始化StringBuilder对象，导致性能问题的出现。

## StringBuffer

StringBuffer是可变类，和线程安全的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。 每个StringBuffer对象都有一定的缓冲区容量，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量。 而String是不可变得字符序列.

StringBuffer与StringBuilder都是继承于AbstractStringBuilder，唯一的区别就是StringBuffer的函数上都有synchronized关键字。

### StringBuffer类的构造方法

```
StringBuffer buf=new StringBuffer(); //分配长16字节的字符缓冲区 
StringBuffer buf=new StringBuffer(512); //分配长512字节的字符缓冲区 
StringBuffer buf=new StringBuffer("this is a test")//在缓冲区中存放了字符串，并在后面预留了16字节的空缓冲区。 
```

### StringBuffer的添加功能

```
public class Demo2_StringBuffer {
	/**
	 * * A:StringBuffer的添加功能
	 * public StringBuffer append(String str):
	 * 可以把任意类型数据添加到字符串缓冲区里面,并返回字符串缓冲区本身
	 */
	public static void main(String[] args) {
		StringBuffer sb = new StringBuffer();
		StringBuffer sb2 = sb.append(true);
		StringBuffer sb3 = sb.append("hello");
		StringBuffer sb4 = sb.append(100);
		////StringBuffer类中重写了toString方法,显示的是对象中的属性值
		System.out.println(sb.toString());			
		System.out.println(sb2.toString());
		System.out.println(sb3.toString());
		System.out.println(sb4.toString());
	}

}
```
输出结果：
```
truehello100
truehello100
truehello100
truehello100
```

结果输出的都是truehello100。这是因为这4个引用指向的是同一个对象，我们操作的都是对同一个对象进行添加。所以说StringBuffer是可变的字符序列，String是不变的字符序列。String在你赋完新值后，之前的就变成垃圾。而StringBuffer不会。

StringBuffer是字符串缓冲区,当new的时候是在堆内存创建了一个对象,底层是一个长度为16的字符数组
当调用添加的方法时,不会再重新创建对象,在不断向原缓冲区添加字符

* public StringBuffer insert(int offset,String str):
* 在指定位置把任意类型的数据插入到字符串缓冲区里面,并返回字符串缓冲区本身
```
	public static void main(String[] args) {
		StringBuffer sb = new StringBuffer("1234");
		sb.insert(3, "hello");	//在指定位置添加元素,如果没有指定位置的索引就会报索引越界异常
		System.out.println(sb);
	}
```
输出:123hello4



### StringBuffer和String的相互转换
String -- StringBuffer
* a:通过构造方法
* b:通过append()方法
```
StringBuffer sb1 = new StringBuffer("hello");	//通过构造方法将字符串转换为StringBuffer对象
System.out.println(sb1);
		
StringBuffer sb2 = new StringBuffer();
sb2.append("hello");		      //通过append方法将字符串转换为StringBuffer对象
System.out.println(sb2);
```

StringBuffer -- String
* a:通过构造方法
* b:通过toString()方法
* c:通过subString(0,length);

```
StringBuffer sb = new StringBuffer("hello");	

String s1 = new String(sb);		 //通过构造将StringBuffer转换为String
System.out.println(s1);
		
String s2 = sb.toString();		 //通过toString方法将StringBuffer转换为String
System.out.println(s2);
		
String s3 = sb.substring(0, sb.length()); //通过截取子字符串将StringBuffer转换为String
System.out.println(s3);
```


## StringBuffer和StringBuilder的区别

StringBuffer和StringBuilder类功能基本相似，主要区别在于StringBuffer类的方法是多线程、安全的，而StringBuilder不是线程安全的，相比而言，StringBuilder类会略微快一点。对于经常要改变值的字符串应该使用StringBuffer和StringBuilder类。 

1.线程安全 
StringBuffer 线程安全 
StringBuilder 线程不安全 

2.速度 
一般情况下,速度从快到慢:StringBuilder>StringBuffer>String,这种比较是相对的，不是绝对的。 

3.总结 
（1）.如果要操作少量的数据用 = String 
（2）.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 
（3）.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer


----------
