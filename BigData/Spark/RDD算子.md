>RDD是Spark对于分布式数据的统一抽象，用于囊括所有内存中和磁盘中的分布式数据实体。
>RDD的全称是Resilient Distributed Dataset，即 “弹性分布式数据集”。它定义了一系列分布式数据的基本属性与处理方法。
# 一、HelloWord

## 首先来引入一个WordCount的例子
---

~~~scala
import org.apache.spark.rdd.RDD 

// 这里的下划线"_"是占位符，代表数据文件的根目录 
val rootPath: String = _ val file: String = s"${rootPath}/text.txt" 

// 读取文件内容 
val lineRDD: RDD[String] = spark.sparkContext.textFile(file) 

// 以行为单位做分词 
val wordRDD: RDD[String] = lineRDD.flatMap(line => line.split(" ")) 
val cleanWordRDD: RDD[String] = wordRDD.filter(word => !word.equals("")) 

// 把RDD元素转换为（Key，Value）的形式 
val kvRDD: RDD[(String, Int)] = cleanWordRDD.map(word => (word, 1)) 

// 按照单词做分组计数 
val wordCounts: RDD[(String, Int)] = kvRDD.reduceByKey((x, y) => x + y) 

// 打印词频最高的5个词汇 
wordCounts.map{case (k, v) => (v, k)}.sortByKey(false).take(5)
~~~

看这段代码中的 RDD

	源文件 -> lineRDD -> wordRDD -> cleadWordRDD -> kvRDD -> wordCounts

在数据形态的转换过程中，每个RDD都会通过dependencies属性来记录它所依赖的前一个、或是多个RDD，简称“父RDD”。与此同时，RDD使用compute属性，来记录从父RDD到当前RDD的转换操作。

拿Word Count当中的wordRDD来举例，它的父RDD是lineRDD，因此，它的dependencies属性记录的是lineRDD。从lineRDD到wordRDD的转换，其所依赖的操作是flatMap，因此，wordRDD的compute属性，记录的是flatMap这个转换函数。而这其中的 `flatMap` 、`filter`、`map`、`reduceByKey` 就被称为算子。

RDD有四大属性：
-   partitions：数据分片 —— 即 数据实体的具体不同形态
-   partitioner：分片切割规则 —— 即 数据不同形态的划分规则
-   dependencies：RDD依赖 —— 即 对前一个或是多个RDD的依赖
-   compute：转换函数 —— 即 从上一种形态转变为当前形态的方式

## RDD的编程模型和延迟计算
---

> 编程模型指导我们如何进行代码实现，而延迟计算是Spark分布式运行机制的基础。

1. 通过调用textFile API生成lineRDD，然后用flatMap算子把lineRDD转换为wordRDD；
2. 接下来，filter算子对wordRDD做过滤，并把它转换为不带空串的cleanWordRDD；
3. 然后，为了后续的聚合计算，map算子把cleanWordRDD又转换成元素为（Key，Value）对的kvRDD；
4. 最终，我们调用reduceByKey做分组聚合，把kvRDD中的Value从1转换为单词计数。
 

> 开发者调用的各类Transformations算子，并不立即执行计算，当且仅当开发者调用Actions算子时，之前调用的转换算子才会付诸执行。这就被称为 “**延迟计算**”（Lazy Evaluation）。

# 二、RDD算子

> Spark官网给出的算子：https://spark.apache.org/docs/latest/rdd-programming-guide.html

## Transformations类算子
---

| 适用范围  |   算子用途    |                            算子合集                            |
|:---------:|:-------------:|:--------------------------------------------------------------:|
|  任意RDD  | RDD内数据转化 | map / flatmap / fliter / mapPartitons / mapPartitionsWithIndex |
| Parid RDD | RDD内数据聚合 |     groupByKey / sortByKey / reduceByKey / aggregateByKey      |
|  任意RDD  | RDD间数据整合 |       union / intersection / join / cogroup / cartesian        |
|  任意RDD  |   数据整理    |                       sample / distinct                        |
|  任意RDD  |   数据分布    |  coalesce / repartition / repartitionAndSortWithinPartitions   | 



## Actions类算子
---

| 适用范围 |  算子用途  |                         算子集合                          |
|:--------:|:----------:|:---------------------------------------------------------:|
| 任意RDD  |  数据收集  | collect / first / take / count / takeSample / takeOrdered |
| 任意RDD  | 数据持久化 |  saveAsTextFile / saveAsSequenceFile / saveAsObjectFile   |
| 任意RDD  |  数据遍历  |                          foreach                          |
