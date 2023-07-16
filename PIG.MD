# Pig

Pig is a platform that allows you to analyze large datasets in a high-level scripting language called Pig Latin. It
provides SQL-like syntax for the map and reduce steps and is highly extensible. Pig runs on top of Hadoop and utilizes
its underlying components such as MapReduce, Tez, YARN, and HDFS.

## TEZ

Tez is a more efficient way of organizing jobs than MapReduce. It can be approximately 10 times faster than MapReduce
for certain workloads. You can use Tez in Ambari by selecting the 'Execute on Tez' option when running a Pig script.

## Running Pig

There are multiple ways to run Pig scripts:

- **Grunt**: Running scripts one line at a time.
- **Script**: Running commands through a script file.
- **Ambari**: Utilizing the user interface provided by Ambari to execute Pig scripts.

## Pig hands-on Example: Find the Oldest Movie with a 5-Star Rating

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

## Pig Latin Commands and Functions

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

## Pig hands-on Exercise: Most-rated one-star movie

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
