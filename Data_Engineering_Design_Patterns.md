## Index
1. [Introducing Data Engineering Design Patterns](#Introducing-Data-Engineering-Design-Patterns)
2. [Data Ingestion Design Patterns](#Data-Ingestion-Design-Patterns)
3. [Error Management Design Patterns](#Error_Management_Design_Patterns)

## Introducing Data Engineering Design Patterns
#### What are Design Patterns
* a recipe is a great representation of what a design pattern should be: a predefined and customizable template for solving a problem.
* The ingredients and the list of preparation steps are the __predefined template__. They give you instructions but remain customizable.
* __contextualization of a design pattern__. Design patterns always respond to a specific problem, which in this example is the problem of how to share a pleasant dessert with friends or how to produce the dessert to generate business revenue.
* __reusability of the pattern__ You can decide to prepare this delicious dessert once or many times.
* __consequences of a pattern__ prepare it every day, you’ll maybe have less time for sports practice, and as a result, you might have some health issues in the long run.
* the recipe __saves you time__ as it has been tested by many other people before.

Example of the above:
* a continuously running job, might be processing a record with a completely invalid format that will throw an exception and stop your job. But you don’t want your whole job to fail because of that simple malformed record. This is our *contextualization*.
* To solve this  issue, you’ll apply a set of best practices to your data processing logic, such as wrapping the risky transformation with a try-catch block to capture bad records and write them to another destination for analysis. That’s the *predefined* template.
* while working on an ELT pipeline and performing the transformations in a data warehouse directly—you can apply the same logic. That’s the *reusability of the pattern*
*  the pattern has some *consequences* you should be aware of. Here, you add extra logic that adds some extra complexity to the codebase.

Note: 
* handling erroneous records without breaking the pipeline has a specific name, __dead-lettering__.
* Besides pure software aspects, you need to think about the data aspects, such as the aforementioned failure management, backfilling, idempotency, and data correctness aspects.

[Top](#Index)

## Data Ingestion Design Patterns
#### Full Load
* pattern refers to the data ingestion scenario that works on a complete dataset each time.
* Scenario where the dataset slowly evolving entity with the total number of rows not exceeding one million. Unfortunately, the data provider doesn’t define any attribute that could help you detect the rows that have changed since the last ingestion.
* simplest implementation relies on two steps, extract and load (EL). ideal for homogeneous data stores because it doesn’t require any data transformation. also known as __passthrough__ jobs because the data is simply passing through the pipeline, from the source to the destination.
* EL jobs are not always possible especially for heterogeneous databases, hence the pipeline becomes ETL. 
* __Challenges__ :
    1. *Data Volume*: example, if the dataset doubles in size from one day to another, the ingestion process will be slower and can even fail due to static hardware limitations.
    2. *Data Consistency*:  data consistency from the consumer’s perspective. What if your data ingestion process runs at the same time as the pipelines reading the dataset? Consumers might process partial data or even not see any data at all if the insert step doesn’t complete. If your data store doesn’t support transactions, you can rely on a *single data exposition abstraction*.
![Single Data Exposition](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098165826/files/assets/dadp_0201.png)

#### Incremental Load
* can be implemented in 2 ways:
    1. *Delta Column*:  identify rows added since the last run. for event-driven data like immutable visits, this column will be ingestion time. needs to remember the last ingestion time value to incrementally process new rows.
    2. *Time-partitioned datasets*: the ingestion job uses time-based partitions to detect the whole new bunch of records to ingest. 
![Incremental Load Implementation](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098165826/files/assets/dadp_0202.png)
* *Hard Deletes*: When a data provider deletes a row, the information physically disappears from the input dataset. However, it’s still present in your version of the dataset because the delta column doesn’t exist for a deleted row. To overcome this issue you can rely on soft deletes, where the producer, instead of physically removing the data, simply marks it as removed. Put differently, it uses the UPDATE operation instead of DELETE.
* *Backfilling*:  imagine a pipeline relying on the delta column implementation. After processing two months of data, you were asked to start a backfill. Now, when you launch the ingestion process, you’ll be doing the full load instead of the incremental one. Therefore, the job will need more resources to accommodate the extra rows. Mitigate the problem by limiting the ingestion window 

#### Change Data Capture
* When you need a lower ingestion latency or built-in support for the physical deletes (both not possible in Incremental load).
* A commit log is an append-only structure. It records any operations on the existing rows at the end of the logfile. The CDC consumer streams those changes and sends them to the streaming broker or any other configured output. From that point on, consumers can do whatever they want with the data, such as storing the whole history of changes or keeping the most recent value for each row.

##### Challenges

1. __Complexity__: Requires other teams involvement to implement the commit logs
2. __Data Scope__: you may be able to get the changes made only after starting the client process. If you are interested in the previous changes too, you will need to combine CDC with other data ingestion patterns
3. __Payload__: additional metadata with the records, such as the operation type (update, insert, delete), modification time, or column type.

#### Replication
* main goal of which is to copy data as is from one location to another.
* Replication is about moving data between the same type of storage and ideally preserving all its metadata attributes, such as primary keys in a database or event positions in a streaming broker. Loading is more flexible and doesn’t have this homogeneous environment constraint.

##### Passthrough Replicator
* A data provider which is not idempotent, plus the required consistency across environments, is a great reason to use the Passthrough Replicator pattern.
* Challenges: 
    1. *Security and isolation* During cross environment communication, there is a risk of negatively impacting the target environment, even to the point of making it unstable. You certainly don’t want to take that risk in production, so for that reason, you should implement the replication with the push approach instead of pull. This means that the environment owning the dataset will copy it to the others and thus control the process with its frequency and throughput.
    2. * PII Data*: replicated dataset stores personally identifiable information (PII), or any kind of information that cannot be propagated from the production environment, use the Transformation Replicator pattern, 
    3. *Latency*: infrastructure-based implementation often has some extra latency, and you should always check the service level agreement (SLA) of the cloud provider to see if it’s acceptable as the solution
    
##### Transformation Replicator
* replicated dataset contains PII data that is not accessible outside the production environment. As a result, you can’t use a simple Passthrough Replicator job. In that scenario, you should implement the Transformation Replicator pattern, which, in addition to the classical read and write parts from the Passthrough Replicator pattern, has a transformation layer in between.
* Challenges :
    1. *Transformation risk for text file formats*: example - the datetime format is different from the standard used by your data processing framework. instead of defining the timestamp columns as is, you can simply configure them as strings and not worry about any silent transformations.
    2. *Desynchronization*: Since data is evolving, the privacy fields you have today will still be valid in the future. New ones will appear or attributes that are not currently considered PII will be reclassified as PII. To avoid this one should rely on a data governance tool, such as a data catalog or a data contract in which the sensitive fields are tagged. With such a tool, you can automatize the transformation logic. Or implement the rules on your own. __Two  approaches__. First is a __data reduction approach__ that eliminates unnecessary fields.
    eg. ```SELECT * EXCEPT (ip, latitude, longitude)```
    Second, a __column-based transformation__ to alter the sensitive fields.

#### Data Compaction
Problem:
* after three months, all the batch jobs are suffering from the metadata overhead problem due to too many small files composing the dataset. As a result, they spend 70% of their execution time on listing files to process and only the remaining 30% on processing the data.

Solution
* Data Compaction solves this issue, Delta Lake employs the ```OPTIMIZE``` command. Basically runs a transactional distributed data processing job under the hood to merge smaller files into bigger ones as a part of the new commit



[Top](#Index)

## Error Management Design Patterns

#### Unprocessable Records
* They cause fatal failures that stop the data processing job. However, maintaining this fail-fast approach won’t always be possible, especially for long-running streaming jobs.

##### Dead-Letter
* you can skip the invalid events and thus lose them forever. Or you can save the bad records elsewhere for further investigation.
* The pattern starts by identifying places in the code where your job can fail. Next add some safety controls over the likely fail spots that have been identified
* Good candidates for the dead-letter stores are object stores in the cloud or streaming brokers since they’re highly available, fast, and easy to monitor.
* Dead-Letter pattern is often quoted in the context of stream processing because it allows the pipeline to run despite erroneous records
![Dead letter pattern](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098165826/files/assets/dadp_0301.png)

Challenges:
* Snowball backfilling effect: consumers will continue processing data that might be partial. Snowball backfilling effect where backfilling Pipeline 1 triggers the same process for all downstream consumers.
![Snowball backfilling effect](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098165826/files/assets/dadp_0302.png)
* Dead-lettered records identification: distinguish them from the rows added in the normal ingestion pipeline. useful to implement a filtering condition skipping replayed records in the downstream consumers or to simply track the origin of each row. add a boolean column or an attribute called was_dead_lettered to indicate each record produced by the Dead-Letter replay job.
* Error-safe functions: instead of throwing a runtime exception in case of an error, they return a NULL value. Moreover, all this logic is fully managed by your framework or database. However, these error-hiding functions make the Dead-Letter pattern implementation more challenging.

#### Duplicate Records
##### Windowed Deduplicator
















