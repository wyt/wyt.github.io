---
title: Interview
date: 2017-07-18 14:45:00
categories:
- 面试
tags:
- 面试题
---

#### 推断打印结果
``` java
public static void main(String args[]) {
  String a = "hello2";
  final String b = "hello";
  String d = "hello";
  String c = b + 2;
  String e = d + 2;
  
  System.out.println((a == c));
  System.out.println((a == e));
}
```
结果：
``` java
true
false
```

解析：由于b的值确定且被final修饰，会被当做编译期常量使用，即编译期出现b变量的地方会被直接替换，
替换后，a和c都指向字符串常量池中的hello2地址，故a==c为true；而String e = d + 2;会被编译器转换为类似于 String e = new StringBuilder(d).append("2"); 故 a==e为false.

#### 请画出a，b，c，d内存分布图
```java
public static void main(String args[]) {
  int[] a = { 1, 2, 3 };
  int[] b = { 4, 5, 6 };
  int[] c = { 7, 8, 9 };
  int[][] d = { a, b, c };
}
```
解析：
![](https://objects.yongtao.wang/subject-momery.png)

#### 判断输出结果

##### I
```java
public class Example {
  String str = new String("Hello");
  char[] ch = {'a','b','c'};

  public static void main(String[] args) {
     Example ex = new Example();
     ex.change(ex.str, ex.ch);
     System.out.println(ex.str);
     System.out.println(ex.ch);
  }

  public void change(String str, char[] ch){
     str= "Hi";
     ch[1]= 'h';
  }
}
```
运行结果：
``` java
Hello
ahc
```
解析：
![](https://objects.yongtao.wang/images/20190705/stack-frame.png)

##### II
```java
public class Person {

  public int age;

  public static void changeEmployee(Person employee) {
    employee = new Person();
    employee.age = 1000;
  }

  public static void main(String[] args) {
    Person employee = new Person();
    employee.age = 100;
    changeEmployee(employee);
    System.out.println(employee.age);
  }
}
```

#### 哪些操作是原子性操作
```java
x = 10;         //语句1
y = x;          //语句2
x++;            //语句3
x = x + 1;      //语句4
```

``` java
x = 10;
```

解析：注意y = x，需要读取和赋值，并非原子性操作。

#### 以下代码是否有问题，并解释。
``` java
public class Example {
  public static void main(String[] args) {
    Outputer outputer = new Outputer();
    final Thread t0 = new Thread(() -> {
      outputer.output2("How are you?");
    });
    final Thread t1 = new Thread(() -> {
      outputer.output("Fine, thank you.");
    });
    t0.start();
    t1.start();
  }

  static class Outputer {
    public synchronized void output(String name) {
      int len = name.length();
      for (int i = 0; i < len; i++) {
        System.out.print(name.charAt(i));
      }
      System.out.println();
    }

    public static synchronized void output2(String name) {
      int len = name.length();
      for (int i = 0; i < len; i++) {
        System.out.print(name.charAt(i));
      }
      System.out.println();
    }
  }
}
```

解析：
`ouput和output2两个方法分别使用类对象和类实例对象的锁，两个线程不会竞争同一个锁资源，因而不会产生互斥，从而导致输出混乱。`


#### SQL查询

员工表emp：

| empno | ename  | job       | mgr  | hiredate   | sal     | comm    | deptno |
|-------|--------|-----------|------|------------|---------|---------|--------|
|  7369 | SMITH  | CLERK     | 7902 | 1980-12-17 | 800.00  | NULL    |     20 |
|  7499 | ALLEN  | SALESMAN  | 7698 | 1981-02-20 | 1600.00 | 300.00  |     30 |
|  7521 | WARD   | SALESMAN  | 7698 | 1981-02-22 | 1250.00 | 500.00  |     30 |
|  7566 | JONES  | MANAGER   | 7839 | 1981-04-02 | 2975.00 | NULL    |     20 |
|  7654 | MARTIN | SALESMAN  | 7698 | 1981-09-28 | 1250.00 | 1400.00 |     30 |
|  7698 | BLAKE  | MANAGER   | 7839 | 1981-05-01 | 2850.00 | NULL    |     30 |
|  7782 | CLARK  | MANAGER   | 7839 | 1981-06-09 | 2450.00 | NULL    |     10 |
|  7788 | SCOTT  | ANALYST   | 7566 | 1987-07-13 | 3000.00 | NULL    |     20 |
|  7839 | KING   | PRESIDENT | NULL | 1981-11-17 | 5000.00 | NULL    |     10 |
|  7844 | TURNER | SALESMAN  | 7698 | 1981-09-08 | 1500.00 | 0.00    |     30 |
|  7876 | ADAMS  | CLERK     | 7788 | 1987-07-13 | 1100.00 | NULL    |     20 |
|  7900 | JAMES  | CLERK     | 7698 | 1981-12-03 | 950.00  | NULL    |     30 |
|  7902 | FORD   | ANALYST   | 7566 | 1981-12-03 | 3000.00 | NULL    |     20 |
|  7934 | MILLER | CLERK     | 7782 | 1982-01-23 | 1300.00 | NULL    |     10 |

部门表dept：

| deptno | dname      | loc      |
|--------|------------|----------|
|     10 | ACCOUNTING | NEW YORK |
|     20 | RESEARCH   | DALLAS   |
|     30 | SALES      | CHICAGO  |
|     40 | OPERATIONS | BOSTON   |

##### 查询部门表在员工表中不存在的部门号;

``` sql
SELECT d.deptno
FROM dept d
WHERE NOT EXISTS (
SELECT NULL
FROM emp e
WHERE d.deptno = e.deptno);
```

##### 查询工资高于部门平均工资的员工;

``` sql
SELECT e2.ename,e2.sal,e2.deptno,r1._avg
FROM emp e2
INNER JOIN (
SELECT e1.deptno, AVG(e1.sal) AS _avg
FROM emp e1
GROUP BY e1.deptno) r1 ON e2.deptno = r1.deptno AND e2.sal > r1._avg
ORDER BY e2.deptno ASC, e2.sal DESC;
```

##### 查询部门工资前三名的员工;

```sql
SELECT t0.deptno, t0.ename, t0.sal
FROM emp t0
WHERE (
SELECT COUNT(1)
FROM emp t1
WHERE t0.deptno = t1.deptno AND t1.sal >= t0.sal) <= 3
ORDER BY deptno,sal DESC;
```

解析： 员工工资排名前3，可以转换为: 工资大于等于自己的人数(count)小于等于3