# Apache Spark

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

## Components and Libraries (apart from Spark Core)

- Spark Streaming (real-time processing)
- Spark SQL (SQL and structured data processing)
- MLlib (machine learning)
- GraphX (graph processing)

## Resilient Distributed Datasets (RDDs)

- RDDs are the core abstraction in Spark
- Easily distributed across a cluster
- Fault-tolerant
- Looks like a dataset to the user

### SparkContext

- Created by the driver program
- Makes RDDs resilient and distributed
- Creates RDDs
- Spark shell automatically creates a SparkContext ("sc")

### Creating RDDs

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

### Transforming RDDs

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

### RDD Actions

Take one RDD and turn it into another

- collect: return all items in the RDD to the driver program
- count: return the number of items in the RDD
- countByValue: return the count of each unique value in the RDD as a dictionary
- take: return the first n items of the RDD
- top: return the top n items
- reduce: aggregate the elements of the RDD using a function
- ...

### Lazy Evaluation

- Nothing is computed until an action is called
    - Transformations are not executed until an action is called

## How to submit a Spark job

If you are using Hortonworks Sandbox 2.6.5, and you want to submit a Spark 1 job, you need to set the following
environment variable:

```shell
export SPARK_MAJOR_VERSION=1
```

After that, you can submit a Spark job using the following command:

```shell
spark-submit <filename.py>
```

*Don't forget to set the environment variable back to 2 if you want to submit a Spark 2 job.*

## Spark RDD hands-on Example: Find the movies with the lowest rating

```python
from pyspark import SparkConf, SparkContext

def loadMovieNames():
    movieNames = {}
    # this uses the u.item file in the ml-100k folder of the local filesystem
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames
    
# field[1] = movieID, field[2] = rating
def parseInput(line):
    fields = line.split()
    return (int(fields[1]), (float(fields[2]), 1.0))
    
if __name__ == "__main__":
    # Create a SparkConf object
    conf = SparkConf().setAppName("WorstMovies")
    sc = SparkContext(conf = conf)
    
    # Load up our movie ID -> name python dictionary
    movieNames = loadMovieNames()
    
    # Load up the raw u.data file in an RDD object
    lines = sc.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    
    # Convert to (movieID, (rating, 1.0)) RDD
    movieRatings = lines.map(parseInput)
    
    # Reduce to (movieID, (sumOfRatings, totalRatings)) RDD
    # Uses reduceByKey to aggregate rating tuples for each movie by summing their ratings and counts.
    # The reduceByKey function operates by applying the provided function to pairs of values associated with the same key.
    ratingTotalsAndCount = movieRatings.reduceByKey(lambda movie1, movie2: (movie1[0] + movie2[0], movie1[1] + movie2[1]))
    
    # Map to (movieID, averageRating) RDD
    averageRatings = ratingTotalsAndCount.mapValues(lambda totalAndCount: totalAndCount[0] / totalAndCount[1])
    
    # Sort by average rating
    sortedMovies = averageRatings.sortBy(lambda ratingTuple: ratingTuple[1])
    
    # Take the top 10 results
    results = sortedMovies.take(10)
    
    # Print them out
    for result in results:
        print(movieNames[result[0]], result[1])
```

*Don't forget to set the SPARK_MAJOR_VERSION environment variable back to 2 after you are done with Spark 1 jobs.*

## SparkSQL (using DataFrames and Datasets)

- Extends RDDs to a "DataFrame" object
- Dataframes:
    - Contain Row objects
    - Can run SQL queries
    - Has a schema (leading to more efficient storage)
    - Read and write from JSON, Hive, Parquet
    - Communicates with JDBC/ODBC, Tableau, etc.

### SparkSQL in Python

```python
from pyspark.sql import SparkSession, Row
hiveContext = HiveContext(sc)
inputData = spark.read.json(dataFile)
inputData.createOrReplaceTempView("myStructuredStuff")
myResultDataFrame = hiveContext.sql("""SELECT foo FROM bar ORDER BY foobar""")
```

### Programmatically working with DataFrames

```python
myResultDataFrame.show() # prints first 20 results to console
myResultDataFrame.select("someFieldName") # returns a new DataFrame with the column "someFieldName"
myResultDataFrame.filter(myResultDataFrame("someFieldName" > 200)) # returns a new DataFrame with the filter applied
myResultDataFrame.groupBy("someFieldName").mean() # returns a new DataFrame with the mean of each group
myResultDataFrame.rdd().map(mapperFunction) # returns a new RDD with the mapper function applied to each row
```

### Datasets

- In Spark 2.0
    - DataFrames are now Datasets of type Row
    - Datasets can wrap known, typed data
