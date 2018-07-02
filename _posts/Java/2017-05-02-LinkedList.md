---
layout: post
title:  深入LinkedList
date:   2017-05-02 19:15:10
categories:  Java
tags:  Java容器
keywords: LinkedList
description: 
---
----------------------------------

## LinkedList

LinkedList是基于**链表**实现的，是一种**双向循环链表**。
LinkedList既然是一种双向循环链表，必然有一个存储单元，看一下LinkedList的基本存储单元，它是LinkedList中的一个内部类：
```
private static class Entry<E> {
    E element;
    Entry<E> next;
    Entry<E> previous;
    ...
}
```

看到LinkedList的Entry中的”E element”，就是它真正存储的数据。”Entry< E > next”和”Entry< E > previous”表示的就是这个存储单元的前一个存储单元的引用地址和后一个存储单元的引用地址。

 ![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/LinkedList1.jpg)
 
### LinkedList的性质
 
![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/LinkedList2.jpg)


## 添加元素
首先看下LinkedList添加一个元素是怎么做的，假如我有一段代码：
```
public static void main(String[] args) {
     List<String> list = new LinkedList<String>();
     list.add("111");
     list.add("222");
}
```

逐行分析main函数中的三行代码是如何执行的，执行完”List< String> list = new LinkedList< String>()”之后可以这么表示：

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/Linkedlist3.jpg)

new了一个Entry出来名为header。
接着看第4行add一个字符串”111″做了什么：
```
public boolean add(E e) {
     addBefore(e, header);
     return true;
```
```
private Entry<E> addBefore(E e, Entry<E> entry) {
    Entry<E> newEntry = new Entry<E>(e, entry, entry.previous);
    newEntry.previous.next = newEntry;
    newEntry.next.previous = newEntry;
    size++;
    modCount++;
    return newEntry;
}
```
第2行new了一个Entry出来，可能不太好理解，根据Entry的构造函数，我把这句话”翻译”一下，可能就好理解了：

1、newEntry.element = e;

2、newEntry.next = header.next;

3、newEntry.previous = header.previous;

header.next和header.previous上图中已经看到了，都是0×00000000，那么假设new出来的这个Entry的地址是0×00000001，继续画图表示：

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/LL1.jpg)

add了一个字符串”222″做了什么，假设新new出来的Entry的地址是0×00000002，画图表示：

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/LL5.jpg)

就是这样不断插入。


## 查找元素
看一下LinkedList的代码是怎么写的：

```
public E get(int index) {
    return entry(index).element;
}
```
```
private Entry<E> entry(int index) {
    if (index < 0 || index >= size)
        throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);
    Entry<E> e = header;
    if (index < (size >> 1)) {
        for (int i = 0; i <= index; i++)
            e = e.next;
    } else {
        for (int i = size; i > index; i--)
            e = e.previous;
    }
    return e;
}
```

这段代码就体现出了双向链表的好处了。双向链表增加了一点点的空间消耗（每个Entry里面还要维护它的前置Entry的引用），同时也增加了一定的编程复杂度，却大大提升了效率。

由于LinkedList是双向链表，所以LinkedList既可以向前查找，也可以向后查找，**第6行~第12行的作用就是：当index小于数组大小的一半的时候（size >> 1表示size / 2，使用移位运算提升代码运行效率），向后查找；否则，向前查找。**

这样，在我的数据结构里面有10000个元素，刚巧查找的又是第10000个元素的时候，就不需要从头遍历10000次了，向后遍历即可，一次就能找到我要的元素。



## 删除元素
看完了添加元素，我们看一下如何删除一个元素。和ArrayList一样，LinkedList支持按元素删除和按下标删除，**前者会删除从头开始匹配的第一个元素**。用按下标删除举个例子好了，比方说有这么一段代码：

```
public static void main(String[] args) {
    List<String> list = new LinkedList<String>();
    list.add("111");
    list.add("222");
    list.remove(0);
}
```

也就是我想删除”111″这个元素。看看删除的源代码：

```
public E remove(int index) {
     return remove(entry(index));
}
```
```
private E remove(Entry<E> e) {
	if (e == header)
    throw new NoSuchElementException();
        E result = e.element;
		e.previous.next = e.next;
		e.next.previous = e.previous;
        e.next = e.previous = null;
        e.element = null;
        size--;
		modCount++;
        return result;
}
```
由源码可以看出，删除是将删除的Entry的previous、element、next都设置为了null，这三步的作用是**让虚拟机可以回收这个Entry**。

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/LinkedList6.jpg)


## 插入元素

插入元素就不细讲了，看一下插入元素的源代码：

```
public void add(int index, E element) {
    addBefore(element, (index==size ? header : entry(index)));
}
```

```
private Entry<E> addBefore(E e, Entry<E> entry) {
    Entry<E> newEntry = new Entry<E>(e, entry, entry.previous);
    newEntry.previous.next = newEntry;
    newEntry.next.previous = newEntry;
    size++;
    modCount++;
    return newEntry;
}
```


## LinkedList和ArrayList的对比

1、顺序插入速度ArrayList会比较快，因为ArrayList是基于数组实现的，数组是事先new好的，只要往指定位置塞一个数据就好了；LinkedList则不同，每次顺序插入的时候LinkedList将new一个对象出来，如果对象比较大，那么new的时间势必会长一点，再加上一些引用赋值的操作，所以顺序插入LinkedList必然慢于ArrayList

2、基于上一点，因为LinkedList里面不仅维护了待插入的元素，还维护了Entry的前置Entry和后继Entry，如果一个LinkedList中的Entry非常多，那么LinkedList将比ArrayList更耗费一些内存

3、数据遍历的速度，看最后一部分，这里就不细讲了，结论是：使用各自遍历效率最高的方式，ArrayList的遍历效率会比LinkedList的遍历效率高一些

4、有些说法认为LinkedList做插入和删除更快，这种说法其实是不准确的：

（1）LinkedList做插入、删除的时候，慢在寻址，快在只需要改变前后Entry的引用地址

（2）ArrayList做插入、删除的时候，慢在数组元素的批量copy，快在寻址

所以，如果待插入、删除的元素是在数据结构的前半段尤其是非常靠前的位置的时候，LinkedList的效率将大大快过ArrayList，因为ArrayList将批量copy大量的元素；越往后，对于LinkedList来说，因为它是双向链表，所以在第2个元素后面插入一个数据和在倒数第2个元素后面插入一个元素在效率上基本没有差别，但是ArrayList由于要批量copy的元素越来越少，操作速度必然追上乃至超过LinkedList。

从这个分析看出，如果你十分确定你插入、删除的元素是在前半段，那么就使用LinkedList；如果你十分确定你删除、删除的元素在比较靠后的位置，那么可以考虑使用ArrayList。如果你不能确定你要做的插入、删除是在哪儿呢？那还是建议你使用LinkedList吧，因为一来LinkedList整体插入、删除的执行效率比较稳定，没有ArrayList这种越往后越快的情况；二来插入元素的时候，弄得不好ArrayList就要进行一次扩容，记住，ArrayList底层数组扩容是一个既消耗时间又消耗空间的操作。
