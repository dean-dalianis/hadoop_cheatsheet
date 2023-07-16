# Non-relation Data Stores

- Why NoSQL?
    - Random Access to Planet-Size Data
    - Scales horizontally (add more machines)

- Scaling MySQL requires extreme measures
    - Denormalization: store redundant data
    - Caching: store data in memory
    - Master/Slave: read from slaves, write to master
    - Sharding: split data across multiple machines
    - Materialized Views: pre-compute results
    - Removing stored procedures

- Do you really need SQL?
    - Your high-transaction queries are probably simple once denormalized
    - A simple get/put API is often sufficient
    - Looking up values by key is simple, fast, and scalable

- Use the right tool for the job
    - For analytics: Hive, Pig, Spark, etc.
    - Export data to a relational database for OLTP
    - If you have giant scale data - export to a non-relational database

- Sample architecture
  ```
  Customer -> Internet -> Web Servers -> MongoDB <- Spark Streaming | Hadoop YARN / HDFS <- Data source(s)
  ```

## HBase

- Non-relational database, scalable, built ON HDFS
- Based on Google's BigTable
- CRUD
    - Create
    - Read
    - Update
    - Delete
- There is no query language, only CRUD APIs

### HBase data model

- Fast access to any given ROW
- A ROW is referenced by a unique KEY
- Each ROW has a small number of COLUMN FAMILIES
- A COLUMN FAMILY may contain an arbitrary number of COLUMNS
- You can have a very large number of COLUMNS per COLUMN FAMILY
- Each CELL can have many VERSIONS with given timestamps
- Sparse data is OK - missing columns in a row take no space

### HBase real-world example

Track all the links that connect to a given URL

#### Schema

Key: com.cnn.www (reverse domain name - keys are stored lexicographically, and we want to store all cnn.com links
together)

- COLUMN FAMILY: Contents -- With the versioning feature, storing 3 copies of the contents of the page:
    - Contents: `<html>...</html>`, ...
- COLUMN FAMILY: Anchor:
    - Anchor: cnnsi.com
    - Anchor: my.look.ca

### Accessing HBase

- HBase shell
- Java API
    - Wrappers for Python, Scala, etc.
- Spark, Hive, Pig, etc.
- REST service
- Thrift service -> Represents the data more compactly -- maximum performance
- Avro service -> Represents the data more compactly -- maximum performance

### HBase hands-on example: A table of movie ratings

- Create an HBase table for movie ratings by user
- Show we can quickly query it for individual users
- Good example of sparse data

#### Structure

- Column family: rating
- The row ID will be the user ID.
- Each individual column will represent a movie ID and the value will be the rating.

#### How to start HBase and a REST server

1. Open ports in the VirtualBox VM:
    - Settings -> Network -> Advanced -> Port Forwarding -> Add a new rule
        - name: HBase REST/Info
        - Protocol: TCP
        - Host IP: 127.0.0.1
        - Host Port: 6666/6667
        - Guest Port: 6666/6667
2. Start HBase
   ```
   Ambari -> HBase -> Service Actions -> Start
   ```
3. Log into the VM and start the REST server:
   ```bash
    sudo su
   /usr/hdp/current/hbase-master/bin/hbase-daemon.sh start rest -p 6666 --infoport 6667
   ```

#### How to create a table

Python Client -> REST API -> HBase | HDFS

- `Install starbase`:
    ```bash
    sudo pip install starbase
    ```
- Write the script:
    ```python
    from starbase import Connection

    c = Connection("127.0.0.1", "6666")
    
    if ratings.exists():
        print("Dropping existing ratings table\n")
        ratings.drop()
  
    ratings.create('rating')
  
    print("Parsing the ml-100k ratings data...\n")
    ratingFile = open("/home/maria_dev/ml-100k/u.data", "r")
  
    batch = ratings.batch()
  
    for line in ratingFile:
        (userID, movieID, rating, timestamp) = line.split()
        batch.update(userID, {'rating': {movieID: rating}})
  
    ratingFile.close()
  
    print("Committing ratings data to HBase via REST service\n")
    batch.commit(finalize=True)
  
    print("Get back ratings for some users...\n")
    print("Ratings for user ID 1:\n")
    print(ratings.fetch("1"))
    print("Ratings for user ID 33:\n")
    print(ratings.fetch("33"))
    ```

### HBase & Pig: Importing big data

- Must create HBase table first
- The relation must have a unique key as its first field, followed by subsequent columns as you want to have them saved
  in HBase
- USING clause allows you to STORE into an HBase table
- Can work at scale

#### Import the Users table

1. Upload the users file to HDFS:
   ```
   Ambari -> Files View -> 'users/maria_dev' -> Upload -> Select the file (`u.user`)-> Upload :
   ```
2. SSH into the VM and run the following command to open the interactive Hbase shell:
    ```bash
    hbase shell
    ```
3. Create the table:
    ```bash
    create 'users', 'userinfo'
    ```
4. Exit the shell:
    ```bash
    exit
    ```
