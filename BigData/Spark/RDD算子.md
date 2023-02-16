>RDD的全称是Resilient Distributed Dataset，即 “弹性分布式数据集”。
>RDD是Spark对于分布式数据的统一抽象，用于囊括所有内存中和磁盘中的分布式数据实体。
>它定义了一系列分布式数据的基本属性与处理方法。RDD有四大属性：
> -   partitions：数据分片
> -   partitioner：分片切割规则
> -   dependencies：RDD依赖
> -   compute：转换函数
# 一、HelloWord

首先来引入一个WordCount的例子

~~~scala
import org.apache.spark.rdd.RDD 

// 这里的下划线"_"是占位符，代表数据文件的根目录 
val rootPath: String = _ val file: String = s"${rootPath}/wikiOfSpark.txt" 

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
