---
title: 理解lambda表达式
date: 2018-3-5 19:00:00
updated: 2018-3-5 20:00:00
comments: true
categories:
- Java
tags:
- 教程
- Java
---

Lambda表达式是Java 8中的新特性，是Java迈入函数式编程的第一步，它不依赖于任何类而被创建，它能像对象一样被传递和按需执行。


## Lambda与单方法接口

函数式编程常被用于实现事件监听器，事件监听器在java中常被定义为一个接口，该接口通常只有一个方法：

```java
public interface StateChangeListener {
    public void onStateChange(State oldState, State newState);
}
```

这个接口定义了一个方法，期望状态发生改变时被调用。

<!--more-->

Java 7中，必须实现此接口才能监听状态更改。假设有一个类`StateOwner`，它能注册状态事件监听器：

```java
public class StateOwner {
    public void addStateListener(StateChangeListener listener) {
        // Add state listener...
    }
}
```

Java 7中，可以使用匿名类实现接口添加事件监听器：

```java
StateOwner stateOwner = new StateOwner();

stateOwner.addStateListener(new StateChangeListener() {
    public void onStateChange(State oldState, State newState) {
        System.out.println("State changed");
    }
});
```

而在Java 8中，可直接使用lambda表达式添加事件监听器：

```java
StateOwner stateOwner = new StateOwner();

stateOwner.addStateListener(
    (oldState, newState) -> System.out.println("State changed")
);
```

下面一行就是lambda表达式:

```java
(oldState, newState) -> System.out.println("State changed")
```

lambda表达式与`addStateListener()`方法参数的参数类型相匹配，如果lambda表达式与参数类型相匹配(本例中为StateChangeListener)，那么lambda表达式将转换为实现与该参数相同的接口的函数。

Java lambda表达式只能用于与之匹配的类型为单一方法的接口。在上面的例子中，lambda表达式被用作类型为`StateChangeListener`的方法的参数。`StateChangeListener`只有一个方法，因此lambda表达式与该接口成功匹配。

### Lambda与接口之间的匹配

单一方法接口有时候也被称之为函数接口。函数接口匹配lambda分为以下几步：

确保接口只有一个方法
lambda表达式参数匹配函数接口方法参数
lambda返回值类型匹配函数接口方法返回值类型


## Lambda表达式类型推导

Java 8之前匿名实现也需要指定具体的接口：

```java
stateOwner.addStateListener(new StateChangeListener() {
    public void onStateChange(State oldState, State newState) {
        // do something with the old and new state.
    }
});
```

下面使用lambda表达式，未提及`StateChangeListener`接口，但是编译器可根据`addStateListener`方法声明推断出参数类型：

```java
stateOwner.addStateListener(
    (oldState, newState) -> System.out.println("State changed")
);
```

lambda表达式中的参数类型也可以类型推导，例如`(oldState, newState)`可通过onStateChange()的方法声明推导而出。


## Lambda表达式参数

Java lambda表达式实际上类似于方法，因此lambda表达式可以像方法一样使用参数。 前面显示的lambda表达式的(oldState, newState)部分指定了lambda表达式所需的参数。这些参数必须与单个方法接口上方法的参数相匹配。在这种情况下，这些参数必须与StateChangeListener接口的onStateChange()方法的参数匹配：

```java
public void onStateChange(State oldState, State newState);
```

至少lambda表达式和方法中的参数数量必须匹配。其次，如果在lambda表达式中指定了任何参数类型，则这些类型也必须匹配。(类型放入lambda表达式参数，本文稍后会介绍)

### 无参数

方法无参数，可以写成这样：

```java
() -> System.out.println("Zero parameter lambda");
```


### 一个参数

方法一个参数：

```java
(param) -> System.out.println("One parameter: " + param);
```

一个参数的时候，亦可以省略圆括号：

```java
param -> System.out.println("One parameter: " + param); 
```

### 多个参数

```java
(p1, p2) -> System.out.println("Multiple parameters: " + p1 + ", " + p2);
```

### 参数类型

如果编译器无法从lambda匹配的函数接口方法中推断出参数类型，那么为lambda表达式指定参数类型有时可能是必需的，这种情况下编译器会提示。Lambda表达式指定参数类型：

```java
(Car car) -> System.out.println("The car is: " + car.getName());
```

如你所见，明确为car指定类型`Car`。

## Lambda函数体

Lambda表达式body在 "->" 右侧被指定，即类似于方法体：
```java
 (oldState, newState) -> System.out.println("State changed")
```

如果需要多行，使用{}括起来：

```java
 (oldState, newState) -> {
    System.out.println("Old state: " + oldState);
    System.out.println("New state: " + newState);
  }
```

## Lambda返回值

Lambda表达式也可以有返回值，使用return语句即可：

```java
 (param) -> {
    System.out.println("param: " + param);
    return "return value";
  }
```

简短的形式：

```java
(a1, a2) -> { return a1 > a2; }
```

等价于:

```java
(a1, a2) -> a1 > a2;
```

## 用作对象

A Java lambda expression is essentially an object. You can assign a lambda expression to a variable and pass it around, like you do with any other object. Here is an example:

```java
public interface MyComparator {
    public boolean compare(int a1, int a2);
}
```
```java
MyComparator myComparator = (a, b) -> a > b;

boolean result = myComparator.compare(2, 5);
```

第一个代码块显示了lambda表达式实现的接口。 第二个代码块显示了lambda表达式的定义，lambda表达式如何分配给变量，最后是如何通过调用它实现的接口方法来调用lambda表达式。

## 集合示例

```java
    // 迭代List
    List<String> items = new ArrayList<>();
    items.add("A");
    items.add("B");
    items.add("C");

    items.forEach(
        item -> {
          System.out.println(item);
        });

    // 迭代Map
    Map<String, Integer> maps = new HashMap<>();
    maps.put("sunwukong", 80);
    maps.put("zhubajie", 59);
    maps.put("shaseng", 62);
    maps.put("tangseng", 87);

    maps.forEach(
        (key, value) -> {
          System.out.println(key + ", " + value);
        });
```

扩展阅读，[Java8 lambda表达式10个示例](http://www.importnew.com/16436.html)

参考：
[Java Lambda Expressions](http://tutorials.jenkov.com/java/lambda-expressions.html)，Jakob Jenkov