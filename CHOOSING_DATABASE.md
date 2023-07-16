# Choosing a database

## Integration considerations

- What systems are already in place?
    - Are there out-of-the-box connectors available for the existing systems?
- Are you moving data from an existing SQL database to a non-relational database?
    - It would be nice to have a non-relation database that offers SQL-like querying capabilities

## Scalability requirements

- Do you need to scale horizontally (add more nodes to the cluster)?
    - What is the transaction rate? Is it high?
        - If so, you need distributed NoSQL databases instead of monolithic relational databases

## Support considerations

- Do you have the expertise to set up and support the database?
    - If not, you may want to consider a managed database service (e.g. MongoDB)

## Budget considerations

- Apart from support, what are the other costs?
    - Do you need to pay for licenses?
    - Do you need to pay for cloud services?
    - Do you need to pay for additional hardware?
    - Do you need to pay for additional software?

## CAP considerations

- Do you need consistency?
- Do you need availability?
- Do you need partition tolerance?

*In the past you had to choose 2 out of 3, but lately there have been some progress in this area.*

### Databases

- MySQL: CA
- MongoDB: CP
- HBase: CP
- Cassandra: AP

## Simplicity

- Keep it as simple as possible
- Avoid unnecessary complexity by deploying a whole new system when you don't need to

## Examples

### Internal phone directory appli

- Scale: limited
- Consistency: Eventually consistent
- Availability: not mission critical
- MySQL is probably already in place

### Mine web server logs for interesting patterns

e.g. What are the most popular times of the day for accessing the website? What's the average session duration?

-> It can be solved just by importing the logs into Hadoop and running a Spark job, or Hive or Pig or even Tableau.

### You have a big Spark job that produces movie recommendations for end users nightly

- Something needs to vend this data to a web application
- You work for a huge company with massive scale
- Downtime is not acceptable
- Must be fast
- Eventually consistent is OK

-> Cassandra is a good choice because it's fast, eventually consistent, and it's easy to scale.

### You are building a massive stock trading system

- Consistency is critical
- Big data is present
- It's really important, so having access to professional support is a good idea

-> MongoDB is a good choice because it's consistent, it's good for big data, and it's easy to get support.
-> HBase is also a good choice because it's consistent, it's good for big data, and you can get support from external
vendors.