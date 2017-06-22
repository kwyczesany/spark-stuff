### Dataset Cheatsheet

#### Initialize Spark Session

~~~
val spark = SparkSession
  .builder()
  .appName("some app")
  .config("spark.master", "local")
  .getOrCreate()
  
import spark.implicits._    //import convenient dsl
~~~

#### Read DataFrame from json file

Important note: File has to contain one json object per line and the objects must be top level objects. They cannot be part of an array.

~~~ 
val df: DataFrame = spark.read.json("src/main/resources/d1.json")
~~~

You can print the DataFrame schema.

~~~
df.printSchema()
//[info] root
//[info]  |-- datasetid: string (nullable = true)
//[info]  |-- fields: struct (nullable = true)
//[info]  |    |-- country: string (nullable = true)
//[info]  |    |-- freq: string (nullable = true)
//[info]  |    |-- indicator: string (nullable = true)
//[info]  |    |-- obs_value: double (nullable = true)
//[info]  |    |-- time_format: string (nullable = true)
//[info]  |    |-- time_period: string (nullable = true)
//[info]  |    |-- unit_measure: string (nullable = true)
//[info]  |    |-- unit_mult: string (nullable = true)
//[info]  |-- record_timestamp: string (nullable = true)
//[info]  |-- recordid: string (nullable = true)
~~~

#### SQL API

Datasets can be treated as a database table, thus there are some functions that operates the same way as sql.

| **name**              | **psuedocode**                                                                                | **comments**                                                                                                                                                                                                          |
|-----------------------|-----------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *select*              | `DateFrame = Dataset[A].select(columns*)`                                                     |                                                                                                                                                                                                                       |
| *filter*              | `Dataset[A] = Dataset[A].filter(columnCondition)`                                             | column is a condition expressed by using `$"columnName" boolean expression` notation (e.g. `$"age" > 21`)                                                                                                             |
| *where*               | `Dataset[A] = Dataset[A].where(columnCondition)`                                              | works as `filter`                                                                                                                                                                                                     |
| *groupBy*             | `RelationalGroupedDataset = Dataset[A].groupBy(columns*)`                                     | `RelationalGroupedDataset` is a type on which can be performed different type of aggregations                                                                                                                         |
| *show*                | `Unit = Dataset[A].show(numRows: Int = 20, truncate: Boolean)`                                | print dataset to the console, if `truncate` set to true long values in columns will be shortened                                                                                                                      |
| *na*                  | `DataFrameNaFunctions = Dataset[A].na()`                                                      | `DataFrameNaFunctions` allows to work on dataset with missing data                                                                                                                                                    |
| *join*                | `DateFrame = Dataset[A].join(right: Dataset[B], usingColumns: Seq[String], joinType: String)` |                                                                                                                                                                                                                       |
| *sort*                | `Dataset[A] = Dataset[A].sort(columns*)`                                                      | new Dataset sorted by column expression                                                                                                                                                                               |
| *sortWithinPartition* | `Dataset[A] = Dataset[A].sortWithinPartition(columns*)`                                       | new Dataset with each partition sorted by column expression                                                                                                                                                           |
| *orderBy*             | `Dataset[A] = Dataset[A].orderBy(columns*)`                                                   | works as `sort`                                                                                                                                                                                                       |
| *apply*               | `Column = Dataset[A].apply(columnName)`                                                       | returns one column from dataset                                                                                                                                                                                       |
| *col*                 | `Column = Dataset[A].col(columnName)`                                                         | works as `apply`                                                                                                                                                                                                      |
| *union*               | `Dataset[A] = Dataset[A].union(Dataset[A])`                                                   | new Dataset containing rows from both Datasets with duplicated elements                                                                                                                                               |
| *intersect*           | `Dataset[A] = Dataset[A].intersect(Dataset[A])`                                               | new Dataset containing only rows that are present in both Datasets, the comparison is made on encoded form thus custom `equals` doesn't work                                                                          |
| *except*              | `Dataset[A] = Dataset[A].except(Dataset[A])`                                                  | new Dataset that is a result of XOR operation on two Datasets                                                                                                                                                         |
| *sample*              | `Dataset[A] = Dataset[A].sample(withReplacement: Boolean, fraction: Double, seed: Long)`      | sample the data - size close to the fraction of Dataset size. If replacement is enabled the data may be sampled multiple times                                                                                        |
| *withColumn*          | `DataFrame = Dataset[A].withColumn(colName: String, col: Column)`                             | new DataFrame with added column                                                                                                                                                                                       |
| *withColumnRenamed*   | `DataFrame = Dataset[A].withColumnRenamed(existingName: String, newName: String)`             | new DataFrame with renamed column                                                                                                                                                                                     |
| *drop*                | `DataFrame = Dataset[A].drop(columns*)`                                                       | new DataFrame without given columns                                                                                                                                                                                   |
| *dropDuplicates*      | `Dataset[A] = Dataset[A].dropDuplicates(columns*)`                                            | new Dataset with unique values in given columns                                                                                                                                                                       |
| *distinct*            | `Dataset[A] = Dataset[A].distinct()`                                                          | new Dataset with unique values                                                                                                                                                                                        |
| *head*                | `A = Dataset[A].head()`                                                                       |                                                                                                                                                                                                                       |
| *head*                | `Array[A] = Dataset[A].head(Int)`                                                             |                                                                                                                                                                                                                       |
| *collect*             | `Array[A] = Dataset[A].collect`                                                               |                                                                                                                                                                                                                       |
| *first*               | `A = Dataset[A].first()`                                                                      |                                                                                                                                                                                                                       |
| *foreach*             | `Unit = Dataset[A].foreach(A => Unit)`                                                        |                                                                                                                                                                                                                       |
| *persist*             | `Dataset[A] = Dataset[A].persist()`                                                           | it saves the value of first computation in memory for further usage                                                                                                                                                   |
| *cache*               | `Dataset[A] = Dataset[A].cache()`                                                             | works as `persist`, but the storage level is always `MEMORY_ONLY`                                                                                                                                                     |
| *agg*                 | `DataFrame = Dataset[A].agg(aggrExpressions)`                                                 | `aggrExpressions` can be list of tuples (`ds.agg("age" -> "max", "salary" -> "avg")`) or map (`ds.agg(Map("age" -> "max", "salary" -> "avg"))`) or list of column expressions (`ds.agg(max($"age"), avg($"salary"))`) |

