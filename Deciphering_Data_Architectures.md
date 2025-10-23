# Deciphering Data Architectures - James Serra
![Book Cover](https://learning.oreilly.com/covers/urn:orm:book:9781098150754/400w/)

## Chapter 2 - Types of Data Architecture
* In relational database, consistency and data integrity is important and hence it follows schema-on-write.
* Schema-on-read more flexibility in storing unstructured and semi structured data. Commonly used in Data Lake.
##### Relational Data Warehouse
* Relational Data Warehouse, type of database, optimized for analytics and holding large data.
* RDW has both compute and storage. whereas cloud based warehouses have them seperate, this helps in scaling independently. This also means that in RDW since they both are tightly coupled we need to upgrade both or some hardware accompanying it.
* Important features of RDW include:
    1. Trasaction support
    2. Audit trail
    3. schema enforcement.

##### Data Lake
* A glorified filesystem, no different from file storage in laptop.
* Has no compute engine associated.

* Unlike RDW which uses relational storage, Data Lake uses object storage. Also, practices schema-on-read.
* Supports all file types, i.e. structured, semi-structured and unstructured.
* Drawback: requires advanced skills to handle.

##### Modern Data Warehouse
* Collab of RDW and Data Lake, by using both side by side.
* Data lake for staging and preparing data, RDW for serving, security and business uses.

##### Data Mesh
* Each department such as sales, finance have their own IT team that takes care of the data, instead of a centralized data dumping ground. 

## Chapter 4 - The Relational Data Warehouse
* Acts as central repo and used as _Single Source Of Truth_.
* Enterprise level data warehouse is called EDM.
* Note, DW is not a dumping ground and not a view with union or joins, instead data actually reside here.
* _Descriptive Analysis:_ used to answer the question "What happened", by using past or history data and summary statistics or visualisation.
* _Diagnostic Analysis:_ used to answer "Why something happened", by using past data and finding the relation. Usually done by changing the values and seeing the impact. eg. root cause analysis
##### Advantage of RDW
1. Optimised for read so returns result faster
2. Get data from multiple sources.
3. Get historical report
4. improve data quality by fixing issues from source, like data type, naming, standardization 
##### Disadvantages of RDW
1. Complex
2. High Cost
3. Limited flexibility
##### Populating RDW
* Many things to consider when populating the RDW, 
    1. Frequency: whether daily, hourly, weekly etc the RDW needs to fetch data from source
    2. Extract method, Incremental or Overwrite the data from source
    3. SCD type: which column to use to identify the change, can be timestamp.

## Chapter 5 - Data Lake
* Unlike RDW, supports semi and unstructured data.
* Stores data in raw format, meaning no transformations are performed. Hence, quick to load data from sources. Follows schema-on-read.
* _Predictive Analytics:_ based on the historical data predict the outcome of an event. eg. predict the customer who might become loan defaulter.
* _Prescriptive analytics:_ one step further to predictive analytics where based on the prediction some suggestions are provided. eg. predict traffic congestion and provide alternative routes.
* _Data Modeling:_ a blue print to understand data. It aids in understanding the relation between data, which data are important. 
##### Best Practices for Data Lake
* Without some rules implied the Data Lake can become a _Data Swamp_, which means an unmanaged and unorganized data.
* First, divide the data into multiple layers, also called zones. These zones can be : 
    1. __Raw Layer__: Landing zone for source data into the data lake.
    2. __Conformed Layer__: where different source file formats are changed to a uniform format, such as csv, json to parquet.
    3. __Cleansed Layer__: transform data to implement the standardization in data. 
    4. __Presentation Layer__: implement the business logics, like aggregations.
    5. __Sandbox Layer__: playground for data science team, so that data is not mixed with the production ready data.
* Second, create folder structure, this can help in the following:
    1. __Data Segregation__: based on source, business type, or data type.
    2. __Access Control__
    3. __Data Lifecycle Management__
    4. __Performance Optimisation__: can help in certain cases where relative data is grouped together. I can only think of z-ordering, will check with gpt.
    5. __Data Versioning__: time travel feature comes to mind in delta tables.
    6. __Backup and Disastor Recovery__: again time travel feature
    7. __Metadata Management__: used to segregate and manage metadata from raw data.
    8. __Data Partitioning__
    9. __Compliance__: can help with region specific compliance.

## Ch 6 - Data Storage Solution and Processes

##### Different Data Storage
1. _Data Mart_: Subset of DW, where the data is specific to a department or a business line. This helps in forming specific governance, better control of data. eg. a Finance and HR department can each have its own Data Mart managed by themselves. 
2. _Operational Data Store (ODS)_: Unlike Data Mart which is more detailed, ODS is more on a high level. It has current or near real time operational data. Data Mart has all the data related to that specific dept, whereas ODS has Enterprise level data. Not optimised for historical or trend analysis, that is done in DW, and might have basic transformations. eg. imagine going to a store to buy PS5 but they are out of stock, but the sales rep quickly checks the inventory of a different branch.
3. _Data Hubs_: Centralized data storage and management system, that collects, integrate, store, organise and share data. Similar to Data Lake, except Data Lake is used for big data and analytics.
4. _Data Catalogue_: Consists only of metadata of tables, used for data discovery and metadata management.
5. _Data Marketplace_: a place to sell, share and buy data.

##### Data Processes
* Different stratergies and solution to capitalize, transform and govern the data.
1. _Master Data Management (MDM)_: Involves creation of a single master record for each person or item in business across internal and external sources.

## Ch 7 - Approaches to Design

##### Online Transaction Processing (OLTP)
* System which process CRUD operations (Create, Read, Update, Delete).
* Which support multiple transactions at the same time -> high concurrency.
* Which use relational model and have low latency -> fast operations.
* Normalized database (3NF)
* Smaller than OLAP (few GBs)

##### Online Analytic Processing (OLAP)
* Used for reporting and analytics
* Optimized for fast reads, follows write once and read many principle.
* Denormalized database
* Bigger database size than OLTP (TBs)

##### Symmetric MultiProcessing vs Massive Parallel Processing
* _Symmetric Multiprocessing_ : think MS SQL or Oracle where multiple CPUs that share the same memory and disk. This is an example of __scale up__. 
* _Massively Parallel Processing_ : Think distributed system, and how name node and worker node work. This is an example of __scale out__.

##### Lambda Architecture
* 
##### END
##### OF
##### NOTES