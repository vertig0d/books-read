# Deciphering Data Architectures - James Serra
![Book Cover](https://learning.oreilly.com/covers/urn:orm:book:9781098150754/400w/)

## Chapter 2 - Types of Data Architecture
* In relational database, consistency and data integrity is important and hence it follows schema-on-write.
* Schema-on-read more flexibility in storing unstructured and semi structured data. Commonly used in Data Lake.
##### Relational Data Warehouse
* Relational Data Warehouse is a type of database that is optimized for analytics and holding large data.
* RDW has both compute and storage. whereas cloud based warehouses have them seperate, this helps in scaling independently. This also means that in RDW since they both are tightly coupled we need to upgrade both or some hardware accompanying it.
* Important features of RDW include:
    1. Trasaction support
    2. Audit trail
    3. schema enforcement.

##### Data Lake
* A glorified filesystem, no different from file storage in laptop.
* Has no compute engine associated.
* Unlike RDW which uses relational storage, Data Lake uses object storage.