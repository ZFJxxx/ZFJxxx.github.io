---
layout: post
title:  多表查询
date:   2017-11-06 19:15:10
categories:  基础
tags:  MySQL
keywords: 
description:         
---

## 合并结果集 
1.作用：合并结果集就是把两个select语句的查询结果合并到一起.

2.合并结果集有两种方式： 
* UNION：去除重复记录，例如：SELECT * FROM t1 UNION SELECT * FROM t2； 
* UNION ALL：不去除重复记录，例如：SELECT * FROM t1 UNION ALL SELECT * FROM t2。

PS:.要求：被合并的两个结果：列数、列类型必须相同。

## 连接查询 （重要） 
连接查询就是求出多个表的乘积，例如t1连接t2，那么查询出的结果就是t1*t2。 
![](http://p7lixluhf.bkt.clouddn.com/LinkQuery.jpg)

连接查询会产生笛卡尔积,那么多表查询产生这样的结果并不是我们想要的，那么怎么去除重复的，不想要的记录呢，当然是通过条件过滤。通常要查询的多个表之间都存在关联关系，那么就通过关联关系去除笛卡尔积。

![](http://p7lixluhf.bkt.clouddn.com/20170414235333391.png)
使用主外键关系做为条件来去除无用信息
```
SELECT * FROM emp,dept WHERE emp.deptno=dept.deptno；
在多表查询中，在使用列时必须指定列所从属的表，例如emp.deptno表示emp表的deptno列
```
上面查询结果会把两张表的 所有列都查询出来，也许你不需要那么多列，这时就可以指定要查询的列了。
```
SELECT emp.ename,emp.sal,emp.COMM,dept.dname 
FROM emp,dept 
WHERE emp.deptno=dept.deptno;
```
还可以为表指定别名，然后在引用列时使用别名即可。
```
SELECT e.ename,e.sal,e.COMM,d.dname 
FROM emp AS e,dept AS d
WHERE e.deptno=d.deptno;     [其中AS是可以省略的]
```

### 内连接查询 
上面的查询结果就是内连接查询，但它不是SQL标准中的查询方式。 内连接最大的特点就是只返回两个表中互相匹配的记录，而那些不能匹配的记录就被自动去除了 
标准的内连接查询为：
```
SELECT * 
FROM 表名1
INNER JOIN 表名2 
ON 连接规则;
INNER JOIN 表名3
ON 连接规则；          [不使用WHERE，而是使用ON]
```
如果将上面查询emp和dept，使用内连接的方式就是：
```
SELECT * FROM emp e INNER JOIN dept d ON e.deptno=d.deptno;
```
可以输出同样的结果。

内连接分为[等值连接,自然连接,不等值连接]3种

* 等值连接就是连接规则用=号连接起来，如上;不等值链接就是连接规则不为等号
* 自然连接:
连接查询会产生无用笛卡尔积，我们通常使用主外键关系等式来去除它。而自然连接无需你去给出主外键等式，它会自动找到这一等式： 
两张连接的表中名称和类型完全一致的列作为条件，例如emp表和dept表都存在deptno列，并且类型一致，所以会被自然连接找到，然后将他们连接在一起.
```
SELECT * FROM emp NATURAL JOIN dept
```

### 左外连接查询 
左外连接查询就是以左表为主，左侧表的所有记录都包含到结果集中，而右边表中有匹配的记录包含到结果集。
例子：Student表
![](http://p7lixluhf.bkt.clouddn.com/SQLstudent.png)

Score表
  ![](http://p7lixluhf.bkt.clouddn.com/Score.png)

```
SELECT s.stuid,s.stuname,c.score 
FROM student AS s 
LEFT OUTER JOIN score AS c 
ON s.stuid = c.stuid;
```
![](http://p7lixluhf.bkt.clouddn.com/leftJoin.png)
### 右外连接查询 
右外连接查询就是以右表为主，右侧表的所有记录都包含到结果集中，而左边表中有匹配的记录包含到结果集。
```
SELECT s.stuid,s.stuname,c.score
FROM student AS s 
RIGHT OUTER JOIN score AS c 
ON s.stuid = c.stuid;
```
![](http://p7lixluhf.bkt.clouddn.com/rightJoin.png)


## 子查询
一个select语句中包含另一个完整的select语句。 子查询就是嵌套查询，即SELECT中包含SELECT，如果一条语句中存在两个，或两个以上SELECT，那么就是子查询语句了。

子查询出现的位置： 
* 1.where后，作为条为被查询的一条件的一部分； 
* 2.from后，作表；

当子查询出现在where后作为条件时，还可以使用如下关键字： 
* any 
* all 所有


练习：

（1）工资高于JONES的员工。 
分析： 查询条件：工资>JONES工资，其中JONES工资需要一条子查询。

```
SELECT * FROM emp WHERE sal > (SELECT sal FROM emp WHERE ename='JONES');
```
（2）查询与SCOTT同一个部门的员工。 
```
SELECT * FRMO emp WHERE deptno = (SELECT deptno FROM emp WHERE ename = 'SCOTT');
```
子查询作为条件：子查询形式为单行单列

（3）工资高于30号部门所有人的员工信息 
```
SELECT * FROM emp WHERE sal>( SELECT MAX(sal) FROM emp WHERE deptno=30);
```
查询条件：工资高于30部门所有人工资，其中30部门所有人工资是子查询。高于所有需要使用all关键字。
```
SELECT * FROM emp WHERE sal > ALL(SELECT sal FROM emp WHERE deptno=30)
``
子查询作为条件：子查询形式为多行单列（当子查询结果集形式为多行单列时可以使用ALL或ANY关键字）
