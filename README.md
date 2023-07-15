# Hadoop Cheatsheet

Quick reference guide to key components in the Hadoop ecosystem.

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

## HDFS (Hadoop Distributed File System):

- Handles big files:
    - Breaks files into blocks (default block size is 128 MB).
    - Stores multiple copies of each block across several computers for fault tolerance.

- Architecture:
    - Single Name Node:
        - Keeps track of block locations and metadata.
    - Multiple Data Nodes:
        - Store the actual blocks of data.
        - Communicate with each other to maintain block replication and consistency.

- Reading a file:
    1. Client contacts the Name Node to retrieve the file's block locations.
    2. Client retrieves the file directly from the respective Data Nodes.

- Writing a file:
    1. Client communicates with the Name Node:
        - Name Node creates an entry for the new file.
    2. Client contacts a single Data Node.
    3. Data Nodes coordinate with each other to write the file in parallel.
    4. Once the data is stored, Data Nodes report block locations to the client.
    5. Client updates the Name Node with the block locations.

- Name Node resilience:
    - Backup metadata:
        - Metadata is written to local disk and a separate NFS (Network File System) for recovery purposes.
    - Secondary Name Node:
        - Maintains a merged copy of the edit log from the primary Name Node.
    - HDFS Federation:
        - Multiple Name Nodes manage separate namespace volumes.
        - If one Name Node fails, only a portion of the data is affected.
    - HDFS High Availability:
        - Hot standby Name Node using shared edit log stored in a different file system (not HDFS).
        - ZooKeeper is used to track the active Name Node.
        - Clients first consult ZooKeeper to determine the active Name Node.
        - Requires complex configuration and ensures only one Name Node runs at a time.

- How to use HDFS:
    - UI (e.g., Ambari):
        - Utilize the user interface provided by tools like Ambari to interact with HDFS.
        - The UI offers a graphical representation of the file system, allowing users to browse, upload, and download
          files.
    - Command-Line Interface:
        - Access HDFS using the command-line interface (CLI) provided by Hadoop.
        - Use commands such as `hdfs dfs` to perform various operations, such as listing files, copying files, or
          creating directories.
    - HTTP / HDFS Proxies:
        - Access HDFS using the HTTP protocol through HDFS proxies.
        - The HDFS proxies provide REST APIs that can be utilized for file operations, such as reading, writing, and
          listing files.
    - Java Interface:
        - Utilize the Java API provided by Hadoop to interact with HDFS programmatically.
        - Develop custom Java applications to read, write, and manipulate files stored in HDFS.
    - NFS Gateway:
        - Mount an HDFS cluster as another server using the NFS gateway.
        - This allows HDFS to be accessed as a regular file system through the standard file system interfaces.
        - Users can interact with HDFS by simply accessing the mounted directory, just like any other local file system.

---

## Installation

