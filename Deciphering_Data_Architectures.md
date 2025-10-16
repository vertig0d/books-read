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
* 
