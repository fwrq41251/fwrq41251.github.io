---
title: Pattern matching in Java(翻译)
date: 2015-11-04
categories:
  - Programming
tags:
  - pattern-matching
  - Functional Programming
  - visitor pattern
slug: pattern-matching-in-java
---

模式匹配是一种来源于函数式语言的特性,它有点像java中的switch,
然而比switch更加强大,它除了能对值进行匹配之外还能对类型进行匹配.
配合scala的模板类,它能写出非常简洁的类型、值匹配的代码.
当你需要根据对象的类型或者值的不同来进行不同的逻辑处理时,就应该使用模式匹配.
下面的内容将主要针对其中的**类型**匹配展开.

### scala中的模式匹配

```
object patterns {
  abstract class Foo
  class Bar(val bar: Int) extends Foo
  class Baz(val baz: String) extends Foo

  def handle(f: Foo) =
    f match {
      case b: Bar => b.bar
      case b: Baz => b.baz
    }

  handle(new Bar(42))                       //> res0: Any = 42
  handle(new Baz("Luhrmann"))               //> res1: Any = Luhrmann
}
```

上面定义了一个抽象类Foo和两个子类Bar,Baz(object关键字说明这个类是单例的),
两个子类各有一个名称与类型都不同的成员属性,另外有一个handle方法来处理子类对象.
scala具有类型推导,所以这里没有声明方法的返回类型.

### java中的做法

#### instanceof

简单的做法是使用instanceof

```
abstract class BadFoo {

  static class Bar extends BadFoo {
    int bar;

    private Bar(int bar) {
      this.bar = bar;
    }

  }

  static class Baz extends BadFoo {
    String baz;

    private Baz(String baz) {
      this.baz = baz;
    }

  }

  static void handle(BadFoo f) {
    if (f instanceof Bar) {
      Bar b = (Bar) f;
      System.out.println("I have " + b.bar + " bars");
    } else if (f instanceof Baz) {
      Baz b = (Baz) f;
      System.out.println("I have the " + b.baz + " baz");
    }
  }

  public static void main(String[] args) {
    handle(new Bar(42));
    handle(new Baz("Luhrmann"));
  }
}
```

然而这么写有几个问题,1.在那些使用动态代理的代码中,产生的对象永远不会是原始类的一个实例,
instanceof不起作用.2.无法在编译时期进行类型判断,`if (f instanceof Bar) Baz b = (Baz) f;`
以上代码能编译通过,但是不能运行.3.当增加了一个新的子类时,不能保证旧代码被及时更新.

#### Moving logic into the class

另一种做法是把逻辑放到数据对象中,就像OOP应该做的.

```
abstract class OoFoo {
  abstract void handle();

  static class Bar extends OoFoo {
    private int bar;

    private Bar(int bar) {
      this.bar = bar;
    }

    @Override
    void handle() {
      System.out.println("I have " + bar + " bars");
    }
  }

  static class Baz extends OoFoo {
    private String baz;

    private Baz(String baz) {
      this.baz = baz;
    }

    @Override
    void handle() {
      System.out.println("I have the " + baz + " baz");
    }
  }

  public static void main(String[] args) {
    new Bar(42).handle();
    new Baz("Luhrmann").handle();
  }
}
```

这么做刚开始还不错,然而当类似的方法逐渐变多,你的数据对象就变成了存放业务逻辑的地方.
使用访问者模式能够保持数据对象简单,把业务逻辑放到响应的处理类中.

### 访问者模式

#### Basic Visitor

基本的访问者模式包括以下内容:

* 一个抽象类或者接口包括一个以Visitor作为参数,返回类型为Visitor中泛型类型的方法match
* 一个Visitor类,包括各个子类对象为参数的case方法
* 一系列实现了match方法的子类
* 实现各个case方法的应用代码

```
abstract class Foo {
  abstract <T> T match(Visitor<T> visitor);

  interface Visitor<T> {
    T caseBar(Bar b);
    T caseBaz(Baz b);
  }

  static class Bar extends Foo {
    final int bar;

    private Bar(int bar) {
      this.bar = bar;
    }

    @Override
    <T> T match(Visitor<T> visitor) {
      return visitor.caseBar(this);
    }
  }

  static class Baz extends Foo {
    final String baz;

    private Baz(String baz) {
      this.baz = baz;
    }

    @Override
    <T> T match(Visitor<T> visitor) {
      return visitor.caseBaz(this);
    }
  }

  static void handle(Foo f) {
    System.out.println(f.match(new Visitor<String>() {
      @Override
      public String caseBar(Bar b) {
        return "I have " + b.bar + " bars";
      }
      @Override
      public String caseBaz(Baz b) {
        return "I have the " + b.baz + " baz";
      }
    }));
  }

  public static void main(String[] args) {
    handle(new Bar(42));
    handle(new Baz("Luhrmann"));
  }
}
```

