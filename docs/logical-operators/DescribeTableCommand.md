# DescribeTableCommand Logical Command

`DescribeTableCommand` is a [LeafRunnableCommand](LeafRunnableCommand.md) (indirectly as `DescribeCommandBase`) that represents a [DescribeRelation](DescribeRelation.md) at execution (and hence [DESCRIBE TABLE](../sql/AstBuilder.md#visitDescribeRelation) SQL statement).

## Creating Instance

`DescribeTableCommand` takes the following to be created:

* <span id="table"> `TableIdentifier`
* <span id="partitionSpec"> `TablePartitionSpec`
* <span id="isExtended"> `isExtended` flag
* <span id="output"> Output [Attribute](../expressions/Attribute.md)s

`DescribeTableCommand` is created when:

* [ResolveSessionCatalog](../logical-analysis-rules/ResolveSessionCatalog.md) logical analysis rule is executed (to resolve [DescribeRelation](DescribeRelation.md) logical operator)

## <span id="describeFormattedTableInfo"> Detailed Table Information

```scala
describeFormattedTableInfo(
  table: CatalogTable,
  buffer: ArrayBuffer[Row]): Unit
```

`describeFormattedTableInfo`...FIXME

---

`describeFormattedTableInfo` is used when:

* `DescribeTableCommand` is [executed](#run) (with a [table](../SessionCatalog.md#isTemporaryTable) and [isExtended](#isExtended) enabled)

<!---
## Review Me

[[output]]
`DescribeTableCommand` uses the following <<Command.md#output, output schema>>:

* `col_name` as the name of the column
* `data_type` as the data type of the column
* `comment` as the comment of the column

[source, scala]
----
spark.range(1).createOrReplaceTempView("demo")

// DESC view
scala> sql("DESC EXTENDED demo").show
+--------+---------+-------+
|col_name|data_type|comment|
+--------+---------+-------+
|      id|   bigint|   null|
+--------+---------+-------+

// DESC table
// Make the demo reproducible
spark.sharedState.externalCatalog.dropTable(
  db = "default",
  table = "bucketed",
  ignoreIfNotExists = true,
  purge = true)
spark.range(10).write.bucketBy(5, "id").saveAsTable("bucketed")
assert(spark.catalog.tableExists("bucketed"))

// EXTENDED to include Detailed Table Information
// Note no partitions used
// Could also be FORMATTED
scala> sql("DESC EXTENDED bucketed").show(numRows = 50, truncate = false)
+----------------------------+-----------------------------------------------------------------------------+-------+
|col_name                    |data_type                                                                    |comment|
+----------------------------+-----------------------------------------------------------------------------+-------+
|id                          |bigint                                                                       |null   |
|                            |                                                                             |       |
|# Detailed Table Information|                                                                             |       |
|Database                    |default                                                                      |       |
|Table                       |bucketed                                                                     |       |
|Owner                       |jacek                                                                        |       |
|Created Time                |Sun Sep 30 20:57:22 CEST 2018                                                |       |
|Last Access                 |Thu Jan 01 01:00:00 CET 1970                                                 |       |
|Created By                  |Spark 2.3.1                                                                  |       |
|Type                        |MANAGED                                                                      |       |
|Provider                    |parquet                                                                      |       |
|Num Buckets                 |5                                                                            |       |
|Bucket Columns              |[`id`]                                                                       |       |
|Sort Columns                |[]                                                                           |       |
|Table Properties            |[transient_lastDdlTime=1538333842]                                           |       |
|Statistics                  |3740 bytes                                                                   |       |
|Location                    |file:/Users/jacek/dev/apps/spark-2.3.1-bin-hadoop2.7/spark-warehouse/bucketed|       |
|Serde Library               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                           |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                             |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                    |       |
|Storage Properties          |[serialization.format=1]                                                     |       |
+----------------------------+-----------------------------------------------------------------------------+-------+

// Make the demo reproducible
val tableName = "partitioned_bucketed_sorted"
val partCol = "part"
spark.sharedState.externalCatalog.dropTable(
  db = "default",
  table = tableName,
  ignoreIfNotExists = true,
  purge = true)
spark.range(10)
  .withColumn("part", $"id" % 2) // extra column for partitions
  .write
  .partitionBy(partCol)
  .bucketBy(5, "id")
  .sortBy("id")
  .saveAsTable(tableName)
assert(spark.catalog.tableExists(tableName))
scala> sql(s"DESC EXTENDED $tableName").show(numRows = 50, truncate = false)
+----------------------------+------------------------------------------------------------------------------------------------+-------+
|col_name                    |data_type                                                                                       |comment|
+----------------------------+------------------------------------------------------------------------------------------------+-------+
|id                          |bigint                                                                                          |null   |
|part                        |bigint                                                                                          |null   |
|# Partition Information     |                                                                                                |       |
|# col_name                  |data_type                                                                                       |comment|
|part                        |bigint                                                                                          |null   |
|                            |                                                                                                |       |
|# Detailed Table Information|                                                                                                |       |
|Database                    |default                                                                                         |       |
|Table                       |partitioned_bucketed_sorted                                                                     |       |
|Owner                       |jacek                                                                                           |       |
|Created Time                |Mon Oct 01 10:05:32 CEST 2018                                                                   |       |
|Last Access                 |Thu Jan 01 01:00:00 CET 1970                                                                    |       |
|Created By                  |Spark 2.3.1                                                                                     |       |
|Type                        |MANAGED                                                                                         |       |
|Provider                    |parquet                                                                                         |       |
|Num Buckets                 |5                                                                                               |       |
|Bucket Columns              |[`id`]                                                                                          |       |
|Sort Columns                |[`id`]                                                                                          |       |
|Table Properties            |[transient_lastDdlTime=1538381132]                                                              |       |
|Location                    |file:/Users/jacek/dev/apps/spark-2.3.1-bin-hadoop2.7/spark-warehouse/partitioned_bucketed_sorted|       |
|Serde Library               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                                              |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                                                |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                                       |       |
|Storage Properties          |[serialization.format=1]                                                                        |       |
|Partition Provider          |Catalog                                                                                         |       |
+----------------------------+------------------------------------------------------------------------------------------------+-------+

scala> sql(s"DESCRIBE EXTENDED $tableName PARTITION ($partCol=1)").show(numRows = 50, truncate = false)
+--------------------------------+-------------------------------------------------------------------------------------------------------------------------------+-------+
|col_name                        |data_type                                                                                                                      |comment|
+--------------------------------+-------------------------------------------------------------------------------------------------------------------------------+-------+
|id                              |bigint                                                                                                                         |null   |
|part                            |bigint                                                                                                                         |null   |
|# Partition Information         |                                                                                                                               |       |
|# col_name                      |data_type                                                                                                                      |comment|
|part                            |bigint                                                                                                                         |null   |
|                                |                                                                                                                               |       |
|# Detailed Partition Information|                                                                                                                               |       |
|Database                        |default                                                                                                                        |       |
|Table                           |partitioned_bucketed_sorted                                                                                                    |       |
|Partition Values                |[part=1]                                                                                                                       |       |
|Location                        |file:/Users/jacek/dev/apps/spark-2.3.1-bin-hadoop2.7/spark-warehouse/partitioned_bucketed_sorted/part=1                        |       |
|Serde Library                   |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                                                                             |       |
|InputFormat                     |org.apache.hadoop.mapred.SequenceFileInputFormat                                                                               |       |
|OutputFormat                    |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                                                                      |       |
|Storage Properties              |[path=file:/Users/jacek/dev/apps/spark-2.3.1-bin-hadoop2.7/spark-warehouse/partitioned_bucketed_sorted, serialization.format=1]|       |
|Partition Parameters            |{totalSize=1870, numFiles=5, transient_lastDdlTime=1538381329}                                                                 |       |
|Partition Statistics            |1870 bytes                                                                                                                     |       |
|                                |                                                                                                                               |       |
|# Storage Information           |                                                                                                                               |       |
|Num Buckets                     |5                                                                                                                              |       |
|Bucket Columns                  |[`id`]                                                                                                                         |       |
|Sort Columns                    |[`id`]                                                                                                                         |       |
|Location                        |file:/Users/jacek/dev/apps/spark-2.3.1-bin-hadoop2.7/spark-warehouse/partitioned_bucketed_sorted                               |       |
|Serde Library                   |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                                                                             |       |
|InputFormat                     |org.apache.hadoop.mapred.SequenceFileInputFormat                                                                               |       |
|OutputFormat                    |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                                                                      |       |
|Storage Properties              |[serialization.format=1]                                                                                                       |       |
+--------------------------------+-------------------------------------------------------------------------------------------------------------------------------+-------+
----

=== [[run]] Executing Logical Command -- `run` Method

[source, scala]
----
run(sparkSession: SparkSession): Seq[Row]
----

NOTE: `run` is part of the <<RunnableCommand.md#run, RunnableCommand Contract>> to execute (run) a logical command.

`run` uses the [SessionCatalog](../SessionCatalog.md) (of the <<SparkSession.md#sessionState, SessionState>> of the input <<SparkSession.md#, SparkSession>>) and branches off per the type of the table to display.

For a [temporary view](../SessionCatalog.md#isTemporaryTable), `run` requests the `SessionCatalog` to [lookupRelation](../SessionCatalog.md#lookupRelation) to access the <<catalyst/QueryPlan.md#schema, schema>> and <<describeSchema, describeSchema>>.

For all other table types, `run` does the following:

. Requests the `SessionCatalog` to [retrieve the table metadata from the external catalog (metastore)](../SessionCatalog.md#getTableMetadata) (as a [CatalogTable](../CatalogTable.md)) and <<describeSchema, describeSchema>> (with the [schema](../CatalogTable.md#schema))

. <<describePartitionInfo, describePartitionInfo>>

. <<describeDetailedPartitionInfo, describeDetailedPartitionInfo>> if the <<partitionSpec, TablePartitionSpec>> is available or <<describeFormattedTableInfo, describeFormattedTableInfo>> when <<isExtended, isExtended>> flag is on

=== [[describeFormattedDetailedPartitionInfo]] Describing Detailed Partition and Storage Information -- `describeFormattedDetailedPartitionInfo` Internal Method

[source, scala]
----
describeFormattedDetailedPartitionInfo(
  tableIdentifier: TableIdentifier,
  table: CatalogTable,
  partition: CatalogTablePartition,
  buffer: ArrayBuffer[Row]): Unit
----

`describeFormattedDetailedPartitionInfo` simply adds the following entries (rows) to the input mutable buffer:

. A new line

. *# Detailed Partition Information*

. *Database* with the [database](../CatalogTable.md#database) of the given `table`

. *Table* with the table of the given `tableIdentifier`

. [Partition specification](../CatalogTablePartition.md#toLinkedHashMap) (of the [CatalogTablePartition](../CatalogTablePartition.md))

. A new line

. *# Storage Information*

. [Bucketing specification](../BucketSpec.md#toLinkedHashMap) of the [table](../CatalogTable.md#bucketSpec) (if defined)

. [Storage specification](../CatalogStorageFormat.md#toLinkedHashMap) of the [table](../CatalogTable.md#storage)

`describeFormattedDetailedPartitionInfo` is used when:

* `DescribeTableCommand` is requested to [describeDetailedPartitionInfo](#describeDetailedPartitionInfo) with a non-empty [partitionSpec](#partitionSpec) and the [isExtended](#isExtended) flag on.
-->