- In general, with Spark 2.0, you should use Datasets instead of DataFrames

### Shell access

- SparkSQL exposes a JDBC/ODBC server (if you Build Spark with Hive support)
- Start the server with `sbin/start-thriftserver.sh`
- Listen on port 10000 by default
- Connect with `bin/beeline -u jdbc:hive2://localhost:10000`
- You can create new tables, or query existing ones that were cached using `hiveCtx.cacheTable("tableName")`

### User-defined functions (UDFs)

```python 
from pyspark.sql.types import IntegerType
hiveCtx.registerFunction("square", lambda x: x * x, IntegerType())
df = hiveCtx.sql("SELECT square('someNumericField') FROM tableName")
```

### RDDs vs DataFrames vs Datasets

|                   | **RDDs**                                                   | **DataFrames**                                              | **Datasets**                                                      |
|-------------------|------------------------------------------------------------|-------------------------------------------------------------|-------------------------------------------------------------------|
| **Description**   | Fundamental data structure of Spark; collection of objects | Immutable distributed collection of data with named columns | Extension of DataFrames; provide a type-safe, object-oriented API |
| **Advantages**    | Provides low-level functionality and complete control      | Higher level abstraction with built-in optimizations        | Best of RDDs and DataFrames; strong typing and query optimization |
| **Disadvantages** | Requires more code; less built-in optimizations            | Lack compile-time type safety                               | Not available in Python                                           |

## Spark 2 DataFrames hands-on Example: Find the movies with the lowest rating

```python
from pyspark.sql import SparkSession
from pyspark.sql import Row, functions

def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames
     
def parseInput(line):
    fields = line.split()
    return Row(movieID = int(fields[1]), rating = float(fields[2]))
    
if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("LowestRatedMovies").getOrCreate()
    
    # Load up our movie ID -> name python dictionary
    movieNames = loadMovieNames()
    
    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    # Convert it to a RDD of Row objects with (movieID, rating)
    movies = lines.map(parseInput)
    # Convert that to a DataFrame
    movieDataFrame = spark.createDataFrame(movies)
    
    # Compute average rating for each movieID
    averageRatings = movieDataFrame.groupBy("movieID").avg("rating")
    
    # Compute count of ratings for each movieID
    counts = movieDataFrame.groupBy("movieID").count()
    
    # Join the two together (We now have movieID, avg(rating), and count columns)
    averagesAndCounts = counts.join(averageRatings, "movieID")
    
    # Pull the top 10 results
    topTen = averagesAndCounts.orderBy("avg(rating)").take(10)
    
    # Print them out, converting movie ID's to names as we go.
    for movie in topTen:
        print (movieNames[movie[0]], movie[1], movie[2])
    
    # Stop the session
    spark.stop()
```

## Using MLLib in Spark

- MLLib is Spark's machine learning library
- It is built on top of Spark

### MLLib installation

`MLLib` is already installed on the VM, but `numpy` is also required. Please install it using the following command:

```shell
sudo pip install numpy==1.16
```

## MLLib hands-on example: Predicting movie ratings

- `ALS` is a prediction algorithm

```python
from pyspark.sql import SparkSession
from pyspark.ml.recommendation import ALS
from pyspark.sql import Row
from pyspark.sql.functions import lit


def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames
     
def parseInput(line):
    fields = line.split()
    return Row(userID = int(fields[0]), movieID = int(fields[1]), rating = float(fields[2]))
    
if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("MovieRecomendations").getOrCreate()
    
    # Load up our movie ID -> name python dictionary
    movieNames = loadMovieNames()
    
    # Get the raw data
    lines = spark.read.text("hdfs:///user/maria_dev/ml-100k/u.data").rdd
    
    # Convert it to a RDD of Row objects with (userID, movieID, rating)
    ratingsRDD = lines.map(parseInput)
    
    # Convert to a DataFrame and cache it
    ratings = spark.createDataFrame(ratingsRDD).cache()
    
    # Create an ALS collaborative filtering model from the complete data set
    # Predicts rating for movies that users have not yet seen/rated
    als = ALS(maxIter=5, regParam=0.01, userCol="userID", itemCol="movieID", ratingCol="rating")
    model = als.fit(ratings)
    
    # Print out ratings from user 0:
    # Note that user 0 is a fabricated user, just to illustrate the process
    # He rated Star Wars & The Empire Strikes Back with a 5
    # He rated Gone with the Wind with a 1
    # So, he really likes sci-fi/fantasy and dislikes historical dramas
    print("\nRatings for user ID 0:")
    userRatings = ratings.filter("userID = 0")
    for rating in userRatings.collect():
        print (movieNames[rating['movieID']], rating['rating'])
        
    print("\nTop 20 recommendations:")
    # Find movies rated more than 100 times
    ratingCounts = ratings.groupBy("movieID").count().filter("count > 100")
    # Construct a "test" dataframe for user 0 with every movie rated more than 100 times
    # lit() is a way of creating a constant column in a DataFrame with a specific value
    # We do this to specify that we want predictions for user 0
    popularMoviesForUserID0 = ratingCounts.select("movieID").withColumn('userID', lit(0))
    
    # Run our model on that list of popular movies for user ID 0
    recommendations = model.transform(popularMoviesForUserID0)
    
    # Get the top 20 movies with the highest predicted rating for this user
    topRecommendations = recommendations.sort(recommendations.prediction.desc()).take(20)
    
    for recommendation in topRecommendations:
        print (movieNames[recommendation['movieID']], recommendation['prediction'])
        
    spark.stop()
```

