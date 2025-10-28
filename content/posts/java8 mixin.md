---
title: Java8 Mixin
date: 2015-09-15
categories:
  - Programming
tags:
  - Java 8
  - Mixin
slug: java8-mixin
---

java8开始接口支持了默认方法,从此mixin也变为可能了.

先来看看mixin是什么:

>In object-oriented programming languages, a mixin is a class that contains a ***combination*** of methods from other classes. How such a combination is done depends on the language. If a combination contains all methods of combined classes, it is equivalent to ***multiple inheritance***. Mixins are sometimes described as being "included" rather than "inherited".

>Mixins encourage code reuse and can be used to avoid the inheritance ambiguity that multiple inheritance can cause (the "[diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)"), or to work around lack of support for multiple inheritance in a language. A mixin can also be viewed as an ***interface with implemented methods***.

mixin混合了其他类的功能但是又不会引发多重继承的问题.

下面是一个简单的迭代器.

```
public interface AbsIterator<T> {

 public boolean hasNext();

 public T next();
}
```

给上面的迭代器增加了foreach方法,接收一个consumer,
对每个元素执行相同的操作.

```
public interface RichIterator<T> extends AbsIterator<T> {

 public default void foreach(Consumer<? super T> consumer) {
  while (hasNext())
   consumer.accept(next());
 }
}
```

一个基于String的实现.

```
public class StringIterator implements AbsIterator<Character> {

 private String s;
 private int i = 0;

 public StringIterator(String s) {
  super();
  this.s = s;
 }

 @Override
 public boolean hasNext() {
  return i < s.length();
 }

 @Override
 public Character next() {
  Character character = s.charAt(i);
  i++;
  return character;
 }

}
```

最后是一个测试类.
测试用的迭代器即有StringIterator的实现,
又具有RichIterator的foreach方法.
在java8以前我们要怎么做这件事呢,
似乎是没法做,
由于接口没有默认方法,
我们必须在每个迭代器实现中重写foreach方法.
mixin增强了代码的重用能力.

```
public class StringIteratorTest {

 public static void main(String[] args) {
  Iter iter = new Iter("test");
  iter.foreach(System.out::println);
 }

 static class Iter extends StringIterator implements RichIterator<Character> {

  public Iter(String s) {
   super(s);
  }

 }
}
```

不过值得注意的是，java的mixin比scala在功能还是要弱一些，由于scala有with关键字，可以在实例化时进行mixin，可以直接强化trait的方法，类似于装饰器的作用。java里子类的实现则会直接覆盖掉父类。

参考:[scala-mixin](http://www.scala-lang.org/old/node/117)
