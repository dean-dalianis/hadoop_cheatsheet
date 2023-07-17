# Query Engines

## Apache Drill

- A SQL query engine for a variety of non-relational datastores
    - Hive, MongoDB, HBase
    - Flat JSON files, Parquet files on HDFS, S3, Azure, Google Cloud Storage, local filesystem
- Based on Google's Dremel
- Not SQL-Like -- Actually SQL
- It has an ODBC/JDBC driver
    - Other tools can connect to it like any relational database (e.g. Tableau)
- Still not a relational database
    - Don't push it with big joins etc.
- Allows SQL analysis without having to transform and load data into a relational database
- If you know how to write SQL, you can use Drill
- You can even do joins between different database technologies
    - Or with flat JSON files

### Setting up Drill

1. Start MongoDB from Ambari

2. Open HiveView from Ambari
    - Import the ratings data
      ```sql
      CREATE DATABASE IF NOT EXISTS movielens;
      ```
    - Click on the "Upload Table" button
    - Change the CSV Field Delimiter to `TAB`
    - Choose the `u.data` file
        - Select the `movielens` database
    - Name the table `ratings`
        - Change column names to `user_id`, `movie_id`, `rating`, `epoch_seconds`
    - Click on "Upload"
3. Import data to MongoDB
   ```bash
   ssh maria_dev@localhost -p 2222
   sudo su
   spark-submit --packages org.mongodb.spark:mongo-spark-connector_2.11.2.6.5 MongoSpark.py
    ```
4. Install Drill
   ```bash
   wget http://archive.apache.org/dist/drill/drill-1.12.0/apache-drill-1.12.0.tar.gz
   tar -xvf apache-drill-1.12.0.tar.gz
   ```
5. Start Drill
   ```bash
    cd apache-drill-1.12.0
    bin/drillbit.sh start -Ddrill.exec.port=8765
    ```
6. Connect through the browser: http://127.0.0.1:8765
7. Connect to MongoDB and Hive
    - Click on `storage` tab and make sure the following are enabled:
        - `hive`
        - `mongo`
    - Click `update` next to `hive`
        - Change the `hive.metastore.uris` to `thrift://localhost:9083`
    - Mongo works out of the box

### Querying with Drill

1. Go to http://127.0.0.1:8765
2. Click on `Query` tab
3. Write and submit the following query:
   ```sql
   SHOW DATABASES;
   ```
4. Query the data from Hive and MongoDB
    ```sql
    SELECT * FROM hive.movielens.ratings LIMIT 10;
    SELECT * FROM mongo.movielens.users LIMIT 10;
    ```
5. Tie the two together by joining data from Hive and MongoDB (how many ratings were provided by each occupation)
    - Join the users and ratings tables with `user_id` column and group by `occupation`
       ```sql
       SELECT u.occupation, COUNT(*) FROM hive.movielens.ratings r JOIN mongo.movielens.users u ON r.user_id = u.user_id GROUP BY u.occupation;
       ```

### Cleaning up

1. Stop Drill from the command line
    ```bash
    bin/drillbit.sh stop
    ```
2. Stop MongoDB from Ambari

## Apache Phoenix

- A SQL driver for HBase
- It supports transactions
- Fast, low latency - Online Transaction Processing (OLTP) support
- Developed by Salesforce, then open sourced
- Exposes a JDBC connector for HBase
- Supports secondary indices and UDFs (user defined functions)
- Integrates with MapReduce, Spark, Hive, Pig, and Flume

**Why Phoenix?**

- Really fast
- Why Phoenix and not Drill?
    - Choose the right tool for the job
    - Focuses exclusively on HBase
- Why not HBase's native clients?
    - SQL might be easier to use
    - Existing applications might already be using SQL
    - Phoenix can do a lot of optimizations for you

### Architecture

```text
Phoenix Client | HBase API -> HBase Region Server(s) | Phoenix Co-Processor ->  HDFS
                                                                            -> Zookeeper
```

### Using Phoenix

- Command-line interface
- JDBC driver
- Java API
- Phoenix Query Server (PQS)
    - Intended to eventually enable non-JVM access
- Provides JARs for MapReduce, Pig, Hive, Pig, Flume, and Spark

### Setting up Phoenix and Command-line Interface

1. Make sure HBase is running from Ambari (by default it is not)
2. Login to the VM and start Phoenix
   ```bash
    ssh maria_dev@localhost -p 2222
    sudo su
    cd /usr/hdp/current/phoenix-client/
    python bin/sqlline.py
    ```
3. Create a table with US population data
   ```sql
    CREATE TABLE IF NOT EXISTS us_population (state CHAR(2) NOT NULL, city VARCHAR NOT NULL, population BIGINT) CONSTRAINT my_pk PRIMARY KEY (state, city);
    !tables
    ```
