---
title: "A little bit of Apache Iceberg"
date: 2022-11-21T00:00:00+08:00
draft: false
language: en
featured_image: ../assets/images/featured/iceberg-unsplash.jpeg
summary: Iceberg is a high-performance format for huge analytic tables. Iceberg brings the reliability and simplicity of SQL tables to big data, while making it possible for compute engines like Spark, Trino, Flink, Presto, Hive and Impala to safely work with the same tables, at the same time.
author: OpenTableFormat
authorimage: ../assets/images/global/author.png
categories: iceberg
tags: iceberg
---

Apache Iceberg[^iceberg] is an open table format for huge analytic datasets, it brings the reliability and simplicity of SQL tables to big data, while making it possible for compute engines like Spark, Trino, Flink, Presto, Hive and Impala to safely work with the same tables, at the same time.

A table format helps you manage, organize, and track all of the files that make up a table, this enable database-like semantics over files; you can easily get features such as ACID compliance, time travel, and schema evolution, making your files much more useful for analytical queries. Iceberg is one of many solutions to implement a table format over sets of files; with table formats the headaches of working with files can disappear. Iceberg was created by Netflix and later donated to the Apache Software Foundation. 

## What Iceberg is and what it isn't:

| ✅ What Iceberg is | ❌ What Iceberg is not |
| --------------- | ------------------- |
| A table format specification | A storage engine or execution engine |
| A set of APIs and libraries for engines to interact with tables following that specification | A service |

## Basic Concepts

![Iceberg Architecture](/images/posts/iceberg-architecture.png)

The Iceberg table’s architecture consist of three layers:

1. Iceberg catalog: The catalog is where services go to find the location of the current metadata pointer, which helps identify where to read or write data for a given table. Here is where references or pointers exist for each table that identify each table's current metadata file.
2. Metadata layer: This layer consists of three components: `metadata file`, `manifest list`, and `manifest file`.
   - The metadata file includes information about a table's schema, partition information, snapshots, and the current snapshot.
   - The manifest list contains a list of manifest files, along with information about the manifest files that make up a snapshot.
   - The manifest files track data files in addition to other details and statistics about each file.
3. Data layer: Each manifest file tracks a subset of data files, which contain details about partition membership, record count, and lower- and upper-bounds of columns.

**After INSERT some data and do a COMMIT, the following process happens:**

1. The data in the form of a Parquet file is first created
2. Then, a manifest file pointing to this data file is created (including the additional details and statistics)
3. Then, a manifest list pointing to this manifest file is created (including the additional details and statistics)
4. Then, a new metadata file is created based on the previously current metadata file with a new snapshot as well as keeping track of the previous snapshots, pointing to this manifest list (including the additional details and statistics)
5. Then, the value of the current metadata pointer for the table is atomically updated in the catalog now point to this new metadata file

During all of these steps, anyone reading the table would continue to read the first metadata file until the atomic step #5 is complete, meaning that no one using the data would ever see an inconsistent view of the table's state and contents.

**When SELECT is executed, the following process happens:**

1. The query engine goes to the Iceberg catalog
2. It then retrieves the current metadata file location entry for the table
3. It then opens this metadata file and retrieves the entry for the manifest list location for the current snapshot
4. It then opens this manifest list, retrieving the location of the manifest files
5. It then opens this manifest files, retrieving the location of data files
6. It then reads these data files

Step #3 can prune some partitions based on the partition specification, step #5 can filter some data file base on the statistics.

Another key capability the Iceberg table format enables is something called "time travel", using the following syntax to retrive the snapshot and use the snapshot to query the table futher.

```
SELECT * FROM table1 AS OF '2021-01-28 00:00:00';
```

