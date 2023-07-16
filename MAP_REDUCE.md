# MapReduce

MapReduce is a data processing paradigm in Hadoop that operates at a conceptual level. It distributes data processing
across the cluster and consists of two main stages: Map and Reduce. Here are the key points:

- Distributes data processing on the cluster, enabling parallel execution.
- Divides data into partitions that are:
    - Mapped (transformed) by a defined mapper function:
        - Extracts and organizes data, associating it with a certain key value.
    - Reduced (aggregated) by a defined reducer function:
        - Aggregates the data based on the keys.
- Resilient to failure, allowing for fault tolerance in distributed environments.

## MapReduce Conceptual Example: MovieLens Dataset

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

## MapReduce hands-on Example: Movie Ratings by Rating Score

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

## Installing MRJob in HDP 2.6.5

To install MRJob in HDP 2.6.5, follow these steps:

```bash
sudo yum-config-manager --save --setopt=HDP-SOLR-2.6-100.skip_if_unavailable=true
sudo yum install https://repo.ius.io/ius-release-el7.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install python-pip
sudo pip install pathlib
sudo pip install mrjob==0.7.4
sudo pip install PyYAML==5.4.1
sudo yum install nano
```

## Running MRJob Locally

To run MRJob locally, use the following command:

```bash
python RatingsBreakdown.py u.data
```

## Running MRJob with Hadoop

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