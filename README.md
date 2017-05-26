### RDD Cheatsheet

#### Initialize Spark Context

~~~
val appName = "learn spark rdd"
val master = "local"  //used for local testing
val conf: SparkConf = new SparkConf().setAppName(appName).setMaster(master)
val sc: SparkContext = new SparkContext(conf)
~~~

#### Create RDD from existing scala collection

~~~
val collection: Seq[Int] = Seq(1, 2, 3, 4, 5)
val parCollection: RDD[Int] = sc.parallelize(collection)
~~~

#### Handle external datasets
Load text file from local filesystem
~~~
val fileContent: RDD[String] = sc.textFile("src/main/resources/global-house-price-index-2016-q2.csv") 
~~~

Load all files from directory as (filename, content)
~~~
val dirContent: RDD[(String, String)] = sc.wholeTextFiles("src/main/resources") 
~~~

Load Hadoop sequence file which is in binary format converted into `[K, V]` types from `sequenceFile[K, V]`
~~~
val seqFileContent: RDD[(String, Int)] = sc.sequenceFile[String, Int]("path/to/file")
~~~


#### Transformations

Transformations don't execute the computations, they can be expressed as simple function `RDD[A] => RDD[B]`

| **name**                | **psuedocode**                                                                   | **comments**                                                                                                                                              |
|-------------------------|----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| *map*                   | `RDD[B] = RDD[A].map(A => B)`                                                    |                                                                                                                                                           |
| *filter*                | `RDD[A] = RDD[A].filter(A => Boolean)`                                           |                                                                                                                                                           |
| *flatMap*               | `RDD[B] = RDD[A].flatMap(A => TraversableOnce[B])`                               | passed function may return RDD or other collection                                                                                                        |
| *mapPartition*          | `RDD[B] = RDD[A].mapPartition(Iterator[A] => Iterator[B])`                       | mapping over every partition separately, might be used to avoid heavy preparation as db connecting for every element and make it once for whole partition |
| *mapPartitionWithIndex* | `RDD[B] = RDD[A].mapPartitionWithIndex((Int,Iterator[A]) => Iterator[B])`        | it additionally gives the index of partition to use                                                                                                       |
| *persist*               | `RDD[A] = RDD[A].persist()`                                                      | it saves the value of first computation in memory for further usage                                                                                       |
| *cache*                 | `RDD[A] = RDD[A].cache()`                                                        | works as `persist`, but the storage level is always `MEMORY_ONLY`                                                                                         |
| *sample*                | `RDD[A] = RDD[A].sample(withReplacement: Boolean, fraction: Double, seed: Long)` | sample the data - size close to the fraction of RDD size. If replacement is enabled the data may be sampled multiple times                                |
| *union*                 | `RDD[A] = RDD[A].union(RDD[A])`                                                  | creates union of two datasets, same elements appears multiple times                                                                                       |
| *intersection*          | `RDD[A] = RDD[A].intersection(RDD[A])`                                           | creates intersection of two datasets, result RDD contains unique elements                                                                                 |
| *distinct*              | `RDD[A] = RDD[A].distinct()`                                                     | result RDD contains unique elements                                                                                                                       |
| *cartesian*             | `RDD[(A,B)] = RDD[A].cartesian(RDD[B])`                                          | result RDD contains cartesian product of two RDD's                                                                                                        |
| *pipe*                  | `RDD[String] = RDD[A].pipe(...)`                                                 | pass elements from RDD to external process, results always as string                                                                                      |
| *coalesce*              | `RDD[A] = RDD[A].coalesce(numOfPartitions, shuffle: Boolean = false)`            | reduce number of partitions in given RDD to `numOfPartitions` number, may not shuffle data depending on shuffle value                                     |
| *repartition*           | `RDD[A] = RDD[A].repartition(numOfPartitions)`                                   | change number of partitions in given RDD to `numOfPartitions` number, always shuffle data                                                                 |

Transformations that work on key-value pair RDD's