A detailed demo can be seen [here](https://www.youtube.com/watch?v=N4gAi_zpN88&t=1480s).

## Supported Engines

Spark is currently the most well-supported compute engine for Iceberg operations, and lots of other compute engines are making efforts to support this promising table format.

| Engine | Supported Operations[^onehouse] | Description |
| ------ | -------------------- | ----------- |
| Spark  | Read + Write | Iceberg uses Apache Spark’s DataSourceV2 API[^spark-ddl] for data source and catalog implementations. |
| Flink  | Read + Write | Iceberg supports both Apache Flink's DataStream API and Table API[^flink]. |
| Hive   | Read + Write | Iceberg supports reading and writing Iceberg tables through Hive by using a StorageHandler[^hive]. |
| Trino  | Read + Write | Using Iceberg connector[^trino] to read and write the supported data file formats Avro, ORC, and Parquet, following the Iceberg specification. |
| Presto | Read + Write | Using Iceberg connector[^presto] to query and write Iceberg tables. |
| Dremio | Read + Write | Dremio integrated support for Iceberg tables to leverage powerful SQL database-like functionality through industry standard methods for data lakes[^dremio]. |
| StarRocks | Read | From v2.1.0, StarRocks provides external tables to query data in Apache Iceberg[^starrocks]. |
| Athena | Read + Write | AWS Athena supports read, time travel, write, and DDL queries for Apache Iceberg tables[^athena] that use the Apache Parquet format for data and the AWS Glue catalog for their metastore. |
| Impala | Read + Write | Currently only Iceberg V1 DML operations are allowed[^impala_iceberg], i.e. INSERT INTO /INSERT OVERWRITE. Iceberg V2 operations like row-level modifications (UPDATE, DELETE) are not supported yet. |
| Doris  | Read         | Iceberg External Table of Doris[^iceberg-of-doris] provides Doris with the ability to access Iceberg external tables directly. |
| Snowflake | Read + Write | With Iceberg Tables, our goal is to get as close to Snowflake native tables as possible, without breaking open source compatibility. |
|

Snowflake supports `Iceberg External Tables` as well as `Iceberg Tables`[^snowflake], you can think of `Iceberg Tables` as Snowflake tables that use open formats and customer-supplied cloud storage. Snowflake provides a flow chart on how to choose different table types:

![Snowflake Table Types](/images/posts/snowflake_table_types.png)

## Involved Vendors

- Cloudera
- Dremio
- Snowflake
- Starburst
- Tabular
- Nexflex

![Iceberg Ecosystem](/images/posts/apache-iceberg-ecosystem.png)

## Diversity of Contributions by Company (Until April 2022)

![Diversity of Contributions by Company](/images/posts/iceberg_contributors.png)


### References:

[1] [Iceberg Table Spec](https://iceberg.apache.org/spec/)<br>
[2] [Apache Iceberg: An Architectural Look Under the Covers](https://www.dremio.com/resources/guides/apache-iceberg-an-architectural-look-under-the-covers) by Jason Hughes<br>
[3] [Expanding the Data Cloud with Apache Iceberg](https://www.snowflake.com/blog/expanding-the-data-cloud-with-apache-iceberg/) by James Malone, Jan 2022<br>
[4] [5 Compelling Reasons to Choose Apache Iceberg](https://www.snowflake.com/blog/5-reasons-apache-iceberg/) by James Malone, Jul 2022<br>
[5] [Iceberg Tables: Powering Open Standards with Snowflake Innovations](https://www.snowflake.com/blog/iceberg-tables-powering-open-standards-with-snowflake-innovations/) by James Malone, Aug 2022<br>
[6] [What Are Iceberg Tables In Snowflake? 6 Minute Demo](https://www.youtube.com/watch?v=Kz5cWY_vRwU)<br>
[7] [Iceberg: a fast table format for S3](https://www.youtube.com/watch?v=nWwQMlrjhy0) by Ryan Blue, Jun 2018<br>
[8] [Apache Iceberg - A Table Format for Huge Analytic Datasets](https://www.youtube.com/watch?v=mf8Hb0coI6o) by Ryan Blue, Nov 2019<br>
[9] [Data architecture in 2022](https://www.youtube.com/watch?v=1oXmBbB77ak) by Ryan Blue, May 2022<br>
[10] [Apache Iceberg 101](https://www.youtube.com/playlist?list=PL-gIUf9e9CCtGr_zYdWieJhiqBG_5qSPa) from Dremio, Sep 2022<br>
[11] [Using Apache Iceberg for Multi-Function Analytics in the Cloud](https://www.youtube.com/watch?v=gCWG6TQZKrQ) by Bill Zhang, Mar 2022<br>
[12] [Apache Iceberg - An Architectural Look Under the Covers](https://www.youtube.com/watch?v=N4gAi_zpN88&t=386s) by Jason Hughes, Nov 2021<br>


[^iceberg]: [Apache Iceberg](https://github.com/apache/iceberg)
[^spark-ddl]: [Spark DDL](https://iceberg.apache.org/docs/latest/spark-ddl/)
[^flink]: [Flink](https://iceberg.apache.org/docs/latest/flink/)
[^hive]: [Hive](https://iceberg.apache.org/docs/latest/hive/)
[^trino]: [Trino Iceberg connector](https://trino.io/docs/current/connector/iceberg.html)
[^presto]: [Presto Iceberg Connector](https://prestodb.io/docs/current/connector/iceberg.html)
[^dremio]: [How Dremio Uses Iceberg Tables](https://docs.dremio.com/software/data-formats/apache-iceberg/)
[^onehouse]: [Lakehouse Feature Comparison](https://www.onehouse.ai/blog/apache-hudi-vs-delta-lake-vs-apache-iceberg-lakehouse-feature-comparison)
[^starrocks]: [Starrocks External tables](https://docs.starrocks.io/en-us/latest/using_starrocks/External_table#apache-iceberg-external-table)
[^athena]: [Athena using Iceberg tables](https://docs.aws.amazon.com/athena/latest/ug/querying-iceberg.html)
[^impala_iceberg]: [Using Impala with Iceberg Tables](https://impala.apache.org/docs/build/html/topics/impala_iceberg.html)
[^iceberg-of-doris]: [Doris On Iceberg](https://doris.apache.org/docs/ecosystem/external-table/iceberg-of-doris/)
[^snowflake]: [Snowflake Apache Iceberg Support](https://docs.snowflake.com/en/LIMITEDACCESS/iceberg.html)
