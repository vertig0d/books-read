![Book Cover](https://learning.oreilly.com/covers/urn:orm:book:9781098165826/400w/)

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

### Duplicate Records
#### Windowed Deduplicator

##### 1. Core Concepts & Scope
* **Goal:** Ensure data logic processes each unique record exactly once.
* **Step 1:** Identify deduplication attributes (unique keys).
* **Step 2:** Define the deduplication scope.
    * **Batch:** Typically limited to the currently processed dataset. Extending to past datasets requires more compute and slows down processing.
    * **Streaming:** Unbounded data is handled by creating **time-based windows**.

##### 2. Implementation Strategies
* **Batch Pipelines:** * Relies on standard SQL operations.
    * Uses `DISTINCT` expressions or `WINDOW` functions paired with `row_number()`.
* **Streaming Pipelines:** * More complex logic due to unbounded data.
    * Requires a **State Store** to remember previously processed records over a specific window duration.

##### 3. State Store Types (Streaming)
| Type | Location | Pros | Cons |
| :--- | :--- | :--- | :--- |
| **Local** | Memory-only | Fastest performance. | State is lost upon failure (not for production). |
| **Local + Fault-Tolerant** | Memory + Remote Backup | Fast access with fault tolerance. | Cost to processing time/consistency during persistence. |
| **Remote** | Remote DB/Store | Natively fault-tolerant. | Higher latency and overall pipeline cost. |

##### 4. Trade-offs & Limitations
* **Processing vs. Delivery:** Exactly-once *processing* does not guarantee exactly-once *delivery*. 
* **Space vs. Time Trade-off:** Keeping all data indefinitely is impossible. Systems use time-based windows to look for duplicates only within a specified period to save space.
* **Idempotent Producers:** Deduplication alone doesn't prevent downstream duplicates caused by transient errors or automatic retries.

##### 5. Tooling: Apache Spark & SQL
* **Batch:** Use Spark's `dropDuplicates` function or SQL `WINDOW` functions.
* **Streaming:** Use `dropDuplicates` mapped to a time-based column.
    * **Watermarks:** Crucial for streaming state management.
        * *Function 1:* Sets the late data boundary (ignores data older than the watermark).
        * *Function 2:* Clears old deduplication keys from the state store to free up memory.

---

##### Visual Flow Diagrams

**Batch Deduplication Flow**
```text
[Input Dataset] 
      │
      ▼
[Identify Keys] ──► (Compare against CURRENT dataset)
      │
      ▼
[Apply DISTINCT / WINDOW + row_number()]
      │
      ▼
[Output Unique Records]
```

**Streaming Deduplication Flow**
```text
[Continuous Input Stream] 
      │
      ▼
[Extract Key & Event Time]
      │
      ▼
[Check STATE STORE within Watermark Window]
      │
      ├─────► (If Key Exists) ──► [Drop Record]
      │
      ▼
[Record is New] ──► (Save Key to State Store)
      │
      ▼
[Process & Output]
```

### Late Data Detection

#### 1. Late Data Detection & Watermarks
* **Core Concept:** You must track **Event Time** (when the action happened), not processing time, to detect late data.
* **Tracking Strategies (Event Time):** The tracked time must be *monotonically increasing* (never go backward).
    * *Partition-Level:* Always use `MAX(event time)`. Using `MIN` causes infinite state growth ("stuck in the past") or forces reopening completed states ("open-close-open loop").
    * *Global-Level:* Use `MIN` to follow the slowest dependency (maximizes data, increases buffer size) or `MAX` to follow the fastest (reduces buffer, but risks dropping data in skewed networks).
* **Watermarks:** Defines the boundary for "on-time" data. 
    * *Formula:* `Watermark = MAX(event time) - allowed lateness`.
* **Tooling:** Apache Spark automatically ignores late events but makes capturing them hard. Apache Flink allows easy capture and redirection of late data using the current watermark and a timestamp assigner.

#### 2. Static Late Data Integrator
* **Concept:** Uses a fixed lookback window (e.g., last 14 days) to routinely reprocess past partitions alongside the current run.
* **Limitation:** Any late data arriving outside this fixed window is permanently ignored. Requires time-based partitions.
* **Execution:** Stateful pipelines must run sequentially (late data first). Stateless pipelines can run in parallel or prioritize current data.

#### 3. Dynamic Late Data Integrator
* **Concept:** Integrates *only* the specific partitions that actually received late data, using a state table to compare `Last update time` against `Last processed time`.
* **Metadata Sources:** Tools like BigQuery (`INFORMATION_SCHEMA.PARTITIONS`) or Iceberg natively provide these last-modified timestamps.
* **Concurrency Management (The Race Condition):**
    * *The Problem:* In concurrent systems, multiple pipelines might read the state table simultaneously and process the exact same late data.
    * *The Solution:* Add an `Is_processed` boolean flag to the state table as a lock (Pipeline only processes if `Update Time > Processed Time` AND `Is_processed = False`).

#### 4. Backfilling & The Snowball Effect
* **The Snowball Effect:** Reprocessing late data forces downstream consumers to also backfill, causing massive compute spikes. Best practice is to notify consumers and let them handle their own pipelines.
* **Backfill Rule:** Never replay all historical runs individually. Simply run the most recent execution that covers the lookback window to avoid overlapped, duplicated processing.
* **Pipeline Design:** Backfilling jobs must remain inside the main pipeline to prevent overlapping execution issues.

#### 5. Static vs. Dynamic Integration
| Feature | Static Integrator | Dynamic Integrator |
| :--- | :--- | :--- |
| **Trigger Mechanism** | Fixed lookback window (e.g., 14 days) | State table changes (Update Time > Processed Time) |
| **Efficiency** | Lower (reprocesses windowed partitions blindly) | Higher (targets only changed partitions/entities) |
| **Implementation** | Simple | Complex (requires state tables, metadata, and locks) |

---

#### Visual Flow Diagrams

**Late Data Detection & Watermark Flow**
```text
[Incoming Record] ──► Extract [Event Time]
      │
      ▼
[System Tracks Time] ──► Watermark = MAX(Event Time) - Allowed Lateness
      │
      ▼
[Compare] Is Record Event Time < Watermark?
      │
      ├─────► [YES: Late] ──► (Spark: Ignore / Flink: Capture)
      │
      ▼
[NO: On Time] ──► [Process Record & Update State]
```

**Dynamic Late Data Flow with Concurrency Lock**
```text
[Pipeline Wakes Up]
      │
      ▼
[Check State Table] ──► Update Time > Processed Time AND Is_processed = False?
      │
      ├─────► (NO / Locked) ──► [Skip Partition]
      │
      ▼
[YES: Target Identified]
      │
      ▼
[Lock Partition] ──► Update State Table: Set Is_processed = True
      │
      ▼
[Process Late Data] ──► Backfill or Overwrite impacted entities
      │
      ▼
[Unlock & Update] ──► Set Processed Time = NOW, Set Is_processed = False
```

### Fault Tolerance & Checkpointing Study Notes

#### 1. Core Concepts
* **Fault Tolerance:** Protection that ensures continuous data processing workflows (streaming) can recover from failures without data loss or duplication.
* **The Challenge:** Streaming data often arrives in an append-only log without the organizational structure of batch data. If a job stops, the system must know exactly where to resume to avoid reprocessing old data or missing new data.
* **The Checkpointer Pattern:** Solves this by persistently recording the job's progress (both the position in the data source and the computed state) outside of the job's transient environment.

#### 2. Where Checkpoints are Stored
Checkpoint data can be managed at two different levels, depending on your architecture:
| Storage Type | Description | Examples |
| :--- | :--- | :--- |
| **Framework-based** | Progress metadata is managed by the data processing framework and stored in a resilient object store. | Apache Spark Structured Streaming, Apache Flink |
| **Data store-based** | Progress is tracked by interacting directly with the data store's SDK and saved within the data store's ecosystem. | Apache Kafka (saves to `__consumer_offsets` topic), Amazon Kinesis (uses KCL to save to DynamoDB) |

#### 3. How Checkpoints are Executed
| Execution Type | Description |
| :--- | :--- |
| **Configuration-Driven** | You define the frequency, and the framework automatically handles the rest. (e.g., Spark, Flink). |
| **Intentional (Manual)** | Your code explicitly confirms/commits the checkpoint after successfully reading and processing records. |

#### 4. Trade-offs: Latency vs. Delivery Guarantee
The biggest drawback of checkpointing is the latency it introduces.
* **Position vs. State Tracking:** Tracking the *position* (a few numbers/offsets) is lightweight and fast. Tracking the *state* (e.g., complex user session data) is heavy and significantly impacts latency.
* **Frequency Trade-off:** You must balance your latency requirements against your recovery needs.

| Checkpoint Frequency | Pros | Cons |
| :--- | :--- | :--- |
| **High Frequency (More often)** | Less data to reprocess in the event of a failure. | Slower job performance due to constant metadata saving overhead. |
| **Low Frequency (Less often)** | Faster job performance (less overhead). | More data to reprocess if a failure occurs. |


[Top](#Index)

# idempotency Design Patterns

* **Idempotency:** A guarantee that no matter how many times a data processing job is executed, the final output remains consistent (yielding zero duplicates, or at least clearly identifiable ones).

## OverWriting

### Fast Metadata Cleaner

* **Metadata Operations:** Operations that happen on the *logical* level rather than the *physical* level. Because they only modify the tiny layer describing the data files (rather than scanning the massive data files themselves), they are exceptionally fast.

#### 1. Data Removal Operations
When trying to achieve idempotency, the way you clear out old or failed data matters immensely for performance.

| Operation | Level | Description & Performance |
| :--- | :--- | :--- |
| **DELETE** | Physical | Slow on large volumes. It is a two-step action: it must scan to identify specific rows, then overwrite the actual data files. |
| **TRUNCATE TABLE** | Logical (Metadata) | Extremely fast. Removes all records exactly like a `DELETE` without conditions, but completely skips the expensive table scan. |
| **DROP TABLE** | Logical (Metadata) | Extremely fast. Completely removes the table definition and its data. |

#### 2. The Fast Metadata Cleaner Pattern
* **Concept:** Uses `DROP TABLE` or `TRUNCATE TABLE` as the foundational building blocks to rapidly clear out data before re-running a job, ensuring a clean, idempotent slate.
* **Mechanism:** Relies heavily on exact dataset partitioning and data orchestration to work correctly.

#### 3. Limitations & Challenges
* **Granularity Restraints:** The partitioning strategy directly locks in your idempotency and backfilling granularity. 
* **The "All-or-Nothing" Partition Rule:** If your data is partitioned weekly, but you only need to fix one broken day, you are forced to `TRUNCATE`/`DROP` and rerun the entire week. (Though you can optimize this by only re-running the heavy logic for the broken day, and just doing a lightweight data load for the remaining valid days).
* **No Fine-Grained Cleanups:** Because `TRUNCATE` and `DROP` operate on whole tables/partitions, you cannot use this pattern to backfill a single user or a specific data provider.
* **Schema Evolution:** If a new optional field is added to a table, using this pattern would force a full, resource-heavy data reprocessing just to update the schema. Schema updates are better handled by isolated, dedicated pipelines.


### Data Overwrite Pattern 

#### 1. Core Concept
* **When to use:** Use this pattern when metadata operations (like `TRUNCATE` or `DROP`) are unavailable—such as in certain object stores—or when using the metadata layer requires too much effort. 
* **Mechanism:** Instead of logical metadata manipulation, this pattern interacts directly with the physical data layer to clean out existing files before writing new ones.

#### 2. Implementation Strategies

**Framework-Based (Configuration-Driven)**
Data processing frameworks often handle the heavy lifting of cleaning before writing through simple configurations.
* **Apache Spark:** Controlled via `save mode` configurations.
* **Apache Flink:** Controlled via `write mode` properties.
* **Delta Lake:** Offers advanced *selective* overwriting. You can target specific parts of a dataset using the `replaceWhere` option.

**Direct SQL Approaches**
If you are interacting directly via SQL, you generally choose between two methods:
| Approach | Description | Flexibility |
| :--- | :--- | :--- |
| **`DELETE FROM` + `INSERT INTO`** | A classic two-step database operation. | **High:** Supports conditional filtering so you can select exact rows to overwrite. |
| **`INSERT OVERWRITE`** | A concise, single-command alternative to the above. | **Low:** Replaces the *entire* table blindly; lacks row-level selection support. |

#### 3. Drawbacks & Mitigations
* **Performance Degradation (The Scaling Problem):** Because this is a physical data operation, overwriting large, unpartitioned datasets becomes progressively slower over time as the data volume naturally grows.
    * *Mitigation:* Apply **partitioning** to your datasets. This isolates the replacement action to smaller, specific chunks of data rather than the entire historical volume.
* **"Dead Rows" & Storage Bloat:** In relational databases and modern table file formats, a `DELETE` operation doesn't usually wipe the data from the disk immediately. Instead, it marks the data blocks as inaccessible to `SELECT` queries, leaving "dead rows" behind.
    * *Mitigation:* You must schedule and run a **vacuum process** periodically to permanently delete these dead rows and actually reclaim the underlying disk space.

## Updates

### The Challenge with Updated Incremental Datasets

#### Why Full Replacement Doesn't Always Work

For some datasets, achieving idempotency by deleting and reloading the entire dataset is straightforward.

However, **updated incremental datasets** behave differently:

- Each delivery contains only the records that changed.
- The complete dataset is not available in every run.
- Rewriting the entire dataset requires reconstructing the latest version of every entity first.
- Backfills and reprocessing become more complex.

#### Example

| Run | Records Received |
|------|------|
| Day 1 | User 1, User 2, User 3 |
| Day 2 | User 2 updated |
| Day 3 | User 1 updated |

The source only sends changes, not the full dataset.

---

### Pattern: Merger

#### Purpose

The **Merger Pattern** is used when only incremental changes are available and must be combined with an existing dataset.

Instead of replacing the entire dataset:

1. Read incoming changes.
2. Match them against existing records.
3. Update existing rows.
4. Insert new rows.

This is commonly implemented using:

- **MERGE**
- **UPSERT**

operations.

---

#### How It Works

#### Step 1: Define Merge Keys

Identify attributes that uniquely identify a record.

Examples:

| Merge Key Type | Example |
|------|------|
| Single Column | User ID |
| Composite Key | Customer ID + Order ID |

The selected attributes must uniquely identify a row.

---

#### Step 2: Perform Merge

Incoming data is merged into the target dataset.

Pseudo Logic:

```sql
MERGE INTO target_table
USING source_table
ON target_table.user_id = source_table.user_id

WHEN MATCHED THEN
    UPDATE ...

WHEN NOT MATCHED THEN
    INSERT ...
```

---

#### Critical Requirement: Uniqueness

##### Why It Matters

The merge process depends entirely on stable and immutable identifiers.

Without uniqueness:

- Updates may become inserts.
- Duplicate records may appear.
- Backfills may create inconsistent data.

##### Good Candidate Keys

- User ID
- Customer ID
- Order ID
- Business-generated immutable identifiers

##### Bad Candidate Keys

- Name
- Email (can change)
- Address
- Mutable business attributes

---

#### Performance Characteristics

##### I/O Behavior

The Merger Pattern is:

- Data-driven
- Compute-intensive
- More expensive than metadata-only approaches

Since data must be inspected and updated, storage reads and writes increase.

##### Modern Optimizations

Modern systems reduce unnecessary reads by using metadata.

Examples:

- Delta Lake
- Apache Iceberg
- Apache Hudi
- Modern data warehouses

Optimization process:

1. Identify impacted records using metadata.
2. Read only relevant files.
3. Skip unaffected data blocks.

This significantly reduces I/O.

---

#### Merger Pattern Trade-Offs

| Advantage | Disadvantage |
|------------|------------|
| Supports incremental updates | Requires unique keys |
| Avoids full reloads | Higher I/O cost |
| Efficient for small changes | Backfills can become inconsistent |
| Widely supported | More complex than truncate-and-load |

---

### Pattern: Stateful Merger

#### Why Stateful Merger Exists

The standard Merger Pattern focuses only on applying updates.

Problem:

- Backfills may overwrite data incorrectly.
- Dataset consistency can be compromised.
- Previous states are not easily restored.

The **Stateful Merger Pattern** addresses this by maintaining additional state information.

---

#### Core Idea

A separate **state table** tracks dataset versions.

This allows:

- Detection of backfills
- Restoration of previous dataset versions
- Consistent reprocessing

---

#### High-Level Workflow

```text
Pipeline Start
      |
      v
Check State Table
      |
      v
Backfill?
  /      \
Yes       No
 |          |
Restore    Continue
Dataset    Normally
 |          |
 v          v
MERGE New Data
      |
      v
Update State Table
      |
      v
Pipeline End
```

[Image of Stateful Merger Workflow]

---

#### Additional Components

##### State Table

Stores metadata such as:

| Field | Purpose |
|---------|---------|
| Dataset Version | Current table version |
| Execution Time | Pipeline run timestamp |
| Previous Version | Version to restore if needed |

---

#### Backfill Detection Logic

##### Option 1: Orchestrator-Based Detection

If the orchestration platform exposes execution context:

- Read pipeline metadata.
- Determine whether execution is:
  - Normal run
  - Backfill run

Examples:

- Airflow
- Databricks Workflows
- Azure Data Factory
- Dagster

---

##### Option 2: State Table-Based Detection

If no execution context exists:

Use dataset versions stored in the state table.

---

#### Detection Scenario 1: First Execution

If no previous version exists:

```sql
TRUNCATE TABLE target_table;
```

Then:

- Load data normally.
- Update state table.

This occurs when:

- Pipeline runs for the first time.
- First execution is being backfilled.

---

#### Detection Scenario 2: Normal Run

Compare:

- Current version
- Previous execution version

Example:

| Execution | Version |
|------------|------------|
| Previous Run | 6 |
| Current Run | 6 |

Result:

- Versions match.
- No restoration required.
- Proceed directly to merge.

---

#### Detection Scenario 3: Backfill Run

Compare:

| Execution | Version |
|------------|------------|
| Previous Version | 4 |
| Latest Version | 6 |

Result:

- Versions differ.
- Indicates backfill execution.
- Dataset must be restored before merge.

---

#### Restore + Merge Process

```text
Detect Backfill
      |
      v
Restore Dataset
to Previous Version
      |
      v
Apply MERGE
      |
      v
Update State Table
```

This guarantees consistency during historical reprocessing.

---

#### Systems Without Table Versioning

Some storage systems do not support native table versioning.

Examples:

- Traditional databases
- Custom storage layers

##### Alternative Approach

Maintain a raw data table containing:

| Column | Purpose |
|----------|----------|
| Execution Time | Pipeline execution timestamp |
| Raw Data | Original records |

---

#### Alternative Backfill Detection

Instead of checking table versions:

Check whether future execution records already exist.

Example:

```text
Current execution = 09:00

Raw table contains:
10:00 data
11:00 data
12:00 data
```

This indicates:

- Data for future executions already exists.
- Current run is likely a backfill.

---

#### Stateful Merger Trade-Offs

| Advantage | Disadvantage |
|------------|------------|
| Handles backfills safely | Additional state table required |
| Supports restoration | More pipeline complexity |
| Maintains consistency | Extra storage overhead |
| Better historical correctness | Additional version management |

---

#### Interview Notes

##### When to Use Merger

Use when:

- Source provides incremental updates.
- Unique keys exist.
- Occasional inconsistency during backfills is acceptable.

##### When to Use Stateful Merger

Use when:

- Historical correctness is critical.
- Backfills are frequent.
- Dataset restoration is required.
- Regulatory or audit requirements exist.

##### Key Interview Question

**Why isn't MERGE alone sufficient for backfills?**

Answer:

A MERGE operation only applies incoming changes. It does not restore the dataset to the correct historical state before applying those changes. During backfills, this can lead to inconsistent results. A Stateful Merger solves this by restoring the appropriate dataset version before performing the merge.

## Keyed Idempotency

### Problem

A pipeline may retry due to:

- Failures
- Restarts
- Backfills
- Executor crashes

Without protection, the same logical record may be written multiple times, creating duplicates.

**Goal:**

> Multiple executions should produce the same final result.

---

### Core Idea

Use a database that supports keys and generate a **deterministic key** for every record.

If the same record is processed again:

- Same key is generated
- Same row is targeted
- Existing value is overwritten/replaced

```text
Same Input
    ↓
Same Key
    ↓
Same Row
```

---

### Mental Model

Instead of making the write operation idempotent:

Make the **key generation** idempotent.

```text
Idempotency = Deterministic Key Generation
```

---

### Example

User session generation:

```text
User A visits site
↓
Session created
↓
Write session to database
```

If the job retries:

```text
Generate same session key
↓
Write again
↓
Existing row replaced
```

Result:

```text
1 session
NOT 2 sessions
```

---

### Golden Rule

Use **immutable attributes** for key generation.

#### Good Candidates

- User ID
- Order ID
- Transaction ID
- Append Time / Ingestion Time

#### Bad Candidates

- Event Time
- Last Updated Time
- Any field affected by late-arriving data

---

### Why Event Time Is Dangerous

Suppose a session key is generated using:

```text
User ID + First Event Time
```

Original run:

```text
10:00
10:05
```

After restart, a late event arrives:

```text
09:55
10:00
10:05
```

Now the first event changes from:

```text
10:00
```

to:

```text
09:55
```

Result:

- New session key generated
- Existing record not found
- Duplicate session created
- Idempotency broken

---

### Interview Soundbite

> Event time is mutable from the pipeline's perspective because late-arriving data can change it. Prefer append time or ingestion time when generating idempotent keys.

---

### Where It Works Best

#### Key-Value Databases

Examples:

- Cassandra
- ScyllaDB
- HBase

Behavior:

```text
Write same key twice
↓
Latest value replaces old value
```

Perfect fit for Keyed Idempotency.

---

### Limitations / Gotchas

#### 1. Database Dependent

Relational databases do not automatically overwrite records with the same key.

This fails:

```sql
INSERT INTO table (...)
VALUES (...);
```

because it causes:

```text
Primary Key Violation
```

Instead use:

```sql
MERGE
UPSERT
ON CONFLICT
```

---

#### 2. Kafka Is Different

Kafka supports keys but is an append-only log.

```text
Kafka Key = Identifier
NOT Immediate Deduplication
```

Example:

```text
Key=A Value=1
Key=A Value=1
```

Both records may exist temporarily.

Log compaction later removes older versions.

Therefore:

```text
Kafka keys help identify duplicates
but do not prevent them immediately.
```

---

#### 3. Mutable Sources Can Break Idempotency

If key generation depends on values that can change:

- Late data
- Retention cleanup
- Compaction effects
- Reordered inputs

then the generated key may differ between runs.

Result:

```text
Different Key
↓
Different Row
↓
Duplicate Business Record
```

---

### When To Use

Use Keyed Idempotency when:

✅ Output store supports unique keys

✅ Stable deterministic keys can be generated

✅ Retries are common

✅ Backfills are common

✅ Latest-value semantics are acceptable

Typical use cases:

- User profiles
- Session tables
- Device status tables
- Customer state tables
- CDC state stores

---

### One-Line Summary

> Keyed Idempotency guarantees repeatable writes by generating the same deterministic key from immutable attributes, allowing key-based databases to overwrite the same logical record instead of creating duplicates.

---

### Interview Memory Hook

```text
Retry Safe =
Stable Key
+
Immutable Attributes
+
Key-Based Storage
```

## Transactional Writer

### Problem

A job may fail after writing only part of its output.

Example:

```text
Task writes 1,000 records
↓
Infrastructure failure
↓
Task retries
↓
Writes same 1,000 records again
```

Result:

- Duplicate records
- Partial datasets visible to consumers
- Inconsistent outputs

The challenge is not just duplicates.

The bigger problem is:

> Consumers can see incomplete data while the job is still running.

---

### Core Idea

Use database transactions so that data becomes visible only after the entire write succeeds.

```text
Begin Transaction
        ↓
Write Data
        ↓
Commit
        ↓
Consumers Can See Data
```

If anything fails:

```text
Begin Transaction
        ↓
Write Data
        ↓
Failure
        ↓
Rollback
        ↓
No Data Visible
```

---

### Mental Model

Think of a transaction as a **private workspace**.

```text
Without Transaction

Write Row 1 → Visible
Write Row 2 → Visible
Write Row 3 → Failure

Consumer sees:
Row 1
Row 2
```

```text
With Transaction

Write Row 1 → Hidden
Write Row 2 → Hidden
Write Row 3 → Hidden
Commit

Consumer sees:
Row 1
Row 2
Row 3
```

Everything appears together.

Or nothing appears at all.

---

### Why It Helps Idempotency

Retries become safer.

Without transactions:

```text
Attempt 1
---------
Write A
Write B
Failure

Attempt 2
---------
Write A
Write B
Write C
```

Final output:

```text
A
B
A
B
C
```

Possible duplicates.

With transactions:

```text
Attempt 1
---------
Write A
Write B
Failure
Rollback

Attempt 2
---------
Write A
Write B
Write C
Commit
```

Final output:

```text
A
B
C
```

Only the successful transaction becomes visible.

---

### High-Level Workflow

```text
START TRANSACTION
        ↓
Write Data
        ↓
Success?
   /        \
 Yes         No
  ↓           ↓
COMMIT    ROLLBACK
```

---

### Key Principle

Transactions provide:

**All-or-Nothing Semantics**

```text
100% Success → Publish Everything

Anything Fails → Publish Nothing
```

---

### Interview Soundbite

> Transactional Writer achieves idempotency by ensuring partial writes are never exposed. Consumers see either the entire successful output or no output at all.

---

### Where It Works Best

Databases with transaction support:

- PostgreSQL
- MySQL
- SQL Server
- Oracle
- Snowflake
- BigQuery
- Redshift
- Delta Lake
- Apache Iceberg

---

### Typical Use Cases

#### Batch Loads

```text
Load 100 million records
↓
Expose only when complete
```

---

#### ELT Pipelines

```text
Transform data in warehouse
↓
Commit results atomically
```

---

#### Incremental Loads

```text
Apply updates
Apply inserts
Apply deletes
↓
Commit together
```

---

### Limitations / Gotchas

#### 1. Database Support Required

Not every storage system supports transactions.

Works well:

- Delta Lake
- PostgreSQL
- Snowflake

Less suitable:

- Raw CSV files
- Plain object storage
- Traditional append-only logs

---

#### 2. Transactions Do Not Eliminate Duplicate Logic

Transaction:

```text
Protects Visibility
```

Not:

```text
Business Deduplication
```

If the same business record is written twice in two separate successful transactions:

```text
Transaction #1 → Commit
Transaction #2 → Commit
```

Duplicates can still exist.

You often combine:

- Transactional Writer
- MERGE / UPSERT
- Keyed Idempotency

---

#### 3. Long Transactions Can Be Expensive

Large transactions may:

- Hold locks
- Consume memory
- Increase contention
- Reduce concurrency

Example:

```text
1 minute transaction → Fine

3 hour transaction → Risky
```

---

### Comparison With Keyed Idempotency

| Aspect | Keyed Idempotency | Transactional Writer |
|----------|----------|----------|
| Main Goal | Prevent duplicate business records | Prevent partial writes |
| Relies On | Deterministic keys | Database transactions |
| Protects Against | Retries creating duplicates | Failures during writes |
| Typical Storage | Cassandra, HBase, ScyllaDB | PostgreSQL, Delta Lake, Snowflake |
| Guarantees | Same record targets same key | All-or-nothing visibility |

---

### Real-World Analogy

Online Banking Transfer:

```text
Debit Account A
Credit Account B
```

Without transaction:

```text
Debit succeeds
Credit fails
```

Money disappears.

With transaction:

```text
Debit succeeds
Credit succeeds
COMMIT
```

or

```text
Debit succeeds
Credit fails
ROLLBACK
```

No inconsistent state.

A Transactional Writer applies the same idea to data pipelines.

---

### One-Line Summary

> Transactional Writer uses database transactions to guarantee that consumers never see partial results, exposing data only after a successful commit.

---

### Interview Memory Hook

```text
Transactional Writer =
BEGIN
+
WRITE
+
COMMIT

Failure
↓
ROLLBACK
```

Remember:

```text
Keyed Idempotency → Same Record

Transactional Writer → Same Transaction
```




