---
title: Item1,考虑使用静态工厂方法替代构造器
date: 2018-10-7 15:50:00
updated: 2018-10-7 15:50:00
comments: true
---

## 简述

客户端可以通过类提供的公共构造方法获取类实例，除此之外，还可以使用静态工厂方法。

``` java
public static Boolean valueOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

<!--more-->

### 好处

#### 拥有名称

使用静态工厂方法替代构造器，不同于构造方法名称固定，可自定义见名知义的方法名，相对于不同参数签名的多个构造方法设计，更加优雅。

#### 不创建新对象

使用静态工厂方法替代构造器，不像构造方法，每次在被调用的时候，都需要new一个实例。

#### 返回子类型

使用静态工厂方法替代构造器，不像构造方法，它们可以返回其返回类型的任何子类型的对象。
例如，Java Collections Framework 的接口有45个实用程序实现，提供不可修改的集合，同步集合等。
几乎所有这些实现都是通过静态工厂方法在一个不可实例化的类`java.util.Collections`中获取的。

``` java
public static <T> List<T> unmodifiableList(List<? extends T> list) {
	return (list instanceof RandomAccess ?
			new UnmodifiableRandomAccessList<>(list) :
			new UnmodifiableList<>(list));
}
```

#### 返回对象可变化

A fourth advantage of static factories is that the class of the returned object can vary from call to call as a function of the input parameters.

``` java
/**
 * Creates an empty enum set with the specified element type.
 *
 * @param <E> The class of the elements in the set
 * @param elementType the class object of the element type for this enum
 *     set
 * @return An empty enum set of the specified type.
 * @throws NullPointerException if <tt>elementType</tt> is null
 */
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
	Enum<?>[] universe = getUniverse(elementType);
	if (universe == null)
		throw new ClassCastException(elementType + " not an enum");

	if (universe.length <= 64)
		return new RegularEnumSet<>(elementType, universe);
	else
		return new JumboEnumSet<>(elementType, universe);
}
```

#### 返回对象不需要存在

A fifth advantage of static factories is that the class of the returned object need not exist when the class containing the method is written

服务提供程序框架java.util.ServiceLoader

### 缺点

#### 不能subclassed

仅提供静态工厂方法的主要限制是，由于类没有public或protected的构造方法(一般不提供公开构造方法，不绝对，例如Boolean)，因此不能子类化。

``` java
class A {
	private A(B b) {
		//...
	}
	
	public static A of(B b) {
		return new A(b);
	}
}
```

#### 不利于查询

使用静态工厂方法的第二个主要的缺点是，程序员查找起来比较困难。
这些方法没有在API文档中突出，因此很难弄清楚如何实例化一个提供静态工厂方法而不是构造函数的类。
以下是静态工厂方法的一些常用名称。

from—A type-conversion method that takes a single parameter and returns a corresponding instance of this type, for example:

Date d = Date.from(instant);

of—An aggregation method that takes multiple parameters and returns an in-stance of this type that incorporates them, for example:

Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);

• valueOf—A more verbose alternative to from and of, for example:

BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);

instance or getInstance—Returns an instance that is described by its pa-rameters (if any) but cannot be said to have the same value, for example:

StackWalker luke = StackWalker.getInstance(options);

create or newInstance—Like instance or getInstance, except that the method guarantees that each call returns a new instance, for example:

Object newArray = Array.newInstance(classObject, arrayLen);

getType—Like getInstance, but used if the factory method is in a different class. Type is the type of object returned by the factory method, for example:

FileStore fs = Files.getFileStore(path);

newType—Like newInstance, but used if the factory method is in a different class. Type is the type of object returned by the factory method, for example:

BufferedReader br = Files.newBufferedReader(path);

• type—A concise alternative to getType and newType, for example:

List<Complaint> litany = Collections.list(legacyLitany);

In summary, static factory methods and public constructors both have their uses, and it pays to understand their relative merits. Often static factories are preferable, so avoid the reflex to provide public constructors without first consid-ering static factories.

参考：
[Effective Java Third Edition](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)，Joshua bloch