5. Download the following file `wget http://media.sundog-soft.com/hadoop/hbase.pig`
   *(this is a Pig script that will import the users table into HBase)*
    ```pig
    users = LOAD '/user/maria_dev/ml-100k/u.user' 
    USING PigStorage('|')
    AS (userID:int, age:int, gender:chararray, occupation:chararray, zip:int);
    
    STORE users INTO 'hbase://users'
    USING org.apache.pig.backend.hadoop.hbase.HBaseStorage (
    'userinfo:age,userinfo:gender,userinfo:occupation,userinfo:zip');
    ```
6. Run the script:
    ```bash
    pig hbase.pig
    ```
7. Check the results:
    ```bash
    hbase shell
    scan 'users'
    ```
8. Delete the table:
    ```bash
    disable 'users'
    drop 'users'
    ```
9. Exit the shell:
    ```bash
    exit
    ```
10. Stop the HBase service:
    ```
    Ambari -> HBase -> Service Actions -> Stop
    ```

## Cassandra

- Unlike HBase, there is no master node - all nodes are the same
- Data model is similar to HBase
- It's non-relational, but it has a query language (CQL) as its interface

### The CAP Theorem

- consistency: every read receives the most recent write or an error
- availability: every request receives a response about whether it succeeded or failed
- partition tolerance: the system continues to work even if the network is partitioned

You can only have two of these at the same time. Cassandra is AP.

- It is "eventually consistent"
- It offers "tunable consistency": you can specify your consistency as part of your requests

*Cassandra is more available but less consistent, but it can be adjusted.*

*Other databases:*

- MongoDB: CP
- HBase: CP
- MySQL: CA

### Architecture

- Every node is identical
- The client chooses a node to connect to (usually the closest one) to know where the requested data lives
- Specifying the consistency level: how many nodes must respond to a request before it is considered successful
- Cassandra is a "ring" of nodes
- Data is partitioned across the nodes using consistent hashing
- Data is replicated across multiple nodes for fault tolerance
- Cassandra's great for fast access to rows of information

### CQL

- An API to do READs and WRITEs
- No JOINS
- All queries must be on primary key
- CQLSH can be used on the command line to interact with Cassandra
- All tables must be in a keyspace - keyspaces are like databases

### Cassandra & Spark

- DataStax offers a Spark-Cassandra connector
- Allows you to read and write Cassandra tables as DataFrames
- It is smart about passing queries on those DataFrames back down to the appropriate level
- Use cases:
    - Use Spark for analytics on Cassandra data
    - Use Spark to transform data and store it into Cassandra for transactional use

## MongoDB

A huMONGOus document database.

- You can store any kind of data in it
- MongoDB stores data in JSON-like documents
- An automatic _id field is added if you don't specify one
- You can enforce schemas if you want, but you don't have to
- You can have different fields in different documents in the same collection
- No single "key"
    - You can index any field or combination of fields
    - You can "shard" across indexes
- Lot of flexibility, but you have to be careful about consistency
- Aimed at enterprise use

### The CAP Theorem

- consistency: every read receives the most recent write or an error
- availability: every request receives a response about whether it succeeded or failed
- partition tolerance: the system continues to work even if the network is partitioned

You can only have two of these at the same time. MongoDB is CP.

### Terminology

- Database: a group of collections
- Collection: a group of documents
- Document: a set of key-value pairs

### Architecture

```
primary -> secondary -> secondary
        -> secondary -> secondary
```

- Single master/primary node
- Maintains backup copies of the database instances
- Secondaries nodes can elect a new master if the master/primary goes down
    - The operation log should be long enough to give time to recover the primary node once it's back up

#### Replica Set Quirk

- A majority of the nodes must agree on the primary
    - Even number of nodes is not recommended
- If you can't have an odd number of nodes, you can have an "arbiter" node
    - You can only have 1 "arbiter" node
- Apps must know enough servers in the replica set to be able to reach one to learn who's the primary
    - Fixed in MongoDB 3.6
- Replicas only address durability, not availability
    - If the primary goes down, you can't write to the database
    - You can still read from the secondaries, but they might be out of date
- Delayed secondaries can be used to recover from human error

### Big Data - Sharding

- Sharding is a way to partition data across multiple machines
- Multiple replica sets and each replica set is responsible for a range of values

#### Sharding Quirks

- Auto-sharding sometimes doesn't work
    - If your config servers go down, things can get into a bad state
- You must have 3 config servers (prior to MongoDB 3.2)
    - In current versions config server are part of a replica set, it just needs to have a primary

### Why MongoDB?

- Not just a NoSQL database -- very flexible
- Shell is a full JavaScript interpreter
- Supports many indices
    - Only 1 can be used for sharding
    - More than a few are discouraged
    - Text indices for text searches
    - Geospatial indices
- Built-in aggregation capabilities, MapReduce, GridFS
    - For some applications, you don't need Hadoop
    - MongoDB still integrates with Hadoop, Spark, etc.
- A SQL connector is available
    - But MongoDB still isn't designed for joins and normalized data
