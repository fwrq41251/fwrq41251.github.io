---
title: spark combineByKey
date: 2016-12-01
categories:
  - Programming
tags:
  - spark
  - scala
slug: spark-combineByKey
---

combineByKey是spark的一个pair transformation。当你需要根据key来合并你的value，而你期望的合并后的value的返回类型和原本value的类型不同的时候你就会用到它（reduceByKey的返回类型则和原本的pair相同）。用groupByKey再对groupBy的结果处理最终可以达到相同的效果，但是性能却不一样，实际上combineByKey提供的是更高级的抽象。性能上的差异可以看这篇文章:[avoid groupByKey](https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/best_practices/prefer_reducebykey_over_groupbykey.html)

combineByKey函数长这样，V表示原本的value的类型，C表示你期望返回的合并之后的类型。

```scala
def combineByKey[C](
      createCombiner: V => C,
      mergeValue: (C, V) => C,
      mergeCombiners: (C, C) => C): RDD[(K, C)] = self.withScope {
    combineByKeyWithClassTag(createCombiner, mergeValue, mergeCombiners)(null)
}
```

看一下combineByKey的几个参数：
第一个参数createCombiner表示如何由一个原本的value构造出combiner的对象，combiner可能包括一些value的field，以及一些value中没有的初始值。
第二个参数mergeValue表示value是如何被合并到combiner里的。
第三个参数mergeCombiners表示各个combiner的结果是如何被合并的。

最后看一个用combineByKey来求平均数的例子。它比求和多了一个步骤，需要把和除以个数（这个时候简单的reduceByKey就满足不了需求了）。所以combiner需要两个参数，一个和，一个个数。

```
val scores: RDD[(String, Int)] = sc.parallelize(List(("chinese", 88), ("chinese", 90), ("math", 60), ("math", 87)))
scores.combineByKey(
      (v) => (v, 1),
      (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1),
      (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
).collectAsMap().foreach(e => println("course:" + e._1 + ", avg:" + e._2._1 / e._2._2))
//    course:chinese, avg:89
//    course:math, avg:73
```

代码在scala-2.11.8,spark-core_2.11-2.0.0下编译通过。
参考:[combinebykey](https://zhangyi.gitbooks.io/spark-in-action/content/chapter2/combinebykey.html)
