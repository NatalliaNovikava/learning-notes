# Foundations of Data Systems
## Reliable, Scalable and Maintainable applications
- [Reliable, scalable, and maintainable applications][#Reliable, scalable and maintainable applications]
  - [Reliability][#Reliability]
  - [Scalability][#Scalability]
  - [Maintainability][#Maintainability]
 
  
   
 
### Reliable, scalable and maintainable applications
A data-intensive application are typically built up from blocks.
- store date (database)
- remember the result of expensive computation (caches)
- allow users to search by key words (indexes)
- send a message to another process (stream processing)
- periodically crunch a large amount of data (batch processing)
- Reliability. To work correctly even in the face of adversity
- Scalability. Reasonable ways to deal with growth
- Maintability. People should be able to work on the system productively.

### Reliability
Typical expectations:
- The application performs the function that the user expected
- It can tolerate the user mistakes and using software in unexpected ways
- It's performance is good enough for the required use case, under the expected load and data volume
- The system prevents any unauthorized access and abuse


The things that can go wrong are called **faults**. 
And system that anticipate faults and can cope with them is called **fault-tolerant** or **resilient**.
**Fault** is usually defined as one component of systems deviating from its spec, whereas **failure** is when the system stops providing the service.

You should generally prefer tolerating faults over preventing faults.

* **Hardware Faults**
The first response is to add redundancy to individual components in order to reduce the failure rate of the system.
There is a move toward systems that can tolerate the loss of entire machines, by using software fault-tolerance techniques in preference or in addition to hardware redundancy.

* **Software Errors**
   - software bug that causes every instance of an application server to crash
   - a runaway process that uses up some shared resources : CPU time, memory, disk space etc
   - service that the system depends on slows down or becoming unresponsive
   - cascading failures

* **Human Errors**
   - minimize opportunities for errors
   - provide fully featured non-production sandbox environment
   - unit tests, integration test, manual and automated tests
   - allow quick and easy recovery from human errors
   - set up monitorings such as performance metrics and error rates

### Scalability
Scalability is the term we use to describe a system's ability to cope with increased load.
* **Describing load**
Load parameters depend on the architecture of your system:
  - request per sec to a web server
  - the ratio of reads to writes in a database
  - the number of simultaneously active users in a chat room
  - the hit rate on a cache
  - average case or bottleneck 
  
Twitter example:
Post tweet: A user can publish a new message to its followers (4.6k req/sec on average, 12k at peak)
Home timeline: A user can view tweets of the people they follow (300k req/sec)

Twitter's scaling challenge is not primarily due to tweet volume, but due to fan-out.\
Each user follows many people.\
And each user is followed by many people.

**Three ways**:\
1 posting a tweet simply inserts the new tweet into a collection of tweets.When user requests home timeline look up all the people they follow,
find all their tweets and merge them (sorted by time).

    SELECT tweets.*, users.*
    FROM tweets
    JOIN users ON tweets.sender_id = users_id
    JOIN follows ON follows.followee_id = users_id
    WHERE follows.follower_id = current_user

2 Maintain a cache for each user's home timeline. When a user posts a tweet, look up all the people the user follows\
and insert tweet into each of their cache.
Downside of this approach that posting requires now a lot of extra work. Each tweet on average is delivered to 75 followers so
4.6k write per second become 345k. Some users have over 30 mil followers.
So for Twitter the distribution of followers per user is the key load parameters for discussing scalability since it determines the fan-out load.

3 Hybrid approach. Small numbers of users with large number of followers are excepted from fan-out. Tweets from celebrities are fetched separately and merged with the home timeline when user requests.

### Describing Performance
To check how system deal with increased load:
   - increase a load parameter and keep the system's resources unchanged, how the performance is affected?
   - increase a load parameter, how much do you need to increase the resources to keep performance unchanged?

**_In batch processing (Hadoop)_** we look at the throughput - the number of records we can process per seconds
Or total time it takes to run a job on the dataset.\
**_In online systems_** service's response time - the time between client sending request and receiving response.

**Response time** includes: actual time to process a request, network delays and queueing delays.\
Latency is the time request is waiting to be handled  - during which it is latent, awaiting service.

Average response time (arithmetic mean) is not very good metric.

_**Percentile.**_\
If you take a list of response times and sort from fastest to slowest, then the median means that half of the users is served with less than median response time.
The median is 5oth percentile - p50.

Common higher percentiles p95, p99, p99.9 are called tail latency.
p99.9 affects only 1 in 1000 requests.\
Percentiles are used in service level objectives SLOs and service level agreements SLAs.

Queueing delays often account for a large part of the response time at high percentile.
So it is important to measure response time on customer side.

To monitor percentile you need rolling window of response times of request in the last 10 min.

### Approaches for Coping with Load
**_Scaling up_** - vertical scaling, moving to a more powerful machine.\
**_Scaling out_** - horizontal scaling, distributing the load across multiple smaller machine (Shared-nothing architecture)
Elastic system can automatically add computing resources when they detect a load increase.
They are useful if the load is unpredictable.

An architecture that scales well for a particular application is built arround assumptions of which
operations will be common and which will be rare.

### Maintainability
- Operability - make it easy for  operation team to keep the system running smoothly
- Simplicity - make it easy for new engineers to understand the system
- Evolvability - also known as extensibility, modifiability,plasticity

## Data Model And Query Languages
#### Relational Model Versus Document Model
Relational data model is most well-known now was proposed in 1970 by Edgard Codd.
Data is organized in relations(called tables in SQL), where each relation is unordered collection of tuples(rows).
By the mid-1980s RDBMSes and SQL had become the tools of choice for most people.

The roots lie in business data processing:
 - transaction processing (banking transactions, airline reservation etc)
 - batch processing(customer invoicing, payroll, reporting)

Alternatives:
 - network model
 - hierarchical model
 - object databases
 - XML databases

#### The Birth Of NoSQL

It was catchy hashtag that retroactively reinterpreted as Not Only SQL.

#### The Object Relational Mismatch
**_Object-relational mapping (ORM)_** frameworks ActiveRecord and Hibernate reduce the amount of boilerplate code, but they can't completely hide the difference between the two models: objects in the application code and the database model of tables, rows and columns.

Databases that support JSON format:
- MongoDB, RethinkDB, CouchDB,Espresso\
The **one-to-many** relationships imply a tree structure and JSON makes this tree structure explicit.
  
#### Many-to-One and Many-To-Many Relationships.

**_Normalization_** in db is removing duplication.\
As a rule of thumb if you're duplicating values that could be stored in just  one place, the schema is not normalized.

Normalizing **many-to-one** relationships doesn't fit nicely into document model.\
In relational db it's normal to refer to rows in other tables by id. In document db joins are not needed for one-to-many tree structures and support for joins is often weak.

#### Are Document Databases Repeating The History?
**Many-to-many** relationships  and joins routinely used in relational db, document db and NoSQL reopened a debate.

**_Information Management System_** IMS - the most popular db in 1970. It uses hierarchical model that has remarkable similarities with the JSON.
IMS worked well with one-to-many relationships, but it made many-to-many relationships difficult and doesn't support joins.
Developers had to decide to denormalize data or perform join in application code.

The proposed solutions were:
  - relational model(became SQL)
  - network model

**_The Network Model_**

CODASYL model (Conference on Data System Languagues) - network model

_**Hierarchical model:**_
   - every record has exactly one parent

_**Network model:**_
   - record could have multiple parents
   - links between records like a pointers in programming languages
   - to access the record you need to follow the path from a root record(access path)
   - the code for querying and updating the db is very complicated

**_The relational model_**
- relations(tables) are collections of tuples(rows)
- query optimizer decides which part of the query to execute in which order and which indexes to use

**_Comparison to document database_**

**one-to-many**
- Document db reverted back to hierarchical model in storing nested records within their parent record.

**many-to-one and many-to-many**
- Document db like relational db: the related item is referenced by unique id which is called
- foreign key in relational db
- document reference in document db

Relational vs Document databases today

The differences in the data model:

Document data model:
- schema flexibility
- better performance due to locality

Relational Model:
- better support for joins
- many-to-one and many-to-many relationships

For highly interconnected data:
- document model is akward
- relational is acceptable
- graph is the most natural

**Schema flexibility in the document model**
 - **_schema-on-read_** (document db)\
   the structure of the document is implicit and only interpreted when the data is read.
   It is advantageous if all items in the collection have the same structure for some reason.
 - **_schema-on-write_** (relational db)\ 
   the schema is explicit and the database ensures all written data conforms to it.\
   
Change of format of the data:
- in document db you just start writing a new format and handle in app code the documents with old format
- in relational you perform a migration.

ALTER TABLE users ADD COLUMN first_name text;
UPDATE users SET first_name = substring_index(name,' ', 1); -- MySQL
- Most relational db performs ALTER TABLE in several milliseconds.(except MySQL)
- UPDATE is likely to be slow on large table. If acceptable first_name maybe set to default NULL and fill it in at read time.

**Data locality for queries**
Documents are stored as a single continuous String, encoded as JSON, XML or binary variant thereof BSON.
The **_locality advantage_** 
 - db typically needs to load the entire document
 - locality advantage is applied if you need large parts of document  at the same time.
 - if you need small portion of it, it can be wasteful

**On updates to a document:**
 - the entire document needs to be rewritten
 - only modifications that don't increase the size of encoded document can easily be performed in place
 - recommended to keep documents fairly small

Locality in relational db:
 - Google's Spanner - allows to declare that's a table's rows should be interleaved within the parent table

Convergence of document and relational databases:
XML documents: Most relational db (other than MySQL) have support for it.
JSON support: PostgreSQL, MySQL, IBM DB2
Relational-like joins in its query language: RethinkDB 
Atomatic resolving db references (client-side join): MongoDB

Query Languages For Data
Imperative - tells computer to perform certain operation in certain order
Declarative - SQL - you just specify the pattern of data you want but not how to achieve that goal.
It is up to the db's query optimizer to decide which indexes and joins to use and in which order to execute various part of query.

Declarative Queries On Web
In web browser, using declarative CSS styling is much better than manipulating styles imperatively in JS.

MapReduce Querying
MapReduce is a programming model for processing large amount of data in bulk across many machines, popularized by Google.
A limited form of MapReduce is supported by MongoDB and CouchDB, as a mechanism for performing read-only queries across many documents.

MapReduce is neither a declarative query language nor fully imperative query API, but in between: the logic of query is expressed with snippet of code which are called repeatedly by the framework.
Two functions:
- map(collect) 
- reduce (fold or inject) 








  
  

  

  












 













[#reliability]: #reliability

[#Reliablity]: #reliability