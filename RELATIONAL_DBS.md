# Relational Data Stores

## HIVE

- Distributing SQL queries across Hadoop clusters
- Translates SQL queries into MapReduce or Tez jobs

**Why HIVE?**

- SQL-like interface (HiveQL)
- Interactive
- Scalable
- Easy OLAP (Online Analytics Processing): easier than writing MapReduce jobs
- Highly optimized
- Extensible
    - User-defined functions (UDFs)
    - Thrift server (talk to Hive as a service)
    - JDBC/ODBC server

**Why not HIVE?**

- High latency -- not appropriate for OLTP (Online Transaction Processing)
- Stores data de-normalized
- SQL is limited in functionality
    - Pig, Spark, etc. are more flexible
- Not transactional
- No record-level updates, inserts, deletes

### HiveQL

- MySQL with some extensions
- e.g. views:
    - Can store the results of a query as a view, which can be queried later as a table
- Allows to specify the format of the data

### HIVE hands-on example: Find the most popular movies

- You can use Ambari to import data into Hive
- Navigate to Hive View
- Navigate to Upload Table
- Set the delimiter to `tab` and upload the `u.data` file
- Set the delimiter to `|` and upload the `u.item` file
- Navigate to Query Editor and press refresh to see the tables

Find the most popular movies depending on the number of ratings:

```sql
CREATE VIEW IF NOT EXISTS topMovieIDs AS
SELECT movieID, COUNT(movieID) as ratingCount
FROM ratings
GROUP BY movieID
ORDER BY ratingCount DESC;

SELECT n.title, ratingCount
FROM topMovieIDs t
         JOIN names n ON t.movieID = n.movieID;
```

Drop the view:

```sql
DROP VIEW IF EXISTS topMovieIDs;
```

### HIVE: under the hood

- Hive maintains a "metastore" that contains the schema of the tables and the location of the data

- LOAD DATA (managed tables)
    - Moves the data from a distributed file system into a Hive
- LOAD DATA LOCAL (external tables)
    - Moves the data from a local file system into a Hive
- Managed vs. External tables
    - Managed tables: Hive manages the data
        - e.g. DROP TABLE will delete the data
    - External tables: Hive does not manage the data
        - e.g. DROP TABLE will not delete the data

- How to create a managed table from a file:
    ```sql
    CREATE TABLE IF NOT EXISTS ratings (
        userID INT,
        movieID INT,
        rating INT,
        time INT
    )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;
    
    LOAD DATA LOCAL INPATH '/home/maria_dev/ml-100k/u.data'
    OVERWRITE INTO TABLE ratings;
    ```

- How to create a external table from a file:
    ```sql
    CREATE TABLE IF NOT EXISTS ratings (
        userID INT,
        movieID INT,
        rating INT,
        time INT
    )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;
    
    LOCATION '/home/maria_dev/ml-100k/u.data'
    ```

- Partitioning
    - You can store data in partitioned subdirectories
        - Huge optimization for queries that filter on the partitioning column
  ```sql
    CREATE TABLE customers (
        name STRING,
        address STRUCT<street:STRING, city:STRING, state:STRING, zip:INT>
    )
    PARTITIONED BY (country STRING);
  ```

- How to use HIVE
    - Interactive shell
        - `hive`
        - Saved query files
            - `hive -f <filename>`
    - Through Ambari/Hue
    - Through JDBC/ODBC server
    - Through Thrift server
        - Hive is not suitable for Online Transaction Processing (OLTP)
    - Via Oozie

### HIVE hands-on exercise: Find the movies with the highest average rating

- AVG() can be used on aggregated data, like COUNT()
- Only consider movies with more than 10 ratings

```sql
CREATE VIEW IF NOT EXISTS topMovieIDs AS
SELECT movieID, COUNT(movieID) as ratingCount, AVG(rating) as averageRating
FROM ratings
GROUP BY movieID
ORDER BY averageRating DESC;


SELECT n.title, averageRating, ratingCount
FROM topMovieIDs t
         JOIN names n ON t.movieID = n.movieID
WHERE ratingCount > 10;
```

## MySQL

- Popular, free, relational database
- Monolithic, not distributed
- Can be used for OLTP (Online Transaction Processing), so it can be used for real-time queries
- Existing data may be stored in MySQL and needs to be imported into Hadoop

