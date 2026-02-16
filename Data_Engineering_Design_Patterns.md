## Index
1. [Introducing Data Engineering Design Patterns](#Introducing-Data-Engineering-Design-Patterns)
2. [Data Ingestion Design Patterns](#Data-Ingestion-Design-Patterns)

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