## Spark RDD & DataFrame hands-on exercise: Filter the lowest rated movies by number of ratings

- The examples above find the lowest rated movies, but they don't take into account the number of ratings
- Modify both the RDD and DataFrame examples to filter out movies with less than 10 ratings

### RDD version

```python
from pyspark import SparkConf, SparkContext

def loadMovieNames():
    movieNames = {}
    # this uses the u.item file in the ml-100k folder of the local filesystem
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames
    
# field[1] = movieID, field[2] = rating
def parseInput(line):
    fields = line.split()
    return (int(fields[1]), (float(fields[2]), 1.0))
    
if __name__ == "__main__":
    # Create a SparkConf object
    conf = SparkConf().setAppName("WorstMovies")
    sc = SparkContext(conf = conf)
    
    # Load up our movie ID -> name python dictionary
    movieNames = loadMovieNames()
    
    # Load up the raw u.data file in an RDD object
    lines = sc.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    
    # Convert to (movieID, (rating, 1.0)) RDD
    movieRatings = lines.map(parseInput)
    
    # Reduce to (movieID, (sumOfRatings, totalRatings)) RDD
    # Uses reduceByKey to aggregate rating tuples for each movie by summing their ratings and counts.
    # The reduceByKey function operates by applying the provided function to pairs of values associated with the same key.
    ratingTotalsAndCount = movieRatings.reduceByKey(lambda movie1, movie2: (movie1[0] + movie2[0], movie1[1] + movie2[1]))
    
    # Filter out movies rated 10 or fewer times
    # Uses filter to remove movies rated 10 or fewer times.
    # The filter function operates by applying the provided function to each element and only passing those elements where the function returns true.
    ratingTotalsAndCountAbove10 = ratingTotalsAndCount.filter(lambda x: x[1][1] > 10)
    
    # Map to (movieID, averageRating) RDD
    averageRatings = ratingTotalsAndCountAbove10.mapValues(lambda totalAndCount: totalAndCount[0] / totalAndCount[1])
    
    # Sort by average rating
    sortedMovies = averageRatings.sortBy(lambda ratingTuple: ratingTuple[1])
    
    # Take the top 10 results
    results = sortedMovies.take(10)
    
    # Print them out
    for result in results:
        print(movieNames[result[0]], result[1])
```

### DataFrame version

```python
from pyspark.sql import SparkSession
from pyspark.sql import Row, functions

def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames
     
def parseInput(line):
    fields = line.split()
    return Row(movieID = int(fields[1]), rating = float(fields[2]))
    
if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("LowestRatedMovies").getOrCreate()
    
    # Load up our movie ID -> name python dictionary
    movieNames = loadMovieNames()
    
    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    # Convert it to a RDD of Row objects with (movieID, rating)
    movies = lines.map(parseInput)
    # Convert that to a DataFrame
    movieDataFrame = spark.createDataFrame(movies)
    
    # Compute average rating for each movieID
    averageRatings = movieDataFrame.groupBy("movieID").avg("rating")
    
    # Compute count of ratings for each movieID
    counts = movieDataFrame.groupBy("movieID").count()
    
    # Join the two together (We now have movieID, avg(rating), and count columns)
    averagesAndCounts = counts.join(averageRatings, "movieID")
    
    # Filter movies rated 10 or fewer times
    # Uses filter to remove movies rated 10 or fewer times.
    averagesAndCountsAbove10 = averagesAndCounts.filter("count > 10")
    
    # Pull the top 10 results
    topTen = averagesAndCountsAbove10.orderBy("avg(rating)").take(10)
    
    # Print them out, converting movie ID's to names as we go.
    for movie in topTen:
        print (movieNames[movie[0]], movie[1], movie[2])
    
    # Stop the session
    spark.stop()
```
