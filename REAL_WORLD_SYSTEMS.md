# Designing Real World Systems

## Technologies

### Impala

- Cloudera's alternative to Hive
- Massively parallel processing (MPP) SQL query engine built on top of Hadoop
- Impala's always running, so you avoid the startup time when starting a Hive query
- Impala is faster than Hive, but Hive is more mature and has more features

### Accumulo

- Another BigTable clone (like HBase)
- Offers a better security model than HBase
    - Cell-level security
- Allows server-side programming
- Consider it for NoSQL applications that require a high level of security

### Redis

- A distributed in-memory data store (like memcache)
- Good support for data structures
- Can persist data to disk
- Can be used as a data store not just a cache
- Popular caching solution for web applications

### Ignite

- "In Memory Data Fabric"
- An alternative to Redis
- Close to a database
    - Supports SQL queries
    - Supports ACID transactions
    - All is done in memory

### Elasticsearch

- A distributed document search and analytics engine
- Really popular
- Can handle real-time search-as-you-type
- Paired with Kibana, great for interactive data exploration

### Kinesis (and the AWS ecosystem)

- Amazon Kinesis is basically the AWS version of Kafka
- Amazon ecosystem
    - Elastic MapReduce (EMR)
    - S3 (distributed storage solution -- can be used instead of HDFS)
    - DynamoDB (NoSQL database)
    - Amazon RDS (relational database)
    - ElastiCache (distributed in-memory cache)
    - AI / ML services
- EMR is an easy way to spin up a Hadoop cluster on demand

### Apache NiFi

- Directed graphs of data routing
    - Can connect to Kafka, HDFS, Hive
- Web UI for designing complex data flows

### Falcon

- A data governance engine on top of Oozie
- Included in Hortonworks Data Platform (HDP)
- Like NiFi, it allows construction of data processing graphs

### Apache Slider

- Deployment tool for general apps on a YARN cluster
- Allows monitoring and scaling of apps
- Manages mixed configurations
- Allows to Start/Stop apps on your cluster

## Understanding the requirements

- Start with end user requirements, not where the data is coming from
- What sort of access patterns will you have?
    - Analytical queries that span large data ranges?
    - Huge amounts of small transactions for very specific rows of data?
    - Both?
- What availability do you need?
- What consistency do you need?
- How big is the big data?
- How much internal infrastructure and expertise is available?
- What about data retention?
- What about security?
- Latency
    - How quickly do end users need to see the data?
- Timeliness
    - Can queries be based on day-old/minute-old data?
    - Or do the need to be near-real-time?

## Sample application: consume webserver logs and keep track of top-sellers

- A system to track and display the top 10 best-selling products on a e-commerce website
- A good question to ask is
    - What does it really means best-selling in terms of time?
        - Last hour?
        - Last day?
        - Last week?
- Millions of end-users, generating thousands of events per second
    - It must be fast, page latency is important
        - We need some distributed NoSQL solution
    - Access patter is simple : "Give me the current top N sellers in category X"
- Hourly updates are good enough (consistency not hugely important)
- Must be highly available (customers don't like broken websites)
- So, we need partition tolerance and availability (AP) more than consistency (C) (CAP theorem)

### Solution

- Cassandra
    - Spark can talk to Cassandra
        - Spark Streaming can add things up over windows of time
- Getting data into Spark Streaming?
    - Kafka or Flume
    - Flume is purpose-built for HDFS
- Don't forget about security
    - Purchase data is sensitive
        - Strip out data that isn't needed
    - Security consideration may force a totally different design

### Architecture

```text
Purchase Servers -> Flume -> Spark Streaming -> Cassandra -> Web Servers
```

### Other ways to do it

- If there is an existing purchase database
    - Instead of streaming, hourly batch jobs
    - Sqoop + Spark -> Cassandra
- DO people need this data for analytics?
    - Store data on HDFS in addition to Cassandra

## Sample application: serving movie recommendations to a website

- Users want to discover movies they haven't seen
- Their own behavior (ratings, purchases, views) are probably the best indicator of what they like
- Availability and partition tolerance are important, but consistency not so much

### Solution

- Cassandra could be a good fit
    - But any NoSQL database would work
- Getting recommendations
    - Spark MLlib
    - Flink could also be an alternative
- Timeliness requirements
    - Real-time ML is a tall order - do you really need real-time?
    - It would be nice
- Creative thinking
    - Precomputing recommendations
        - Isn't timely
        - Waste of resources
    - Item-based collaborative filtering
        - Store movies similar to other movies (these relationships don't change often)
        - A runtime, recommend movies similar to ones you've liked (based on real-time behavior data)
    - We need something that can quickly look up movies similar to ones you've liked at scale
        - Could reside within the web app, but probably needs its own service
- Web service to create recommendations on demand
    - It will talk to a fast NoSQL data store with movie similarities data
    - It also needs user's past ratings/purchases/views
    - Movie similarities data (which are expensive) can be updated infrequently

### Architecture

```text 
               Oozie -> Spark/MLLib    <-> 
Web servers -> Flume -> Spark Streaming ->  HBase (user ratings and movie similarities) | HBase -> Recs service (on YARN or Slider)
                                                                                                ->
```