#### Default visitor

有时候Visitor没必要重写全部的case方法,只重写部分方法,其他时候返回默认值.

```
class DefaultFooVisitor<T> implements Foo.Visitor<T> {
  T defaultValue;

  private DefaultFooVisitor(T defaultValue) {
    this.defaultValue = defaultValue;
  }

  @Override
  public T caseBar(Bar b) {
    return defaultValue;
  }

  @Override
  public T caseBaz(Baz b) {
    return defaultValue;
  }

  static int countBars(Foo f) {
    return f.match(new DefaultFooVisitor<Integer>(0) {
      @Override
      public Integer caseBar(Bar b) {
        return b.bar;
      }
    });
  }
}
```

缺点是当增加了新的子类,新增的case方法可能会忘记重写.
解决这个问题的一种方法是先把DefaultFooVisitor中新增的case方法改为抽象的,
解决了所有的编译问题之后再来实现DefaultFooVisitor中新增的case方法.

#### Void or Unit return values

有时候返回值是不需要的,可以用`Unit.unit`或者`java.lang.Void`来表达空的返回类型.

```
class PartitionFoos {
  void doStuff(Collection<Foo> foos) {
    final List<Bar> bars = new ArrayList<>();
    final List<Baz> bazes = new ArrayList<>();
    for (Foo f : foos) {
      f.match(new Visitor<Void>() {

        @Override
        public Void caseBar(Bar b) {
          bars.add(b);
          return null;
        }

        @Override
        public Void caseBaz(Baz b) {
          bazes.add(b);
          return null;
        }

      });
    }
  }

}
```

### 解构模式匹配

前面的例子都只包含了类型匹配,最后的一个例子会展示java处理包含类型以及值匹配的方法.
先来看一下scala的代码:

```
object patterns {
  abstract class Foo
  case class Bar(val bar: Int) extends Foo
  case class Baz(val baz: String) extends Foo

  def handle(f: Foo) =
    f match {
      case Bar(i) if (i > 10) => 10
      case Bar(i) => i
      case Baz(s) => s
    }

  handle(new Bar(42))                             //> res0: Any = 10
  handle(new Bar(3))                              //> res0: Any = 3
  handle(new Baz("Luhrmann"))                     //> res1: Any = Luhrmann
}
```

简洁,灵活.再来看一下java版本的:

```
abstract class Foo {
  abstract <T> T match(Visitor<T> visitor);

  interface Visitor<T> {
    T caseManyBar();
    T caseBar(int i);
    T caseBaz(String s);
  }

  static class Bar extends Foo {
    private int bar;

    private Bar(int bar) {
      this.bar = bar;
    }

    @Override
    <T> T match(Visitor<T> visitor) {
      return bar > 10 ? visitor.caseManyBar() : visitor.caseBar(bar);
    }
  }

  static class Baz extends Foo {
    private String baz;

    private Baz(String baz) {
      this.baz = baz;
    }

    @Override
    <T> T match(Visitor<T> visitor) {
      return visitor.caseBaz(baz);
    }
  }

  static void handle(Foo f) {
    System.out.println(f.match(new Visitor<String>() {
      @Override
      public String caseBar(int i) {
        return "I have " + i + " bars";
      }
      @Override
      public String caseManyBar() {
        return "I have many bars";
      }
      @Override
      public String caseBaz(String s) {
        return "I have the " + s + " baz";
      }
    }));
  }

  public static void main(String[] args) {
    handle(new Bar(42));
    handle(new Bar(3));
    handle(new Baz("Luhrmann"));
  }
}
```

16vs58,很难说用多余的42行来模拟相同的功能是否值得.
访问者模式的好处是,让你以一种类型安全的方式来处理子类.
在你添加新的子类时,编译器会告诉你哪些代码需要修改.
当你下一次遇到ClassCastException时你应该考虑使用它.

原文:
[Pattern matching in Java with the Visitor pattern](http://eng.wealthfront.com/2015/02/pattern-matching-in-java-with-visitor.html)
