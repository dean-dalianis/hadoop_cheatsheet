# Data Streaming

## Kafka

**TODO**

## Setting up a Kafka cluster

**TODO**

## Publishing web logs to Kafka

**TODO**

## Flume

- Another way to stream data into Hadoop
- Made from the start with Hadoop in mind
    - Built-in sinks for HDFS and HBbase
- Originally made to handle log aggregation
    - Can handle other types of data as well

### Flume agents

```text
Web server -> Source -> Channel -> Sink -> HBbase
```

- Source
    - where the data comes from
    - can optionally have channel selectors to filter data
    - can optionally have interceptors to modify data
- Channel
    - How the data is transferred (e.g. memory, file, etc.)
- Sink
    - Where the data is going
    - Can be organized into groups
    - A sink can connect to only one channel
        - Channel is notified to dlete a message once it has been processed by the sink

### Built-in sources

- Spooling directory
- Avro
- Kafka
- Exec
- Thrift
- Netcat
- HTTP
- Custom sources in Java

### Built-in Sink

- HDFS
- Hive
- HBbase
- Avro
- Thrift
- ElasticSearch
- Kafka
- Custom sinks in Java

### Avro

- Agents can connect to other agents via Avro
- Scalable architecture

## Setting up Flume

**TODO**

## Using Flume to monitor a directory and store its data in HDFS

**TODO**

## Spark Streaming

- Analyze data streams in real time, instead of huge batch jobs daily
- Not really real time, but close enough
- RDD processing can happen in parallel

### DStreams

- Discretized Streams
- Generates RDDs for each step and can produce output
- Can be transformed and acted on like RDDs
- It can give access to the underlying RDDs
- Actions:
    - Map
    - FlatMap
    - Filter
    - reduceByKey
- Stateful data
    - Long live states
    - e.g. running totals, broken down by key
    - Windowed Transformations
        - Allow to compute over a sliding window of data
- Batch interval vs Slide interval vs Window interval
    - THe Batch interval is how often data is captured into a DStream
    - The Slide interval is how often a windowed transformation is computed
    - The Window interval is how far back in time the windowed transformation looks
- The batch interval is set up with the SparkContext
  ```python
  ssc = StreamingContext(sc, 1) # 1 second batch interval
  ```
- ReduceByKeyAndWindow
  ```python
  hashtag_counts = hashtag_key_values.reduceByKeyAndWindow(lambda x, y: x + y, lambda x, y: x - y, 300, 1) # 5 minute window, 1 second slide
  ```
- Structured streaming
    - New in Spark 2.0 as an experimental feature
    - It's the future of Spark Streaming
    - Uses DataSets instead of RDDs
        - Like a DataFrame, but it never ends
        - New data is appended to the end

### Analyze web logs published with Flume using Spark Streaming

**TODO**

### Monitor FLume-published logs for errors in real time

**TODO**

### Apache Storm

- Another framework for real-time data processing
    - Can run on top of YARN
- Works on individual events, not micro-batches (like Spark Streaming)
    - Ideal for sub-second latency
- A stream consists of tuples of data
- Spouts are sources of data (e.g. Kafka, Twitter, etc.)
- Bolts process the data (e.g. Transform, aggregate, write to DB/HDFS, etc.)
- A topology is a graph of spouts and bolts
  ```text
  Spout -> Bolt
                 -> Bolt        
        -> Bolt
  Spout 
        -> Bolt
  ```
- Architecture
  ```text
                      -> Supervisor
         -> Zookeeper -> Supervisor
  Nimbus -> Zookeeper -> Supervisor
         -> Zookeeper -> Supervisor
                      -> Supervisor
  ```
- Usually done with Java
- Storm Core
    - The low-level API for Storm
    - Offers "at least once" semantics
- Trident
    - A high-level API for Storm
    - Offers "exactly once" semantics
    - Can be used on top of Storm Core
- Storm runs applications "forever" (until stopped)

#### Storm vs Spark Streaming

- If true real-time is needed, Storm is the way to go
- Core Storm offers "tumbling windows" in addition to "sliding windows"
- Kafka and Storm are a popular combination
- A bit more established than Spark Streaming, but Spark Streaming is catching up

### Counting words with Storm

**TODO**

### Flink

- Another data streaming framework
- German for "quick and nimble"
- Similar to Storm
- Can run on a standalone cluster or on YARN or Mesos
- Highly scalable (1000's of nodes)
- Fault tolerant
    - Can recover from failures while guaranteeing exactly once semantics
    - Uses "state snapshots" to recover from failures

#### Flink vs Storm vs Spark Streaming

- Flink is faster than Storm
- Flink offers "real time" streaming like Storm (but if you're using Trident with Storm, it's not really real time)
- Flink offers a high-level API like Trident or Spark, but still does real-time streaming
- Flink has good Scala support
- Flink has its own ecosystem (e.g. FlinkML, Gelly, etc.)
- Can process data on event time, not when it arrives
    - Impressive windowing system
- Flink is the newest of the three
- All three are gradually converging
    - Spark's "Structured Streaming" paves the way for a real-time event processing system

### Counting words with Flink

**TODO**