| **name**         | **psuedocode**                                                            | **comments**                                                                                                                                                                   |
|------------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *groupByKey*     | `RDD[(A,Iterable[B])] = RDD[(A,B)].groupByKey()`                          | group all entries with the same key                                                                                                                                            |
| *reduceByKey*    | `RDD[(A,B)] = RDD[(A,B)].reduceByKey((B,B) => B)`                         | performs reduce on all entries with the same key                                                                                                                               |
| *aggregateByKey* | `RDD[(A,C)] = RDD[(A,B)].aggregateByKey(zero: C)((C,B) => C, (C,C) => C)` | performs aggregation on all entries with the same key, the first function is used to aggregate given partition and the second one is to consolidate results between partitions |
| *sortByKey*      | `RDD[(A,B)] = RDD[(A,B)].sortByKey(ascending: Boolean = true)`            | sort all entries by key                                                                                                                                                        |
| *join*           | `RDD[(A,(B,C))] = RDD[(A,B)].join(RDD[(A,C)])`                            | return all pairs of elements where keys match                                                                                                                                  |
| *leftOuterJoin*  | `RDD[(A,(B,Option[C]))] = RDD[(A,B)].leftOuterJoin(RDD[(A,C)])`           | return all pairs of elements where keys match if the second RDD doesn't contain given key, the returned pair will contain None                                                 |
| *rightOuterJoin* | `RDD[(A,(Option[B],C))] = RDD[(A,B)].rightOuterJoin(RDD[(A,C)])`          | return all pairs of elements where keys match if the first RDD doesn't contain given key, the returned pair will contain None                                                  |
| *fullOuterJoin*  | `RDD[(A,(Option[B],Option[C]))] = RDD[(A,B)].fullOuterJoin(RDD[(A,C)])`   | return all pairs of elements, represented as tuple of 2 Option                                                                                                                 |
| *cogroup*        | `RDD[(A,(Iterable[B], Iterable[C])] = RDD[(A,B)].cogroup(RDD[(A,C)])`     | return RDD with tuple of key and tuple of all values from both RDD matching to given key. Works up to 3 RDD's passed in parameter. `groupWith` is an alias for cogroup         |

#### Actions

Actions execute the computations, it can be expressed as function `RDD[A] => A`

| **name**           | **psuedocode**                                                     | **comments**                                                                                                                                                                   |
|--------------------|--------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| *reduce*           | `A = RDD[A].reduce((A,A) => A)`                                    |                                                                                                                             |
| *collect*          | `Array[A] = RDD[A].collect()`                                      | it brings all computed values from different workers into the driver scope. Without collecting the result may be incomplete |
| *take*             | `Array[A] = RDD[A].take(Int)`                                      | works in similar way as `collect`, safer to use instead of `collect` due to memory issues                                   |
| *first*            | `A = RDD[A].first()`                                               | takes first element                                                                                                         |
| *count*            | `Long = RDD[A].count()`                                            | count number of elements in RDD                                                                                             |
| *takeSample*       | `Array[A] = RDD[A].takeSample(withReplacement: Boolean, num: Int)` | takes sample data of `num` size. If replacement is enabled the data may be sampled multiple times                           |
| *takeOrdered*      | `Array[A] = RDD[A].takeOrdered(num: Int)`                          | takes num elements from RDD ordered in ASC direction                                                                        |
| *saveAsTextFile*   | `Unit = RDD[A].saveAsTextFile(path: String)`                       | saves data to the file in given path. Elements are written in they `toString` form                                          |
| *saveAsObjectFile* | `Unit = RDD[A].saveAsObjectFile(path: String)`                     | saves data to the SequenceFile in given path. Elements are written in they serialized form                                  |

Actions that work on key-value pair RDD's

| **name**     | **psuedocode**                         | **comments**                                             |
|--------------|----------------------------------------|----------------------------------------------------------|
| *countByKey* | `Map[A,Int] = RDD[(A,B)].countByKey()` | count all entries with the same key and create local map |
| *foreach*    | `Unit = RDD[A].foreach(A => Unit)`     |                                                          |

