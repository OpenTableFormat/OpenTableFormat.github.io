---
title: "What is Open Table Format"
date: 2022-11-20T11:10:36+08:00
draft: false
language: en
summary: OpenTableFormat is to object storage what constellation is to stars.
featured_image: ../assets/images/featured/interior-wooden-frame-mockup-shelf-blue-wall-3d-rendering.jpeg
author: OpenTableFormat
authorimage: ../assets/images/global/author.png
tags:
 - iceberg
 - hudi
 - deltalake
---

### What's a table format?

A good way to define a table format is a way to organize a dataset's files to present them as a single "table". The primary goal of a table format is to provide the abstraction of a table to people and tools and allow them to efficiently interact with that table's underlying data.

Table formats are nothing new — they've been around since System R, Multics, and Oracle first implemented Edgar Codd's relational model, although "table format" wasn't the term used at the time. All interaction with the underlying data, like writing or reading it, was handled by the database's storage engine. No other engine could interact with the files directly without corrupting the system.

Apache Hive is one of the earliest and most used table formats in big data world, it stores the metadata information(like schema) of the datasets in HMS(Hive Metastore, backed by a central realtional DB), and provides SQL programming model to query the datasets as a table, the Hive table format has been the de facto standard ever since, but it has the following drawbacks when used at larger scale:

- Changes to the data are inefficent (need to copy the whole partion for changing small amount of data)
- No way to safely change data in multiple partitions as part of one operation
- Multiple jobs modifying the same dataset isn’t a safe operation
- All of the directory listings needed for large tables take a long time
- Users have to know the physical layout of the table
- Hive table statistics are usually stale
- The filesystem layout has poor performance on cloud object storage(same prefix hit the same object storage node)
- Metastore is a scale bottleneck

### What is open table format?

To address those problems, communities started creating new open table formats:

- Apache Iceberg — Netflix
- Apache Hudi — Uber
- Delta table — Databricks

Though organized in different layouts, they all provide [Lakehouse](https://www.cidrdb.org/cidr2021/papers/cidr2021_paper17.pdf) features:

- Transactions (ACID)
- Unified Batch & streaming
- Schema enforcement, evolution & versioning
- Metadata scaling
- Time Travel
- Concurrent read & write
- Independent consumption from storage


#### References:

[1] [Apache Iceberg: An Architectural Look Under the Covers](https://www.dremio.com/resources/guides/apache-iceberg-an-architectural-look-under-the-covers/)<br>
[2] [Open Table Formats — Delta, Iceberg & Hudi](https://medium.com/geekculture/open-table-formats-delta-iceberg-hudi-732f682ec0bb)