4. Insert some data (INSERT doesn't work, use UPSERT)
    ```sql
    UPSERT INTO us_population VALUES ('NY', 'New York', 8143197);
    UPSERT INTO us_population VALUES ('CA', 'Los Angeles', 3844829);
    ```
5. Query the data
    ```sql
    SELECT * FROM us_population;
    SELECT * FROM us_population WHERE state = 'CA';
    ```
6. Clean up
    ```sql
    DROP table us_population;
    quit;
    ```

### Querying with Phoenix (Integrating with Pig)

- Use the `u.data` file from the MovieLens dataset
- We will use Pig to load the data into HBase via Phoenix
- We will then query the data using Phoenix

1. Create a table in Phoenix
    ```sql
    CREATE TABLE IF NOT EXISTS users (USERID INTEGER NOT NULL, AGE INTEGER, GENDER CHAR(1), OCCUPATION VARCHAR, ZIP VARCHAR CONSTRAINT my_pk PRIMARY KEY (USERID));
    ```
2. Load the data into HBase via Pig and Phoenix
    - Go to the home directory: `cd /home/maria_dev`
    - Create a directory for the ml-100k dataset: `mkdir ml-100k` if it doesn't exist
     ```bash
     cd ml-100k
     wget http://media.sundog-soft.com/hadoop/ml-100k/u.user
     cd ..
     ```
    - Get the Pig script
        ```bash
        wget http://media.sundog-soft.com/hadoop/phoenix.pig
        ```
        ```pig
        set zookeeper.znode.parent '/hbase-unsecure'
        REGISTER /usr/hdp/current/phoenix-client/phoenix-client.jar
        users = LOAD '/user/maria_dev/ml-100k/u.user'
        USING PigStorage('|')
        AS (USERID:int, AGE:int, GENDER:chararray, OCCUPATION:chararray, ZIP:chararray);
   
        STORE users into 'hbase://users' using
        org.apache.phoenix.pig.PhoenixHBaseStorage('localhost','-batchSize 5000');
   
        occupations = load 'hbase://table/users/USERID,OCCUPATION' using org.apache.phoenix.pig.PhoenixHBaseLoader('localhost');
   
        grpd = GROUP occupations BY OCCUPATION;
        cnt = FOREACH grpd GENERATE group AS OCCUPATION,COUNT(occupations);
        DUMP cnt;
        ```
3. Run the Pig script
   ```bash
    pig phoenix.pig
    ```
4. Clean up:
   ```bash
    /usr/hdp/current/phoenix-client/bin/sqlline.py
   ```
   ```sql
    DROP TABLE users;
   ```
5. Stop HBase from Ambari

## Presto

- Similar to Apache Drill
    - Can connect to many different "big data" databases
    - Familiar SQL syntax
    - Optimized for OLAP (Online Analytical Processing), data warehousing
    - Exposes JDBC, CLI, and Tableau interfaces
- Made by Facebook, and still partially maintained by them
- Open source
- Can talk to Cassandra - Drill can't
- Can't talk to MongoDB - Drill can

### Why Presto?

- Instead of Drill?
    - It has a Cassandra connector
- It's used by Facebook against 30PB of data, Dropbox, and AirBnB
- A single Presto query can combine data from multiple sources

### What can Presto connect to?

- Cassandra
- Hive
- MongoDB
- MySQL
- PostgreSQL
- Local files
- Kafka
- JMX
- Redis
- Accumulo

### Setting up Presto

1. Login to the VM
    ```bash
    ssh maria_dev@localhost -p 2222
    sudo su
    ```
2. Download and install Presto
    ```bash 
    wget https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.165/presto-server-0.165.tar.gz
    tar -xvf presto-server-0.165.tar.gz
    ```
3. Get the configuration files (presto has a lot of configuration files, the guide to set them up
   is [here](https://prestodb.io/docs/current/installation/deployment.html), but we will use pre-made ones for the HDP
   VM)
    ```bash
    wget https://raw.githubusercontent.com/sundog-education/hadoop/presto-hdp-config.tgz
    tar -xvf presto-hdp-config.tgz
    ```
4. Download the CLI for presto
   ```bash
    wget https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.165/presto-cli-0.165-executable.jar
    mv presto-cli-0.165-executable.jar presto
    chmod +x presto
   ```
5. Start Presto
   ```bash
    cd presto-server-0.165
    bin/launcher start
    ```
6. Connect to Presto UI: http://localhost:8090
7. Run the CLI
   ```bash
    ./presto --server 127.0.0.1:8090 --catalog hive
    ```
8. Run queries against Hive
   ```sql
    show tables from default;
    select * from default.ratings limit 10;
    select * from default.ratings where rating = 5 limit 10;
    select count(*) from default.ratings where rating = 1;
    ```

### Querying Cassandra with Presto

1. Start Cassandra, enable thrift, and enter the CQL shell
    ```bash
    service cassandra start
    node enablethrift
    cqlsh --cqlversion="3.4.0"
    ```
2. Check if movielens keyspace exists
    ```sql
    DESCRIBE keyspaces; use movielens; describe tables; select * from users limit 10;
    ```
3. Quit the CQL shell and set up the Cassandra connector for Presto
    ```sql
    exit
    ```
    ```bash
    cd /home/maria_dev/presto-server-0.165/etc/catalog
    nano cassandra.properties
    ```
4. Add the following to the file
      ```yaml
      connector.name=cassandra
      cassandra.contact-points=127.0.0.1
      ```
5. Start presto
    ```bash
    cd /home/maria_dev/presto-server-0.165
    bin/launcher start
    ```
6. Connect to the CLI
    ```bash
    ./presto --server 127.0.0.1:8090 --catalog hive,cassandra
    ```
7. Query data
    ```sql
    show tables from cassandra.movielens;
    describe cassandra.movielens.users;
    select * from cassandra.movielens.users limit 10;
    select count(*) from hive.default.ratings limit 10;
    select u.occupation, count(*) from hive.default.ratings r join cassandra.movielens.users u on r.user_id = u.user_id group by u.occupation
    ```