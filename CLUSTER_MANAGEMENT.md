# Managing a Hadoop Cluster

## YARN

- Yet Another Resource Negotiator
    - Introduced in Hadoop 2.0
    - Separates the resource management from MapReduce
    - Enabled development of MapReduce alternatives (Spark, Tez) built on top of YARN

### Architecture

```text
MapReduce   Spark    Tez       ->     YARN Applications
  |           |       |
  |           |       |
  |           |       |
             YARN              ->     Cluster Compute Layer
              |
              |
              |
             HDFS              ->    Cluster Storage Layer
```

### How YARN Works?

- An application talks to the Resource Manager to distribute work across the cluster
- Data locality can be specified to run tasks on the same node as the data
    - YARN will try to do this by default
- Different scheduling options are available
    - More than one application can be running at a time
    - FIFO: First In First Out - runs jobs in the order they are submitted
    - Capacity Scheduler - many jobs in parallel if there are enough resources
    - Fair Scheduler - shares resources between jobs, cuts into large jobs if necessary

## Tez

- Built on top of YARN
- Another data processing framework you can use
    - Makes Hive, Pig, or MapReduce jobs run faster
- Constructs Directed Acyclic Graphs (DAGs) for more efficient processing of distributed jobs
    - Eliminates unnecessary steps and dependencies
- Optimizes physical data flow and resource usage

### Using Hive with Tez and MapReduce

1. Open Ambari: http://localhost:8080
2. Open the `Hive` View
3. Make sure the ratings database is loaded
4. Import the `u.item` file, under the name `names` in the database `movielens` with
   names `movie_id`, `name`, `release_date`, ...
5. Go back to the `Query` tab and write the following query:
    ```sql
    DROP VIEW IF EXISTS topMovieIDS;
    CREATE VIEW topMovieIDS AS
    SELECT movie_id, COUNT(movie_id) AS ratingCount
    FROM movielens.ratings
    GROUP BY movie_id
    ORDER BY ratingCount DESC;
   
   SELECT n.name, ratingCount
    FROM topMovieIDS t JOIN movielens.names n ON t.movie_id = n.movie_id;
    ```
6. Click on the `gear` icon on the right, `add` and select `hive.execution.engine` and set it to `tez`
7. Go back to the `Query` tab and run the query (it should take about 21 seconds)
8. Go back to the `gear` icon and set the `hive.execution.engine` to `mr`
9. Rerun the query (it should take about 1 minute and 11 seconds)

## Mesos

- Another resource negotiator
- Came out of Twitter
- Not just for big data, it can allocate resources for web server, small jobs, etc.
- Meant to solve the problem of resource allocation for multiple applications, not just big data
- Isn't tied to Hadoop, but can be used with Spark, Storm, etc.
- Hadoop YARN can be integrated with Mesos using Myriad

### Mesos vs YARN

- YARN is monolithic
- Mesos is a two-tiered system
    - Makes offers of resources to "frameworks"
    - Frameworks can accept or reject offers
    - It can run custom scheduling algorithms
- Yarn is optimized for long, analytical jobs
- Mesos is built to handle the above, but also long-lived and short-lived processes as well (web servers, etc.)

### How Mesos fits in?

- Looking for an architecture for all of your applications? Mesos can be used for all of them
    - Kubernetes / Docker are also options
- If you're just looking for a resource negotiator for Hadoop, YARN is probably the way to go
    - Spark on Mesos is limited to one executor per node
- You can also tie Mesos and YARN together using Myriad to get the best of both worlds

### When to use Mesos?

- If it's already used in your organization
    - Integrate the Hadoop cluster with Mesos
- Otherwise, probably not worth the effort
    - YARN is the standard for Hadoop

## ZooKeeper

- Keeps track of information to be synchronized across the cluster
    - Which node is the master?
    - What tasks are assigned to each node?
    - Which worker nodes are available?
- A tool that applications can use to recover from partial failures
- An integral part of HBase, High-Availability MapReduce, Drill, Storm, Solr, Kafka, etc.

### Failure Modes

- Master crashes, needs to fail over to a backup: ZooKeeper keeps track of the master
- Worker node crashes, work needs to be reassigned: ZooKeeper keeps track of the work
- Network issues, nodes can't communicate with each other: ZooKeeper keeps track of the nodes

### Primitive Operations (not the same as the ZooKeeper API)

- Master election
    - One node registers itself as a master, and holds a "lock" on that data
    - Other nodes cannot become master until the lock is released
    - Only one node allowed to hold the lock at a time
- Crash detection
    - Nodes periodically send "heartbeats" to ZooKeeper
    - If a node stops sending heartbeats, it's considered dead
