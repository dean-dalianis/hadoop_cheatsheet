# Hadoop Cheatsheet

Quick reference guide to key components in the Hadoop ecosystem based on the udemy
course [The Ultimate Hands-On Hadoop - Tame your Big Data!](https://www.udemy.com/the-ultimate-hands-on-hadoop-tame-your-big-data/learn/v4/overview).
We'll start by describing the components of the Hadoop and link to different guides for each component.

## Hadoop Core

Hadoop Core is the foundational framework of the Apache Hadoop ecosystem, providing essential components for distributed
data processing and storage.

### HDFS (Hadoop Distributed File System)

- Storage system for big data across a cluster.
- Provides reliable, scalable, and distributed data storage.
- Replicates data for fault tolerance.
- Optimized for batch processing workloads.

### YARN (Yet Another Resource Negotiator)

- Manages resources in a Hadoop cluster.
- Handles allocation of nodes and computing capacity.
- Efficiently utilizes resources for applications.

### MapReduce

- Programming model for distributed data processing in Hadoop.
- Divides tasks into map and reduce phases.
- Enables parallel execution across the cluster.

## Other Components

Additional components that complement the Hadoop ecosystem can be found here.

### Pig

- High-level language for data analysis and transformation on Hadoop.
- Allows complex queries and data processing.
- Utilizes SQL-like syntax for ease of use.
- Supports data processing pipelines and custom processing logic.

### Hive

- Data warehouse infrastructure for querying structured data stored in Hadoop.
- Provides a SQL-like interface.
- Manages metadata for easy data exploration.
- Supports schema evolution and query optimization.

### Ambari

- Management and monitoring tool for Hadoop clusters.
- Offers comprehensive view and control of components and services.
- Facilitates cluster administration and monitoring.

### Mesos

- Resource management platform for Hadoop clusters.
- Efficiently allocates resources, such as nodes and computing capacity.
- Works alongside YARN or serves as an alternative resource negotiator.

### Spark

- Fast and powerful data processing engine for Hadoop.
- Supports in-memory processing, real-time streaming, and machine learning.
- Enables interactive queries and analysis.
- Provides a rich set of libraries and APIs for various data processing tasks.

### Tez

- Framework for optimizing data processing in Hadoop.
- Utilizes Directed Acyclic Graphs (DAGs) for efficient execution of complex queries.
- Often used in conjunction with Hive.

### HBase

- Distributed, column-oriented NoSQL database for Hadoop.
- Enables low-latency, random access to large volumes of structured and semi-structured data.

### Storm

- Real-time stream processing system for Hadoop.
- Handles continuous streams of data.
- Enables real-time analytics and decision-making.
- Supports fault-tolerance and complex stream processing topologies.

### Oozie

- Workflow scheduler for managing complex Hadoop jobs.
- Allows defining and executing interconnected tasks.
- Facilitates job coordination, scheduling, and monitoring.

### ZooKeeper

- Coordination service for distributed systems in Hadoop.
- Maintains shared configuration, synchronization, and naming services.
- Facilitates coordination and management of cluster components.
- Provides distributed coordination and high availability features.

## Data Ingestion

Tools for ingesting data into the Hadoop ecosystem:

### Sqoop

- Tool for transferring data between Hadoop and relational databases.
- Facilitates importing data into Hadoop or exporting data from Hadoop to external databases.

### Flume

- Distributed data collection and aggregation system for Hadoop.
- Enables reliable and scalable ingestion of streaming data into Hadoop.

### Kafka

- Distributed streaming platform for collecting, storing, and processing real-time data streams.
- Publishes data from various sources to Hadoop.

## External Data Storage

### MySQL

- Widely used open-source relational database management system.
- Can be integrated with Hadoop for external data storage and processing.

### Cassandra

- Highly scalable and fault-tolerant distributed NoSQL database.
- Suitable for storing large volumes of structured and unstructured data.

### MongoDB

- Document-oriented NoSQL database with flexible schema design.
- Provides high performance and scalability for storing and querying data.

## Query Engines

Engines for querying and analyzing data within the Hadoop ecosystem:

### Drill (Apache Drill)

- Distributed SQL query engine for analyzing data in various formats, including NoSQL databases.
- Supports querying structured and semi-structured data.

### Hue

- Web-based interface for interacting with Hadoop components, such as Hive and HBase.
- Provides a user-friendly environment for querying and data exploration.

### Phoenix (Apache Phoenix)

- SQL query engine for HBase.
- Enables executing SQL queries directly on HBase, facilitating interaction with HBase data.

### Presto

- Distributed SQL query engine for interactive analytics on large datasets.
- Provides high performance.
- Supports querying data from multiple sources.

### Zeppelin (Apache Zeppelin)

- Web-based notebook interface for data exploration and visualization.
- Supports multiple query engines.
- Provides an interactive environment for data analysis.

---

## Installation

This section covers the basic installation of the HDP Sandbox on a local machine. The HDP Sandbox is a pre-configured
virtual machine that can be used to run Hadoop on a single node. Further installation & configuration steps for
different components will be covered in their respective sections.

- VirtualBox: [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Hortonworks Sandbox VirtualBox
  Image: [Download Hortonworks Sandbox](https://www.cloudera.com/downloads/hortonworks-sandbox/hdp.html)
- Dataset: [MovieLens 100K Dataset](https://grouplens.org/datasets/movielens/100k/)
- Ambari: localhost:8080 (username: maria_dev, password: maria_dev)
- SSH: `ssh maria_dev@127.0.0.1 -p 2222`
- Admin Access to Ambari:
    - `sudo su`
    - Run `ambari-admin-password-reset` to set the admin password.

---

## Cheat Sheets by Component

- [HDFS](./HDFS.MD)
- [MapReduce](./MAP_REDUCE.md)
- [Pig](./PIG.MD)
- [Spark](./SPARK.md)
- [Relational Data Stores](./RELATIONAL_DBS.md)
- [Non-relational Data Stores](./NON_RELATIONAL_DBS.md)