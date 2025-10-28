---
title: exception handling and dynamic scope
date: 2017-07-26
categories:
  - Programming
tags:
  - dynamic scope
slug: exception-handling -and-dynamic-scope
---

最近在上coursera上的programming language的时候，Dan提到exception handling和dynamic scope很相似，但是只是简单的提了一下，并没有说哪里相似，接下来就比较一下这两者。

先看一个例子。

```scala
val x = 1

def f() = x

def f2() = {
  val x = 2
  f()
}
```

这里f2()的值是什么呢，在scala里是1，因为scala用的是lexical scope，如果是dynamic scope那就是2。

为了说明lexical scope和dynamic scope，首先要搞清楚自由变量的概念，自由变量就是函数里除了传进来的参数以外的变量，在f()里面就是x，因为x不是f()的参数。

在lexical scope的情况下，自由变量的值是函数定义时自由变量的值。这时候f()里面的x就是1。实际上这时候函数包括了两个部分，一部分是它的code，另一部分是自由变量和它的值的映射，这两个部分合起来称为闭包。函数调用的时候自由变量的值是确定的，就是闭包里的值。

在dynamic scope的情况下，自由变量的值就不太好确定了，函数调用的时候会去查runtime stack的符号表，自由变量最后的值（后面的值会覆盖前面的）就是就是它这次函数调用里的值。

最后来看一下exception handling是怎么一回事，假设我们在代码的很多地方，handle了同一个异常，我们光看代码是不能确定最后哪个handling会起作用的。当一个异常发生时，只有在runtime，函数调用栈顶的那个函数最后会handle那个异常。是不是和dynamic scope有点相似呢。