- Group membership
- Metadata
    - List of outstanding tasks, task assignments, etc.

### ZooKeeper API

- Instead of the above, it's a more general purpose system that makes it easy to build the above
- A little distributed file system
    - With strong consistency guarantees
    - Replace the concept of a file with a znode
- Notifying Clients
    - Clients can register to be notified when a znode changes
        - Avoids polling
- Persistent znodes
    - Persistent znodes remain stored until they are explicitly deleted
- Ephemeral znodes
    - Ephemeral znodes are deleted when the client that created them disconnects or crashes

### Architecture

- ZooKeeper is a distributed system that runs on a cluster of machines
- Clients connect to one of the machines in the cluster

### ZooKeeper Quorums

- ZooKeeper's quorums are used to ensure that the system is available even if some of the nodes are down.
- Nodes need to agree on the state of the system
- A quorum is a majority of nodes
    - If there are 5 nodes, 3 of them need to agree
    - If there are 6 nodes, 4 of them need to agree
- In general, we should have at least 5 servers with a quota of 3

### Simulating a Failing master

1. Login to the VM
    ```bash
    ssh maria_dev@localhost -p 2222
    ```
2. Navigate to the ZooKeeper directory
    ```bash
    cd /usr/hdp/current/zookeeper-client/bin
    ```
3. Start the ZooKeeper client
    ```bash
    ./zkCli.sh
    ```
4. List the / path
    ```bash
    ls /
    ```
5. Create a test master
   ```bash
   create -e /testmaster "127:0.0.1:2223" # -e makes it ephemeral
   ```
6. Get the master
   ```bash
   get /testmaster
   ``` 
7. Quit the ZooKeeper client -> for ZooKeeper the testmaster node died
   ```bash
   quit
   ```
8. Start the ZooKeeper client again and check the /testmaster node -> it's no longer there
   ```bash
   ./zkCli.sh
    get /testmaster
   ```
9. If another system tried this, it would try to be the master
    ```bash
    create -e /testmaster "127:0.0.1:2224"
    ```
10. If a 3rd system tried this, it would fail because there is already a master
    ```bash
    create -e /testmaster "127:0.0.1:2225"
    ```

## Oozie

- A system for running and scheduling Hadoop jobs

### Workflow

- A workflow is a collection of actions
- Chains together MapReduce, Pig, Hive, Sqoop and distcp jobs
- A workflow is an Acyclic Directed Graph (DAG) of actions
    - Specified via XML

#### Example workflow

```text
              -> pig (Extract top-rated movies ID's)
start -> fork                                           -> join -> hive (Join and filter results) -> end
              -> sqoop (Extract movie titles)
```

#### Setting up a Workflow

- Make sure each action works on its own
- Make a directory in HDFS for the workflow
- Create a workflow.xml file in the directory
- Create a job.properties defining any variables workflow.xml needs
    - This is stored in the local file system where the workflow is run from
    - These properties can also be set in the workflow.xml file

#### Running

```bash
oozie job -oozie http://localhost:11000/oozie -config /home/maria_devjob.properties -run
```

*You can monitor the job in the Oozie web UI: http://localhost:11000/oozie*

### Coordinators

- Schedules workflow execution
- Launches workflows at a specified time or when data becomes available
- Run in exactly the same way as workflows

### Oozie Bundles

- New in Oozie 3.0
- A bundle is a collection of coordinators that can be managed together
- Example: a bunch of coordinators that process log data
    - By grouping them together, you can start and stop them all at once (e.g. if there is some problem with the data)

### A simple oozie workflow

**TODO**

## Zeppelin

- A notebook-style interface for Big Data
- Same idea as iPtyhon notebooks
- Lets you run scripts / code interactively
- Can interleave code with formatted notes
- Notebooks can be shared with others on the cluster

### Apache Spark Integration

- Can run Spark code interactively
    - Speeds up development
    - Allows for interactive data exploration
- Can execute SQL queries against SparkSQL
- Query results can be visualized
- Makes Spark feel more like a data science tool

### Zeppelin Interpreters

- Zeppelin uses interpreters to run code
- Each interpreter is responsible for running a specific language

### Analyzing movie ratings with Zeppelin

**TODO**

## Hue

- Hadoop User Experience
- A web UI for Hadoop (Cloudera)

## Other technologies

### Ganglia

- Distributed monitoring system
    - Developed by Berkeley
    - Originally widely used by the scientific community
    - Wikipedia used to use it
- It's dead -- last update was in 2008

### Chukwa

- System for collecting and analyzing logs from Hadoop
- Initially adopted by Netflix, but they've since moved on
- Got replaced by Flume and Kafka
- It's dead -- last update was in 2010
