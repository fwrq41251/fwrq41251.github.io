---
title: Pattern matching in Java(续)
date: 2015-11-30
categories:
  - Programming
tags:
  - pattern-matching
  - Java 8
slug: pattern-matching-in-java(续)
---

上一篇文章里翻译了java8以前模式匹配在java中的使用情况.
这一篇文章会简单介绍cyclops库中模式匹配的相关内容.
[cyclops](https://github.com/aol/cyclops)是基于java8的一个扩展库,包括了模式匹配,stream工具类,monad,元组等内容.

直接上完整的代码.handleByType方法匹配类型,handleByValue方法匹配值.
isType方法传入一个名为TypedFunction的函数式接口,这个接口里有一个默认方法可以获取lanbda表达式中的泛型具体类型.
需要注意的是当需要值匹配时必须重写类的equals方法,因为Object中的equals方法认为两个不同的对象是不相等的.
```
package org.gradle.pattern;

import org.junit.Test;

import com.aol.cyclops.matcher.builders.Matching;

public class Patterns {

	abstract class Foo {

	}

	class Bar extends Foo {

		public Bar(Integer bar) {
			super();
			this.bar = bar;
		}

		private Integer bar;

		public Integer getBar() {
			return bar;
		}

		public void setBar(Integer bar) {
			this.bar = bar;
		}

	}

	class Baz extends Foo {

		public Baz(String baz) {
			super();
			this.baz = baz;
		}

		private String baz;

		public String getBaz() {
			return baz;
		}

		public void setBaz(String baz) {
			this.baz = baz;
		}

		@Override
		public int hashCode() {
			final int prime = 31;
			int result = 1;
			result = prime * result + getOuterType().hashCode();
			result = prime * result + ((baz == null) ? 0 : baz.hashCode());
			return result;
		}

		@Override
		public boolean equals(Object obj) {
			if (this == obj)
				return true;
			if (obj == null)
				return false;
			if (getClass() != obj.getClass())
				return false;
			Baz other = (Baz) obj;
			if (!getOuterType().equals(other.getOuterType()))
				return false;
			if (baz == null) {
				if (other.baz != null)
					return false;
			} else if (!baz.equals(other.baz))
				return false;
			return true;
		}

		private Patterns getOuterType() {
			return Patterns.this;
		}

	}

	Object handleByType(Foo foo) {
		return Matching
				.when().isType((Bar bar) -> bar.getBar())
				.when().isType((Baz baz) -> baz.getBaz())
				.match(foo)
				.get();
	}

	Object handleByValue(Foo foo) {
		return Matching
				.when().isValue(new Baz("Luhrmann")).thenApply(baz -> baz.getBaz())
				.match(foo)
				.orElse("match nothing");
	}

	@Test
	public void test() {
		System.out.println(handleByType(new Bar(42)));          // 42
		System.out.println(handleByType(new Baz("letty")));     // letty
		System.out.println(handleByValue(new Baz("Luhrmann"))); // Luhrmann
		System.out.println(handleByValue(new Baz("")));         // match nothing
	}
}
```
虽然和scala中的guard和case class相比,java中的模式匹配还是不够灵活,
然而这毕竟缺少了语言层面的支持.尽管如此,使用cyclops还是比起用访问者模式
在可读性和实现的难易度上都大大进步了.
预告:下一篇博客可能会继续介绍cyclops,或者是和动静态语言相关的内容.
