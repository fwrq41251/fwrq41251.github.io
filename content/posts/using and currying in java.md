---
title: Using and Currying in Java
date: 2016-03-09
categories:
  - Programming
tags:
  - using
  - currying
  - Java 8
slug: using-and-currying-in-java
---

C#里面有一个常用的关键字using,它可以保证资源在使用完毕后被正常关闭,使得程序员不必手动的关闭资源.
use using:

```
using (Font font1 = new Font("Arial", 10.0f))
{
  byte charset = font1.GdiCharSet;
}
```

without using:

```
{
  Font font1 = new Font("Arial", 10.0f);
  try
  {
    byte charset = font1.GdiCharSet;
  }
  finally
  {
    if (font1 != null)
    ((IDisposable)font1).Dispose();
  }
}
```

java 7以后java里也有了类似的特性。如果一个类实现了java.lang.AutoCloseable，那么你可以使用如下的语法。

```
try(ClassImplementingAutoCloseable obj = new ClassImplementingAutoCloseable())
{
  ...
}
```

接下来介绍如果一个类没有实现AutoCloseable，而又想实现类似using的功能应该怎么做。
MyFile是一个在使用之后需要被关闭的资源(需要调用close方法)。

```
class MyFile {

  private final String fileName;

  MyFile(String fileName) {
    super();
    this.fileName = fileName;
  }

  void close() {
    System.out.println("closing file:" + fileName);
  }

  void print() {
    System.out.println("fileName:" + fileName);
  }
}
```

MyFileHelper是MyFile的一个工具类。
currier接收第一个参数myFile，返回一个`Consumer<Consumer<MyFile>>`。
意思是这个consumer接收另外一个consumer作为参数（另外这个consumer才会真正的消费myFile）。
在currier的第二个参数被应用之前，它是不知道myFile会如何被消费的。
当然你也可以把consumer换成function，资源消费之后再产生一个结果。
这里用到了[Currying](https://en.wikipedia.org/wiki/Currying)，Currying是在函数式语言中接收单参数的函数模拟多参数的函数的技术。
使用Currying让它在语法上它更接近using，把资源的初始化和资源的处理分开。
同时，consumer和myfile两个参数能够在不同的地方被确定，它们可以被互相隔离的测试。

```
class MyFileHelper {

  static Consumer<Consumer<MyFile>> using(MyFile file) {
    Function<MyFile, Consumer<Consumer<MyFile>>> currier = myFile -> consumer -> {
      try {
        consumer.accept(myFile);
        } finally {
          myFile.close();
        }
      };
      Consumer<Consumer<MyFile>> curried = currier.apply(file);
      return curried;
    }
  }
}
```

最后是测试代码。

```
public static void main(String[] args) {
  MyFileHelper.using(new MyFile("my-file")).accept(
    myFile -> myFile.print());
  }
}
```

输出:

```
fileName:my-file
closing file:my-file
```

完整的代码可以在这里看到：
[gist](https://gist.github.com/fwrq41251/f6b82e201ff3b63d11ca)