- VirtualBox: [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Hortonworks Sandbox VirtualBox
  Image: [Download Hortonworks Sandbox](https://www.cloudera.com/downloads/hortonworks-sandbox/hdp.html)
- Dataset: [MovieLens 100K Dataset](https://grouplens.org/datasets/movielens/100k/)
- Ambari: localhost:8080 (username: maria_dev, password: maria_dev)
- SSH: `ssh maria_dev@127.0.0.1 -p 2222`
- Admin Access to Ambari:
    - `sudo su`
    - Run `ambari-admin-password-admin` to set the admin password

---

## Basic HDFS Terminal Commands

Use the following HDFS terminal commands to interact with the Hadoop Distributed File System:

- `hadoop fs --<command>`: Execute HDFS command
- `hadoop fs -mkdir ml-100k`: Create a directory in HDFS
- `hadoop fs -ls`: List directory contents in HDFS
- `hadoop fs -copyFromLocal <local_file> <hdfs_path>`: Copy a file from the local filesystem to HDFS
- `hadoop fs -rm <hdfs_path>`: Remove a file from HDFS
- `hadoop fs -rmdir <hdfs_path>`: Remove a directory from HDFS
- `hadoop fs`: Show help and available commands

---

## MapReduce

MapReduce is a data processing paradigm in Hadoop that operates at a conceptual level. It distributes data processing
across the cluster and consists of two main stages: Map and Reduce. Here are the key points:

- Distributes data processing on the cluster, enabling parallel execution.
- Divides data into partitions that are:
    - Mapped (transformed) by a defined mapper function:
        - Extracts and organizes data, associating it with a certain key value.
    - Reduced (aggregated) by a defined reducer function:
        - Aggregates the data based on the keys.
- Resilient to failure, allowing for fault tolerance in distributed environments.

### MapReduce Conceptual Example: MovieLens Dataset

Let's consider an example of counting how many movies each user rated in the MovieLens dataset. Here's how MapReduce
would work for this scenario:

- The MAPPER converts raw data into key/value pairs.
    - **Input Data**:
      ```
      User ID | Movie ID | ...
      196       242
      186       302
      196       377
      244       51
      166       346
      186       474
      186       265
      ```
    - The **KEY** is the user ID, and the **VALUE** is the movie ID.

- The REDUCER processes each key's values to obtain the final result.

    - After the MAPPER stage, the intermediate key/value pairs would look like this:
      ```
      {196: 242, 186: 302, 196: 377, 244: 51, 166: 346, 186: 474, 186: 265}
      ```

    - The **MapReduce Magic "Shuffle & Sort"** step groups the values by key:
      ```
      {166: 346, 186: 302, 474, 265, 196: 242, 377, 244: 51}
      ```

    - Finally, the **REDUCER** counts the number of movies for each user ID:
      ```
      {166: 1, 186: 3, 196: 2, 244: 1}
      ```

### MapReduce hands-on Example: Movie Ratings by Rating Score

To solve the problem of analyzing movie ratings using MapReduce, we can follow these steps:

- **Making it a MapReduce problem**:
    - We'll convert the data into key-value pairs and utilize the MapReduce paradigm for processing.

- **Solution as a Python MRJob**:

    ```python
    from mrjob.job import MRJob
    from mrjob.step import MRStep
    
    class RatingsBreakdown(MRJob):
        def steps(self):
            return [
                MRStep(mapper=self.mapper_get_ratings,
                        reducer=self.reducer_count_ratings)
            ]
    
        def mapper_get_ratings(self, _, line):
            (userID, movieID, rating, timestamp) = line.split('\t')
            yield rating, 1
    
        def reducer_count_ratings(self, key, values):
            yield key, sum(values)
    
    if __name__ == '__main__':
        RatingsBreakdown.run()
    ```

### Installing MRJob in HDP 2.6.5

To install MRJob in HDP 2.6.5, follow these steps:

```bash
yum-config-manager --save --setopt=HDP-SOLR-2.6-100.skip_if_unavailable=true
yum install https://repo.ius.io/ius-release-el7.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install python-pip
pip install pathlib
pip install mrjob==0.7.4
pip install PyYAML==5.4.1
yum install nano
```

### Running MRJob Locally

To run MRJob locally, use the following command:

```bash
python RatingsBreakdown.py u.data
```

### Running MRJob with Hadoop

To run MRJob with Hadoop, use the following command:

```bash
python RatingsBreakdown.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoops-streaming.jar u.data
```

## MapReduce hands-on Exercise: Rank Movies by Their Popularity

To rank movies by their popularity, we can modify the previous MRJob as follows:

```python
from mrjob.job import MRJob
from mrjob.step import MRStep

class MovieViewsBreakdown(MRJob):
    def steps(self):
        return [
            MRStep(mapper=self.mapper_get_ratings,
                    reducer=self.reducer_count_ratings),
            MRStep(reducer=self.reducer_sorted_output)
        ]

    def mapper_get_ratings(self, _, line):
        (userID, movieID, rating, timestamp) = line.split('\t')
        yield movieID, 1

    def reducer_count_ratings(self, key, values):
        yield str(sum(values)).zfill(5), key

    def reducer_sorted_output(self, key, values):
        for movie in movies:
            yield movie, count

if __name__ == '__main__':
    MovieViewsBreakdown.run()
```

---

## Pig

Pig is a platform that allows you to analyze large datasets in a high-level scripting language called Pig Latin. It
provides SQL-like syntax for the map and reduce steps and is highly extensible. Pig runs on top of Hadoop and utilizes
its underlying components such as MapReduce, Tez, YARN, and HDFS.

### TEZ

Tez is a more efficient way of organizing jobs than MapReduce. It can be approximately 10 times faster than MapReduce
for certain workloads. You can use Tez in Ambari by selecting the 'Execute on Tez' option when running a Pig script.

### Running Pig

There are multiple ways to run Pig scripts:

- **Grunt**: Running scripts one line at a time.
- **Script**: Running commands through a script file.
- **Ambari**: Utilizing the user interface provided by Ambari to execute Pig scripts.

### Pig hands-on Example: Find the Oldest Movie with a 5-Star Rating

To find the oldest movie with a 5-star rating, you can follow these Pig Latin steps:

```pig
ratings = LOAD '/user/maria_dev/ml-100k/u.data' 
          AS (userID:int, movieID:int, rating:int, ratingTime:int);
          
metadata = LOAD '/user/maria_dev/ml-100k/u.item' 
           USING PigStorage('|') 
           AS (movieID:int, movieTitle:chararray, releaseDate:chararray, videoRelease:chararray, imdbLink:chararray);

nameLookup = FOREACH metadata 
             GENERATE movieID, movieTitle, ToUnixTime(ToDate(releaseDate, 'dd-MMM-yyyy')) AS releaseTime;

ratingsByMovie = GROUP ratings BY movieID;

avgRatings = FOREACH ratingsByMovie 
             GENERATE group AS movieID, AVG(ratings.rating) AS avgRating;

filterStarMovies = FILTER avgRatings BY avgRating > 4.0;

fiveStarsWithData = JOIN filterStarMovies BY movieID, nameLookup BY movieID;

oldestFiveStarMovies = ORDER fiveStarsWithData BY nameLookup::releaseTime;

DUMP oldestFiveStarMovies;
```

### Pig Latin Commands and Functions

- **Basic Commands**:
    - `LOAD`: Loads data into Pig from a specified location.
    - `STORE`: Stores the output of a Pig script to a specified location.
    - `DUMP`: Displays the contents of a relation.
    - `FILTER`: Filters tuples based on a condition.
    - `DISTINCT`: Removes duplicate tuples from a relation.
    - `FOREACH/GENERATE`: Generates new fields or transformations on existing fields.
    - `MAPREDUCE`: Executes a custom MapReduce job.
    - `STREAM`: Invokes an external program or script on each tuple.
    - `SAMPLE`: Randomly samples a fraction of the data.
    - `JOIN`: Joins two or more relations based on common fields.
    - `COGROUP`: Groups data from multiple relations based on common fields.
    - `GROUP`: Groups data within a relation based on specified fields.
    - `CROSS`: Produces the cross product of two or more relations.
    - `CUBE`: Generates all possible combinations of grouping sets.
    - `ORDER`: Sorts the data within a relation.
    - `RANK`: Assigns a rank to each tuple based on a specified field.
    - `LIMIT`: Limits the number of tuples in the output.
    - `UNION`: Combines multiple relations into a single relation.
    - `SPLIT`: Splits a relation into multiple relations based on a condition.

- **Diagnostics**:
    - `DESCRIBE`: Provides information about the schema of a relation.
    - `EXPLAIN`: Displays the logical, physical, and map-reduce execution plans.
    - `ILLUSTRATE`: Visualizes the data flow in a Pig script.

- **User-Defined Functions (UDFs)**:
    - `REGISTER`: Registers a user-defined function (UDF) written in Java, Python, or other languages.
    - `DEFINE`: Defines a Pig UDF or streaming command.
    - `IMPORT`: Imports a set of user-defined functions from a given namespace.

- **Other Functions**:
    - `AVG`: Calculates the average value of a field within a relation.
    - `CONCAT`: Concatenates multiple strings or fields.
    - `COUNT`: Counts the number of tuples in a relation or the number of elements in a bag.
    - `MAX`: Finds the maximum value of a field within a relation.
    - `MIN`: Finds the minimum value of a field within a relation.
    - `SIZE`: Returns the size of a bag or a tuple.
    - `SUM`: Calculates the sum of a field within a relation.
    - `PigStorage`: Loads or stores data using a specified delimiter.
    - `TextLoader`: Loads data from text files.
    - `JsonLoader`: Loads data from JSON files.
    - `AvroStorage`: Loads or stores Avro data.
    - `ParquetLoader`: Loads Parquet data.
    - `OrcStorage`: Loads or stores ORC (Optimized Row Columnar) data.
    - `HBaseStorage`: Loads or stores data from/to Apache HBase.

### Pig hands-on Exercise: Most-rated one-star movie

Find the most-rated one-star movie (less than 2.0 rating, sort by total number of ratings)

```pig
  ratings = LOAD '/user/maria_dev/ml-100k/u.data' 
            AS (userID:int, movieID:int, rating:int, ratingTime:int);
            
  metadata = LOAD '/user/maria_dev/ml-100k/u.item' 
             USING PigStorage('|') 
             AS (movieID:int, movieTitle:chararray, releaseDate:chararray, videoRelease:chararray, imdbLink:chararray);
             
  nameLookup = FOREACH metadata GENERATE movieID, movieTitle;
               
  ratingsByMovie = GROUP ratings BY movieID;
  
  avgRatingsWithCount = FOREACH ratingsByMovie 
                        GENERATE group AS movieID, AVG(ratings.rating) AS avgRating, COUNT(ratings.rating) AS ratingsCount;
                        
  twoStarMovies = FILTER avgRatingsWithCount BY avgRating < 2.0;
  
  twoStarMoviesWithData = JOIN twoStarMovies BY movieID, nameLookup BY movieID;
  
  mostOneStarRatedMovies = ORDER twoStarMoviesWithData BY ratingsCount DESC;
  
  DUMP mostOneStarRatedMovies;
```

---

## Spark Core

- A fast and general engine for large-scale data processing
- It's scalable
    - Driver Program (Spark Context) -> Cluster Manager (Spark, MESOS, YARN) -> Executors (Cache, Tasks)
- It's fast
    - ~100x faster than Hadoop MapReduce in memory or ~10x faster on disk
    - DAG (Directed Acyclic Graph) engine optimizes workflows
- Code in Java, Scala, Python
- Built around Resilient Distributed Datasets (RDDs)
- Spark 2.0 introduced Datasets on top of RDDs
- Written in Scala
- We'll use Python but for production, Scala is recommended (usage is similar)

### Components and Libraries (apart from Spark Core)

- Spark Streaming (real-time processing)
- Spark SQL (SQL and structured data processing)
- MLlib (machine learning)
- GraphX (graph processing)

### Resilient Distributed Datasets (RDDs)

- RDDs are the core abstraction in Spark
- Easily distributed across a cluster
- Fault-tolerant
- Looks like a dataset to the user

#### SparkContext

- Created by the driver program
- Makes RDDs resilient and distributed
- Creates RDDs
- Spark shell automatically creates a SparkContext ("sc")

#### Creating RDDs

- Create from a collection
    - `nums = parallelize([1, 2, 3, 4])`
- Create from a file
    - `sc.textFile("file.txt")` or `sc.wholeTextFiles("dir")` (e.g. s3n://, hdfs://)
- Create from HIVE context
    ```python
    hiveCtx = HiveContext(sc)
    rows = hiveCtx.sql("SELECT name, age FROM users")
    ```
- Any database
    - JDBC
    - Cassandra
    - HBase
    - Elasticsearch
    - JSON, CSV, sequence files, object files, various compressed formats, etc.

#### Transforming RDDs

- map: apply a function to each item in the RDD
- flatMap: each input item can be mapped to 0 or more output items
- filter: return a new RDD with a subset of items in the file
- distinct: return a new RDD with distinct items from the original RDD
- sample: create a smaller RDD from a larger RDD
- union, intersection, subtract, cartesian: combine RDDs

**RDD Example - map:**

```python
rdd = sc.parallelize([1, 2, 3, 4])
squareRDD = rdd.map(lambda x: x * x)
```

This yields a new RDD with the following elements: 1, 4, 9, 16

*Lambda explanation:*

```python
rdd.map(lambda x: x * x)

# is equivalent to

def square(x):
    return x * x

rdd.map(square)
```

#### RDD Actions

Take one RDD and turn it into another

- collect: return all items in the RDD to the driver program
- count: return the number of items in the RDD
- countByValue: return the count of each unique value in the RDD as a dictionary
- take: return the first n items of the RDD
- top: return the top n items
- reduce: aggregate the elements of the RDD using a function
- ...

#### Lazy Evaluation

- Nothing is computed until an action is called
    - Transformations are not executed until an action is called