#### Column expressions

| **name**           | **psuedocode**                                              | **comments**                                                                                                                      |
|--------------------|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| *-*                | `Column = -Column`                                          | negates all values in column                                                                                                      |
| *!*                | `Column = !Column`                                          | inverse boolean condition for the column                                                                                          |
| *===*              | `Column = Column === Any`                                   | checks equality the column and given value                                                                                        |
| *equalTo*          | `Column = Column.equalTo(Any)`                              | works as `===`                                                                                                                    |
| *=!=*              | `Column = Column =!= Any`                                   | checks inequality between column and given value                                                                                  |
| *notEqual*         | `Column = Column.notEqual(Any)`                             | works as `=!=`                                                                                                                    |
| *>*                | `Column = Column > Any`                                     | checks if column values are greater than given value                                                                              |
| *gt*               | `Column = Column.gt(Any)`                                   | works as `>`                                                                                                                      |
| *>=*               | `Column = Column >= Any`                                    | checks if column values are greater or equal to the given value                                                                   |
| *geq*              | `Column = Column.geq(Any)`                                  | works as `>=`                                                                                                                     |
| *<*                | `Column = Column < Any`                                     | checks if column values are less than given value                                                                                 |
| *lt*               | `Column = Column.lt(Any)`                                   | works as `<`                                                                                                                      |
| *<=*               | `Column = Column <= Any`                                    | checks if column values are less or equal to the given value                                                                      |
| *leq*              | `Column = Column.leq(Any)`                                  | works as `<=`                                                                                                                     |
| *<=>*              | `Column = Column <=> Any`                                   | checks inequality between column and given value with saf null value handling                                                     |
| *eqNullSafe*       | `Column = Column.eqNullSafe(Any)`                           | works as `<=>`                                                                                                                    |
| *when*             | `Column = Column.when(condition: Column, value: Any)`       | if given condition is true then return value, if no `otherwise` declared the null value will be returned for unmatched conditions |
| *otherwise*        | `Column = Column.otherwise(value: Any)`                     | return value in case of unmatched conditions in `when` clauses                                                                    |
| *between*          | `Column = Column.between(lowerBound: Any, upperBound: Any)` | checks if column values are in given range inclusively                                                                            |
| *isNan*            | `Column = Column.isNaN`                                     | checks if column values aren't numbers                                                                                            |
| *isNull*           | `Column = Column.isNull`                                    | checks if column values are nulls                                                                                                 |
| *isNotNull*        | `Column = Column.isNotNull`                                 | checks if column values aren't nulls                                                                                              |
| *||*               | `Column = Column || Any`                                    | boolean OR on conditions                                                                                                          |
| *or*               | `Column = Column.or(Column)`                                | works as `||`                                                                                                                     |
| *&&*               | `Column = Column && Any`                                    | boolean AND on conditions                                                                                                         |
| *and*              | `Column = Column.and(Column)`                               | works as `&&`                                                                                                                     |
| *+*                | `Column = Column + Any`                                     | result of sum of column values and expression                                                                                     |
| *plus*             | `Column = Column.plus(Any)`                                 | works as `+`                                                                                                                      |
| *-*                | `Column = Column - Any`                                     | result of subtract of column values and expression                                                                                |
| *minus*            | `Column = Column.minus(Any)`                                | works as `-`                                                                                                                      |
| *\**               | `Column = Column * Any`                                     | result of multiply of column values and expression                                                                                |
| *multiply*         | `Column = Column.multiply(Any)`                             | works as `*`                                                                                                                      |
| */*                | `Column = Column / Any`                                     | result of divide of column values and expression                                                                                  |
| *divide*           | `Column = Column.divide(Any)`                               | works as `/`                                                                                                                      |
| *%*                | `Column = Column % Any`                                     | result of modulo of column values and expression                                                                                  |
| *mod*              | `Column = Column.mod(Any)`                                  | works as `%`                                                                                                                      |
| *isin*             | `Column = Column.isin(Any*)`                                | checks if column values are in given set of values                                                                                |
| *like*             | `Column = Column.like(String)`                              | checks if column values match the given string                                                                                    |
| *rlike*            | `Column = Column.rlike(String)`                             | checks if column values match the given string with regex patterns                                                                |
| *substr*           | `Column = Column.substr(startPos: Column, len: Column)`     | return substrings of given column values, parameters expressed as column expressions                                              |
| *substr*           | `Column = Column.substr(startPos: Int, len: Int)`           | return substrings of given column values                                                                                          |
| *contains*         | `Column = Column.contains(Any)`                             | checks if column values contains given value                                                                                      |
| *startsWith*       | `Column = Column.startsWith(Column)`                        | checks if column values starts with given value                                                                                   |
| *endsWith*         | `Column = Column.endsWith(Column)`                          | checks if column values ends with given value                                                                                     |
| *alias*            | `Column = Column.alias(String)`                             | gives the alias name for column in `select` output                                                                                |
| *as*               | `Column = Column.as(String)`                                | works as `alias`                                                                                                                  |
| *desc*             | `Column = Column.desc`                                      | sort descending                                                                                                                   |
| *desc_nulls_first* | `Column = Column.desc_nulls_first`                          | sort descending with nulls appearing before the values                                                                            |
| *desc_nulls_last*  | `Column = Column.desc_nulls_last`                           | sort descending with nulls appearing after the values                                                                             |
| *asc*              | `Column = Column.asc`                                       | sort ascending                                                                                                                    |
| *asc_nulls_first*  | `Column = Column.asc_nulls_first`                           | sort ascending with nulls appearing before the values                                                                             |
| *asc_nulls_last*   | `Column = Column.asc_nulls_last`                            | sort ascending with nulls appearing after the values                                                                              |
| *explain*          | `Column = Column.explain(Boolean)`                          | prints expression for debug, parameter decides if the print should be extended by additional info                                 |

#### API for grouped column data

| **name**              | **psuedocode**                                                                                | **comments**                                                                                                                                                                                                          |
|-----------------------|-----------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *count*               | `DateFrame = RelationalGroupedDataset.count()`                                                     | count the number of rows for each group |
| *mean*                | `DateFrame = RelationalGroupedDataset.mean(colNames: String*)`                                                     | compute average for each group for given numeric columns |
| *avg*                 | `DateFrame = RelationalGroupedDataset.avg(colNames: String*)`                                                     | works as `mean` |
| *sum*                 | `DateFrame = RelationalGroupedDataset.sum(colNames: String*)`                                                     | compute sum for each group for given numeric columns |
| *max*                 | `DateFrame = RelationalGroupedDataset.max(colNames: String*)`                                                     | compute max value for each group for given numeric columns |
| *min*                 | `DateFrame = RelationalGroupedDataset.min(colNames: String*)`                                                     | compute min value for each group for given numeric columns |