### Sqoop

- Kicks off MapReduce jobs to import data from relational databases into Hadoop
- The jobs are called "Mappers"

#### MySQL -> HDFS

```bash
# -m 1 means that we want to use 1 mapper
sqoop import --connect jdbc:mysql://localhost/movielens --driver com.mysql.jdbc.Driver --table movies -m 1
```

#### MySQL -> HIVE

```bash
sqoop import --connect jdbc:mysql://localhost/movielens --driver com.mysql.jdbc.Driver --table movies -m 1 --hive-import
```

#### Incremental imports

- You can keep your relational database in sync with Hadoop
- You can specify a column to use as a "check column": `--check-column`
- You can specify a value to use as a "last value": `--last-value`

#### HIVE -> MySQL

- Target table must exist in MySQL, with columns in the same order as in HIVE

```bash
sqoop export --connect jdbc:mysql://localhost/movielens -m 1 --driver com.mysql.jdbc.Driver --table exported_movies --export-dir /apps/hive/warehouse/movies --input-fields-terminated-by '\0001'
```

### Installing MySQL on HDP 2.6.5 & importing data

- MySQL is already installed on HDP 2.6.5 but we need to set the root password
    ```bash
        sudo su
        systemctl stop mysqld
        systemctl set-environment MYSQLD_OPTS="--skip-grant-tables --skip-networking"
        systemctl start mysqld
        mysql -uroot
    ```
- Set the root password
    ```sql 
        FLUSH PRIVILEGES;
        alter user 'root'@'localhost' IDENTIFIED BY 'hadoop';
        FLUSH PRIVILEGES;
        QUIT;
    ```
- Restart MySQL
    ```bash
    systemctl unset-environment MYSQLD_OPTS
    systemctl restart mysqld
    ```
- Login to MySQL (password is `hadoop`)
    ```bash
    mysql -u root -p
    ```
- Create a database
    ```sql
    CREATE DATABASE movielens;
    ```
- Exit MySQL
    ```sql
    QUIT;
    ```
- Download the movielens data in SQL format
    ```bash
    wget http://media.sundog-soft.com/hadoop/movielens.sql
    ```

- Import the data into MySQL (password is `hadoop`)
    ```bash
    ```bash
    mysql -u root -p
    ```

    ```sql
    SET NAMES 'utf8';
    SET CHARACTER SET utf8;
    use movielens;
    source movielens.sql;
    SELECT * FROM movies LIMIT 10;
    DESCRIBE ratings;
    ```

- Find the most popular movies
  ```sql
    SELECT movies.title, COUNT(ratings.movie_id) AS ratings_count
    FROM movies
    INNER JOIN ratings ON movies.id = ratings.movie_id
    GROUP BY movies.title
    ORDER BY ratings_count;
  ```

### Using Sqoop to import data from MySQL into HDFS

- Login to MySQL (password is `hadoop`)
    ```bash
    mysql -u root -p
    ```

- Give access to Sqoop
  ```sql
    GRANT ALL PRIVILEGES ON movielens.* TO 'root'@'localhost' IDENTIFIED BY 'hadoop';
    exit
  ```

- Import data from MySQL into HDFS
  ```bash
    sqoop import --connect jdbc:mysql://localhost/movielens --driver com.mysql.jdbc.Driver --table movies -m 1 --username root --password hadoop
  ```

- Import data from MySQL into Hive
  ```bash
    sqoop import --connect jdbc:mysql://localhost/movielens --driver com.mysql.jdbc.Driver --table movies -m 1 --username root --password hadoop --hive-import
  ```

- Import data from Hive to MySQL
    - Login to MySQL (password is `hadoop`)
      ```bash
      mysql -u root -p
      ``` 
    - Select the database and create a table
      ```sql
        use movielens;
        CREATE TABLE exported_movies (
            id INTEGER,
            title VARCHAR(255),
            releaseDate DATE
        );
        exit
        ```
    - Export the data from Hive to MySQL
      ```bash
      sqoop export --connect jdbc:mysql://localhost/movielens -m 1 --driver com.mysql.jdbc.Driver --table exported_movies --export-dir /apps/hive/warehouse/movies --input-fields-terminated-by '\0001' --username root --password hadoop
      ```